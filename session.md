# Session共享

正常情况下，HttpSession是通过Servlet容器创建并进行管理的，创建成功之后都是保存在内存中。如果开发者需要对项目进行横向扩展搭建集群，那么可以利用一些硬件或者软件工具来做负载均衡，此时，来自同一用户的HTTP请求就有可能被分发到不同的实例上去，如何保证各个实例之间Session的同步就成为一个必须解决的问题。Springboot提供了自动化的Session共享配置，它结合Redis可以非常方便的解决这个问题，使用Redis解决Session共享问题的原理非常简单，就是把原本存储在不同服务器上的Session拿出来放在一个独立的服务器上。

当一个请求到达Nginx服务器后，首先进行请求分发，假设请求被其中一个server处理了，这个server在处理请求时，无论是存储Session还是读取Session，都去操作Session服务器而不是操作自身内存中的Session，其他server在处理请求时也是如此，这样就可以实现session共享了。



## 1. Session共享配置

### 1.1 添加依赖

```
<dependency>    
    <groupId>org.springframework.boot</groupId>    
    <artifactId>spring-boot-starter-data-redis</artifactId>    
    <exclusions>        
        <exclusion>            
            <groupId>io.lettuce</groupId>            
            <artifactId>lettuce-core</artifactId>        
        </exclusion>    
    </exclusions>
</dependency>

<dependency>    
    <groupId>redis.clients</groupId>    
    <artifactId>jedis</artifactId>
</dependency>

<dependency>    
    <groupId>org.springframework.boot</groupId>    
    <artifactId>spring-boot-starter-web</artifactId>
</dependency>

<dependency>    
    <groupId>org.springframework.session</groupId>    
    <artifactId>spring-session-data-redis</artifactId>
</dependency>
```

除了redis依赖之外，这里还要提供spring-session-data-redis依赖，SpringSession可以做到透明化地换掉应用的Session容器；



### 1.2 配置yml

```
spring:  
  redis:    
    database: 0    
    host: 47.107.46.16    
    port: 6379    
    password: 123@456    
    jedis:      
      pool:        
        max-active: 8        
        max-idle: 8        
        max-wait: -1ms        
        min-idle: 0
```

### 1.3 创建Controller来测试

```
@RestController
public class HelloController {    
    @Value("${server.port}")    
    String port;
    
    @PostMapping("/save")    
    public String saveName(String name, HttpSession session){        				         
        session.setAttribute("name", name);        
        return port;    
    }    
    
    @GetMapping("/get")    
    public String getName(HttpSession session){        
        return port + ":" + session.getAttribute("name").toString();    
    }
}
```

这里提供了两个方法，一个save接口用来向Session中存储数据，还有一个get接口用来从Session中获取数据，这里注入了项目启动的端口号server.port，主要是为了区分到底是哪个服务器提供的服务。另外，虽然还是操作HttpSession，但是实际上HttpSession容器已经被透明替换，真正的Session此时存储在Redis服务器上。

项目创建完成以后，将项目打成jar包上传到centos上，然后执行命令启动项目

打包命令

```
mvn clean package -Dmaven.test.skip=true
```

启动命令

```
nohup java -jar session-0.0.1-SNAPSHOT.jar --server.port=8080 &
nohup java -jar session-0.0.1-SNAPSHOT.jar --server.port=8081 &
```

nohup 表示不挂断程序运行，即当终端窗口关闭后，程序依然在后台运行，最后的&表示让程序在后台运行。--server.port表示设置启动端口，一个为8080，另一个为8081。启动成功后，接下来就可以配置负载均衡器了。

注意事项：

- 打包成功以后在项目的target目录会有一个session-0.0.1-SNAPSHOT.jar文件，这就是打包好的项目jar文件，然后可以用xftp直接传到centos上即可，其实直接用git上传也行，然后用脚本执行，会更方便

- 开启服务器的端口8080，8081，6379（redis），对应阿里云控制台的端口也要开

- redis配置可参考redis项目

  

## 2. 配置nginx

### 2.1 下载并安装nginx

执行如下命令下载并解压

```
wget https://nginx.org/download/nginx-1.14.0.tar.gz
tar -zxvf nginx-1.14.0.tar.gz
```

然后进入nginx-1.14.0目录执行编译安装，代码如下

```
cd nginx-1.14.0
./configure
make
make install
```

### 2.2 启动nginx

nginx默认目录在/usr/local/nginx ，我们执行如下命令即可启动nginx

```
/usr/local/nginx/sbin/nginx
```

nginx启动成功后，默认端口是80，可以在浏览器直接访问，浏览器输入centos服务器外网ip即可，会看到一个welcome to nginx，说明nginx启动成功。

### 2.3 配置nginx

进入nginx安装目录下的conf目录，修改nginx.conf文件，代码如下

```
vim nginx.conf
```

对nginx.conf文件进行编辑，编辑内容如下

```
upstream sang.bojin.work {
	server 外网ip:8080 weight=1;
    server 外网ip:8081 weight=1;
}

server {
	listen 80;
	server_name localhost;
	location / {
		proxy_pass http://sang.bojin.work;
		proxy_redirect default;
	}
}
```

这里只列出了修改的配置，找到相应位置加上去即可。

在修改的配置中首先配置上游服务器，即两个real server，两个real server的权重都是1，意味着请求将平均分配到两个real server上，然后在server中配置拦截规则，将拦截到的请求转发到定义好的real server上。

配置完成后，重启nginx，重启命令如下：

```
/usr/local/nginx/sbin/nginx -s reload
```



## 3. 测试请求转发

当项目的两个端口（8080,8081，可以看成两个server）都启动后，调用"/save"接口存储数据，使用postman测试

地址栏输入

```
http://外网ip:80/save?name=江南一点雨
```

调用的端口是80，即调用的是nginx服务器，请求会被转发到real server上进行处理，返回值为8080，说明真正处理请求的real server是8080那台服务器，接下来调用get接口获取数据

地址栏输入

```
http://外网ip:80/get
```

调用端口依然是80，但是返回值是8081，说明是8081那台real server提供的服务，如果这里不是8081，再访问一次即可。

经过以上步骤，就完成了利用Redis实现Session共享的功能，基本上不需要额外配置，开箱即用。

spring:
  redis:
    database: 0
    host: 47.107.46.16
    port: 6379
    password: 123@456
    jedis:
      pool:
        max-active: 8
        max-idle: 8
        max-wait: -1ms
        min-idle: 0
