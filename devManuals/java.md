# Java开发规范

## 通用规范
所有的情况下都通用
1. 【强制】命名全部使用英文，禁止中文或者中英混合。项目名除外，因为有的项目是按域名来命名的，域名本身有可能是中文拼音。
```
域名：kecheng.xxx.com
项目名：xxx-web-kecheng
```
2. 【强制】禁止使用缩写，除非提供一个缩写列表

反例：
```
# 这里的t到底是什么意思？topic_id?还是teacher_id?
字段：t_id
```
3.  【强制】禁止出现除了后缀或者前缀3个单词。如果超过3个，说明想表达的职责太多，可以拆分或者封装。

4. 【强制】使用maven构建项目。

    使用maven的优势
    - maven是跨平台的，最大的消除了构建的重复。
    - maven不仅是构建工具，它还是依赖管理工具和项目管理工具，提供了中央仓库，能够帮我们自动下载构件。
    - 为了解决的依赖的增多，版本不一致，版本冲突，依赖臃肿等问题，它通过一个坐标系统来精确地定位每一个构件（artifact）。
    - 还能帮助我们分散在各个角落的项目信息，包括项目描述，开发者列表，版本控制系统，许可证，缺陷管理系统地址。
    - maven还为全世界的java开发者提供了一个免费的中央仓库，在其中几乎可以找到任何的流行开源软件。通过衍生工具(Nexus),我们还能对其进行快速搜索
    - maven对于目录结构有要求，约定优于配置，用户在项目间切换就省去了学习成本。
## JAVA编程语言

### 类

1. 【强制】需要有统一的后缀或者前缀。为了一看类名，就知道这个类干什么的。

前缀列表：
- 抽象类（Abstract）
- 接口（I）

正例：

接口：`IViewTag` <br/>
抽象类：`AbstractViewTag`<br/>
具体实现类：`UserViewTag`

