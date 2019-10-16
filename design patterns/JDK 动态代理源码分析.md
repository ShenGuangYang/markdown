# JDK 动态代理源码分析



本文基于 jdk 1.8 版本

动态代理类必须需要实现InvocationHandler接口，重写invoke方法。通过`Proxy.newProxyInstance(ClassLoader loader, Class<?>[] interfaces, InvocationHandler h)` 方法产生代理对象。



```java
public static Object newProxyInstance(ClassLoader loader,
                                          Class<?>[] interfaces,
                                          InvocationHandler h)
        throws IllegalArgumentException
    // 校验 h 是否为空
    Objects.requireNonNull(h);
	// 接口的类对象拷贝一份
    final Class<?>[] intfs = interfaces.clone();
	// 进行一些安全性检查
    final SecurityManager sm = System.getSecurityManager();
    if (sm != null) {
        checkProxyAccess(Reflection.getCallerClass(), loader, intfs);
    }

    /*
    * Look up or generate the designated proxy class.
    * 查询（缓存）或生成代理类的class对象
    */
	// 生成代理类的主要实现
    Class<?> cl = getProxyClass0(loader, intfs);

    /*
    * Invoke its constructor with the designated invocation handler.
    */
    try {
        if (sm != null) {
            checkNewProxyPermission(Reflection.getCallerClass(), cl);
        }
		//获取参数类型是InvocationHandler.class的代理类构造器
        final Constructor<?> cons = cl.getConstructor(constructorParams);
        final InvocationHandler ih = h;
        //如果代理类是不可访问的, 就使用特权将它的构造器设置为可访问
        if (!Modifier.isPublic(cl.getModifiers())) {
            AccessController.doPrivileged(new PrivilegedAction<Void>() {
                public Void run() {
                    cons.setAccessible(true);
                    return null;
                }
            });
        }
        //这里生成代理对象
        return cons.newInstance(new Object[]{h});
    } catch (IllegalAccessException|InstantiationException e) {
        throw new InternalError(e.toString(), e);
    } catch (InvocationTargetException e) {
        Throwable t = e.getCause();
        if (t instanceof RuntimeException) {
            throw (RuntimeException) t;
        } else {
            throw new InternalError(t.toString(), t);
        }
    } catch (NoSuchMethodException e) {
        throw new InternalError(e.toString(), e);
    }
}
```

这段代码核心就是通过 `getProxyClass0(loader, intfs)` 得到代理类的Class对象，然后通过Class对象得到构造方法，进而创建代理对象。



```java
/**
* Generate a proxy class.  Must call the checkProxyAccess method
* to perform permission checks before calling this.
*/
private static Class<?> getProxyClass0(ClassLoader loader,
                                       Class<?>... interfaces) {
    if (interfaces.length > 65535) {
        throw new IllegalArgumentException("interface limit exceeded");
    }

    // If the proxy class defined by the given loader implementing
    // the given interfaces exists, this will simply return the cached copy;
    // otherwise, it will create the proxy class via the ProxyClassFactory
    // 意思是：如果代理类被指定的类加载器loader定义了，并实现了给定的接口interfaces，
    // 那么就返回缓存的代理类对象，否则使用ProxyClassFactory创建代理类。
    return proxyClassCache.get(loader, interfaces);
}

```

在进入get方法之前，我们看下 `proxyClassCache`是什么？高能预警，前方代码看起来可能有乱，但我们只需要关注重点即可。

```java
private static final WeakCache<ClassLoader, Class<?>[], Class<?>>
        proxyClassCache = new WeakCache<>(new KeyFactory(), new ProxyClassFactory());
```

​     `proxyClassCache`是个WeakCache类的对象，调用proxyClassCache.get(loader, interfaces); 可以得到缓存的代理类或创建代理类（没有缓存的情况）。说明WeakCache中有`get`这个方法。先看下WeakCache类的定义（这里先只给出变量的定义和构造函数）：

```java

//K代表key的类型，P代表参数的类型，V代表value的类型。
// WeakCache<ClassLoader, Class<?>[], Class<?>>  proxyClassCache  说明proxyClassCache存的值是Class<?>对象，正是我们需要的代理类对象。
final class WeakCache<K, P, V> {

    private final ReferenceQueue<K> refQueue
        = new ReferenceQueue<>();
    // the key type is Object for supporting null key
    private final ConcurrentMap<Object, ConcurrentMap<Object, Supplier<V>>> map
        = new ConcurrentHashMap<>();
    private final ConcurrentMap<Supplier<V>, Boolean> reverseMap
        = new ConcurrentHashMap<>();
    private final BiFunction<K, P, ?> subKeyFactory;
    private final BiFunction<K, P, V> valueFactory;


    public WeakCache(BiFunction<K, P, ?> subKeyFactory,
                     BiFunction<K, P, V> valueFactory) {
        this.subKeyFactory = Objects.requireNonNull(subKeyFactory);
        this.valueFactory = Objects.requireNonNull(valueFactory);
    }
}
```



