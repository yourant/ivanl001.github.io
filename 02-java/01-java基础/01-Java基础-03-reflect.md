[toc]

<extoc></extoc>



## 1, 反射的基本使用

```java
package org.bool.a02_reflect;

import java.lang.reflect.Constructor;
import java.lang.reflect.Field;
import java.lang.reflect.Method;
import java.lang.reflect.Modifier;

/**
 * @author : 不二
 * @date : 2021/4/26-下午5:50
 * @desc : 反射演示
 **/
public class Reflect {
    public static void main(String[] args) throws Exception {

        Class dogClass = Dog.class;
        System.out.println(dogClass);


        // -----------------------------成员变量-----------------------------------
        // 获取所有变量(只有公开的能被访问)
        Field[] dogFields = dogClass.getFields();
        printFields(dogFields);

        // 获取自身成员变量
        Field[] dogDeclaredFields = dogClass.getDeclaredFields();
        printFields(dogDeclaredFields);

        // 获取指定好的成员变量
        Field dogNameField = dogClass.getDeclaredField("name");
        printFields(new Field[]{dogNameField});


        // -----------------------------成员方法-----------------------------------
        //获取所有方法
        Method[] dogMethods = dogClass.getMethods();
        printMethods(dogMethods);

        //获取自身方法，也就是不包括父类方法
        Method[] dogDeclaredMethods = dogClass.getDeclaredMethods();
        printMethods(dogDeclaredMethods);

        //获取某个不带参数的方法
        Method dogRunMethod = dogClass.getDeclaredMethod("run", null);
        printMethods(new Method[]{dogRunMethod});


        //获取某个带有参数的方法
        Method dogRunWithParamMethod = dogClass.getDeclaredMethod("run", String.class);
        printMethods(new Method[]{dogRunWithParamMethod});


        //⚠️：这里默认可以通过new方法创建对象，如果不能的话，参考后面通过反射构建对象的方式，也就是通过反射获取构造器进行构建
        //无参方法调用
        dogRunMethod.invoke(new Dog());

        //有参数方法调用
        dogRunWithParamMethod.invoke(new Dog(), "ivanl001");


        // -----------------------------整体实现-----------------------------------
        System.out.println("\n");
        System.out.println("开始调用成员属性");
        Dog dog = new Dog();
        dog.setName("Awe Dog");
        // 这里可以通过对象拿到
        // Class<? extends Dog> dogClass01 = dog.getClass();
        // 也可以通过类名拿到
        Class<?> dogClass01 = Class.forName("org.bool.a02_reflect.Dog");
        // name这里是private修饰，所以需要修改才能获取到
        Field dog01NameField = dogClass01.getDeclaredField("name");
        dog01NameField.setAccessible(true);
        // 这里就拿到成员变量name了。
        String dog01Name = (String) dog01NameField.get(dog);
        System.out.println(dog01Name);


        // -----------------------------构造方法-----------------------------------
        // 通过无参构造创建实例
        //--------------------通过反射的newInstance，适用于有公开的空参构造器的类-------------------
        //1.1，创建一个实例对象,newInstance是通过公开的空参构造器进行构造的，如果空参构造器没有公开，请用第二种方法
        Object theDogInstanceObj = dogClass.newInstance();
        dogRunMethod.invoke(theDogInstanceObj);

        // 通过带参数构造器创建实例
        Constructor nameConstructor = dogClass.getConstructor(String.class);
        Object theDogInstanceObj01 = nameConstructor.newInstance("ivanl001");
        dogRunMethod.invoke(theDogInstanceObj01);

        //----------------------通过反射的构造器进行，不公开的构造器也可以-------------------
        Constructor privateConstructor = dogClass.getDeclaredConstructor(String.class, String.class);
        // 如果不添加允许权限，则创建的时候会报错
        privateConstructor.setAccessible(true);
        Object theDogInstanceObj02 = privateConstructor.newInstance("ivanl001", "good dog");
        dogRunMethod.invoke(theDogInstanceObj02);


        //--------------------------------反射获取修饰符----------------------------------
        int modifiers = dogRunMethod.getModifiers();
        System.out.println("是否接口:" + Modifier.isInterface(modifiers));
        System.out.println("是否public:" + Modifier.isPublic(modifiers));
        System.out.println("是否isFinal:" + Modifier.isFinal(modifiers));

    }

    public static void printMethods(Method[] methods) {
        System.out.println("method打印开始");
        for (Method method : methods) {
            System.out.println("method是：" + method);
        }
        System.out.println("method打印完成");
        System.out.println("\n");
    }


    public static void printFields(Field[] fields) {
        System.out.println("field打印开始");
        for (Field field : fields) {
            System.out.println("field是：" + field);
        }
        System.out.println("field打印完成");
        System.out.println("\n");
    }
}

```



## 2, 通过反射实现关闭缓存区的案例

