## 为什么要使用泛型和迭代器 + 面试题

### 泛型

#### 1）为什么要用泛型？

在泛型没有诞生之前，我们经常会遇到这样的问题，如以下代码所示：

```java
ArrayList arrayList = new ArrayList();
arrayList.add("Java");
arrayList.add(24);
for (int i = 0; i < arrayList.size(); i++) {
    String str = (String) arrayList.get(i);
    System.out.println(str);
}
```

看起来好像没有什么大问题，也能正常编译，但真正运行起来就会报错：

> Exception in thread "main" java.lang.ClassCastException: java.lang.Integer cannot be cast to java.lang.String
>
> at xxx(xxx.java:12)
>
> 类型转换出错，当我们给 ArrayList 放入不同类型的数据，却使用一种类型进行接收的时候，就会出现很多类似的错误，可能更多的时候，是因为开发人员的不小心导致的。那有没有好的办法可以杜绝此类问题的发生呢？这个时候 Java 语言提供了一个很好的解决方案——“泛型”。
>
> 泛型出现的必要：多态出现类型转换时，可能出现ClassCastException,

#### 2）泛型介绍

**泛型**：泛型本质上是类型参数化，解决了不确定对象的类型问题。
泛型的使用，请参考以下代码：

```
ArrayList<String> arrayList = new ArrayList();
arrayList.add("Java");
```

这个时候如果给 arrayList 添加非 String 类型的元素，编译器就会报错，提醒开发人员插入相同类型的元素。

报错信息如下图所示：

