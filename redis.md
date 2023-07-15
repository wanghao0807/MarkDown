# Redis集群整合SpringBoot

## 1.搭建redis集群

### （1）集群原理：

在redis集群中，所有的Redis节点彼此互联，节点内部使用二进制协议优化传输速度和带宽。当一个节点挂掉后，集群中超过半数的节点检测失效时才认为该节点已失效。不同于Tomcat集群需要使用反向代理服务器，Redis集群中的任意节点都可以直接和Java客户端连接。Redis集群上的数据分配则是采用哈希槽（HASH SLOT）,Redis集群中内置了16384个哈希槽，当有数据需要存储时，Redis会首先使用CRC16算法对key进行计算，将计算获得的结果对16384取余，这样每一个key都会对应一个取值在0~16383之间的哈希槽，Redis则根据这个余数将该条数据存储到对应的Redis节点上，开发者可根据每个Redis实例的性能来调整每个Redis实例上哈希槽的分布范围。

### （2）集群规划：

```
本案例在同一台服务器上用不同的端口表示不同的Redis服务器（伪分布式集群）
主节点：公网ip:8001, 公网ip:8002, 公网ip:8003 
从节点：公网ip:8004, 公网ip:8005, 公网ip:8006
```



### （3）集群配置(环境：阿里云CentOS 7.3)

Redis集群管理工具redis-trib.rb依赖Ruby环境，首先需要安装Ruby环境，由于Centos 7.X yum库中默认的Ruby版本较低，因此建议采用如下步骤进行安装首先安装RVM,RVM是一个命令行工具，可以提供一个便捷的多版本Ruby环境的管理和切换，安装命令如下

```
gpg --keyserver hkp://keys.gnupg.net --recv-keys 409B6B1796C275462A1703113804BB82D39DC0E3
```

```
rl -L get.rvm.io | bash -s stable
```

```
source /usr/local/rvm/scripts/rvm
```

执行最后一个命令使RVM生效，但是注意第一个命令最后的公钥，可能会失效，如果第一句没有成功的话，请百度查找原因（只需要找到能用的公钥即可）上述命令成功执行完以后，执行

```
rvm list know
```

```
如果显示出ruby的各个版本，则说明配置成功，选择最新的稳定版进行安装，执行
```

```
rvm install 2.5.1
```

```
安装redis依赖，执行
```

```
gem install redis
```

安装成功以后在redisCluster文件夹(自己在CentOS系统中创建一个合适的文件夹，便于寻找，如：在根目录下创建一个developer的目录，然后在这个目录下执行命令)，执行

```
mkdir redisCluster
```

```
进入到redisCluster目录，执行
```

```
cd redisCluster
```

```
下载redis安装包，执行
```

```
wget http://download.redis.io/releases/redis-4.0.10.tar.gz
```

```
解压到当前文件夹，执行
```

```
tar -zxvf redis-4.0.10.tar.gz
```

```
进入到redis-4.0.10目录，执行
```

```
cd redis-4.0.10
```

```
安装redis，执行
```

```
make MALLOC=libc
```

```
make install
```

```
安装成功后，将redis-4.0.10/src 目录下的redis-trib.rb文件复制到redisCluster目录下，执行：
```

```
cp -f ./redis-4.0.10/src/redis-trib.rb ./
```

然后在redisCluster目录下创建6个文件夹，分别命名8001,8002,8003,8004,8005,8006，再将redis-4.0.10目录下的redis.conf文件分别往这6个目录复制一份，然后对每个目录中的redis.conf文件进行修改，以8001目录下的
redis.conf文件为例，主要修改如下配置：

```
# bind 127.0.0.1
port 8001
cluster-enabled yes
cluster-config-file nodes-8001.conf
protected no
daemonize yes
requirepass 123@456
masterauth
```

说明：为了方便，可以使用Xshell配合Xftp修改（可以把文件直接弄到windows上操作），找到配置文件中的bind 127.0.0.1,注释掉，然后将其他配置粘贴进去，同时开启查找替换，因为有的配置可能在文件中默认有，找到以后注释掉就行，改完以后。同时复制八份，每一份修改下端口和cluster-config-file的配置就行。以上配置说明：

```
# bind 127.0.0.1 表示允许连接该Redis实例的地址，默认情况下只允许本地连接，将默认配置注释掉，外网就可以连接redis了
port 8001 表示端口为8001
cluster-enabled yes 表示开启集群
cluster-config-file nodes-8001.conf 表示集群节点的配置文件
protected no 表示关闭保护模式
daemonize yes 表示允许Redis在后台启动
requirepass 表示登录该Redis实例所需的密码
masterauth 由于每个节点都开启了密码认证，因此又增加了masterauth配置，使得从机可以登录到主机上
```

```
然后进入到redis-4.0.10目录下，分别启动6个Redis实例，执行
```

```
1 redis-server ../8001/redis.conf
2 redis-server ../8002/redis.conf
3 redis-server ../8003/redis.conf
4 redis-server ../8004/redis.conf
5 redis-server ../8005/redis.conf
6 redis-server ../8006/redis.conf
```

这里是重点,有可能启动不成功，包括执行下述命令时都会碰到这种情况，那就是防火墙端口问题，而且关闭防火墙也不一定能解决。

```
还是按照正常的开启防火墙，并且开启以上所有端口，执行
```

```
firewall-cmd --state 查询防火墙是否开启
```

```
如果提示not running，则表示防火墙未开启，running则表示开启
```

```
如果未开启，我们将其开启 service firewalld start
```