```java
package org.bool.a02_reflect;

import java.lang.reflect.Field;
import java.lang.reflect.Method;
import java.nio.ByteBuffer;

/**
 * @author : 不二
 * @date : 2021/4/26-下午7:46
 * @desc : 这是一个通过反射实现关闭缓存区的案例
 **/
public class Reflect01 {
    public static void main(String[] args) throws Exception {

        //0, 创建一个500M的离堆缓存
        ByteBuffer byteBuffer = ByteBuffer.allocateDirect(500 * 1024 * 1024);
        ByteBuffer byteBuffer01 = ByteBuffer.allocateDirect(100 * 1024 * 1024);

        //----------------------------通过反射方式释放上面的500M内存----------------------------------
        //1，获得DirectByteBuffer类，因为上面ByteBuffer.allocateDirect创建的其实就是DirectByteBuffer类类
        Class clazz = Class.forName("java.nio.DirectByteBuffer");

        //2，根据字段描述符查找指定的字段
        Field field = clazz.getDeclaredField("cleaner");
        //因为是私有的，所以需要设置一下权限让可以访问
        field.setAccessible(true);

        //3, 拿到字段之后，取得field在byteBuffer上的对象，其实也就是Cleaner在DirectByteBuffer的实例
        Object cleaner = field.get(byteBuffer);//这里拿到对象了
        System.out.println(cleaner);

        //4, 再同样的拿到方法
        Class clazz01 = Class.forName("sun.misc.Cleaner");
        Method method = clazz01.getDeclaredMethod("clean");

        method.invoke(cleaner);//这里是通过方法调对象
    }
}
```





## 3, 反射操纵对象变量及方法

* 通过拿到对象，可以通过反射实现获取对象字段和调用对象方法作用

```java
package org.bool.a02_reflect;

import java.lang.reflect.Field;
import java.lang.reflect.Method;

/**
 * @author : 不二
 * @date : 2021/4/26-下午8:02
 * @desc : 这里通过拿到对象，可以通过反射实现获取对象字段和调用对象方法作用
 **/
public class Reflect02 {

    public static void main(String[] args) throws Exception {

        // 假设说有一个Dog已经建立，内部已经设置好Cat。那么现在通过反射的方式拿到Dog的Cat，并调用Cat的run方法
        Dog dog = new Dog();
        dog.setName("doggy");
        dog.setFriend(new Cat("miaomiao"));

        // 通过反射，获取Dog类，并获取friend方法
        Class<?> dogClass = Class.forName("org.bool.a02_reflect.Dog");
        Field friendField = dogClass.getDeclaredField("friend");
        friendField.setAccessible(true);

        // 通过调用friend方法， 这里就拿到Cat对象了
        Object catObj = friendField.get(dog);

        // 这里获取到Cat对象
        Class<?> catClass = Class.forName("org.bool.a02_reflect.Cat");
        Field nameField = catClass.getDeclaredField("name");
        nameField.setAccessible(true);
        Object nameObj = nameField.get(catObj);
        System.out.println("获取的名字为: " + nameObj.toString());

        Method runMethod = catClass.getDeclaredMethod("run");
        System.out.println("获取的方法为: " + runMethod.getName());

        runMethod.setAccessible(true);
        runMethod.invoke(catObj);

    }
}
```



## 4, 反射获取修饰符

> 参考1中

```java
@Test
public void testModifier() throws NoSuchMethodException {
  
  Class dogClazz = Dog.class;
  Method run = dogClazz.getDeclaredMethod("run", null);
	
  //说明：这个int值其实也是位运算来枚举的一个很好的例子哦
  int modifiers = run.getModifiers();
  System.out.println(Modifier.isInterface(modifiers));
}
```





## 基础类型准备

### Dog.java

```java
package org.bool.a02_reflect;

/**
 * @author : 不二
 * @date : 2021/4/26-下午5:48
 * @desc : 用于反射演示的一个类
 **/
public class Dog {

    // 公开的方法可以被访问
    public String desc;
    private String name;

    private Cat friend;

//    Dog(String name) {
//        this.name = name;
//    }

    // 用到反射的时候，空参必须要公开
    public Dog(){
    }

    private String saySomething() {
        return "hey";
    }

    public void run(){
        System.out.println("dog start to run!!!!!!!");
    }
    public void run(String name){
        System.out.println("dog start to run!!!!!!!" + name);
    }
    public String getName() {
        return name;
    }
    public void setName(String name) {
        this.name = name;
    }
    public void toPrintSomething() {
        System.out.println("我是dog内部的打印");
    }

    public Cat getFriend() {
        return friend;
    }

    public void setFriend(Cat friend) {
        this.friend = friend;
    }
}
```

### Cat.java

```java
package org.bool.a02_reflect;

/**
 * @author : 不二
 * @date : 2021/4/26-下午7:41
 * @desc :
 **/
public class Cat {

    private String name;

    public Cat(String name) {
        this.name = name;
    }

    private void run(){
        System.out.println("cat is running: ------" + name);
    }

}
```

