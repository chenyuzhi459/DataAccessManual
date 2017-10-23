# 数据接入

数据接入主要有两个比较重要的参数，一是从哪里接入，可以从 kafka hadoop 本地 等接入，二是接入何种类型的数据，比较常用的有
json csv url regex等，


- [LuceneKafkaSupervisorSpec](#LuceneKafkaSupervisorSpec) 
- [DefaultKafkaSupervisorSpec](#DefaultKafkaSupervisorSpec) 
- [LuceneMetaqSupervisorSpec](#LuceneMetaqSupervisorSpec) 
- [LuceneMetaqIndexTask](#LuceneMetaqIndexTask) 
- [LuceneKafkaIndexTask](#LuceneKafkaIndexTask) 
- [LuceneIndexRealtimeTask](#LuceneIndexRealtimeTask) 
- [BatchKafkaIndexTask](#BatchKafkaIndexTask) 
- [HadoopLuceneIndexTask](#HadoopLuceneIndexTask) 
- [LuceneIndexTask](#LuceneIndexTask) 
- [LuceneKafkaSupervisorSpec](#LuceneKafkaSupervisorSpec) 


## <a id="LuceneKafkaSupervisorSpec" href="LuceneKafkaSupervisorSpec"></a> LuceneKafkaSupervisorSpec
LuceneKafkaSupervisorSpec用来从kafka上面接入数据。

```
{
  "type":"lucene_supervisor",
  "dataSchema":{
    "dataSource":"wiki",
    "parser":{
      "type":"string",
      "parseSpec":{
        "format":"url",
        "timestampSpec":{
          "column":"the_date",
          "format":"millis"
        },
        "dimensionSpec":{
          "dimensions":[
            {
              "name":"name",
              "type":"string"
            },
            {
              "name":"age",
              "type":"int"
            },
            {
              "name":"score",
              "type":"float"
            },
            {
              "name":"create_time",
              "type":"date",
              "format":"yyyy-MM-dd HH:mm:ss"
            }
          ]
        }
      }
    },
    "metricsSpec":"aggregators",
    "granularitySpec":{
      "type":"uniform",
      "segmentGranularity": "YEAR",
      "queryGranularity": <queryGranularity>,
      "rollup": false,
      "intervals": [<interval>]
    }
  },
  "tuningConfig":{
    "type":kakfa 
    "maxRowsInMemory": "...",
    "maxRowsPerSegment": "...",
    "intermediatePersistPeriod": "...",
    "basePersistDirectory": "...",
    "basePersistDirectoryStr": "...",
    "maxPendingPersists": "...",
    "indexSpec":{
      "bitmap": {
        "type"： "concise"
      },
      "dimensionCompression": "dimensionCompression_string",
      "metricCompression": "metricCompression_string"
    },
    "buildV9Directly": "...",
    "reportParseExceptions": "...",
    "handoffConditionTimeout": "...",
    "workerThreads": "...",
    "chatThreads": Integer chatThreads,   
    "chatRetries": Long chatRetries,
    "httpTimeout": Period httpTimeout,
    "shutdownTimeout": Period shutdownTimeout,
    "maxWarmCount": Integer maxWarmCount, 
    "consumerThreadCount": "..."
  },
  "ioConfig":{
    "topic": "wiki",
    "replicas": 1,
    "taskCount": 1,
    "taskDuration": "PT3600S",
    "consumerProperties": {
      "bootstrap.servers": "192.168.0.101:9092,192.168.0.102:9092,192.168.0.103:9092"
    },
    "startDelay": "...",
    "period": Period period,
    "useEarliestOffset": true,
    "completionTimeout": "...",
    "lateMessageRejectionPeriod": Period lateMessageRejectionPeriod
  },
  "writerConfig":{
    "maxBufferedDocs": Integer maxBufferedDocs,
    "ramBufferSizeMB": Double ramBufferSizeMB,
    "indexRefreshIntervalSeconds": Long indexRefreshIntervalSeconds,
    "isIndexMerge": Boolean isIndexMerge,
    "isCompound": Boolean isCompound,
    "maxMergeAtOnce"： Integer maxMergeAtOnce,
    "maxMergedSegmentMB"： Integer maxMergedSegmentMB,
    "maxMergesThreads": Integer maxMergesThreads,
    "mergeSegmentsPerTire": Double mergeSegmentsPerTire,
    "writeThreads": Integer writeThreads,
    "limiterMBPerSec": Double limiterMBPerSec
  }
}
```
- **`type:`** 从kafka加载数据到druid时固定配置为lucene_supervisor
- **`dataSchema.dataSource`**  数据源的名称，类似关系数据库中的表名
- **`dataSchema.parser.type`**  固定string
- **`dataSchema.parser.parseSpec.format`** kafka中单条数据的格式，比如`url/json/csv/tsv/hive`。

- **`url:`** 数据类似`url`参数串，如：`name=jack&age=21&score=78.5&the_date= 1489131528601`

- **`dataSchema.parser.parseSpec.timestampSpec:`** 时间戳列以及时间的格式  
- **`dataSchema.parser.parseSpec.timestampSpec.format`** 时间格式类型：推荐`millis`  
	> `yy-MM-dd HH:mm:ss`: 自定义的时间格式  
	> `auto`: 自动识别时间，支持iso和millis格式  
  > `iso`：iso标准时间格式，如”2016-08-03T12:53:51.999Z”  
  > `posix`：从1970年1月1日开始所经过的秒数,10位的数字  
  > `millis`：从1970年1月1日开始所经过的毫秒数，13位数字  
- **`dataSchema.parser.parseSpec.dimensionsSpec.dimensions:`** 维度定义列表，每个维度的格式为：`{“name”: “age”, “type”:”string”}`。Type支持的类型：`string`、`int`、`float`、`long`、`date`  

- **`json:`** json格式数据
	- 数据格式：`{"name":"jack","age": 21,"score": 78.5,"the_date": 1489131528601}`

- **`csv：`** csv格式数据  
	- **`dataSchema.parser.parseSpec.listDelimiter：`** 分隔符
	- **`dataSchema.parser.parseSpec.columns：`** 维度列表，格式：`["name","age","score"]`  

- **`hive：`** hive数据文件
	- **`dataSchema.parser.parseSpec.timestampSpec`**
	- **`dataSchema.parser.parseSpec.dimensionsSpec`**
	- **`dataSchema.parser.parseSpec.separator:`** 维度分隔符
	- **`dataSchema.parser.parseSpec.columns:`** 维度列表

- **`dataSchema.granularitySpec.type:`** 默认使用`uniform`
- **`dataSchema.granularitySpec.segmentGranularity:`** 段粒度，根据每天的数据量进行设置。  
小数据量建议`DAY`，大数据量（每天百亿）可以选择`HOUR`。可选项：`SECOND`、`MINUTE`、`FIVE_MINUTE`、`TEN_MINUTE`、`FIFTEEN_MINUTE`、`HOUR`、`SIX_HOUR`、`DAY`、`MONTH`、`YEAR`。

- **`tuningConfig:`** 用于优化数据摄取的过程
- **`tuningConfig.type:`** 设置为`kafka`
- **`tuningConfig.maxRowsInMemory:`** 在存盘之前内存中最大的存储行数，指的是聚合后的行数
- **`tuningConfig.maxRowsPerSegment:`** 每个segment最大的存储行数
- **`tuningConfig.intermediatePersistPeriod:`** 多长时间数据临时存盘一次
- **`tuningConfig.basePersistDirectory:`** 任务中接收到的数据临时存放路径，需要注意用户的读写权限。
- **`tuningConfig.basePersistDirectoryStr:`** 临时存盘目录
- **`tuningConfig.maxPendingPersists:`** 最大同时存盘请求数，达到上限，摄取将会暂停

- **`tuningConfig.indexSpec.bitmap:`** 索引服务说明
- **`tuningConfig.indexSpec.bitmap.type:`** 使用哪种类型的位图索引，默认是concise类型，也可手动指定为roaring类型
- **`tuningConfig.indexSpec.dimensionCompression:`** 指定维度列的压缩格式，允许为null
- **`tuningConfig.indexSpec.metricCompression:`** 指定指标列的压缩格式，允许为null
- **`tuningConfig.buildV9Directly:`** 是否直接构建v9版本的索引
- **`tuningConfig.reportParseExceptions:`** 是否汇报数据解析错误
- **`tuningConfig.handoffConditionTimeout:`** 切换环境多长时间超时
- **`tuningConfig.workerThreads:`** worker线程数
- **`tuningConfig.httpTimeout:`** http传输的超时时间
- **`tuningConfig.shutdownTimeout:`** shutdown执行多长时间超时
- **`tuningConfig.consumerThreadCount:`** 消费者线程数

- **`ioConfig:`** 指明真正的具体的数据源  
- **`ioConfig.topic:`** kafka中的topic名  
- **`ioConfig.replicas:`** 任务的副本数  
- **`ioConfig.taskCount:`** 启动的任务进程数  
- **`ioConfig.taskDuration:`** 任务持续时间，超过指定时间后，任务会停止接收数据，等数据持久化之后会创建新的任务进程。可设置的格式：一分钟：`PT60S`, 十分钟：`PT10M`, 一天：`PT1D`  
- **`ioConfig.consumerProperties:`** kafka消费端接口的配置，比如kafka的服务器配置  
- **`ioConfig.startDelay:`** 开始时延
- **`ioConfig.useEarliestOffset:`** 从kafka的最早的`offset`开始消费 
- **`ioConfig.completionTimeout:`** 完成花费多长时间超时


## <a id="DefaultKafkaSupervisorSpec" href="DefaultKafkaSupervisorSpec"></a> DefaultKafkaSupervisorSpec
```
{
  "type":"default_supervisor",
  "dataSchema":{
    "dataSource":"dataSource_string",
    "parser":[<parser>],
    "metricsSpec":"aggregators",
    "granularitySpec":{
      "type":"uniform",
      "segmentGranularity": <segmentGranularity>,
      "queryGranularity": <queryGranularity>,
      "rollup": false,
      "intervals": [<interval>]
    }
  },
  "tuningConfig":{
    "maxRowsInMemory": "...",
    "maxRowsPerSegment": "...",
    "intermediatePersistPeriod": "...",
    "basePersistDirectory": "...",
    "basePersistDirectoryStr": "...",
    "maxPendingPersists": "...",
    "indexSpec":{
      "bitmap": {
        "type"： "concise"
      },
      "dimensionCompression": "dimensionCompression_string",
      "metricCompression": "metricCompression_string"
    },
    "buildV9Directly": "...",
    "reportParseExceptions": "...",
    "handoffConditionTimeout": "...",
    "workerThreads": "...",
    "chatThreads": Integer chatThreads,  
    "chatRetries": Long chatRetries,
    "httpTimeout": Period httpTimeout,
    "shutdownTimeout": Period shutdownTimeout,
    "maxWarmCount": Integer maxWarmCount, 
    "consumerThreadCount": "..."
  },
  "ioConfig":{
    "topic": "...",
    "replicas": Integer replicas,
    "taskCount": "...",
    "taskDuration": "...",
    "consumerProperties": "...",
    "startDelay": "...",
    "period": Period period,
    "useEarliestOffset": true,
    "completionTimeout": "...",
    "lateMessageRejectionPeriod": Period lateMessageRejectionPeriod
  },
  "writerConfig":{
    "maxBufferedDocs": Integer maxBufferedDocs,
    "ramBufferSizeMB": Double ramBufferSizeMB,
    "indexRefreshIntervalSeconds": Long indexRefreshIntervalSeconds,
    "isIndexMerge": Boolean isIndexMerge,
    "isCompound": Boolean isCompound,
    "maxMergeAtOnce"： Integer maxMergeAtOnce,
    "maxMergedSegmentMB"： Integer maxMergedSegmentMB,
    "maxMergesThreads": Integer maxMergesThreads,
    "mergeSegmentsPerTire": Double mergeSegmentsPerTire,
    "writeThreads": Integer writeThreads,
    "limiterMBPerSec": Double limiterMBPerSec
  },
  "dataSource":"dataSource_string",
  "dimensionsSpec":"",
  "platform":"platform_string", 
  "parseSpec":{},
  "ipLibSpec":{}
}
```
- **`type:`** 固定配置为`default_supervisor`
- **`dataSchema.dataSource`**  数据源的名称，类似关系数据库中的表名
- **`dataSchema.parser`**  包含如何解析数据的相关内容
- **`dataSchema.metricsSpec`**  所有的指标列和所使用的聚合函数

- **`dataSchema.granularitySpec:`** 数据的存储和查询粒度
- **`dataSchema.granularitySpec.type:`** 默认使用`uniform`
- **`dataSchema.granularitySpec.segmentGranularity:`** 段粒度，根据每天的数据量进行设置。  
小数据量建议`DAY`，大数据量（每天百亿）可以选择`HOUR`。可选项：`SECOND`、`MINUTE`、`FIVE_MINUTE`、`TEN_MINUTE`、`FIFTEEN_MINUTE`、`HOUR`、`SIX_HOUR`、`DAY`、`MONTH`、`YEAR`。
- **`dataSchema.granularitySpec.queryGranularity:`** 查询粒度  
- **`dataSchema.granularitySpec.rollup:`** 是否进行汇总统计  
- **`dataSchema.granularitySpec.intervals:`** 区间  

- **`tuningConfig:`** 用于优化数据摄取的过程
- **`tuningConfig.type:`** 设置为`kafka`
- **`tuningConfig.maxRowsInMemory:`** 在存盘之前内存中最大的存储行数，指的是聚合后的行数
- **`tuningConfig.maxRowsPerSegment:`** 每个segment最大的存储行数
- **`tuningConfig.intermediatePersistPeriod:`** 多长时间数据临时存盘一次
- **`tuningConfig.basePersistDirectory:`** 任务中接收到的数据临时存放路径，需要注意用户的读写权限。
- **`tuningConfig.basePersistDirectoryStr:`** 临时存盘目录
- **`tuningConfig.maxPendingPersists:`** 最大同时存盘请求数，达到上限，摄取将会暂停

- **`tuningConfig.indexSpec.bitmap:`** 索引服务说明
- **`tuningConfig.indexSpec.bitmap.type:`** 使用哪种类型的位图索引，默认是concise类型，也可手动指定为roaring类型
- **`tuningConfig.indexSpec.dimensionCompression:`** 指定维度列的压缩格式，允许为null
- **`tuningConfig.indexSpec.metricCompression:`** 指定指标列的压缩格式，允许为null
- **`tuningConfig.buildV9Directly:`** 是否直接构建v9版本的索引
- **`tuningConfig.reportParseExceptions:`** 是否汇报数据解析错误
- **`tuningConfig.handoffConditionTimeout:`** 切换环境多长时间超时
- **`tuningConfig.workerThreads:`** worker线程数
- **`tuningConfig.httpTimeout:`** http传输的超时时间
- **`tuningConfig.shutdownTimeout:`** shutdown执行多长时间超时
- **`tuningConfig.consumerThreadCount:`** 消费者线程数

- **`ioConfig:`** 指明真正的具体的数据源  
- **`ioConfig.topic:`** kafka中的topic名  
- **`ioConfig.replicas:`** 任务的副本数  
- **`ioConfig.taskCount:`** 启动的任务进程数  
- **`ioConfig.taskDuration:`** 任务持续时间，超过指定时间后，任务会停止接收数据，等数据持久化之后会创建新的任务进程。可设置的格式：一分钟：`PT60S`, 十分钟：`PT10M`, 一天：`PT1D`  
- **`ioConfig.consumerProperties:`** kafka消费端接口的配置，比如kafka的服务器配置  
- **`ioConfig.startDelay:`** 开始时延
- **`ioConfig.useEarliestOffset:`** 从kafka的最早的`offset`开始消费 
- **`ioConfig.completionTimeout:`** 完成花费多长时间超时


## <a id="LuceneMetaqSupervisorSpec" href="LuceneMetaqSupervisorSpec"></a> LuceneMetaqSupervisorSpec
```
{
  "type":"lucene_metaq_supervisor",
  "dataSchema": {
    "dataSource":"dataSource_string",
    "parser":[<parser>],
    "metricsSpec":"aggregators",
    "granularitySpec":{
      "type":"uniform",
      "segmentGranularity": <segmentGranularity>,
      "queryGranularity": <queryGranularity>,
      "rollup": false,
      "intervals": [<interval>]
    }
  },
  "tuningConfig":{
    "maxRowsInMemory": "...",
    "maxRowsPerSegment": "...",
    "intermediatePersistPeriod": "...",
    "basePersistDirectory": "...",
    "basePersistDirectoryStr": "...",
    "maxPendingPersists": "...",
    "indexSpec":{
      "bitmap": {
        "type"： "concise"
      },
      "dimensionCompression": "dimensionCompression_string",
      "metricCompression": "metricCompression_string"
    },
    "buildV9Directly": "...",
    "reportParseExceptions": "...",
    "handoffConditionTimeout": "...",
    "workerThreads": "...",
    "chatThreads": Integer chatThreads,  
    "chatRetries": Long chatRetries,
    "httpTimeout": Period httpTimeout,
    "shutdownTimeout": Period shutdownTimeout,
    "maxWarmCount": Integer maxWarmCount, 
    "consumerThreadCount": "..."
  },
  "ioConfig": {
    "topic": "...",
    "replicas": Integer replicas,
    "taskCount": "...",
    "taskDuration": "...",
    "consumerProperties": "...",
    "startDelay": "...",
    "period": Period period,
    "useEarliestOffset": true,
    "completionTimeout": "...",
    "lateMessageRejectionPeriod": Period lateMessageRejectionPeriod,
    "zkConnect": String zkConnect,
    "zkRoot": String zkRoot,
    "maxFetchSize": Integer maxFetchSize
  },         
  "writerConfig": {
    "maxBufferedDocs": Integer maxBufferedDocs,
    "ramBufferSizeMB": Double ramBufferSizeMB,
    "indexRefreshIntervalSeconds": Long indexRefreshIntervalSeconds,
    "isIndexMerge": Boolean isIndexMerge,
    "isCompound": Boolean isCompound,
    "maxMergeAtOnce"： Integer maxMergeAtOnce,
    "maxMergedSegmentMB"： Integer maxMergedSegmentMB,
    "maxMergesThreads": Integer maxMergesThreads,
    "mergeSegmentsPerTire": Double mergeSegmentsPerTire,
    "writeThreads": Integer writeThreads,
    "limiterMBPerSec": Double limiterMBPerSec
  }
}
```
- **`type:`** 固定配置为`default_supervisor`
- **`dataSchema.dataSource`**  数据源的名称，类似关系数据库中的表名
- **`dataSchema.parser`**  包含如何解析数据的相关内容
- **`dataSchema.metricsSpec`**  所有的指标列和所使用的聚合函数

- **`dataSchema.granularitySpec:`** 数据的存储和查询粒度
- **`dataSchema.granularitySpec.type:`** 默认使用`uniform`
- **`dataSchema.granularitySpec.segmentGranularity:`** 段粒度，根据每天的数据量进行设置。  
小数据量建议`DAY`，大数据量（每天百亿）可以选择`HOUR`。可选项：`SECOND`、`MINUTE`、`FIVE_MINUTE`、`TEN_MINUTE`、`FIFTEEN_MINUTE`、`HOUR`、`SIX_HOUR`、`DAY`、`MONTH`、`YEAR`。
- **`dataSchema.granularitySpec.queryGranularity:`** 查询粒度  
- **`dataSchema.granularitySpec.rollup:`** 是否进行汇总统计  
- **`dataSchema.granularitySpec.intervals:`** 区间  

- **`tuningConfig:`** 用于优化数据摄取的过程
- **`tuningConfig.type:`** 设置为`kafka`
- **`tuningConfig.maxRowsInMemory:`** 在存盘之前内存中最大的存储行数，指的是聚合后的行数
- **`tuningConfig.maxRowsPerSegment:`** 每个segment最大的存储行数
- **`tuningConfig.intermediatePersistPeriod:`** 多长时间数据临时存盘一次
- **`tuningConfig.basePersistDirectory:`** 任务中接收到的数据临时存放路径，需要注意用户的读写权限。
- **`tuningConfig.basePersistDirectoryStr:`** 临时存盘目录
- **`tuningConfig.maxPendingPersists:`** 最大同时存盘请求数，达到上限，摄取将会暂停

- **`tuningConfig.indexSpec.bitmap:`** 索引服务说明
- **`tuningConfig.indexSpec.bitmap.type:`** 使用哪种类型的位图索引，默认是concise类型，也可手动指定为roaring类型
- **`tuningConfig.indexSpec.dimensionCompression:`** 指定维度列的压缩格式，允许为null
- **`tuningConfig.indexSpec.metricCompression:`** 指定指标列的压缩格式，允许为null
- **`tuningConfig.buildV9Directly:`** 是否直接构建v9版本的索引
- **`tuningConfig.reportParseExceptions:`** 是否汇报数据解析错误
- **`tuningConfig.handoffConditionTimeout:`** 切换环境多长时间超时
- **`tuningConfig.workerThreads:`** worker线程数
- **`tuningConfig.httpTimeout:`** http传输的超时时间
- **`tuningConfig.shutdownTimeout:`** shutdown执行多长时间超时
- **`tuningConfig.consumerThreadCount:`** 消费者线程数

- **`ioConfig:`** 指明真正的具体的数据源  
- **`ioConfig.topic:`** kafka中的topic名  
- **`ioConfig.replicas:`** 任务的副本数  
- **`ioConfig.taskCount:`** 启动的任务进程数  
- **`ioConfig.taskDuration:`** 任务持续时间，超过指定时间后，任务会停止接收数据，等数据持久化之后会创建新的任务进程。可设置的格式：一分钟：`PT60S`, 十分钟：`PT10M`, 一天：`PT1D`  
- **`ioConfig.consumerProperties:`** kafka消费端接口的配置，比如kafka的服务器配置  
- **`ioConfig.startDelay:`** 开始时延
- **`ioConfig.useEarliestOffset:`** 从kafka的最早的`offset`开始消费 
- **`ioConfig.completionTimeout:`** 完成花费多长时间超时

## <a id="LuceneMetaqIndexTask" href="LuceneMetaqIndexTask"></a> LuceneMetaqIndexTask
```
{
  "type":"lucene_index_metaq",
  "id": "id_string",
  "resource": {
    "availabilityGroup":"availabilityGroup_string",
    "requiredCapacity":10  
  },
  "dataSchema": {
    "dataSource":"dataSource_string",  
    "parser":[<parser>],  
    "metricsSpec":[],   
    "granularitySpec":{
      "type": "uniform" 
      "intervals": [<interval>] , 
      "segmentGranularity": <segmentGranularity>, 
      "rollup": true | false,
      "queryGranularity"：　<queryGranularity>, 
        
    }  
  },
  "tuningConfig": {
    "maxRowsInMemory": Integer maxRowsInMemory,  
    "maxRowsPerSegment": Integer maxRowsPerSegment,  
    "intermediatePersistPeriod": Period intermediatePersistPeriod,  
    "basePersistDirectory": File basePersistDirectory,  
    "basePersistDirectoryStr": String basePersistDirectoryStr,  
    "maxPendingPersists": Integer maxPendingPersists,  
    "indexSpec":{
      "bitmap": {
        "type"： "type_string",
      },  
      "dimensionCompression": "dimensionCompression_string", 
      "metricCompression": "metricCompression_string" 
    }, 
    "buildV9Directly": true | false,   
    "reportParseExceptions": true | false,  
    "handoffConditionTimeout":  Long handoffConditionTimeout,
    "maxWarmCount": Integer maxWarmCount, 
    "consumerThreadCount": Integer consumerThreadCount
  },
  "ioConfig": {
    "baseSequenceName":  "baseSequenceName_string",
    "startPartitions": {
      "topic"："topic_string" ,   
      "group": "group_string",
      "partitionOffsetMap": Map<String, Long> partitionOffsetMap
    },
    "endPartitions": {
      "topic"："topic_string" ,   
      "group": "group_string",
      "partitionOffsetMap": Map<String, Long> partitionOffsetMap
    },
    "consumerProperties": {},
    "useTransaction": true | false, 
    "pauseAfterRead": true | false,  
    "minimumMessageTime": DateTime minimumMessageTime,
    "zkConnect": String zkConnect,  
    "zkRoot": String zkRoot,
    "maxFetchSize": Integer maxFetchSize   
  },
  "writerConfig": {
    "maxBufferedDocs": Integer maxBufferedDocs,
    "ramBufferSizeMB": Double ramBufferSizeMB,
    "indexRefreshIntervalSeconds": Long indexRefreshIntervalSeconds,
    "isIndexMerge": Boolean isIndexMerge,
    "isCompound": Boolean isCompound,
    "maxMergeAtOnce": Integer maxMergeAtOnce,
    "maxMergedSegmentMB": Integer maxMergedSegmentMB,
    "maxMergesThreads": Integer maxMergesThreads,
    "mergeSegmentsPerTire": Double mergeSegmentsPerTire,
    "writeThreads": Integer writeThreads,  
    "limiterMBPerSec": Double limiterMBPerSec
  },
  "context": Map<String, Object> context
}
```

- **`type:`** 指定数据接入的类型,固定为 `lucene_index_metaq`
- **`id:`** 指定创建的task的名称,若不设置则采用默认生成的task名称
- **`resource`**　关于任务需要的资源说明
- **`resource.availabilityGroup:`** 指定所属的可用性组ID
- **`resource.requiredCapacity:`** task将需要的工人槽数（worker slots）
- **`dataSchema:`** 关于接入数据的概要说明
- **`dataSchema.dataSource:`** 数据源的名称，类似关系数据库中的表名
- **`dataSchema.parser:`** 数据转换器
- **`dataSchema.metricsSpec:`** 所有的指标列和所使用的聚合函数,可以不设置

- **`dataSchema.granularitySpec:`** 数据粒度说明
- **`dataSchema.granularitySpec.type:`** 默认使用uniform
- **`dataSchema.granularitySpec.intervals:`** 数据时间戳范围,可以指定多个范围
- **`dataSchema.granularitySpec.segmentGranularity:`** 段粒度，根据每天的数据量进行设置。 小数据量建议`DAY`，大数据量（每天百亿）可以选择`HOUR`。可选项：`SECOND`、`MINUTE`、`FIVE_MINUTE`、`TEN_MINUTE`、`FIFTEEN_MINUTE`、`HOUR、SIX_HOUR`、`DAY`、`MONTH`、`YEAR`。

- **`dataSchema.granularitySpec.rollup:`** 是否对数据进行上卷，也就是以更大的粒度存储
- **`dataSchema.granularitySpec.queryGranularity:`**　查询粒度

- **`tuningConfig:`** 优化说明
- **`tuningConfig.maxRowsInMemory:`** 在存盘之前内存中最大的存储行数，指的是聚合后的行数
- **`tuningConfig.maxRowsPerSegment:`** 每个segment最大的存储行数
- **`tuningConfig.intermediatePersistPeriod:`** 多长时间数据临时存盘一次
- **`tuningConfig.basePersistDirectory:`** 任务中接收到的数据临时存放路径，需要注意用户的读写权限。
> 注意：`basePersistDirectory`需要保证有相应的权限   
- **`tuningConfig.basePersistDirectoryStr:`** 临时存盘目录
- **`tuningConfig.maxPendingPersists:`** 最大同时存盘请求数，达到上限，摄取将会暂停
- **`tuningConfig.indexSpec:`**　索引说明
- **`tuningConfig.indexSpec.bitmap:`**　`bitmap`说明
- **`tuningConfig.indexSpec.bitmap.type:`**　采用的`bitmap`索引类型，可选项为：`concise`、`roaring`，默认为`concise`
- **`tuningConfig.indexSpec.dimensionCompression:`**　指定维度列的压缩策略，若不设置则默认为`LZ4`策略
- **`tuningConfig.indexSpec.metricCompression:`**　指定度量列的压缩策略，若不设置则默认为`LZ4`策略
- **`tuningConfig.buildV9Directly:`**　是否直接构建v9版本的索引
- **`tuningConfig.reportParseExceptions:`** 是否汇报数据解析错误
- **`tuningConfig.handoffConditionTimeout:`** 切换环境多长时间超时
- **`tuningConfig.consumerThreadCount:`** 消费者线程数

- **`ioConfig:`** 数据的IO说明
- **`ioConfig.baseSequenceName:`** 
- **`ioConfig.startPartitions:`** 消息的起始分区说明
- **`ioConfig.startPartitions.topic:`** 起始分区的 `topic` 名称
- **`ioConfig.startPartitions.group:`** 起始分区的 `group` 名称
- **`ioConfig.startPartitions.partitionOffsetMap:`** 起始分区的 `offset` 设置

- **`ioConfig.endPartitions:`** 消息的结束分区说明
- **`ioConfig.endPartitions.topic:`** 结束分区的 `topic` 名称
- **`ioConfig.endPartitions.group:`** 结束分区的 `group` 名称
- **`ioConfig.endPartitions.partitionOffsetMap:`** 结束分区的 `offset` 设置

- **`ioConfig.consumerProperties:`** 消费者的配置
- **`ioConfig.useTransaction:`** 是否使用事务，默认为 `true`
- **`ioConfig.pauseAfterRead:`** 读取之后是否暂停,默认为 `false`
- **`ioConfig.minimumMessageTime:`** 
- **`ioConfig.zkConnect:`** 指定zookeeper连接
- **`ioConfig.zkRoot:`**
- **`ioConfig.maxFetchSize:`**　最大的数据拉取量


- **`writerConfig:`**　设置task的`LuceneWriterConfig`，也可以不设置
- **`writerConfig.maxBufferedDocs:`**　缓存的最大行数
- **`writerConfig.ramBufferSizeMB:`**　缓存使用的内存数
- **`writerConfig.indexRefreshIntervalSeconds:`**　写磁盘的时间间隔,只有写磁盘后实时数据才可以查询，默认6s
- **`writerConfig.isIndexMerge:`**　是否对索引文件进行压缩存储，默认压缩
- **`writerConfig.isCompound:`**　是否启动`compound`，默认为`false`
- **`writerConfig.maxMergeAtOnce:`**　达到多少记录时立即进行合并，默认不设置
- **`writerConfig.maxMergedSegmentMB:`**　数据达到多少MB时进行合并，默认不设置
- **`writerConfig.maxMergesThreads:`**　最大的合并线程数
- **`writerConfig.mergeSegmentsPerTire:`**　合并时的参数，默认不设置
- **`writerConfig.writeThreads:`**　写线程数
- **`writerConfig.limiterMBPerSec:`**　每秒数据量限制
- **`context:`**　任务上下文环境设置








## <a id="LuceneKafkaIndexTask" href="LuceneKafkaIndexTask"></a> LuceneKafkaIndexTask
```
{
  "type": "lucene_index_kafka",
  "id": "id_string", 
  "resource": {
    "availabilityGroup":"availabilityGroup_string", 
    "requiredCapacity":10 
  },
  "dataSchema": {
    "dataSource":"dataSource_string",  
    "parser":[<parser>],  
    "metricsSpec":[],   
    "granularitySpec":{
      "type": "uniform" 
      "intervals": [<interval>] , 
      "segmentGranularity": <segmentGranularity>, 
      "rollup": true | false,
      "queryGranularity"：　<queryGranularity>, 
    }  
  },
  "tuningConfig": {
    "maxRowsInMemory": Integer maxRowsInMemory,  
    "maxRowsPerSegment": Integer maxRowsPerSegment,  
    "intermediatePersistPeriod": Period intermediatePersistPeriod,  
    "basePersistDirectory": File basePersistDirectory,  
    "basePersistDirectoryStr": String basePersistDirectoryStr,  
    "maxPendingPersists": Integer maxPendingPersists,  
    "indexSpec":{
      "bitmap": {
        "type"： "type_string",
      },  
      "dimensionCompression": "dimensionCompression_string", 
      "metricCompression": "metricCompression_string" 
    }, 
    "buildV9Directly": true | false,   
    "reportParseExceptions": true | false,  
    "handoffConditionTimeout":  Long handoffConditionTimeout,
    "maxWarmCount": Integer maxWarmCount, 
    "consumerThreadCount": Integer consumerThreadCount
  },
  "ioConfig": {
    "type":"kafka"
    "baseSequenceName":  "baseSequenceName_string",
    "startPartitions": {
      "topic"："topic_string" ,   
      "group": "group_string",
      "partitionOffsetMap": {
      }
    },    
    "endPartitions": {
      "topic"："topic_string" ,  
      "group": "group_string",
      "partitionOffsetMap": {}
    },,
    "consumerProperties": {},       
    "useTransaction": true | false, 
    "pauseAfterRead": true | false,  
    "minimumMessageTime": DateTime minimumMessageTime,
  },
  "writerConfig": {
    "maxBufferedDocs": Integer maxBufferedDocs,
    "ramBufferSizeMB": Double ramBufferSizeMB,
    "indexRefreshIntervalSeconds": Long indexRefreshIntervalSeconds,
    "isIndexMerge": Boolean isIndexMerge,
    "isCompound": Boolean isCompound,
    "maxMergeAtOnce": Integer maxMergeAtOnce,
    "maxMergedSegmentMB": Integer maxMergedSegmentMB,
    "maxMergesThreads": Integer maxMergesThreads,
    "mergeSegmentsPerTire": Double mergeSegmentsPerTire,
    "writeThreads": Integer writeThreads,   
    "limiterMBPerSec": Double limiterMBPerSec
  },
  "context": Map<String, Object> context
}
```
- **`type:`** 指定数据接入的类型，固定为： `lucene_index_kafka`
- **`id:`** 指定创建的task的名称,若不设置则采用默认生成的task名称

- **`resource`**　关于任务需要的资源说明
- **`resource.availabilityGroup:`** 指定所属的可用性组ID
- **`resource.requiredCapacity:`** task将需要的工人槽数（worker slots）

- **`dataSchema:`** 关于接入数据的概要说明
- **`dataSchema.dataSource:`** 数据源的名称，类似关系数据库中的表名
- **`dataSchema.parser:`** 数据转换器
- **`dataSchema.metricsSpec:`** 所有的指标列和所使用的聚合函数,可以不设置

- **`dataSchema.granularitySpec:`** 数据粒度说明
- **`dataSchema.granularitySpec.type:`** 默认使用uniform
- **`dataSchema.granularitySpec.intervals:`** 数据时间戳范围,可以指定多个范围
- **`dataSchema.granularitySpec.segmentGranularity:`** 段粒度，根据每天的数据量进行设置。 小数据量建议`DAY`，大数据量（每天百亿）可以选择`HOUR`。可选项：`SECOND`、`MINUTE`、`FIVE_MINUTE`、`TEN_MINUTE`、`FIFTEEN_MINUTE`、`HOUR、SIX_HOUR`、`DAY`、`MONTH`、`YEAR`。

- **`dataSchema.granularitySpec.rollup:`** 是否对数据进行上卷，也就是以更大的粒度存储
- **`dataSchema.granularitySpec.queryGranularity:`**　查询粒度

- **`tuningConfig:`** 优化说明
- **`tuningConfig.maxRowsInMemory:`** 在存盘之前内存中最大的存储行数，指的是聚合后的行数
- **`tuningConfig.maxRowsPerSegment:`** 每个segment最大的存储行数
- **`tuningConfig.intermediatePersistPeriod:`** 多长时间数据临时存盘一次
- **`tuningConfig.basePersistDirectory:`** 任务中接收到的数据临时存放路径，需要注意用户的读写权限。
> 注意：`basePersistDirectory`需要保证有相应的权限   
- **`tuningConfig.basePersistDirectoryStr:`** 临时存盘目录
- **`tuningConfig.maxPendingPersists:`** 最大同时存盘请求数，达到上限，摄取将会暂停
- **`tuningConfig.indexSpec:`**　索引说明
- **`tuningConfig.indexSpec.bitmap:`**　`bitmap`说明
- **`tuningConfig.indexSpec.bitmap.type:`**　采用的`bitmap`索引类型，可选项为：`concise`、`roaring`，默认为`concise`
- **`tuningConfig.indexSpec.dimensionCompression:`**　指定维度列的压缩策略，若不设置则默认为`LZ4`策略
- **`tuningConfig.indexSpec.metricCompression:`**　指定度量列的压缩策略，若不设置则默认为`LZ4`策略
- **`tuningConfig.buildV9Directly:`**　是否直接构建v9版本的索引
- **`tuningConfig.reportParseExceptions:`** 是否汇报数据解析错误
- **`tuningConfig.handoffConditionTimeout:`** 切换环境多长时间超时
- **`tuningConfig.consumerThreadCount:`** 消费者线程数

- **`ioConfig:`** 数据的IO说明
- **`ioConfig.type:`** 数据IO说明的类型，固定为：`kafka`
- **`ioConfig.baseSequenceName:`** 
- **`ioConfig.startPartitions:`** 消息的起始分区说明
- **`ioConfig.startPartitions.topic:`** 起始分区的 `topic` 名称
- **`ioConfig.startPartitions.group:`** 起始分区的 `group` 名称
- **`ioConfig.startPartitions.partitionOffsetMap:`** 起始分区的 `offset` 设置

- **`ioConfig.endPartitions:`** 消息的结束分区说明
- **`ioConfig.endPartitions.topic:`** 结束分区的 `topic` 名称
- **`ioConfig.endPartitions.group:`** 结束分区的 `group` 名称
- **`ioConfig.endPartitions.partitionOffsetMap:`** 结束分区的 `offset` 设置

- **`ioConfig.consumerProperties:`** 消费者的配置
- **`ioConfig.useTransaction:`** 是否使用事务，默认为 `true`
- **`ioConfig.pauseAfterRead:`** 读取之后是否暂停,默认为 `false`
- **`ioConfig.minimumMessageTime:`** 

- **`writerConfig:`**　设置task的`LuceneWriterConfig`，也可以不设置
- **`writerConfig.maxBufferedDocs:`**　缓存的最大行数
- **`writerConfig.ramBufferSizeMB:`**　缓存使用的内存数
- **`writerConfig.indexRefreshIntervalSeconds:`**　写磁盘的时间间隔,只有写磁盘后实时数据才可以查询，默认6s
- **`writerConfig.isIndexMerge:`**　是否对索引文件进行压缩存储，默认压缩
- **`writerConfig.isCompound:`**　是否启动`compound`，默认为`false`
- **`writerConfig.maxMergeAtOnce:`**　达到多少记录时立即进行合并，默认不设置
- **`writerConfig.maxMergedSegmentMB:`**　数据达到多少MB时进行合并，默认不设置
- **`writerConfig.maxMergesThreads:`**　最大的合并线程数
- **`writerConfig.mergeSegmentsPerTire:`**　合并时的参数，默认不设置
- **`writerConfig.writeThreads:`**　写线程数
- **`writerConfig.limiterMBPerSec:`**　每秒数据量限制
- **`context:`**　任务上下文环境设置

## <a id="LuceneIndexRealtimeTask" href="LuceneIndexRealtimeTask"></a> LuceneIndexRealtimeTask
```
{
  "type":"lucene_index_realtime",
  "id": "id_string",
  "resource": {
    "availabilityGroup":"availabilityGroup_string",
    "requiredCapacity":10
  },
  "luceneConfig":{
    "maxBufferedDocs": Integer maxBufferedDocs,
    "ramBufferSizeMB": Double ramBufferSizeMB,
    "indexRefreshIntervalSeconds": Long indexRefreshIntervalSeconds,
    "isIndexMerge": Boolean isIndexMerge,
    "isCompound": Boolean isCompound,
    "maxMergeAtOnce": Integer maxMergeAtOnce,
    "maxMergedSegmentMB": Integer maxMergedSegmentMB,
    "maxMergesThreads": Integer maxMergesThreads,
    "mergeSegmentsPerTire": Double mergeSegmentsPerTire,
    "writeThreads": Integer writeThreads,
    "limiterMBPerSec": Double limiterMBPerSec
  },
  "spec": {
    "dataSchema": {
      "metricsSpec": [],
      "parser": {<parser>},   
      },
      "granularitySpec": {
        "intervals": ["2001-01-01/2020-01-01"],
        "segmentGranularity": "YEAR",
        "rollup": true | false,
        "queryGranularity"：　<queryGranularity>, 
        "type": "uniform"  
      },
      "dataSource": "com_HyoaKhQMl_project_r1U1FDLjg"
    },
    "ioConfig": {
      "firehose": {
        "type": "local",   
        "filter": "filter_string",   
        "parser":<parser>,
        "baseDir": "baseDir_string"  
      },  
      "plumber":PlumberSchool plumberSchool,
      "firehoseV2":FirehoseFactoryV2 firehoseFactoryV2,  
      "type": "realtime" 
    },  
    "tuningConfig": {
      "type": "realtime",
      "basePersistDirectory": "path_string",   
      "reportParseExceptions":true | false, 
      "rejectionPolicy": {
        "type": "type_string"
      },     
      "maxRowsInMemory":Integer maxRowsInMemory
      "intermediatePersistPeriod":　Period intermediatePersistPeriod
      "windowPeriod":Period windowPeriod
      "versioningPolicy":{
        "type":"type_string"
      },
      "maxPendingPersists":Integer maxPendingPersists
      "shardSpec":{
        "type":"type_string"
      },
      "indexSpec":{
          "bitmap":{
              "type":"type_string"
          },
          "dimensionCompression":"dimensionCompression_string",
          "metricCompression":"metricCompression_string"
      }
      "buildV9Directly":　true | false
      "persistThreadPriority": int persistThreadPriority
      "mergeThreadPriority":　int mergeThreadPriority
      "handoffConditionTimeout":　Long handoffConditionTimeout
      "maxWarmCount":　Integer maxWarmCount
      "consumerThreadCount":Integer consumerThreadCount
      "needLuceneMerge":　true | false
    }
  },
　"context": Map<String, Object> context
}
```
- **`type:`** 指定数据接入的类型
- **`id:`** 指定创建的task的名称,若不设置则采用默认生成的task名称

- **`resource`**　关于任务需要的资源说明
- **`resource.availabilityGroup:`** 指定所属的可用性组ID
- **`resource.requiredCapacity:`** task将需要的工人槽数（worker slots）

- **`luceneConfig:`** 设置task的`LuceneWriterConfig`，可以不设置
- **`luceneConfig.maxBufferedDocs:`**　缓存的最大行数
- **`luceneConfig.ramBufferSizeMB:`**　缓存使用的内存数
- **`luceneConfig.indexRefreshIntervalSeconds:`**　写磁盘的时间间隔,只有写磁盘后实时数据才可以查询，默认6s
- **`luceneConfig.isIndexMerge:`**　是否对索引文件进行压缩存储，默认压缩
- **`luceneConfig.isCompound:`**　是否启动`compound`，默认为`false`
- **`luceneConfig.maxMergeAtOnce:`**　达到多少记录时立即进行合并，默认不设置
- **`luceneConfig.maxMergedSegmentMB:`**　数据达到多少MB时进行合并，默认不设置
- **`luceneConfig.maxMergesThreads:`**　最大的合并线程数
- **`luceneConfig.mergeSegmentsPerTire:`**　合并时的参数，默认不设置
- **`luceneConfig.writeThreads:`**　写线程数
- **`luceneConfig.limiterMBPerSec:`**　每秒数据量限制

- **`spec:`** 一些参数说明
- **`spec.dataSchema:`** 关于接入数据的概要说明
- **`spec.dataSchema.datasource:`** 数据源的名称，类似关系数据库中的表名
- **`spec.dataSchema.metricsSpec:`** 所有的指标列和所使用的聚合函数,可以不设置
- **`spec.dataSchema.parser:`** 数据转换器

- **`spec.dataSchema.granularitySpec:`** 数据粒度说明
- **`spec.dataSchema.granularitySpec.intervals:`** 数据时间戳范围,可以指定多个范围
- **`spec.dataSchema.granularitySpec.segmentGranularity:`** 段粒度，根据每天的数据量进行设置。 小数据量建议`DAY`，大数据量（每天百亿）可以选择`HOUR`。可选项：`SECOND`、`MINUTE`、`FIVE_MINUTE`、`TEN_MINUTE`、`FIFTEEN_MINUTE`、`HOUR、SIX_HOUR`、`DAY`、`MONTH`、`YEAR`。

- **`spec.dataSchema.granularitySpec.rollup:`** 是否对数据进行上卷，也就是以更大的粒度存储
- **`spec.dataSchema.granularitySpec.queryGranularity:`**　查询粒度
- **`spec.dataSchema.granularitySpec.type:`** 默认使用`uniform`

- **`spec.ioConfig:`** 数据的IO说明
- **`spec.ioConfig.firehose:`** 数据源适配器
- **`spec.ioConfig.firehose.type:`** 数据源适配器的类型,一般用`local`
- **`spec.ioConfig.firehose.filter:`** 文件名匹配符，如`*.csv` 匹配所有的`csv`文件
- **`spec.ioConfig.firehose.parser:`** 数据转换器
- **`spec.ioConfig.firehose.baseDir:`** 数据文件所在目录
- **`spec.ioConfig.plumber:`** 设置数据管道，可选项有：`realtime`、`appenderator`、`flushing`，默认为`realtime`
- **`spec.ioConfig.firehoseV2:`** 第二个版本的firehose,`firehoseV2`和`firehose`不可以同时设置，只能二选一
- **`spec.ioConfig.type:`** 固定为`realtime`

- **`spec.tuningConfig:`** 优化说明
- **`spec.tuningConfig.type:`** 固定为`realtime`
- **`spec.tuningConfig.basePersistDirectory:`** 保存数据的临时目录

> 注意：`basePersistDirectory`需要保证有相应的权限

- **`spec.tuningConfig.reportParseExceptions:`**　是否汇报数据解析错误，默认为`false`
- **`spec.tuningConfig.rejectionPolicy:`**　 数据过滤策略，如果不过滤则不需要修改。可选项有：`serverTime`、`messageTime`、`none`,若不设置则默认为`serverTime`

- **`spec.tuningConfig.maxRowsInMemory:`**　在存盘之前内存中最大的存储行数，指的是聚合后的行数
- **`spec.tuningConfig.intermediatePersistPeriod:`**　多长时间数据临时存盘一次
- **`spec.tuningConfig.windowPeriod:`**　设置数据在实时节点停留的时间窗口
- **`spec.tuningConfig.versioningPolicy:`**　版本策略
- **`spec.tuningConfig.versioningPolicy.type:`**　设置具体类型版本策略，可选项有：`intervalStart`、`custom`，若不设置则默认为`intervalStart`
- **`spec.tuningConfig.maxPendingPersists:`**　最大同时存盘请求数，达到上限，摄取将会暂停
- **`spec.tuningConfig.shardSpec:`**　分片说明，目前只有`none`可选
- **`spec.tuningConfig.indexSpec:`**　索引说明
- **`spec.tuningConfig.indexSpec.bitmap:`**　`bitmap`说明
- **`spec.tuningConfig.indexSpec.bitmap.type:`**　采用的`bitmap`索引类型，可选项为：`concise`、`roaring`，默认为`concise`
- **`spec.tuningConfig.indexSpec.dimensionCompression:`**　指定维度列的压缩策略，若不设置则默认为`LZ4`策略
- **`spec.tuningConfig.indexSpec.metricCompression:`**　指定度量列的压缩策略，若不设置则默认为`LZ4`策略
- **`spec.tuningConfig.buildV9Directly:`**　是否直接构建v9版本的索引
- **`spec.tuningConfig.persistThreadPriority:`**　持久化线程的优先级
- **`spec.tuningConfig.mergeThreadPriority:`**　合并线程的优先级
- **`spec.tuningConfig.handoffConditionTimeout:`**　切换环境多长时间超时
- **`spec.tuningConfig.consumerThreadCount:`**　消费者线程数,默认为１
- **`spec.tuningConfig.needLuceneMerge:`**　是否需要合并，默认为`true`
- **`context:`**　任务上下文环境设置，可以不设置

## <a id="BatchKafkaIndexTask" href="BatchKafkaIndexTask"></a> BatchKafkaIndexTask
```
{
  "type":"batch_lucene_index_kafka",
  "id": String id,
  "resource": {
    "availabilityGroup":"availabilityGroup_string",
    "requiredCapacity":10
  },
  "dataSchema":{
    "dataSource":"dataSource_string",
    "parser":{<parser>},
    "metricsSpec":"aggregators",
    "granularitySpec":{
      "type":"uniform",
      "segmentGranularity": <segmentGranularity>,
      "queryGranularity": <queryGranularity>,
      "rollup": false,
      "intervals": [<interval>]
    }
  },
  "tuningConfig": {
    "maxRowsInMemory": Integer maxRowsInMemory,
    "maxRowsPerSegment": Integer maxRowsPerSegment,
    "intermediatePersistPeriod": Period intermediatePersistPeriod,
    "basePersistDirectory": File basePersistDirectory,
    "basePersistDirectoryStr": String basePersistDirectoryStr,
    "maxPendingPersists": Integer maxPendingPersists,
    "indexSpec":{
      "bitmap": {
        "type"： "concise"
      },  
      "dimensionCompression": "dimensionCompression_string",
      "metricCompression": "metricCompression_string"
    },
    "buildV9Directly": Boolean buildV9Directly,
    "reportParseExceptions": Boolean reportParseExceptions,
    "handoffConditionTimeout": Long handoffConditionTimeout,
    "maxWarmCount": Integer maxWarmCount,
    "consumerThreadCount": Integer consumerThreadCount
  },
  "ioConfig": {
    "baseSequenceName":  "baseSequenceName_string",
    "startPartitions": {
      "topic": "topic_string",
      "partitionOffsetMap": {}
    },
    "endPartitions": {
      "topic": "topic_string",
      "partitionOffsetMap": {}
    },
    "consumerProperties": {},
    "useTransaction": true | false,
    "pauseAfterRead": true | false,
    "minimumMessageTime": DateTime minimumMessageTime
  },
  "writerConfig": {
    "maxBufferedDocs": Integer maxBufferedDocs,
    "ramBufferSizeMB": Double ramBufferSizeMB,
    "indexRefreshIntervalSeconds": Long indexRefreshIntervalSeconds,
    "isIndexMerge": Boolean isIndexMerge,
    "isCompound": Boolean isCompound,
    "maxMergeAtOnce": Integer maxMergeAtOnce,
    "maxMergedSegmentMB": Integer maxMergedSegmentMB,
    "maxMergesThreads": Integer maxMergesThreads,
    "mergeSegmentsPerTire": Double mergeSegmentsPerTire,
    "writeThreads": Integer writeThreads,
    "limiterMBPerSec": Double limiterMBPerSec
  },
  "context": Map<String, Object> context,
  "ipLibSpec": Map<String, Object> ipLibSpec
}
```
- **`type:`** 指定数据接入的类型
- **`id:`** 指定创建的task的名称,若不设置则采用默认生成的task名称

- **`resource`**　关于任务需要的资源说明
- **`resource.availabilityGroup:`** 指定所属的可用性组ID
- **`resource.requiredCapacity:`** task将需要的工人槽数（worker slots）

- **`dataSchema`** 接入数据的概要说明
- **`dataSchema.dataSource`** 数据源名字
- **`dataSchema.parser`** 包含如何解析数据的相关内容
- **`dataSchema.metricsSpec`** 所有的指标列和所使用的聚合函数
- **`dataSchema.granularitySpec`** 数据的存储和查询粒度
- **`dataSchema.granularitySpec.type`** 默认使用uniform
- **`dataSchema.granularitySpec.segmentGranularity`**　段粒度，根据每天的数据量进行设置。 小数据量建议DAY，大数据量（每天百亿）可以选择HOUR。可选项：SECOND、MINUTE、FIVE_MINUTE、TEN_MINUTE、FIFTEEN_MINUTE、HOUR、SIX_HOUR、DAY、MONTH、YEAR。
- **`dataSchema.granularitySpec.queryGranularity`** 查询粒度
- **`dataSchema.granularitySpec.rollup`** 是否进行汇总统计
- **`dataSchema.granularitySpec.intervals`** 区间

- **`tuningConfig:`** 用于优化数据摄取的过程
- **`tuningConfig.type:`** 设置为`kafka`
- **`tuningConfig.maxRowsInMemory:`** 在存盘之前内存中最大的存储行数，指的是聚合后的行数
- **`tuningConfig.maxRowsPerSegment:`** 每个segment最大的存储行数
- **`tuningConfig.intermediatePersistPeriod:`** 多长时间数据临时存盘一次
- **`tuningConfig.basePersistDirectory:`** 任务中接收到的数据临时存放路径，需要注意用户的读写权限。
- **`tuningConfig.basePersistDirectoryStr:`** 临时存盘目录
- **`tuningConfig.maxPendingPersists:`** 最大同时存盘请求数，达到上限，摄取将会暂停

- **`tuningConfig.indexSpec.bitmap:`** 索引服务说明
- **`tuningConfig.indexSpec.bitmap.type:`** 使用哪种类型的位图索引，默认是concise类型，也可手动指定为roaring类型
- **`tuningConfig.indexSpec.dimensionCompression:`** 指定维度列的压缩格式，允许为null
- **`tuningConfig.indexSpec.metricCompression:`** 指定指标列的压缩格式，允许为null
- **`tuningConfig.buildV9Directly:`** 是否直接构建v9版本的索引
- **`tuningConfig.reportParseExceptions:`** 是否汇报数据解析错误
- **`tuningConfig.handoffConditionTimeout:`** 切换环境多长时间超时
- **`tuningConfig.consumerThreadCount:`** 消费者线程数

- **`ioConfig:`** 数据的IO说明
- **`ioConfig.baseSequenceName:`** 
- **`ioConfig.startPartitions:`** 消息的起始分区说明
- **`ioConfig.startPartitions.topic:`** 起始分区的 `topic` 名称
- **`ioConfig.startPartitions.group:`** 起始分区的 `group` 名称
- **`ioConfig.startPartitions.partitionOffsetMap:`** 起始分区的 `offset` 设置

- **`ioConfig.endPartitions:`** 消息的结束分区说明
- **`ioConfig.endPartitions.topic:`** 结束分区的 `topic` 名称
- **`ioConfig.endPartitions.group:`** 结束分区的 `group` 名称
- **`ioConfig.endPartitions.partitionOffsetMap:`** 结束分区的 `offset` 设置

- **`writerConfig:`** 设置`writerConfig`的参数，可以不设置
- **`writerConfig.maxBufferedDocs:`**　缓存的最大行数
- **`writerConfig.ramBufferSizeMB:`**　缓存使用的内存数
- **`writerConfig.indexRefreshIntervalSeconds:`**　写磁盘的时间间隔,只有写磁盘后实时数据才可以查询，默认6s
- **`writerConfig.isIndexMerge:`**　是否对索引文件进行压缩存储，默认压缩
- **`writerConfig.isCompound:`**　是否启动`compound`，默认为`false`
- **`writerConfig.maxMergeAtOnce:`**　达到多少记录时立即进行合并，默认不设置
- **`writerConfig.maxMergedSegmentMB:`**　数据达到多少MB时进行合并，默认不设置
- **`writerConfig.maxMergesThreads:`**　最大的合并线程数
- **`writerConfig.mergeSegmentsPerTire:`**　合并时的参数，默认不设置
- **`writerConfig.writeThreads:`**　写线程数
- **`writerConfig.limiterMBPerSec:`**　每秒数据量限制

- **`context:`**　任务上下文环境设置，可以不设置

## <a id="HadoopLuceneIndexTask" href="HadoopLuceneIndexTask"></a> HadoopLuceneIndexTask
HadoopLuceneIndexTask 用来从hadoop上面接入数据。
```
{
  "type":"lucene_index_hadoop",
  "id": "id_string",
  "spec": {
    "dataSchema": {},
    "ioConfig": {
      "inputSpec": Map<String, Object> pathSpec,
      "metadataUpdateSpec": {
        "type": "type_string",
        "connectURI": "connectURI_string",
        "user": "user_string",
        "password":{
          "type": "default",
          "password": "password_string"
        },
        "segmentTable": "segmentTable_string"
      },
      "segmentOutputPath": "segmentOutputPath_string"
    },
    "tuningConfig": {      
      "workingPath": "workingPath_string",
      "version": "version_string",
      "partitionsSpec": {
        "type": "type_string",
        "targetPartitionSize" : Long targetPartitionSize,
        "maxPartitionSize": Long maxPartitionSize,
        "assumeGrouped": true | false,
        "numShards": Integer numShards,
        "partitionDimensions":[<partitionDimensions_string>,<partitionDimensions_string>,...]
      },
      "shardSpecs": Map<DateTime, List<HadoopyShardSpec>> shardSpecs,
      "indexSpec":{
        "bitmap": {
          "type"： "concise"
        },  
        "dimensionCompression": "dimensionCompression_string",
        "metricCompression": "metricCompression_string"
      },
      "maxRowsInMemory": Integer maxRowsInMemory,
      "leaveIntermediate": true | false,  
      "cleanupOnFailure": true | false,  
      "overwriteFiles": true | false,  
      "ignoreInvalidRows": true | false,  
      "jobProperties": Map<String, String> jobProperties,
      "combineText": true | false,  
      "useCombiner": true | false,  
      "rowFlushBoundary": Integer maxRowsInMemoryCOMPAT,
      "buildV9Directly": true | false,
      "numBackgroundPersistThreads": Integer numBackgroundPersistThreads
    },
    "uniqueId": "uniqueId_string"
  },
  "hadoopCoordinates": "hadoopCoordinates_string",
  "hadoopDependencyCoordinates": ["hadoopDependencyCoordinates_string"],
  "classpathPrefix": "classpathPrefix_string",
  "context": Map<String, Object> context
}
```

- **`type:`** 指定数据接入的类型
- **`id:`** 指定创建的task的名称,若不设置则采用默认生成的task名称

- **`spec:`** 一些参数说明
- **`spec.dataSchema:`** 关于接入数据的概要说明

- **`spec.ioConfig:`** 数据的IO说明
- **`spec.ioConfig.inputSpec:`** 
- **`spec.ioConfig.metadataUpdateSpec:`** 
- **`spec.ioConfig.metadataUpdateSpec.type:`** 
- **`spec.ioConfig.metadataUpdateSpec.connectURI:`** 
- **`spec.ioConfig.metadataUpdateSpec.user:`**
- **`spec.ioConfig.metadataUpdateSpec.password:`**
- **`spec.ioConfig.metadataUpdateSpec.password.type:`**
- **`spec.ioConfig.metadataUpdateSpec.password.password:`**
- **`spec.ioConfig.metadataUpdateSpec.segmentTable:`**
- **`spec.ioConfig.segmentOutputPath:`**

- **`spec.tuningConfig:`** 优化说明
- **`spec.tuningConfig.workingPath:`** 路径
- **`spec.tuningConfig.version:`** 版本
- **`spec.tuningConfig.partitionsSpec:`**
- **`spec.tuningConfig.partitionsSpec.type:`**
- **`spec.tuningConfig.partitionsSpec.targetPartitionSize:`**
- **`spec.tuningConfig.partitionsSpec.maxPartitionSize:`** partition的容量
- **`spec.tuningConfig.partitionsSpec.assumeGrouped:`** 
- **`spec.tuningConfig.partitionsSpec.numShards:`** 分片数
- **`spec.tuningConfig.partitionsSpec.partitionDimensions:`** partition的维度
- **`spec.tuningConfig.shardSpecs:`** 分片说明，目前只有`none`可选
- **`spec.tuningConfig.indexSpec:`** 索引说明
- **`spec.tuningConfig.indexSpec.bitmap:`**　`bitmap`说明
- **`spec.tuningConfig.indexSpec.bitmap.type:`**　采用的`bitmap`索引类型，可选项为：`concise`、`roaring`，默认为`concise`
- **`spec.tuningConfig.indexSpec.dimensionCompression:`**　指定维度列的压缩策略，若不设置则默认为`LZ4`策略
- **`spec.tuningConfig.indexSpec.metricCompression:`**　指定度量列的压缩策略，若不设置则默认为`LZ4`策略
- **`spec.tuningConfig.maxRowsInMemory:`**　在存盘之前内存中最大的存储行数，指的是聚合后的行数
- **`spec.tuningConfig.leaveIntermediate:`**
- **`spec.tuningConfig.cleanupOnFailure:`** 失败是是否清除
- **`spec.tuningConfig.overwriteFiles:`** 是否覆盖文件
- **`spec.tuningConfig.ignoreInvalidRows:`** 是否无视无效行
- **`spec.tuningConfig.jobProperties:`**
- **`spec.tuningConfig.combineText:`**
- **`spec.tuningConfig.useCombiner:`**
- **`spec.tuningConfig.rowFlushBoundary:`** 触发flush的行数
- **`spec.tuningConfig.buildV9Directly:`**　是否直接构建v9版本的索引
- **`spec.tuningConfig.numBackgroundPersistThreads:`** 后台存在的线程数
- **`spec.uniqueId:`** 唯一id

- **`hadoopCoordinates:`** 
- **`hadoopDependencyCoordinates:`** 
- **`classpathPrefix:`** 路径前缀
- **`context:`** 　任务上下文环境设置，可以不设置


## <a id="LuceneMergeTask" href="LuceneMergeTask"></a> LuceneMergeTask
```
{
  "type":"lucene_merge",
  "id": "id_string",
  "dataSource": "dataSource_string",
  "interval": Interval interval,
  "mergeGranularity": Granularity mergeGranularity,
  "triggerMergeCount": int triggerMergeCount,
  "context": Map<String, Object> context
}
```

- **`type:`** 指定数据接入的类型,固定为`lucene_merge`
- **`id:`** 指定创建的task的名称,若不设置则采用默认生成的task名称
- **`dataSource:`** 指定**已存在于系统**的`datasource`名称
- **`interval:`** 指定要进行合并的区间
- **`mergeGranularity:`** 指定合并的粒度，可以不设置，可选项有：`SECOND`、`MINUTE`、`FIVE_MINUTE`、`TEN_MINUTE`、`FIFTEEN_MINUTE`、`HOUR`、`SIX_HOUR`、`DAY、WEEK`、`MONTH`、`YEAR`
- **`triggerMergeCount:`**　触发合并的条数，指定把多少个分片合并成一片，若不设置则是把该记录的所有分片合为一片
- **`context:`**　指定task的执行环境，可以不设置

## <a id="LuceneIndexTask" href="LuceneIndexTask"></a> LuceneIndexTask
```
{
    "type": "lucene_index",
    "id": "id_string",
    "worker":String worker,
    "resource":{
        ""availabilityGroup"":"availabilityGroup_string",
        "requiredCapacity":int requiredCapacity
    },
    "spec":{
        "dataSchema": {
            "dataSource": "dataSource_string"  
            "metricsSpec": "aggregators", 
            "parser": {<parser>},   
            "granularitySpec": {
                "intervals": [<interval>] , 
                "segmentGranularity": <segmentGranularity>, 
                "rollup": true | false,
                "queryGranularity"：　<queryGranularity>, 
                "type": "uniform"   
            }  
        },
        "ioConfig": {
            "type":"lucene_index",
            "firehose": {
                "type": "local",   
                "filter": "filter_string",   
                "parser":<parser>,
                "baseDir": "baseDir_string"  
            }
        },
        "tuningConfig":{
            "type": "lucene_index",
            "maxRowsPerSegment":Integer maxRowsPerSegment,
            "numShards":Integer numShards,
            "basePersistDirectory": "path_string", 
            "reportParseExceptions" true | false
        }  
    },
    "writerConfig":{
        "maxBufferedDocs": Integer maxBufferedDocs,
        "ramBufferSizeMB": Double ramBufferSizeMB,
        "indexRefreshIntervalSeconds": Long indexRefreshIntervalSeconds,
        "isIndexMerge": Boolean isIndexMerge,
        "isCompound": Boolean isCompound,
        "maxMergeAtOnce": Integer maxMergeAtOnce,
        "maxMergedSegmentMB": Integer maxMergedSegmentMB,
        "maxMergesThreads": Integer maxMergesThreads,
        "mergeSegmentsPerTire": Double mergeSegmentsPerTire,
        "writeThreads": Integer writeThreads,      
        "limiterMBPerSec": Double limiterMBPerSec
    },
    "context":Map<String, Object> context
}
```

- **`type:`** 指定数据接入的类型,固定为`lucene_index`
- **`id:`** 指定创建的task的名称,若不设置则采用默认生成的task名称
- **`worker:`** 指定具体的worker,一般的形式为`ip:port`，如`dev224.sugo.net:8091`

- **`resource`**　关于任务需要的资源说明
- **`resource.availabilityGroup:`** 指定所属的可用性组ID
- **`resource.requiredCapacity:`** task将需要的工人槽数（worker slots）

- **`spec:`** 一些参数说明
- **`spec.dataSchema:`** 关于接入数据的概要说明
- **`spec.dataSchema.datasource:`** 数据源的名称，类似关系数据库中的表名
- **`spec.dataSchema.metricsSpec:`** 所有的指标列和所使用的聚合函数,可以不设置
- **`spec.dataSchema.parser:`** 数据转换器

- **`spec.dataSchema.granularitySpec:`** 数据粒度说明
- **`spec.dataSchema.granularitySpec.intervals:`** 数据时间戳范围,可以指定多个范围
- **`spec.dataSchema.granularitySpec.segmentGranularity:`** 段粒度，根据每天的数据量进行设置。 小数据量建议`DAY`，大数据量（每天百亿）可以选择`HOUR`。可选项：`SECOND`、`MINUTE`、`FIVE_MINUTE`、`TEN_MINUTE`、`FIFTEEN_MINUTE`、`HOUR、SIX_HOUR`、`DAY`、`MONTH`、`YEAR`。

- **`spec.dataSchema.granularitySpec.rollup:`** 是否对数据进行上卷，也就是以更大的粒度存储
- **`spec.dataSchema.granularitySpec.queryGranularity:`**　查询粒度
- **`spec.dataSchema.granularitySpec.type:`** 默认使用`uniform`

- **`spec.ioConfig:`** 数据的IO说明
- **`spec.ioConfig.type:`** 固定为`lucene_index`
- **`spec.ioConfig.firehose:`** 数据源适配器
- **`spec.ioConfig.firehose.type:`** 数据源适配器的类型,一般用`local`
- **`spec.ioConfig.firehose.filter:`** 文件名匹配符，如`*.csv` 匹配所有的`csv`文件
- **`spec.ioConfig.firehose.parser:`** 数据转换器,与上面配置的 `spec.dataSchema.parser` 一样
- **`spec.ioConfig.firehose.baseDir:`** 数据文件所在目录

- **`spec.tuningConfig:`** 优化说明
- **`spec.tuningConfig.type:`** 固定为`lucene_index`
- **`spec.tuningConfig.maxRowsPerSegment:`** 每个`segment`最大的存储行数
- **`spec.tuningConfig.numShards:`** 每个`segment`的分片数
- **`spec.tuningConfig.basePersistDirectory:`** 保存数据的临时目录

> 注意：`basePersistDirectory`需要保证有相应的权限

- **`spec.tuningConfig.reportParseExceptions:`** 是否汇报数据解析错误，默认为`false`

- **`writerConfig:`** 设置`writerConfig`的参数，可以不设置
- **`writerConfig.maxBufferedDocs:`**　缓存的最大行数
- **`writerConfig.ramBufferSizeMB:`**　缓存使用的内存数
- **`writerConfig.indexRefreshIntervalSeconds:`**　写磁盘的时间间隔,只有写磁盘后实时数据才可以查询，默认6s
- **`writerConfig.isIndexMerge:`**　是否对索引文件进行压缩存储，默认压缩
- **`writerConfig.isCompound:`**　是否启动`compound`，默认为`false`
- **`writerConfig.maxMergeAtOnce:`**　达到多少记录时立即进行合并，默认不设置
- **`writerConfig.maxMergedSegmentMB:`**　数据达到多少MB时进行合并，默认不设置
- **`writerConfig.maxMergesThreads:`**　最大的合并线程数
- **`writerConfig.mergeSegmentsPerTire:`**　合并时的参数，默认不设置
- **`writerConfig.writeThreads:`**　写线程数
- **`writerConfig.limiterMBPerSec:`**　每秒数据量限制

- **`context:`**　任务上下文环境设置，可以不设置