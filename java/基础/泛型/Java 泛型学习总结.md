# Java 泛型学习总结

## 前言

Java 5 添加了泛型，提供了编译时类型安全检测机制，该机制允许程序员在编译时检测到非法的类型。
泛型的本质是参数化类型，可以为以前处理通用对象的类和方法，指定具体的对象类型。听起来有点抽象，所以我们将马上看一些泛型用在集合上的例子：

## 泛型集合

先看一个没有泛型的集合例子：
```Java
List list = new ArrayList();
list.add(new Integer(2));
list.add("a String");
```
因为 List 可以添加任何对象，所以从 List 中取出的对象时，因为不确定（List不记住元素类型）当时候保存进 List 的元素类型，这个对象的类型只能是 Object ，还必须由程序编写者记住添加元素类型，然后取出时再进行强制类型转换就，如下：

```Java
Integer integer = (Integer) list.get(0);
String string   = (String) list.get(1);
```

通常，我们只使用带有单一类型元素的集合，并且不希望其他类型的对象被添加到集合中，例如，只有 Integer 的 List ，不希望将 String 对象放进集合，并且也不想自己记住元素类型，并且强制类型转换还可能会出现错误。

使用 Generics（泛型）就可以设置集合的类型，以限制可以将哪种对象插入集合中。 这可以确保集合中的元素，都是同一种已知类型的，因此取出数据的时候就不必进行强制类型转换了，下面是一个 String 类型的 List 的使用例子：

```Java
List<String> strings = new ArrayList<String>();
strings.add("a String");
String aString = strings.get(0);
```

以上这个 List 集合只能放入 String 对象，如果视图放入别的对象那么编译器会报错，让代码编写者必须进行处理，这就是额外的类型检查。另外注意List 的 <> 尖括号里面只能是对象引用类型，不包括基本类型（可以使用对应包装类代替）。
Java 泛型从 Java 7 开始，编译器可以自动类型判断，可以省略构造器中的泛型，下面是一个Java 7 泛型例子：

```Java
List<String> strings = new ArrayList<>();
```

这也叫做菱形语法（<>）， 在上面的示例中，实例化 ArrayList 的时候，编译器根据前面 List 知道 new 的 ArrayList 泛型信息是 String。
foreach 循环可以很好地与泛型集合整合，如下：

```java
List<String> strings = new ArrayList<>();

// 这里省略将 String 元素添加进集合的代码...

for(String aString : strings){
  System.out.println(aString);
}
```
也可以使用泛型迭代器遍历集合，如下：

```java
List<String> list = new ArrayList<>;

Iterator<String> iterator = list.iterator();

while(iterator.hasNext()){
  String aString = iterator.next();
}
```
注意，泛型类型检查仅在编译时存在。在运行时，可以使用反射或者其他方式使字符串集合插入其他对象，但一般不会这样做。
当然泛型的用处不仅仅限于集合。

## 泛型类
从上面的内容中，我们已了解泛型的大概理念。
可以在自定义 Java 类上使用泛型，并不局限于 Java API 中的预定义类。定义泛型类只需要在类名后紧跟 尖括号，其中 T 是类型标识符，也可以是别的，比如 E 、V 等，如下例：

```java
public class GenericFactory<T> {
    Class theClass = null;

    public GenericFactory(Class theClass) {
        this.theClass = theClass;
    }

    public T createInstance() throws Exception {
        return (T) this.theClass.newInstance();
    }
}
```
其中 Class.newInstance() 方法是反射知识的内容，这里只要知道此方法用于 theClass 类对象的创建就行。
T 是一个类型标记，表示这个泛型类在实例化时可以拥有的类型集。下面是一个例子：

```java
GenericFactory<MyClass> factory = new GenericFactory<MyClass>(MyClass.class);
MyClass myClassInstance = factory.createInstance();

GenericFactory<SomeObject> factory1 = new GenericFactory<SomeObject>(SomeObject.class);
SomeObject someObjectInstance = factory1.createInstance();
```
使用泛型，我们就不必转换从 factory.createInstance() 方法返回的对象，会自动返回 new GenericFactory 的 <> 尖括号中的类型对象。

## 泛型方法

一个泛型方法定义如下：
```java
public static <T> T addAndReturn(T element, Collection<T> collection){
    collection.add(element);
    return element;
}
```
其中 T 是方法返回值， 放在方法返回值之前，是所有泛型方法必须有的类型参数声明，表示这是一个泛型方法，如果没有 只有 T ，那就不是泛型方法，而是泛型类的一个成员方法，就像上面泛型类定义的方法：

```java
public T createInstance() throws Exception { ... }
```

这不是一个泛型方法，而是 GenericFactory泛型类的一个成员方法而已，泛型方法必须在方法返回值之前有似于 的标记。
注意在泛型方法中，参数有 2 个，第一个是 T 类型的参数，第二个是元素类型为 T 的 Collection 集合，编译器会根据实际参数来推断 T 为何种类型 ，如下这样是完全可行的：

```java
String stringElement = "stringElement";
List<Object> objectList = new ArrayList<Object>();

Object theElement = addAndReturn(stringElement, objectList);
```

如上，第一个参数为 String 类型，第二个是 List 类型，貌似两个 T 是不匹配的，但是这里编译器会自动将 String 转换为 Object 类型，并将 T 标识为 Object 。

## 继承关系的泛型参数

定义如下类：

```java
public class A { }
public class B extends A { }
public class C extends A { }
```

新建并初始化如下类对象：
```java
List<A> listA = new ArrayList<A>();
List<B> listB = new ArrayList<B>();
```
因为 A 是 B 和 C 类的共同父类，那么 ListA 是不是 ListB 和 ListC 的共同父类吗？换句话说，是否可以将 List 引用指向 List 对象呢？或者反过来让 List 引用指向 List 对象呢？

答案是不可以，第一种情况，父类引用指向子类对象，如将 ListA 引用指向 ListB 对象，先假设这是可以的，有如下代码：
```java
List<A> listA = listB;
```
这会出现问题，因为 listA 是 List 类型，它可以放入任何 A 对象作为元素，可以放入 B 对象，也可以放入 C 对象，但是它实际指向的是一个 List 集合，List 集合中本应该只存储 B 类对象的，现在却不可预料地放入了 C 对象，这和当初的定义不相符 `List<B> listB = new ArrayList<B>();` ，所以这种情况是不允许的。

第二种情况，让 List 引用指向 List 对象，这样肯定也是不可以的，因为已存在的 List 集合中，可能已经存在了 A 类的非 B 类子类对象，如 C 类对象，这样也不能保证从 List 集合中取出的元素一定就是 B 对象。

但是，确实存在需要使用泛型继承关系的情况，看如下代码：

```java
public void processElements(List<A> elements){
   for(A o : elements){
      System.out.println(o.getValue()); // 这里假设的 A 类存在 getValue() 方法
   }
}
```

我们希望定义一个类，让其能处理所有指定类型元素的 List ，如 processElements(List elements) 方法，既可以处理 List ，也可以处理 List ，但是如上所述， List 和 List 根本就不存在任何继承关系。
这时就可以使用类型通配符如 <?> 来完成此需求。

## 类型通配符
有 3 中类型通配符用法，如下代码：
```java
List<?> listUknown = new ArrayList<A>(); // 任何类型元素都可接受
List<? extends A> listUknown = new ArrayList<A>(); // 可接受 A 子类类型元素
List<? super A> listUknown = new ArrayList<A>(); // 可接受 A 父类类型元素
```
还可以有多个限定，比如限定要同时可比较和可序列化，就可以使用 `<T extends Comparable & Serializable>`

### 无边界类型通配符<?>

List<?> 表示未知类型的 List 。 这可以是 List<A>，List<B> 或 List<C> 等。
由于不知道 List 的类型，所以只能从集合中读取，不能放入新元素，并且只能将读取的元素视为 Object 类型，如下例：
```java
public void processElements(List<?> elements){
   for(Object o : elements){
      System.out.println(o);
   }
}
```
现在 processElements() 方法可以使用任何泛型参数的 List 集合作为参数，如下：

```java
List<A> listA = new ArrayList<A>();
processElements(listA);
```

### 上界类型通配符<? extends A>
这样将表示可接受的泛型参数，必须是 A 类或者 A 类的子类，A 类就是最大的范围，所以叫做上界。
定义如下方法：
```java
public void processElements(List<? extends A> elements){
   for(A a : elements){
      System.out.println(a.getValue());
   }
}
```
这样，它可以接收 List, List 或 List 集合作为参数，如下：
```java
List<A> listA = new ArrayList<A>();
processElements(listA);

List<B> listB = new ArrayList<B>();
processElements(listB);

List<C> listC = new ArrayList<C>();
processElements(listC);
```
但 ```processElements(List<? extends A> elements)``` 方法仍然无法插入元素到 List 中，因为不知道传入的 List 中的元素的具体类型，它可能是 A, B 或 C 类 ，如果插入成功，那么取出来的时候就无法确认，强制转换就会失败。

但是可以读取元素，因为 List 里面已保存的元素一定是 A 类或其子类对象，转换成 A 类不会报错。

### 下界类型通配符 <? super A>

