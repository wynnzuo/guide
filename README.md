# Linux 下 各种软件应用 或 服务安装配置手册
v0.1

此Guide为工作中，对于各种技术实际的运用形成文档记录。包含安装，配置，运维等方面。对于后期的运营维护，以及新同事加入带来极大的好处。
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

