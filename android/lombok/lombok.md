# Lombok

Lombok核心特征是你需要用注解来创建代码，目的是减少你要写的样板代码的数量。它为你提供如下注解:

* [@Getter](http://projectlombok.org/features/GetterSetter.html) 和 [@Setter](http://projectlombok.org/features/GetterSetter.html): 为你的字段创建getter/setter
* [@EqualsAndHashCode](http://projectlombok.org/features/EqualsAndHashCode.html): 实现`equals()`和`hashCode()`
* [@ToString](http://projectlombok.org/features/ToString.html): 实现`toString()`
* [@Data](http://projectlombok.org/features/Data.html): 使用上面四个注解的特征
* [@Cleanup](http://projectlombok.org/features/Cleanup.html): 关闭流
* [@Synchronized](http://projectlombok.org/features/Synchronized.html): 对象上同步
* [@SneakyThrows](http://projectlombok.org/features/SneakyThrows.html): 抛出异常
* [@Builder/@Singular](https://projectlombok.org/features/Builder.html): 生成链式调用方法
* [@NoArgsConstructor/@RequiredArgsConstructor/@AllArgsConstructor](https://projectlombok.org/features/Constructor.html): 生成不同参数构造函数
* [@Accessors](https://projectlombok.org/features/experimental/Accessors.html): 处理setter/getter前缀, 调用方法等

项目地址: [https://github.com/rzwitserloot/lombok](https://github.com/rzwitserloot/lombok)

依赖:

```
compile "org.projectlombok:lombok:1.16.10"
```

## @Getter and @Setter

当你学习OO（面向对象）方式来编码，你经常是把你的字段设置为私有同时编写公共的访问器来访问这些字段：

```java
public class Person {
    private boolean employed = true;
    private String name;

    public boolean isEmployed() {
      return employed;
    }

    public void setEmployed(final boolean employed) {
      this.employed = employed;
    }

    protected void setName(final String name) {
      this.name = name;
    }
}
```

因为这种写法是很烦人的，一些（并非全部）IDE会有一个功能来生成访问器。它有一些缺点：
* 如果你移除一个字段，相应地你要手动移除访问器。
* 它会把你的真实代码和样板代码弄得混乱
* 结果：要花费精力在一个长的类里去检查这个字段是不是已经有访问器存在了

此外，手动创建访问器是比较容易出错的：有一次，我花费几个小时在同事写的代码里找一个bug，最后发现是setter写错了。

因为getter/setter仅仅是字段的元数据，Lombok的立场就是把这些方法看成这样：Java里的元数据都通过注解来管理。看下面的代码

```java
import org.lombok.Getter;
import org.lombok.Setter;

public class Person {
    @Getter @Setter private boolean employed = true;
    @Setter(AccessLevel.PROTECTED) private String name;
}
```

我反编译了生成的class文件（使用 [JAD](http://en.wikipedia.org/wiki/JAD_%28JAva_Decompiler%29)），发现它其实创建的字节码都是完全相同的，只是源代码更为简洁和更不容易产生错误。

## @NonNull

```java
@Getter @Setter @NonNull
private List<Person> members;
```

等效的Java代码:

```java
@NonNull
private List<Person> members;

public Family(@NonNull final List<Person> members) {
    if (members == null) throw new java.lang.NullPointerException("members");
    this.members = members;
}

@NonNull
public List<Person> getMembers() {
    return members;
}

public void setMembers(@NonNull final List<Person> members) {
    if (members == null) throw new java.lang.NullPointerException("members");
    this.members = members;
}
```

## @ToString

```java
@ToString(callSuper=true,exclude="someExcludedField")
public class Foo extends Bar {
    private boolean someBoolean = true;
    private String someStringField;
    private float someExcludedField;
}
```

等效的Java代码:

```java
public class Foo extends Bar {
    private boolean someBoolean = true;
    private String someStringField;
    private float someExcludedField;

    @java.lang.Override
    public java.lang.String toString() {
        return "Foo(super=" + super.toString() +
            ", someBoolean=" + someBoolean +
            ", someStringField=" + someStringField + ")";
    }
}
```

## @EqualsAndHashCode

```java
@EqualsAndHashCode(callSuper=true,exclude={"address","city","state","zip"})
public class Person extends SentientBeing {
    enum Gender { Male, Female }

    @NonNull private String name;
    @NonNull private Gender gender;

    private String ssn;
    private String address;
    private String city;
    private String state;
    private String zip;
}
```

等效的Java代码:

```java
public class Person extends SentientBeing {

    enum Gender {
        /*public static final*/ Male /* = new Gender() */,
        /*public static final*/ Female /* = new Gender() */;
    }
    @NonNull
    private String name;
    @NonNull
    private Gender gender;
    private String ssn;
    private String address;
    private String city;
    private String state;
    private String zip;

    @java.lang.Override
    public boolean equals(final java.lang.Object o) {
        if (o == this) return true;
        if (o == null) return false;
        if (o.getClass() != this.getClass()) return false;
        if (!super.equals(o)) return false;
        final Person other = (Person)o;
        if (this.name == null ? other.name != null : !this.name.equals(other.name)) return false;
        if (this.gender == null ? other.gender != null : !this.gender.equals(other.gender)) return false;
        if (this.ssn == null ? other.ssn != null : !this.ssn.equals(other.ssn)) return false;
        return true;
    }

    @java.lang.Override
    public int hashCode() {
        final int PRIME = 31;
        int result = 1;
        result = result * PRIME + super.hashCode();
        result = result * PRIME + (this.name == null ? 0 : this.name.hashCode());
        result = result * PRIME + (this.gender == null ? 0 : this.gender.hashCode());
        result = result * PRIME + (this.ssn == null ? 0 : this.ssn.hashCode());
        return result;
    }
}
```

## @Data

`@Data`注释可能是最常用的注释在项目里的Lombok工具集。它结合了`@ToString`的功能、`@EqualsAndHashCode`、`@Getter`和`@Setter`。


```java
@Data(staticConstructor="of") //静态工厂方法名
public class Company {
    private final Person founder;
    private String name;
    private List<Person> employees;
}
```

等效的Java代码:

```java
public class Company {
    private final Person founder;
    private String name;
    private List<Person> employees;

    private Company(final Person founder) {
        this.founder = founder;
    }

    public static Company of(final Person founder) {
        return new Company(founder);
    }

    public Person getFounder() {
        return founder;
    }

    public String getName() {
        return name;
    }

    public void setName(final String name) {
        this.name = name;
    }

    public List<Person> getEmployees() {
        return employees;
    }

    public void setEmployees(final List<Person> employees) {
        this.employees = employees;
    }

    @java.lang.Override
    public boolean equals(final java.lang.Object o) {
        if (o == this) return true;
        if (o == null) return false;
        if (o.getClass() != this.getClass()) return false;
        final Company other = (Company)o;
        if (this.founder == null ? other.founder != null : !this.founder.equals(other.founder)) return false;
        if (this.name == null ? other.name != null : !this.name.equals(other.name)) return false;
        if (this.employees == null ? other.employees != null : !this.employees.equals(other.employees)) return false;
        return true;
    }

    @java.lang.Override
    public int hashCode() {
        final int PRIME = 31;
        int result = 1;
        result = result * PRIME + (this.founder == null ? 0 : this.founder.hashCode());
        result = result * PRIME + (this.name == null ? 0 : this.name.hashCode());
        result = result * PRIME + (this.employees == null ? 0 : this.employees.hashCode());
        return result;
    }

    @java.lang.Override
    public java.lang.String toString() {
        return "Company(founder=" + founder + ", name=" + name + ", employees=" + employees + ")";
    }
}
```

## @Cleanup

关闭流

```java
public void testCleanUp() {
    try {
        @Cleanup ByteArrayOutputStream baos = new ByteArrayOutputStream();
        baos.write(new byte[] {'Y','e','s'});
        System.out.println(baos.toString());
    } catch (IOException e) {
        e.printStackTrace();
    }
}
```

等效的Java代码:

```java
public void testCleanUp() {
    try {
        ByteArrayOutputStream baos = new ByteArrayOutputStream();
        try {
            baos.write(new byte[]{'Y', 'e', 's'});
            System.out.println(baos.toString());
        } finally {
            baos.close();
        }
    } catch (IOException e) {
        e.printStackTrace();
    }
}
```

## @Synchronized

对象上同步

```java
private DateFormat format = new SimpleDateFormat("MM-dd-YYYY");

@Synchronized
public String synchronizedFormat(Date date) {
    return format.format(date);
}
```

等效的Java代码:

```java
private final java.lang.Object $lock = new java.lang.Object[0];
private DateFormat format = new SimpleDateFormat("MM-dd-YYYY");

public String synchronizedFormat(Date date) {
    synchronized ($lock) {
        return format.format(date);
    }
}
```

## @SneakyThrows

抛出异常

```java
@SneakyThrows
public void testSneakyThrows() {
    throw new IllegalAccessException();
}
```

等效的Java代码:

```java
public void testSneakyThrows() {
    try {
        throw new IllegalAccessException();
    } catch (java.lang.Throwable $ex) {
        throw lombok.Lombok.sneakyThrow($ex);
    }
}
```

## @NoArgsConstructor, @RequiredArgsConstructor and @AllArgsConstructor

构造函数定制

```java
import lombok.AccessLevel;
import lombok.RequiredArgsConstructor;
import lombok.AllArgsConstructor;
import lombok.NonNull;

@RequiredArgsConstructor(staticName = "of")
@AllArgsConstructor(access = AccessLevel.PROTECTED)
public class ConstructorExample<T> {
    private int x, y;
    @NonNull private T description;

    @NoArgsConstructor
    public static class NoArgsExample {
      @NonNull private String field;
    }
}
```

等效的Java代码:

```java
public class ConstructorExample<T> {
    private int x, y;
    @NonNull private T description;

    private ConstructorExample(T description) {
        if (description == null) throw new NullPointerException("description");
        this.description = description;
    }

    public static <T> ConstructorExample<T> of(T description) {
        return new ConstructorExample<T>(description);
    }

    @java.beans.ConstructorProperties({"x", "y", "description"})
    protected ConstructorExample(int x, int y, T description) {
        if (description == null) throw new NullPointerException("description");
        this.x = x;
        this.y = y;
        this.description = description;
    }

    public static class NoArgsExample {
        @NonNull private String field;

        public NoArgsExample() {
        }
    }
}
```

## @Builder, @Singular

`@Builder`注释产生复杂的构建器api类, 可使用链式调用:

`@Singular`对集合进行特殊处理, 将生成一个单数形式, 一个复数形式增加参数方法.

`@Singular`支持数据类型:


* java.util:
  * `Iterable`, `Collection`, and `List` (backed by a compacted unmodifiable ArrayList in the general case).
  * `Set`, `SortedSet`, and `NavigableSet` (backed by a smartly sized unmodifiable HashSet or TreeSet in the general case).
  * `Map`, `SortedMap`, and `NavigableMap` (backed by a smartly sized unmodifiable HashMap or TreeMap in the general case).
* Guava's com.google.common.collect:
  * `ImmutableCollection` and `ImmutableList` (backed by the builder feature of ImmutableList).
  * `ImmutableSet` and `ImmutableSortedSet` (backed by the builder feature of those types).
  * `ImmutableMap`, `ImmutableBiMap`, and `ImmutableSortedMap` (backed by the builder feature of those types).
  * `ImmutableTable` (backed by the builder feature of ImmutableTable).

```java
import lombok.Builder;
import lombok.Singular;
import java.util.Set;

@Builder
public class BuilderExample {
    private String name;
    private int age;
    @Singular private Set<String> occupations;
}
```

等效的Java代码:

```java
import java.util.Set;

public class BuilderExample {
    private String name;
    private int age;
    private Set<String> occupations;

    BuilderExample(String name, int age, Set<String> occupations) {
        this.name = name;
        this.age = age;
        this.occupations = occupations;
    }

    public static BuilderExampleBuilder builder() {
        return new BuilderExampleBuilder();
    }

    public static class BuilderExampleBuilder {
      	private String name;
      	private int age;
      	private java.util.ArrayList<String> occupations;

      	BuilderExampleBuilder() {
      	}

      	public BuilderExampleBuilder name(String name) {
        		this.name = name;
        		return this;
      	}

      	public BuilderExampleBuilder age(int age) {
        		this.age = age;
        		return this;
      	}

      	public BuilderExampleBuilder occupation(String occupation) {
        		if (this.occupations == null) {
        			this.occupations = new java.util.ArrayList<String>();
        		}

        		this.occupations.add(occupation);
        		return this;
      	}

      	public BuilderExampleBuilder occupations(Collection<? extends String> occupations) {
        		if (this.occupations == null) {
        			this.occupations = new java.util.ArrayList<String>();
        		}

        		this.occupations.addAll(occupations);
        		return this;
      	}

      	public BuilderExampleBuilder clearOccupations() {
        		if (this.occupations != null) {
        			this.occupations.clear();
        		}

        		return this;
      	}

      	public BuilderExample build() {
        		// complicated switch statement to produce a compact properly sized immutable set omitted.
        		// go to https://projectlombok.org/features/Singular-snippet.html to see it.
        		Set<String> occupations = ...;
        		return new BuilderExample(name, age, occupations);
      	}

      	@java.lang.Override
    		public String toString() {
    			return "BuilderExample.BuilderExampleBuilder(name = " + this.name + ", age = " + this.age + ", occupations = " + this.occupations + ")";
    		}
    }
}
```

注解后对象使用方法:

```java
BuilderExample.builder().name("Adam Savage").age(12).occupation("Japan").build();
```

## @Accessors

`@Accessors`注释用于配置Lombok生成和如何寻找getter和setter。

* fluent:  If true, the getter for pepper is just pepper(), and the setter is pepper(T newValue).
* chain
* prefix: 剔除定义前缀后生成getter和setter

```java
import lombok.experimental.Accessors;
import lombok.Getter;
import lombok.Setter;

@Accessors(fluent = true)
public class AccessorsExample {
    @Getter @Setter
    private int age = 10;
}

class PrefixExample {
    @Accessors(prefix = "f") @Getter
    private String fName = "Hello, World!";
}
```

等效的Java代码:

```java
public class AccessorsExample {
    private int age = 10;

    public int age() {
        return this.age;
    }

    public AccessorsExample age(final int age) {
        this.age = age;
        return this;
    }
}

class PrefixExample {
    private String fName = "Hello, World!";

    public String getName() {
        return this.fName;
    }
}
```
