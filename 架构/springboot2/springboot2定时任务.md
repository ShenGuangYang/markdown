# springboot2定时任务

## 测试代码

```java
@Configuration
@EnableScheduling
public class ScheduledTask {

    // 间隔1秒执行，配置文件取值
    @Scheduled(fixedRateString= "${scheduler.time}") 
    public void taskDemo() {
        System.out.println("taskDemo: " + LocalDateTime.now().format(DateTimeFormatter.ofPattern("yyyy-MM-dd HH:mm:ss")));
    }

    // 容器启动3秒后， 间隔5秒执行
    @Scheduled(initialDelay = 3000, fixedRate = 5000)
    public void testDemo1() {
        System.out.println("testDemo1: " + LocalDateTime.now().format(DateTimeFormatter.ofPattern("yyyy-MM-dd HH:mm:ss")));
    }

    // cron 表达式执行
    @Scheduled(cron = "......")
    public void testDemo2() {
        System.out.println("testDemo2: " + LocalDateTime.now().format(DateTimeFormatter.ofPattern("yyyy-MM-dd HH:mm:ss")));
    }

}

```



## Scheduled 配置项



| 配置项       | 类型   | 描述                                           |
| ------------ | ------ | ---------------------------------------------- |
| cron         | String | 使用表达式执行任务                             |
| zone         | String | 设置区域时间                                   |
| fixedDelay   | long   | 上一个任务完成到下一个任务开始的间隔，单位毫秒 |
| initialDelay | long   | 首次延迟多少毫秒执行                           |
| fixedRate    | long   | 上一个任务开始到下一个任务开始的间隔，单位毫秒 |
|              |        |                                                |
|              |        |                                                |

