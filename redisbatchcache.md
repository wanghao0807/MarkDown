# Redis集群配置缓存

1. 添加依赖（参考项目）

2. 配置缓存（参考config包）

3. 使用缓存（在启动类上添加注解）

4. 创建测试环境（dao类和测试类）

5. 搭建redis集群（参考redis集群项目）

6. 启动测试类，如果测试通过，并打印出相关日志说明配置成功

7. 在Redis服务器上查看

   ```
   # 进入redis
   redis-cli -p 8001 -a 123@456 -c
   
   # 查看key
   get sang:100
   
   # 查看过期时间
   ttl sang:100
   
   # 查看key
   get c2::99
   
   # 查看过期时间
   ttl c2::99
   ```