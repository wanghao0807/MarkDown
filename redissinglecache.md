# Redis单机缓存

1. 添加依赖（参考项目）
2. 配置yml文件（参考项目）
3. 开启缓存（在启动类添加注解）
4. 创建测试类（参考项目）
5. 配置redis（参考redis项目）

   ```
# 缓存配置
spring:
  cache:
    # 缓存名称
    cache-names: c1,c2
    redis:
      # 缓存有效期
      time-to-live: 1800s

# Redis配置
  redis:
    database: 0
    host: 119.23.110.130
    port: 6379
    password: 123@456
    jedis:
      pool:
        max-active: 8
        max-idle: 8
        max-wait: -1ms
        min-idle: 0
   ```


