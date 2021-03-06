# 接口规范

本规范适用范围为：内部系统


~~Api主要是基于http(s) + OAuth2.0协议实现。~~

Api主要是基于http(s)协议实现。
使用json 格式进行报文传输。


## 接口名规范
1. 风格统一（都为英文小写）
2. 模式固定（xx.xx.xx.xx)
3. 简洁明了
4. 有扩展性
5. 分类与分层

举例：ctxszs.custom.order.get

格式：ctxszs.业务分类.模块简称.操作符

## 文档规范化
每个接口的文档规范，都应包含下面的内部
1. 简介
2. 接口描述
3. 公共参数
4. 请求参数
5. 响应参数
6. 请求示例
7. 响应示例
8. 异常示例
9. 业务异常码
    - 参数必填
    - 参数的可选项值错误
    - 参数格式错误
    - 查询没结果
    - 服务不可用
    - ...
10. 公共异常码
    - Api方法不存在
    - Api方法已过期
    - 无权操作此Api
    - 请求报文格式错误
    - 请求开始时间错误
    - 业务数据查询成功
    - 业务数据查询为空
    - pageNo格式错误
    - pageSize格式错误
    - pageNo,pageSize不在要求范围内
    - 不存在开始时间（节点）
    - 不存在结果时间（节点）
    - 时间不合法，格式yyyy-MM-dd HH:mm:ss
    - 网络异常
    - 访问限制
    - header参数为空
    - ...

## 异常码标准化

异常码的设计问题

异常码常见的问题包括：无统一风格、无分类、无唯一性、难以理解等。基于以上问题， API 网关在异常码方面进行一些标准化设计，使异常码变得有如下特征。

- 唯一性
- 风格一致
- 有层级有分类
- 可扩展

异常码的分类
1. 系统级异常码
    - 流控类
    - 权限类
    - 系统校验类
    - 服务不可用
2. 业务级异常码
    - 业务字段
        - 非空
        - 格式
        - 枚举
    - 业务场景
        - 单场景
        - 组合场景

## 安全性

1．接口安全：目前一般都是在APP客户端和服务器通过约定的算法，对传递的参数值进行验证匹配。但是如果APP程序被反编译，这些约定的算法就会暴露，特别是在安卓APP中，有了算法，完全就可以通过验证模拟接口请求。

2．加密规范：在传递用户名密码时，应采用规范的加密算法如MD5、RSA、DES，进行数据通信请求。

3．接口版本控制：对于接口版本控制，需要应对不断的APP版本升级，新、旧接口的处理，因而需要关注接口版本控制。

4．时间戳：接口请求和响应都加时间戳，通过对时间戳的验证，可以一定程度上防止重放攻击。

### 实现细则
名词解释：

客户端：api的调用方

服务端：提供api服务

客户端与服务端的通信就需要2个token。
1. 第一个token是针对接口的`api_token`
2. 第二个token是针对用户的`user_token`

#### api_token

它的职责是保持接口访问的隐蔽性和有效性，保证接口只能给自家人用。

接口token生成规则参考如下：

`api_token = md5 ('业务分类' + '模块简称' + '操作符' + 调用日期（天） + '加密密钥')`

其中的
1.  *'调用日期（天）'* 为当天时间，格式yyyy-MM-dd
2. *'加密密钥'* 为私有的加密密钥，客户端需要在服务端注册一个“接口使用者”账号后，系统会分配一个账号及密码，数据表设计参考如下：

字段名 | 字段类型 | 注释
---|---|---
client_id | varchar(20) |客户端ID
client_secret | varchar(20)| 客户端(加密)密钥

> 此数据可以放在redis,并且数据策略为：**永不过期**

#### user_token

其职责是保护用户的用户名及密码多次提交，以防密码泄露。

如果接口需要用户登录，其访问流程如下：
1. 用户提交“用户名”和“密码”，实现登录（条件允许，这一步最好走https）；
2. 登录成功后，服务端返回一个 user_token，生成规则参考如下：<br/>

    `user_token = md5('用户的uid' + 'Unix时间戳') `<br/>

服务端用数据表维护user_token的状态，表设计如下：

字段名 | 字段类型 | 注释
---|---|---
user_id| int| 用户ID
user_token| varchar(36)| 用户token
expire_time| int |过期时间

> 此数据可以放在redis,并且数据策略为：**过期时间为 expire_time**

服务端生成 user_token 后，返回给客户端（自己存储），客户端每次接口请求时，如果接口需要用户登录才能访问，则需要把 user_id 与 user_token 传回给服务端，服务端接受到这2个参数后，需要做以下几步：
1. 检测 api_token的有效性；
2. 删除过期的 user_token 表记录；(redis自动删除)
3. 根据 user_id，user_token 获取记录，如果记录不存在，直接返回错误，如果记录存在，则进行下一步；

4. ~~更新 user_token的过期时间（延期，保证其有效期内连续操作不掉线）；~~ 过期后，客户端调用登录接口注册user_token
5. 返回接口数据；