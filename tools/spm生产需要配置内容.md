## 1.nacos 配置--已经配置

文件名 mall-data-analyse-pro.yml   :K8S-预发布环境-645d15b0/ECS-生产环境-db1e8200

```yml
dubbo:
  application:
    name: mall-data-analyse
  protocol:
    name: dubbo
    port: 20881
  registry:
    address: nacos://mse-db1e8200-nacos-ans.mse.aliyuncs.com:8848
    check: false
    timeout: 5000
  scan:
    base-packages: com.anchu.analyse.service
  config-center:
    check:
  consumer:
    check: false
    timeout: 5000
#server:
  #port: 9002 
spring:
  cloud:
    config:
      override-none: true
  datasource:
    driver-class-name: com.clickhouse.jdbc.ClickHouseDriver
    url: jdbc:clickhouse://cc-bp18392i0big3biok.public.clickhouse.ads.aliyuncs.com:8123/ck_actrade_online
    userName: clickhouse
    password: clickHouse123!@# 
    druid:
      # 按照自己连接的 clickhouse 数据库来
      driver-class-name: com.clickhouse.jdbc.ClickHouseDriver
      #filters: none
      #配置初始化大小/最小/最大
      initial-size: 5
      min-idle: 5
      max-active: 20
      #获取连接等待超时时间
      max-wait: 60000
      #间隔多久进行一次检测，检测需要关闭的空闲连接
      time-between-eviction-runs-millis: 60000
      #一个连接在池中最小生存的时间
      min-evictable-idle-time-millis: 30000
      validation-query: SELECT 1
      stat-view-servlet:
        enabled: false
  redis:
    host: r-bp1dmdey1xuav2e93f.redis.rds.aliyuncs.com
    port: 6379
    jedis:
      pool:
        max-idle: 8
        min-idle: 0
        max-active: 8
        max-wait: -1
    timeout: 2592000
    password: wsnb123!@#hzanchu0818_online

#mybatis
mybatis-plus:
  mapper-locations: classpath:mapper/*.xml
  configuration:
    log-impl: org.apache.ibatis.logging.stdout.StdOutImpl
  global-config:
    db-config:
      logic-delete-field: flag  # 全局逻辑删除的实体字段名(since 3.3.0,配置后可以忽略不配置步骤2)
      logic-delete-value: 1 # 逻辑已删除值(默认为 1)
      logic-not-delete-value: 0 # 逻辑未删除值(默认为 0)

logging:
  config: classpath:log4j2.xml
```



## 2.ECS-生产环境 bootstrap.yml--**未配置**

```yml
# nacos配置
server:
  port: 9002

spring:
  application:
    name: mall-data-analyse
  profiles:
    active: pro
  cloud:
    config:
      override-none: true
      allow-override: true
      override-system-properties: false
    nacos:
      discovery:
        server-addr: mse-db1e8200-nacos-ans.mse.aliyuncs.com #Nacos服务注册中心地址
      config:
        server-addr: mse-db1e8200-nacos-ans.mse.aliyuncs.com #Nacos作为配置中心地址
        file-extension: yml #指定yaml格式的配置
        group: DEFAULT_GROUP
        namespace: 6e8a6c48-60fa-4bd5-9091-692c062472c6
```



## 3.k8s生产环境 bootstrap.yml----**未配置**

```
# nacos配置
# nacos配置
server:
  port: 9002

spring:
  application:
    name: mall-data-analyse
  profiles:
    active: pro
  cloud:
    config:
      override-none: true
      allow-override: true
      override-system-properties: false
    nacos:
      discovery:
        server-addr: 9a0fdf7e-5208-4588-9f05-56684e8ab2a7 #Nacos服务注册中心地址
      config:
        server-addr: 9a0fdf7e-5208-4588-9f05-56684e8ab2a7 #Nacos作为配置中心地址
        file-extension: yml #指定yaml格式的配置
        group: DEFAULT_GROUP
        namespace: 9a0fdf7e-5208-4588-9f05-56684e8ab2a7
```



## 4.机器-待确认





## 5.域名-未配置

data-analyse.hzanchu.com



## 6.nginx--未配置

域名：data-analyse.hzanchu.com

端口：9002

参考测试环境配置

```conf
upstream  ac-data-analyse-test  {
      server   172.16.16.41:9002  weight=1;
      #server   172.16.16.33:8089  weight=1;
  }




server {
      listen       80;
      listen  443 ssl;
      server_name data-analyse-test.hzanchu.com;
      
      include /home/admin/nginx/conf/balck_ip.conf;    

        ssl_certificate  /home/admin/nginx/conf/7434588__hzanchu.com.pem;
        ssl_certificate_key  /home/admin/nginx/conf/7434588__hzanchu.com.key;
        ssl_session_timeout 5m;
        ssl_protocols TLSv1 TLSv1.1 TLSv1.2;
        ssl_ciphers ECDHE-RSA-AES128-GCM-SHA256:ECDHE:ECDH:AES:HIGH:!NULL:!aNULL:!MD5:!ADH:!RC4;
        ssl_prefer_server_ciphers on;

      location / {
            proxy_pass_header token;
            proxy_pass_header User-Agent;
            proxy_connect_timeout 3;
            proxy_read_timeout 5;
            proxy_send_timeout 5;
            proxy_pass_header User-Real-IP;

	    allow 122.224.206.178;
	    deny all;

            proxy_set_header X-Real-IP $remote_addr;
            proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
            client_max_body_size    1000m;
            client_body_timeout     10s;
            proxy_pass http://ac-data-analyse-test;
      }
  }

```

## 7.配置打包部署脚本（ecs jenkins  或者docker file）--未配置