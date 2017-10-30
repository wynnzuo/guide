开发人员可以通过SolrJ和Solr
# Client API操作
SolrJ是Java程序操作Solr的API。它隐藏了许多和Solr交互的细节，这使得应用程序可以简单的，方便的，使用Solr的一些高级功能。

#### 构建和运行一个SolrJ 应用程序

##### 添加maven依赖
maven dependency in pom.xml
```xml
<dependency>
    <groupId>org.apache.solr</groupId>
    <artifactId>solr-solrj</artifactId>
    <version>6.6.2</version>
</dependency>
```
#### 构建SolrClient
SolrClient是一个抽象类，所以连接到远程Solr服务实例，你会创建一个HttpSolrClient或者 CloudSolrClient实例。 它们都是通过HTTP和Solr通信。HttpSolrClient是通过配置一个Solr URL,而CloudSolrClient是通过zhHost字符串和SolrCloud cluster通信

##### 单节点模式使用
```java
String urlString = "http://localhost:8983/solr/files";
SolrClient solrClient = new HttpSolrClient.Builder(urlString).build();
```

##### 集群模型使用
```java
// Using a ZK Host String
private static final String zkHostString = "10.128.91.191:2181,10.128.91.191:2182,10.128.91.191:2183";
CloudSolrClient solrClient = new CloudSolrClient.Builder().withZkHost(zkHostString).build();
# 设置默认查询Collection
solrClient.setDefaultCollection("appkms-db");
 
```

#### 创建与Document 对应的Java Bean
```
public class Search {
  @Field
  private String id;
  @Field
  private String name;
  @Field("descs")//如果和managed-scheme里面配置的field name不一致需要在注解内指定
  private String description;
  @Field
  private String area;
  @Field("last_modified")
  private String lastModified;
  //省略 getter setter 方法
}
```
#### 使用Solr Client API
```java
// 查询
@Test
public void query() throws IOException, SolrServerException {
  String keyword = "测试";//关键词
  //构造查询参数
  SolrQuery params = new SolrQuery();
  params.setQuery("keyword");//查询条件
  params.setStart(0);//分页起始偏移量(非页码)
  params.setRows(10);//分页尺寸
  params.setHighlight(true);//开启高亮
  params.addHighlightField("*");//添加需要高亮的字段
  params.setHighlightSimplePre("<span style=\"color:red\">");//高亮文本前缀
  params.setHighlightSimplePost("<span/>");//高亮文本刚、后缀

  //执行查询，获得响应结果
  QueryResponse response = solrClient.query(params);
  //将查询结果转换为Java Bean (Json - > Bean)
  List<Search> searchList = response.getBeans(Search.class);
  Assert.assertTrue(searchList.size() == 1);
}

// 添加索引
@Test
public void addOne() throws IOException, SolrServerException {
    solrClient.addBean(new Search(...));
}
//添加多个
@Test
public void addList() throws IOException, SolrServerException {
    int i = 1;
    int max = 100;

    List<Query> queryList = new ArrayList<Query>(max);
    while (i <= max) {
        queryList.add(new Query(...));
        i++;
    }

    solrClient.addBeans(userList);
}

@Test
public void delete() throws IOException, SolrServerException {
    solrClient.deleteById("1");//删除一个

    solrClient.deleteByQuery("*:*");//删除所有
}
```
除了查询当然还有像addBeans()、addBean()、deleteById()、deleteByQuery()等API，基本就是增删改查啦 Solr J已经帮我们封装的极其简单啦.当然Solr Client也包括操纵Collection、Zookeeper的一些API，这些都是通过Http Client Get 请求对Solr HTTP REST API的封装。

在实际使用场景下，我们使用客户端通常也只是做查询操作，而数据的导入、同步(增量、全量)、Collection的管理、Zookeeper的管理还是使用Shell脚本来比较合理。

##### 调用/update/extract完成文件索引

```java

    SolrClient client;
    
    //单个文件索引
    public void  indexFromFile(String fileName,Integer cid,String id) throws Exception{
        //ContentStreamUpdateRequest 是专门用来提交文件的
        ContentStreamUpdateRequest  request=new ContentStreamUpdateRequest("/update/extract");
        String contentType="application/text";

        request.addFile(new File(fileName), contentType);
        //literal.xxx 文件以外的字段，xxx将直接映射到schema.xml中的同名字段
        request.setParam("literal.id", String.valueOf(id));  
        request.setParam("literal.cid", cid);  

        request.setAction(AbstractUpdateRequest.ACTION.OPTIMIZE, true, true);   
        client.request(request);

        client.commit();

    }
    
```
调用
```
client.indexFromFile("/test.pdf", 1, "test.pdf");
```

# API查询参数详解

## 查询参数
通过向 Solr 集群 GET 请求/solr/core-name/select?query形式的查询 API 完成查询，其中 core-name 为查询的 Core 名称。查询语句 query 由以下基本元素项组成，按使用频率先后排序：

