[toc]



## 单元测试

### 01, 单元测试的基本使用

 ```java
// 引入类
import org.junit.Assert;
import org.junit.Test;


@Test
public void testSum() {
  int a = 10;
  int b = 5;
  int result = Util.sum(a, b);

  // 第一个参数为期望的结果，第二个为实现后的结果，如果和期望相同，则代码绿色通过。如果不相同，则会报错
  Assert.assertEquals(10,  Util.sum(a, b));
}
 ```

### 02, @Before和@After注解

```java
package org.bool.f_junit;

import org.junit.After;
import org.junit.Assert;
import org.junit.Before;
import org.junit.Test;

public class UtilTest {

    @Before
    public void before() {
        System.out.println("代码运行前");
    }

    @After
    public void after() {
        System.out.println("代码运行end");
    }

    @Test
    public void testSum() {
        int a = 10;
        int b = 5;
        int result = Util.sum(a, b);

        System.out.println("代码运行中");

        // 第一个参数为期望的结果，第二个为实现后的结果，如果和期望相同，则代码绿色通过。如果不相同，则会报错
        Assert.assertEquals(15,  Util.sum(a, b));
    }

}
```

