# 类与类之间的关系图(Class Diagram,UML图)

## 简介

类是对象的集合，展示了对象的结构以及与系统的交互行为。类主要有属性（Attribute）和方法（Method）构成,属性代表对象的状态，如果属性被保存到数据库，此称之为“持久化”；方法代表对象的操作行为，类具有继承关系，可以继承于父类，也可以与其他的Class进行交互。

类图展示了系统的逻辑结构，类和接口的关系。

## 类的构成

类主要有属性和方法构成。比如商品属性有：名称、价格、高度、宽度等；商品的方法有：计算税率，获得商品的评价等等。如下图

![](http://ww2.sinaimg.cn/large/006tNc79gw1f9knk2t16xj306g04m3yk.jpg)

## 类之间的关系（Relationship）

|关系类型|箭头指向|
|---|---|
|泛化|带三角箭头的实线，箭头指向父类|

1. 泛化(Generalization)

  是一种继承关系，表示一般与特殊的关系，它指定了子类如何特化父类的所有特征和行为。

  箭头指向：带三角箭头的实线，箭头指向父类

  ![](http://ww4.sinaimg.cn/large/006tNc79gw1f9knzodfisj309i05o3yk.jpg)

  系统的使用者包括：B2C会员、B2B会员和B2E会员。

2. 实现(Realization)

  是一种类与接口的关系，表示类是接口所有特征和行为的实现.

  箭头指向：带三角箭头的虚线，箭头指向接口

  ![](http://ww1.sinaimg.cn/large/006tNc79gw1f9ko2kiqj9j304a05cdfr.jpg)

3. 关联（Association)

  是一种拥有的关系，它使一个类知道另一个类的属性和方法；如：老师与学生，丈夫与妻子关联可以是双向的，也可以是单向的。双向的关联可以有两个箭头或者没有箭头，单向的关联有一个箭头。

  代码体现：成员变量

  箭头指向：带普通箭头的实心线，指向被拥有者

  * 单向关联

    A1->A2: 表示A1认识A2，A1知道A2的存在，A1可以调用A2中的方法和属性

    场景：订单和商品，订单中包括商品，但是商品并不了解订单的存在。

    类与类之间的单向关联图:

    ![](http://ww2.sinaimg.cn/large/006tNc79gw1f9knlqg8ejj30c004m74f.jpg)

  * 双向关联

    B1-B2: 表示B1认识B2，B1知道B2的存在，B1可以调用B2中的方法和属性；同样B2也知道B1的存在，B2也可以调用B1的方法和属性。

    场景：订单和客户，订单属于客户，客户拥有一些特定的订单

    类与类之间的双向关联图

    ![](http://ww1.sinaimg.cn/large/006tNc79gw1f9knq46q1jj30d602k0sr.jpg)

  * 自身关联

    同一个类对象之间的关联

    类与类之间自身关联图

    ![](http://ww3.sinaimg.cn/large/006tNc79gw1f9knrffe3xj306s04p0sl.jpg)

  * 多维关联(N-ary Association)

    多个对象之间存在关联

    场景：公司雇用员工，同时公司需要支付工资给员工

    类与类之间的多维关联图

    ![](http://ww2.sinaimg.cn/large/006tNc79gw1f9knvm6o73j30am06y3yj.jpg)

4. 聚合(Aggregation)

  是整体与部分的关系，且部分可以离开整体而单独存在。如车和轮胎是整体和部分的关系，轮胎离开车仍然可以存在。

  聚合关系是关联关系的一种，是强的关联关系；关联和聚合在语法上无法区分，必须考察具体的逻辑关系。

  代码体现：成员变量

  箭头及指向：带空心菱形的实心线，菱形指向整体

  场景：商品和他的规格、样式就是聚合关系。

  ![](http://ww2.sinaimg.cn/large/006tNc79gw1f9ko920cg0j308y050dft.jpg)

  ```java
  class Driver {  
    //使用成员变量形式实现聚合关系  
    Car mycar;  
    public void drive(){  
        mycar.run();  
    }  
  }
  ```

5. 组合(Composite)

  是整体与部分的关系，但部分不能离开整体而单独存在。如公司和部门是整体和部分的关系，没有公司就不存在部门。

  组合关系是关联关系的一种，是比聚合关系还要强的关系，它要求普通的聚合关系中代表整体的对象负责代表部分的对象的生命周期。

  代码体现：成员变量

  箭头及指向：带实心菱形的实线，菱形指向整体

  场景： Window窗体由滑动条slider、头部Header 和工作区Panel组合而成。

  类与类的组合关系图

  ![](http://ww1.sinaimg.cn/large/006tNc79gw1f9kobcqeiij30bq050glo.jpg)

  ```java
  class Driver {  
    //使用成员变量形式实现关联  
    Car mycar;  
    public void drive(){  
        mycar.run();  
    }  
    ...  
    //使用方法参数形式实现关联  
    public void drive(Car car){  
        car.run();  
    }  
  }
  ```

6. 依赖(Dependency)

  是一种使用的关系，即一个类的实现需要另一个类的协助，所以要尽量不使用双向的互相依赖.

  代码表现：局部变量、方法的参数或者对静态方法的调用

  箭头及指向：带箭头的虚线，指向被使用者

  场景：本来人与电脑没有关系的，但由于偶然的机会，人需要用电脑写程序，这时候人就依赖于电脑。

  类与类的依赖关系图

  ![](http://ww4.sinaimg.cn/large/006tNc79gw1f9ko6qu9kaj309f04qaa2.jpg)

  ```java
  class Car {  
    public static void run(){  
        System.out.println("汽车在奔跑");  
    }  
  }  

  class Driver {  
    //使用形参方式发生依赖关系  
    public void drive1(Car car){  
        car.run();  
    }  
    //使用局部变量发生依赖关系  
    public void drive2(){  
        Car car = new Car();  
        car.run();  
    }  
    //使用静态变量发生依赖关系  
    public void drive3(){  
        Car.run();  
    }  
  }
  ```

> 依赖关系实际上是一种比较弱的关联，聚合是一种比较强的关联，而组合则是一种更强的关联，所以笼统的来区分的话，实际上这四种关系、都是关联关系
