## 保存和恢复状态

一种测试Activities/Fragments保存和恢复状态的方法就是在你的Espresso测试中旋转屏幕.

```java
private void rotateScreen() {
    Context context = InstrumentationRegistry.getTargetContext();
    int orientation = context.getResources().getConfiguration().orientation;

    Activity activity = activityRule.getActivity();
    activity.setRequestedOrientation(
            (orientation == Configuration.ORIENTATION_PORTRAIT) ?
            ActivityInfo.SCREEN_ORIENTATION_LANDSCAPE :
            ActivityInfo.SCREEN_ORIENTATION_PORTRAIT);
}
```

有了上面的辅助方法,你可以写出如下的测试:

```java
@Test
public void incrementTwiceAndRotateScreen() {
    onView(withId(R.id.count))
        .check(matches(withText("0")));

    onView(withId(R.id.increment_button))
        .perform(click(), click());
    onView(withId(R.id.count))
        .check(matches(withText("2")));

    rotateScreen();

    onView(withId(R.id.count))
        .check(matches(withText("2")));
}
```
