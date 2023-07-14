# 谢晨

## HR关键词

全日制本科 ｜ 计算机相关专业 ｜ 3年开发经验 ｜Golang ｜ Python ｜ Kafka

## 自我评价

## 技术栈

**Golang**：Gin ｜  
**Python**：Django ｜ Flask ｜ FastAPI ｜ Celery ｜

## Contact

\| [GitHub](htttps://github.com/timqian) \| [Twitter](https://twitter.com/tim_qian) \| [Email](mailto:timqian@t9t.io) \| [Website](https://timqian.com) \| [Blog](https://blog.t9t.io) \| [Patreon](https://www.patreon.com/timqian) \|

## 技术栈

* Backend: Node.js; Express; AWS lambda; serverless; GraphQL; RabbitMQ
* Database: DynamoDB; MongoDB; Postgres; SQLite; AWS RDS; Redis
* Frontend: react.js; SASS/CSS; d3.js
* Chrome Extension: react.js; SASS/CSS;


## 工作经历
* <b>[pendanaan.com](https://pendanaan.com/)</b> 东南亚互联网金融独角兽 *(2022/08 - 2023/07)*  
  **后端开发 + 数仓**  
  我在后端开发和数据仓库的职责中展现了广泛的技能，包括数据库管理、报表优化、风控模型开发，以及优化数据追踪体系等。
  * SQL语句运行速度提升10+倍，方便实时数据分析和报表，基于阿里云AnalyticDB MySQL，从0到1搭建分析数据库，并迁移数据。
  * Apache DolphinScheduler平台 SQL跑批任务修改为使用阿里云ADB，根据需求新增报表及优化慢查询
  * **风控模型**：参与贷前贷后模型变量开发，为风控模型提供参数变量，并记录变量值，提升公司风控管理水平。
  * **埋点数据落库**：优化了移动应用的数据追踪体系，Appsflyer埋点数据全量（1000w+/天）落库，使用nginx日志保存数据，并调研使用Firebase埋点备用方案流程。
  * **客服系统**：基于WhatsApp实现客服系统，记录客服聊天内容，实现催收批量发送模版消息、历史聊天记录查询，以提高客服效率和方便绩效考核。
  * **异常报警**：通过 Grafana 定时执行SQL，监控系统运行状态，出现异常时推送给API服务，转成模版格式推送到相应的群组。
  * **招聘&员工管理系统**：基于飞书多维表格，设计并实施HR系统自动化解决方案，包括面试时间同步生成等模块，以提高招聘效率。

  离职原因：工作中的任务碎片化程度较高，这使我难以深入研究和精进于某一特定领域。公司的工作节奏过度侧重于加班的过程而非实际产出，这与我对效率和结果导向的工作理念有所冲突。

---
* <b>[jinse.com](www.jinse.com)</b> 国内头部区块链资讯平台 *(2020/12 - 2022/07)*  
  **后端开发工程师**  
  金色财经行情板块负责人，全程参与技术可行性分析、流程设计、数据库表设计、业务开发、测试到部署上线。主导并执行交易所行情等相关数据的清洗入库以及输出，向主流搜索引擎提供高并发、低延时的数字货币价格和K线图趋势数据。
  * **数据获取与清洗**： 查阅交易所接口文档，通过 WebSocket 协议获取交易所分钟粒度行情信息，清洗成 OHLCV 蜡烛图格式，推送至消息队列。
  * **数据聚合与展示**： 从消息队列取出数据，聚合产生不同时间粒度的数据，存入数据库分区表；通过WebSocket服务，为Web端和APP端用户提供实时更新的K线。
  * **性能优化**： 提升API服务并发量，通过增加访问量大的API接口缓存，使用pgSQL分区表，降低数据库访问频次，实现常用接口响应时间不超过200ms，QPS突破1000。
  * **特色功能开发**：
    * 针对指定币种剧烈波动及整数播报，推送到资讯板块及APP推送弹窗。
    * 提供加密货币市值榜单，定时计算加密货币市值及排名，存入数据库。
    * 聚合多家交易所币种价格，为搜索引擎高并发提供主流数字货币趋势。[点击查看>>>](https://m.baidu.com/s?word=BTC)
  * **开源贡献**： 为满足业务需要，为GitHub [ormar](https://github.com/collerek/ormar) 开源项目增加数据类型。 [点击查看>>>](https://github.com/collerek/ormar/releases/tag/0.10.16)

  离职原因：监管政策变化，负责业务无法满足政策合规要求。

---
* <b>鼓豆学堂</b> 面向 k12 的课外辅导和兴趣教育平台 *(2019/10 - 2020/11)*  
  **实习** 技术栈：Django


## 教育经历
* **大连东软信息学院** 计算机科学与技术系 物联网工程 全日制本科 *(2016.09 - 2020.06)*

<a href="resume.pdf" download="My_Resume">
  <button id="downloadButton" style="
    background-color: #4CAF50; /* Green */
    border: none;
    color: white;
    padding: 15px 30px;
    text-align: center;
    text-decoration: none;
    display: inline-block;
    font-size: 16px;
    margin: 4px 2px;
    cursor: pointer;
    border-radius: 12px;
    box-shadow: 0 4px 8px 0 rgba(0,0,0,0.2), 0 6px 20px 0 rgba(0,0,0,0.19);
    transition-duration: 0.4s;
  " onmouseover="hoverOn(this)" onmouseout="hoverOff(this)">
    点击下载PDF简历
  </button>
</a>

<script>
  function hoverOn(element) {
    element.style.backgroundColor = "#45a049"; // Darker green
  }

  function hoverOff(element) {
    element.style.backgroundColor = "#4CAF50"; // Original green
  }
</script>