后缀列表：
- 实体(Entity)。数据库持久对象。
- 表单（Form)。用于封装、校验http参数。
- 数据传输对象(DTO)。用于暴露接口的返回数据
- 基础服务(BaseService)。单实体可以自描述的服务。
- 业务服务(BusinessService)。集合多个单实体的服务。
- 页面服务(ViewService)。涉及到视图页面的服务。
- 模块(Module)。http入口模块。
- 异常(Exception)
- 工具(Util)
- 枚举(Enum)
- 视图标签（ViewTag）
- ....(其他的，比如：Filter之类）


正例：

实体：`UserEntity`<br/>
基础服务：`UserBaseService`<br/>
业务服务：`AuthorityBusinessService`



2. 【强制】所有参与业务的类禁止使用内部类。
3. 【强制】修饰符应该按照如下顺序排列：public, protected, private, abstract, static, final, transient, volatile, synchronized, native, strictfp。
4. 【建议】类内容定义顺序。
- 静态成员变量 / Static Fields
- 静态初始化块 / Static Initializers
- 成员变量 / Fields
- 初始化块 / Initializers
- 构造器 / Constructors
- 静态成员方法 / Static Methods
- 成员方法 / Methods
- 重载自Object的方法如toString(), hashCode() 和main方法
- 类型(内部类) / Types(Inner Classes)

    同等的顺序下，再按public, protected,default, private的顺序排列。


### 属性
1. 【强制】常量必须是：大写+下划线，禁止多个单词连在一起

反例：
```
private final static String PAGESIZE=10;
private final static String pageSize=10;
```
正例：
`private final static String PAGE_SIZE=10;`
2. 【强制】布尔类型禁止添加"is"前缀。部分框架解析会引起序列化错误。

反例：
```
# 对应的getter和setter为：isRead和setRead
private boolean isRead
```
正例:
```
# 对应的getter和setter为：isRead和setRead
private boolean read;
```
3. 【强制】计数器禁止使用复数

反例：
`private int readCounts;`

正例:
`private int readCount;`
4. 【强制】自描述属性里不要出现类名的描述

反例：
```java
//UserEntity类
private String userName;
private int userAge;
```
正例：
```java
//UserEntity类
private String name;
private int age;
```
5. 【强制】关联其他实体的属性命名规则：对应的实体去掉后缀+用途。

反例：

`属性名：tId。根本不知道是哪个实体的外键。有可能是Teacher有可能是Topic，还得猜半天`

正例：
```
属性名：teacherId ,对应的实体是TeacherEntity
属性名：favorCount，对应的实体是FavorEntity
```
6. 【强制】禁止通过定义定义成常量（1，2）来维护类型值，需要通过枚举

反例：
```
private final static int SUCESS=1;
private final static int FAIL=2;
```
正例：`定义一个枚举`

### 方法
1. 【强制】接口里的方法禁止有修饰符。

反例：`public void eat();`

正例：`void eat();`


2. 【推荐】方法参数必须使用final来修饰。final可提高程序响应效率。

反例：`public void eat(int size);`

正例：`public void eat(final int size);`


3. 【强制】每一个方法参数都需要被处理。module层的方法里的对象参数可以不判空，因为架构已经做处理了，不可能为空。

    被处理指的是：

    - 抛异常。
    - 直接返回。
    - 有对应的业务处理逻辑。

    例子：
    ```
    public void add(long userId,String content){
        //异常验证
        ExceptionUtil.checkId(userId,"用户id")
        //直接返回
        if(Util.isEmpty(content)){
            return ;
        }
    }

    public List<CourseEntity> list(int type){
        Cnd cnd = Cnd.limit();
        //有对应的业务逻辑处理
        if(type>0){
            cnd.and("type","=",type);
        }
        return dbDao.query(CourseEntity.class,cnd,null);
    }
    ```
4. 【强制】同一个类里有多个一致的参数（3个以上）的方法，需要抽取接口或者通过实体来承载。

反例：
```
public FavorEntity add(int type,long sourceId,long userId);
public FavorEntity delete(int type,long sourceId,long userId);
```
正例：
```
public FavorEntity add(IFavor favor);
public FavorEntity delete(IFavor favor);
```
5. 【强制】方法名必须是动词或者动宾。http接口需要知明达意，可以不按这个规则。
比如：mycourse,home,banner

    方法命名格式：
    - is+动词|形容词
    - 动词【+名词|形容词】

    例子：
    ```
    public void isSucess();
    public void on();
    public void sendEmail();
    ```

    统一命名列表：
    - add 新增
    - update 修改
    - delete 删除
    - get 获取单个对象
    - list 获取集合对象
    - getMap 获取map数据
    - count 数量

    方法前缀后缀命名说明：
    原则上不添加后缀，如果添加后缀的话，如果有添加，命名格式为：updatexxxx4yyyyByzzzz
    - xxxx：表示对象的属性
    - yyyy：表示查询的条件（根据自描述属性查询）
    - zzzz：表示查询的条件（根据其他描述属性查询）

    例子：
    ```
    --xxxx情况:用户
    public void updateName();();
    public void updateNickName();
    --yyyy情况：资讯
    public List<NewsEntity> list4Latest();
    public List<NewsEntity> list4Top();
    --zzzz情况：课程
    public List<CourseEntity> listByTeacher();
    public List<CourseEntity> listByKnowledge();

    --综合使用
    public List<CourseEntity> listCourse4TopByTeacher();
    ```
6. 【强制】一个方法里代码行数不能超过1屏（即30行）。一般来说超过30的行，业务关注点、复杂数比较高，很难维护。超过３０行需要封装方法
7. 【强制】局部变量命名不能有连续的名称。连续的命名不具有可维护性。每个变量都需要有清晰的概念。

反例：
```
String head1;
String head2;
```
正例：
```
String title;
String content;
```
8. 【强制】禁止有任何魔鬼数据独立存在。可以定义一个有含义的变量来承载

反例：
```
if(type ==1){
  // TODO somthing
}
```
正例：
```
private final static int SUCESS=1;
...
.if(type ==SUCESS){
// TODO somthing
}
```
9. 【强制】判断表达式要使用布尔变量或者封装方法。

反例：
```
if(user!=null&&!Util.isEmpty(user.name)&&!Util.isEmpty(user.provicne)){
 // TODO somthing
}
```
正例：
```
if(isFilledBaseInfo(user)){
 // TODO somthing
}
```
10. 【强制】if()...else if()...else个数不能多于4个，嵌套不能深于3层

    可以通过以下的方法来消除：

    - 设计模式: 策略模式
    反例：
    ```
    public class UseIfElse {
    	public static void main(String args[]){
    		MyPaper myPaper = new MyPaper(PaperColor.RED);
    		if(myPaper.getMyPaperColor() == PaperColor.BLACK){
    			System.out.println("You need a black pen!");
    		}else if(myPaper.getMyPaperColor() == PaperColor.BLUE){
    			System.out.println("You need a blue pen!");
    		}else if(myPaper.getMyPaperColor() == PaperColor.RED){
    			System.out.println("You need a red pen!");
    		}else if(myPaper.getMyPaperColor() == PaperColor.WHITE){
    			System.out.println("You need a white pen!");
    		}
    	}
    }

    class MyPaper{
    	private PaperColor paperColor;
    	public MyPaper(PaperColor paperColor){
    		this.paperColor = paperColor;
    	}
    	public PaperColor getMyPaperColor(){
    		return this.paperColor;
    	}
    }
    enum PaperColor{
    	WHITE, BLACK, BLUE, RED
    }
    ```
    正例：
    ```
    public class NoIfElse {
    	public static void main(String args[]){
    		MyPaper myPaper = new MyPaper(new White());
    		myPaper.choicePen();
    	}
    }
    interface PaperColor{
    	public void getPenColor();
    }
    class White implements PaperColor{
    	public void getPenColor(){
    		System.out.println("You need a white pen!");
    	}
    }
    class Red implements PaperColor{
    	public void getPenColor(){
    		System.out.println("You need a red pen!");
    	}
    }
    class Blue implements PaperColor{
    	public void getPenColor(){
    		System.out.println("You need a blue pen!");
    	}
    }
    class Black implements PaperColor{
    	public void getPenColor(){
    		System.out.println("You need a black pen!");
    	}
    }
    class MyPaper{
    	private PaperColor paperColor;
    	public MyPaper(PaperColor paperColor){
    		this.paperColor = paperColor;
    	}
    	public PaperColor getPaperColor(){
    		return this.paperColor;
    	}
    	public void choicePen(){
    		this.paperColor.getPenColor();
    	}
    }
    ```
    - 抽取方法
    - 使用return

    反例：
    ```
    if(isAdmin()){
     ...
    }else if(isTeacher()){
     ...
    }
    ```
    正例：
    ```
    if(isAdmin()){
     ... return;
    }if(isTeacher()){
     ... return;
    }
    ```
11. 【推荐】采用防御式编程，先判断错误的业务，然后再写正确的业务。

防御式编程结构清晰分明：先把所有错误穷举，然后集中处理正确逻辑。

反例：
```
if(null!=user && user.hasAuth()){
 正确逻辑
}
```
正例：
```
if( null == null || !user.hasAuth()){return;}

正确逻辑
```
12. 【推荐】for里不建议写io。io包括：数据库、缓存，文件读写等
13. 【强制】多个不同的结构（业务相近的代码），需要有且只有一个空行
反例：
```
long userId = fetchUser.getCurrentUserId();
Sql sql = latentCustomerQueryForm.pager(sqlManager);
Map<String, Object> map = FormUtil.list(dbDao, sql, pager);

List<DictInfoEntity> grades = dictBaseService.listDict(DictInfoEnum.GRADE.stringKey());
List<DictInfoEntity> infoOrigins = dictBaseService.listDict(DictInfoEnum.INFOORIGIN.stringKey());

map.put("queryForm", latentCustomerQueryForm);
map.put("grades", grades);
map.put("infoOrigins", infoOrigins);
return map;
```
正例：
```
long userId = fetchUser.getCurrentUserId();
Sql sql = latentCustomerQueryForm.pager(sqlManager);
Map<String, Object> map = FormUtil.list(dbDao, sql, pager);

List<DictInfoEntity> grades = dictBaseService.listDict(DictInfoEnum.GRADE.stringKey());
List<DictInfoEntity> infoOrigins = dictBaseService.listDict

map.put("queryForm", latentCustomerQueryForm);
map.put("grades", grades);
map.put("infoOrigins", infoOrigins);
return map;
```
14. 【推荐】不参与计算的变量不要定义变量。

反例：
```
long userId = fetchUser.getCurrentUserId();
Sql sql = latentCustomerQueryForm.pager(sqlManager);
Map<String, Object> map = FormUtil.list(dbDao, sql, pager);

List<DictInfoEntity> grades = dictBaseService.listDict(DictInfoEnum.GRADE.stringKey());
List<DictInfoEntity> infoOrigins = dictBaseService.listDict(DictInfoEnum.INFOORIGIN.stringKey());

map.put("queryForm", latentCustomerQueryForm);
map.put("grades", grades);
map.put("infoOrigins", infoOrigins);
return map;
```
正例：
```
long userId = fetchUser.getCurrentUserId();
Sql sql = latentCustomerQueryForm.pager(sqlManager);
Map<String, Object> map = FormUtil.list(dbDao, sql, pager);

map.put("grades", dictBaseService.listDict(DictInfoEnum.GRADE.stringKey()));
map.put("infoOrigins", dictBaseService.listDict(DictInfoEnum.INFOORIGIN.stringKey()));
return map;
```
15. 【强制】如果有捕获异常，必须有对应的处理业务。如果没有对应的处理业务，不要捕获，可以直接throw，让架构统一处理

自己catch，肯定不希望用户看到错误日志，那么 从用户那边看到是正常业务。catch了什么都没干，用户往往看到的是什么都没发生，他会以为服务挂了或者功能快。

16. 【强制】禁止使用exception.getMessge()处理错误信息。
应该使用exception.toString()。因为exception.getMessage()，在抛出空指针异常的时候，什么信息都不显示。

反例：
```java
try{
    // TODO somthing
} catch (Exception e) {
    LOG.error(e.getMessage());
}
```
正例：
```java
try{
    // TODO somthing
} catch (Exception e) {
    LOG.error(e.toString());
}
```
17. 【强制】禁止使用System.out.print。
统一使用Log4j2插件生成日志（不要定义具体的日志实现，要定义的是slf4j的接口）
18. 【推荐】公开的接口，一旦发布成稳定版，禁止修改方法签名（方法名，参数）
如果要修改,需要提供新的接口，老的不能修改。
比如：js调用可能就报错了，功能就没办法使用；工具类接口一变，其他项目就会报错了，没办法向下兼容。
19. 【推荐】方法放置顺序：public-->protected-->private。
    一个类，往往使用者更关注的是public的。构造方法、重载方法、雷同方法，按顺序放在一起


### 注释
1. 【强制】格式结构统一使用eclipse模板，禁止自定义。
2. 【强制】类、方法、属性都必须有注释。如果实在来不及，可以先生成TODO。因为可以通过TODO视图，把注解补回来。
3. 【强制】类上必须要有作者，如果有修改，还要添加上修改者，如果有结队也要写上。要有用户名还要有邮箱

例子：

```java
/**
 * 字典
 *
 * @author   yaloo(yaloo_yang@189.cn)
 * @version  2017-09-06
 */
```
4. 【强制】注释要直译，描述要写算法或者思路或者注意事项。不要在注释上代码里的每一行完全暴露出来，使用者根本不关注实现。

反例：
```java
/**
 * 学生分页查询   #方法里根据没有学生。。。。
 *
 * @param page 分页对象
 * @param studentForm 学生
 * @return 分页对象
 */
public Page<Map<String, Object>> findPageList(Page<Map<String, Object>> page, StudentForm studentForm) {
```
5. 【强制】方法里禁止写注释。不要有多余的注释，让变量和属性自描述或者抽取方法。如果有算法写到方法注释上。

反例：
```java
//填充基本信息
fillBaseInfo();
//填充账号信息
fillAccoutInfo();
```
6. 【强制】方法里禁止注释掉代码。统一通过版本控制软件（git)来解决。逻辑是正确的，但是现在暂时不能使用，可以暂时注释，但是必须写上TODO。

