---
layout:       post
title:        "MySQL存储引擎：InnoDB与MyISAM的比较"
subtitle:     "MySQL storage engine: comparison of InnoDB and MyISAM"
header-img:   "img/in-post/head/2023-07-21-mysql-Innodb-and-myisam.png"
date:         2023-07-21 00:00:00
author:       "Xic"
tags:
    - 数据库
    - MySQL
    - InnoDB
    - MyISAM
---
MySQL中两种常见的存储引擎是InnoDB和MyISAM。它们在事务支持，锁定机制，数据完整性和恢复，以及存储空间和全文索引等方面都有所不同。

## 事务支持

- InnoDB提供了对事务的支持，具有ACID事务特性（原子性，一致性，隔离性和持久性）。
- MyISAM不支持事务，每次查询被视为一个独立的事务。

## 锁定机制

- MyISAM只支持表级锁定，即每次锁定的都是整张表，不适合高并发的写操作。
- 而InnoDB支持行级锁定，也支持外键，更适合处理大量数据和高并发的情况。

## 数据完整性和外键支持

- InnoDB支持外键，并通过外键约束来维护数据的引用完整性。MyISAM则不支持。

## 崩溃后的数据恢复

- InnoDB通过其日志文件和事务特性提供更好的崩溃恢复。
- MyISAM在MySQL崩溃后可能会丢失数据或者导致数据不一致。

## 存储空间

- InnoDB需要更多的磁盘空间来保存数据的完整性和高效性。InnoDB表会预留一部分空间用于插入新的数据和索引。
- MyISAM表仅使用实际数据量的存储空间。

## 全文索引

- 在MySQL 5.6版本之前，只有MyISAM支持全文索引。
- 从MySQL 5.6开始，InnoDB也开始支持全文索引。

## 索引
- 索引类型：InnoDB和MyISAM都支持主键索引（PRIMARY KEY）、唯一索引（UNIQUE KEY）、全文索引（FULLTEXT，在MySQL 5.6及以上版本InnoDB开始支持）和普通索引（INDEX）。
- 索引结构：MyISAM使用B+Tree作为索引结构，叶节点的data域存放的是数据记录的地址。InnoDB也使用B+Tree作为索引结构，但是它的叶节点data域对于主键索引来说存放的是数据记录本身，对于非主键索引来说存放的是对应主键的值。
- 聚簇索引：InnoDB中的数据按照主键聚集在一起，这种存储方式称为聚簇索引。MyISAM中的表数据是按照插入顺序存放的，索引与数据是分开的。
- 主键选择：对InnoDB来说，如果表中没有显式定义主键，MySQL系统会选择一个能够唯一标识数据行的列作为主键，如果不存在这种列，MySQL会生成一个隐藏的主键。而MyISAM不会这样，MyISAM允许你创建没有任何索引或者主键的表。

选择InnoDB还是MyISAM，取决于你的具体需求。如果需要处理大量数据并发，需要事务支持，对数据完整性要求较高，那么InnoDB可能是更好的选择。如果你只是进行简单的读取操作，数据一致性和完整性要求不是那么高，那么MyISAM可能会更适合。
