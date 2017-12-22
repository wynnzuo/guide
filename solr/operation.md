# Solr常用的角本命令

**启动集群节点**

```
cd /app/solr-6.6.2/
bin/solr start -cloud -p 8983 -s "/app/solr_home/node" -z "10.128.91.191:2181,10.128.91.192:2181,10.128.91.193:2181"
```

**停止集群节点**

```
cd /app/solr-6.6.2/
bin/solr stop -cloud -p 8983 -s "/app/solr_home/node" -z "10.128.91.191:2181,10.128.91.192:2181,10.128.91.193:2181"
# 停止全部
bin/solr stop -all
```

**查看状态**

```
bin/solr status -cloud -p 8983 -s "/app/solr_home/node" -z "10.128.91.191:2181,10.128.91.192:2181,10.128.91.193:2181"
```

**创建Collection**

```
bin/solr create_collection
          -c appkms-db -shards 2
          -replicationFactor 3
          -d  /app/solr_home/collection/appkms-db/conf -p 8983
```

-c 核心名称<br>
-shards 分片数量<br>
-replicationFactor 副本数量 （一般指有几台solr集群）

**修改同步配置文件**

```shell
cd /app/solr-6.6.2/server/scripts/cloud-scripts

./zkcli.sh -cmd upconfig -confdir /app/solr_home/collection/appkms-db/conf -confname appkms-db -z "10.128.91.191:2181,10.128.91.192:2181,10.128.91.193:2181"

./zkcli.sh -cmd linkconfig -collection appkms-db -confname appkms-db -z "10.128.91.191:2181,10.128.91.192:2181,10.128.91.193:2181"
```

# 常见优化点

## 提交方式

solr提供了两种提交方式：硬提交和软提交。硬提交(hard commit)指的是在客户端每提交一批数据都通知服务器把数据刷新到硬盘上，检索的时候数据可以立马可见；软提交(soft commit)是让服务器自己控制在一定的时间范围之内刷新数据。很明显，硬提交是非常损耗性能的操作，并且它还会阻塞其他数据的提交，所以我们选择软提交，具体配置方式如下所示：

```xml
<!--solrconfig.xml file -->
<autoCommit>
  <maxTime>${solr.autoCommit.maxTime:30000}</maxTime>
  <openSearcher>false</openSearcher>
</autoCommit>
<autoSoftCommit>
  <maxTime>${solr.autoSoftCommit.maxTime:60000}</maxTime>
</autoSoftCommit>
```

在这份配置文件中，我们指定了服务器每隔60秒对数据做一次软提交，另外推荐opensearch设置为false，否则每次提交都会开启一个opensearch，这个也会损耗性能。

## 关闭副本

solr一个collection会有多个分片（shard），多个shard分别位于不同的节点上，每个节点上可能会有shard的多个复制，其中一个为leader shard，在追求极限速度的情况下，可以将副本数设置为1，这样减去了副本间的数据同步等资源消耗。不过这样做带来的弊端就是数据容灾性降低，和检索性能急剧下降。
