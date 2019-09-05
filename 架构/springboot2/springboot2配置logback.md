# logback 简介

+ log4j的后续项目
+ `springboot`默认使用的日志框架是`logback`。
+ `logback`和`log4j`是一个人写的
+ 三个模块组成
  - logback-core
  - logback-classic
  - logback-access



大体配置模板：

```xml
<?xml version="1.0" encoding="UTF-8" ?>

<configuration debug="false">

    <appender name="STDOUT" class="ch.qos.logback.core.ConsoleAppender">
        <encoder>
            <charset>UTF-8</charset>
            <pattern>[%d{'MM-dd HH:mm:ss,SSS',GMT+8:00}] %-5p [%.10t][%X{IP}][%X{OP}][%X{OPAS}] %logger{36}[%L] - %m%n</pattern>
        </encoder>
    </appender>

    <appender name="INFO" class="ch.qos.logback.core.rolling.RollingFileAppender">
        <append>true</append>
        <file>./logs/http-test.log</file>
        <rollingPolicy class="ch.qos.logback.core.rolling.TimeBasedRollingPolicy">
            <FileNamePattern>./logs/http-test.log.%d{yyyy-MM-dd}</FileNamePattern>
            <MaxHistory>30</MaxHistory>
        </rollingPolicy>
        <encoder class="ch.qos.logback.classic.encoder.PatternLayoutEncoder">
            <pattern>%d{yyyy-MM-dd HH:mm:ss.SSS} [%thread] %-5level %logger{50} - %msg%n</pattern>
            <charset>UTF-8</charset>
        </encoder>
    </appender>

    <appender name="ASYNC-INFO" class="ch.qos.logback.classic.AsyncAppender">
        <!-- 不丢失日志.默认的,如果队列的80%已满,则会丢弃TRACT、DEBUG、INFO级别的日志 -->
        <discardingThreshold>0</discardingThreshold>
        <!-- 更改默认的队列的深度,该值会影响性能.默认值为256 -->
        <queueSize>256</queueSize>
        <!-- 添加附加的appender,最多只能添加一个 -->
        <appender-ref ref="INFO"/>
    </appender>

    <root level="INFO">
        <appender-ref ref="STDOUT" />
        <!--<appender-ref ref="ASYNC-INFO" />-->
        <appender-ref ref="INFO" />
    </root>

</configuration>
```







# 配置文件详解

## configuration

```xml
<configuration scan="true" scanPeriod="60 seconds" debug="false">  

    <property name="glmapper-name" value="glmapper-demo" /> 
    <contextName>${glmapper-name}</contextName> 
    
    <appender>
        //xxxx
    </appender>   
    
    <logger>
        //xxxx
    </logger>
    
    <root>             
       //xxxx
    </root>  
</configuration>
```



> ps：想使用spring扩展profile支持，要以logback-spring.xml命名，其他如property需要改为springProperty

   

- scan:当此属性设置为true时，配置文件如果发生改变，将会被重新加载，默认值为true。
- scanPeriod:设置监测配置文件是否有修改的时间间隔，如果没有给出时间单位，默认单位是毫秒。当scan为true时，此属性生效。默认的时间间隔为1分钟。
- debug:当此属性设置为true时，将打印出logback内部日志信息，实时查看logback运行状态。默认值为false。

### contextName

每个`logger`都关联到`logger`上下文，默认上下文名称为`“default”`。但可以使用`contextName`标签设置成其他名字，用于区分不同应用程序的记录

### property

用来定义变量值的标签，`property`标签有两个属性，`name`和`value`；其中`name`的值是变量的名称，`value`的值时变量定义的值。通过`property`定义的值会被插入到`logger`上下文中。定义变量后，可以使“${name}”来使用变量。如上面的`xml`所示。

### logger

用来设置某一个包或者具体的某一个类的日志打印级别以及指定`appender`。

### root

根logger，也是一种logger，且只有一个level属性

### appender

负责写日志的组件，下面会细说

### filter

filter其实是appender里面的子元素。它作为过滤器存在，执行一个过滤器会有返回DENY，NEUTRAL，ACCEPT三个枚举值中的一个。

- DENY：日志将立即被抛弃不再经过其他过滤器
- NEUTRAL：有序列表里的下个过滤器过接着处理日志
- ACCEPT：日志会被立即处理，不再经过剩余过滤器

# 案例分析

首先来配置一个非常简单的文件。这里申请下，我使用的是 `logback-spring.xml`。和 `logback.xml` 在`properties`上有略微差别。其他都一样。

properties中就是指定了日志的打印级别和日志的输出位置：