其中map变量是实现缓存的核心变量，他是一个双重的Map结构: `(key, sub-key) -> value`。其中key是传进来的Classloader进行包装后的对象，sub-key是由WeakCache构造函数传人的`KeyFactory()`生成的。value就是产生代理类的对象，是由WeakCache构造函数传人的`ProxyClassFactory()`生成的。如下，回顾一下:

```java
private static final WeakCache<ClassLoader, Class<?>[], Class<?>>
        proxyClassCache = new WeakCache<>(new KeyFactory(), new ProxyClassFactory());
```

产生sub-key的KeyFactory代码如下，这个我们不去深究，只要知道他是根据传入的ClassLoader和接口类生成sub-key即可。

```java
private static final class KeyFactory
    implements BiFunction<ClassLoader, Class<?>[], Object>
{
    @Override
    public Object apply(ClassLoader classLoader, Class<?>[] interfaces) {
        switch (interfaces.length) {
            case 1: return new Key1(interfaces[0]); // the most frequent
            case 2: return new Key2(interfaces[0], interfaces[1]);
            case 0: return key0;
            default: return new KeyX(interfaces);
        }
    }
}
```





通过sub-key拿到一个`Supplier<Class<?>>`对象，然后调用这个对象的get方法，最终得到代理类的Class对象。

好，大体上说完WeakCache这个类的作用，我们回到刚才`proxyClassCache.get(loader, interfaces);`这句代码。get是WeakCache里的方法。源码如下：



```java
//K和P就是WeakCache定义中的泛型，key是类加载器，parameter是接口类数组
public V get(K key, P parameter) {
	// 校验parameter是否为空
    Objects.requireNonNull(parameter);
	// 清除无效的缓存
    expungeStaleEntries();
	// cacheKey就是(key, sub-key) -> value里的一级key
    Object cacheKey = CacheKey.valueOf(key, refQueue);

    // lazily install the 2nd level valuesMap for the particular cacheKey
    // 根据一级key得到 ConcurrentMap<Object, Supplier<V>>对象。如果之前不存在，则新建一个ConcurrentMap<Object, Supplier<V>>和cacheKey（一级key）一起放到map中。
    ConcurrentMap<Object, Supplier<V>> valuesMap = map.get(cacheKey);
    if (valuesMap == null) {
        ConcurrentMap<Object, Supplier<V>> oldValuesMap
            = map.putIfAbsent(cacheKey,
                              valuesMap = new ConcurrentHashMap<>());
        if (oldValuesMap != null) {
            valuesMap = oldValuesMap;
        }
    }

    // create subKey and retrieve the possible Supplier<V> stored by that
    // subKey from valuesMap
    // 这部分就是调用生成sub-key的代码，上面我们已经看过怎么生成的了
    Object subKey = Objects.requireNonNull(subKeyFactory.apply(key, parameter));
    //通过sub-key得到supplier
    Supplier<V> supplier = valuesMap.get(subKey);
    //supplier实际上就是这个factory
    Factory factory = null;

    while (true) {
        if (supplier != null) {
            // supplier might be a Factory or a CacheValue<V> instance
            //如果缓存里有supplier ，那就直接通过get方法，得到代理类对象，返回，就结束了
            V value = supplier.get();
            if (value != null) {
                return value;
            }
        }
        // else no supplier in cache
        // or a supplier that returned null (could be a cleared CacheValue
        // or a Factory that wasn't successful in installing the CacheValue)

        // lazily construct a Factory
        //下面的所有代码目的就是：如果缓存中没有supplier，则创建一个Factory对象，把factory对象在多线程的环境下安全的赋给supplier。
        //因为是在while（true）中，赋值成功后又回到上面去调get方法，返回才结束。
        if (factory == null) {
            factory = new Factory(key, parameter, subKey, valuesMap);
        }

        if (supplier == null) {
            supplier = valuesMap.putIfAbsent(subKey, factory);
            if (supplier == null) {
                // successfully installed Factory
                supplier = factory;
            }
            // else retry with winning supplier
        } else {
            if (valuesMap.replace(subKey, supplier, factory)) {
                // successfully replaced
                // cleared CacheEntry / unsuccessful Factory
                // with our Factory
                supplier = factory;
            } else {
                // retry with current supplier
                supplier = valuesMap.get(subKey);
            }
        }
    }
}


```



​     所以接下来我们看Factory类中的get方法。