例子：
```java
//TODO 张三 当前用户还未处理，因为登录还没有调通
//long userId = fetchUser.getCurrentUserId();
long userId = 1L;
....
```

### 其它
1. 字符串连接误用

错误的写法：
```
String s = "";
for (Person p : persons) {
    s += ", " + p.getName();
}
s = s.substring(2); //remove first comma
```
正确的写法：
```
StringBuilder sb = new StringBuilder(persons.size() * 16); // well estimated buffer
for (Person p : persons) {
    if (sb.length() > 0) sb.append(", ");
    sb.append(p.getName);
}

```

2. 错误的使用StringBuffer

错误的写法：
```
StringBuffer sb = new StringBuffer();
sb.append("Name: ");
sb.append(name + '\n');
sb.append("!");
...
String s = sb.toString();
```
问题在第三行，append char比String性能要好，另外就是初始化StringBuffer没有指定size，导致中间append时可能重新调整内部数组大小。如果是JDK1.5最好用StringBuilder取代StringBuffer，除非有线程安全的要求。还有一种方式就是可以直接连接字符串。缺点就是无法初始化时指定长度。

正确的写法：
```
StringBuilder sb = new StringBuilder(100);
sb.append("Name: ");
sb.append(name);
sb.append("\n!");
String s = sb.toString();
```
或者这样写：

