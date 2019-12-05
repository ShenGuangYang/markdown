# fastjson

## 基本使用

### 序列化

```java
String jsonString = JSONObject.toJSONString(obj);
```

### 反序列化

```java
VO vo = JSONObject.parseObject("...", VO.class);
List<VO> list = JSONObject.parseArray("...", VO.class);
```

### 泛型序列化

```java
List<VO> list = JSONObject.parseObject("...", new TypeReference<List<VO>>() {});
```



### 日期处理

```java
JSONObject.toJSONStringWithDateFormat(test, "yyyy-MM-dd HH:mm:ss", SerializerFeature.WriteDateUseDateFormat);
```



## JSONField && JSONType

### JSONField

**私有属性必须有set方法才能进行反序列化**

```java
public @interface JSONField {
    // 配置序列化和反序列化的顺序
    int ordinal() default 0;
     // 指定字段的名称
    String name() default "";
    // 指定字段的格式，对日期格式有用
    String format() default "";
    // 是否序列化
    boolean serialize() default true;
    // 是否反序列化
    boolean deserialize() default true;
}
```

测试代码

```java
@Data
@AllArgsConstructor
public class A {
    @JSONField(ordinal = 2)
    private int id;
    @JSONField(ordinal = 1, format = "yyyy-MM-dd")
    private Date date;
    @JSONField(ordinal = 3, serializeUsing = Test.class)
    private int money;
}
```

```java
public class Test implements ObjectSerializer {
    @Override
    public void write(JSONSerializer serializer, Object object, Object fieldName, Type fieldType, int features) throws IOException {
        Integer value = (Integer) object;
        String text = value + "元";
        serializer.write(text);
    }
    public static void main(String[] args) {
        A a = new A(1, new Date(), 200);
        String s = JSONObject.toJSONString(a);
        System.out.println(s);
        // {"date":"2019-12-05","id":1,"money":"200元"}
    }
}
```



### JSONType

```java
public @interface JSONType {
    // 包含字段
    String[] includes() default {};
	// 忽略字段
    String[] ignores() default {};
}
```

测试代码

```java
@Data
@AllArgsConstructor
@JSONType(includes = {"name", "sex"}, ignores = {"addr"})
public class B {
    private String name;
    private String sex;
    private String addr;
}
```

```java
public class Test {
    public static void main(String[] args) {
        B b = new B("shen", "男", "杭州");
        String s = JSONObject.toJSONString(b);
        System.out.println(s);
    }
}
```



## SerializeFilter 定制序列化

通过SerializeFilter可以使用扩展编程的方式实现定制序列化。fastjson提供了多种SerializeFilter：

- PropertyPreFilter 根据PropertyName判断是否序列化；
- PropertyFilter 根据PropertyName和PropertyValue来判断是否序列化；
- NameFilter 修改Key，如果需要修改Key，process返回值则可；
- ValueFilter 修改Value；
- BeforeFilter 序列化时在最前添加内容；
- AfterFilter 序列化时在最后添加内容。
- LabelFilter 根据 label 来判断字段是否序列化

实例对象：

```java
@Data
@AllArgsConstructor
public class B {
    private int id;
    @JSONField(label = "bbb")
    private String name;
    private List<String> demo;
}
```



### PropertyFilter 根据PropertyName和PropertyValue定制序列化

可以通过扩展实现根据object或者属性名称或者属性值进行判断是否需要序列化。例如：

```java
public class Test {
    public static void main(String[] args) {
        PropertyFilter filter = (object, name, value) -> {
            if ("id".equals(name)) {
                int id = (int) value;
                return id > 10;
            }
            if ("name".equals(name)) {
                return true;
            }
            return false;
        };
        B b1 = new B(10, "shen-10", null);
        B b2 = new B(20, "shen-20", null);
        String json1 = JSONObject.toJSONString(b1, filter);
        String json2 = JSONObject.toJSONString(b2, filter);
        System.out.println(json1); // 输出：{"name":"shen-10"}
        System.out.println(json2); // 输出：{"id":20,"name":"shen-20"}
    }
}
```



### NameFilter 序列化时修改Key

```java
public class Test {
    public static void main(String[] args) {
        NameFilter filter = (object, name, value) -> {
            if ("name".equals(name)) {
                return name + "_cn";
            }
            return name;
        };
        B b1 = new B(10, "shen-10", null);
        String json1 = JSONObject.toJSONString(b1, filter);
        System.out.println(json1); // 输出: {"id":10,"name_cn":"shen-10"}
    }
}
```



### ValueFilter 序列化时修改Value

```java
public class Test {
    public static void main(String[] args) {
        ValueFilter filter = (object, name, value) -> {
            if ("name".equals(name)) {
                return "修改一下value";
            }
            return value;
        };
        B b1 = new B(10, "shen-10", null);
        String json1 = JSONObject.toJSONString(b1, filter);
        System.out.println(json1); // 输出: {"id":10,"name":"修改一下value"}
    }
}
```



### BeforeFilter 序列化之前执行

```java
public class Test {
    public static void main(String[] args) {

        BeforeFilter filter = new BeforeFilter() {
            @Override
            public void writeBefore(Object object) {
                B b = (B) object;
                System.out.println("BeforeFilter====" + b);
                b.setName(b.getName() + " 10  Before");
            }
        };
        B b = new B(10, "shen-10", null);
        String json2 = JSONObject.toJSONString(b, filter);
        System.out.println("json2====" + json2);
        System.out.println("b====" + b);

        // 输出
        // BeforeFilter====B(id=10, name=shen-10, demo=null)
        // json2===={"id":10,"name":"shen-10 10  Before"}
        // b====B(id=10, name=shen-10 10  Before, demo=null)
    }
}
```



### AfterFilter 序列化之后执行

```java
public class Test {
    public static void main(String[] args) {

        AfterFilter filter = new AfterFilter() {
            @Override
            public void writeAfter(Object object) {
                B b = (B) object;
                System.out.println("AfterFilter====" + b);
                b.setName(b.getName() + " 10  after");
            }
        };
        B b = new B(10, "shen-10", null);
        String json2 = JSONObject.toJSONString(b, filter);
        System.out.println("json2====" + json2);
        System.out.println("b====" + b);

        // 输出
        // AfterFilter====B(id=10, name=shen-10, demo=null)
        // json2===={"id":10,"name":"shen-10"}
        // b====B(id=10, name=shen-10 10  after, demo=null)
    }
}
```



### LabelFilter 根据 label 来判断字段是否序列化

```java
public class Test {
    public static void main(String[] args) {
        B b = new B(10, "shen-10");
        String a = JSONObject.toJSONString(b, Labels.includes("bbb"));
        String aa = JSONObject.toJSONString(b, Labels.excludes("bbb"));
        System.out.println(a);
        System.out.println(aa);
        // {"name":"shen-10"}
        // {"id":10}
    }
}
```



## ParseProcess定制反序列化



ParseProcess是编程扩展定制反序列化的接口。fastjson支持如下ParseProcess：

- ExtraProcessor 用于处理多余的字段；
- ExtraTypeProvider 用于处理多余字段时提供类型信息。





