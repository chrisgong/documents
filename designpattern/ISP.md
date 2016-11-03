# 接口隔离原则
> Interface Segregation Principle(ISP)

## 定义

在讲接口隔离原则之前，先明确接口的两种：

* 实例接口（Object Interface），在Java中声明一个类，然后用new关键字产生一个实例，它是对一个类型的事物的描述，这是一种接口。
* 类接口（Class Interface），Java中经常使用的interface关键字定义的接口。

隔离定义：

* 客户端不应该依赖它不需要的接口。
* 类间的依赖关系应该建立在最小的接口上。

> 接口尽量细化，同时接口中的方法尽量少。
> 单一职责要求的是类和接口职责单一，注重的是职责，这是业务逻辑上的划分，而接口隔离原则要求接口的方法尽量少。

![](http://odr63yrh6.bkt.clouddn.com/ISP.jpg)

美女类

```java
public interface IPettyGirl {
  public void goodLooking();
  public void niceFigure();
  public void greatTemperament();
}
```

美女实现类

```java
public vlass PettyGirl implements IPettyGirl {
  private String name;

  public PettyGirl(String _name) {
    this.name = name;
  }

  public void goodLooking() {
    System.out.println(this.name + "---脸蛋很漂亮!");
  }

  public void greatTemperament() {
    System.out.println(this.name + "---气质非常好!");
  }

  public void niceFigure() {
    System.out.println(this.name + "---身材非常棒!");
  }
}
```

星探抽象类

```java
public abstract class AsbtractSearcher {
  protected IPettyGirl pettyGirl;
  public AsbtractSearcher(IPettyGirl _pettyGirl) {
    this.pettyGirl = _pettyGirl;
  }
  public abstract void show();
}
```

星探类

```java
public class Searcher extends AbstractSearcher {
  public Searcher(IPettyGirl _pettyGirl) {
    super(_pettyGirl);
  }

  public void show() {
    System.out.println("--------美女的信息如下：--------");
    super.pettyGirl.goodLooking();
    super.pettyGirl.niceFigure();
    super.pettyGirl。greatTemperament();
  }
}
```

场景类

```java
public class Client {
  public static void main(String[] args) {
    IPettyGirl yanYan = new PettyGirl("嫣嫣");
    AbstractSearcher searcher = new Searcher(yanyan);
    searcher.show();
  }
}
```

---

IPettyGirl的设计是有缺陷的，过于庞大了，容纳了一些可变的因素，根据接口隔离原则，星探AbstractSearcher应该以来于具有部分特质的女孩子，而我们却把这些特质都封装了起来，放到一个接口中，封装过度了。

![](http://odr63yrh6.bkt.clouddn.com/ISP2.jpg)

两种类型的美女定义

```java
public interface IGoodBodyGirl {
  public void goodLooking();
  public void niceFigure();
}

public interface IGreatTemperamentGirl {
  public void greatTemperament();
}
```

最标准的美女

```java
public class PettyGirl implements IGoodBodyGirl, IGreatTemperamentGirl {
  private String name;
  public PettyGirl(String _name) {
    this.name = _name;
  }

  public void goodLooking() {
    System.out.println(this.name + "---脸蛋很漂亮!");
  }

  public void greatTemperament() {
    System.out.println(this.name + "---气质非常好!");
  }

  public void niceFigure() {
    System.out.println(this.name + "---身材非常棒!");
  }
}
```

以上把一个臃肿的接口变更为两个独立的接口所以来的原则就是接口隔离原则，让星探AbstractSearcher依赖两个专用的接口比依赖一个综合的接口要灵活。接口是我们设计时对外提供的契约，通过分散定义多个接口，可以预防未来变更的扩散，提高系统的灵活性和可维护性。

## 保证接口的纯洁性

* 接口要尽量小： 小的限度是不能违反单一职责原则。
* 接口要高内聚： 高内聚就是提高接口、类、模块的处理能力，减少对外的交互。要求在接口中尽量少公布public方法，接口是对外的承诺，承诺越少对系统的开发越有利，变更的风险也就越少，同时也有利于降低成本。
* 定制服务： 单独为一个个体提供优良的服务。要求只提供访问者需要的方法。
* 接口设计是有限度的： 接口设计粒度越小，系统越灵活。

## 最佳实践

* 一个接口只服务于一个子模块或业务逻辑。
* 通过业务逻辑压缩接口中的public方法。
* 已经被污染了的接口，尽量去修改，若变更的风险较大，则采用适配器模式进行转化处理。
* 了解环境，拒绝盲从。
