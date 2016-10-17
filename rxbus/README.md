# RxBus

> 基于RxJava的事件总线处理库, 能够很方便的进行事件分发及线程切换等

## Gradle依赖及项目地址

项目地址: [https://github.com/AndroidKnife/RxBus](https://github.com/AndroidKnife/RxBus)

Gradle依赖:

```
compile 'com.hwangjr.rxbus:rxbus:1.0.5'
```

## 使用方法

```java
public class MainActivity extends AppCompatActivity {
    ...

    @Override
    protected void onCreate(Bundle savedInstanceState) {
        ...
        RxBus.get().register(this); //注册当前页
        ...
    }

    @Override
    protected void onDestroy() {
        ...
        RxBus.get().unregister(this); //解除注册
        ...
    }

    /**
     * 消费事件
     */
    @Subscribe
    public void eat(String food) {
        // purpose
    }

    /**
     * 消费事件, 切换线程, 指定接收tag
     */
    @Subscribe(
        thread = EventThread.IO,
        tags = {
            @Tag(BusAction.EAT_MORE)
        }
    )
    public void eatMore(List<String> foods) {
        // purpose
    }

    /**
     * 发送事件及数据
     */
    @Produce
    public String produceFood() {
        return "This is bread!";
    }

    /**
     * 发送事件及数据, 切换线程, 指定接收tag
     */
    @Produce(
        thread = EventThread.IO,
        tags = {
            @Tag(BusAction.EAT_MORE)
        }
    )
    public List<String> produceMoreFood() {
        return Arrays.asList("This is breads!");
    }

    /**
     * 发送事件
     */
    public void post() {
        RxBus.get().post(this);
    }

    /**
     * 发送指定tag事件
     */
    public void postByTag() {
        RxBus.get().post(Constants.EventType.TAG_STORY, this);
    }
    ...
}
```
