# Java ����ѧϰ�ܽ�

## ǰ��

Java 5 ����˷��ͣ��ṩ�˱���ʱ���Ͱ�ȫ�����ƣ��û����������Ա�ڱ���ʱ��⵽�Ƿ������͡�
���͵ı����ǲ��������ͣ�����Ϊ��ǰ����ͨ�ö������ͷ�����ָ������Ķ������͡��������е�����������ǽ����Ͽ�һЩ�������ڼ����ϵ����ӣ�

## ���ͼ���

�ȿ�һ��û�з��͵ļ������ӣ�
```Java
List list = new ArrayList();
list.add(new Integer(2));
list.add("a String");
```
��Ϊ List ��������κζ������Դ� List ��ȡ���Ķ���ʱ����Ϊ��ȷ����List����סԪ�����ͣ���ʱ�򱣴�� List ��Ԫ�����ͣ�������������ֻ���� Object ���������ɳ����д�߼�ס���Ԫ�����ͣ�Ȼ��ȡ��ʱ�ٽ���ǿ������ת���ͣ����£�

```Java
Integer integer = (Integer) list.get(0);
String string   = (String) list.get(1);
```

ͨ��������ֻʹ�ô��е�һ����Ԫ�صļ��ϣ����Ҳ�ϣ���������͵Ķ�����ӵ������У����磬ֻ�� Integer �� List ����ϣ���� String ����Ž����ϣ�����Ҳ�����Լ���סԪ�����ͣ�����ǿ������ת�������ܻ���ִ���

ʹ�� Generics�����ͣ��Ϳ������ü��ϵ����ͣ������ƿ��Խ����ֶ�����뼯���С� �����ȷ�������е�Ԫ�أ�����ͬһ����֪���͵ģ����ȡ�����ݵ�ʱ��Ͳ��ؽ���ǿ������ת���ˣ�������һ�� String ���͵� List ��ʹ�����ӣ�

```Java
List<String> strings = new ArrayList<String>();
strings.add("a String");
String aString = strings.get(0);
```

������� List ����ֻ�ܷ��� String ���������ͼ�����Ķ�����ô�������ᱨ���ô����д�߱�����д�������Ƕ�������ͼ�顣����ע��List �� <> ����������ֻ���Ƕ����������ͣ��������������ͣ�����ʹ�ö�Ӧ��װ����棩��
Java ���ʹ� Java 7 ��ʼ�������������Զ������жϣ�����ʡ�Թ������еķ��ͣ�������һ��Java 7 �������ӣ�

```Java
List<String> strings = new ArrayList<>();
```

��Ҳ���������﷨��<>���� �������ʾ���У�ʵ���� ArrayList ��ʱ�򣬱���������ǰ�� List ֪�� new �� ArrayList ������Ϣ�� String��
foreach ѭ�����Ժܺõ��뷺�ͼ������ϣ����£�

```java
List<String> strings = new ArrayList<>();

// ����ʡ�Խ� String Ԫ����ӽ����ϵĴ���...

for(String aString : strings){
  System.out.println(aString);
}
```
Ҳ����ʹ�÷��͵������������ϣ����£�

```java
List<String> list = new ArrayList<>;

Iterator<String> iterator = list.iterator();

while(iterator.hasNext()){
  String aString = iterator.next();
}
```
ע�⣬�������ͼ����ڱ���ʱ���ڡ�������ʱ������ʹ�÷������������ʽʹ�ַ������ϲ����������󣬵�һ�㲻����������
��Ȼ���͵��ô����������ڼ��ϡ�

## ������
������������У��������˽ⷺ�͵Ĵ�����
�������Զ��� Java ����ʹ�÷��ͣ����������� Java API �е�Ԥ�����ࡣ���巺����ֻ��Ҫ����������� �����ţ����� T �����ͱ�ʶ����Ҳ�����Ǳ�ģ����� E ��V �ȣ���������

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
���� Class.newInstance() �����Ƿ���֪ʶ�����ݣ�����ֻҪ֪���˷������� theClass �����Ĵ������С�
T ��һ�����ͱ�ǣ���ʾ�����������ʵ����ʱ����ӵ�е����ͼ���������һ�����ӣ�

```java
GenericFactory<MyClass> factory = new GenericFactory<MyClass>(MyClass.class);
MyClass myClassInstance = factory.createInstance();

GenericFactory<SomeObject> factory1 = new GenericFactory<SomeObject>(SomeObject.class);
SomeObject someObjectInstance = factory1.createInstance();
```
ʹ�÷��ͣ����ǾͲ���ת���� factory.createInstance() �������صĶ��󣬻��Զ����� new GenericFactory �� <> �������е����Ͷ���

## ���ͷ���

һ�����ͷ����������£�
```java
public static <T> T addAndReturn(T element, Collection<T> collection){
    collection.add(element);
    return element;
}
```
���� T �Ƿ�������ֵ�� ���ڷ�������ֵ֮ǰ�������з��ͷ��������е����Ͳ�����������ʾ����һ�����ͷ��������û�� ֻ�� T ���ǾͲ��Ƿ��ͷ��������Ƿ������һ����Ա�������������淺���ඨ��ķ�����

```java
public T createInstance() throws Exception { ... }
```

�ⲻ��һ�����ͷ��������� GenericFactory�������һ����Ա�������ѣ����ͷ��������ڷ�������ֵ֮ǰ������ �ı�ǡ�
ע���ڷ��ͷ����У������� 2 ������һ���� T ���͵Ĳ������ڶ�����Ԫ������Ϊ T �� Collection ���ϣ������������ʵ�ʲ������ƶ� T Ϊ�������� ��������������ȫ���еģ�

```java
String stringElement = "stringElement";
List<Object> objectList = new ArrayList<Object>();

Object theElement = addAndReturn(stringElement, objectList);
```

���ϣ���һ������Ϊ String ���ͣ��ڶ����� List ���ͣ�ò������ T �ǲ�ƥ��ģ�����������������Զ��� String ת��Ϊ Object ���ͣ����� T ��ʶΪ Object ��

## �̳й�ϵ�ķ��Ͳ���

���������ࣺ

```java
public class A { }
public class B extends A { }
public class C extends A { }
```

�½�����ʼ�����������
```java
List<A> listA = new ArrayList<A>();
List<B> listB = new ArrayList<B>();
```
��Ϊ A �� B �� C ��Ĺ�ͬ���࣬��ô ListA �ǲ��� ListB �� ListC �Ĺ�ͬ�����𣿻��仰˵���Ƿ���Խ� List ����ָ�� List �����أ����߷������� List ����ָ�� List �����أ�

���ǲ����ԣ���һ���������������ָ����������罫 ListA ����ָ�� ListB �����ȼ������ǿ��Եģ������´��룺
```java
List<A> listA = listB;
```
���������⣬��Ϊ listA �� List ���ͣ������Է����κ� A ������ΪԪ�أ����Է��� B ����Ҳ���Է��� C ���󣬵�����ʵ��ָ�����һ�� List ���ϣ�List �����б�Ӧ��ֻ�洢 B �����ģ�����ȴ����Ԥ�ϵط����� C ������͵����Ķ��岻��� `List<B> listB = new ArrayList<B>();` ��������������ǲ�����ġ�

�ڶ���������� List ����ָ�� List ���������϶�Ҳ�ǲ����Եģ���Ϊ�Ѵ��ڵ� List �����У������Ѿ������� A ��ķ� B ����������� C ���������Ҳ���ܱ�֤�� List ������ȡ����Ԫ��һ������ B ����

���ǣ�ȷʵ������Ҫʹ�÷��ͼ̳й�ϵ������������´��룺

```java
public void processElements(List<A> elements){
   for(A o : elements){
      System.out.println(o.getValue()); // �������� A ����� getValue() ����
   }
}
```

����ϣ������һ���࣬�����ܴ�������ָ������Ԫ�ص� List ���� processElements(List elements) �������ȿ��Դ��� List ��Ҳ���Դ��� List ���������������� List �� List �����Ͳ������κμ̳й�ϵ��
��ʱ�Ϳ���ʹ������ͨ����� <?> ����ɴ�����

