# lambda

## Stream api的简单使用

```java
public class Demo {

    private static final Trader raoul = new Trader("Raoul", "Cambridge");
    private static final Trader mario = new Trader("Mario", "Milan");
    private static final Trader alan = new Trader("Alan", "Cambridge");
    private static final Trader brian = new Trader("Brian", "Cambridge");

    private static List<Transaction> transactions = Arrays.asList(
            new Transaction(brian, 2011, 300),
            new Transaction(raoul, 2012, 1000),
            new Transaction(raoul, 2011, 400),
            new Transaction(mario, 2012, 710),
            new Transaction(mario, 2012, 700),
            new Transaction(alan, 2012, 950)
    );

    public static void main(String[] args) {
        //找出2011年的所有交易并按交易额排序（从高到低）
        List<Transaction> orderList = transactions.stream().filter(x -> x.getYear() == 2011)
                .sorted(Comparator.comparing(Transaction::getValue, Comparator.reverseOrder()))
                .collect(Collectors.toList());
        //交易员都在哪些不同的城市工作过
        Set<String> cityList_1 = transactions.stream()
                .map(x -> x.getTrader().getCity())
                .collect(toSet());
        List<String> cityList_2 = transactions.stream()
                .map(x -> x.getTrader().getCity())
                .distinct()
                .collect(Collectors.toList());
        //查找所有来自于剑桥的交易员，并按姓名排序(排序)
        List<Trader> cambridge = transactions.stream()
                .filter(x -> "Cambridge".equals(x.getTrader().getCity()))
                .map(Transaction::getTrader)
                .sorted(Comparator.comparing(Trader::getName))
                .collect(Collectors.toList());
        //返回所有交易员的姓名字符串，按字母顺序排序(排序、拼接)
        String names = transactions.stream()
                .map(x -> x.getTrader().getName())
                .distinct()
                .sorted()
                .collect(joining(","));
        // names = Alan,Brian,Mario,Raoul

        //有没有交易员是在米兰工作的(筛选)
        boolean b = transactions.stream()
                .anyMatch(x -> "Milan".equals(x.getTrader().getCity()));

        //打印生活在剑桥的交易员的总计交易额(求综合)
        Integer sum = transactions.stream()
                .filter(x -> "Cambridge".equals(x.getTrader().getCity()))
                .map(Transaction::getValue)
                .reduce(0, Integer::sum);

        //所有交易中，最高的交易额是多少(寻找最大值)
        Integer max = transactions.stream() // 速度慢
                .map(Transaction::getValue)
                .reduce(0, Integer::max);

        Optional<Transaction> max_1 = transactions.stream() // 速度快
                .max(Comparator.comparing(Transaction::getValue));

        // 交易在哪几年进行 (分组)
        Map<Integer, List<Transaction>> groupByYear = transactions.stream().collect(groupingBy(Transaction::getYear));

        // 按年份分组，并每个分组的数量
        Map<Integer, Long> groupByCount = transactions.stream().collect(groupingBy(Transaction::getYear, counting()));

    }
}


@AllArgsConstructor
@Data
@ToString
public class Trader {

    private final String name;
    private final String city;

}

@Data
@AllArgsConstructor
@ToString
public class Transaction {
    private final Trader trader;
    private final int year;
    private final int value;
}

```



## 高效使用并行流

1. 留意装箱。自动装箱和拆箱操作会大大降低性能。

2. 有些操作本身在并行流上的性能就比顺序流差。特别是limit和findFirst等依赖于元 

   素顺序的操作，它们在并行流上执行的代价非常大。

3. 较小的数据量，选择并行流几乎从来都不是一个好的决定。

4. 要考虑流背后的数据结构是否易于分解。（ArrayList > HashSet、TreeSet > LinkedList、Stream.iterate >  IntStream.range）

5. 在不确定就进行测试



# 函数式接口

## Predicate

Predicate<T>接口定义了一个名叫test的抽象方法，它接受泛型 T 对象，并返回一个boolean。这恰恰和你先前创建的一样，现在就可以直接使用了。**在你需要表示一个涉及类型T的布尔表达式时，就可以使用这个接口。**
比如，你可以定义一个接受String 对象的Lambda表达式，如下所示。

```java
public class lambda1 {

    public static <T> List<T> filter(List<T> list, Predicate<T> p) {
        List<T> results = new ArrayList<>();
        for (T s : list) {
            if (p.test(s)) {
                results.add(s);
            }
        }
        return results;
    }

    public static void main(String[] args) {
        Predicate<String> nonEmptyStringPredicate = (String s) -> !s.isEmpty();
        List<String> list = Arrays.asList("a","","b","","c","");
        List<String> nonEmpty = filter(list, nonEmptyStringPredicate);
        System.out.println(nonEmpty);
    }

}
```

## Consumer
Consumer<T>定义了一个名叫accept的抽象方法，它接受泛型 T 的对象，没有返回（void）。**你如果需要访问类型T的对象，并对其执行某些操作，就可以使用这个接口。**
比如，你可以用它来创建一个forEach方法，接受一个Integers的列表，并对其中每个元素执行操作。在下面的代码中，你就可以使用这个forEach方法，并配合Lambda来打印列表中的所有元素。

```java
public class lambda2 {
    public static <T> void forEach(List<T> list, Consumer<T> c) {
        for (T t : list) {
            c.accept(t);
        }
    }
    public static void main(String[] args) {
        forEach(Arrays.asList(1, 2, 3, 4, 5, 6), System.out::println);
    }
}

```
## Function

Function<T, R>接口定义了一个叫作apply的方法，它接受一个泛型 T 的对象，并返回一个泛型 R 的对象。**如果你需要定义一个Lambda，将输入对象的信息映射到输出，就可以使用这个接口（比如提取苹果的重量，或把字符串映射为它的长度）。**
在下面的代码中，我们向你展示如何利用它来创建一个map方法，以将一个String列表映射到包含每个String长度的Integer列表。
```java
public class lambda3 {

    public static <T, R> List<R> map(List<T> list, Function<T, R> f) {
        List<R> result = new ArrayList<>();
        for (T t : list) {
            result.add(f.apply(t));
        }
        return result;
    }

    public static void main(String[] args) {
        List<Integer> list = map(Arrays.asList("java8", "lambda", "study"),
                (String s) -> s.length());

        System.out.println(list);
    }
}
```

## Supplier

Supplier<T>接口定义了一个叫作get的方法，它接受一个泛型 T 的对象，并返回一个泛型 T 的对象。如果你需要定义一个Lambda，**创建一个对象**，就可以使用这个接口。
```java
public class Lambda {
    public static void main(String[] args) throws Exception {
        Supplier<Apple> supplier = () -> new Apple();
        Apple apple = supplier.get();
    }
}
class Apple {
    private String color;
    private int weight;

    public int getWeight() {
        return weight;
    }

    public void setWeight(int weight) {
        this.weight = weight;
    }

    public String getColor() {

        return color;
    }

    public void setColor(String color) {
        this.color = color;
    }
}
```