```
然后查看8001端口是否开放 firewall-cmd --query-port=8001/tcp     
```

```
如果提示no说明未开放，输入以下命令开放8001端口 firewall-cmd --permanent --add-port=8001/tcp  
```

```
然后重启防火墙，改变配置以后一定要重启防火墙 firewall-cmd --reload
```

```
按照以上操作开启所需的防火墙，再执行命令，可能还是会有问题，因为还要在阿里云控制台里面开启对应的防火墙规则。
```

当6个Redis实例都启动成功后，回到redisCluster目录下，首先对redis-trib.rb文件进行修改，由于配置了密码登录，而该命令在执行时默认没有密码，因此将登录不上各个Redis实例，此时用vi编辑器打开redis-trib.rb文件,搜索到如下一行

```
@r = Redis.new(:host => @info[:host], :port => @info[:port], :timeout => 60)
```

```
修改这一行，添加密码参数
```

```
@r = Redis.new(:host => @info[:host], :port => @info[:port], :timeout => 60, :password=>"123@456")
```

```
123@456就是各个redis实例的登录密码,这些都配置完成以后，接下来就可以创建Redis集群了。
```

###   

### （4）创建集群     

```
 执行如下命令创建Redis集群
```

```
  ./redis-trib.rb create --replicas 1 外网ip:8001 外网ip:8002 外网ip:8003 外网ip:8004 外网ip:8005 外网ip:8006        
```

可能会出现卡在wait for the cluster to join.......,那是因为redis集群不仅需要开通redis客户端连接的端口，而且需要开通集群总线端口，集群总线端口为redis客户端连接的端口 + 10000,然后按照上述方式分别开通对应的端口即可。再重新执行该命令，也不一定成功，可能此时会报 in `call': ERR Slot 8579 is already busy (Redis::CommandError)这种错误,这种错误意思思slot插槽被占用了，因为刚刚执行上述命令的时候，可能会创建一个配置信息和数据。解决方案是 用redis-cli 登录到每个节点执行flushall和cluster reset就可以了。再重新执行创建节点的命令，应该能成功了。其中，replicas表示每个主节点的slave数量，在集群的创建过程中会分配主机和从机，每个集群在创建过程中都将分配到一个唯一的id并分配到一段slot。当集群创建成功以后，进入到redis-4.0.10目录中，登录任意Redis实例，执行   

```
redis-cli -p 8001 -a 123@456 -c  
```

   -p表示要登录的集群的端口，-a表示要登录的集群的密码，-c则表示以集群的方式登录。登录成功以后，通过cluster info命令,查询集群状态信息，通过cluster nodes命令查询集群节点信息      



### （5）添加主节点   

当集群创建成功以后，随着业务的增长，有可能需要添加主节点，添加主节点需要先构建主节点实例，将redisCluster目录下的8001目录再复制一份，名为8007，根据第三步修改redis.conf文件的方式再修改8007下的redis.conf文件即可。修改完成后，在redis-4.0.10目录下运行如下命令启动该节点 

```
redis-server ../8007/redis.conf   
```

```
启动成功后，进入redisCluster目录下，执行如下命令将该节点添加到集群中
```

```
 ./redis-trib.rb add-node 外网ip:8007 外网ip:8001
```

中间的参数是要添加的Redis实例地址，最后的参数是集群中的实例，添加成功后，登录任意一个Redis实例，查看集群节点信息就可以看到该实例已经被添加进集群了,但是由于slot已经被之前的实例分配完了，新添加的实例没有slot，也就意味着新添加的实例没有存储数据的机会，此时需要从另外三个实例中拿出一部分slot分配给新实例，具体操作如下,首先，在redisCluster目录下执行如下命令对slot重新分配：

```
./redis-trib.rb reshard 外网ip:8001  
```

第二个参数表示连接集群中的任意一个实例，在执行命令的过程中，有三个核心配置需要手动配置,第一个是问要拿出多少个slot分配给新实例，可以输入1000,第二个是问拿出来的实例分配给谁 找到8007那个实例id即可,第三个是问这1000个slot由哪个实例出，例如从端口为8001的实例中拿出1000个slot分配给端口为8007的实例，那么这里输入8001的id后按回车键，在输入done按回车键即可。如果想要将1000个slot均摊到原有的所有实例中，那么这里输入all按回车键即可.slot分配成功后，再查看节点信息，就可以看到新实例也有slot了



### （6）添加从节点

上面添加的节点是主节点，从节点的添加相对要容易一些。步骤如下,首先将redisCluster目录下的8001目录复制一份，命名为8008，然后按照第三步中的方式配置redis.conf，修改完成后，启动该实例，然后输入如下命令启动该节点

```
./redis-trib.rb add-node --slave --master-id 节点id 外网ip:8008 外网ip:8001   
```

```
--master-id 表示该从节点的master的id
外网ip:8008 表示从节点的地址
外网ip:8001 表示集群中任意一个实例的地址
当从节点添加成功后，登录集群中任意一个Redis实例，通过cluster nodes命令就可以看到从节点信息
```

​        

### （7）删除节点             

    如果删除的是一个从节点，直接运行如下命令即可删除
```
./redis-trib.rb del-node 外网ip:8001 从节点id
```

中间的实例地址表示集群中的任意一个实例，最后的参数表示要删除的节点id，但若删除的节点占有slot，则会删除失败，此时按照第五步提到的方法，先将要删除节点的slot全部都分配出去，然后运行如上命令就可以成功删除一个占有slot的节点了。


​       