这样将表示可接受的泛型参数，必须是 A 类（A 类的子类也属于 A 类）或者 A 类的父类，A 类就是最小的范围，所以叫做下界。
当知道 List 中的元素肯定是 A 类或 A 的父类时，可以安全地将 A 类的对象或 A 的子类对象（例如 B 或 C）插入 List 中。 下面是一个例子：

```java
public static void insertElements(List<? super A> list){
    list.add(new A());
    list.add(new B());
    list.add(new C());
}
```
此处插入的所有元素都是 A 类对象或 A 类的父类的对象。由于 B 和 C 都继承了 A ，如果 A 有一个父类，B 和 C 也将是该父类的对象。
现在可以使用 List 或类型为 A 的父类调用`insertElements()`，如下类：

```java
List<A> listA = new ArrayList<A>();
insertElements(listA);

List<Object> listObject = new ArrayList<Object>();
insertElements(listObject);
```
但是，insertElements() 方法无法从 List 中读取元素，除非它将读取到的对象强制转换为 Object 。 调用 insertElements() 时，List 中已经存在的元素可以是 A 类或其父类的任何类型，但不知道它是哪个类。 由于 Java 所有的类都是 Object 的子类，因此如果将它们转换为 Object ，则可以从列表中读取对象。如下代码是正确的：

```java
Object object = list.get(0);
```
但是如下代码是错误的，因为父类对象不能转换为子类对象：

```java
A object = list.get(0);
```

## 泛型擦除
Java 语言中的泛型只在程序源码中存在，在编译后的字节码文件中，就已经替换为原来的原生类型（Raw Type，也称为裸类型）了，并且在相应的地方插入了强制转型代码，因此，对于运行期的 Java 语言来说，ArrayList＜int＞与 ArrayList＜String＞就是同一个类，所以泛型技术实际上是 Java 语言的一颗语法糖，这就是泛型擦除，基于这种方法实现的泛型称为伪泛型。

Java 的泛型是伪泛型，仅仅提供编译时检查，泛型确保了只要在编译时不出现错误，运行时就不出现强制转换异常。在编译完成后，所有的泛型信息都会被擦除掉，泛型附带的类型信息对 JVM 是不可见的。

### 当泛型遇见重载
定义 2 个分别处理 List 和 List 元素的 processElements() 函数，代码如下：

```java
public void processElements(List<String> elements) {
}
public void processElements(List<Integer> elements) {
}
```
编译器会报如下错误：
Error:(12, 17) java: 名称冲突: processElements(java.util.List<java.lang.Integer>)和processElements(java.util.List<java.lang.String>)具有相同疑符

意思就是我们定义了 2 个重复的方法，即两个方法的特征签名（方法简单名和方法中各个参数在常量池中的字段符号引用的集合）是一样的，说明虽然泛型参数不一样，但到底 List 和 List 是一个东西，它们的原始类型是一样的，都是同一个 List 类对象，可以使用如下代码验证：

```java
Class strListClass = new ArrayList<String>().getClass();
Class intListClass = new ArrayList<Integer>().getClass();
System.out.println(strListClass); // class java.util.ArrayList
System.out.println(intListClass); // class java.util.ArrayList
System.out.println(strListClass == intListClass); // ture
```
那么把其中一个方法的返回值修改成和另一个不一样，可以重载成功吗？
```java
public String processElements(List<String> elements) {
}
```
很明显，也会提示一样的报错，无法通过编译，因为方法返回值是不参与重载选择的，那为什么呢？

因为调用一个函数，并不一定需要接受它返回值，如果可以根据返回值重载，那么调用 `processElements(list);` 时，编译器并不知道期望返回的值是什么，因为我们根本就不需要返回值，所以编译器就不知道具体要调用哪个方法。
了解虚拟机的朋友可能知道，从 Class 文件方法表（method_info）的数据结构来看，方法重载要求方法具备不同的特征签名，返回值并不包含在方法的特征签名之中，所以返回值不参与重载选择。

原始类型（raw type）就是擦除（crased）掉泛型信息后的真正类型，泛型参数信息会被擦除，运行时最终使用 Object 或具体的限定类。
原始类型（raw type）就是擦除（crased）掉泛型信息后的真正类型，泛型参数信息会被擦除，运行时最终使用 Object 或具体的限定类。如`Class strListClass = new ArrayList<String>().getClass();` 编译后悔变成
`Class strListClass = new ArrayList().getClass();`

## 其他

JDK 文档中经常能看到 T、K、V、E、N 等类型参数，实际上这些符号随意选择使用也是没问题的，甚至可以使用如 A、B 等，但建议和 JDK 文档风格保持一致，至少得让人容易理解。
常见各符号的含义：
T：type
K：key
V：value
E：element
N：Number

## 泛型数组
