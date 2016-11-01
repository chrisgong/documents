# 里氏替换原则
> Liskov Substitution Principle(LSP)

## 定义

所有引用基类的地方必须能透明地使用其子类的对象。
> Functions that use pointers or references to base classes must be able to use objects of derived classes without knowing it.

通俗的讲， 只要父类能出现的地方子类就可以出现， 而且替换成子类也不会产生任何错误和异常， 使用者可能根本就不需要知道是父类还是子类。 但是， 反过来就不行了， 有子类出现的地方， 父类未必就能适应。

## 4层含义

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
4. 覆写或实现父类的方法时输出结果可以被缩小
