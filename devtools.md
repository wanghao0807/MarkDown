# devtools

1. 添加依赖

   ```
   <!-- optional配置为true，是为了防止将devtools依赖传递到其他模块中，当把应用打包后，devtools会被自动禁用 -->
   <dependency>    
   	<groupId>org.springframework.boot</groupId>    
   	<artifactId>spring-boot-devtools</artifactId>    
   	<optional>true</optional>
   </dependency>
   ```

2. 配置IDEA

   ```
   Compiler下的
   勾选Build project automatically
   
   配置Registry，高级设置里面
   勾选compiler.automake.allow.when.app.running
   ```



## 注意事项

classpath路径下的静态资源或者视图模板等发生变化时，并不会导致项目重启



## 使用LiveReload

谷歌商店搜索LiveReload安装，当静态资源发生改变时，不用手动刷新，浏览器会自动刷新页面



## 全局配置

如果项目模块众多，开发者可以在当前用户目录下创建.spring-boot-devtools.properties文件来对devtools进行全局配置，这个配置文件适用于当前计算机上任何使用了devtools模块的SpringBoot项目，比如在D:\java\devloper 目录下创建.spring-boot-devtools.properties文件，内容如下

```
spring.devtools.restart.trigger-file=.trigger-file
```

