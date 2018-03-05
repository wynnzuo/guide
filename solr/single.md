# Solr单机安装配置
内置Jetty运行Solr。

名词解释：
- ==**core**==
在Solr单机环境中，core本质上就是单个index。若需有多个index，那必须创建多个core。在SolrCloud环境中，单个index可以横跨多个Solr实例，这意味着单个index是由不同机器上的多个cores组成。
- **==collection==**
由core组成的逻辑index叫做collection，一个collection是跨越多个cores的index，这使index可扩展并冗余备份。
- ==**shard**==
在SolrCloud中可以有多个collections。Collections可被分片，每个分片可有多个副本（Replica），同一副本下的相同分片称为shards。每个shards下的有一个分片为leader，该leader通过选举策略产生。
- **==node==**
 SolrCloud中，node是运行Solr的Java虚拟机实例，也就是Server（例如Tomcat、Jetty）。

理解core和collection的区别非常重要。在传统的单node solr中，core和collection的概念等同，都代表一个逻辑index。在SolrCloud中，多个nodes下的cores形成一个collection。

### 环境准备
- 1台服务器
- jdk1.8

## solr安装
#### 下载
```shell
cd /app
wget -c http://mirror.bit.edu.cn/apache/lucene/solr/6.6.2/solr-6.6.2.tgz
```
#### 解压
```
tar zxf solr-6.6.2.tgz
```

#### 启动集群节点
```
bin/solr start
```
#### 创建Solr应用
```
# 知识库内容
bin/solr create -c appkmsContent
# 问答
bin/solr create -c appkmsQuestion
# 爆料
bin/solr create -c appkmsTipOff
```
目录生成在${solr_home}/server/solr