### Solr配置文件说明
#### managed-schema
该配置文件主要用于配置数据源，字段类型定义，搜索类型定义等，是加载数据，创建索引和数据时的核心数据结构的配置文件。
solr的数据结构如下：
- document：一个文档、一条记录
- field：域、属性
solr通过搜索某个或某些field，返回若干个符合条件的document，或者按搜索的score排序返回。

如果跟数据库对比，document相当于数据库的表，field相当于表中的字段。而managed-schema就是为了定义一个表的结构（定义各个field的名字、类型、约束、等等）。


基本结构为：
```xml
<schema>
    <types>
    <fields>
    <uniquekey>
    <copyFiled>
</schema>
```
常用配置说明：
- field：定义一个document中的各个fields
    - name：必填。该field的名字。前后都有下划线的name是系统保留的名字，比如“_version_”
    - type：必填。类型，对应于fieldType的name
    - default：字段的默认值，经常用在字段是必须的，但是有时候又无法提供的情况，solr就会用默认值替代。
    ```
    <field name="recordTime" type="date" indexed="true" stored="true" required="true" default="NOW+8HOUR"/>
    ```
    - indexed：true/false，是否为该field建立索引，以让用户可以搜索它、统计它（facet），在lucene中，被索引的字段将会建立倒排表.
    -  stored：true/false，一个字段是否被存储，取决于你是否想在solr的查询结果中得到它，也就是说你是否想在查询结果中看到它，它将会消耗cpu和io和磁盘空间等资源。
    -  multiValued：true/false，是否可以容纳多个值（比如多个copyField的dest指向它）。如果是true，则该field不能被排序、不能作为uniqueKey
    -  required：true/false，提示solr拒绝添加没有这个字段的任何文档，默认这个值设置为假。简单来说就是这个字段是必须的，类似数据库的非空，如果添加的文档字段为空，则添加不到索引上去。
    -  docValues：true/false，建立document-to-value索引，以提高某些特殊搜索的效率（排序、统计、高亮）
    - omitNorms
    - termVectors
    - termPositions和termOffset、termPayLoads
    - precisionStep 和positionIncrementGap
    - sortMissingFirst 和sortMissingLast
- copyField：把一个field的内容拷贝到另外一个field中。一般用来把几个不同的field copy到同一个field中，以方便只对一个field进行搜索
    - source：被拷贝的field，支持用通配符指定多个field，比如：*_name
    - dest：拷贝到的目的field
    - maxChars：最大字符数
- uniqueKey：指定一个field为唯一索引
- fieldType：定义field的类型，包括下面一些属性
    - name：必填，被field配置使用
    - class：必填，filedType的实现类。solr.TextField是路径缩写，"等价于"org.apache.solr.schema.TextField"
    - multiValued：？
    - positionIncrementGap：指定mutiValued的距离
    - ananlyzer：如果class是solr.TextField，这个配置是必填的。告诉solr如何处理某些单词、如何分词，比如要不要去掉“a”，要不要全部变成小写……
        - type：index或query
        - tokenizer：分词器，比如：StandardTokenizerFactory
        - filter：过滤器，比如：LowerCaseFilterFactory
- dynamicField：用通配符定义一个field来存在没有被field定义的漏网之鱼
    - name：使用通配符，比如“*_i”，来处理类似“cost_i”之类的field 
#### solrconfig.xml 索引配置和查询处理配置
solrconfig.xml配置文件主要定义了SOLR的一些处理规则，包括索引数据的存放位置，更新，删除，查询的一些规则配置。
- datadir 节点：<dataDir>${solr.data.dir:}</dataDir> 定义了索引数据和日 志文件的存放位置
- luceneMatchVersion：<luceneMatchVersion>6.6.2</luceneMatchVersion>表 示 solr 底 层 使 用 的 是 lucene6.6.2
- lib:<lib dir="../...jar"/> 表示 solr 引用包的位置, 当 dir 对应的目录不存在时候，会忽略此属性
##### directoryFactory: 索引存储方案，共有以下存储方案
- solr.StandardDirectoryFactory,这是一个基于文件系统存储目录的工厂，它会试 图选择最好的实现基于你当前的操作系统和 Java 虚拟机版本。
-  solr.SimpleFSDirectoryFactory,适用于小型应用程序，不支持大数据和多线程。
-  solr.NIOFSDirectoryFactory,适用于多线程环境，但是不适用在 windows 平台 （很慢），是因为 JVM 还存在 bug。
-  solr.MMapDirectoryFactory,这个是 solr3.1 到 4.0 版本在 linux64 位系统下默认 的实现。它是通过使用虚拟内存和内核特性调用 mmap 去访问存储在磁盘中 的索引文件。它允许 lucene 或 solr 直接访问 I/O 缓存。如果不需要近实时搜 索功能，使用此工厂是个不错的方案。
- solr.NRTCachingDirectoryFactory,此工厂设计目的是存储部分索引在内存中， 从而加快了近实时搜索的速度。
- solr.RAMDirectoryFactory,这是一个内存存储方案，不能持久化存储，在系统 重启或服务器 crash 时数据会丢失。且不支持索引复制
##### 索引indexConfig

