# 关于使用libpq代替VIP建立连接问题

## 1. 认识libpq

### 1.1 概念

libpq 是 PostgreSQL的C语言应用程序的接口。libpq 是一套允许客户程序向PostgreSQL 后端服务进程发送查询 并且获得查询返回的库．libpq 同时也是其他几个 PostgreSQL 应用接口下面的引擎， 包括libpq++ （C++），libpgtcl（Tcl），Perl，和ecpg。



注意事项：

1. 在C语言程序中，需要包含<libpq-fe.h>头文件，并必须在编译时添加相应链接标记：-lpq。

2. 在C++语言程序中，有两套头文件及其库函数，分别是早期的<libpq++.h>和<pqxx/pqxx>，两者库函数完全不同。其中<libpq++.h>为更早期的，网上示例较多。

### 1.2 使用

连接数据库是最基本的操作之一，PostgreSQL libpq支持URI的连接模式，格式如下：

```
postgresql://[user[:password]@][netloc][:port][,...][/dbname][?param1=value1&...]  
```

 例子

```
postgresql://  
postgresql://localhost  
postgresql://localhost:5433  
postgresql://localhost/mydb  
postgresql://user@localhost  
postgresql://user:secret@localhost  
postgresql://other@localhost/otherdb?connect_timeout=10&application_name=myapp  
postgresql://host1:123,host2:456/somedb?target_session_attrs=any&application_name=myapp  
postgresql:///mydb?host=localhost&port=5433  
postgresql://[2001:db8::1234]/database  
postgresql:///dbname?host=/var/lib/postgresql  
postgresql://%2Fvar%2Flib%2Fpostgresql/dbname  
```

 从10开始，支持连接多个主机

```
postgresql://host1:port1,host2:port2,host3:port3/  
```

 出了使用URI的写法，还支持这种格式

```
host=localhost port=5432 dbname=mydb connect_timeout=10  
```

 连接时，支持设定一些连接参数，例如application_name，target_session_attrs等等。还有一些数据库client参数也可以通过options这个参数传入（例如timezone），在建立连接后自动设置。

 URI中支持的parameter详见：

 https://www.postgresql.org/docs/10/static/libpq-connect.html

[使用参考]: https://blog.csdn.net/yunqishequ1/article/details/78018730

### 1.3  数据库连接控制函数-PQconnectdbParams

​        下面是**libpq - C 库数据库连接控制函数 的函数处理与PostgreSQL服务器联接**的事情。 一个应用程序一次可以与多个服务器建立联接。 （这么做的原因之一是访问多于一个数据库。） 每个连接都是用一个从函数 `PQconnectdb`、`PQconnectdbParams`或`PQsetdbLogin` 获得的`PGconn`对象表示。

PQconnectdbParams与数据库服务器建立一个新的连接。

```
PGconn *PQconnectdbParams(const char * const *keywords,
                          const char * const *values,
                          int expand_dbname);
```

这个函数用从两个`NULL`结束的数组中来的参数打开一个新的数据库连接。 第一个，`keywords`，定义为一个字符串的数组，每个都成为一个关键字。 第二个，`values`，给每个关键字一个值。

当`expand_dbname`是非零的时，**允许将`dbname` 的关键字值看做一个连接字符串**。可能的格式的更详细信息在**标题2.5**中显示。

传入的参数可以为空，表明使用所有缺省的参数，或者可以包含一个或更多个参数设置。 它们的长度应该匹配。处理将会在`keywords`数组的最后一个非 `NULL`元素停止。

