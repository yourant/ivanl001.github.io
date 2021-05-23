[toc]

<extoc><extoc>



> 注解(Annotation), 是代码级别的说明, 是JDK1.5之后的新特性, 与类，接口，枚举是在同一层次。它可以声明在包，类，字段，方法，局部变量，方法参数等前面，用于对这些元素进行说明，注释。



## 1, annotation作用

### 1.1, 编写文档

* 通过代码里标识的元数据生成文档

### 1.2, 代码分析

* 通过代码里标识的元数据对代码进行分析[使用反射]

### 1.3, 编译检查

* 通过代码里标识的元数据让编译器能够实现基本的编译检查



## 2, JDK中预定义注解

### 2.1, @Override

* 继承父类方法

### 2.2, @Deprecated

* 过期方法

### 2.3, @SuppressWarnings

* 压制警告



## 3, 注解定义

### 3.1, 注解的本质

```java
@Target({ElementType.METHOD,ElementType.FIELD})
public @interface AnnoDemo01 {
  	// 这里其实是成员方法，因为使用的时候按照属性设置，所以也称为注解的成员属性
    int sex();
}
```



* 通过编译注解，然后再反编译(javap)出来，可以看到注解其实就是一个接口

```java

Compiled from "AnnoDemo01.java"
// 注解的本质就是一个接口，该接口默认继承java.lang.annotation.Annotation
public interface org.bool.a_annotation.AnnoDemo01 extends java.lang.annotation.Annotation {
  public abstract int sex();
}
```

### 3.2, 注解属性

* 注解内部的抽象方法叫做注解的属性

#### 3.2.1, 属性返回值类型

* 8大基本类型：byte short  int long  float double boolean char
* String
* 枚举
* 注解
* 以上类型的数组

#### 3.2.2, 属性value

* ```java
  @AnnoDemo01(sex=1)
  @AnnoDemo01(value=1)
  ```

* 如果只有一个属性名称叫做value，则注解赋值的时候，value=可以省略。

* ```java
  @AnnoDemo01(1)
  
  // ---------
  @AnnoDemo01(1)
  ```

### 3.3, 元注解

* @Target表明注解的作用范围
  * ElementType.TYPE ： 作用在类上
  * ElementType.METHOD： 作用在方法上
  * ElementType.FIELD：作用在成员变量上

```java
@Target({ElementType.METHOD,ElementType.FIELD})
public @interface AnnoDemo01 {
  
  	// 这里其实是成员方法，因为使用的时候按照属性设置，所以也称为注解的成员属性
    int sex();
}
```

* @Retention表明注解被保留的阶段
  * RetentionPolicy.SOURCE ： 只保留到源码阶段，不会进入字节码class文件中
  * RetentionPolicy.CLASS     ：保留到class文件中，但不会被jvm读取到
  * RetentionPolicy.RUNTIME ：  保留到运行时状态，并会被JVM读取到
* @Documented表明注解是否将被抽取到api文档中

## 4, 注解的解析使用

* 注解大多数都是替换配置文件的

* 注解给谁用：
  * 编译器
  * 给解析程序用
* 注解不是程序的一部分， 只是相当于是一个标签。注解大多时候只是帮助生成正确程序的一个手段而已

* 使用的代码参考
  * [04-reflect和annotation综合案例-实现失败重试](04-reflect和annotation综合案例-实现失败重试.md)