```java
public synchronized V get() { // serialize access
    // re-check
    Supplier<V> supplier = valuesMap.get(subKey);
    //重新检查得到的supplier是不是当前对象
    if (supplier != this) {
        // something changed while we were waiting:
        // might be that we were replaced by a CacheValue
        // or were removed because of failure ->
        // return null to signal WeakCache.get() to retry
        // the loop
        return null;
    }
    // else still us (supplier == this)

    // create new value
    V value = null;
    try {
         //代理类就是在这个位置调用valueFactory生成的
         //valueFactory就是我们传入的 new ProxyClassFactory()
        //一会我们分析ProxyClassFactory()的apply方法
        value = Objects.requireNonNull(valueFactory.apply(key, parameter));
    } finally {
        if (value == null) { // remove us on failure
            valuesMap.remove(subKey, this);
        }
    }
    // the only path to reach here is with non-null value
    assert value != null;

    // wrap value with CacheValue (WeakReference)
    //把value包装成弱引用
    CacheValue<V> cacheValue = new CacheValue<>(value);

    // put into reverseMap
    // reverseMap是用来实现缓存的有效性
    reverseMap.put(cacheValue, Boolean.TRUE);

    // try replacing us with CacheValue (this should always succeed)
    if (!valuesMap.replace(subKey, this, cacheValue)) {
        throw new AssertionError("Should not reach here");
    }

    // successfully replaced us with new CacheValue -> return the value
    // wrapped by it
    return value;
}
```



来到ProxyClassFactory的apply方法，代理类就是在这里生成的。

```java
//这里的BiFunction<T, U, R>是个函数式接口，可以理解为用T，U两种类型做参数，得到R类型的返回值
private static final class ProxyClassFactory
        implements BiFunction<ClassLoader, Class<?>[], Class<?>>
{
    // prefix for all proxy class names
    //所有代理类名字的前缀
    private static final String proxyClassNamePrefix = "$Proxy";
    
    // next number to use for generation of unique proxy class names
    //用于生成代理类名字的计数器
    private static final AtomicLong nextUniqueNumber = new AtomicLong();

    @Override
    public Class<?> apply(ClassLoader loader, Class<?>[] interfaces) {
          
        Map<Class<?>, Boolean> interfaceSet = new IdentityHashMap<>(interfaces.length);
        //验证代理接口，可不看
        for (Class<?> intf : interfaces) {
            /*
             * Verify that the class loader resolves the name of this
             * interface to the same Class object.
             */
            Class<?> interfaceClass = null;
            try {
                interfaceClass = Class.forName(intf.getName(), false, loader);
            } catch (ClassNotFoundException e) {
            }
            if (interfaceClass != intf) {
                throw new IllegalArgumentException(
                    intf + " is not visible from class loader");
            }
            /*
             * Verify that the Class object actually represents an
             * interface.
             */
            if (!interfaceClass.isInterface()) {
                throw new IllegalArgumentException(
                    interfaceClass.getName() + " is not an interface");
            }
            /*
             * Verify that this interface is not a duplicate.
             */
            if (interfaceSet.put(interfaceClass, Boolean.TRUE) != null) {
                throw new IllegalArgumentException(
                    "repeated interface: " + interfaceClass.getName());
            }
        }
        //生成的代理类的包名 
        String proxyPkg = null;     // package to define proxy class in
        //代理类访问控制符: public ,final
        int accessFlags = Modifier.PUBLIC | Modifier.FINAL;

        /*
         * Record the package of a non-public proxy interface so that the
         * proxy class will be defined in the same package.  Verify that
         * all non-public proxy interfaces are in the same package.
         */
        //验证所有非公共的接口在同一个包内；公共的就无需处理
        //生成包名和类名的逻辑，包名默认是com.sun.proxy，类名默认是$Proxy 加上一个自增的整数值
        //如果被代理类是 non-public proxy interface ，则用和被代理类接口一样的包名
        for (Class<?> intf : interfaces) {
            int flags = intf.getModifiers();
            if (!Modifier.isPublic(flags)) {
                accessFlags = Modifier.FINAL;
                String name = intf.getName();
                int n = name.lastIndexOf('.');
                String pkg = ((n == -1) ? "" : name.substring(0, n + 1));
                if (proxyPkg == null) {
                    proxyPkg = pkg;
                } else if (!pkg.equals(proxyPkg)) {
                    throw new IllegalArgumentException(
                        "non-public interfaces from different packages");
                }
            }
        }

        if (proxyPkg == null) {
            // if no non-public proxy interfaces, use com.sun.proxy package
            proxyPkg = ReflectUtil.PROXY_PACKAGE + ".";
        }

        /*
         * Choose a name for the proxy class to generate.
         */
        long num = nextUniqueNumber.getAndIncrement();
        //代理类的完全限定名，如com.sun.proxy.$Proxy0.calss
        String proxyName = proxyPkg + proxyClassNamePrefix + num;

        /*
         * Generate the specified proxy class.
         */
        //核心部分，生成代理类的字节码
        byte[] proxyClassFile = ProxyGenerator.generateProxyClass(
            proxyName, interfaces, accessFlags);
        try {
            //把代理类加载到JVM中，至此动态代理过程基本结束了
            return defineClass0(loader, proxyName,
                                proxyClassFile, 0, proxyClassFile.length);
        } catch (ClassFormatError e) {
            /*
             * A ClassFormatError here means that (barring bugs in the
             * proxy class generation code) there was some other
             * invalid aspect of the arguments supplied to the proxy
             * class creation (such as virtual machine limitations
             * exceeded).
             */
            throw new IllegalArgumentException(e.toString());
        }
    }
}
```