```proper
#设置应用的日志级别
logging.level.com.example.httptest=INFO
#路径
logging.path=E:/logs/test/http-test.log
```

## 通过控制台输出的log

### logback-spring.xml的配置如下：

```xml
<configuration>
    <!-- 默认的控制台日志输出，一般生产环境都是后台启动，这个没太大作用 -->
    <appender name="STDOUT" class="ch.qos.logback.core.ConsoleAppender">
        <encoder class="ch.qos.logback.classic.encoder.PatternLayoutEncoder">
            <Pattern>%d{HH:mm:ss.SSS} %-5level %logger{80} - %msg%n</Pattern>
        </encoder>
    </appender>
    
    <root level="info">
        <appender-ref ref="STDOUT"/>
    </root>
</configuration>
```

上面的就是通过控制台打印出来的，这个时候因为我们没有指定日志文件的输出，因为不会在工程目录下生产`logs`文件夹。

## 控制台不打印，直接输出到日志文件

先来看下配置文件：

```xml
<configuration>
    <!-- 属性文件:在properties文件中找到对应的配置项 -->
    <springProperty scope="context" name="logging.path"  source="logging.path"/>
    <springProperty scope="context" name="logging.level" source="logging.level.com.example.httptest"/>
    <!-- 默认的控制台日志输出，一般生产环境都是后台启动，这个没太大作用 -->
    <appender name="STDOUT"
        class="ch.qos.logback.core.ConsoleAppender">
        <encoder class="ch.qos.logback.classic.encoder.PatternLayoutEncoder">
            <Pattern>%d{HH:mm:ss.SSS} %-5level %logger{80} - %msg%n</Pattern>
        </encoder>
    </appender>
    
   <appender name="INFO" class="ch.qos.logback.core.rolling.RollingFileAppender">
        <append>true</append>
        <file>E:/logs/test/http-test.log</file>
        <rollingPolicy class="ch.qos.logback.core.rolling.TimeBasedRollingPolicy">
            <FileNamePattern>E:/logs/test/http-test.log.%d{yyyy-MM-dd}</FileNamePattern>
            <MaxHistory>30</MaxHistory>
        </rollingPolicy>
        <encoder class="ch.qos.logback.classic.encoder.PatternLayoutEncoder">
            <pattern>%d{yyyy-MM-dd HH:mm:ss.SSS} [%thread] %-5level %logger{50} - %msg%n</pattern>
            <charset>UTF-8</charset>
        </encoder>
    </appender>
    
    <root level="info">
        <appender-ref ref="INFO"/>
    </root>
</configuration>
```



## logger和appender的关系

上面两种是一个基本的配置方式，通过上面两个案例，我们先来了解下`logger/appender/root`之间的关系，然后再详细的说下`logger`和`appender`的配置细节。

在最前面介绍中提到，`root`是根`logger`,所以他两是一回事；只不过`root`中不能有`name`和`additivity`属性，是有一个`level`。

`appender`是一个日志打印的组件，这里组件里面定义了打印过滤的条件、打印输出方式、滚动策略、编码方式、打印格式等等。但是它仅仅是一个打印组件，如果我们不使用一个`logger`或者`root`的`appender-ref`指定某个具体的`appender`时，它就没有什么意义。

因此`appender`让我们的应用知道怎么打、打印到哪里、打印成什么样；而`logger`则是告诉应用哪些可以这么打。例如某个类下的日志可以使用这个`appender`打印或者某个包下的日志可以这么打印。



## appender 配置详解



```xml
<appender name="INFO" class="ch.qos.logback.core.rolling.RollingFileAppender">
    <append>true</append>
    <filter class="ch.qos.logback.classic.filter.ThresholdFilter">
        <level>${logging.level}</level>
    </filter>
    <file>${logging.path}/glmapper-spring-boot/glmapper-loggerone.log</file>
    <rollingPolicy class="ch.qos.logback.core.rolling.TimeBasedRollingPolicy">
        <FileNamePattern>log.log.%d{yyyy-MM-dd}</FileNamePattern>
        <MaxHistory>30</MaxHistory>
    </rollingPolicy>
    <encoder class="ch.qos.logback.classic.encoder.PatternLayoutEncoder">
        <pattern>%d{yyyy-MM-dd HH:mm:ss.SSS} [%thread] %-5level %logger{50} - %msg%n</pattern>
        <charset>UTF-8</charset>
    </encoder>
</appender>
```



`appender` 有两个属性 `name`和`class`;`name`指定`appender`名称，`class`指定`appender`的全限定名。上面声明的是名为`GLMAPPER-LOGGERONE`，`class`为`ch.qos.logback.core.rolling.RollingFileAppender`的一个`appender`。

### appender 的种类

- ConsoleAppender：把日志添加到控制台
- FileAppender：把日志添加到文件
- RollingFileAppender：滚动记录文件，先将日志记录到指定文件，当符合某个条件时，将日志记录到其他文件。它是FileAppender的子类



### append 子标签

```xml
<append>true</append>
```

### filter 子标签

在简介中提到了`filter`；作用就是上面说的。可以为`appender` 添加一个或多个过滤器，可以用任意条件对日志进行过滤。`appender` 有多个过滤器时，按照配置顺序执行。

#### ThresholdFilter

临界值过滤器，过滤掉低于指定临界值的日志。当日志级别等于或高于临界值时，过滤器返回`NEUTRAL`；当日志级别低于临界值时，日志会被拒绝。

```xml
<filter class="ch.qos.logback.classic.filter.ThresholdFilter">
    <level>INFO</level>
</filter>
```

#### LevelFilter

级别过滤器，根据日志级别进行过滤。如果日志级别等于配置级别，过滤器会根据`onMath`(用于配置符合过滤条件的操作) 和 `onMismatch`(用于配置不符合过滤条件的操作)接收或拒绝日志。

```xml
<filter class="ch.qos.logback.classic.filter.LevelFilter">   
  <level>INFO</level>   
  <onMatch>ACCEPT</onMatch>   
  <onMismatch>DENY</onMismatch>   
</filter>
```

### file 子标签

`file` 标签用于指定被写入的文件名，可以是相对目录，也可以是绝对目录，如果上级目录不存在会自动创建，没有默认值。

### rollingPolicy 子标签

这个子标签用来描述滚动策略的。这个只有`appender`的`class`是`RollingFileAppender`时才需要配置。这个也会涉及文件的移动和重命名（a.log->a.log.2018.07.22）。

#### TimeBasedRollingPolicy

最常用的滚动策略，它根据时间来制定滚动策略，既负责滚动也负责出发滚动。这个下面又包括了两个属性：

- FileNamePattern
- maxHistory

```xml
<rollingPolicy class="ch.qos.logback.core.rolling.TimeBasedRollingPolicy">
    <!--日志文件输出的文件名:按天回滚 daily -->
    <FileNamePattern>
        ${logging.path}/glmapper-spring-boot/glmapper-loggerone.log.%d{yyyy-MM-dd}
    </FileNamePattern>
    <!--日志文件保留天数-->
    <MaxHistory>30</MaxHistory>
</rollingPolicy>
```

上面的这段配置表明**每天生成一个日志文件，保存30天的日志文件**

#### FixedWindowRollingPolicy

根据固定窗口算法重命名文件的滚动策略。

### encoder 子标签

对记录事件进行格式化。它干了两件事：

- 把日志信息转换成字节数组
- 把字节数组写入到输出流

```xml
<encoder class="ch.qos.logback.classic.encoder.PatternLayoutEncoder">
    <pattern>%d{yyyy-MM-dd HH:mm:ss.SSS} [%thread] %-5level %logger{50}
    - %msg%n</pattern>
    <charset>UTF-8</charset>
</encoder>
```

目前`encoder`只有`PatternLayoutEncoder`一种类型。

### 定义一个只打印error级别日志的appcener

```xml
<!-- 错误日志 appender ： 按照每天生成日志文件 -->
<appender name="INFO" class="ch.qos.logback.core.rolling.RollingFileAppender">
    <append>true</append>
    <!-- 过滤器，只记录 error 级别的日志 -->
    <filter class="ch.qos.logback.classic.filter.ThresholdFilter">
        <level>error</level>
    </filter>
    <!-- 日志名称 -->
    <file>${logging.path}/glmapper-spring-boot/glmapper-error.log</file>
    <!-- 每天生成一个日志文件，保存30天的日志文件 -->
    <rollingPolicy class="ch.qos.logback.core.rolling.TimeBasedRollingPolicy">
        <!--日志文件输出的文件名:按天回滚 daily -->
        <FileNamePattern>error.log.%d{yyyy-MM-dd}</FileNamePattern>
        <!--日志文件保留天数-->
        <MaxHistory>30</MaxHistory>
    </rollingPolicy>
    <encoder class="ch.qos.logback.classic.encoder.PatternLayoutEncoder">
        <!--格式化输出：%d表示日期，%thread表示线程名，%-5level：级别从左显示5个字符宽度%msg：日志消息，%n是换行符-->
        <pattern>%d{yyyy-MM-dd HH:mm:ss.SSS} [%thread] %-5level %logger{50} - %msg%n</pattern>
        <!-- 编码 -->
        <charset>UTF-8</charset>
    </encoder>
</appender>
```

