# MycatTree
Mycat中间件技术研究

![](https://i.imgur.com/qfryC9m.png)


<pre>
Mycat是一个开源的分布式数据库系统，其并没有存储引擎，但是Mycat实现了mysql协议，前段用户可以把它当做一个proxy。其核心功能是分库分表，即将一个
大表分库分表为N个小表，存储在后端的Mysql存储引擎里面。最新版本的Mycat不仅支持MS SqlServer， Oracle，DB2等关系型数据库，而且还支持MongoDB
这种NoSQL。
</pre>

<pre>
Mycat的原理是先拦截用户的SQL语句做分析：分片分析，路由分析，读写分离分析，缓存分析，再将该SQL语句发送到指定规则的数据库，最终将存储引擎返回的
结果返回给用户。
</pre>
