# vsftp安装配置

## 前言
正如vsftpd官方宣传中所说 Probably the most secure and fastest FTP server for UNIX-like systems.
我相信这是大多数人选择vsftpd来搭建Linux的FTP服务器的原因，当然ProFTPD用的人应该也不在少数。
文章将以清晰直观的方式介绍安装vsftpd以及配置FTP虚拟用户的过程，希望对大家有帮助。


**扩展阅读**

ProFTPD - http://www.proftpd.org/
vsftpd - http://vsftpd.beasts.org

## 安装vsftpd

### 查看当前系统版本

```
$cat /etc/redhat-release
CentOS release 6.5 (Final)

```

### 查看是否已经安装vsftpd

```
$rpm -qa | grep vsftpd
#如果没有，就安装，并设置开机启动
$yum -y install vsftpd
$chkconfig vsftpd on

```

## 基于虚拟用户的配置
所谓虚拟用户就是没有使用真实的帐户，只是通过映射到真实帐户和设置权限的目的。
虚拟用户不能登录CentOS系统。

### 配置修改

```
vi /etc/vsftpd/vsftpd.conf

#服务器独立运行
listen=YES
#设定不允许匿名访问
anonymous_enable=NO
#设定本地用户可以访问。注：如使用虚拟宿主用户，在该项目设定为NO的情况下所有虚拟用户将无法访问
local_enable=YES
#使用户不能离开主目录
chroot_list_enable=YES
#设定支持ASCII模式的上传和下载功能
ascii_upload_enable=YES
ascii_download_enable=YES
#PAM认证文件名。PAM将根据/etc/pam.d/vsftpd进行认证
pam_service_name=vsftpd
#设定启用虚拟用户功能
guest_enable=YES
#指定虚拟用户的宿主用户，CentOS中已经有内置的ftp用户了
guest_username=ftp
#设定虚拟用户个人vsftp的CentOS FTP服务文件存放路径。存放虚拟用户个性的CentOS FTP服务文件(配置文件名=虚拟用户名)
user_config_dir=/etc/vsftpd/vuser_conf
#配置vsftpd日志（可选）
xferlog_enable=YES
xferlog_std_format=YES
xferlog_file=/var/log/xferlog
dual_log_enable=YES
vsftpd_log_file=/var/log/vsftpd.log
file_open_mode=0755
anon_other_write_enable=YES

```
> 目录下一定要有一个空文件chroot_list

### 进行认证

#### 安装Berkeley DB工具
很多人找不到db_load的问题就是没有安装这个包
```
yum install db4 db4-utils
```
#### 创建用户密码文本
注意奇行是用户名，偶行是密码
```
vi /etc/vsftpd/vuser_passwd.txt

test
test@123#qwe
```

#### 生成虚拟用户认证的db文件

```
db_load -T -t hash -f /etc/vsftpd/vuser_passwd.txt /etc/vsftpd/vuser_passwd.db
```

#### 编辑认证文件
编辑认证文件，全部注释掉原来语句，再增加以下两句
```
vi /etc/pam.d/vsftpd

auth required pam_userdb.so db=/etc/vsftpd/vuser_passwd
account required pam_userdb.so db=/etc/vsftpd/vuser_passwd

```
#### 配置虚拟用户

##### 创建虚拟用户配置文件
```
mkdir /etc/vsftpd/vuser_conf/
#文件名等于vuser_passwd.txt里面的账户名，否则下面设置无效

vi /etc/vsftpd/vuser_conf/test
```

##### 修改虚拟用户配置
```
vi /etc/vsftpd/vuser_conf/myapp

#虚拟用户根目录,根据实际情况修改（#设置登录后禁锢的目录）
local_root=/data03/FTP_DATA/test
#开放写权限
write_enable=YES
anon_umask=022
#开放下载权限
anon_world_readable_only=NO
#开放上传权限
anon_upload_enable=YES
#开放创建目录的权限
anon_mkdir_write_enable=YES
#开放删除和重命名的权限
anon_other_write_enable=YES

```

### 设置FTP根目录权限

最新的vsftpd要求对主目录不能有写的权限所以ftp为755，主目录下面的子目录再设置777权限

#### 目录权限设置

```
mkdir -p /data03/FTP_DATA/
chown -R ftp:ftp /data03/FTP_DATA/
chmod -R 755 /data03/FTP_DATA/
chmod -R 777 /data04/FTP_DATA/test

```
#### 建立限制用户访问目录的空文件
```
touch /etc/vsftpd/chroot_list
```

#### 建立日志文件
如果启用vsftpd日志需手动建立日志文件
```
touch /var/log/xferlog
touch /var/log/vsftpd.log
```

### 配置PASV模式（可选）
vsftpd默认没有开启PASV模式，现在FTP只能通过PORT模式连接，
要开启PASV默认需要通过下面的配置

```
vim /etc/vsftpd/vsftpd.conf


#开启PASV模式
pasv_enable=YES
#最小端口号
pasv_min_port=40000
#最大端口号
pasv_max_port=40080
pasv_promiscuous=YES

```
#### 在防火墙配置内开启40000到40080端口
```
-A INPUT -m state --state NEW -m tcp -p -dport 40000:40080 -j ACCEPT
```

#### 重启iptabls和vsftpd
```
service iptables restart
service vsftpd restart

```
现在可以使用PASV模式连接你的FTP服务器了~
### Selinux和防火墙
该关闭的关闭，该放行的放行

service vsftpd start



## 常见问题

如果登录时出现

500 OOPS: priv_sock_get_result. Connection closed by remote host.

这样的错误，需要升级pam
