logging.level.org.springframework.web=trace
#management.endpoint.shutdown.enabled=true

#management.endpoints.enabled-by-default=false
#management.endpoint.info.enabled=true

#management.endpoints.web.exposure.include=mappings,metrics
#management.endpoints.web.exposure.include=*

spring.security.user.name=sang
spring.security.user.password=123
spring.security.user.roles=ADMIN

management.endpoint.httptrace.cache.time-to-live=100s

#management.endpoints.web.base-path=/
#management.endpoints.web.path-mapping.health=healthcheck

# 跨域：允许端点处理来自http://localhost:8081地址的请求，允许请求的方法为GET和POST
management.endpoints.web.cors.allowed-origins=http://localhost:8081
management.endpoints.web.cors.allowed-methods=GET,POST

management.endpoints.web.exposure.include=*
management.endpoint.health.show-details=always
management.endpoint.health.roles=ADMIN

management.health.status.order=FATAL,DOWN,OUT_OF_SERVICE,UP,UNKNOWN
management.health.status.http-mapping.FATAL=503

#info.app.encoding=@project.build.sourceEncoding@
#info.app.java.source=@java.version@
#info.app.java.target=@java.version@
#info.author.name=wanghao
#info.author.email=wangsong0210@gmail.com

management.info.git.mode=full

server.port=8081
spring.boot.admin.client.url=http://localhost:8080
