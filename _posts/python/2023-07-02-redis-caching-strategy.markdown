---
layout:       post
title:        "优化 Python Redis 缓存：通用的Redis缓存”装饰器“"
subtitle:     "How will property taxes change our lives?"
header-img:   "img/post/property-tax-system.png"
date:         2023-07-02 00:00:00
author:       "Xic"
tags:
    - Python
    - Redis
---
# 介绍
在日常工作中，我们经常会遇到需要对函数结果进行缓存的场景，尤其是在处理网站首页的资讯、文章、行情数据等时。这些情况中，由于返回值的重复率较高，同时伴随着大量的 IO 操作，不可避免地对数据库产生了巨大的压力，甚至可能形成性能瓶颈。  
我个人在处理这类问题时，通常的做法是对接口的返回值或者中间的某些通用值使用 Redis 进行缓存。在函数调用时，首先会在 Redis 中寻找对应的数据，如果找到了直接返回，反之再执行函数。
最初，我是直接在函数内部增加这些处理方法，但随着时间推移，我发现这些代码过于通用，而且在维护上也较为困难。因此，在编写了几次类似代码后，我决定采用装饰器的形式来处理这个问题。然而，在编写代码的过程中，我遇到了很多的问题。通过不断地修改和优化，最终，我实现了一个我认为完美的缓存装饰器。
下面是我所实现的代码，如果你对这个过程的设计思路感兴趣的话，可以继续阅读下文，如果你觉得有什么改进的空间，欢迎在评论区留言。  

# 代码
```python
import json
import asyncio
import logging
import time

import aioredis
from functools import wraps
from hashlib import md5

redis_engines = {}


async def _get_engine() -> aioredis.Redis:
    loop = asyncio.get_event_loop()
    if loop not in redis_engines:
        redis_engines[loop] = await aioredis.from_url('redis://:123456@127.0.0.1:6379')
    return redis_engines[loop]


def cache(
    ttl: int = 60 * 60 * 24,
    time_out: float = None,
    blocking_timeout: float = None,
    ignored_args_: list = None,
    cache_none=True,
):
    """
    缓存装饰器
    :param ttl: 缓存时间
    :param time_out: 超时时间
    :param blocking_timeout:
    :param ignored_args_: 需要
    :param cache_none: 缓存函数结果为 none 的值
    :return:
    """
    def outer(func):
        @wraps(func)
        async def wrapper(*args, **kwargs):
            ignored_args = ignored_args_ or []
            engine = await _get_engine()
            kwargs_for_key = {k: v for k, v in kwargs.items() if k not in ignored_args}
            key = f"{func.__module__}:{func.__class__.__name__}:{func.__name__}:" \
                  f"{md5(json.dumps((args, kwargs_for_key), default=str).encode()).hexdigest()}"

            async def get_cached_result():
                ret = await engine.get(key)
                if ret is not None:
                    return json.loads(ret)
                return None

            async def cache_redis():
                if cache_none or res is not None:
                    # 5.1 当返回值不为 None 或者需要缓存 None 时，将返回值缓存到 redis 中
                    await engine.set(key, json.dumps(res))
                    await engine.expire(key, int(ttl))
                else:
                    # 5.2 如果函数结果为 None 且不需要缓存 None，不做任何操作
                    logging.info(f"Function result is None and cache_none is False, do not cache the result")

            # 1. 先检查一次数据是否在缓存中
            res = await get_cached_result()
            if res is not None:
                return res

            # 2.1 申请该key值的锁，防止缓存击穿
            try:
                async with engine.lock(f'lock:{key}', timeout=time_out, blocking_timeout=blocking_timeout):
                    # 3. 再检查一次数据是否在缓存中
                    res = await get_cached_result()
                    if res is not None:
                        return res

                    # 4. 执行装饰的函数
                    res = await func(*args, **kwargs)
                    await cache_redis()
            # 2.2 如果 blocking_timeout 秒内申请不到锁
            except aioredis.exceptions.LockError:
                # 3. 再检查一次数据是否在缓存中
                res = await get_cached_result()
                if res is not None:
                    return res

                # 4. 执行装饰的函数
                res = await func(*args, **kwargs)
                await cache_redis()
            except Exception as e:
                logging.error(f"Function {func.__name__} with parameters {args} {kwargs} raised an exception: {e}")

            logging.info(f"Function {func.__name__} with parameters {args} {kwargs} returned {res}")
            return res
        return wrapper
    return outer
```
# 测试用例
```python
@cache(ttl=60)
async def test_dict(domain_name: str, website_name: str) -> dict:
    print('test_dict运行')
    return {
        "domain_name": domain_name,
        "website_name": website_name
    }


@cache(ttl=60)
async def test_string(domain_name: str) -> str:
    print('test_string运行')
    return domain_name


@cache(ttl=60)
async def test_int(num: int) -> int:
    print('test_int运行')
    return num


@cache(ttl=60)
async def test_float(num: float) -> float:
    print('test_float运行')
    return num
@cache()
async def fab(n):
    if n <=2:
        return n
    return await fab(n-2) + await fab(n-1)


# 测试代码
async def main():

    await test_dict('www.godev.me', 'Xic')
    assert await test_dict('www.godev.me', 'Xic') == {"domain_name": 'www.godev.me', "website_name": 'Xic'}

    await test_string('www.godev.me')
    assert await test_string('www.godev.me') == 'www.godev.me'

    await test_int(1998)
    assert await test_int(1998) == 1998

    await test_float(10.25)
    assert await test_float(10.25) == 10.25

    start_time = time.time()
    await fab(900)
    print("非波那契数列运行时间>>>", time.time() - start_time)


asyncio.run(main())
```
# 功能描述
1. 在函数执行前，它首先会在 Redis 中查找是否存在已缓存的数据。如果找到了，就直接返回这个数据，否则继续进行下一步。
2. 为了防止缓存击穿，当缓存中没有找到数据时，它会尝试获取一个锁，以同步访问数据库的操作。锁的名称是由 'lock:' 和缓存的键名组成的。
3. 获取到锁之后，它会再次查找 Redis 中是否存在已缓存的数据，以避免在等待锁的过程中数据已经被其他进程查询并缓存。如果找到了，就直接返回这个数据。
4. 如果再次查找依然没有找到已缓存的数据，它会执行被装饰的函数，并将函数的返回结果缓存到 Redis 中。
5. 如果函数的执行结果为 None，根据 cache_none 参数的设定，它会决定是否将这个结果也缓存到 Redis 中。如果 cache_none 为 True，或者函数返回结果不为 None，它会将结果缓存到 Redis 中。
6. 这个装饰器也提供了一些错误处理机制。如果在获取锁或执行被装饰的函数时出现了异常，它会捕获这些异常并记录到日志中。
7. 这个装饰器允许你设置各种参数，包括缓存的有效期 (ttl)、获取锁的超时时间 (time_out)、阻塞的超时时间 (blocking_timeout)、被忽略的参数 (ignored_args_)，以及是否缓存 None 结果 (cache_none)。