`String s = "Name: " + name + "\n!";`

3. 数字转换成字符串

错误的写法：
```
"" + set.size()
new Integer(set.size()).toString()
```
正确的写法：

`String.valueOf(set.size())`
4. 未对数据流进行缓存

错误的写法：
```java
InputStream in = new FileInputStream(file);
int b;
while ((b = in.read()) != -1) {
...
}
```
上面的代码是一个byte一个byte的读取，导致频繁的本地JNI文件系统访问，非常低效，因为调用本地方法是非常耗时的。最好用BufferedInputStream包装一下。

正确的写法：

`InputStream in = new BufferedInputStream(new FileInputStream(file));`

5. 无限使用heap内存

错误的写法：

`byte[] pdf = toPdf(file);`

这里有一个前提，就是文件大小不能讲JVM的heap撑爆。否则就等着OOM吧，尤其是在高并发的服务器端代码。最好的做法是采用Stream的方式边读取边存储(本地文件或database)。

正确的写法：

`File pdf = toPdf(file);`
另外，对于服务器端代码来说，为了系统的安全，至少需要对文件的大小进行限制。

6. 最好用静态final定义Log变量

private static final Log log = LogFactory.getLog(MyClass.class);
这样做的好处有三：

可以保证线程安全
静态或非静态代码都可用
不会影响对象序列化

