# 面试

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