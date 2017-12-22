# 前端开发规范

 不涉及太深层的性能优化。其次，它指的只是个人所认为的前端开发规范化和专业化路上的第一步，主要分为HTML开发规范、CSS开发规范和Javascript开发规范三个模块。


## HTML规范

 - 页面的第一行添加标准模式声明 <!DOCTYPE html>
 - 代码缩进：tab键设置四个空格（通常在软件右下角设置相应空格大小，有些IDE已自动设置好。）
 - html中除了开头的DOC和 'UTF-8'或者head里特殊情况可以大写外，其他都为小写，css类都为小写。
 - 为 html 根元素指定 lang 属性，用来定义当前文档显示的语言类型，例如 lang="zh-CN"、lang="en"，利于搜索引擎、屏幕阅读器等对页面的语言类型准确识别。
 - 设置针对IE8浏览器的兼容设置， <meta http-equiv="X-UA-Compatible" content="IE=Edge">（用来指定IE8浏览器去模拟某个特定版本的IE浏览器的渲染方式）； <meta http-equiv="X-UA-Compatible" content="IE=Edge,chrome=1" />添加”chrome=1“将允许站点在使用了谷歌浏览器内嵌框架（Chrome Frame）的客户端渲染，对于没有使用的，则没有任何影响。
 - 引入css文件使用<link>标签而不用@import（指令慢，@import多用于css文件内引入。）
 - 在HTML5规范下，在引入CSS和JS时不需要指明 type（text/css 和 text/javascript 分别是他们的默认值）
 - 注释全局统一（"<! -- 注释内容--!>" --后不要留空格，若留空格，其他注释全加空格，方便后续程序开发）
 - 命名规范：class用 “-” ；ID用 “_” ；name:data-自定义名；
 根据内容书写语义化编码：可用以项目名简写开头-语义化名称；
 **class="tb-head"; id="tb_head"; data-head="tb-head";**
### 属性排序（方便整理代码)
 1. 根据属性定义排序 : class,id,name排序 （建议用此方法）
 (class用于标识高度可复用组件，因此排首位，id用于标识具体组件，慎用，因此排第二位);
 2. 特殊属性可根据元素跟随排序，后根据（1）方法排序;
 3. 根据字母开头顺序排序;
 ```
 <a class="..." id="..." data-modal="toggle" href="#">
   Example link
 </a>
 ```
 4. 减少标签数量（DOM元素）
 5. 在HTML5规范下，布尔型属性可以在声明时不赋值 **<input type="checkbox" value="1" checked>**

## CSS规范
###  缩进
 使用 4 个空格做为一个缩进层级，不允许使用 2 个空格 或 tab 字符。
 ### 添加空格
 投入生产前，可以通过某些工具压缩css文件，去掉空格
 1. 每个声明块的左花括号前添加一个空格；
 2. 每条声明语句的" : "后应该插一个空格；
 3. 属性多值每个逗号后应该插入空格 eg:box-shadow,border-image；
 4. 不要在rgb()、rgba()、hsl()、hsla() 或 rect() 值的内部的逗号后面插入空格；
 5. 对于属性值或颜色参数，省略小于 1 的小数前面的 0 （例如，.5 代替 0.5；-.5px 代,替 -0.5px）
 6. \>、+、~ 选择器的两边各保留一个空格。
 ```
 .content > .content-header > .title {
     font-size: 2em;
 }
 ```
### 选择器规范
 1. 当构建选择器时应该使用清晰，准确和有语义的 class 名。尽量减少使用标签选择器；
 ```
 不推荐：
 div.content > header.content-header > h2.title {
     font-size: 2em;
 }

  推荐：
 .content > .content-header > .title {
     font-size: 2em;
 }
 ```
 2. 使用提供的简写属性（font、background等等）
 3. 为选择器中的属性添加双引号 不允许单引号
     - 例如，input[type="text"]。只有在某些情况下是可选的，但是，为了代码的一致性，建议都加上双引号。
 4. 选择器的嵌套层级应不大于 3 级，位置靠后的限定条件应尽可能精确，避免使用不必要的嵌套，只有在必须将样式限制在父元素内（也就是后代选择器），并且存在多个需要嵌套的元素时才使用嵌套。
 ```
 // Without nesting
 .table > thead > tr > th { … }
 .table > thead > tr > td { … }

 // With nesting
 .table > thead > tr {
   > th { … }
   > td { … }
 }
 ```
 5. 为选择器分组时，将单独的选择器单独存放在一行 。
 ```
 .selector,
 .selector-second,
 .selector[type="text"] {
     padding: 15px;
     margin-bottom: 15px;
     background-color: rgba(0, 0, 0, 0.5);
     box-shadow: 0 1px 2px #ccc, inset 0 1px 0 #fff;
 }
 ```
### 简写

 十六进制值应该全部小写,尽可能简写 eg:#fff；.避免为 0 值指定单位；例如，用 margin: 0; 代替 margin: 0px;
 同时，应当尽量限制使用简写形式的属性声明，过度使用简写形式的属性声明会导致代码混乱，并且会对属性值带来不必要的覆盖从而引起意外的副作用。常见的滥用简写属性声明如下几个：

 ```
 margin: 0 0 10px;
 可以用
 margin-bottom: 10px;
 background: red;
 可以用
 background-color: red;
 background: url("image.jpg");
 可以用
 background-image: url("image.jpg");
 ```

### 行规范
 1.  每行不得超过 120 个字符，除非单行不可分割。
 2.  声明块右花括号应当成行。
 3.  对于超长的样式，在样式值的 空格 处或 , 后换行，建议按逻辑分组。