## ����ͨ���
�� 3 ������ͨ����÷������´��룺
```java
List<?> listUknown = new ArrayList<A>(); // �κ�����Ԫ�ض��ɽ���
List<? extends A> listUknown = new ArrayList<A>(); // �ɽ��� A ��������Ԫ��
List<? super A> listUknown = new ArrayList<A>(); // �ɽ��� A ��������Ԫ��
```
�������ж���޶��������޶�Ҫͬʱ�ɱȽϺͿ����л����Ϳ���ʹ�� `<T extends Comparable & Serializable>`

### �ޱ߽�����ͨ���<?>

List<?> ��ʾδ֪���͵� List �� ������� List<A>��List<B> �� List<C> �ȡ�
���ڲ�֪�� List �����ͣ�����ֻ�ܴӼ����ж�ȡ�����ܷ�����Ԫ�أ�����ֻ�ܽ���ȡ��Ԫ����Ϊ Object ���ͣ���������
```java
public void processElements(List<?> elements){
   for(Object o : elements){
      System.out.println(o);
   }
}
```
���� processElements() ��������ʹ���κη��Ͳ����� List ������Ϊ���������£�

```java
List<A> listA = new ArrayList<A>();
processElements(listA);
```

### �Ͻ�����ͨ���<? extends A>
��������ʾ�ɽ��ܵķ��Ͳ����������� A ����� A ������࣬A ��������ķ�Χ�����Խ����Ͻ硣
�������·�����
```java
public void processElements(List<? extends A> elements){
   for(A a : elements){
      System.out.println(a.getValue());
   }
}
```
�����������Խ��� List, List �� List ������Ϊ���������£�
```java
List<A> listA = new ArrayList<A>();
processElements(listA);

List<B> listB = new ArrayList<B>();
processElements(listB);

List<C> listC = new ArrayList<C>();
processElements(listC);
```
�� ```processElements(List<? extends A> elements)``` ������Ȼ�޷�����Ԫ�ص� List �У���Ϊ��֪������� List �е�Ԫ�صľ������ͣ��������� A, B �� C �� ���������ɹ�����ôȡ������ʱ����޷�ȷ�ϣ�ǿ��ת���ͻ�ʧ�ܡ�

���ǿ��Զ�ȡԪ�أ���Ϊ List �����ѱ����Ԫ��һ���� A ������������ת���� A �಻�ᱨ��

### �½�����ͨ��� <? super A>

��������ʾ�ɽ��ܵķ��Ͳ����������� A �ࣨA �������Ҳ���� A �ࣩ���� A ��ĸ��࣬A �������С�ķ�Χ�����Խ����½硣
��֪�� List �е�Ԫ�ؿ϶��� A ��� A �ĸ���ʱ�����԰�ȫ�ؽ� A ��Ķ���� A ������������� B �� C������ List �С� ������һ�����ӣ�

```java
public static void insertElements(List<? super A> list){
    list.add(new A());
    list.add(new B());
    list.add(new C());
}
```
�˴����������Ԫ�ض��� A ������ A ��ĸ���Ķ������� B �� C ���̳��� A ����� A ��һ�����࣬B �� C Ҳ���Ǹø���Ķ���
���ڿ���ʹ�� List ������Ϊ A �ĸ������`insertElements()`�������ࣺ

```java
List<A> listA = new ArrayList<A>();
insertElements(listA);

List<Object> listObject = new ArrayList<Object>();
insertElements(listObject);
```
���ǣ�insertElements() �����޷��� List �ж�ȡԪ�أ�����������ȡ���Ķ���ǿ��ת��Ϊ Object �� ���� insertElements() ʱ��List ���Ѿ����ڵ�Ԫ�ؿ����� A ����丸����κ����ͣ�����֪�������ĸ��ࡣ ���� Java ���е��඼�� Object �����࣬������������ת��Ϊ Object ������Դ��б��ж�ȡ�������´�������ȷ�ģ�

```java
Object object = list.get(0);
```
�������´����Ǵ���ģ���Ϊ���������ת��Ϊ�������

```java
A object = list.get(0);
```

## ���Ͳ���
Java �����еķ���ֻ�ڳ���Դ���д��ڣ��ڱ������ֽ����ļ��У����Ѿ��滻Ϊԭ����ԭ�����ͣ�Raw Type��Ҳ��Ϊ�����ͣ��ˣ���������Ӧ�ĵط�������ǿ��ת�ʹ��룬��ˣ����������ڵ� Java ������˵��ArrayList��int���� ArrayList��String������ͬһ���࣬���Է��ͼ���ʵ������ Java ���Ե�һ���﷨�ǣ�����Ƿ��Ͳ������������ַ���ʵ�ֵķ��ͳ�Ϊα���͡�

Java �ķ�����α���ͣ������ṩ����ʱ��飬����ȷ����ֻҪ�ڱ���ʱ�����ִ�������ʱ�Ͳ�����ǿ��ת���쳣���ڱ�����ɺ����еķ�����Ϣ���ᱻ�����������͸�����������Ϣ�� JVM �ǲ��ɼ��ġ�

### ��������������
���� 2 ���ֱ��� List �� List Ԫ�ص� processElements() �������������£�

```java
public void processElements(List<String> elements) {
}
public void processElements(List<Integer> elements) {
}
```
�������ᱨ���´���
Error:(12, 17) java: ���Ƴ�ͻ: processElements(java.util.List<java.lang.Integer>)��processElements(java.util.List<java.lang.String>)������ͬ�ɷ�

��˼�������Ƕ����� 2 ���ظ��ķ���������������������ǩ�������������ͷ����и��������ڳ������е��ֶη������õļ��ϣ���һ���ģ�˵����Ȼ���Ͳ�����һ���������� List �� List ��һ�����������ǵ�ԭʼ������һ���ģ�����ͬһ�� List ����󣬿���ʹ�����´�����֤��

```java
Class strListClass = new ArrayList<String>().getClass();
Class intListClass = new ArrayList<Integer>().getClass();
System.out.println(strListClass); // class java.util.ArrayList
System.out.println(intListClass); // class java.util.ArrayList
System.out.println(strListClass == intListClass); // ture
```
��ô������һ�������ķ���ֵ�޸ĳɺ���һ����һ�����������سɹ���
```java
public String processElements(List<String> elements) {
}
```
�����ԣ�Ҳ����ʾһ���ı����޷�ͨ�����룬��Ϊ��������ֵ�ǲ���������ѡ��ģ���Ϊʲô�أ�

��Ϊ����һ������������һ����Ҫ����������ֵ��������Ը��ݷ���ֵ���أ���ô���� `processElements(list);` ʱ������������֪���������ص�ֵ��ʲô����Ϊ���Ǹ����Ͳ���Ҫ����ֵ�����Ա������Ͳ�֪������Ҫ�����ĸ�������
�˽�����������ѿ���֪������ Class �ļ�������method_info�������ݽṹ��������������Ҫ�󷽷��߱���ͬ������ǩ��������ֵ���������ڷ���������ǩ��֮�У����Է���ֵ����������ѡ��

ԭʼ���ͣ�raw type�����ǲ�����crased����������Ϣ����������ͣ����Ͳ�����Ϣ�ᱻ����������ʱ����ʹ�� Object �������޶��ࡣ
ԭʼ���ͣ�raw type�����ǲ�����crased����������Ϣ����������ͣ����Ͳ�����Ϣ�ᱻ����������ʱ����ʹ�� Object �������޶��ࡣ��`Class strListClass = new ArrayList<String>().getClass();` �����ڱ��
`Class strListClass = new ArrayList().getClass();`

## ����

JDK �ĵ��о����ܿ��� T��K��V��E��N �����Ͳ�����ʵ������Щ��������ѡ��ʹ��Ҳ��û����ģ���������ʹ���� A��B �ȣ�������� JDK �ĵ���񱣳�һ�£����ٵ�����������⡣
���������ŵĺ��壺
T��type
K��key
V��value
E��element
N��Number

## ��������
