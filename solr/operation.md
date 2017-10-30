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
 -c 核心名称<br/>
-shards 分片数量<br/>
-replicationFactor 副本数量 （一般指有几台solr集群）


**修改同步配置文件**

```shell
cd /app/solr-6.6.2/server/scripts/cloud-scripts

./zkcli.sh -cmd upconfig -confdir /app/solr_home/collection/appkms-db/conf -confname appkms-db -z "10.128.91.191:2181,10.128.91.192:2181,10.128.91.193:2181"

./zkcli.sh -cmd linkconfig -collection appkms-db -confname appkms-db -z "10.128.91.191:2181,10.128.91.192:2181,10.128.91.193:2181"
```