![enter image description here](https://images.gitbook.cn/83dce010-cdeb-11e9-932d-6123ff488b55)

这样就可以避免开头示例中，类型不一致导致程序运行过程中报错的问题了。

#### 3）泛型的优点

泛型的优点主要体现在以下三个方面。

- 安全：不用担心程序运行过程中出现类型转换的错误。
- 避免了类型转换：如果是非泛型，获取到的元素是 Object 类型的，需要强制类型转换。
- 可读性高：编码阶段就明确的知道集合中元素的类型。



#### 4) 泛型的使用

##### 1 定义泛型类和泛型接口

```java
//此处T可以随便写为任意标识，常见的如T、E、K、V等形式的参数常用于表示泛型
//在实例化泛型类时，必须指定T的具体类型
public class Generic<T>{ 
    //key这个成员变量的类型为T,T的类型由外部指定  
    private T key;

    public Generic(T key) { //泛型构造方法形参key的类型也为T，T的类型由外部指定
        this.key = key;
    }

    public T getKey(){ //泛型方法getKey的返回值类型为T，T的类型由外部指定
        return key;
    }
}


//泛型的类型参数只能是类类型（包括自定义类），不能是简单类型
//传入的实参类型需与泛型的类型参数类型相同，即为Integer.
Generic<Integer> genericInteger = new Generic<Integer>(123456);

//传入的实参类型需与泛型的类型参数类型相同，即为String.
Generic<String> genericString = new Generic<String>("key_vlaue");
Log.d("泛型测试","key is " + genericInteger.getKey());
Log.d("泛型测试","key is " + genericString.getKey());
```



##### 2 定义泛型数组

> 在java中是”不能创建一个确切的泛型类型的数组”的。

```java
也就是说下面的这个例子是不可以的：
List<String>[] ls = new ArrayList<String>[10];  

而使用通配符创建泛型数组是可以的，如下面这个例子：
List<?>[] ls = new ArrayList<?>[10];  

这样也是可以的：
List<String>[] ls = new ArrayList[10];

```



**举例**

```java
List<String>[] lsa = new List<String>[10]; // Not really allowed.    
Object o = lsa;    
Object[] oa = (Object[]) o;    
List<Integer> li = new ArrayList<Integer>();    
li.add(new Integer(3));    
oa[1] = li; // Unsound, but passes run time store check    
String s = lsa[1].get(0); // Run-time error: ClassCastException.

> 这种情况下，由于JVM泛型的擦除机制，在运行时JVM是不知道泛型信息的，所以可以给oa[1]赋上一个ArrayList而不会出现异常，但是在取出数据的时候却要做一次类型转换，所以就会出现ClassCastException，如果可以进行泛型数组的声明，上面说的这种情况在编译期将不会出现任何的警告和错误，只有在运行时才会出错。

> 而对泛型数组的声明进行限制，对于这样的情况，可以在编译期提示代码有类型安全问题，比没有任何提示要强很多。

> 下面采用通配符的方式是被允许的:数组的类型不可以是类型变量，除非是采用通配符的方式，因为对于通配符的方式，最后取出数据是要做显式的类型转换的。
```



```java
List<?>[] lsa = new List<?>[10]; // OK, array of unbounded wildcard type.    
Object o = lsa;    
Object[] oa = (Object[]) o;    
List<Integer> li = new ArrayList<Integer>();    
li.add(new Integer(3));    
oa[1] = li; // Correct.    
Integer i = (Integer) lsa[1].get(0); // OK 
```



##### 3 使用？通配符

我们知道Ingeter是Number的一个子类，同时在特性章节中我们也验证过Generic<Ingeter>与Generic<Number>实际上是相同的一种基本类型。那么问题来了，在使用Generic<Number>作为形参的方法中，能否使用Generic<Ingeter>的实例传入呢？在逻辑上类似于Generic<Number>和Generic<Ingeter>是否可以看成具有父子关系的泛型类型呢？

```java
public void showKeyValue1(Generic<Number> obj){
    Log.d("泛型测试","key value is " + obj.getKey());
}

Generic<Integer> gInteger = new Generic<Integer>(123);
Generic<Number> gNumber = new Generic<Number>(456);

showKeyValue(gNumber);

// showKeyValue这个方法编译器会为我们报错：Generic<java.lang.Integer> 
// cannot be applied to Generic<java.lang.Number>
// showKeyValue(gInteger);

通过提示信息我们可以看到Generic<Integer>不能被看作为`Generic<Number>的子类。由此可以看出:同一种泛型可以对应多个版本（因为参数类型是不确定的），不同版本的泛型类实例是不兼容的。

回到上面的例子，如何解决上面的问题？总不能为了定义一个新的方法来处理Generic<Integer>类型的类，这显然与java中的多台理念相违背。因此我们需要一个在逻辑上可以表示同时是Generic<Integer>和Generic<Number>父类的引用类型。由此类型通配符应运而生。

我们可以将上面的方法改一下：
public void showKeyValue1(Generic<?> obj){
   Log.d("泛型测试","key value is " + obj.getKey());
}
```

> 类型通配符一般是使用？代替具体的类型实参，注意了，此处’？’是类型实参，而不是类型形参 。重要说三遍！此处’？’是类型实参，而不是类型形参 ！ 此处’？’是类型实参，而不是类型形参 ！再直白点的意思就是，此处的？和Number、String、Integer一样都是一种实际的类型，可以把？看成所有类型的父类。是一种真实的类型。
>
> 可以解决当具体类型不确定的时候，这个通配符就是 ? ；当操作类型时，不需要使用类型的具体功能时，只使用Object类中的功能。那么可以用 ? 通配符来表未知类型。



##### 4 定义泛型方法

1. 泛型方法的基本介绍

   ```java
   * @param tClass 传入的泛型实参
    * @return T 返回值为T类型
    * 说明：
    *     1）public 与 返回值中间<T>非常重要，可以理解为声明此方法为泛型方法。
    *     2）只有声明了<T>的方法才是泛型方法，泛型类中的使用了泛型的成员方法并不是泛型方法。
    *     3）<T>表明该方法将使用泛型类型T，此时才可以在方法中使用泛型类型T。
    *     4）与泛型类的定义一样，此处T可以随便写为任意标识，常见的如T、E、K、V等形式的参数常用于表示泛型。
    */
   public <T> T genericMethod(Class<T> tClass)throws InstantiationException ,
     IllegalAccessException{
           T instance = tClass.newInstance();
           return instance;
   }
   
   Object obj = genericMethod(Class.forName("com.test.test"));
   ```

2. 泛型方法的基本用法

   ```java
   public class GenericTest {
      //这个类是个泛型类，在上面已经介绍过
      public class Generic<T>{     
           private T key;
   
           public Generic(T key) {
               this.key = key;
           }
   
           //我想说的其实是这个，虽然在方法中使用了泛型，但是这并不是一个泛型方法。
           //这只是类中一个普通的成员方法，只不过他的返回值是在声明泛型类已经声明过的泛型。
           //所以在这个方法中才可以继续使用 T 这个泛型。
           public T getKey(){
               return key;
           }
   
           /**
            * 这个方法显然是有问题的，在编译器会给我们提示这样的错误信息"cannot reslove symbol E"
            * 因为在类的声明中并未声明泛型E，所以在使用E做形参和返回值类型时，编译器会无法识别。
           public E setKey(E key){
                this.key = keu
           }
           */
       }
   
       /** 
        * 这才是一个真正的泛型方法。
        * 首先在public与返回值之间的<T>必不可少，这表明这是一个泛型方法，并且声明了一个泛型T
        * 这个T可以出现在这个泛型方法的任意位置.
        * 泛型的数量也可以为任意多个 
        *    如：public <T,K> K showKeyName(Generic<T> container){
        *        ...
        *        }
        */
       public <T> T showKeyName(Generic<T> container){
           System.out.println("container key :" + container.getKey());
           //当然这个例子举的不太合适，只是为了说明泛型方法的特性。
           T test = container.getKey();
           return test;
       }
   
       //这也不是一个泛型方法，这就是一个普通的方法，只是使用了Generic<Number>这个泛型类做形参而已。
       public void showKeyValue1(Generic<Number> obj){
           Log.d("泛型测试","key value is " + obj.getKey());
       }
   
       //这也不是一个泛型方法，这也是一个普通的方法，只不过使用了泛型通配符?
       //同时这也印证了泛型通配符章节所描述的，?是一种类型实参，可以看做为Number等所有类的父类
       public void showKeyValue2(Generic<?> obj){
           Log.d("泛型测试","key value is " + obj.getKey());
       }
   
        /**
        * 这个方法是有问题的，编译器会为我们提示错误信息："UnKnown class 'E' "
        * 虽然我们声明了<T>,也表明了这是一个可以处理泛型的类型的泛型方法。
        * 但是只声明了泛型类型T，并未声明泛型类型E，因此编译器并不知道该如何处理E这个类型。
       public <T> T showKeyName(Generic<E> container){
           ...
       }  
       */
   
       /**
        * 这个方法也是有问题的，编译器会为我们提示错误信息："UnKnown class 'T' "
        * 对于编译器来说T这个类型并未项目中声明过，因此编译也不知道该如何编译这个类。
        * 所以这也不是一个正确的泛型方法声明。
       public void showkey(T genericObj){
   
       }
       */
   
       public static void main(String[] args) {
   
   
       }
   }
   ```

3. 类中的泛型方法

   ```java
   public class GenericFruit {
       class Fruit{
           @Override
           public String toString() {
               return "fruit";
           }
       }
   
       class Apple extends Fruit{
           @Override
           public String toString() {
               return "apple";
           }
       }
   
       class Person{
           @Override
           public String toString() {
               return "Person";
           }
       }
   
       class GenerateTest<T>{
           public void show_1(T t){
               System.out.println(t.toString());
           }
   
           //在泛型类中声明了一个泛型方法，使用泛型E，这种泛型E可以为任意类型。可以类型与T相同，也可以不同。
           //由于泛型方法在声明的时候会声明泛型<E>，因此即使在泛型类中并未声明泛型，编译器也能够正确识别泛型方法中识别的泛型。
           public <E> void show_3(E t){
               System.out.println(t.toString());
           }
   
           //在泛型类中声明了一个泛型方法，使用泛型T，注意这个T是一种全新的类型，可以与泛型类中声明的T不是同一种类型。
           public <T> void show_2(T t){
               System.out.println(t.toString());
           }
       }
   
       public static void main(String[] args) {
           Apple apple = new Apple();
           Person person = new Person();
   
           GenerateTest<Fruit> generateTest = new GenerateTest<Fruit>();
           //apple是Fruit的子类，所以这里可以
           generateTest.show_1(apple);
           //编译器会报错，因为泛型类型实参指定的是Fruit，而传入的实参类是Person
           //generateTest.show_1(person);
   
           //使用这两个方法都可以成功
           generateTest.show_2(apple);
           generateTest.show_2(person);
   
           //使用这两个方法也都可以成功
           generateTest.show_3(apple);
           generateTest.show_3(person);
       }
   }
   ```

4. 泛型方法与可变参数

   ```java
   public <T> void printMsg( T... args){
       for(T t : args){
           Log.d("泛型测试","t is " + t);
       }
   }
   ```

5. 静态方法与泛型

   > 静态方法有一种情况需要注意一下，那就是在类中的静态方法使用泛型：静态方法无法访问类上定义的泛型；如果静态方法操作的引用数据类型不确定的时候，必须要将泛型定义在方法上。

   ```java
   即：如果静态方法要使用泛型的话，必须将静态方法也定义成泛型方法 。
   
   public class StaticGenerator<T> {
       ....
       ....
       /**
        * 如果在类中定义使用泛型的静态方法，需要添加额外的泛型声明（将这个方法定义成泛型方法）
        * 即使静态方法要使用泛型类中已经声明过的泛型也不可以。
        * 如：public static void show(T t){..},此时编译器会提示错误信息：
             "StaticGenerator cannot be refrenced from static context"
        */
       public static <T> void show(T t){
   
       }
   }
   ```



##### 5 泛型注意事项

1 实例共享

Array<String>  与 Array<Integer>  共享同一个运行时类

2 不允许一个类有两个同名的方法

3 

```java
if  (cs instance Collection<string>) {....}  

// 编译出错 Collection cs = new ArrayList<String>(); 

if (cs instanceof Collection<?>) {....}   // 通过编译
```

 

4 不能使用泛型类型来进行强制类型转换

5 泛型只在编译期有效

6 泛型类型在逻辑上看成多个不同类型，但实质都是同一个数据类型，编译之后会采取去泛型化类型

7 每个泛型都有其原生态类型

如List<E> 的原生类型就是List 存在目的是为了相兼容

List<String> 是List的子类型，而不是List<Object>（参数化类型）的子类型

由于泛型在运行期间可以被擦除，所以List<String.class> 和 List<?>.class是非法的





### 迭代器（Iterator）

#### 1）为什么要用迭代器？

我们回想一下，在迭代器（Iterator）没有出现之前，如果要遍历数组和集合，需要使用方法。

数组遍历，代码如下：

```java
String[] arr = new String[]{"Java", "Java虚拟机", "Java中文社群"};
for (int i = 0; i < arr.length; i++) {
    String item = arr[i];
}
```

集合遍历，代码如下：

```java
List<String> list = new ArrayList<String>() {{
    add("Java");
    add("Java虚拟机");
    add("Java中文社群");
}};
for (int i = 0; i < list.size(); i++) {
    String item = list.get(i);
}
```

而迭代器的产生，就是为不同类型的容器遍历，提供标准统一的方法。

迭代器遍历，代码如下：

```java
Iterator iterator = list.iterator();
while (iterator.hasNext()) {
    Object object = iterator.next();
    // do something
}
```

**总结**：使用了迭代器就可以不用关注容器的内部细节，用同样的方式遍历不同类型的容器。

#### 2）迭代器介绍

迭代器是用来遍历容器内所有元素对象的，也是一种常见的设计模式。

迭代器包含以下四个方法。

- hasNext():boolean —— 容器内是否还有可以访问的元素。
- next():E —— 返回下一个元素。
- remove():void —— 删除当前元素。
- forEachRemaining(Consumer):void —— JDK 8 中添加的，提供一个 lambda 表达式遍历容器元素。

迭代器使用如下：

```java
List<String> list = new ArrayList<String>() {{
    add("Java");
    add("Java虚拟机");
    add("Java中文社群");
}};
Iterator iterator =  list.iterator();
// 遍历
while (iterator.hasNext()){
    String str = (String) iterator.next();
    if (str.equals("Java中文社群")){
        iterator.remove();
    }
}
System.out.println(list);
```

程序执行结果：

```
[Java, Java虚拟机]
```

forEachRemaining 使用如下：

```java
List<String> list = new ArrayList<String>() {{
    add("Java");
    add("Java虚拟机");
    add("Java中文社群");
}};
// forEachRemaining 使用
list.iterator().forEachRemaining(item -> System.out.println(item));
```

### 相关面试题

#### 1.为什么迭代器的 next() 返回的是 Object 类型？

答：因为迭代器不需要关注容器的内部细节，所以 next() 返回 Object 类型就可以接收任何类型的对象。

#### 2.HashMap 的遍历方式都有几种？

答：HashMap 的遍历分为以下四种方式。

- 方式一：entrySet 遍历
- 方式二：iterator 遍历
- 方式三：遍历所有的 key 和 value
- 方式四：通过 key 值遍历

以上方式的代码实现如下：

```java
Map<String, String> hashMap = new HashMap();
hashMap.put("name", "老王");
hashMap.put("sex", "你猜");
// 方式一：entrySet 遍历
for (Map.Entry item : hashMap.entrySet()) {
  System.out.println(item.getKey() + ":" + item.getValue());
}
// 方式二：iterator 遍历
Iterator<Map.Entry<String, String>> iterator = hashMap.entrySet().iterator();
while (iterator.hasNext()) {
  Map.Entry<String, String> entry = iterator.next();
  System.out.println(entry.getKey() + ":" + entry.getValue());
}
// 方式三：遍历所有的 key 和 value
for (Object k : hashMap.keySet()) {
  // 循环所有的 key
  System.out.println(k);
}
for (Object v : hashMap.values()) {
  // 循环所有的值
  System.out.println(v);
}
// 方式四：通过 key 值遍历
for (Object k : hashMap.keySet()) {
  System.out.println(k + ":" + hashMap.get(k));
}
```

#### 3.以下关于泛型说法错误的是？

A：泛型可以修饰类
B：泛型可以修饰方法
C：泛型不可以修饰接口
D：以上说法全错

答：选 C，泛型可以修饰类、方法、接口、变量。
例如：

```
public interface Iterable<T> {
}
```

#### 4.以下程序执行的结果是什么？

```java
List<String> list = new ArrayList<>();
List<Integer> list2 = new ArrayList<>();
System.out.println(list.getClass() == list2.getClass());
```

答：程序的执行结果是 `true`。
题目解析：Java 中泛型在编译时会进行类型擦除，因此 `List<String> list` 和 `List<Integer> list2` 类型擦除后的结果都是 java.util.ArrayLis ，进而 list.getClass() == list2.getClass() 的结果也一定是 true。

#### 5. `List<Object>` 和 `List<?>` 有什么区别？

答：`List<?>` 可以容纳任意类型，只不过 `List<?>` 被赋值之后，就不允许添加和修改操作了；而 `List<Object>` 和 `List<?>` 不同的是它在赋值之后，可以进行添加和修改操作，如下图所示：

![enter image description here](https://images.gitbook.cn/b3a90d00-cdeb-11e9-932d-6123ff488b55)

#### 6.可以把 `List<String>` 赋值给 `List<Object>` 吗？

答：不可以，编译器会报错，如下图所示：

![enter image description here](https://images.gitbook.cn/cb2aa8d0-cdeb-11e9-b572-5118f14310d8)

#### 7. `List` 和 `List<Object>` 的区别是什么？

答： `List` 和 `List<Object>` 都能存储任意类型的数据，但 `List` 和 `List<Object>` 的唯一区别就是，`List` 不会触发编译器的类型安全检查，比如把 `List<String>` 赋值给 `List` 是没有任何问题的，但赋值给 `List<Object>` 就不行，如下图所示：

![enter image description here](https://images.gitbook.cn/e34947f0-cdeb-11e9-932d-6123ff488b55)

#### 8.以下程序执行的结果是？

```java
List<String> list = new ArrayList<>();
list.add("Java");
list.add("Java虚拟机");
list.add("Java中文社群");
Iterator iterator = list.iterator();
while (iterator.hasNext()) {
    String str = (String) iterator.next();
    if (str.equals("Java中文社群")) {
        iterator.remove();
    }
}
while (iterator.hasNext()) {
    System.out.println(iterator.next());
}
System.out.println("Over");
```

答：程序打印结果是 `Over`。
题目解析：因为第一个 while 循环之后，iterator.hasNext() 返回值就为 false 了，所以不会进入第二个循环，之后打印最后的 Over。

#### 9.泛型的工作原理是什么？为什么要有类型擦除？

答：泛型是通过类型擦除来实现的，类型擦除指的是编译器在编译时，会擦除了所有类型相关的信息，比如 `List<String>` 在编译后就会变成 `List` 类型，这样做的目的就是确保能和 Java 5 之前的版本（二进制类库）进行兼容。

### 总结

通过本文知道了泛型的优点：安全性、避免类型转换、提高了代码的可读性。泛型的本质是类型参数化，但编译之后会执行类型擦除，这样就可以和 Java 5 之前的二进制类库进行兼容。本文也介绍了迭代器（Iterator）的使用，使用迭代器的好处是不用关注容器的内部细节，用同样的方式遍历不同类型的容器。