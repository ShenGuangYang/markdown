# springboot2 异步执行



## 定义异步线程池

```java
@Configuration
@EnableAsync // 启用异步
public class MyAsyncConfig implements AsyncConfigurer{
    @Nullable
    @Override
    public Executor getAsyncExecutor() {

        ThreadPoolTaskExecutor taskExecutor = new ThreadPoolTaskExecutor();
        taskExecutor.setCorePoolSize(10);  // 核心线程数
        taskExecutor.setMaxPoolSize(50);    // 最大线程数
        taskExecutor.setQueueCapacity(5000);// 最大队列数
        taskExecutor.initialize();          //初始化
        return taskExecutor;
    }
}
```



## 使用

```java
@RestController
@RequestMapping("async")
public class AsyncDemoController {

    @Autowired
    private AsyncDemo asyncDemo;

    @RequestMapping("index")
    public String asyncDemo() {
        asyncDemo.test();
        asyncDemo.testAsync();
        return "asyncDemo";
    }

}
```

```java
public interface AsyncDemo {

    void test();

    void testAsync();
}
```



```java
@Service
public class AsyncDemoImpl implements AsyncDemo{

    @Override
    public void test() {
        System.out.println("AsyncDemoImpl 执行完");
    }

    @Override
    @Async
    public void testAsync() {
        System.out.println("testAsync 执行开始 " +Thread.currentThread().getName());
        try {
            TimeUnit.SECONDS.sleep(3);
        } catch (InterruptedException e) {
            e.printStackTrace();
        }
        System.out.println("testAsync 执行结束" + Thread.currentThread().getName());
    }

}
```