这是一个非常强大的缓存工具，它使用了 Redis 作为缓存存储，并使用了锁来保证操作的原子性。

# 问题描述
## 处理多种数据类型的返回值
> 函数的返回值可能是多种类型，因此我们需要找到一个通用的解决方案来处理这些不同类型的返回值。
### 解决方案
为了统一处理，我决定在代码中使用 `json.dumps()` 函数来将数据序列化并存储到 Redis 中，然后用 `json.loads()` 来解析 Redis 中的数据。
## 避免缓存击穿
> 当某个热点数据过期时，大量请求可能会同时穿透缓存直接向数据库发起请求，这可能会导致数据库压力剧增，这个现象我们称之为"缓存击穿"。
### 解决方案
使用锁来同步操作，当一个热点数据过期，只有获得锁的请求可以去数据库查询并更新缓存，其他的请求则需等待缓存更新完成后再进行查询。
## 防止缓存穿透
> 在缓存系统中，如果我们查询的数据不存在，那么每次查询都会去数据库进行检索，这种情况就失去了使用缓存的意义。
### 解决方案
我们可以设定一个空对象（例如：`None`）到缓存中，当查询一个不存在的数据时，即使在数据库中也没有找到，我们仍然可以将这个空对象缓存起来，这样下一次相同的查询就能直接在缓存中找到这个空对象，从而避免了对数据库的访问。
## 锁的性能瓶颈
> 在高并发的环境下，每次获取和释放锁可能会成为性能瓶颈，这是因为锁的获取和释放本身需要消耗时间和资源。
## 解决方案
为了减少锁的使用，我决定使用被称为"双检锁(double-checked locking)"的设计模式。在获取锁之前，我们首先检查数据是否已经在缓存中，如果存在，我们就直接返回数据；如果不存在，我们再获取锁，然后再次检查数据是否在缓存中。这样，当数据已经在缓存中时，我们就不需要获取和释放锁，从而提高性能。

# 写在最后
这个装饰器启发我思考如何利用 Python 的高级特性，比如装饰器和异步编程，来设计并实现出高效且健壮的代码。  
也希望这篇文章能对你有所帮助，能够帮助你解决在实际开发过程中遇到的缓存相关问题，特别是针对缓存击穿和缓存穿透的解决方案，以及如何高效地处理返回值的多样性。文章中所提到的方法，主要是通过使用 Redis 和 Python 的装饰器以及异步编程技术来实现。  
这个缓存装饰器函数不仅能帮助你优化应用程序的性能，降低数据库压力，还提供了非常灵活的参数设置，允许你根据自身的需求来进行定制。  
如果你在使用过程中遇到任何问题，或者有任何建议和反馈，欢迎在评论区留言讨论。  
最后，祝你在编程的路上越走越远，不断提高，不断创新。