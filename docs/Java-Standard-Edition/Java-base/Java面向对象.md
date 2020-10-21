# Java面向对象

  * [一、什么是对象](#一什么是对象)
     * [1.1 什么是程序？](#11-什么是程序)
     * [1.2 程序员眼中的面对对象思想](#12-程序员眼中的面对对象思想)
     * [1.3 对象的创建](#13-对象的创建)
        * [1.3.1 对象创建过程](#131-对象创建过程)
     * [1.4 对象的内存分配](#14-对象的内存分配)
  * [二、什么是类](#二什么是类)
     * [2.1 什么是类？](#21-什么是类)
     * [2.2 类的定义](#22-类的定义)
  * [三、类的组成](#三类的组成)
     * [3.1 类的组成](#31-类的组成)
     * [3.2 类与对象的关系](#32-类与对象的关系)
     * [3.3 什么是实例变量](#33-什么是实例变量)
     * [3.4 实例变量与局部变量的区别](#34-实例变量与局部变量的区别)
     * [3.5 了解实例方法](#35-了解实例方法)
  * [四、方法重载](#四方法重载)
     * [4.1 什么是方法的重载？](#41-什么是方法的重载)
  * [五、构造方法](#五构造方法)
     * [5.1 什么是构造方法呢？](#51-什么是构造方法呢)
     * [5.2 构造方法的重载](#52-构造方法的重载)
     * [5.3 默认构造方法](#53-默认构造方法)
     * [5.4 构造方法为属性赋值](#54-构造方法为属性赋值)
  * [六、this关键字](#六this关键字)
     * [6.1 了解this关键字](#61-了解this关键字)
     * [6.2 this关键字的用法](#62-this关键字的用法)

------

> 一说起对象来，大家肯定第一时间想起了现实中的女朋友，但此对象非彼对象。废话不多说，看下面的对象吧，看你是怎么面向她的！哈哈！

## 一、什么是对象
### 1.1 什么是程序？
程序简单来说就是为了模拟现实世界，解决现实问题而使用计算机语言编写的指令集和



### 1.2 程序员眼中的面对对象思想

 - 一切客观存在的食物都是对象，即：“万物皆对象”
 - 任何对象一定具有自己的特征和行为（即：属性和方法）
 - **特征：** 称为属性，一般为名词，代表对象有什么
 - **行为：** 称为方法，一般为动词，代表对象能做什么
 - 现实中的对象多数来自于“模板”，而程序中的对象也不例外也具有相应的“模板”
![在这里插入图片描述](https://github.com/Yangliang266/programmingKnowledge/blob/master/media/pictures/Java-Standard-Edition/Java面向对象/面向对象思想.png)



### 1.3 对象的创建

![在这里插入图片描述](https://github.com/Yangliang266/programmingKnowledge/blob/master/media/pictures/Java-Standard-Edition/Java面向对象/对象的创建.png)



#### 1.3.1 对象创建过程

![对象创建过程](https://github.com/Yangliang266/programmingKnowledge/blob/master/media/pictures/Java-Standard-Edition/Java面向对象/对象创建过程.png)



### 1.4 对象的内存分配

存储对象的变量中保存对象的地址，通过变量中的地址访问对象的属性和方法
***

## 二、什么是类
### 2.1 什么是类？
类（Class）是面向对象程序设计实现信息封装的基础
![在这里插入图片描述](https://github.com/Yangliang266/programmingKnowledge/blob/master/media/pictures/Java-Standard-Edition/Java面向对象/什么是类.png)



### 2.2 类的定义

![在这里插入图片描述](https://github.com/Yangliang266/programmingKnowledge/blob/master/media/pictures/Java-Standard-Edition/Java面向对象/类的定义.png)
***
<a id="3"> </a>
## 三、类的组成

### 3.1 类的组成

 - **属性：** 比如：学生的属性有姓名、性别、出生年月、家庭住址、电话等
 - **比如：** 学生的行为（也就是方法）有学习、打游戏、看电影等



### 3.2 类与对象的关系

 - **类：** 定义了对象具有的特征和行为，类是对象的模板
 - **对象：** 拥有多个特征和行为的实体，对象是类的实例

![在这里插入图片描述](https://gitee.com/Ziphtracks/Figurebed/raw/master/img/20200503194601.png)
### 3.3 什么是实例变量
![在这里插入图片描述](https://gitee.com/Ziphtracks/Figurebed/raw/master/img/20200503194635.png)
### 3.4 实例变量与局部变量的区别

| 种类     | 局部变量                 | 成员变量                       | 种类     |
| -------- | ------------------------ | ------------------------------ | -------- |
| 定义位置 | 方法或方法内的构造当中   | 类的内部，方法的外部           | 定义位置 |
| 默认值   | 无默认值                 | 字面值与数组相同               | 默认值   |
| 适用范围 | 从定义行到包含其构造结束 | 本类有效                       | 适用范围 |
| 命名冲突 | 不允许重名               | 可与局部变量重名，局部变量优先 | 命名冲突 |



### 3.5 了解实例方法

**实例方法包含两部分：** 方法的声明和方法的实现

 - **方法的声明：**
	* 代表对象能做什么
	* **组成：** 修饰符、返回值类型和方法名（形参列表）
 - 方法的实现：
	* **代表对象怎么做：** 即如何实现对应的功能
	* **组成：**{}
***
<a id="4"> </a>
## 四、方法重载
### 4.1 什么是方法的重载？


 - 一个类中定义多个相同名称的方法
 - **要求：** 
	* 方法名称相同 
	* 参数列表不同（类型、个数、顺序）
	* 与访问修饰符、返回值无关
 - **好处：** 屏蔽使用差异、灵活、方便

<font color="red">**注意：**</font>只是参数不同并不能构成方法的重载
![在这里插入图片描述](https://github.com/Yangliang266/programmingKnowledge/blob/master/media/pictures/Java-Standard-Edition/Java面向对象/方法重载.png)

***
<a id="5"> </a>
## 五、构造方法
### 5.1 什么是构造方法呢？

 - 类中的特殊方法，主要用于创建对象
 - **特点：**
	* 名称与类名完全相同 
	* 没有返回值类型 
	* 创建对象时，触发构造方法的调用，不可通过句点手动调用

<font color="red">**注意：**</font>如果没有在类中显示定义构造方法，则编译器默认提供无参构造方法



### 5.2 构造方法的重载

构造方法也可以重载，遵循重载规则
![在这里插入图片描述](https://github.com/Yangliang266/programmingKnowledge/blob/master/media/pictures/Java-Standard-Edition/Java面向对象/构造方法的重载.png)



### 5.3 默认构造方法

![在这里插入图片描述](https://github.com/Yangliang266/programmingKnowledge/blob/master/media/pictures/Java-Standard-Edition/Java面向对象/默认的构造方法.png)
### 5.4 构造方法为属性赋值
![在这里插入图片描述](https://github.com/Yangliang266/programmingKnowledge/blob/master/media/pictures/Java-Standard-Edition/Java面向对象/构造方法为属性赋值.png)
***
<a id="6"> </a>
## 六、this关键字

### 6.1 了解this关键字

![在这里插入图片描述](https://github.com/Yangliang266/programmingKnowledge/blob/master/media/pictures/Java-Standard-Edition/Java面向对象/了解this关键字.png)



### 6.2 this关键字的用法

 1. 调用实例属性、实例方法，如：this.name、this.sayHi();
![在这里插入图片描述](https://github.com/Yangliang266/programmingKnowledge/blob/master/media/pictures/Java-Standard-Edition/Java面向对象/this关键字的用法1.png)
 2. 调用本类中其他构造方法，如：this(); this(实参);
![在这里插入图片描述](https://github.com/Yangliang266/programmingKnowledge/blob/master/media/pictures/Java-Standard-Edition/Java面向对象/this关键字的用法2.png)
***



> 上一章[【Java数组】](https://github.com/Yangliang266/programmingKnowledge/blob/master/docs/Java-Standard-Edition/Java-base/Java数组.md)

> 下一章[【Java面向对象三大特性】](https://github.com/Yangliang266/programmingKnowledge/blob/master/docs/Java-Standard-Edition/Java-base/Java面向对象三大特性.md)

