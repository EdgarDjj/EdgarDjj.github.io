# Java注解


# Java注解

## 前言

实际上Java注解与普通修饰符(public、static、void等)的使用方式并没有多大区别，下面的例子是常见的注解：

```java
public class AnnotationDemo {
    //@Test注解修饰方法A
    @Test
    public static void A(){
        System.out.println("Test.....");
    }

    //一个方法上可以拥有多个不同的注解
    @Deprecated
    @SuppressWarnings("uncheck")
    public static void B(){

    }
}
```

通过在方法上使用@Test注解后，在运行该方法时，测试框架会自动识别该方法并单独调用，@Test实际上是一种标记注解，起标记作用，运行时告诉测试框架该方法为测试方法。而对于@Deprecated和@SuppressWarnings(“uncheck”)，则是Java本身内置的注解，在代码中，可以经常看见它们，但这并不是一件好事，毕竟当方法或是类上面有@Deprecated注解时，说明该方法或是类都已经过期不建议再用，@SuppressWarnings 则表示忽略指定警告，比如@SuppressWarnings(“uncheck”)，这就是注解的最简单的使用方式，那么下面我们就来看看注解定义的基本语法

> 想象代码具有生命，注解就是对于代码中某些鲜活的个体贴上去的一张标签。

## 基本语法

### 声明注解和元注解

#### 注解的定义

注解通过 `@interface` 来进行定义

```
public @interface Test {

}
```

我们先来看看前面的Test注解是如何声明的：

```java
//声明Test注解
@Target(ElementType.METHOD)
@Retention(RetentionPolicy.RUNTIME)
public @interface Test {

}
```

我们使用了`@interface`声明了Test注解，并使用`@Target`注解传入`ElementType.METHOD`参数来标明@Test只能用于方法上，`@Retention(RetentionPolicy.RUNTIME)`则用来表示该注解生存期是运行时，从代码上看注解的定义很像接口的定义，确实如此，毕竟在编译后也会生成Test.class文件。对于`@Target`和`@Retention`是由Java提供的元注解，所谓元注解就是标记其他注解的注解

- @Target 用来约束注解可以应用的地方（如方法、类或字段），其中ElementType是枚举类型，其定义如下，也代表可能的取值范围

  ```java
  public enum ElementType {
      /**标明该注解可以用于类、接口（包括注解类型）或enum声明*/
      TYPE,
  
      /** 标明该注解可以用于字段(域)声明，包括enum实例 */
      FIELD,
  
      /** 标明该注解可以用于方法声明 */
      METHOD,
  
      /** 标明该注解可以用于参数声明 */
      PARAMETER,
  
      /** 标明注解可以用于构造函数声明 */
      CONSTRUCTOR,
  
      /** 标明注解可以用于局部变量声明 */
      LOCAL_VARIABLE,
  
      /** 标明注解可以用于注解声明(应用于另一个注解上)*/
      ANNOTATION_TYPE,
  
      /** 标明注解可以用于包声明 */
      PACKAGE,
  
      /**
       * 标明注解可以用于类型参数声明（1.8新加入）
       * @since 1.8
       */
      TYPE_PARAMETER,
  
      /**
       * 类型使用声明（1.8新加入)
       * @since 1.8
       */
      TYPE_USE
  
  ```

  请注意，当注解未指定Target值时，则此注解可以用于任何元素之上，多个值使用{}包含并用逗号隔开，如下：

  ```java
  @Target(value={CONSTRUCTOR, FIELD, LOCAL_VARIABLE, METHOD, PACKAGE, PARAMETER, TYPE}
  ```

- **@Retention用来约束注解的生命周期**，分别有三个值，源码级别（source），类文件级别（class）或者运行时级别（runtime），其含有如下：

  - SOURCE：注解将被编译器丢弃（该类型的注解信息只会保留在源码里，源码经过编译后，注解信息会被丢弃，不会保留在编译好的class文件里）
  - CLASS：注解在class文件中可用，但会被VM丢弃（该类型的注解信息会保留在源码里和class文件里，在执行的时候，不会加载到虚拟机中），请注意，当注解未定义Retention值时，默认值是CLASS，如Java内置注解，@Override、@Deprecated、@SuppressWarnning等
  - RUNTIME：注解信息将在运行期(JVM)也保留，因此可以通过反射机制读取注解的信息（源码、class文件和执行的时候都有注解的信息），如SpringMvc中的@Controller、@Autowired、@RequestMapping等。

而这个两个注解@Rentention和@Target都是元注解，是注解在注解上的注解。

元注解有5个：

* @Retention：用来约束注解的生命周期
* @Target：标注注解所能运用的地方
* @Documented：能将注解中的元素包含到Javadoc中去
* @Inherited：继承的意思，就是说如果一个父类被@Inherited注解过的注解进行注解的话，那么他的子类没有任何注解应用的话，那么这个子类就是继承了父类的注解
* @Repeatable：是Java8加入进来，即注解的值可以取多个，例如一个人可以是程序员，画家，数学家等等

### 注解元素及其数据结构类型

注解支持的元素数据类型除了基本的数据类型，还包含如下：

* String
* Class
* enum
* Anootation

除此以外，使用其他数据类型编译器会报错。

#### 编译器对默认值的限制

1. 元素不能有不确定的值，就是说，元素要么具有默认值，要么在声明的时候定义默认值，无论如何都不能以null作为值

### Java内置注解与其他元注解

- @Override：用于标明此方法覆盖了父类的方法，源码如下

  ```
  @Target(ElementType.METHOD)
  @Retention(RetentionPolicy.SOURCE)
  public @interface Override {
  }
  ```

- @Deprecated：用于标明已经过时的方法或类，源码如下

  ```java
  @Documented
  @Retention(RetentionPolicy.RUNTIME)
  @Target(value={CONSTRUCTOR, FIELD, LOCAL_VARIABLE, METHOD, PACKAGE, PARAMETER, TYPE})
  public @interface Deprecated {
  }
  ```

- @SuppressWarnnings：用于有选择的关闭编译器对类、方法、成员变量、变量初始化的警告，其实现源码如下：

  ```java
  @Target({TYPE, FIELD, METHOD, PARAMETER, CONSTRUCTOR, LOCAL_VARIABLE})
  @Retention(RetentionPolicy.SOURCE)
  public @interface SuppressWarnings {
      String[] value();
  }
  ```

  其内部有一个String数组，主要接收值如下：

  ```
  deprecation：使用了不赞成使用的类或方法时的警告；
  unchecked：执行了未检查的转换时的警告，例如当使用集合时没有用泛型 (Generics) 来指定集合保存的类型; 
  fallthrough：当 Switch 程序块直接通往下一种情况而没有 Break 时的警告;
  path：在类路径、源文件路径等中有不存在的路径时的警告; 
  serial：当在可序列化的类上缺少 serialVersionUID 定义时的警告; 
  finally：任何 finally 子句不能正常完成时的警告; 
  all：关于以上所有情况的警告。
  ```


