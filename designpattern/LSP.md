# 里氏替换原则
> Liskov Substitution Principle(LSP)

采用里氏替换原则的目的是增强程序的健壮性，版本升级时也可以保持非常好的兼容性。即使增加子类，原有的子类还可以继续运行。在实际项目中，每个子类对应不同的业务含义，使用父类作为参数，传递不同的子类完成不同的业务逻辑。

## 定义

所有引用基类的地方必须能透明地使用其子类的对象。
> Functions that use pointers or references to base classes must be able to use objects of derived classes without knowing it.

通俗的讲， 只要父类能出现的地方子类就可以出现， 而且替换成子类也不会产生任何错误和异常， 使用者可能根本就不需要知道是父类还是子类。 但是， 反过来就不行了， 有子类出现的地方， 父类未必就能适应。

## 规则

1. 子类必须完全实现父类的方法

  CS例子:

  ![](http://odr63yrh6.bkt.clouddn.com/lsp.png)

  枪支的抽象类：

  ```java
  public abstract class AbstractGun {
    public abstract void shoot();
  }
  ```

  手枪、步枪、机枪的实现类：

  ```java
  public class Handgun extends AbstractGun {
    @Override
    public void shoot() {
      System.out.print("手枪射击...");
    }
  }

  public class Rifle extends AbstractGun {
    @Override
    public void shoot() {
      System.out.print("步枪射击...");
    }
  }

  public class MachineGun extends AbstractGun {
    @Override
    public void shoot() {
      System.out.print("机枪扫射...");
    }
  }
  ```

  士兵的实现类：

  ```java
  public class Soldier {
    private AbstractGun gun;
    public void setGun(AbstractGun _gun) {
      this.gun = _gun;
    }

    public void killEnemy() {
      System.out.println("士兵开始杀敌人...");
      gun.shoot();
    }
  }
  ```

  场景类：

  ```java
  public class Client {
    public static void main(String[] args) {
      Soldier sanMao = new Soldier();
      sanMao.setGun(new Rifle());
      sanMao.killEnemy();
    }
  }

  //士兵开始杀敌人...
  //步枪射击...
  ```
  > ***TIP***
  > * 在类中调用其他类时务必要使用父类或接口， 如果不能使用父类或接口， 则说明类的设计已经违背LSP原则。
  > * 如果子类不能完整地实现父类的方法， 或者父类的某些方法在子类中已经发生“畸变”， 则建议断开父子继承关系， 采用依赖、聚集、组合等关系代替继承。

2. 子类可以有自己的个性

  子类可以有自己的方法和属性。 子类出现的地方， 父类未必可以胜任。

  CS例子， 引入步枪具体型号： AK47、AUG， 狙击手（Snipper）直接使用AUG狙击步枪。

  ![](http://odr63yrh6.bkt.clouddn.com/lsp2.jpg)

  AUG狙击步枪类

  ```java
  public class AUG extends Rifle {
    public void zoomOut() {
      System.out.println("通过望远镜察看敌人...");
    }

    public void shoot() {
      System.out.println("AUG射击...");
    }
  }
  ```

  狙击手类

  ```java
  public class Snipper {
    public void killEnemy(AUG aug) {
      aug.zoomOut();
      aug.shoot();
    }
  }
  ```

  场景类

  ```java
  public class Client {
    public static void main(String[] args) {
      Snipper sanMao = new Snipper();
       //sanMao.setRifle((AUG)new Rifle()); ClassCastException异常， 向下转型是不安全的， 从里氏替换原则来看，有子类出现的地方父类未必胜任。
      sanMao.setRifle(new AUG());
      sanMao.killEnemy();
    }
  }

  //通过望远镜察看敌人...
  //AUG设计...
  ```

3. 覆盖或实现父类的方法时输入参数可以被放大

Father类

```java
public class Father {
  public Collection dosomething(HashMap map) {
    System.out.println("父类被执行...");
    return map.values();
  }
}
```

子类

```java
public class Son extends Father {
  public Collection dosomething(Map map) {
    System.out.println("子类被执行...");
    return map.values();
  }
}
```

场景类

```java
public class Client {
  public static void invoker() {
    Father f = new Father();
    HashMap map = new HashMap();
    f.dosomething();
  }

  public static void main(String[] args) {
    invoker();
  }
}

//父类被执行
```

子类替换父类后的场景类

```java
public class Client {
  public static void invoker() {
    Son f = new Son();
    HashMap map = new HashMap();
    f.dosomething();
  }

  public static void main(String[] args) {
    invoker();
  }
}

//父类被执行
```

如果父类和子类的参数范围反过来， 则子类被执行。

> 子类中的方法的前置条件必须与超类中被覆写的方法的前置条件相同或者更宽松。

4. 覆写或实现父类的方法时输出结果可以被缩小

父类的一个方法的返回值是一个类型T，子类的相同方法（重载或覆写）的返回值是S，那么里氏替换原则就要求S必须小于等于T，也就是说，要么S和T是同一个类型，要么S是T的子类。

* 如果是覆写，父类和子类的同名方法的输入参数是相同的，两个方法的范围值S小于等于T。
* 如果是重载，要求方法的输入参数类型或数量不相同，在里氏替换原则要求下，就是子类的输入参数宽于或等于父类的输入参数，也就是说你写的这个方法是不会被调用的。

## 最佳实践

在项目中，采用里氏替换原则时，尽量避免子类的“个性”，一旦子类有“个性”，这个子类和父类之间的关系就很难调和了，把子类当做父类用，子类的“个性”被抹杀；把子类单独作为一个业务来使用，则会让代码间的耦合关系变得扑朔迷离--缺乏类替换的标准。