错误的代码：
```
Class clazz = Class.forName(name);
Class clazz = getClass().getClassLoader().loadClass(name);
```
这里本意是希望用当前类来加载希望的对象, 但是这里的getClass()可能抛出异常, 特别在一些受管理的环境中, 比如应用服务器, web容器, Java WebStart环境中, 最好的做法是使用当前应用上下文的类加载器来加载。

正确的写法：
```
ClassLoader cl = Thread.currentThread().getContextClassLoader();
if (cl == null) cl = MyClass.class.getClassLoader(); // fallback
Class clazz = cl.loadClass(name);
```
7. 反射使用不当

错误的写法：
```
Class beanClass = ...
if (beanClass.newInstance() instanceof TestBean) ...
```
这里的本意是检查beanClass是否是TestBean或是其子类, 但是创建一个类实例可能没那么简单, 首先实例化一个对象会带来一定的消耗, 另外有可能类没有定义默认构造函数. 正确的做法是用Class.isAssignableFrom(Class) 方法。

正确的写法：
```
Class beanClass = ...
if (TestBean.class.isAssignableFrom(beanClass)) ...
```
8. Calendar.getInstance()的误用
错误的写法：
```
Calendar c = Calendar.getInstance();
c.set(2009, Calendar.JANUARY, 15);
```
Calendar.getInstance()依赖local来选择一个Calendar实现, 不同实现的2009年是不同的, 比如有些Calendar实现就没有January月份。