Solr 性能因素，来了解与各种更改相关的性能权衡。 下面概括了可控制 Solr 索引处理的各种因素：
- useCompoundFile：通过将很多 Lucene 内部文件整合到一个文件来减少使用中的文件的数量。这可有助于减少 Solr 使用的文件句柄数目，代价是降低了性能。除非是应用程序用完了文件句柄，否则 false 的默认值应该就已经足够。
- ramBufferSizeMB：
- maxBufferedDocs：
- mergeFactor：	
决定低水平的 Lucene 段被合并的频率。较小的值（最小为 2）使用的内存较少但导致的索引时间也更慢。较大的值可使索引时间变快但会牺牲较多的内存。
- maxIndexingThreads：indexWriter生成索引时使用的最大线程数
- unlockOnStartup：告知 Solr 忽略在多线程环境中用来保护索引的锁定机制。在某些情况下，索引可能会由于不正确的关机或其他错误而一直处于锁定，这就妨碍了添加和更新。将其设置为 true 可以禁用启动锁定，进而允许进行添加和更新。
- lockType：
    - single: 在只读索引或是没有其它进程修改索引时使用.
    - native: 使用操作系统本地文件锁,不能使用多个Solr在同一个JVM中共享一个索引.
    - simple :使用一个文本文件锁定索引.
- updateHandler：设置索引库更新日志，默认路径为 solrhome 下面的 data/tlog。随着索引库的频 繁更新，tlog 文件会越来越大，所以建议提交索引时采用硬提交方式 ，即批量提交。

##### 查询配置query
- maxBooleanClauses：最大的BooleanQuery数量. 当值超出时，抛出 TooManyClausesException.注意这个是全局的,如果是多个SolrCore都会使用一个值,每个Core里设置不一样的化,会使用最后一个的.
- filterCache：filterCache存储了无序的lucene document id集合，
    - 存储了filter queries(“fq”参数)得到的document id集合结果。
    - 还可用于facet查询3.  
    - 如果配置了useFilterForSortedQuery，那么如果查询有filter，则使用filterCache。
- queryResultCache：缓存搜索结果,一个文档ID列表
- documentCache：缓存Lucene的Document对象,不会自热
- fieldValueCache：	
字段缓存使用文档ID进行快速访问。默认情况下创建fieldValueCache即使这里没有配置。
- enableLazyFieldLoading：	
若应用程序预期只会检索 Document 上少数几个 Field，那么可以将属性设置为 true。延迟加载的一个常见场景大都发生在应用程序返回和显示一系列搜索结果的时候，用户常常会单击其中的一个来查看存储在此索引中的原始文档。初始的显示常常只需要显示很短的一段信息。若考虑到检索大型 Document 的代价，除非必需，否则就应该避免加载整个文档。
- queryResultWindowSize：一次查询中存储最多的doc的id数目.
- queryResultMaxDocsCached：查询结果doc的最大缓存数量, 例如要求每页显示10条,这里设置是20条,也就是说缓存里总会给你多出10条的数据.让你点示下一页时很快拿到数据.
- listener：选项定义 newSearcher 和 firstSearcher 事件，您可以使用这些事件来指定实例化新搜索程序或第一个搜索程序时应该执行哪些查询。如果应用程序期望请求某些特定的查询，那么在创建新搜索程序或第一个搜索程序时就应该反注释这些部分并执行适当的查询。
- useColdSearcher：是否使用冷搜索,为false时使用自热后的searcher
- maxWarmingSearchers：最大自热searcher数量

#### dataconfig.xml（数据源配置）