名称 | 描述| 示例
---|---|---|
wt | 响应结果的格式  | json
fl | 指定结果集的字段  | *（所有字段）
fq | 过滤查询  | id:5
start | 指定结果集起始返回的行数，默认 0  | 0
rows | 指定结果集返回的行数，默认 10  | 15 
sort | 结果集的排序规则  | id + asc
defType | 设置查询解析器名称  | dismax 
timeAllowed | 查询超时时间  |
### defType
defType设置查询解析器名称。如果不特别指定，默认defType=lucene<br/>
可选参数：dismax,edismax<br/>
Solr默认有三种查询解析器（Query Parser）：
- Standard Query Parser
- DisMax Query Parser
- Extended DisMax Query Parser (eDisMax)<br/>
第一种是标准的Parser，最后一种是最强大的，也是Sunspot默认使用的Parser。

### wt
wt 设置结果集格式，支持 json、xml、csv、php、ruby、pthyon，序列化的结果集，常使用 json 格式。

### fl
fl 指定返回的字段，多指使用“空格”和“,”号分割，但只支持设置了stored=true的字段。*表示返回全部字段，一般情况不需要返回文档的全部字段。

**字段别名**：使用displayName:fieldName形式指定字段的别名，例如：
```
fl=id,name,last_modified:lastModify,createUser
```
**函数**：fl还支持使用Solr内置函数

### fq 
fq 过滤查询条件，可充分利用 cache，所以可以利用 fq 提高检索性能。<br/>
写法：fq=<field_name>:<value><br/>
样例：fq=net_type:1；fq=net_type:1 AND (idt_id:12011 OR idt_id:5004) AND time_type:1；fq=net_type:1&fq=idt_id:12011

### sort
sort 指定结果集的排序规则，格式为<fieldName>+<sort>，支持 asc（倒序） 和 desc（正序，默认值）【不区分大小写】 两种排序规则。例如按照名称倒序排列：
```
sort=name+desc
```
也可以多字段排序，id和名称排序
```
sort=id+asc,name+desc
```
## 查询语法
1. 匹配所有文档：\*:*
2. 强制、阻止和可选查询：
    1)    Mandatory：查询结果中必须包括的(for example, only entry name containing the word make)
Solr/Lucene Statement：+make, +make +up ,+make +up +kiss
    2)    prohibited：(for example, all documents except those with word believe)
Solr/Lucene Statement：+make +up -kiss
    3)    optional：
Solr/Lucene Statement：+make +up kiss
3. 布尔操作：AND、OR和NOT布尔操作（必须大写）与Mandatory、optional和prohibited相似。
    1)  make AND up ＝ +make +up :AND左右两边的操作都是mandatory
    2)  make || up ＝ make OR up＝make up :OR左右两边的操作都是optional
    3) +make +up NOT kiss ＝ +make +up –kiss
    4) make AND up OR french AND Kiss不可以达到期望的结果，因为AND两边的操作都是mandatory的。
4. 子表达式查询（子查询）：可以使用“()”构造子查询。
示例：(make AND up) OR (french AND Kiss)
5. 子表达式查询中阻止查询的限制：
示例：make (-up):只能取得make的查询结果；要使用make (-up *:*)查询make或者不包括up的结果。
6. 多字段fields查询：通过字段名加上分号的方式（fieldName:query）来进行查询
示例：entryNm:make AND entryId:3cdc86e8e0fb4da8ab17caed42f6760c
7. 通配符查询（wildCard Query）：
    1) 通配符？和*：“*”表示匹配任意字符；“？”表示匹配出现的位置。
示例：ma?*（ma后面的一个位置匹配），ma??*(ma后面两个位置都匹配)
    2) 查询字符必须要小写:+Ma +be**可以搜索到结果；+Ma +Be**没有搜索结果.
    3) 查询速度较慢，尤其是通配符在首位：主要原因一是需要迭代查询字段中的每个term，判断是否匹配；二是匹配上的term被加到内部的查询，当terms数量达到1024的时候，查询会失败。
    4) Solr中默认通配符不能出现在首位（可以修改QueryParser，设置
setAllowLeadingWildcard为true）
    5) set setAllowLeadingWildcard to true.
8. 模糊查询、相似查询：不是精确的查询，通过对查询的字段进行重新插入、删除和转换来取得得分较高的查询解决（由Levenstein Distance Algorithm算法支持）。
    1) 一般模糊查询：示例：make-believ~
    2) 门槛模糊查询：对模糊查询可以设置查询门槛，门槛是0~1之间的数值，门槛越高表面相似度越高。示例：make-believ~0.5、make-believ~0.8、make-believ~0.9
