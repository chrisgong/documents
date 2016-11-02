# 依赖倒置原则
> Dependence Inversion Principle(DIP)

## 定义

> High level modules should not depend upon low level modules. Both should depend upon abstractions. Abstractions should not depend upon details. Details should depend upon abstractions.

* 高层模块不应该依赖底层模块， 两者都应该依赖其抽象；
* 抽象不应该依赖细节
* 细节应该依赖抽象

依赖倒置原则在Java中的表现是：

* 模块间的依赖通过抽象发生，实现类之间不发生直接的依赖关系，其依赖关系是通过接口或抽象类产生的；
* 接口或抽象类不依赖于实现类；
* 实现类依赖接口或抽象类。

采用依赖倒置原则可以减少类间的耦合性，提高系统的稳定性，降低并行开发引起的风险，提高代码的可读性和可维护性。

## show the code

![](http://odr63yrh6.bkt.clouddn.com/DIP.jpg)

司机类

```java
public class Driver {
  public void drive(Benz benz) {
    benz.run();
  }
}
```

奔驰车类

```java
public class Benz {
  public void run(){
    System.out.println("奔驰汽车开始运行...");
  }
}
```

场景类

```java
public class Client {
  public static void main(String[] args) {
    Driver zhangSan = new Driver();
    Benz benz = new Benz();
    zhangSan.drive(benz);
  }
}
```

需求变更，要求zhangeSan开宝马车。

宝马车类

```java
public class BWW {
  public void run(){
    System.out.println("宝马汽车开始运行...");
  }
}
```

但是zhangSan没有办法开动宝马车。因为Driver参数是Benz。
司机类和奔驰车类之间是紧耦合的关系，其导致的结果就是系统的可维护性大大降低，可读性降低，两个相似的类需要阅读两个文件。稳定性也下降，增加一个车类就需要修改司机类，被依赖者的变更竟然让依赖者来承担修改的成本。

## 改造

![](http://odr63yrh6.bkt.clouddn.com/DIP2.jpg)

建立两个接口： IDriver和ICar，分别定义了司机汽车的各个职能

司机接口

```java
public interface IDriver {
  public void drive(ICar car);
}
```

司机类的实现

```java
public class Driver implements IDriver {
  public void drive(ICar car) {
    car.run();
  }
}
```

汽车接口及两个实现类

```java
public interface ICar {
  public void run();
}

public class Benz implements ICar {
  public void run() {
    System.out.println("奔驰汽车开始运行...");
  }
}

public class BMW implements ICar {
  public void run() {
    System.out.println("宝马汽车开始运行...");
  }
}
```

场景类

```java
public class Client {
  public static void main(String[] args) {
    IDriver zhangSan = new Driver();
    ICar benz = new Benz();
    zhangSan.drive(benz);
  }
}

public class Client {
  public static void main(String[] args) {
    IDriver zhangSan = new Driver();
    ICar bmw = new BMW();
    zhangSan.drive(bmw);
  }
}
```

业务需求变更时，只修改了业务场景类，也就是高层模块，对其他底层模块如Driver类不需要做任何修改，业务就可以运行，把“变更”引起的风险扩散降低到最小。

## 对并行开发的影响

两个类之间有依赖关系，只要制定出两者之间的接口（或抽象类）就可以独立开发了，而且项目之间的单元测试也可以独立地运行，而TDD开发模式就是依赖倒置原则的最高级应用。

测试类

```java
public class DriverTest extends TestCast {
  Mockery context = new Junit4Mockery();

  @Test
  public void testDriver() {
    final ICar car = context.mock(ICar.class);
    IDriver driver = new Driver();
    context.checking(new Expectations() {
      oneIf(car).run();
    });
    driver.drive(car);
  }
}
```

抽象是对实现的约束，对依赖者而言，也是一种契约，不仅仅约束自己，还同时约束自己与外部的关系，其目的是保证所有细节不脱离契约的范围，确保约束双方按照既定的契约共同发展。

## 依赖的三种写法

* 构造函数传递依赖对象
* Setter方法传递依赖对象
* 接口声明依赖对象

## 最佳实践

* 每个类尽量都有接口或抽象类，或者抽象类和接口两者都具备
* 变量的便面类型尽量是接口或者抽象类
* 任何类都不应该从具体类派生
* 尽量不要覆写基类的方法
* 结合里氏替换原则使用

通俗规则： 接口负责定义public属性和方法，并且声明与其他对象的依赖关系，抽象类负责公共构造部分的实现，实现类准确的实现业务逻辑，同时在适当的时候对父类进行细化。
