# Solr集群安装配置

内置Jetty运行SolrCloud。

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

## 环境准备

- 3台服务器（10.128.91.191，10.128.91.192，10.128.91.193）
- zookeeper集群
- jdk1.8

  ### 集群架构

  ![集群架构](855612-20170206211846307-1231965985.png)

## zookeeper集群安装

solrcloud 配置集群由zookeeper管理，zookeeper相关，参见《[zookeeper集群安装]》。

## solr安装

### 下载

```shell
cd /app
wget -c http://mirror.bit.edu.cn/apache/lucene/solr/6.6.2/solr-6.6.2.tgz
```

### 解压

```
tar zxf solr-6.6.2.tgz
```

### 微调生产设置

#### 内存和 GC 设置。默认情况下，bin/solr 脚本将最大 Java 堆大小设置为 512M（-Xmx512m）

```
SOLR_JAVA_MEM="-Xms4g -Xmx4g"
```

#### 使用 SolrCloud 进行生产

要以 SolrCloud 模式运行 Solr，您需要在包含文件中设置 ZK_HOST 变量，以指向您的 ZooKeeper 集合。在生产环境中不支持运行嵌入式 ZooKeeper。例如，如果您在默认客户端端口 2181（zk1，zk2 和 zk3）上的以下三台主机上托管 ZooKeeper 集成，则可以设置：

```
ZK_HOST=zk1,zk2,zk3
```

当 ZK_HOST 变量被设置时，Solr 将以"cloud"模式启动。

##### ZooKeeper chroot

如果您使用的是其他系统共享的 ZooKeeper 实例，建议使用 ZooKeeper 的 chroot 支持来隔离 SolrCloud znode 树。例如，要确保 SolrCloud 创建的所有 znode都存储在 /solr 下面，您可以在 ZK_HOST 连接字符串的末尾放置 /solr，例如：

```
ZK_HOST=zk1,zk2,zk3/solr
```

首次使用 chroot 之前，您需要使用 Solr 控制脚本在 ZooKeeper 中创建根路径（znode）。我们可以使用 mkroot命令：

```
bin/solr zk mkroot /solr -z <ZK_node>:<ZK_PORT>
```

> 你还想用现有的 solr_home 引导 ZooKeeper，你可以改为使用 zkcli.sh 或zkcli.bat bootstrap命令，如果它不存在，也会创建 chroot 路径。有关更多信息，请参阅命令行实用程序。

#### Solr 主机名

使用 SOLR_HOST 包含文件中的变量来设置 Solr 服务器的主机名。

```
SOLR_HOST=solr1.example.com
```

建议设置 Solr 服务器的主机名，尤其是在 SolrCloud 模式下运行时，因为这决定了在向 ZooKeeper 注册时节点的地址。

#### 重写 solrconfig.xml 中的设置

Solr 允许使用 -Dproperty=value 语法在启动时传递的 Java 系统属性重写配置属性。例如，在 solrconfig.xml 中，默认的 "自动软提交" 设置被设置为：

```xml
<autoSoftCommit>
  <maxTime>${solr.autoSoftCommit.maxTime:-1}</maxTime>
</autoSoftCommit>
```

一般来说，无论何时在使用 ${solr.PROPERTY:DEFAULT_VALUE} 语法的 Solr 配置文件中看到一个属性，都可以使用 Java 系统属性重写它。例如，要将 soft-commits 的 maxTime 设置为10秒，则可以使用以下命令启动 Solr -Dsolr.autoSoftCommit.maxTime=10000，例如：

```
bin/solr start -Dsolr.autoSoftCommit.maxTime=10000
```

该 bin/solr 脚本只是在启动时将以 -D 开头的选项传递给 JVM。为了在生产环境中运行，我们建议在 include 文件中定义 SOLR_OPTS 的变量中设置这些属性。按照我们的 soft-commit 例子，在 /etc/default/solr.in.sh 中，你可以这样做：

```
SOLR_OPTS="$SOLR_OPTS -Dsolr.autoSoftCommit.maxTime=10000"
```

### 启动集群节点

```
bin/solr start -cloud -m 4g -p 8983 -s "/app/solr_home/node" -z "sa2-app2:2181,sa2-app3:2181,sa2-api1:2181"
```

-m 以定义的值启动 Solr ：JVM 的 min（-Xms）和 max（-Xmx）堆大小。

