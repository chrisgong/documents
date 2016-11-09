# 开闭原则
> Open Closed Principle（OCP）

## 定义

一个软件实体如类、模块和函数应该对扩展开放，对修改关闭。

软件实体包括以下几个部分：

* 项目或软件产品中按照一定的逻辑规则划分的模块。
* 抽象和类。
* 方法

## show me the code

![](http://odr63yrh6.bkt.clouddn.com/OCP.jpg)

书籍接口

```java
public interface IBook {
  public String getName();
  public int getPrice();
  public String getAuthor();
}
```

小说类

```java
public class NovelBook implements IBook {
  private String name;
  private int price;
  private String author;
  public NovelBook(String _name, int _price, String _author) {
    this.name = _name;
    this.price = _price;
    this.author = _author;
  }

  public String getAuthor() {
    return this.author;
  }

  public String getName() {
    return this.name;
  }

  public int getPrice() {
    return this.price;
  }
}
```

书店售书类

```java
public class BookStore {
  private final static ArrayList<IBook> bookList = new ArrayList<IBook>();

  static {
    bookList.add(new NovelBook("天龙八部", 3200, "金庸"));
    bookList.add(new NovelBook("巴黎圣母院", 5600, "雨果"));
    bookList.add(new NovelBook("悲惨世界", 3500, "雨果"));
    bookList.add(new NovelBook("金瓶梅", 4300, "兰陵笑笑生"));
  }

  public static void main(String[] args) {
    NumberFormat formatter = NumberFormat.getCurrentcyInstance();
    formatter.setMaximumFractionDigits(2);
    System.out.println("------------书店卖出去的书籍记录如下：---------------");
    for(IBook book : bookList) {
      System.out.println("书籍名称：" + book.getName() + "\t书籍作者：" + book.getAuthor() + "\t书籍价格：" + formatter.format(book.getPrice() / 100.0) + "元");
    }
  }
}
```
---
添加打折逻辑

![](http://odr63yrh6.bkt.clouddn.com/OCP2.jpg)

打折销售的小说类

```java
public class OffNovelBook extends NovelBook {
  public OffNovelBook(String _name, int _price, String _author) {
    super(_name, _price, _author);
  }

  @Override
  public int getPrice() {
    int selfPrice = super.getPrice();
    int offPrice = 0;
    if(selfPrice > 4000) {
      offPrice = selfPrice * 90 / 100;
    } else {
      offPrice = selfPrice * 80 / 100;
    }
    return offPrice;
  }
}
```

书店打折销售类

```java
public class BookStore {
  private final static ArrayList<IBook> bookList = new ArrayList<IBook>();
  static {
    bookList.add(new OffNovelBook("天龙八部", 3200, "金庸"));
    bookList.add(new OffNovelBook("巴黎圣母院", 5600, "雨果"));
    bookList.add(new OffNovelBook("悲惨世界", 3500, "雨果"));
    bookList.add(new OffNovelBook("金瓶梅", 4300, "兰陵笑笑生"));
  }

  public static void main(String[] args) {
    NumberFormat formatter = NumberFormat.getCurrentcyInstance();
    formatter.setMaximumFractionDigits(2);
    System.out.println("------------书店卖出去的书籍记录如下：---------------");
    for(IBook book : bookList) {
      System.out.println("书籍名称：" + book.getName() + "\t书籍作者：" + book.getAuthor() + "\t书籍价格：" + formatter.format(book.getPrice() / 100.0) + "元");
    }
  }
}
```

开闭对扩展开放，对修改关闭，并不意味着不做任何修改，底层模块的变更，必然要有高层模块进行耦合，否则就是一个孤立无意义的代码片段。

我们可以吧变化归纳为以下三种类型：

* 逻辑变化：只变化一个逻辑，而不涉及其他模块，比如原有的一个算法是a*b+c，现在需要修改为a*b*c，可以通过修改原有类中的方法的方式来完成，前提条件是所有以来或关联类都按照相同的逻辑处理。
* 子模块变化：一个模块变化，会对其他的模块产生影响，特别是一个低层次的模块变化必然引起高层模块的变化，因此在通过扩展完成变化时，高层次的模块修改是必然的。
* 可见视图的变化

## why ocp

1. 开闭原则对测试的影响，增加扩展相关测试
2. 开闭原则可以提高复用性
3. 开闭原则可以提高可维护性
4. 面向对象开发的要求

## how ocp

1. 抽象约束
  通过接口或抽象类可以约束一组可能变化的行为，并且能够实现对扩展开放，其包含三层含义：
  * 通过接口或抽象类约束扩展，对扩展进行边界限定，不允许出现在接口或抽象类中不存在的public方法
  * 参数类型、引用对象尽量使用接口或抽象类，而不是实现类
  * 抽象层尽量保持稳定，一旦确定即不允许修改。

  增加业务品种后的书店售书类图

  ![](http://odr63yrh6.bkt.clouddn.com/OCP3.jpg)

  ```java
  public inteface IComputerBook extends IBook {
    //计算机书籍是有一个范围
    public String getScope();
  }
  ```

  计算机书籍类

  ```java
  public class ComputerBook implements IComputerBook {
    private String name;
    private String scope;
    private String author;
    private int price;
    public ComputerBook(String _name, int _price, String _author, String _scope) {
      this.name = _name;
      this.price = _price;
      this.author = _author;
      this.scope = _scope;
    }

    public String getScope() {
      return this.scope;
    }

    public String getAuthor() {
      return this.author;
    }

    public String getName() {
      return this.name;
    }

    public int getPrice() {
      return this.price;
    }
  }
  ```

  书店销售计算机书籍

  ```java
  public class BookStore {
    private final static ArrayList<IBook> bookList = new ArrayList<IBook>();
    static {
      bookList.add(new NovelBook("天龙八部", 3200, "金庸"));
      bookList.add(new NovelBook("巴黎圣母院", 5600, "雨果"));
      bookList.add(new NovelBook("悲惨世界", 3500, "雨果"));
      bookList.add(new NovelBook("金瓶梅", 4300, "兰陵笑笑生"));
      bookList.add(new ComputerBook("Think in Java", 4300, "Bruce Eckel", "编程语言"));
    }

    public static void main(String[] args) {
      NumberFormat formatter = NumberFormat.getCurrentcyInstance();
      formatter.setMaximumFractionDigits(2);
      System.out.println("------------书店卖出去的书籍记录如下：---------------");
      for(IBook book : bookList) {
        System.out.println("书籍名称：" + book.getName() + "\t书籍作者：" + book.getAuthor() + "\t书籍价格：" + formatter.format(book.getPrice() / 100.0) + "元");
      }
    }
  }
  ```
2. 元数据(metadata)控制模块行为
3. 指定项目章程
4. 封装变化
  * 将相同的变化封装到一个接口或抽象类中。
  * 将不同的变化封装到不同的接口或抽象类中，不应该有两个不同的变化出现在同一个接口或抽象类中。
