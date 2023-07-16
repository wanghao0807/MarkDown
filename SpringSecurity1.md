# SpringSecurity实战

1. 参考项目

spring:
  datasource:
    type: com.alibaba.druid.pool.DruidDataSource
    username: root
    password: haosql
    url: jdbc:mysql:///security

mybatis:
  mapper-locations: classpath:mapper/*.xml

logging:
  level:
    com.scoundrel.springsecurity1.mapper: trace