正确的写法：
```
Calendar c = new GregorianCalendar(timeZone);
c.set(2009, Calendar.JANUARY, 15);
```
9. SimpleDateFormat非线程安全误用

错误的写法：
```
public class Constants {
public static final SimpleDateFormat date = new SimpleDateFormat("dd.MM.yyyy");
}
```
SimpleDateFormat不是线程安全的. 在多线程并行处理的情况下, 会得到非预期的值. 这个错误非常普遍! 如果真要在多线程环境下公用同一个SimpleDateFormat, 那么做好做好同步(cache flush, lock contention), 但是这样会搞得更复杂, 还不如直接new一个实在。

10. 使用全局参数配置常量类/接口
```
public interface Constants {
String version = "1.0";
String dateFormat = "dd.MM.yyyy";
String configFile = ".apprc";
int maxNameLength = 32;
String someQuery = "SELECT * FROM ...";
}
```
很多应用都会定义这样一个全局常量类或接口, 但是为什么这种做法不推荐? 因为这些常量之间基本没有任何关联, 只是因为公用才定义在一起. 但是如果其他组件需要使用这些全局变量, 则必须对该常量类产生依赖, 特别是存在server和远程client调用的场景。

比较好的做法是将这些常量定义在组件内部. 或者局限在一个类库内部。

11. HashMap size陷阱

错误的写法：
```
Map map = new HashMap(collection.size());
for (Object o : collection) {
  map.put(o.key, o.value);
}
```
这里可以参考guava的Maps.newHashMapWithExpectedSize的实现. 其本意是希望给HashMap设置初始值, `避免扩容(resize)的开销`. 但是没有考虑当添加的元素数量达到HashMap`容量`的`75%`时将出现resize。

正确的写法：

`Map map = new HashMap(1 + (int) (collection.size() / 0.75));`

12. 不使用finally块释放资源

错误的写法：
```java
public void save(File f) throws IOException {
    OutputStream out = new BufferedOutputStream(new FileOutputStream(f));
    out.write(...);
    out.close();
}
public void load(File f) throws IOException {
    InputStream in = new BufferedInputStream(new FileInputStream(f));
    in.read(...);
    in.close();
}
```
上面的代码打开一个文件输出流, 操作系统为其分配一个文件句柄, 但是文件句柄是一种非常稀缺的资源, 必须通过调用相应的close方法来被正确的释放回收. 而为了保证在异常情况下资源依然能被正确回收, 必须将其放在finally block中. 上面的代码中使用了BufferedInputStream将file stream包装成了一个buffer stream, 这样将导致在调用close方法时才会将buffer stream写入磁盘. 如果在close的时候失败, 将导致写入数据不完全. 而对于FileInputStream在finally block的close操作这里将直接忽略。

