

## 泛型

### 泛型的定义（什么是）

>  泛型是一种数据类型，参数化类型，泛型要求包容的是对象类型，不能是基本类型

### 泛型的功能（为什么用）

> 一般的类和方法，只能使用具体的类型，要么是基本类型要么是自定义的类，如果要编写可以应用于多种类型的代码，可以用泛型
>
> 泛型具有可重用，类型安全，效率等特性

#### 用和不用有什么区别

##### 原生类型（不用）

> 原生类型：List，不会进行泛型检查，编译期间不会进行类型检查，会导致运行期间出现类型转换的错误
>
> 泛型的父类都是其原生类型，并且在编译期间之后，去泛型化(为了使得与java5之前没有使用泛型的代码进行互用)

##### 泛型（使用）

| 类型             | 范例                               | 说明                                                         |
| ---------------- | ---------------------------------- | ------------------------------------------------------------ |
| 参数化类型       | `List<String>`                     | 进行泛型检查，统一规定类型，告知编译器能够持有任意类型的对象，避免出现运行期间类型转换错误 |
| 实际类型参数     | `String`                           | 数据类型的定义                                               |
| 泛型             | `List<E>`                          | E类型的List                                                  |
| 形式类型参数     | `E`                                | 表示具体的类型，用于类，方法进行泛化                         |
| 无限制通配符     | `List<?>`                          | 表示某个未知的类型，创建泛型类型的数组List<String>[](非法) -->  List<?>[](合法) |
| 原生态类型       | `List`                             | 所有List的泛型都是List的子类                                 |
| 有限制类型参数   | `<E extends Number>`               | 形式类型参数<E>的扩展，表示具体的类型，控制参数的粒度，通常用于泛型类和泛型方法 |
| 递归类型限制     | `<T extends Comparable<T>>`        |                                                              |
| 有限制通配符类型 | `List<? extends Number>`           | 无限制通配符？的扩展，表示某个未知的类型，控制参数的粒度，通常用在方法层面 |
| 泛型方法         | `static <E> List<E> aslist(E[] a)` | 独立于泛型类之外                                             |
| 类型令牌         | `String.class`                     |                                                              |



### 泛型的用法（怎么用）

#### 涉及到的泛型

##### 通配符？

1. **？是什么**

   > 表示只能包含某种对象类型，具有唯一性

2. **为什么用**

   > 在某些不确定或者不在乎集合中的元素类型的情况下，如果不使用通配符，使用原生类型（不安全未进行类型检查）或者具体化参数的泛型（不符合业务场景，具有单一性）都存在一定的问题
   >
   > 通配符？，解释为某个类型的集合，在某些不确定集合元素的情况下，能够起到很好的作用（通常用于某个方法的参数传递）

3. **怎么用**

   > List<?> 表示某个类型的List集合，某个方法的参数传递，泛型数组的表示等等

##### 泛型类和泛型接口

##### 泛型数组

1. **泛型数组是什么**

   1. **创建方式(创建泛型数组是非法的)**

   ```java
   // 方式一
   List<String>[] lsa = new List<String>[10] 
   // 编译错误 在java中是不能创建一个确切的泛型类型的数组，不能创建不能具体化的数组（泛型不能具体化）。
   
   // 方式二
   List<?>[] lsa = new List<?>[10]
   // 数组用此种方式，运行时可以显示转换，创建无限制通配符类型的数组是合法的，可具体化
   ```

   2. **数组与泛型**

   > 数组是可协变可以具体化的，泛型是不可变且可以被擦除的

   ```java
   List<String>不是List<Object>的子类，而是List的子类，泛型不可变
   
   String[] 是Object[]的子类，数组可协变（String是Object的子类）
   ```

   > 数组提供了运行时的类型安全，没有编译时的类型安全，泛型反之亦然

   ```java
   Fruit []fruit = new Apple[SIZE];  //ok
   fruit[0] = new Apple();  //ok
   fruit[1] = new Fruit();   //compile ok, run error
   fruit[2] = new Banana();   //comole ok, run error
   
   因为Apple是Fruit的子类，所以上面的写法是完全正确的。但是将Fruit的对象赋值到fruit[1]中，因为fruit本来就是一个Fruit类型所以编译期这是没有任何问题的，但是因为运行中的数组实际类型是Apple所以，后面两种写法在运行时将会报类型错误。
   ```

   > 泛型是不可具体化类型：运行时表示法包含的信息比编译时表示法包含的信息更少的类型， 比如`List<String>,List<E>`由于编译之后**泛型的擦除【元素类型信息】**，即运行时丢弃【元素类型信息】，所以在运行时获取数据进行类型转换时，**不知道是什么类型**，会出现类型转换错误。

   > 具体化类型：唯一可具体化的参数化类型是**List<?>** (无限制的通配符类型),在运行时取出数据需要**显示的类型转换**（指定转换类型）。

   

2. **为什么用（默认创建数组以方式二创建）**

   > 数组和泛型不能很好的混合使用，如果混合使用，用列表代替数组
   >

