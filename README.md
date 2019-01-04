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

<pre>
Sharding-JDBC 与 Mycat 有一定的相似性，区别点在于对于 SQL 语句的自解析上，是否可以这么理解？

      从设计理念上看确实有一定的相似性，主要流程都是SQL解析--> SQL改写-->SQL路由-->SQL
      执行-->结果归并，但架构设计时不同的，Mycat是基于Proxy，它复写了Mysql协议，将Mycat
      Server伪装成一个Mysql Server数据库，而Sharding-JDBC是基于JDBC接口的扩展，是以
      Jar包的形式提供轻量级服务的。

      SQL解析这块，现在的Sharding-JDBC和Mycat也比较相似，都是使用Druid作为SQL解析的基础
      类库，但Sharding-JDBC正在重写SQL解析这块，是去掉Druid的完全自研版本，不可否认，
      Druid是一个优秀的连接池，而且SQL解析这块做的也很强，但它毕竟不是一个专门为了Sharding
      而做的SQL解析器，它的大致解析流程是 Lexer -> Parser -> AST -> Visitor，使用者需要
      实现它的Visitor接口，将自己的业务逻辑在Visitor中实现.
</pre>

<pre>
Sharding-JDBC 从 1.3.0 开始支持读写分离

分库分表使用 like 查询，是否能查询出来？性能如何？会去查询所有的库和表吗？
      1)分库分表使用 like 查询是有限制的。目前 Shariding-JDBC 不支持 like 语句中包含分片
        键，但不包含分片键的 like 语句可以正确执行。
      2)至于 like 性能问题，是与数据库相关的，Shariding-JDBC 仅仅是解析 SQL 以及路由至正
        确的数据源而已。
      3)是否会查询所有的库和表是根据分片键决定的，如果 SQL 中不包括分片键，就会查询所有库和
        表，这个和是否有 like 没有关系。
</pre>

<pre>
有一个问题一直很疑惑，目前分库分表的中间件有两种思想，分别是：

     1)类似 Sharding-JDBC 及 TDDL 的增强版 JDBC 驱动思想
     2)类似 Mycat 增加中间层，然后在中间层进行分库分表思想
     我想问的是，这两种思想都有什么优势和劣势呢，大公司的主流选型又是哪种？

     两种方式各有优缺点：
         JDBC驱动版的优点：
             1）轻量，范围更加容易界定，只是JDBC增强，不包括HA,事务以及数据库元数据管理
             2）开发的工作量较小，无需关注NIO，各个数据库协议等。
             3）运维无需改动，无需关注中间件本身的HA
             5）性能高，JDBC直连数据库，无需二次转发
             6）可支持各种基于JDBC协议的数据库，如Mysql, Oracle, SqlServer
         
         Proxy的优点：
             1）可以负责更多的内容，将数据迁移，分布式事务等纳入Proxy的防臭
             2）更有效的管理数据库的连接
             3）整合大数据思路，将OLTP和OLAP分离处理。

      因此两种方式互相可以互补，建议使用 Java 的团队，且仅 OLTP 的互联网前端操作。有可能会
      使用多种数据库的情况，可以选择 JDBC 层的中间件；如果需要 OLAP 和 OLTP 混合，加以重
      量级的操作，如数据迁移，分布式事务等，可以考虑 Proxy 层的中间件
</pre>
