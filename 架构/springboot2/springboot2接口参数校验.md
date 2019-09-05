# springboot2接口参数校验

## validation包下的注解说明

这里只列举了 `javax.validation` 包下的注解，同理在 `spring-boot-starter-web` 包中也存在 `hibernate-validator` 验证包，里面包含了一些 `javax.validation` 没有的注解，有兴趣的可以看看

|            注解             |                             说明                             |
| :-------------------------: | :----------------------------------------------------------: |
|         `@NotNull`          |                     **限制必须不为null**                     |
|         `@NotEmpty`         | **验证注解的元素值不为 null 且不为空（字符串长度不为0、集合大小不为0）** |
|         `@NotBlank`         | **验证注解的元素值不为空（不为null、去除首位空格后长度为0），不同于@NotEmpty，@NotBlank只应用于字符串且在比较时会去除字符串的空格** |
|      `@Pattern(value)`      |               **限制必须符合指定的正则表达式**               |
|      `@Size(max,min)`       |  **限制字符长度必须在 min 到 max 之间（也可以用在集合上）**  |
|          `@Email`           | **验证注解的元素值是Email，也可以通过正则表达式和flag指定自定义的email格式** |
|        `@Max(value)`        |             **限制必须为一个不大于指定值的数字**             |
|        `@Min(value)`        |             **限制必须为一个不小于指定值的数字**             |
|    `@DecimalMax(value)`     |             **限制必须为一个不大于指定值的数字**             |
|    `@DecimalMin(value)`     |             **限制必须为一个不小于指定值的数字**             |
|           `@Null`           |                 **限制只能为null（很少用）**                 |
|       `@AssertFalse`        |                **限制必须为false （很少用）**                |
|        `@AssertTrue`        |                **限制必须为true （很少用）**                 |
|           `@Past`           |                 **限制必须是一个过去的日期**                 |
|          `@Future`          |                 **限制必须是一个将来的日期**                 |
| `@Digits(integer,fraction)` | **限制必须为一个小数，且整数部分的位数不能超过 integer，小数部分的位数不能超过 fraction （很少用）** |



### 使用代码样例

#### 对象验证

```java
import lombok.Data;
import org.hibernate.validator.constraints.Length;

import javax.validation.constraints.DecimalMin;
import javax.validation.constraints.NotBlank;
import javax.validation.constraints.NotNull;
import java.math.BigDecimal;

@Data
public class Book {

    private Integer id;
    @NotBlank(message = "name 不允许为空")
    @Length(min = 2, max = 10, message = "name 长度必须在 {min} - {max} 之间")
    private String name;
    @NotNull(message = "price 不允许为空")
    @DecimalMin(value = "0.1", message = "价格不能低于 {value}")
    private BigDecimal price;

}
```



#### 参数验证

```java
import com.example.validation.entity.Book;
import org.hibernate.validator.constraints.Length;
import org.springframework.validation.annotation.Validated;
import org.springframework.web.bind.annotation.GetMapping;
import org.springframework.web.bind.annotation.RestController;

import javax.validation.constraints.NotBlank;

@RestController
@Validated
public class TestController {

    @GetMapping("/test2")
    public String test2(@NotBlank(message = "name 不能为空")
                        @Length(min = 2, max = 10, message = "name 长度必须在 {min} - {max} 之间") String name) {
        return "success";
    }

    @GetMapping("/test3")
    public String test3(@Validated Book book) {
        return "success";
    }

}
```





## 自定义 Validator 注解

熟悉 `ConstraintValidator` 接口并且编写自己的数据验证注解

### 自定义实现日期格式的校验

这里定义了一个 `@DateTime` 注解，在该注解上标注了 `@Constraint` 注解，它的作用就是指定一个具体的校验器类。



