# 需求概述
MySQL 中某张表或者某些表插入数据时，需要发送消息到消息队列中，但是又不希望把这个插入操作耦合到业务逻辑中。即使耦合到业务中，写入`MySQL + Kafka`不进行一些额外处理也无法保证原子性。基于此，考虑使用 Flink CDC 捕获 MySQL 的数据变化，将消息发送到消息队列。

## example
### 版本信息
| MySQL | Flink | Flink CDC | Kafka |
| - | - | - | - |
|8.0|1.20.2|3.3.0| 3.4 |

### MySQL

### Kafka

### 基本流程
1. 部署基础服务 MySQL、Kafka；
2. 启动 Flink 集群；
3. 编写 Flink CDC 作业 yaml 配置`mysql-to-kafka.yaml`；
4. 将一些依赖包复制到 flink cdc lib 目录下；
```bash
# 仅列举手动导入的包
$ tree flink-cdc-3.3.0/lib
flink-cdc-3.3.0/lib
├── flink-cdc-pipeline-connector-kafka-3.3.0.jar
├── flink-cdc-pipeline-connector-mysql-3.3.0.jar
├── flink-sql-connector-mysql-cdc-3.3.0.jar
└── mysql-connector-java-8.0.27.jar
```

5. 提交 flink cdc 作业`bin/flink-cdc.sh mysql-to-kafka.yaml --jar lib/mysql-connector-java-8.0.27.jar`；

### 测试（todo）

### 结论

### 原理

# 参考链接
* [MySQL Connector ](https://nightlies.apache.org/flink/flink-cdc-docs-master/docs/connectors/pipeline-connectors/mysql/)
* [mysql to kafka 官方示例](https://nightlies.apache.org/flink/flink-cdc-docs-master/docs/connectors/pipeline-connectors/kafka/)
* [quick-start](https://nightlies.apache.org/flink/flink-cdc-docs-release-3.3/docs/deployment/standalone/)
