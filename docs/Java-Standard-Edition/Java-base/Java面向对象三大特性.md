# Java面向对象三大特性

  * [一、封装](#一封装)
     * [1.1 封装的必要性](#11-封装的必要性)
     * [1.2 什么是封装？](#12-什么是封装)
     * [1.3 公共访问方法](#13-公共访问方法)
     * [1.4 过滤有效数据](#14-过滤有效数据)
  * [二、继承](#二继承)
     * [2.1 认识继承](#21-认识继承)
     * [2.2 继承的规律](#22-继承的规律)
     * [2.3 父类的抽象](#23-父类的抽象)
     * [2.4 继承的用法和好处](#24-继承的用法和好处)
     * [2.5 继承的特点](#25-继承的特点)
     * [2.6 不可继承](#26-不可继承)
  * [三、访问修饰符](#三访问修饰符)
  * [四、方法重写](#四方法重写)
     * [4.1 方法重写的概念](#41-方法重写的概念)
     * [4.2 方法的覆盖](#42-方法的覆盖)
     * [4.3 super关键字](#43-super关键字)
        * [4.3.1 super的访问方法](#431-super的访问方法)
        * [4.3.2 super的访问属性](#432-super的访问属性)
     * [4.4 继承中的对象创建](#44-继承中的对象创建)
        * [4.4.1 继承后的对象创建过程](#441-继承后的对象创建过程)
     * [4.5 super调用父类无参构造](#45-super调用父类无参构造)
     * [4.6 super调用父类的有参构造](#46-super调用父类的有参构造)
     * [4.7 this与super](#47-this与super)
  * [五、多态](#五多态)
     * [5.1 认识多态](#51-认识多态)
     * [5.2 多态的作用](#52-多态的作用)
     * [5.3 多态中的方法覆盖](#53-多态中的方法覆盖)
     * [5.4 多态的应用](#54-多态的应用)
  * [六、拆箱装箱](#六拆箱装箱)
     * [6.1 向上转型（装箱）](#61-向上转型装箱)
     * [6.2 向下转型（拆箱）](#62-向下转型拆箱)
     * [6.3 类型转换异常](#63-类型转换异常)
     * [6.4 instanceof关键字](#64-instanceof关键字)

------

## 一、封装
### 1.1 封装的必要性

在对象的外部为对象的属性赋值，可能存在非法数据的录入，存在不安全隐患。就目前的技术而言，并没有办法对属性的赋值加以控制。所以要实现属性的封装非常重要！



### 1.2 什么是封装？

 - **概念：** 尽可能隐藏对象的内部实现细节，控制对象的修改及访问权限
 - **访问修饰符：** private（可将属性修饰为私有，仅限本类可见）
![在这里插入图片描述](https://github.com/Yangliang266/programmingKnowledge/blob/master/media/pictures/Java-Standard-Edition/Java面向对象三大特性/封装.png)



### 1.3 公共访问方法

![在这里插入图片描述](https://github.com/Yangliang266/programmingKnowledge/blob/master/media/pictures/Java-Standard-Edition/Java面向对象三大特性/公共访问方法.png)



### 1.4 过滤有效数据

![在这里插入图片描述](https://github.com/Yangliang266/programmingKnowledge/blob/master/media/pictures/Java-Standard-Edition/Java面向对象三大特性/过滤有效数据.png)
![在这里插入图片描述](https://github.com/Yangliang266/programmingKnowledge/blob/master/media/pictures/Java-Standard-Edition/Java面向对象三大特性/get和set方法.png)

<font color="red">**注意：**</font>外部访问只可以访问公共空间，不可以直接访问属性设置为private的私有空间，而get、set方法是外界访问对象私有属性的唯一通道，方法内部可对数据进行检测和过滤
<a id="2"> </a>

## 二、继承

### 2.1 认识继承

>**所谓继承，生活中的继承有儿子继承父亲的资产，而程序的继承，是类与类之间特征和行为的一中赠与或获得，好比，所有生物都会呼吸，而动物类也都会呼吸，动物类就可以继承生物类的呼吸特征；人类会走路，而老年人类就不会走路了吗？答案是不对的，当然老年人也是会走路的哈，所以定义的老年人类就可以继承人类的走路行为。**



### 2.2 继承的规律

 - 功能越精细，重合点越多，越接近直接父类
 - 功能越粗略，重合点越少，越接近Object类（所谓：万物皆对象）



### 2.3 父类的抽象

![在这里插入图片描述](https://github.com/Yangliang266/programmingKnowledge/blob/master/media/pictures/Java-Standard-Edition/Java面向对象三大特性/父类的抽象.png)



### 2.4 继承的用法和好处

 - 语法：class子类extends父类{} //定义子类时，显示继承父类
 - 应用：产生继承关系之后，子类可以使用父类中的属性和方法，也可以定义子类独有的属性和方法
 - 好处：既提高了代码的复用性，又提高了代码的可扩展性



### 2.5 继承的特点

![在这里插入图片描述](https://github.com/Yangliang266/programmingKnowledge/blob/master/media/pictures/Java-Standard-Edition/Java面向对象三大特性/继承的特点.png)



### 2.6 不可继承

 - 构造方法：类中的构造方法，只负责创建本类对象，不可继承
 - private修饰的属性和方法：访问修饰符的一种，仅本类可见（详见访问修饰符）
 - 父子类不在同一个package中时，default修饰的属性和方法：访问修饰符的一种，仅同包（package）可见（详见访问修饰符）
***
<a id="3"> </a>
## 三、访问修饰符
![访问修饰符](https://github.com/Yangliang266/programmingKnowledge/blob/master/media/pictures/Java-Standard-Edition/Java面向对象三大特性/访问修饰符.png)
***
<a id="4"> </a>
## 四、方法重写

### 4.1 方法重写的概念

> 当父类提供的方法无法满足子类需求时，可在子类中定义和父类相同的方法进行覆盖（Override）



### 4.2 方法的覆盖

 - **方法的覆盖原则：**
 - 方法名称、参数列表、返回值类型必须与父类相同
 - 访问修饰符可与父类相同或是比父类更宽泛
 - **方法覆盖的执行：**
 - 子类覆盖父类方法后，调用时优先执行子类覆盖后的方法



### 4.3 super关键字

 - **第一种用法：**
	* 在子类的方法中使用“super.”的形式访问父类的属性和方法
	* 例如：super.父类属性、super.父类方法();
 - **第二种用法**
	* 在子类的构造方法的首行，使用“super()”或“super(参数)”，调用父类构造方法
 - <font color="red">**注意：**</font>
	* 如果子类构造方法中，没有显示定义super()或super(实参)，则默认提供super()
	* 同一个子类构造方法中，super()、this()不可同时存在（因为都是要声明在首行）

&nbsp;

 - 在子类中，可直接访问从父类继承到的属性和方法，但如果父子类的属性或方法存在重名（属性遮蔽、方法覆盖）时，需要加以区分，才可专项访问
![在这里插入图片描述](https://github.com/Yangliang266/programmingKnowledge/blob/master/media/pictures/Java-Standard-Edition/Java面向对象三大特性/super关键字.png)



#### 4.3.1 super的访问方法

![在这里插入图片描述](https://github.com/Yangliang266/programmingKnowledge/blob/master/media/pictures/Java-Standard-Edition/Java面向对象三大特性/super的访问方法.png)



#### 4.3.2 super的访问属性

![在这里插入图片描述](https://github.com/Yangliang266/programmingKnowledge/blob/master/media/pictures/Java-Standard-Edition/Java面向对象三大特性/super的访问属性.png)



### 4.4 继承中的对象创建

 - 在具有继承关系的对象创建中，构建子类对象会先构建父类对象
 - 由父类的共性内容，叠加子类的独有内容，组合成完整的子类对象
![在这里插入图片描述](https://github.com/Yangliang266/programmingKnowledge/blob/master/media/pictures/Java-Standard-Edition/Java面向对象三大特性/继承中的对象创建.png)



#### 4.4.1 继承后的对象创建过程

![在这里插入图片描述](https://github.com/Yangliang266/programmingKnowledge/blob/master/media/pictures/Java-Standard-Edition/Java面向对象三大特性/继承后的对象创建过程.png)



### 4.5 super调用父类无参构造

![在这里插入图片描述](https://github.com/Yangliang266/programmingKnowledge/blob/master/media/pictures/Java-Standard-Edition/Java面向对象三大特性/super调用父类无参构造.png)



### 4.6 super调用父类的有参构造

![super调用父类的有参构造](https://github.com/Yangliang266/programmingKnowledge/blob/master/media/pictures/Java-Standard-Edition/Java面向对象三大特性/super调用父类的有参构造.png)



### 4.7 this与super

![在这里插入图片描述](https://github.com/Yangliang266/programmingKnowledge/blob/master/media/pictures/Java-Standard-Edition/Java面向对象三大特性/this与super.png)
***
<a id="5"> </a>
## 五、多态

### 5.1 认识多态

 - **概念：** 父类引用指向子类对象，从而产生多种形态
 - 二者具有直接或间接的继承关系时，父类引用可指向子类对象，即形成多态
 - 父类引用仅可调用父类所声明的属性和方法，不可调用子类独有的属性和方法
![在这里插入图片描述](https://github.com/Yangliang266/programmingKnowledge/blob/master/media/pictures/Java-Standard-Edition/Java面向对象三大特性/什么是多态.png)



### 5.2 多态的作用

 - 屏蔽子类间的差距
 - 灵活、耦合度低



### 5.3 多态中的方法覆盖

实际运行过程中，依旧遵循覆盖原则，如果子类覆盖了父类中的方法，则执行子类中覆盖后的方法，否则执行父类中的方法



### 5.4 多态的应用

 - **场景一：**
 - 使用父类作为方法形参实现多态，使方法参数的类型更为宽泛
 - 调用方法时，可传递的实参类型包括：本类型对象+其所有的子类对象
 - **场景二：**
 - 使用父类作为方法返回值实现多态，使方法可以返回子类对象
 - 调用方法后，可得到的结果类型包括：本类型对象+其所有的子类对象

![在这里插入图片描述](https://github.com/Yangliang266/programmingKnowledge/blob/master/media/pictures/Java-Standard-Edition/Java面向对象三大特性/多态的应用.png)
***
<a id="6"> </a>
## 六、拆箱装箱
### 6.1 向上转型（装箱）
![在这里插入图片描述](https://github.com/Yangliang266/programmingKnowledge/blob/master/media/pictures/Java-Standard-Edition/Java%E9%9D%A2%E5%90%91%E5%AF%B9%E8%B1%A1%E4%B8%89%E5%A4%A7%E7%89%B9%E6%80%A7/%E5%90%91%E4%B8%8A%E8%BD%AC%E5%9E%8B%EF%BC%88%E8%A3%85%E7%AE%B1%EF%BC%89.png)



### 6.2 向下转型（拆箱）

![在这里插入图片描述](https://github.com/Yangliang266/programmingKnowledge/blob/master/media/pictures/Java-Standard-Edition/Java%E9%9D%A2%E5%90%91%E5%AF%B9%E8%B1%A1%E4%B8%89%E5%A4%A7%E7%89%B9%E6%80%A7/%E5%90%91%E4%B8%8B%E8%BD%AC%E5%9E%8B%EF%BC%88%E6%8B%86%E7%AE%B1%EF%BC%89.png)



### 6.3 类型转换异常

![在这里插入图片描述](https://github.com/Yangliang266/programmingKnowledge/blob/master/media/pictures/Java-Standard-Edition/Java%E9%9D%A2%E5%90%91%E5%AF%B9%E8%B1%A1%E4%B8%89%E5%A4%A7%E7%89%B9%E6%80%A7/%E7%B1%BB%E5%9E%8B%E8%BD%AC%E6%8D%A2%E5%BC%82%E5%B8%B8.png)



### 6.4 instanceof关键字

 - 向下转型前，应判断引用中的对象真实类型，保证类型转换的正确性
 - **语法：** 引用instanceof类型	//返回boolean类型结果
![在这里插入图片描述](https://github.com/Yangliang266/programmingKnowledge/blob/master/media/pictures/Java-Standard-Edition/Java%E9%9D%A2%E5%90%91%E5%AF%B9%E8%B1%A1%E4%B8%89%E5%A4%A7%E7%89%B9%E6%80%A7/instanceof%E5%85%B3%E9%94%AE%E5%AD%97.png)

***



> 上一章[【Java面向对象】](https://github.com/Yangliang266/programmingKnowledge/blob/master/docs/Java-Standard-Edition/Java-base/Java面向对象.md)

> 下一章[【Java三个修饰符】](https://github.com/Yangliang266/programmingKnowledge/blob/master/docs/Java-Standard-Edition/Java-base/Java三个修饰符.md)