### 声明顺序
 1.  . 相关的属性声明应当归为一组，并按照下面的顺序排列：
 Positioning、Box model、Typographic、Visual
 由于定位（positioning）可以从正常的文档流中移除元素，并且还能覆盖盒模型（box model）相关的样式，因此排在首位。盒模型排在第二位，因为它决定了组件的尺寸和位置。其他属性只是影响组件的内部（inside）或者是不影响前两组属性，因此排在后面。
 ```
 .declaration-order {
   /* Positioning */
   position: absolute;
   top: 0;
   right: 0;
   bottom: 0;
   left: 0;
   z-index: 100;

   /* Box-model */  display: block;
   float: right;
   width: 100px;
   height: 100px;

   /* Typography */
   font: normal 13px "Helvetica Neue", sans-serif;
   line-height: 1.5;
   color: #333;
   text-align: center;

   /* Visual */
   background-color: #f5f5f5;
   border: 1px solid #e5e5e5;
   border-radius: 3px;

   /* Misc */
   opacity: 1;
 }
 ```

### 带前缀的属性:

 当使用特定厂商的带有前缀的属性时，通过缩进的方式，让每个属性的值在垂直方向对齐，这样便于多行编辑。
 ```
 -webit-box-show: 0 1px 2px rgba(0,0,0,.15);
        box-show: 0 1px 2px rgba(0,0,0,.15);
 ```

## Javascript规范

### 命名
####  变量命名
 变量命名使用有意义的单词和驼峰式命名。<br/>
 临时变量简写：str,num,bol,fun,arr。<br/>
 循环变量简写：i , j , k。<br/>
 全局变量使用g作为前缀，如gUserName , gLoginTime。<br/>
 常量全部字母都大写，如： PI ,<br/> COPYRIGHT。注:常量可存在于函数中，也可存在于全局。

#### 函数（方法）命名
 统一使用动词或动词加名词的形式。<br/>如getVersion() , submitForm()。
 涉及返回逻辑值的函数可以使用is , has 等表示逻辑的词语代替动词。<br/> 内部函数前加上 ‘‘前缀。<br/>
 可选参数以 ‘opt’ 开头.
#### 类（对象）命名

 类名首字母大写。<br/>
 属性名为具有一定意义的名词。私有属性加 ““。<br/>
 方法名为有意义的动词[+名词]，首字母小写。私有方法加 ‘_”。

### 命名空间
 为避免全局命名的冲突,在全局作用域上使用一个与项目或文件相关的名字来划分作用域空间.（类似于java中的封装）。
 ```
 var calcultor = {};
 calcultor.add = function() {
 ...
 };
 calcultor.sub = function() {
 ...
 };

 ```

### 传递的类型

 原始值：按值传递 string/number/boolean/null/undefined 注:null和undefine是不同的.

 复杂类型：按引用传递 object/array/function.

### 对象规范
 1. 使用字面值创建对象。
 ```
 /*bad*/
 var item = new Object();
 /*good*/
 var item = {};
 ```
 2. 不要使用保留字作为键。
 ```
 /*bad*/
 var superman = {
 class:'superhero',
  default:{ clark:'kent' },
 private: true
 }
 /*good*/
 var superman = {
 klass: 'superhero',
 defaults:{ clark: 'kent' },
 hidden: true
 };

 ```
 3. 当使用变量访问对象中的属性时使用中括号（俗称方括号）。
 ```
 var luke = {
     jedi: true,
     age: 28
 };
 function getProp(prop){
     return luke[prop];
 }
 var isJedi = getProp('jedi');

 ```
### 数组规范
 1. 使用字面量创建数组。
 ```
 /*bad*/
 var items = new Array();
 /*good*/
 var items = [];
 ```
 2. 数组长度不知时，使用push添加元素。
 ```
 var someStack = [];
 /*bad*/
 someStack[someStack.length] = 'abcdefg';
 /*good*/
 someStack.push('abcdefg');
 ```
 3. 使用slice()方法拷贝数组。
 ```
 var len = items.length,
      itemsCopy = [],
      i;
 /*bad*/
 for(i = 0; i < len; i++){
     itemsCopy[i] = items[i];
 }
 /*good*/
 itemsCopy = items.slice();
 ```

### 函数规范
 1. 不要在一个非函数块里面声明一个函数，应该把那个函数赋给一个变 量。浏览器让你这么做，但是解析的情况是不同的。
 ```
 /*bad*/
 if(currentUser){
     function test(){
         console.log('Nope.');
     }
 }
 /*good*/
 if(currentUser){
     var test = function test(){
         console.log('Yup.');
     };
 }
 ```
 2. 请勿把参数命名为 arguments,这会对函数内的 arguments对象产生影响。
 ```
 /*bad*/
 function nope(name, options, arguments){

 }
 /*good*/
 function yup(name, options, args){

 }
 ```

### 条件表达式的强类型转换规则
 对象被计算为true<br/>
 Undefined被计算为false<br/>
 Null被计算为false<br/>
 布尔值被计算为布尔的值<br/>
 数字如果是+0, -0,NaN被计算为false<br/>
 字符如果是空字符串，则被计算为false，否则为true<br/>
 字符串如果是空字符串，则被计算为false，否则为true
### 事件
 当给事件附加数据时，传入一个哈希而不是原始值，这可以让后面的贡献者加入更多数据到事件数据 里而不用找出并更新那个事件的事件处理器。
 ```
 /*bad*/
 $(this).trigger('listingUpdated', listing.id);
 $(this).on('listingUpdated', function(e,listingId){
     /*do something with listingId*/
 });
 /*good*/
 $(this).trigger('listingUpdated', {listingId: listing.id});
 $(this).on('listingUpdated', function(e, data){
    /*do something with data.listingId*/
 });

 ```