[toc]

<extoc></extoc>



# 失败重试注解案例

## RetryMethod

* 主函数
* 这个方法会调用方法， 如果失败(指定异常)，重试指定次数。并且按照指数时间间隔重试

```java
package org.bool.a04_ano_and_reflect;

import java.lang.reflect.Method;

/**
 * @author : 不二
 * @date : 2021/4/27-下午4:00
 * @desc : retry演示
 **/
public class RetryMain {

    public static void main(String[] args) throws Exception {

        /*RetryMethod retryMethod = new RetryMethod();
        retryMethod.retryTest();*/

        // 获取类
        Class<?> retryMethodClass = Class.forName("org.bool.a04_ano_and_reflect.RetryMethod");

        // 通过类的方法
        Method retryTestMethod = retryMethodClass.getDeclaredMethod("retryTest");

        // 拿到方法上的注解
        Retryable retryableAnno = retryTestMethod.getAnnotation(Retryable.class);
        // 得到注解里面的内容
        System.out.println("---------------------解析注解开始---------------------");
        int i = retryableAnno.maxAttempts();
        System.out.println("重试次数是:" + i);
        Class<? extends Throwable> value = retryableAnno.value();
        System.out.println("value是：" + value);
        System.out.println("---------------------解析注解结束---------------------");


        // 无参构造创建实例，来演示如何通过注解来实现失败重试
        int times = 0;
        Object retryObj = retryMethodClass.newInstance();

        // 重复调用方法，直到达到重试次数或者成功
        while (true) {
            try {
                times++;
                retryTestMethod.invoke(retryObj);
                // 如果正常运行则跳出运行
                System.out.println("执行成功，不需要再重试");
                break;
            } catch (Exception e) {
                if (times >= i || e.getClass().isAssignableFrom(retryableAnno.value())) {
                    // 如果重试次数或者异常类型不匹配， 也直接跳出异常
                    break;
                }
                // 第一次失败，隔1s重试， 第二次失败，隔2s重试， 第三次隔4s，第四次隔8s
                Thread.sleep(1000 * times);
            }
        }
    }
}
```



## Retryable.java

* 先写注解类

```java
package org.bool.a04_ano_and_reflect;

import java.lang.annotation.*;

/**
 * #Author : 不二
 * #Date   : 11/18/2019-4:21 PM
 * #Desc   : 失败重试的一个注解，不过后来没有用上，直接使用了springboot的重试注解
 *
 * @author ivanl001*/
@Target({ElementType.METHOD})
@Retention(RetentionPolicy.RUNTIME)
@Documented
public @interface Retryable {

    /**
     * Exception type that are retryable.
     * @return exception type to retry
     * 这里设定了一个默认值，就是RuntimeException。如果想要拦截其他异常，可以在添加注解的时候进行设置。
     */
    Class<? extends Throwable> value() default RuntimeException.class;

    /**
     * 包含第一次失败
     * @return the maximum number of attempts (including the first failure), defaults to 3
     */
    int maxAttempts() default 3;

}
```



## RetryMethod

* 用于失败重试的一个类

```java
package org.bool.a04_ano_and_reflect;

/**
 * @author : 不二
 * @date : 2021/4/27-下午3:59
 * @desc : 重试注解
 **/
public class RetryMethod {

    int i = 0;

    @Retryable(maxAttempts = 5)
    public void retryTest() {
        System.out.println("执行次数：" + ++i);
        if (i == 3) {
            System.out.println("该次成功----");
        } else {
            throw new RuntimeException("抛出异常啦");
        }
//         throw new RuntimeException("抛出异常啦");
    }
}
```



# 代码校验注解

## CheckMain

```java
package org.bool.a04_ano_and_reflect.checkDemo;

import java.io.BufferedWriter;
import java.io.FileWriter;
import java.lang.reflect.Method;

/**
 * @author : 不二
 * @date : 2021/4/27-下午5:45
 * @desc : check注解代码
 **/
public class CheckMain {

    public static void main(String[] args) throws Exception {

        BufferedWriter bufferedWriter = new BufferedWriter(new FileWriter("bug.txt", true));

        // 通过路径创建对象
        Class<?> calculatorClass = Class.forName("org.bool.a04_ano_and_reflect.checkDemo.Calculator");
        Object calculatorOjb = calculatorClass.newInstance();

        // 通过对象拿到定义的方法
        Method[] declaredMethods = calculatorClass.getDeclaredMethods();
        for (Method declaredMethod : declaredMethods) {
            System.out.println("对应的方法有：" + declaredMethod.getName());

            // 这里验证是否有注解
            /*Check annotation = declaredMethod.getAnnotation(Check.class);
            System.out.println("annotation注释：" + annotation);
            if (annotation != null) {
                System.out.println("含有注解的方法有：" + declaredMethod.getName());
            }*/

            // 还有一个更简单的方法
            if (declaredMethod.isAnnotationPresent(Check.class)) {
                System.out.println("---------------含有注解的方法有：---------------" + declaredMethod.getName());

                // 如果有注解，那么就调用一下进行测试
                try {
                    // declaredMethod.invoke(calculatorClass.newInstance());
                    declaredMethod.invoke(calculatorClass.newInstance());
                } catch (Exception e) {
                    System.out.println("异常了。。。");
                    // e.printStackTrace();
                    bufferedWriter.write(e.getCause().getClass().getSimpleName());
                    bufferedWriter.newLine();
                    bufferedWriter.write(e.getCause().getMessage());

                }
            }
        }

        bufferedWriter.flush();
        bufferedWriter.close();
    }
}
```



## Check注解类

```java
package org.bool.a04_ano_and_reflect.checkDemo;


import java.lang.annotation.*;

@Target(ElementType.METHOD)
@Retention(RetentionPolicy.RUNTIME)
@Documented
public @interface Check {
}
```



## Calculator类

```java
package org.bool.a04_ano_and_reflect.checkDemo;

/**
 * @author : 不二
 * @date : 2021/4/27-下午5:36
 * @desc : 定义一个计算器类
 **/
public class Calculator {

    @Check
    public void add() {
        System.out.println("1+0=" + (1 + 0));
    }
    @Check
    public void sub() {
        System.out.println("1-0=" + (1 - 0));
    }
    @Check
    public void mul() {
        System.out.println("1*0=" + (1 * 0));
    }
    @Check
    public void div() {
        System.out.println("1/0=" + (1 / 0));
    }

    public void show() {
        System.out.println("永无bug");
    }
}
```