如果没有指定任何参数，则使用对应的环境变量（参阅[第 31.14 节](http://www.postgres.cn/docs/9.3/libpq-envars.html)）。 如果环境变量也没有设置，则使用表示的内建缺省。

通常，关键字是从这些数组的开始以索引的顺序处理的。这样的影响是，当关键字重复时， 获得最后处理的值。因此，通过小心的放置`dbname`关键字， 有可能决定哪个被`conninfo`字符串覆盖，哪个不被覆盖。

#### 2.4.1   连接字符串

几个libpq函数分析用户指定的字符串以获取连接参数。 这些字符串有两个可接受的格式：纯`keyword = value`字符串和 [RFC 3986](http://www.ietf.org/rfc/rfc3986.txt) URIs。

##### 2.4.1.1 关键字/值连接字符串

在第一中格式中，每个参数以`keyword = value`的形式设置。 等号周围的空白是可选的。要写一个空值或者一个包含空白的值，你可以用一对单引号包围它们， 例如，`keyword = 'a value'`。数值内部的单引号和反斜杠必须用一个反斜杠转义， 比如，`\'`或`\\`。

示例：

```
host=localhost port=5432 dbname=mydb connect_timeout=10
```

##### 2.4.1.2. 连接URI

连接URI的通用格式是：

```
postgresql://[user[:password]@][netloc][:port][/dbname][?param1=value1&...]   
```

URI模式标识符可以是`postgresql://`或 `postgres://`。URI的每个部分都是可选的。 下列示例举例说明了有效的URI语法使用：

```
postgresql://
postgresql://localhost
postgresql://localhost:5433
postgresql://localhost/mydb
postgresql://user@localhost
postgresql://user:secret@localhost
postgresql://other@localhost/otherdb?connect_timeout=10&application_name=myapp
```

URI的层次部分的组件也可以作为参数给出。例如：

```
postgresql:///mydb?host=localhost&port=5433
```

百分号可以用在URI的任何部分来包含特殊含义的符号。

忽略任何不对应于在[第 31.1.2 节](http://www.postgres.cn/docs/9.3/libpq-connect.html#LIBPQ-PARAMKEYWORDS)列出的关键字的连接参数， 并将关于它们的警告消息发送到`stderr`。

为了提高JDBC连接URI的兼容性，参数`ssl=true` 的实例被翻译成`sslmode=require`。

主机部分是主机名或者IP地址。要指定一个IPv6主机地址，将它包含在方括号中：

```
postgresql://[2001:db8::1234]/database   
```

主机部分解释为参数[host](http://www.postgres.cn/docs/9.3/libpq-connect.html#LIBPQ-CONNECT-HOST)的描述。特别的， 如果主机部分为空或者以斜线开头，那么选择一个Unix域套接字连接，否则初始化一个TCP/IP连接。 不过要注意，斜线是URI分层部分的一个保留字符。所以，要指定一个非标准Unix域套接字路径， 要么在URI中省略主机声明并指定主机为一个参数，要么在URI的主机部分添加百分号：

```
postgresql:///dbname?host=/var/lib/postgresql
postgresql://%2Fvar%2Flib%2Fpostgresql/dbname
```

[^参考文档]: [PostgreSQL 9.3.1 中文手册-章 31. libpq - C 库-数据库连接控制函数](http://www.postgres.cn/docs/9.3/libpq-connect.html#LIBPQ-CONNSTRING)



## 2.测试几种工具对libpq连接模式的支持



```shell
#搭建测试主备库
#服务器192.73.0.254
#切换到postgres用户空间
sudo su - postgres

#设置pg bin目录环境变量
export PATH=$PATH:/usr/pgsql-12/bin

#创建monitor节点
pg_autoctl create monitor --hostname 192.73.0.254 --auth trust --ssl-self-signed --pgdata ~/paf/monitor --pgport 6000 --run&

#配置ip，用户访问权限
echo "listen_addresses = '*'" >> ~/paf/monitor/postgresql.conf
echo "host    all             all             192.73.0.1/23           trust" >>  ~/paf/monitor/pg_hba.conf

#重启monitor

pg_autoctl stop --pgdata ./paf/monitor/
pg_autoctl run --pgdata ./paf/monitor/ &

#服务器192.73.0.254
initdb -W -D ~/paf/master
initdb -W -D ~/paf/worker1

echo "shared_preload_libraries = 'citus'" >> ~/paf/master/postgresql.conf
echo "shared_preload_libraries = 'citus'" >> ~/paf/worker1/postgresql.conf

echo "listen_addresses = '*'" >> ~/paf/master/postgresql.conf
echo "listen_addresses = '*'" >> ~/paf/worker1/postgresql.conf

echo "host    all             all             192.73.0.1/23           trust" >>  ~/paf/master/pg_hba.conf
echo "host    all             all             192.73.0.1/23           trust" >>  ~/paf/worker1/pg_hba.conf

pg_autoctl create formation --formation a --pgdata ~/paf/monitor --kind pgsql &

#master节点主库 5000端口
pg_autoctl create postgres --hostname 192.73.0.254 --name master --auth trust --ssl-self-signed --monitor 'postgres://autoctl_node@192.73.0.254:6000/pg_auto_failover?sslmode=require' --formation a --pgdata ~/paf/master --pgport 5000 --run &

#worker1 节点主库 5001端口 --模拟会出故障，并重启的机器，使用默认formation

pg_autoctl create postgres --hostname 192.73.0.254 --name worker1 --auth trust --ssl-self-signed --monitor 'postgres://autoctl_node@192.73.0.254:6000/pg_auto_failover?sslmode=require'  --pgdata ~/paf/worker1 --pgport 5001 --run &


#服务器192.73.0.254
psql -p 5000 -h 192.73.0.254 -c "CREATE EXTENSION citus;"
psql -p 5001 -h 192.73.0.254 -c "CREATE EXTENSION citus;"


#master主库设置分片数为4
psql -p 5000 -h 192.73.0.254 -c "alter system  set citus.shard_count =4;"
psql -p 5000 -h 192.73.0.254 -c "select pg_reload_conf();"

#master节点备库 5000端口
pg_autoctl create postgres --hostname 192.73.0.254 --name masterBackup --auth trust --ssl-self-signed --monitor 'postgres://autoctl_node@192.73.0.254:6000/pg_auto_failover?sslmode=require' --formation a --pgdata ~/paf/masterBackup --pgport 5002 --run &

#worker1 节点备库 5001端口 --模拟会出故障，并重启的机器，使用默认formation
pg_autoctl create postgres --hostname 192.73.0.254 --name worker1Backup --auth trust --ssl-self-signed --monitor 'postgres://autoctl_node@192.73.0.254:6000/pg_auto_failover?sslmode=require'  --pgdata ~/paf/worker1Backup --pgport 5003 --run &

#查看当前集群及分组uri信息
pg_autoctl show uri --pgdata ~/paf/monitor/
```

![image-20201016180546626](C:\Users\Lenovo\AppData\Roaming\Typora\typora-user-images\image-20201016180546626.png)



### 2.1 pgbench工具测试

```shell
#服务器 192.73.0.254

pgbench -i -s 10 -d "postgres://192.73.0.254:5001,192.73.0.254:5003/postgres?target_session_attrs=read-write&sslmode=require" -n -q

pgbench -c 32 -T 100 -P 5  "postgres://192.73.0.254:5001,192.73.0.254:5003/postgres?target_session_attrs=read-write&sslmode=require"
```

![image-20201016180709930](C:\Users\Lenovo\AppData\Roaming\Typora\typora-user-images\image-20201016180709930.png)

![image-20201016180833405](C:\Users\Lenovo\AppData\Roaming\Typora\typora-user-images\image-20201016180833405.png)

### 2.2 psql工具测试

```sql
#服务器 192.73.0.254

psql -d "postgres://192.73.0.254:5001,192.73.0.254:5003/postgres?target_session_attrs=read-write&sslmode=require" -c "select count(*) from pgbench_accounts;"
```

![image-20201016180927663](C:\Users\Lenovo\AppData\Roaming\Typora\typora-user-images\image-20201016180927663.png)

### 2.3 citus工具测试

```shell

SELECT * from master_add_node('postgres://192.73.0.254:5001,192.73.0.254:5003/postgres?target_session_attrs=read-write&sslmode=require', 5001);

SELECT * from master_add_node('192.73.0.254,192.73.0.254', 5001,5002);
```

![image-20201016181618943](C:\Users\Lenovo\AppData\Roaming\Typora\typora-user-images\image-20201016181618943.png)

**citus不支持libpq**





### 2.4 源码分析



#### **2.4.1 uxsql (等同于psql)工具源码：**

使用脚本传入**-d**  参数时，会将**-d**参数后面的赋值放入options.dbname，放到keywords参数数组，即上文提到的**连接字符串，**支持格式**参照标题2.4.1**，然后使用**PQconnectdbParam方法**创建连接，所以此工具**支持**libpq代替VIP建立连接

![image-20201016170619521](C:\Users\Lenovo\AppData\Roaming\Typora\typora-user-images\image-20201016170619521.png)

#### 2.4.2 uxbench(等同于pgbench)工具源码：

方式同上，使用脚本传入**-d**  连接url参数，即上文提到的**连接字符串**，然后使用**PQconnectdbParam方法**创建连接，所以此工具**支持**libpq代替VIP建立连接

![image-20201016171720524](C:\Users\Lenovo\AppData\Roaming\Typora\typora-user-images\image-20201016171720524.png)



#### 2.4.2 uxmpp（等同于citus）工具源码：

uxmpp添加worker时，使用的是SELECT * from master_add_node(hostname, post);的方式添加的，

对于**连接字符串**这种添加方式，从源码来看，是不支持的。所以此工具**不支持**libpq代替VIP建立连接

![image-20201016173910999](C:\Users\Lenovo\AppData\Roaming\Typora\typora-user-images\image-20201016173910999.png)



## 3. 总结

 由于源码设计上的问题，pgbench，psql(即uxbench,uxsql)**支持**libpq代替VIP建立连接，citus(即uxmpp)**不支持**libpq代替VIP建立连接。