```java
import javax.validation.Constraint;
import javax.validation.Payload;

import java.lang.annotation.Retention;
import java.lang.annotation.Target;

import static java.lang.annotation.ElementType.FIELD;
import static java.lang.annotation.ElementType.PARAMETER;
import static java.lang.annotation.RetentionPolicy.RUNTIME;

@Target({FIELD, PARAMETER})
@Retention(RUNTIME)
@Constraint(validatedBy = DateTimeValidator.class)
public @interface DateTime {


    String message() default "格式错误";

    String format() default "yyyy-MM-dd";

    Class<?>[] groups() default {};

    Class<? extends Payload>[] payload() default {};
}
```

继承ConstraintValidator，实现校验逻辑

```java
import javax.validation.ConstraintValidator;
import javax.validation.ConstraintValidatorContext;
import java.text.ParseException;
import java.text.SimpleDateFormat;

public class DateTimeValidator implements ConstraintValidator<DateTime, String> {
    private DateTime dateTime;

    @Override
    public void initialize(DateTime dateTime) {
        this.dateTime = dateTime;
    }

    @Override
    public boolean isValid(String value, ConstraintValidatorContext context) {
        // 如果 value 为空则不进行格式验证，为空验证可以使用 @NotBlank @NotNull @NotEmpty 等注解来进行控制，职责分离
        if (value == null) {
            return true;
        }
        String format = dateTime.format();
        if (value.length() != format.length()) {
            return false;
        }
        SimpleDateFormat simpleDateFormat = new SimpleDateFormat(format);
        try {
            simpleDateFormat.parse(value);
        } catch (ParseException e) {
            return false;
        }
        return true;
    }
}
```

测试

```java
@RestController
@Validated
public class TestController {

    @GetMapping("/test")
    public String test(@DateTime(message = "您输入的格式错误，正确的格式为：{format}", format = "yyyy-MM-dd HH:mm:ss") String date) {
        return "success";
    }

}
```



## 分组验证

有的时候，我们对一个实体类需要有多中验证方式，在不同的情况下使用不同验证方式，比如说对于一个实体类来的 id 来说，新增的时候是不需要的，对于更新时是必须的，这个时候你是选择写一个实体类呢还是写两个呢？

@DateTime注解需要有一个 `groups` 属性，这个属性的作用又是什么呢?

**groups 属性的作用就让 @Validated 注解只验证与自身 value 属性相匹配的字段，可多个，只要满足就会去纳入验证范围；**我们都知道针对新增的数据我们并不需要验证 ID 是否存在，我们只在做修改操作的时候需要用到，因此这里将 ID 字段归纳到 `Groups.Update.class` 中去，而其它字段是不论新增还是修改都需要用到所以归纳到 `Groups.Default.class` 中…



###   代码实现



定义一个验证组，里面写上不同的空接口类即可

```java
public class Groups {

    public interface Update {}

    public interface Default {}
}
```

**groups 属性的作用就让 @Validated 注解只验证与自身 value 属性相匹配的字段，可多个，只要满足就会去纳入验证范围；**我们都知道针对新增的数据我们并不需要验证 ID 是否存在，我们只在做修改操作的时候需要用到，因此这里将 ID 字段归纳到 `Groups.Update.class` 中去，而其它字段是不论新增还是修改都需要用到所以归纳到 `Groups.Default.class` 中…

```java
@Data
public class Book {

    @NotNull(message = "id 不能为空", groups = Groups.Update.class)
    private Integer id;
    @NotBlank(message = "name 不允许为空", groups = Groups.Default.class)
    private String name;
    @NotNull(message = "price 不允许为空", groups = Groups.Default.class)
    private BigDecimal price;

}
```



测试



```java
@RestController
@Validated
public class TestController {

    @GetMapping("/insert")
    public String insert(@Validated(value = Groups.Default.class) Book book) {
        return "insert";
    }

    @GetMapping("/update")
    public String update(@Validated(value = {Groups.Default.class, Groups.Update.class}) Book book) {
        return "update";
    }
}

```