3. **怎么用**

   1. 使用

   > List<?> 表示某个类型的List集合
   >
   > 使用时可以以 `List<?>[]  lsa = new List<?>[10]`通配符的形式表现

   ```java
   // 编写用数组支持泛型时
   public Class Test {
       private E[] e; // 编译时
       private static final int DEFAULT = 16;
       
       public Test() {
           e = new E[DEFAULT]; // 编译时会出错，运行时会擦除元素类型信息（不知道E是什么）
       }
       
       // 方案 
       // 增加注解，确保未受检的类型转换是安全的
       @SuppressWarning("unchecked")
       public Test() {
           e =(E[]) new Object[DEFAULT]; // 绕过创建数组的禁令,创建objet数组，显示转换成泛型数组类型
       }
       
   }
   ```

   

   

##### 泛型方法与非泛型方法

1. **是什么**

   ```java
   首先在public与返回值之间的<T>必不可少，这表明这是一个泛型方法，并且声明了一个泛型T
   这个T可以出现在这个泛型方法的任意位置.
   泛型的数量也可以为任意多个 
   public <T,K> K showKeyName(Generic<T> container){
   ...
    }
   ```

   

   > 泛型方法使用原则：泛型方法能够独立于类而产生变化，如果泛型方法能够取代整个类泛型化，就使用泛型方法
   >
   > 非泛型方法(带参数化类型的参数时)使用原则：非泛型方法依赖于泛型类参数化类型

2. **为什么用**

   > 静态工具方法尤其适合泛型化,确保方法不用类型转换就能使用，可以应用于多种类型

   

3. **怎么用**

   1. **泛型方法**

      ```java
      public class Test<A> {
          public A a;
          public Test(A a) {
              this.a = a;
          }
          public Test() {
          }
          // 未加通配符 - Test的某个参数化类型
          public static <C> Test<C> push(Test<C> push) {
              return push;
          }
      
          // 有限制的通配符类型-生产者 C的某个子类型的Test
          public static <C> Test<C> pushProduct(Test<? extends C> push,Test<? extends C> push1) {
              return (Test<C>) push;
          }
      
          // 有限制的通配符类型-消费者 C的某种超类的Test
          public static <C> Test<C> pushConsume(Test<? super C> push) {
              return (Test<C>) push;
          }
      
      	public static void main(String[] args) {
              Test<Integer> integerTest = new Test<>();
              Test<Double> doubleTest = new Test<>();
              // 编译报错，泛型是不可变的不能进行类型转换
              Test<Number> numberTest = push(integerTest, doubleTest); 
              // 正常编译，由于Number的子类是Integer和Double
              Test<Number> numberTest1 = pushProduct(integerTest, doubleTest);
              // 正常编译，由于Double是Number的超类
              Test<Double> numberTest1 = pushConsume(numberTest1);
          }
      	// 注意点：泛型方法能够独立于类而产生变化，所以可以加上static关键字，外部调用
      }
      ```

      > 有限制的通配符类型相对于不可变的参数化类型更加灵活，根据需求更改参数的粒度

      

   2. **非泛型方法**（参数是泛型）

      ```java
      public class Test<A> {
          public A a;
          public Test(A a) {
              this.a = a;
          }
          public Test() {
          }
          // 不是泛型方法
          // 有限制的通配符类型 A的某个子类型的Test，A是类的参数化类型
          public Test<A> pushExtend(Test<? extends A> push,Test<? extends A> push1) {
              return push;
          }
          // 未加通配符 - Test的某个参数化类型A, A是类的参数化类型
          public Test<A> pushNoExtend(Test<A> push) {
              return push;
          }
          public static void main(String[] args) {
              Test<Integer> integerTest = new Test<>();
              Test<Double> doubleTest = new Test<>();
              Test<Number> numbertest = new Test<>();
              
              // 正常编译，由于Double是Number的子类 pushExtend的参数化类型和Test的参数化类型息息相关（继承关系）
              numbertest.pushExtend(integerTest, doubleTest);
              // 编译错误，由于参数化类型不可变，pushExtend的参数化类型和Test的参数化类型未定义成任何关系
              numbertest.pushNoExtend(doubleTest);
          }
          
          // 注意点：由于非泛型化方法的参数化类型和类的参数化类型相关联，所以不允许外部调用即不能使用static关键字
      }
      ```

   3. **泛型方法与非泛型方法**

      > 参数类型和通配符之间具有双重性，以上的`pushProduct,pushExtend`方法和`pushNoExtend,push`方法都实现同样的功能
      >
      > 一般使用通配符
      >
      > 如果类型参数只在方法声明中出现一次，可以用通配符取代
      >
      > 如果是无限制的类型参数，就用无限制的通配符取代（Test<E> push  -->  Test<?> push）
      >
      > 如果是有限制的类型参数，就用有限制的通配符取代（Test<E extends A> push  -->  Test<? extends A> push）