-cloud 以cloud方式启动

-p 指定端口

-s 指定根目录

-z 指定zookeeper（用ip:端口。集群:ip:端口,ip:端口...）<br>

> **==其它节点机器, 按此章节以上步骤顺序执行。==**

## solr配置

### 创建solr_clound_home

```
mkdir /app/solr_home/node
cp /app/solr-6.6.2/server/solr/solr.xml /app/solr_home/node
```

> 如果更改了端口，则需要更改<br>
> <int name="hostPort">${jetty.port:8983}</int>

#### jar包准备
```
cp ${solr.home}/contrib/extraction/lib/* ${solr.home}/server/solr-webapp/webapp/WEB-INF/lib
cp ${solr.home}/dist/solrj-lib/noggit-0.6.jar ${solr.home}/server/solr-webapp/webapp/WEB-INF/lib
cp ${solr.home}/server/lib/metrics*.jar ${solr.home}/server/solr-webapp/webapp/WEB-INF/lib
cp ik-analyzer-solr6-6.0.jar ${solr.home}/server/solr-webapp/webapp/WEB-INF/lib
cp ojdbc6-11.2.0.3.jar ${solr.home}/server/solr-webapp/webapp/WEB-INF/lib
```

> Solr6适配升级版中文分词器IK Analyzer,请参见<http://blog.csdn.net/jiangchao858/article/details/53153272><br>
> oracle jdbc驱动包ojdbc6-11.2.0.3.jar,请参见[Oracle JDBC Drivers]

### Collection操作

#### 创建Collection
solr索引集合由zookeeper管理，所以我们创建核心，需要将配置文件上传到zookeeper，然后创建核心。
创建core
- 核心名称：appkms-db，appkms-file
- 创建核心core（appkms-db,appkms-file）配置存放目录
  ```
  #collection的根目录/app/solr_home/collection
  mkdir -p /app/solr_home/collection/appkms-db
  cp -r /app/solr-6.6.2/example/example-DIH/solr/solr/conf /app/solr_home/collection/appkms-db
  cp /app/solr-6.6.2/example/example-DIH/solr/solr/core.properties /app/solr_home/collection/appkms-db
  ```
- 创建核心并上传配置文件到zookeeper
```
bin/solr create_collection \
      -c appkms-db -shards 2 \
      -replicationFactor 3 \
      -d  /app/solr_home/collection/appkms-db/conf -p 8983

bin/solr create_collection \
      -c appkms-file -shards 2 \
      -replicationFactor 3 \
      -d /app/solr_home/collection/appkms-file/conf -p 8983

```
-c 核心名称<br/>
-shards 分片数量<br/>
-replicationFactor 副本数量 （一般指有几台solr集群）
> 执行命令前，需要先配置好collection配置文件，配置详情见章节。
#### 核心配置文件修改
collection的根目录：/app/solr_home/collection
```
[was@sa2-test7 collection]$ cd /app/solr_home/collection/
drwxrwxr-x 3 was was 39 Oct 27 14:39 appkms-db
drwxrwxr-x 3 was was 39 Oct 27 17:14 appkms-file
```
##### 核心appkmsContent配置修改
核心appkms-db配置文件所在路径：<br/>
/app/solr_home/collection/appkms-db/conf
- 编码 solrconfig.xml
- 创建db-data-config.xml
- 编辑 managed-schema

配置参见svn项目中.

ExtractingRequestHandler底层实际是使用apache Tika进行文件内容抽取的

**配置解释**
- <requestHandler name="/update/extract" startup="lazy"  class="solr.extraction.ExtractingRequestHandler" >
其中name=update/extract为request的请求路径。
- fmap.xxx 为从文件中抽取的内容，定义这些内容如何存储。如在这里：
```
<str name="fmap.content">content</str>  <!--文件内容-->
<str name="fmap.Content-Type">Content-Type</str> <!--文件类型-->
```
- **==uprefix==** 这个配置用于将文件中其它不需要的内容统一加上指定前缀，如这里加上了ignored_。在managed-schema中有该字段与类型配置：
```
<dynamicField name="ignored_*" type="ignored" multiValued="true"/>
<fieldType name="ignored" stored="false" indexed="false" multiValued="true" class="solr.StrField" />
```
这是个动态字段，即所有以ignored_开头的字段都按ignored这个type处理。在这达到的忽略这些数据的目的。

