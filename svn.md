# 配置SVN服务端（centOS7.3）

## 1. 安装SVN

直接执行如下命令即可

```
yum install subversion
```



## 2. 配置SVN

### 2.1 创建仓库

我们在 /developer目录下新建一个名为svn-repository的仓库，以后所有代码都放在这个下面，执行如下命令

```
cd /developer
mkdir svn-repository
svnadmin create /developer/svn-repository
```

执行完以后，在svn-repository目录下多了几个文件夹

```
conf  db  format  hooks locks README.txt
```

主要是conf文件夹，这里是存放配置文件的，我们进入到conf文件夹，查看里面的文件

```
authz  passwd  svnserve.conf
```

其中：

- authz 是权限控制文件
- passwd 是账号密码文件
- svnserve.conf 是SVN服务配置文件

接下来我们依次修改这三个文件

### 2.2 配置passwd

执行vim passwd，然后添加如下配置

```
[users]
test1=123456
test2=123456
```

找到[users]，在下面添加两个用户即可，左边是用户名，右边是密码

### 2.3 配置authz

执行vim authz，然后添加如下配置

```
[/]
test1=rw
test2=r
*=
```

上述配置解释如下

```
[/]      表示仓库下所有文件
test1=rw 表示用户test1可读可写
test2=r  表示用户test2只可读
*=       表示其它用户无任何权限
```

#### 2.3.1 拓展：使用用户分组

执行vim authz，然后添加如下配置

```
[groups]
group1 = test1
group2 = test2
[/]
@group1 = rw
@group2 = r
*= 
```

上述配置解释如下

```
[groups]    	表示这里是配置分组
group1 = test1 	表示用户test1属于分组1
group2 = test2 	表示用户test2属于分组2
[/] 			表示仓库下的所有文件
@group1 = rw 	表示group1这个组拥有读写权限
@group2 = r 	表示group2这个组拥有写权限
*= 				表示其他用户无任何权限
```

### 2.4 配置svnserve.conf

执行vim svnserve.conf，然后更改如下配置

```
anon-access = read
auth-access = write
password-db = passwd
authz-db = authz
realm = /developer/svn-repository
```

上述配置默认都被注释掉了，直接打开即可，只需要修改最后一个为你的仓库地址就行了。（注意打开注释以后前面不要留空格）

以上配置解释如下

```
anon-access = read 					匿名用户可读
auth-acces = write 					授权用户可读
password-db = passwd 				使用哪个文件作为账号文件
authz-db = authz  					使用哪个文件作为权限文件
realm = /developer/svn-repository 	认证空间名，版本库所在目录
```



## 3. 配置防火墙

为了能够远程连接上SVN，还需要配置CentOS防火墙

执行如下命令查看SVN的端口是否开启，SVN默认端口是3690,

```
firewall-cmd --query-port=3690/tcp
```

如果显示no，则执行如下命令开启3690端口

```
firewall-cmd --permanent --add-port=3690/tcp
```

出现success则说明开启成功，此时再执行如下命令重启防火墙

```
firewall-cmd --reload
```

centOS上的防火墙配置完成以后，还得在阿里云控制台里面配置防火墙规则，添加3690端口

## 4. 启动与停止

执行如下命令启动

```
svnserve -d -r /developer/svn-repository
```

执行如下命令停止

```
killall svnserve
```

上述启动命令中，-d表示守护进程，-r表示在后台执行，停止还可以采用杀死进程的方式

```
ps -ef | grep svnserve
kill -9 进程号
```



## 5. 客户端连接（重要）

安装好svn客户端，一般是TortoiseSVN，执行SVN检出（英文就是SVN checkout）

一定注意这个检出的URL，它是

```
svn://外网ip
```

并不是

```
svn://外网ip/developer/svn-reposiory
```

原因是你启动的时候执行svnserve -d -r /developer/svn-repository，已经加了路径了，现在输入svn://外网ip，指的就是你服务器上那个仓库的地址了，如果你加了后缀，则会报错

```
那为什么有时候看到别人检出的时候后面带了路径呢？
那是因为别人导出以后，后来在svn里面加的
```

我们检出svn以后，可以看到有个.svn文件，我们在同级目录下创建一个doc文件夹，然后放一个txt文件进去。

然后退到svn检出的目录，执行svn提交（英文是svn commit）,此时会提示有两个新创建的文件不受控制，一个是doc文件夹，一个txt文件，全部勾选，然后点击确定，就可以将新创建的文件提交到版本库了。

好了，你想要的答案来了。

此时你换了台电脑，或者在另外一个目录检出svn项目，这个时候就可以输入svn://外网ip/doc了，表示只检出doc目录的文件
