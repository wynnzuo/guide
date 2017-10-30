# 环境准备
- 3台服务器（10.128.91.191，10.128.91.192，10.128.91.193）
- 安装包 zookeeper-3.4.9.tar.gz
- jdk1.7及以上版本

# 安装zookeeper
- 下载
```
wget -c http://mirror.bit.edu.cn/apache/zookeeper/zookeeper-3.4.9/zookeeper-3.4.9.tar.gz
```
- 解压
```
tar zxf zookeeper-3.4.9.tar.gz
```
- 修改配置
将${zookeeper.home}/conf/zoo_sample.cfg文件拷贝一份，命名为zoo.cfg
修改zoo.cfg内容为:

```shell
# The number of milliseconds of each tick
tickTime=2000
# The number of ticks that the initial 
# synchronization phase can take
initLimit=10
# The number of ticks that can pass between 
# sending a request and getting an acknowledgement
syncLimit=5
# the directory where the snapshot is stored.
# do not use /tmp for storage, /tmp here is just 
# example sakes.
dataDir=/app/zookeeperdir/zookeeper-data
dataLogDir=/app/zookeeperdir/logs

# the port at which the clients will connect
clientPort=2181
#
# Be sure to read the maintenance section of the 
# administrator guide before turning on autopurge.
#
# http://zookeeper.apache.org/doc/current/zookeeperAdmin.html#sc_maintenance
#
# The number of snapshots to retain in dataDir
#autopurge.snapRetainCount=3
# Purge task interval in hours
# Set to "0" to disable auto purge feature
#autopurge.purgeInterval=1

# 2888,3888 are election port
server.1=10.128.91.191:2888:3888
server.2=10.128.91.192:2888:3888
server.3=10.128.91.193:2888:3888
```
- 创建myid文件
进入配置文件中的**dataDir**目录，创建myid文件。
在myid文件中插入配置文件中最后部分主机的server后面的数字
```
10.128.91.191 的myid中插入1
10.128.91.192 的myid中插入2
10.128.91.193 的myid中插入3
```
- 在/etc/profile文件中设置PATH
```
vim /etc/profile
###### 添加如下内容
export ZOOKEEPER_HOME=/app/zookeeper-3.4.9
PATH=$ZOOKEEPER_HOME/bin:$PATH
export PATH
```
# 启动并测试zookeeper
- 在所有服务器中执行
```
/app/zookeeper-3.4.9/bin/zkServer.sh start
```
- 验证
输入jsp命令查看进程，其中，QuorumPeerMain是zookeeper进程，启动正常。

# 常用命令
```
#启动进程
/app/zookeeper-3.4.9/bin/zkServer.sh start
#查看状态
/app/zookeeper-3.4.9/bin/zkServer.sh status
#停止进程
/app/zookeeper-3.4.9/bin/zkServer.sh stop
#启动客户端脚本
/app/zookeeper-3.4.9/bin/zkCli.sh -server 10.128.91.191:2181
```