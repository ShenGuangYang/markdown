# 执行的六大原则

> 1. 父类：静态成员赋值，静态代码块
> 2. 子类：静态成员赋值，静态代码块
> 3. 父类：实例成员赋值，实例代码块
> 4. 父类：构造器中的其他方法
> 5. 子类：实例成员赋值，实例代码块
> 6. 子类：构造器中的其他方法



# 下面程序会输出什么？

```java

public class Happy{
    static{
        System.out.println("Happy");
    }
    public static void main(String[] args){
        System.out.println("(1)");
        new D();
    }
}


class E{
    E(){
        System.out.println("(2)");
    }
    public void funcOfE(){
        System.out.println("(3)");
    }
}

class F{
    F(){
        System.out.println("(4)");
    }
    public void funcOfF(){
        System.out.println("(5)");
    }
}

class B{
    E e=new E();
    static F f=new F();
    public String sb=getSb();
    static{
        System.out.println("(6)");
        f.funcOfF();
    }
    {
        System.out.println("(7)");
    }
    B(){
        System.out.println("(8)");
        e.funcOfE();
    }
    public String getSb(){
        System.out.println("(9)");
        return "sb";
    }
}

class C extends B{
    static{
        System.out.println("(10)");
    }
    {
        System.out.println("(11)");
    }

    C(){
        System.out.println("(12)");
    }
}

class D extends C{
    public String sd1=getSd1();
    public static String sd=getSd();
    static{
        System.out.println("(13)");
    }
    {
        System.out.println("(14)");
    }
    D(){
        System.out.println("(15)"+sb+" "+sd+" "+sd1);
    }
    static public String getSd(){
        System.out.println("(16)");
        return "sd";
    }
    public String getSd1(){
        System.out.println("(17)");
        return "sd1";
    }
}
```



# 运行结构

```
Happy
(1)
(4)
(6)
(5)
(10)
(16)
(13)
(2)
(9)
(7)
(8)
(3)
(11)
(12)
(17)
(14)
(15)sb sd sd1
```

