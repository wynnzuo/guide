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

### 环境准备
- 3台服务器（10.128.91.191，10.128.91.192，10.128.91.193）
- zookeeper集群
- jdk1.8
### 集群架构
![image](http://images2015.cnblogs.com/blog/855612/201702/855612-20170206211846307-1231965985.png)

### zookeeper集群安装
solrcloud 配置集群由zookeeper管理，zookeeper相关，参见《[zookeeper集群安装]》。

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
bin/solr start -cloud -p 8983 -s "/app/solr_home/node" -z "172.168.1.101:2181,172.168.1.102:2181,172.168.1.103:2181"
```


-cloud 以cloud方式启动

-p 指定端口

-s 指定根目录

-z 指定zookeeper（用ip:端口。集群:ip:端口,ip:端口...）
<br/>
> **==其它节点机器, 按此章节以上步骤顺序执行。==**

## solr配置
####  创建solr_clound_home
```
mkdir /app/solr_home/node
cp /app/solr-6.6.2/server/solr/solr.xml /app/solr_home/node
```
> 如果更改了端口，则需要更改<br/>
> <int name="hostPort">${jetty.port:8983}</int>
#### jar包准备
```
cp ${solr.home}/contrib/extraction/lib/* ${solr.home}/server/solr-webapp/webapp/WEB-INF/lib
cp ${solr.home}/dist/solrj-lib/noggit-0.6.jar ${solr.home}/server/solr-webapp/webapp/WEB-INF/lib
cp ${solr.home}/server/lib/metrics*.jar ${solr.home}/server/solr-webapp/webapp/WEB-INF/lib
cp ik-analyzer-solr6-6.0.jar ${solr.home}/server/solr-webapp/webapp/WEB-INF/lib
cp ojdbc6-11.2.0.3.jar ${solr.home}/server/solr-webapp/webapp/WEB-INF/lib
```
> Solr6适配升级版中文分词器IK Analyzer,请参见http://blog.csdn.net/jiangchao858/article/details/53153272<br/>
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
 bin/solr create_collection 
           -c appkms-db -shards 2 
           -replicationFactor 3 
           -d  /app/solr_home/collection/appkms-db/conf -p 8983
           
bin/solr create_collection 
           -c appkms-file -shards 2 
           -replicationFactor 3 
           -d  /app/solr_home/collection/appkms-file/conf -p 8983
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
##### 核心appkms-db配置修改
核心appkms-db配置文件所在路径：<br/>
/app/solr_home/collection/appkms-db/conf
- 编码 solrconfig.xml 添加如下配置：
```
  <lib dir="${solr.install.dir:../../../..}/dist/" regex="solr-dataimporthandler-.*\.jar" />

  <requestHandler name="/dataimport" class="org.apache.solr.handler.dataimport.DataImportHandler">
     <lst name="defaults">
        <str name="config">db-data-config.xml</str>
     </lst>
  </requestHandler>

  <requestHandler name="/update/extract" 
                  startup="lazy"
                  class="solr.extraction.ExtractingRequestHandler" >
    <lst name="defaults">
      <str name="lowernames">true</str>
      <str name="uprefix">ignored_</str>

      <!-- capture link hrefs but ignore div attributes -->
      <str name="captureAttr">true</str>
      <str name="fmap.a">links</str>
      <str name="fmap.div">ignored_</str>
    </lst>
	<lst name="date.formats">
      <str>yyyy-MM-dd HH:mm:ss</str>
    </lst>
  </requestHandler>
```
- 创建db-data-config.xml
文件内容如下：
```xml
<dataConfig>
    <dataSource driver="oracle.jdbc.driver.OracleDriver" url="jdbc:oracle:thin:@10.128.49.65:59926/saleshdb"
                user="appkms" password="appkms" type="JdbcDataSource" />
    <document>
        <entity name="item" query="select id,tc.name,create_user,description,AREA,modify_date from t_content"
                deltaQuery="select id ,tc.name,create_user,description,AREA,modify_date from t_content where modify_date > '${dataimporter.last_index_time}'">
            <field column="ID" name="id" />
            <field column="NAME" name="name" />
            <field column="CREATE_USER" name="createUser" />
            <field column="DESCRIPTION" name="descs" />
            <field column="AREA" name="area" />            
            <field column="MODIFY_DATE" name="last_modified" />
        </entity>
    </document>
</dataConfig>
```
- 编辑 managed-schema 完整配置如下：
```
<?xml version="1.0" encoding="UTF-8" ?>
<schema name="appkms-db" version="1.6">
   <!-- Only remove the "id" field if you have a very good reason to. While not strictly
     required, it is highly recommended. A <uniqueKey> is present in almost all Solr 
     installations. See the <uniqueKey> declaration below where <uniqueKey> is set to "id".
   -->   
   <field name="id" type="string" indexed="true" stored="true" required="true" multiValued="false" /> 
   <field name="name" type="text_ik" indexed="true" stored="true"/>
   <field name="createUser" type="text_ik" indexed="true" stored="true" omitNorms="true"/>
   <field name="area" type="text_ik" indexed="true" stored="true" />
   <field name="descs" type="text_ik" indexed="true" stored="true" termVectors="true" termPositions="true" termOffsets="true" />

   <!-- Common metadata fields, named specifically to match up with
     SolrCell metadata when parsing rich documents such as Word, PDF.
     Some fields are multiValued only because Tika currently may return
     multiple values for them. Some metadata is parsed from the documents,
     but there are some which come from the client context:
       "content_type": From the HTTP headers of incoming stream
       "resourcename": From SolrCell request param resource.name
   -->
   <field name="title" type="text_ik" indexed="true" stored="true" multiValued="true"/>
   <field name="subject" type="text_ik" indexed="true" stored="true"/>
   <field name="description" type="text_ik" indexed="true" stored="true"/>
   <field name="comments" type="text_ik" indexed="true" stored="true"/>
   <field name="author" type="text_ik" indexed="true" stored="true"/>
   <field name="keywords" type="text_ik" indexed="true" stored="true"/>
   <field name="category" type="text_ik" indexed="true" stored="true"/>
   <field name="resourcename" type="text_ik" indexed="true" stored="true"/>
   <field name="url" type="text_ik" indexed="true" stored="true"/>
   <field name="content_type" type="string" indexed="true" stored="true" multiValued="true"/>
   <field name="last_modified" type="date" indexed="true" stored="true"/>
   <field name="links" type="string" indexed="true" stored="true" multiValued="true"/>

   <!-- Main body of document extracted by SolrCell.
        NOTE: This field is not indexed by default, since it is also copied to "text"
        using copyField below. This is to save space. Use this field for returning and
        highlighting document content. Use the "text" field to search the content. -->
   <field name="content" type="text_ik" indexed="false" stored="true" multiValued="true"/>
   

   <!-- catchall field, containing all other searchable text fields (implemented
        via copyField further on in this schema  -->
   <field name="text" type="text_ik" indexed="true" stored="false" multiValued="true"/>
   
   ...
<!-- Field to use to determine and enforce document uniqueness. 
      Unless this field is marked with required="false", it will be a required field
   -->
 <uniqueKey>id</uniqueKey>

  <!-- copyField commands copy one field to another at the time a document
        is added to the index.  It's used either to index the same field differently,
        or to add multiple fields to the same field for easier/faster searching.  -->
 
   <copyField source="name" dest="text"/>
   <copyField source="createUser" dest="text"/>
   <copyField source="descs" dest="text"/>
   <copyField source="area" dest="text"/>

   <!-- Copy the price into a currency enabled field (default USD) 
   <copyField source="price" dest="price_c"/>-->

   <!-- Text fields from SolrCell to search by default in our catch-all field -->
   <copyField source="title" dest="text"/>
   <copyField source="author" dest="text"/>
   <copyField source="description" dest="text"/>
   <copyField source="keywords" dest="text"/>
   <copyField source="content" dest="text"/>
   <copyField source="content_type" dest="text"/>
   <copyField source="resourcename" dest="text"/>
   <copyField source="url" dest="text"/>

   <!-- Create a string version of author for faceting -->
   <copyField source="author" dest="author_s"/>
   
   ...
   <!-- IK中文分词 -->
   <fieldType name="text_ik" class="solr.TextField">
      <analyzer type="index" isMaxWordLength="false" class="org.wltea.analyzer.lucene.IKAnalyzer"/>
      <analyzer type="query" isMaxWordLength="true" class="org.wltea.analyzer.lucene.IKAnalyzer"/>
    </fieldType>
</schema>
```
##### 核心appkms-file配置修改
核心appkms-file配置文件所在路径：<br/>
/app/solr_home/collection/appkms-file/conf
###### 编码 solrconfig.xml 添加如下配置：
```
  <lib dir="${solr.install.dir:../../../..}/dist/" regex="solr-dataimporthandler-.*\.jar" />
  <requestHandler name="/update/extract" startup="lazy"  class="solr.extraction.ExtractingRequestHandler" >
    <lst name="defaults">
      <str name="fmap.content">content</str>
      <str name="fmap.Content-Type">Content-Type</str>
      <str name="uprefix">ignored_</str>

      <!-- capture link hrefs but ignore div attributes -->
      <str name="captureAttr">true</str>
      <str name="fmap.a">links</str>
      <str name="fmap.div">ignored_</str>
    </lst>
	<lst name="date.formats">
      <str>yyyy-MM-dd HH:mm:ss</str>
    </lst>
  </requestHandler>
```
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

##### 编辑 managed-schema 完整配置如下：
```
<?xml version="1.0" encoding="UTF-8" ?>
<schema name="appkms-file" version="1.6">
   <!-- Only remove the "id" field if you have a very good reason to. While not strictly
     required, it is highly recommended. A <uniqueKey> is present in almost all Solr 
     installations. See the <uniqueKey> declaration below where <uniqueKey> is set to "id".
   -->   
   <field name="id" type="string" indexed="true" stored="true" required="true" multiValued="false" /> 
   <field name="cid" type="int" indexed="true" stored="true" />

   <!-- Common metadata fields, named specifically to match up with
     SolrCell metadata when parsing rich documents such as Word, PDF.
     Some fields are multiValued only because Tika currently may return
     multiple values for them. Some metadata is parsed from the documents,
     but there are some which come from the client context:
       "content_type": From the HTTP headers of incoming stream
       "resourcename": From SolrCell request param resource.name
   -->
   <field name="title" type="text_ik" indexed="true" stored="true" multiValued="true"/>
   <field name="subject" type="text_ik" indexed="true" stored="true"/>
   <field name="description" type="text_ik" indexed="true" stored="true"/>
   <field name="comments" type="text_ik" indexed="true" stored="true"/>
   <field name="author" type="text_ik" indexed="true" stored="true"/>
   <field name="keywords" type="text_ik" indexed="true" stored="true"/>
   <field name="category" type="text_ik" indexed="true" stored="true"/>
   <field name="resourcename" type="text_ik" indexed="true" stored="true"/>
   <field name="url" type="text_ik" indexed="true" stored="true"/>
   <field name="content_type" type="string" indexed="true" stored="true" multiValued="true"/>
   <field name="last_modified" type="date" indexed="true" stored="true"/>
   <field name="links" type="string" indexed="true" stored="true" multiValued="true"/>

   <!-- Main body of document extracted by SolrCell.
        NOTE: This field is not indexed by default, since it is also copied to "text"
        using copyField below. This is to save space. Use this field for returning and
        highlighting document content. Use the "text" field to search the content. -->
   <field name="content" type="text_ik" indexed="false" stored="true" multiValued="true"/>
   

   <!-- catchall field, containing all other searchable text fields (implemented
        via copyField further on in this schema  -->
   <field name="text" type="text_ik" indexed="true" stored="false" multiValued="true"/>
   
   ...
<!-- Field to use to determine and enforce document uniqueness. 
      Unless this field is marked with required="false", it will be a required field
   -->
 <uniqueKey>id</uniqueKey>

  <!-- copyField commands copy one field to another at the time a document
        is added to the index.  It's used either to index the same field differently,
        or to add multiple fields to the same field for easier/faster searching.  -->

   <!-- Text fields from SolrCell to search by default in our catch-all field -->
   <copyField source="title" dest="text"/>
   <copyField source="author" dest="text"/>
   <copyField source="description" dest="text"/>
   <copyField source="keywords" dest="text"/>
   <copyField source="content" dest="text"/>
   <copyField source="content_type" dest="text"/>
   <copyField source="resourcename" dest="text"/>
   <copyField source="url" dest="text"/>

   <!-- Create a string version of author for faceting -->
   <copyField source="author" dest="author_s"/>
   
   ...
   <!-- IK中文分词 -->
   <fieldType name="text_ik" class="solr.TextField">
      <analyzer type="index" isMaxWordLength="false" class="org.wltea.analyzer.lucene.IKAnalyzer"/>
      <analyzer type="query" isMaxWordLength="true" class="org.wltea.analyzer.lucene.IKAnalyzer"/>
    </fieldType>
</schema>
```
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

