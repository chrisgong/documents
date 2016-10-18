Realm Java 让你能够高效地编写 app 的模型层代码，保证你的数据被安全、快速地存储。

# 前提

* 目前我们还不支持 Android 以外的 Java 环境；

* Android Studio >= 1.5.1

* JDK 版本 >=7

* Android API > 9

# 安装

Realm 作为一个 Gradle 插件来安装需要如下两个步骤：

## 第一步： 在项目的 build.gradle 文件中添加如下 class path 依赖。

```
buildscript {
    repositories {
        jcenter()
    }

    dependencies {
        classpath "io.realm:realm-gradle-plugin:1.1.0"
    }
}
```

## 第二步： 在 app 的 build.gradle 文件中应用 realm-android 插件。

```
apply plugin: 'realm-android'
```

> **Chrome Stetho支持**
> [Stetho](http://facebook.github.io/stetho/) 是 Facebook 提供的桥接安卓调试和 Chrome 浏览器的组件。
> [Stetho-Realm](https://github.com/uPhyca/stetho-realm) is a Realm module for Stetho.
> ```
> repositories {
>     maven {
>         url 'https://github.com/uPhyca/stetho-> realm/raw/master/maven-repo'
>     }
> }
>

> dependencies {
>     compile 'com.facebook.stetho:stetho:1.3.1'
>     compile 'com.uphyca:stetho_realm:0.9.0'
> }
> ```