#### 删除Collection
- 通过控制台删除collection
- 通过访问api形式删除
```
#例子
curl "http://10.128.91.191:8983/solr/admin/collections?action=DELETE&name=appkms-file"
```
> 删除collections不会删除zookeeper中的配置信息

#### 修改上传managed-schema文件
- 上传managed-schema文件与新建collection上传配置文件到zookeeper相似，替换对应配置中的managed-schema文件
```shell
cd /app/solr-6.6.2/server/scripts/cloud-scripts
##./zkcli.sh -zkhost 10.128.91.191:2181 -cmd putfile /configs/appkms-file/managed-schema ~/managed-schema

./zkcli.sh -cmd upconfig -confdir /app/solr_home/collection/appkms-db/conf -confname appkms-db -z "10.128.91.191:2181,10.128.91.192:2181,10.128.91.193:2181"

./zkcli.sh -cmd linkconfig -collection appkms-db -confname appkms-db -z "10.128.91.191:2181,10.128.91.192:2181,10.128.91.193:2181"
```
-cmd putfile:修改配置
/configs/appkms-file/managed-schema为zookeeper节点位置
- 重新加载collection
```
curl "http://10.128.91.191:8983/solr/admin/collections?action=RELOAD&name=appkms-file"
```
 [zookeeper集群安装]: ../zookeeper.md

 [Oracle JDBC Drivers]: http://www.oracle.com/technetwork/database/features/jdbc/index-091264.html

#### 定时自动增量导入配置
##### 下载Jar

 下载apache-solr-dataimportscheduler-1.1.jar包

```
cp apache-solr-dataimportscheduler-1.1.jar ${solr.home}/server/solr-webapp/webapp/WEB-INF/lib
```
##### 修改web.xml

修改${solr.home}/server/solr-webapp/webapp/WEB-INF/web.xml,添加如下配置：

```xml
<listener>
    <listener-class>org.apache.solr.handler.dataimport.scheduler.ApplicationListener</listener-class>
</listener>
```

##### 修改时区为东8区
修改${solr.home}/bin/solr.in.sh，添加`﻿SOLR_TIMEZONE="UTC+8"`

##### dataimport.properties

在solr_home下(`启动命令的 -s参数`，

eg: bin/solr start -cloud -m 4g -p 8983 -s `"/app/solr_home/node" `),创建`conf`目录，
并添加文件`dataimport.properties'，文件具体内容如下：

```
#################################################
#                                               #
#       dataimport scheduler properties         #
#                                               #
#################################################

#  to sync or not to sync
#  1 - active; anything else - inactive
syncEnabled=1

#  which cores to schedule
#  in a multi-core environment you can decide which cores you want syncronized
#  leave empty or comment it out if using single-core deployment
# syncCores=game,resource #因为我的是single-core，所以注释掉了，默认就是collection1
syncCores=addressUser
#,appkmsContent,appkmsQuestion,appkmsTipOff
#  solr server name or IP address
#  [defaults to localhost if empty]
server=localhost

#  solr server port
#  [defaults to 80 if empty]
port=8983
#  application name/context
#  [defaults to current ServletContextListener's context (app) name]
webapp=solr
#  URL params [mandatory]
#  remainder of URL
#  增量更新的请求参数
params=/dataimport?command=delta-import&clean=true&commit=true
#  schedule interval
#  number of minutes between two runs
#  [defaults to 30 if empty]
#  这里配置的是2min一次
interval=2
#  重做索引的时间间隔，单位分钟，默认7200，即5天;
#  为空,为0,或者注释掉:表示永不重做索引
reBuildIndexInterval=7200
#  重做索引的参数
reBuildIndexParams=/dataimport?command=full-import&clean=true&commit=true
#  重做索引时间间隔的计时开始时间，第一次真正执行的时间=reBuildIndexBeginTime+reBuildIndexInterval*60*1000；
#  两种格式：2012-04-11 03:10:00 或者  03:10:00，后一种会自动补全日期部分为服务启动时的日期
reBuildIndexBeginTime=03:10:00

```

##### 在Collection的conf目录下,添加文件`dataimport.properties`

```
#Fri Nov 10 10:10:24 UTC 2017
item.last_index_time=2018-06-10 10\:10\:23
last_index_time=2018-06-10 10\:10\:23
stackoverflow.last_index_time=2018-06-10 10\:10\:23
```

