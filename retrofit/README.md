# Retrofit – Java(Android) 的REST 接口封装类库

Retrofit  和Java领域的ORM概念类似， ORM把结构化数据转换为Java对象，而Retrofit 把REST API返回的数据转化为Java对象方便操作。同时还封装了网络代码的调用。

项目主页: [http://square.github.io/retrofit/](http://square.github.io/retrofit/)

例如:

```java
public interface GitHubService {
  @GET("/users/{user}/repos")
  List<Repo> listRepos(@Path("user") String user);
}
```

定义上面的一个REST API接口。 该接口定义了一个函数 listRepos ,  该函数会通过HTTP GET请求去访问服务器的/users/{user}/repos路径并把返回的结果封装为List<Repo> Java对象返回。
其中URL路径中的{user}的值为listRepos 函数中的参数 user的取值。
然后通过 RestAdapter 类来生成一个 GitHubService 接口的实现；

```java
GitHubService service = restAdapter.create(GitHubService.class);
```

获取接口的实现后就可以调用接口函数来和服务器交互了；

```java
List<Repo> repos = service.listRepos("octocat");
```

从上面的示例可以看出， Retrofit 使用注解来声明HTTP请求
* 支持 URL 参数替换和查询参数
* 返回结果转换为Java对象(返回结果可以为  JSON, protocol buffers)
* 支持 Multipart请求和文件上传

## 具体使用文档

函数和函数参数上的注解声明了请求方式

### 请求方式

每个函数都必须带有 HTTP 注解来表明请求方式和请求的URL路径。类库中有5个HTTP注解: `GET`, `POST`, `PUT`, `DELETE`, 和 `HEAD`。 注解中的参数为请求的相对URL路径

```java
@GET("/users/list")
```

在URL路径中也可以指定URL参数

```java
@GET("/users/list?sort=desc")
```

### URL处理

请求的URL可以根据函数参数动态更新。一个可替换的区块为用{ 和 }包围的字符串，而函数参数必需用 `@Path` 注解表明，并且注解的参数为同样的字符串

```java
//注意 字符串id
//注意 Path注解的参数要和前面的字符串一样 id
@GET("/group/{id}/users")
List<User> groupList(@Path("id") int groupId);
```

还支持查询参数

```java
@GET("/group/{id}/users")
List<User> groupList(@Path("id") int groupId, @Query("sort") String sort, @Query("key") String key);
```
> 实际接口格式: /group/xxx/users?sort=xxx&key=xxx

### 请求体（Request Body）

通过 `@Body` 注解可以声明一个对象作为请求体发送到服务器。

```java
@POST("/users/new")
void createUser(@Body User user, Callback<User> cb);
```

对象将被 RestAdapter使用对应的转换器转换为字符串或者字节流提交到服务器。

> ```java
addConverterFactory(GsonConverterFactory.create())
```
增加`Gson`转换支持后`Body`可以直接序列化为`Json`格式参数
例如:
> ```java
> public class LoginRequest {
>     public long phone;
>     public String password;
>     public LoginRequest(long phone, String password) {
>         this.phone = phone;
>         this.password = password;
>     }
> }
> ```
> ```java
> //此处boby source为{"phone": xxxxxxxx, "password": xxxxxxx}
> ApiUtils.getKintonInstance().login(request)
>                 .subscribeOn(Schedulers.io())
>                 .observeOn(AndroidSchedulers.mainThread())
>                 .subscribe(response -> {}, error -> {});
> ```

### FORM ENCODED AND MULTIPART 表单和Multipart

函数也可以注解为发送表单数据和multipart 数据

使用 `@FormUrlEncoded` 注解来发送表单数据；使用 @Field注解和参数来指定每个表单项的Key，value为参数的值。

```java
@FormUrlEncoded
@POST("/user/edit")
User updateUser(@Field("first_name") String first, @Field("last_name") String last);
```
使用 `@Multipart` 注解来发送multipart数据。使用 `@Part` 注解定义要发送的每个文件。

```java
@Multipart
@PUT("/user/photo")
User updateUser(@Part("photo") TypedFile photo, @Part("description") TypedString description);
```

Multipart 中的Part使用 RestAdapter 的转换器来转换，也可以实现 TypedOutput 来自己处理序列化。

### 异步 VS 同步

每个函数可以定义为异步或者同步。
具有返回值的函数为同步执行的。

```java
@GET("/user/{id}/photo")
Photo listUsers(@Path("id") int id);
```

而异步执行函数没有返回值并且要求函数最后一个参数为Callback对象

```java
@GET("/user/{id}/photo")
void listUsers(@Path("id") int id, Callback<Photo> cb);
```

在 Android 上，callback对象会在主(UI)线程中调用。而在普通Java应用中，callback在请求执行的线程中调用。
服务器结果转换为Java对象
使用RestAdapter的转换器把HTTP请求结果（默认为JSON）转换为Java对象，Java对象通过函数返回值或者Callback接口定义

```java
@GET("/users/list")
List<User> userList();

@GET("/users/list")
void userList(Callback<List<User>> cb);
```

如果要直接获取HTTP返回的对象，使用 Response 对象。

```java
@GET("/users/list")
Response userList();

@GET("/users/list")
void userList(Callback<Response> cb);
```

### HEADER设置

你可以使用`@Headers`注释。

```java
@Headers("Cache-Control: max-age=640000")
@GET("widget/list")
Call<List<Widget>> widgetList();
```

```java
@Headers({
    "Accept: application/vnd.github.v3.full+json",
    "User-Agent: Retrofit-Sample-App"
})
@GET("users/{username}")
Call<User> getUser(@Path("username") String username);
```
> 注意: Headers不会互相覆盖。

可以使用`@Header`动态更新头内容

```java
@GET("user")
Call<User> getUser(@Header("Authorization") String authorization)
```
> Tip: 如果希望头设置对全部接口生效(例如统计等), 可以对Okhttp设置自定义拦截器(addInterceptor(new StatisticsInterceptor()));

### Rxjava支持

如果想对操作结果Rx操作, 实现如下:

```java
Retrofit.Builder retrofitBuilder = new Retrofit.Builder()
                        .addCallAdapterFactory(RxJavaCallAdapterFactory.create()) //rx回调支持
                        .addConverterFactory(GsonConverterFactory.create()) //gson转化支持
                        .baseUrl(host);
```

```java
@POST("/user/feedback")
Observable<BaseResult> uploadFeedBackInfo(@Body FeedBackInfoRequest body);
```

```java
ApiUtils.getKintonInstance()
               .uploadFeedBackInfo(new FeedBackInfoRequest(mToken, feedback, contact, "cba"))
               .subscribeOn(Schedulers.io())
               .observeOn(AndroidSchedulers.mainThread())
               .subscribe(response -> {
                  //...
               }, error -> {
                   //...
               });
```
