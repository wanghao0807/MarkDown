# Jpa实现Rest

## 1.添加依赖

参考项目

## 2.创建实体类

参考项目

## 3.创建repository

参考项目

## 4.测试

是的，只需要写完repositroy就可以测试了，不需要controller了。直接使用postman测试

```
添加功能：
URL: http://localhost:8080/books
类型：POST
头信息： Body>raw>JSON(application/json)
参数：{"name":"红楼梦","author":"曹雪芹"}
```

```
查询功能：
URL: http://localhost:8080/books
类型：GET
```

```
修改功能：
URL: http://localhost:8080/books/2
类型：PUT
头信息： Body>raw>JSON(application/json)
参数：{"name":"红楼梦1","author":"曹雪芹1"}
```

```
删除功能：
URL: http://localhost:8080/books/1
类型：DELETE
```



## 5. 自定义请求路径

默认情况下，请求路径都是实体类名小写加s，如果想自定义路径的话，在repository中添加@RepositoryRestResource注解即可实现，详情参考项目



## 6. 自定义查询方法

默认的查询方法支持分页查询，排序查询以及按照id查询，如果开发者想要按照某个属性查询，只需要在repository中定义相关方法并暴露出去即可，代码如下。

```
@RestResource(path = "author", rel = "author")
List<Book> findByAuthorContains(@Param("author") String author);

@RestResource(path = "name", rel = "name")
Book findByNameEquals(@Param("name") String name);
```

- 自定义查询只需要在repository中定义相关查询方法即可，方法定义好之后可以不添加@RestResource注解，默认路径就是方法名。以上面的第一个方法为例，若不添加@RestResource注解，则默认该方法的调用路径为

  ```
  http://localhost:8080/bs/search/findByAuthorContains?author=鲁迅
  ```

  

- 如果想对查询的路径自定义，只需要添加@RestResource注解即可，以上面的第一个方法为例，添加注解以后的查询路径为

  ```
  http://localhost:8080/bs/search/author?author=鲁迅
  ```

- 用户可以直接访问

  ```
  http://localhost:8080/bs/search
  ```

  路径查看该repository暴露出来了哪些方法

  

## 7. 隐藏方法

可添加如下注解

```
@RepositoryRestResource(exported = false)
public interface BookRepository extends JpaRepository<Book, Integer> {

    @RestResource(exported = false)
    void deleteById(Integer integer);
}


```

@RepositoryRestResource(exported = false)：表示该接口下的所有方法都会屏蔽

@RestResource(exported = false)：表示该方法会被屏蔽



## 8. 配置CORS

一般前后端分离的项目都必须支持跨域，关于跨域的配置，参考cors项目

spring:
  datasource:
    type: com.alibaba.druid.pool.DruidDataSource
    username: root
    password: 123456
    url: jdbc:mysql:///jparestful
  jpa:
    hibernate:
      ddl-auto: update
    database: mysql
    properties:
      hibernate:
        dialect: org.hibernate.dialect.MySQL57Dialect
    show-sql: true
    open-in-view: true
  data:
    rest:
      # 每页默认记录数，默认值为20
      default-page-size: 2
      # 分页查询页码参数名，默认值为page
      page-param-name: page
      # 分页查询记录数参数名，默认值为size
      limit-param-name: size
      # 分页查询排序参数名，默认值为sort
      sort-param-name: sort
      # base-path表示给所有请求路径都加上前缀
      base-path: /api
      # 添加成功时是否返回添加内容
      return-body-on-create: true
      # 更新成功时是否返回更新内容
      return-body-on-update: true
