# Robolectric

## 关于Robolectric

作为一个软件开发攻城狮，无论你多不屑多排斥单元测试，它都是一种非常好的开发方式，且不谈TDD，为自己写的代码负责，测试自己写的代码，在自己力所能及的范围内提高产品的质量，本是理所当然的事情。

那么如何测试自己写的代码？点点界面，测测功能固然是一种方式，但是如果能留下一段一劳永逸的测试代码，让代码测试代码，岂不两全其美？所以，写好单元测试，爱惜自己的代码，爱惜颜值高的QA妹纸，爱惜有价值的产品，人人有责！

## 环境搭建

### Gradle配置

在build.gradle中配置如下依赖关系：

```java
testCompile "org.robolectric:robolectric:3.0"
```

### 通过注解配置TestRunner

```java
@RunWith(RobolectricGradleTestRunner.class)
@Config(constants = BuildConfig.class)
public class SampleActivityTest {

}
```

### Android Studio的配置

1. 在Build Variants面板中，将Test Artifact切换成Unit Tests模式，如下图：

![](http://img4.07net01.com/upload/images/2016/02/17/2080330171030102.png)

配置Test Artifact

2. working directory 设置为$MODULE_DIR$

如果在测试过程遇见如下问题，解决的方式就是设置working directory的值：

```java
Java.io.FileNotFoundException: build\intermediates\bundles\debug\AndroidManifest.xml (系统找不到指定的路径。)
```

设置方法如下图所示:

![](http://img4.07net01.com/upload/images/2016/02/17/2080330171030103.png)

Edit Configurations

![](http://img4.07net01.com/upload/images/2016/02/17/2080330171030104.png)

Working directory的配置
更多环境配置可以参考[官方网站](http://robolectric.org)。

## Activity的测试

1. 创建Activity实例

```java
@Test
public void testActivity() {
    SampleActivity sampleActivity = Robolectric.setupActivity(SampleActivity.class);
    assertNotNull(sampleActivity);
    assertEquals(sampleActivity.getTitle(), "SimpleActivity");
}
```

2. 生命周期

```java
@Test
public void testLifecycle() {
    ActivityController<SampleActivity> activityController = Robolectric.buildActivity(SampleActivity.class).create().start();
    Activity activity = activityController.get();
    TextView textview = (TextView) activity.findViewById(R.id.tv_lifecycle_value);
    assertEquals("onCreate",textview.getText().toString());
    activityController.resume();
    assertEquals("onResume", textview.getText().toString());
    activityController.destroy();
    assertEquals("onDestroy", textview.getText().toString());
}
```

3. 跳转

```java
@Test
public void testStartActivity() {
    //按钮点击后跳转到下一个Activity
    forwardBtn.performClick();
    Intent expectedIntent = new Intent(sampleActivity, LoginActivity.class);
    Intent actualIntent = ShadowApplication.getInstance().getNextStartedActivity();
    assertEquals(expectedIntent, actualIntent);
}
```

4. UI组件状态

```java
@Test
public void testViewState(){
    CheckBox checkBox = (CheckBox) sampleActivity.findViewById(R.id.checkbox);
    Button inverseBtn = (Button) sampleActivity.findViewById(R.id.btn_inverse);
    assertTrue(inverseBtn.isEnabled());

    checkBox.setChecked(true);
    //点击按钮，CheckBox反选
    inverseBtn.performClick();
    assertTrue(!checkBox.isChecked());
    inverseBtn.performClick();
    assertTrue(checkBox.isChecked());
}
```

5. Dialog

```java
@Test
public void testDialog(){
    //点击按钮，出现对话框
    dialogBtn.performClick();
    AlertDialog latestAlertDialog = ShadowAlertDialog.getLatestAlertDialog();
    assertNotNull(latestAlertDialog);
}
```

6. Toast

```java
@Test
public void testToast(){
    //点击按钮，出现吐司
    toastBtn.performClick();
    assertEquals(ShadowToast.getTextOfLatestToast(),"we love UT");
}
```

7. Fragment的测试

如果使用support的Fragment，需添加以下依赖

```java
testCompile "org.robolectric:shadows-support-v4:3.0"
```

shadow-support包提供了将Fragment主动添加到Activity中的方法：SupportFragmentTestUtil.startFragment(),简易的测试代码如下

```java
@Test
public void testFragment(){
    SampleFragment sampleFragment = new SampleFragment();
    //此api可以主动添加Fragment到Activity中，因此会触发Fragment的onCreateView()
    SupportFragmentTestUtil.startFragment(sampleFragment);
    assertNotNull(sampleFragment.getView());
}
```

8. 访问资源文件

```java
@Test
public void testResources() {
    Application application = RuntimeEnvironment.application;
    String appName = application.getString(R.string.app_name);
    String activityTitle = application.getString(R.string.title_activity_simple);
    assertEquals("LoveUT", appName);
    assertEquals("SimpleActivity",activityTitle);
}
```

## BroadcastReceiver的测试

首先看下广播接收者的代码

```java
public class MyReceiver extends BroadcastReceiver {
    @Override
    public void onReceive(Context context, Intent intent) {
        SharedPreferences.Editor editor = context.getSharedPreferences(
                "account", Context.MODE_PRIVATE).edit();
        String name = intent.getStringExtra("EXTRA_USERNAME");
        editor.putString("USERNAME", name);
        editor.apply();
    }
}
```

广播的测试点可以包含两个方面，一是应用程序是否注册了该广播，二是广播接受者的处理逻辑是否正确，关于逻辑是否正确，可以直接人为的触发onReceive()方法，验证执行后所影响到的数据。

```java
@Test
public void testBoradcast(){
    ShadowApplication shadowApplication = ShadowApplication.getInstance();

    String action = "com.geniusmart.loveut.login";
    Intent intent = new Intent(action);
    intent.putExtra("EXTRA_USERNAME", "geniusmart");

    //测试是否注册广播接收者
    assertTrue(shadowApplication.hasReceiverForIntent(intent));

    //以下测试广播接受者的处理逻辑是否正确
    MyReceiver myReceiver = new MyReceiver();
    myReceiver.onReceive(RuntimeEnvironment.application,intent);
    SharedPreferences preferences = shadowApplication.getSharedPreferences("account", Context.MODE_PRIVATE);
    assertEquals( "geniusmart",preferences.getString("USERNAME", ""));
}
```

## Service的测试

Service的测试类似于BroadcastReceiver，以IntentService为例，可以直接触发onHandleIntent()方法，用来验证Service启动后的逻辑是否正确。

```java
public class SampleIntentService extends IntentService {
    public SampleIntentService() {
        super("SampleIntentService");
    }

    @Override
    protected void onHandleIntent(Intent intent) {
        SharedPreferences.Editor editor = getApplicationContext().getSharedPreferences(
                "example", Context.MODE_PRIVATE).edit();
        editor.putString("SAMPLE_DATA", "sample data");
        editor.apply();
    }
}
```

以上代码的单元测试用例：

```java
@Test
public void addsDataToSharedPreference() {
    Application application = RuntimeEnvironment.application;
    RoboSharedPreferences preferences = (RoboSharedPreferences) application
            .getSharedPreferences("example", Context.MODE_PRIVATE);

    SampleIntentService registrationService = new SampleIntentService();
    registrationService.onHandleIntent(new Intent());

    assertEquals(preferences.getString("SAMPLE_DATA", ""), "sample data");
}
```

## Shadow的使用

Shadow是Robolectric的立足之本，如其名，作为影子，一定是变幻莫测，时有时无，且依存于本尊。因此，框架针对Android SDK中的对象，提供了很多影子对象（如Activity和ShadowActivity、TextView和ShadowTextView等），这些影子对象，丰富了本尊的行为，能更方便的对Android相关的对象进行测试。

1. 使用框架提供的Shadow对象

```java
@Test
public void testDefaultShadow(){
    MainActivity mainActivity = Robolectric.setupActivity(MainActivity.class);

    //通过Shadows.shadowOf()可以获取很多Android对象的Shadow对象
    ShadowActivity shadowActivity = Shadows.shadowOf(mainActivity);
    ShadowApplication shadowApplication = Shadows.shadowOf(RuntimeEnvironment.application);

    Bitmap bitmap = BitmapFactory.decodeFile("Path");
    ShadowBitmap shadowBitmap = Shadows.shadowOf(bitmap);

    //Shadow对象提供方便我们用于模拟业务场景进行测试的api
    assertNull(shadowActivity.getNextStartedActivity());
    assertNull(shadowApplication.getNextStartedActivity());
    assertNotNull(shadowBitmap);
}
```

2. 如何自定义Shadow对象

```java
public class Person {
    private String name;
    public Person(String name) {
        this.name = name;
    }
    public String getName() {
        return name;
    }
}
```

其次，创建Person的Shadow对象

```java
@Implements(Person.class)
public class ShadowPerson {

    @Implementation
    public String getName() {
        return "geniusmart";
    }
}
```

接下来，需自定义TestRunner，添加Person对象为要进行Shadow的对象

```java
public class CustomShadowTestRunner extends RobolectricGradleTestRunner {

    public CustomShadowTestRunner(Class<?> klass) throws InitializationError {
        super(klass);
    }

    @Override
    public InstrumentationConfiguration createClassLoaderConfig() {
        InstrumentationConfiguration.Builder builder = InstrumentationConfiguration.newBuilder();
        /**
         * 添加要进行Shadow的对象
         */
        builder.addInstrumentedClass(Person.class.getName());
        return builder.build();
    }
}
```

最后，在测试用例中，ShadowPerson对象将自动代替原始对象，调用Shadow对象的数据和行为

```java
@RunWith(CustomShadowTestRunner.class)
@Config(constants = BuildConfig.class,shadows = {ShadowPerson.class})
public class ShadowTest {

    /**
     * 测试自定义的Shadow
     */
    @Test
    public void testCustomShadow(){
        Person person = new Person("genius");
        //getName()实际上调用的是ShadowPerson的方法
        assertEquals("geniusmart", person.getName());

        //获取Person对象对应的Shadow对象
        ShadowPerson shadowPerson = (ShadowPerson) ShadowExtractor.extract(person);
        assertEquals("geniusmart", shadowPerson.getName());
    }
}
```
