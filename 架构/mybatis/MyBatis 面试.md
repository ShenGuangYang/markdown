# mybatis

## 缓存

### 一级缓存

​		一级缓存也叫本地缓存，`MyBatis `的一级缓存是在会话（`SqlSession`）层面进行缓存的。`MyBatis `的一级缓存是默认开启的，不需要任何的配置。

​		是放在 `DefaultSqlSession ` 的父类执行器 `BaseExecutor` 中进行维护的。在同一个会话里面，多次执行相同的`SQL `语句，会直接从内存取到缓存的结果，不会再发送`SQL `到数据库。

**一级缓存的生命周期**

1. `MyBatis`在开启一个数据库会话时，会 创建一个新的`SqlSession`对象，`SqlSession`对象中会有一个新的Executor对象，Executor对象中持有一个新的`PerpetualCache`对象；当会话结束时，`SqlSession`对象及其内部的Executor对象还有`PerpetualCache`对象也一并释放掉。
2. 如果`SqlSession`调用了`close()`方法，会释放掉一级缓存`PerpetualCache`对象，一级缓存将不可用；
3. 如果`SqlSession`调用了`clearCache()`，会清空`PerpetualCache`对象中的数据，但是该对象仍可使用；
4. `SqlSession`中执行了任何一个update操作(update()、delete()、insert()) ，都会清空`PerpetualCache`对象的数据，但是该对象可以继续使用。

**一级缓存的工作流程：**

1. 对于某个查询，根据`statementId`,`params`,`rowBounds`来构建一个`key`值，根据这个`key`值去缓存`Cache`中取出对应的`key`值存储的缓存结果
2. 判断从`Cache`中根据特定的`key`值取的数据数据是否为空，即是否命中；
3. 如果命中，则直接将缓存结果返回；
4. 如果没命中：
   1. 去数据库中查询数据，得到查询结果；
   2. 将key和查询到的结果分别作为key,value对存储到Cache中；
   3. 将查询结果返回；



**一级缓存的局限性：**

1. 一级缓存是SqlSession级别的，所以只是针对同一SqlSession生效，要是多个SqlSession，一级缓存不共享。



### 二级缓存

​		二级缓存是用来解决一级缓存不能跨会话共享的问题的，范围是`namespace `级别的，可以被多个`SqlSession `共享，生命周期和应用同步。如果你的`MyBatis`使用了二级缓存，并且你的`Mapper`和`select`语句也配置使用了二级缓存，那么在执行`select`查询的时候，`MyBatis`会先从二级缓存中取输入，其次才是一级缓存，即`MyBatis`查询数据的顺序是：二级缓存   —> 一级缓存 —> 数据库。

​		使用装饰器模式，在外层的`CachingExecutor`来维护二级缓存。



**二级缓存的生命周期**

​		启用了二级缓存，`MyBatis `在创建`Executor `对象的时候会对`Executor `进行装饰。`CachingExecutor `对于查询请求，会判断二级缓存是否有缓存结果，如果有就直接返回，如果没有委派交给真正的查询器Executor 实现类，比如`SimpleExecutor `来执行查询，再走到一级缓存的流程。最后会把结果缓存起来，并且返回给用户。



**二级缓存的更新机制**

未使用事务情况下：

更新操作(update、delete、insert)都会清空二级缓存

使用了事务下：

二级缓存使用`TransactionalCacheManager（TCM）`来管理，最后又调用了`TransactionalCache `的`getObject()`、`putObject() `和`commit()`方法。在`putObject() `的时候，只是添加到了`entriesToAddOnCommit `里面，只有它的`commit()`方法被调用的时候才会调用`flushPendingEntries()`真正写入缓存。它就是在`DefaultSqlSession `调用`commit()`的时候被调用的。

**二级缓存的局限性**

1. 因为所有的增删改都会刷新二级缓存，导致二级缓存失效，所以适合在查询为主的应用中使用，比如历史交易、历史订单的查询。否则缓存就失去了意义。
2. 如果多个`namespace `中有针对于同一个表的操作，比如`Blog `表，如果在一个`namespace `中刷新了缓存，另一个`namespace `中没有刷新，就会出现读到脏数据的情况。所以，推荐在一个`Mapper `里面只操作单表的情况使用。



## resultType 和 resultMap区别



resultType  适用简单对象，只能自动映射，适合单表简单查询。

resultMap 适合复杂对象，可知道映射关系，适合关联复合查询。



## #{}，${} 的区别

`# `会解析为PreparedStatement的参数标记符，参数用？代替，这样会经过类型检查和安全检查

`$`只会做字符串替换，不能防止sql注入

## collection 和 association的区别

collection 一对一

association 一对多，多对多



## PrepareStatement 和 Statement 的区别

两个都是接口，PrepareStatement 继承 Statement 的；

Statement 处理静态sql，PrepareStatement 执行带有参数的语句。

PrepareStatement 的addBatch() 方法一次性发送多个查询给数据库。

PrepareStatement 可以防止sql注入

mybatis 默认值是：PrepareStatement 



## mybatis 中使用的设计模式

工厂模式：SqlSessionFactory,MapperProxyFactory, ObjectFactory

建造者：XMLConfigBuilder、XMLMapperBuilder、XMLStatementBuilder

单例：SqlSessionFactory，Configuration

责任链：InterceptChain

动态代理：MapperProxy，Plugin

装饰：Cache实现

模板方法：BaseExecutor 与子类SimpleExecutor、BatchExecutot、ReuseExecutor





## mybatis 解决了什么问题、为什么要用、核心特性



1. 资源管理（数据源）
2. 结果集自动映射
3. 参数映射、动态sql
4. sql语句跟代码分离
5. 缓存、插件





## mybatis中核心对象及其作用

Configuration： 管理所有配置信息

SqlSessionFactory：创建回话

SqlSession：提供操作接口

MapperProxy：代理mapper接口，用于找到对应的sql

Executor：执行sql的具体操作者



## java 类型和数据库类型怎么实现相互映射

通过TypeHandler，例如java类型中的String 对应varchar，就会调用相应的handler。也可以自己实现



## SimpleExecutor、ReuseExecutor、BatchExecutor的区别

SimpleExecutor 使用后直接关闭Statement。

ReuseExecutor 放在缓存中，可复用。

BatchExecutor支持复用，可以批量执行



## 关联查询的延迟加载是怎么实现的？

动态代理，在创建实体类对象时进行代理，在调用代理对象的相关方法时触发二次查询。



## Mapper没有实现类，mybatis的方法时怎么执行的？

MapperProxy代理，代理中的invoke() 方法中调用了SqlSession中对象的方法



## 插件实现的原理

在被拦截对象的方法调用的时候，先走到 Plugin 的 invoke()方法，再走到
Interceptor 实现类的 intercept()方法，最后通过 Invocation.proceed()方法调用被
拦截对象的原方法



## DefaultSqlSession 和 SqlSessionTemplate 的区别是什么



SqlSessionTemplate 是线程安全的，内部通过ThreadLocal获取

DefaultSqlSession 是非线程安全的

















