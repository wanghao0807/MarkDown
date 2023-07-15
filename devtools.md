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


```
#spring:
#  devtools:
#    restart:
      # 这个配置表示从默认不触发重启的目录中除去static目录，即static目录里面的资源发生变化时，会触发重启
      #exclude: static/**
      # 这个表示直接配置需要监控自动重启的目录
      #additional-paths: src/main/resources/static
      # 这个配置表示修改代码时，项目不会自动重启，需要重启时，只需要修改这个文件里面的内容即可，需要注意的时，如果项目没有改动，只是单纯的改了这个文件，项目不会重启
      # trigger-file: .trigger-file

# 关闭LiveReload特性（建议开发时使用LiveReload特性实现静态资源的动态加载）
#spring:
#  devtools:
#    livereload:
#      enabled: false

spring:
  devtools:
    livereload:
      enabled: true
    # 禁用自动重启
    restart:
      enabled: false
```
