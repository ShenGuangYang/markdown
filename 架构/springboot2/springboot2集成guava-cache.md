# springboot2 集成guava-cache

## pom

```xml
<dependency>
    <groupId>com.google.guava</groupId>
    <artifactId>guava</artifactId>
    <version>18.0</version>
</dependency>
```



## demo

```java
@Configuration
public class CacheConfig {

    @Bean
    public LoadingCache<String, Object> loadingCache() {
        LoadingCache<String, Object> build = CacheBuilder.newBuilder()
                .expireAfterWrite(1, TimeUnit.MINUTES) // 写入一分钟后回收
                // .expireAfterAccess(1, TimeUnit.MINUTES) // 一分钟没有读取就回收
                .maximumSize(100) //设置最大值,超出容量会尝试清除不常用的值
                .build(new CacheLoader<String, Object>() {
                    @Override
                    public Object load(String key) throws Exception {
                        return null;
                    }
                });

        build.put("1", "111");
        build.put("2", "22");
        build.put("3", "33");
        return build;
    }

}
```



```java
@RestController
public class CacheDemoController {

    @Autowired
    private LoadingCache<String, Object> loadingCache;

    @GetMapping("index")
    public Object testDemo() {

        Object o = null;
        try {
            o = loadingCache.get("1");
        } catch (ExecutionException e) {
            e.printStackTrace();
        }
        return o;
    }

}
```

