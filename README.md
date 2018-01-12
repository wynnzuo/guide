# Linux 下 各种软件应用 或 服务安装配置手册
v0.1

此Guide为工作中，对于各种技术实际的运用形成文档记录。包含安装，配置，运维等方面。对于后期的运营维护，以及新同事加入带来极大的好处。


## 目录结构
- [Solr6.6.2集群安装配置](solr/README.md)

  - [Solr开发人员手册](solr/develop.md)
  - [Solr配置说明](solr/config.md)
  - [Solr运维说明](solr/operation.md)

- [zookepper安装配置](zookeeper.md)

- [软件开发流程规范](devManuals/README.md)
    - [前端开发规范](devManuals/web.md)
    - [数据库开发规范](devManuals/db.md)
    - [java开发规范](devManuals/java.md)
    - [接口开发规范](devManuals/interface.md)
    - [IOS开发规范](devManuals/ios.md)
    - [GIT管理规范](devManuals/git.md)

## 介绍

Most Popular Linux Install Guides

以 Ubuntu 为例：

    $ sudo aptitude install -y retext git nodejs npm
    $ sudo ln -fs /usr/bin/nodejs /usr/bin/node
    $ sudo aptitude install -y calibre fonts-arphic-gbsn00lp
    $ sudo npm install gitbook-cli -g

### 下载

    $ git clone http://42.99.16.145:19492/yaloo_yang/guide.git
    $ cd guide/

### 编译

    $ gitbook build  // 编译成网页
    $ gitbook pdf    // 编译成 pdf

### 纠错

欢迎大家指出不足，如有任何疑问，请邮件联系 yaloo_yang@189.cn 或者直接修复并提交 Pull Request。

