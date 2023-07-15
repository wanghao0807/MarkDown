# 单机版redis

## 1. 下载redis（centOS7.3）

首先执行如下命令下载Redis：

```
wget http://download.redis.io/releases/redis-4.0.10.tar.gz
```

若提示未找到wget，则先执行如下命令按照wget，再下载redis

```
yum install wget
```



## 2. 安装redis

首先解压下载的文件，然后进入到redis-4.0.10目录进行编译，执行如下四条命令

```
tar -zxvf redis-4.0.10.tar.gz
```

```
cd redis-4.0.10
```

```
make MALLOC=libc
```

```
make install
```

若在执行make MALLOC=libc命令时提示“gcc: 未找到命令”，则先安装gcc，命令如下

```
yum install gcc
```

安装成功以后再进行编译安装



## 3. 配置redis

Redis安装成功后，接下来进行配置，打开Redis，修改redis.conf文件，主要修改如下几个地方：

```
daemonize yes
#bind 127.0.0.1
requirepass 123@456
protected-mode no
```

配置解释：

- 第一行配置表示允许Redis在后台启动
- 第二行配置表示允许连接该Redis实例的地址，默认情况下只允许本地连接，将默认配置注释掉，外网就可以连接redis了
- 第三行配置表示登录该Redis实例所需的密码
- 由于有了第三行配置的密码登录，因此第四行就可以关闭保护模式了。



## 4. 配置防火墙

为了能够远程连接上Redis，还需要配置CentOS防火墙

执行如下命令查看Redis的端口是否开启，Redis默认端口是6379,

```
firewall-cmd --query-port=6379/tcp
```

如果显示no，则执行如下命令开启6379端口

```
firewall-cmd --permanent --add-port=6379/tcp
```

出现success则说明开启成功，此时再执行如下命令重启防火墙

```
firewall-cmd --reload
```

centOS上的防火墙配置完成以后，还得在阿里云控制台里面配置防火墙规则，添加6379端口



## 5. Redis启动与关闭

最后，执行如下命令启动Redis：

```
redis-server redis.conf
```

Redis启动成功后，再执行如下命令进入Redis控制台，其中-a表示Redis登录密码：

```
redis-cli -a 123@456
```

进入控制台后执行ping命令，如果能看到PONG，表示redis安装成功

如果想关闭Redis实例，可以在控制台执行SHUTDOWN，然后使用exit退出，或者直接执行如下命令

```
redis-cli -p 6379 -a 123@456 shutdown
```

其中，-p表示要关闭的Redis实例的端口号，-a表示Redis登录密码
