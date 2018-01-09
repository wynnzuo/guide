# IOS编码规范

此文档根据apple、google以及其他一些业界知名的oc编码规范整理而成，并作了大量精简，
旨在为大家的iOS开发规范提供一份简单、清晰、统一的参考指南。

## 格式和换行

- 只使用2个空格来缩进，不使用tab符
- 方法长度不超过100行，建议不超过80行
- 方法- 和 + 和返回值之前为1个空格；方法参数之间有一个空格，其他地方不出现多余的空格。
- 条件语句的格式，推荐如下:

```
if (user.isHappy)
{
  //Do something
}
else
{
  //Do something else
}
```

或:(苹果官方代码缩进方式)

```
if (user.isHappy) {
  //Do something
} else {
  //Do something else
}
```

反例：

```
if (user.isHappy)
{
    //Do something
}
else {
    //Do something else
}
```

## 命名

- 命名统一使用驼峰命名法；只采纳有广为人知含义的缩写，比如info、msg、UI、HTTP这类。
自造的缩写不被认可。总体的命名原则是清晰和一致，避免歧义。
iOS命名一般采用自注释方式，看见名称即可推测出该变量或者方法的功能，这也是iOS方法名称较长的原因。
例如： 接口请求方法requestData，切勿因为界面接口多，而将方法写成requestData1, requestData2。

- 命名类、协议、常量和typedef结构体时考虑使用前缀,主要目的是为了避免冲突。
方法名存在特定类的命名空间内，无需使用前缀。 协议一般使用”类名+Delegate”，
block命名也应保持自注释原则。定义结构体使用同一前缀，方便用户查看该结构体内所定义的内容。
例如：LoginControllerDelegate为LoginController的代理，LoginSuccessBlock为登录成功后的回调方法。
- 方法首字母小写（方法以大写缩略词开始除外）；文件夹名和类名首字母大写。
不使用下划线作为私有方法的前缀，此方式被苹果保留。
例如：只有一个简单动词是不必大写如-(void)login；
多个单词时需要首单词小写，其他单词大写：-(void)jumpLoginVc;
- 全局常量尽量不要使用宏定义。宏很可能被重定义，而且引用不同的文件可能会导致宏的不同，
所以尽量使用const来定义常量。
- 避免使用newXXX、getXXX、setXXX来命名变量和方法。
定义类属性时系统会自动生成getXXX setXXX newXXX方法，避免和系统生成的方法重名。


## oc下的cocoa编码规范

- 使用#pragma mark来分类方法，参考以下结构

```c
#pragma mark - Lifecycle

- (instancetype)init {}
- (void)dealloc {}
- (void)viewDidLoad {}
- (void)viewWillAppear:(BOOL)animated {}
- (void)didReceiveMemoryWarning {}

#pragma mark - Custom Accessors

- (void)setCustomProperty:(id)value {}
- (id)customProperty {}

#pragma mark - IBActions

- (IBAction)submitData:(id)sender {}

#pragma mark - Public

- (void)publicMethod {}

#pragma mark - Private

- (void)privateMethod {}

#pragma mark - Protocol conformance
#pragma mark - UITextFieldDelegate
#pragma mark - UITableViewDataSource
#pragma mark - UITableViewDelegate

#pragma mark - NSCopying

- (id)copyWithZone:(NSZone *)zone {}

#pragma mark - NSObject

- (NSString *)description {}
#pragma mark - Lifecycle

- (instancetype)init {}
- (void)dealloc {}
- (void)viewDidLoad {}
- (void)viewWillAppear:(BOOL)animated {}
- (void)didReceiveMemoryWarning {}

#pragma mark - Custom Accessors

- (void)setCustomProperty:(id)value {}
- (id)customProperty {}

#pragma mark - IBActions

- (IBAction)submitData:(id)sender {}

#pragma mark - Public

- (void)publicMethod {}

#pragma mark - Private

- (void)privateMethod {}

#pragma mark - Protocol conformance
#pragma mark - UITextFieldDelegate
#pragma mark - UITableViewDataSource
#pragma mark - UITableViewDelegate

#pragma mark - NSCopying

- (id)copyWithZone:(NSZone *)zone {}

#pragma mark - NSObject

- (NSString *)description {}

```

## 注释要求

注释一般用来解释代码的意图。要保持注释和代码同步更新，表达准确，避免误导。
尽可能写自注释的代码。尽量使用单行注释而不是块注释；
需要文档化的代码可以使用vvdocumenter-xcode插件来帮助生成。

## 其它

- delegate对象不应该被retain,这样做会造成retain环。
- 使用`__weak`来消除block中的retain环，例如：
- 苹果官方主要推荐的是MVC的架构，其他各种流行架构也是在MVC基础上的变种。
写代码时需要注意model/view/controller之间的分离，保持清晰的层次关系。