9. 范围查询（Range Query）：Lucene支持对数字、日期甚至文本的范围查询。结束的范围可以使用“*”通配符。
示例：
    1) 日期范围（ISO-8601 时间GMT）：sa_type:2 AND a_begin_date:[1990-01-01T00:00:00.000Z TO 1999-12-31T24:59:99.999Z]
    2) 数字：salary:[2000 TO *]
    3) 文本：entryNm:[a TO a]
10. 日期匹配：YEAR, MONTH, DAY, DATE (synonymous with DAY) HOUR, MINUTE, SECOND, MILLISECOND, and MILLI (synonymous with MILLISECOND)可以被标志成日期。
示例：
    1)  r_event_date:[* TO NOW-2YEAR]：2年前的现在这个时间
    2)  r_event_date:[* TO NOW/DAY-2YEAR]：2年前前一天的这个时间

## 高亮显示
solr 高亮显示是根据我们搜索的内容中，根据搜索的关键字，在内容中取一段摘要，类似百度，google搜索结果中出现关键字的一段描述。

首先要solr的高亮功能work，必须在搜索请求的URL里加上参数：hl=on,该参数只是告诉solr我要高亮查询。接下来就是其他的参数，下面主要说明重要的参数：

solr 高亮显示是根据我们搜索的内容中，根据搜索的关键字，在内容中取一段摘要，类似百度，google搜索结果中出现关键字的一段描述。

首先要solr的高亮功能work，必须在搜索请求的URL里加上参数：hl=on,该参数只是告诉solr我要高亮查询。接下来就是其他的参数，下面主要说明重要的参数：

### hl.fl  
hl.fl是说明你要关键字的摘要在那个field中取，我们一般是content字段。
### hl.useFastVectorHighlighter

该参数很重要，如果你不看代码，是很难发现他的好处，默认是false，即文本段的划分是按每50个字符来划分，然后在这个50个字符中取关键字相关的摘要，摘要长度为100，参考后面的参数（hf.fragsize)，如果我们在参数中指定值为true，那么SOLR
会根据关键词的在文本中的偏移量来计算摘要信息，前提是你的field要加上 termPositions="true" termOffsets="true"这两项。

### hl.snippets
hl.snippets参数是返回高亮摘要的段数，因为我们的文本一般都比较长，含有搜索关键字的地方有多处，如果hl.snippets的值大于1的话，会返回多个摘要信息，即文本中含有关键字的几段话，默认值为1，返回含关键字最多的一段描述。solr会对多个段进行排序。

### hl.fragsize

hl.fragsize参数是摘要信息的长度。默认值是100，这个长度是出现关键字的位置向前移6个字符，再往后100个字符，取这一段文本。

### hl.boundaryScanner  hl.bs.maxScan  hl.bs.chars

boundaryScanner是边界扫描，就是怎么取我们高亮摘要信息的起始位置和结束位置，这个是我们摘要信息的关键，因为我们模式高亮摘要的开始和结束可能是某句话中截段的。上面这三个参数需要放在一起来说明，因为后两个是在我们没有给hl.boundaryScanner设定值，即默认值时才会有效，对应的class为：SimpleBoundaryScanner，上面讲了hl.fragsize参数，
SimpleBoundaryScanner会根据hl.fragsize参数决定的关键字的起始偏移量和结束偏移量，重新计算摘要的起始偏移量，首先说开始偏移量，源码如下：
```java
public int findStartOffset(StringBuilder buffer, int start) {
    // avoid illegal start offset
    if( start > buffer.length() || start < 1 ) return start;
    //maxScan是hl.bs.maxScan的值，它是说明从关键字出现的位置往前6个字符开始向前，在maxScan个字符内找是否出 现一个
     //一个由参数hl.bs.chars指定的分界符。即从这里作为摘要的起始偏移。 如果往前maxScan个字符内没有发现指定的字符，
    //则按起始便宜为start，即关键词往前的6个字符。 
    int offset, count = maxScan;
    for( offset = start; offset > 0 && count > 0; count-- ){
      // found?
      if( boundaryChars.contains( buffer.charAt( offset - 1 ) ) ) return offset;
      offset--;
    }
    // if we scanned up to the start of the text, return it, its a "boundary"
    if (offset == 0) {
      return 0;
    }
    // not found
    return start;
  }
```
结束便宜量和计算起始偏移量是一样的，只不过是从关键词的位置往后100个字符的位置往后找分隔符，maxScan的默认值为10，hl.sc.chars的默认值为.,!? &#9;&#10;&#13;

hl.boundaryScanner参数我们不指定值时，按上面的算法计算高亮摘要信息，但solr还提供了一种算法，即breakIterator，我们在请求参数中添加&hl.boundaryScanner=breakIterator时生效，这是solr通过java jdk的BreakIterator来计算分界符的。他相关的参数为，hl.bs.type，这个是主要的参数，决定BreakIterator怎么划分界定符，值有：CHARACTER, WORD, SENTENCE and LINE，SENTENCE 是按句子来划分，即你高亮摘要信息是一个完整的句子，而不会被截断。

