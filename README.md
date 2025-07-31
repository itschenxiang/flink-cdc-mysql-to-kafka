# 需求概述
MySQL 中某张表或者某些表插入数据时，需要发送消息到消息队列中，但是又不希望把这个插入操作耦合到业务逻辑中。即使耦合到业务中，写入`MySQL + Kafka`不进行一些额外处理也无法保证原子性。基于此，考虑使用 Flink CDC 捕获 MySQL 的数据变化，将消息发送到消息队列。

## Example
### 版本信息
| MySQL | Flink | Flink CDC | Kafka |
| - | - | - | - |
|8.0|[1.20.2](https://dlcdn.apache.org/flink/flink-1.20.2/flink-1.20.2-bin-scala_2.12.tgz)|[3.3.0](https://github.com/apache/flink-cdc/releases/tag/release-3.3.0)| 3.4 |

### MySQL

### Kafka

### 基本流程
1. 部署基础服务 MySQL、Kafka；
```bash
docker compose up -d
```

数据库初始化：
```sql
-- 创建数据库 test_db
CREATE DATABASE IF NOT EXISTS test_db;

-- 使用数据库 test_db
USE test_db;

-- 创建学生表 students
CREATE TABLE IF NOT EXISTS students (
    id INT AUTO_INCREMENT PRIMARY KEY,
    name VARCHAR(50) NOT NULL,         
    age INT,                           
    class VARCHAR(50),                 
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);

CREATE TABLE IF NOT EXISTS teachers (
    id INT AUTO_INCREMENT PRIMARY KEY,
    name VARCHAR(50) NOT NULL,         
    age INT,                           
    class VARCHAR(50),                 
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);

-- 插入测试数据
INSERT INTO students (name, age, class) VALUES
('张三', 18, '高一1班'),
('李四', 17, '高一2班'),
('王五', 16, '高一3班'),
('赵六', 19, '高一1班'),
('孙七', 18, '高一2班');

INSERT INTO teachers (name, age, class) VALUES
('张三', 18, '高一1班'),
('李四', 17, '高一2班'),
('王五', 16, '高一3班'),
('赵六', 19, '高一1班'),
('孙七', 18, '高一2班');

```

预创建主题：
```bash
# 随机进入一个本地 kafka 集群的一个容器
cd /opt/bitnami/kafka
bin/kafka-topics.sh --create --topic quickstart-events --bootstrap-server kafka1:19092 --partitions 4 --replication-factor 3

```

2. 启动 Flink 集群；
```bash
tar -xzf flink-1.20.2-bin-scala_2.12.tgz
cd flink-1.20.2
export FLINK_HOME=$(pwd)
nohup bin/start-cluster.sh &

# check start success
ps aux | grep flink
```

3. 将一些依赖包复制到 flink cdc lib 目录下；
> * [flink-cdc 相关 connector jar 下载地址](https://github.com/apache/flink-cdc/releases)
> * [mysql-connector-java-8.0.27.jar](https://repo1.maven.org/maven2/mysql/mysql-connector-java/8.0.27/mysql-connector-java-8.0.27.jar)

```bash
tar -xzf flink-cdc-3.3.0-bin.tar.gz

# 仅列举手动导入的包
tree lib 
lib
├── flink-cdc-pipeline-connector-kafka-3.3.0.jar
├── flink-cdc-pipeline-connector-mysql-3.3.0.jar
├── flink-sql-connector-mysql-cdc-3.3.0.jar
└── mysql-connector-java-8.0.27.jar
```

4. 编写 Flink CDC 作业 yaml 配置`mysql-to-kafka.yaml`；
```yaml
source:
   type: mysql
   name: MySQL Source
   hostname: 127.0.0.1
   port: 33060
   username: root
   password: 123456
   tables: test_db.\.*
   server-id: 5401-5404

sink:
  type: kafka
  name: Kafka Sink
  topic: test_db_topic
  properties.bootstrap.servers: PLAINTEXT://localhost:19094

pipeline:
  name: MySQL to Kafka Pipeline
  parallelism: 1
```

5. 提交 flink cdc 作业；
```bash
bin/flink-cdc.sh mysql-to-kafka.yaml --jar lib/mysql-connector-java-8.0.27.jar
```
### 测试验证⌛️
```bash
# 随机进入一个本地 kafka 集群的一个容器
cd /opt/bitnami/kafka
bin/kafka-console-consumer.sh --topic test_db_topic --from-beginning --bootstrap-server kafka1:19092

```

## 原理⌛️

### 生产环境如何部署及注意事项⌛️

# 参考链接
* [MySQL Connector ](https://nightlies.apache.org/flink/flink-cdc-docs-master/docs/connectors/pipeline-connectors/mysql/)
* [mysql to kafka 官方示例](https://nightlies.apache.org/flink/flink-cdc-docs-master/docs/connectors/pipeline-connectors/kafka/)
* [quick-start](https://nightlies.apache.org/flink/flink-cdc-docs-release-3.3/docs/deployment/standalone/)
