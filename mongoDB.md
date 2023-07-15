Springboot整合MongoDB
1.MongoDB简介
MongoDB是一种面向文档的数据库管理系统，它是一个介于关系型数据库和非关系型数据库之间的产品，MondoDB功能丰富，它支持一种类似 JSON 的 BSON数据格式，既可以存储简单的数据格式，也可以存储复杂的数据类型。MongoDB最大的特点是它支持的查询语言非常强大，并且还支持对数据建立索引。总体来说，MongoDB是一款应用相当广泛的NoSQL数据库。

2.MongoDB安装
2.1 下载MongoDB
执行如下命令下载MongoDB

wget https://fastdl.mongodb.org/linux/mongodb-linux-x86_64-4.0.0.tgz
下载完成后，将MongoDB解压，并将解压后的文件夹重命名为mongodb，执行如下命令

tar -zxvf mongodb-linux-x86_64-4.0.0.tgz
mv mongodb-linux-x86_64-4.0.0 mongodb
2.2 配置MongoDB
进入mongodb目录下，创建两个文件夹db和logs，分别用来保存数据和日志，代码如下：

cd mongodb
mkdir db
mkdir logs
然后进入bin目录下，创建一个mongo.conf的文件，用来存放配置信息，执行命令如下：

cd bin
touch mongo.conf
vim mongo.conf

要添加的内容
dbpath=/你的路径/mongodb/db
logpath=/你的路径/mongodb/logs/mongodb.log
port=27017
fork=true
然后保存文件即可。

2.3 MongoDB的启动和关闭
配置完成后，还是在bin目录下，运行如下命令启动MongoDB：

./mongod -f mongo.conf --bind_ip_all
-f表示指定配置文件的位置，--bind_ip_all则表示允许所有的远程地址连接该MongoDB实例，MongoDB启动成功后，在bin目录下执行 ./mongo 命令，进入MongoDB控制台，然后输入db.version()，如果能看到MongoDB的版本号，就表示安装成功。

默认情况下，启动后连接的是MongoDB中的test库，而关闭MongoDB的命令需要在admin库中执行，因此关闭MongoDB需要首先切换到admin库，然后执行db.shutdownServer();命令，具体操作命令如下

use admin
db.shutdownServer();
exit
服务关闭后，执行exit命令退出控制台，此时如果再执行./mongo命令就会执行失败。

2.4 安全管理
默认情况下，启动的MongoDB没有登录密码，在生产环境中这是非常不安全的，但是不同于MySQL，Oracle等关系型数据库，MongoDB中每一个库都有独立的密码，在哪一个库中创建用户就要在哪一个库中验证密码，要配置密码，首先要创建一个用户。

我们登录进去以后，使用show dbs可以查看当前有哪些库，默认有admin，config，local。此时发现并没有test库，而我们使用db查看当前库的时候，发现当前库是test，原因应该是Mongodb隐藏了test库（未证明），不管怎样，我们操作下test库，在test库下创建用户，执行如下命令

use test (如果当前在test下就不用执行这个，查看当前库用db命令即可)
db.createUser({user:"sang",pwd:"123",roles:["readWrite"]})
新创建的用户名为sang，密码是123，roles表示该用户具有的角色，这里表示该用户对test库具有读和写两项权限。

在test库下执行如下代码，测试是否认证成功

db.auth("sang","123")
如果执行结果为1，就表示认证成功，说明该用户可以对test库进行读写操作。

3.整合Springboot
3.1 添加依赖
<dependency>    
<groupId>org.springframework.boot</groupId>    
<artifactId>spring-boot-starter-data-mongodb</artifactId>
</dependency>
3.2 配置yml文件
spring:  
data:    
mongodb:      
uri: mongodb://sang:123@公网ip/test
3.3 使用方法
见controller中的方法

#spring:
#  data:
#    mongodb:
#      authentication-database: admin
#      database: config
#      host: 119.23.110.130
#      port: 27017
#      username: wang
#      password: 123
#      uri: mongodb://wang:123@119.23.110.130/config