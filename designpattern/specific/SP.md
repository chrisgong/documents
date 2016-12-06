# 单例模式
> Singleton Pattern

## 定义

确保某一个类只有一个实例，而且自行实例化并向整个系统提供这个实例。
> Ensure a class has only one instance, and provide a global point of access to it.

## 优点

* 减少了内存开支
* 减少性能开销
* 减少资源多重占用

## 缺点

* 扩展困难（无接口）
* 不利于测试，并行开发中，如果单例模式没有完成，是不能进行测试的，没有接口也不能使用mock的方式虚拟一个对象。
* 单例模式与单一职责原则有冲突。一个类应该只实现一个逻辑，而不关心它是否是单例，是不是单例取决于环境，单例模式把“要单例”和业务逻辑融合在一个类中。

## 使用场景

* 要求生成唯一序列号的环境
* 在整个项目中需要一个共享访问点或共享数据
* 创建一个对象需要消耗的资源过多
* 需要定义大量的静态常量和静态方法

## 注意事项

### 不安全单例

```java
public class Singleton {
  private static Singleton singleton = null;

  private Singleton() {
  }

  public static Singleton getSingleton() {
    if(singleton == null) {
      singleton = new Singleton();
    }
    return singleton;
  }
}
```

该单例模式在地并发的情况下尚不会出现问题，如果并发量增加时则可能在内存中出现多个实例。
如一个线程A执行到singleton = new Singleton()，但还没有获得对象，第二个线程B也在执行，执行到（singleton == null）判断，那么线程B获得判断条件也是为真，于是继续运行下去，线程A获得了一个对象，线程B也获得了一个对象，在内存中就出现两个对象。