如果BufferedOutputStream.close()方法执行顺利则万事大吉, 如果失败这里有一个潜在的bug(http://bugs.sun.com/view_bug.do?bug_id=6335274): 在close方法内部调用flush操作的时候, 如果出现异常, 将直接忽略. 因此为了尽量减少数据丢失, 在执行close之前显式的调用flush操作。

下面的代码有一个小小的瑕疵: 如果分配file stream成功, 但是分配buffer stream失败(OOM这种场景), 将导致文件句柄未被正确释放. 不过这种情况一般不用担心, 因为JVM的gc将帮助我们做清理。

```
// code for your cookbook
public void save() throws IOException {
    File f = ...
    OutputStream out = new BufferedOutputStream(new FileOutputStream(f));
    try {
        out.write(...);
        out.flush(); // don't lose exception by implicit flush on close
    } finally {
        out.close();
    }
}

public void load(File f) throws IOException {
    InputStream in = new BufferedInputStream(new FileInputStream(f));
    try {
        in.read(...);
    } finally {
        try { in.close(); } catch (IOException e) { }
    }
}

```


13. 使用findbugs检查规范

    找出的bug有3种颜色：
    - 黑色的臭虫标志是分类。
    - 红色的臭虫表示严重bug，发现后必须修改代码。
    - 橘黄的臭虫表示潜在警告性bug，尽量修改。

## 工程|项目

### 项目
语法：平台-分层职责-服务名称

例子：
```
# ctxszs是平台,web是分层,course是服务名称(因为是web，所以对应的是子域名)，对应的域名是course.uxuexi.com
ctxszs-web-course
# ctxszs是平台,business是分层，sso是服务名称
uxuexi-business-sso
# ctxszs是平台,base是分层，course是服务名称
ctxszs-base-course
# we是平台(因为是通用的，没有对应的域名，所以使用we),business是分层，sso是服务名称
we-business-sso
# we是平台(因为是通用的，没有对应的域名，所以使用we),core是分层，db是服务名称
we-core-db
```
1. 平台
使用的域名去掉组织后缀。

例子：
```
域名：www.ctxszs.com
-------
子域名：www
平台名：ctxszs
组织：com
```
2. 分层职责
    - core(能力层。与具体业务无关，提供能力）
    - base(基础服务层。可以独立存在，有且只有一个具有实际意义的服务，不依赖于其他的服务）
    - business(业务服务层。依赖多个基础服务，一般是一个流程性的服务)
    - webapp(应用层。对互联网用户提供直接服务)
3. 服务名称

    如果是web项目，使用子域名当作服务名称。其他的项目，根据职责划分来自行命名。

### 分包
java源码
1. 根目录

语法：域名组织-项目名。如果web层项目，直接使用对应的域名倒置即可。

例子：
```
# web层项目，直接使用对应的域名倒置即可
项目名：ctxszs-web-course
根目录为：com.ctxszs.course
# 其他层项目，使用：域名组织+项目名
项目名：ctxszs-business-sso
根目录为：com.ctxszs.business.sso
```
2. java包

    语法：分包【+子模块】+文件（类+后缀）。其中子模块是参考业务的包才有，通用功能可以没有子模块。

    分包清单：
    - module：http路由，负责根据不同的业务跳转到不同的url。
    - form：负责http参数的封装、验证、传输。负责sql的编写
    - entity：数据库持久对象。数据的持久及对象本身业务的实现
    - dto：接口返回的实体
    - service：逻辑单元。逻辑处理或者叫计算单元
    - util：工具
    - enums：枚举
    - vt：视图标签
    - ...

    有子模块的业务包有：module,form,entity,dto,service。如果有通用的逻辑，可以使用common子模块包名

    例子：
    ```
    包名：module.student.course ，类名：StudentCourseModule
    包名：module.user,类名：UserModule
    包名：util,类名：StringUtil
    ```
### 资源文件
1. 根目录 `resources`
2. sql包

和service及form的包名一致，文件名和java调用的类名一致

例子：
```
# java
# com.ctxszs.www 根目录
# module 分包
# student.course 子模块
# StudentCourseViewService.java 文件
com.ctxszs.wwww.module.student.course.StudentCourseViewService.java

# sql
# resources 根目录
# sql 分包
# student.course 子模块
# StudentCourseViewService.sql 文件
resources.sql.student.course.StudentCourseViewService.sql

```
### 视图文件
1. 根目录 `WEB-INF`
2. 视图包

module类单词小写分隔，文件名和module里的方法名一致

例子：
```
# java
# com.ctxszs.www 根目录
# module 分包
# student.course 子模块
# StudentCourseModule 文件
# list 方法
com.ctxszs.wwww.module.student.course.StudentCourseModule.list()

# 视图
# WEB-INF 根目录
# student.course 子模块
# list.jsp 文件
WEB-INF.student.course.list.jsp
```