### 定义一个输出到控制台的appender

```xml
<!-- 默认的控制台日志输出，一般生产环境都是后台启动，这个没太大作用 -->
<appender name="STDOUT" class="ch.qos.logback.core.ConsoleAppender">
    <encoder class="ch.qos.logback.classic.encoder.PatternLayoutEncoder">
        <Pattern>%d{HH:mm:ss.SSS} %-5level %logger{80} - %msg%n</Pattern>
    </encoder>
</appender>
```

## logger 配置详解

```xml
<logger name="com.glmapper.spring.boot.controller"
        level="${logging.level}" additivity="false">
    <appender-ref ref="INFO" />
</logger>
```



上面的这个配置文件描述的是：`com.glmapper.spring.boot.controller`这个包下的`${logging.level}`级别的日志将会使用`GLMAPPER-LOGGERONE`来打印。`logger`有三个属性和一个子标签：

- name:用来指定受此`logger`约束的某一个包或者具体的某一个类。
- level:用来设置打印级别（`TRACE`, `DEBUG`, `INFO`, `WARN`, `ERROR`, `ALL` 和 `OFF`），还有一个值`INHERITED`或者同义词`NULL`，代表强制执行上级的级别。如果没有设置此属性，那么当前`logger`将会继承上级的级别。
- addtivity:用来描述是否向上级`logger`传递打印信息。默认是`true`。

`appender-ref`则是用来指定具体`appender`的。

## 不同日志隔离打印案例

在前面的例子中我们有三种appender,一个是指定包约束的，一个是控制error级别的，一个是控制台的。然后这小节我们就来实现下不同日志打印到不同的log文件中。

### 根据类进行日志文件隔离

这个其实也是和上面那个差不过，只不过粒度更细一点，一般情况下比如说我们有个定时任务类需要单独来记录其日志信息，这样我们就可以考虑使用基于类维度来约束打印。

```xml
<!--特殊功能单独appender 例如调度类的日志-->
<appender name="SCHEDULERTASKLOCK-APPENDER" class="ch.qos.logback.core.rolling.RollingFileAppender">
    <append>true</append>
    <file>scheduler-task-lock.log</file>
    <!-- 每天生成一个日志文件，保存30天的日志文件 -->
    <rollingPolicy class="ch.qos.logback.core.rolling.TimeBasedRollingPolicy">
        <!--日志文件输出的文件名:按天回滚 daily -->
        <FileNamePattern>${logging.path}/glmapper-spring-boot/scheduler-task-lock.log.%d{yyyy-MM-dd}</FileNamePattern>
        <!--日志文件保留天数-->
        <MaxHistory>30</MaxHistory>
    </rollingPolicy>
    <encoder class="ch.qos.logback.classic.encoder.PatternLayoutEncoder">
        <!--格式化输出：%d表示日期，%thread表示线程名，%-5level：级别从左显示5个字符宽度%msg：日志消息，%n是换行符-->
        <pattern>%d{yyyy-MM-dd HH:mm:ss.SSS} [%thread] %-5level %logger{50} - %msg%n</pattern>
        <!-- 编码 -->
        <charset>UTF-8</charset>
    </encoder>
</appender>

<!--这里指定到了具体的某一个类-->
<logger name="com.glmapper.spring.boot.task.TestLogTask" level="${logging.level}" additivity="true">
    <appender-ref ref="SCHEDULERTASKLOCK-APPENDER" />
</logger>
```



## 异步输出日志

```xml
<appender name="FILE" class="ch.qos.logback.core.rolling.RollingFileAppender">
    <rollingPolicy class="ch.qos.logback.core.rolling.TimeBasedRollingPolicy">
        <fileNamePattern>logs/context-log.%d{yyyy-MM-dd}.log</fileNamePattern>
        <maxHistory>30</maxHistory>
    </rollingPolicy>
    <encoder charset="UTF-8">
        <pattern>[%-5level] %date --%thread-- [%logger] %msg %n</pattern>
    </encoder>
</appender>
 
<appender name ="ASYNC_FILE" class= "ch.qos.logback.classic.AsyncAppender">
    <discardingThreshold >0</discardingThreshold>
    <queueSize>1234</queueSize>
    <appender-ref ref = "FILE"/>
</appender>
```

​      由以上测试结果可以得出异步方式记录日志并不是什么情况下都能提升性能的，相反由于线程间的同步开销，甚至可能降低性能；==**只有像在JDBC操作或是SMTP之类的记录耗时比较长的情况下，使用异步入库方式才是个好选择。**==


  