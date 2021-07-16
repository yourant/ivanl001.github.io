[toc]

> 参考：
>
> https://blog.csdn.net/qq_27721169/article/details/88029436



## 1, 反射工具类

```java
package com.example.job_scheduler.utils;

import java.lang.reflect.Field;
import java.lang.reflect.Method;
import java.lang.reflect.Modifier;
import java.util.*;

/**
 * #Author : ivanl001
 * #Date   : 2019/11/9 11:16
 * #Desc   : TODO
 **/
public class BeanUtils {

    public static Field findField(Class<?> clazz, String name) {
        try {
            return clazz.getField(name);
        } catch (NoSuchFieldException ex) {
            return findDeclaredField(clazz, name);
        }
    }

    public static Field findDeclaredField(Class<?> clazz, String name) {
        try {
            return clazz.getDeclaredField(name);
        } catch (NoSuchFieldException ex) {
            if (clazz.getSuperclass() != null) {
                return findDeclaredField(clazz.getSuperclass(), name);
            }
            return null;
        }
    }

    public static Method findMethod(Class<?> clazz, String methodName, Class<?>... paramTypes) {
        try {
            return clazz.getMethod(methodName, paramTypes);
        } catch (NoSuchMethodException ex) {
            return findDeclaredMethod(clazz, methodName, paramTypes);
        }
    }

    public static Method findDeclaredMethod(Class<?> clazz, String methodName, Class<?>... paramTypes) {
        try {
            return clazz.getDeclaredMethod(methodName, paramTypes);
        } catch (NoSuchMethodException ex) {
            if (clazz.getSuperclass() != null) {
                return findDeclaredMethod(clazz.getSuperclass(), methodName, paramTypes);
            }
            return null;
        }
    }

    public static Object getProperty(Object obj, String name) throws NoSuchFieldException {
        Object value = null;
        Field field = findField(obj.getClass(), name);
        if (field == null) {
            throw new NoSuchFieldException("no such field [" + name + "]");
        }
        boolean accessible = field.isAccessible();
        field.setAccessible(true);
        try {
            value = field.get(obj);
        } catch (Exception e) {
            throw new RuntimeException(e);
        }
        field.setAccessible(accessible);
        return value;
    }

    public static void setProperty(Object obj, String name, Object value) throws NoSuchFieldException {
        Field field = findField(obj.getClass(), name);
        if (field == null) {
            throw new NoSuchFieldException("no such field [" + name + "]");
        }
        boolean accessible = field.isAccessible();
        field.setAccessible(true);
        try {
            field.set(obj, value);
        } catch (Exception e) {
            throw new RuntimeException(e);
        }
        field.setAccessible(accessible);
    }

    public static Map<String, Object> obj2Map(Object obj, Map<String, Object> map) {
        if (map == null) {
            map = new HashMap<>();
        }
        if (obj != null) {
            try {
                Class<?> clazz = obj.getClass();
                do {
                    Field[] fields = clazz.getDeclaredFields();
                    for (Field field : fields) {
                        int mod = field.getModifiers();
                        if (Modifier.isStatic(mod)) {
                            continue;
                        }
                        boolean accessible = field.isAccessible();
                        field.setAccessible(true);
                        map.put(field.getName(), field.get(obj));
                        field.setAccessible(accessible);
                    }
                    clazz = clazz.getSuperclass();
                } while (clazz != null);
            } catch (Exception e) {
                throw new RuntimeException(e);
            }
        }
        return map;
    }

    /**
     * 获得父类集合，包含当前class
     *
     * @param clazz
     * @return
     */
    public static List<Class<?>> getSuperclassList(Class<?> clazz) {
        List<Class<?>> clazzes = new ArrayList<>(3);
        clazzes.add(clazz);
        clazz = clazz.getSuperclass();
        while (clazz != null) {
            clazzes.add(clazz);
            clazz = clazz.getSuperclass();
        }
        return Collections.unmodifiableList(clazzes);
    }


}

```





## 2, SchedulingConfigurer实现类

```java
package com.example.job_scheduler.job;

import com.example.job_scheduler.utils.BeanUtils;
import org.springframework.context.annotation.Configuration;
import org.springframework.scheduling.SchedulingException;
import org.springframework.scheduling.TaskScheduler;
import org.springframework.scheduling.annotation.EnableScheduling;
import org.springframework.scheduling.annotation.SchedulingConfigurer;
import org.springframework.scheduling.config.ScheduledTaskRegistrar;
import org.springframework.scheduling.config.TriggerTask;

import java.util.Map;
import java.util.Set;
import java.util.concurrent.ConcurrentHashMap;
import java.util.concurrent.ScheduledFuture;

/**
 * #Author : ivanl001
 * #Date   : 2019/11/9 11:18
 * #Desc   : TODO
 **/
@Configuration
@EnableScheduling
public class DefaultSchedulingConfigurer implements SchedulingConfigurer {

    private ScheduledTaskRegistrar taskRegister;
    private Set<ScheduledFuture<?>> scheduledFutures = null;
    private Map<String, ScheduledFuture<?>> taskFutures = new ConcurrentHashMap<>();


    @Override
    public void configureTasks(ScheduledTaskRegistrar scheduledTaskRegistrar) {
        this.taskRegister = scheduledTaskRegistrar;
    }



    private Set<ScheduledFuture<?>> getScheduledFutures() {
        if (scheduledFutures == null) {
            try {
                // spring版本不同选用不同字段scheduledFutures
                scheduledFutures = (Set<ScheduledFuture<?>>) BeanUtils.getProperty(taskRegister, "scheduledTasks");
            } catch (NoSuchFieldException e) {
                throw new SchedulingException("not found scheduledFutures field.");
            }
        }
        return scheduledFutures;
    }


    /**
     * 添加任务
     */
    public void addTriggerTask(String taskId, TriggerTask triggerTask) {
        if (taskFutures.containsKey(taskId)) {
            throw new SchedulingException("the taskId[" + taskId + "] was added.");
        }
        TaskScheduler scheduler = taskRegister.getScheduler();
        ScheduledFuture<?> future = scheduler.schedule(triggerTask.getRunnable(), triggerTask.getTrigger());
        getScheduledFutures().add(future);
        taskFutures.put(taskId, future);
    }

    /**
     * 取消任务
     */
    public void cancelTriggerTask(String taskId) {
        ScheduledFuture<?> future = taskFutures.get(taskId);
        if (future != null) {
            future.cancel(true);
        }
        taskFutures.remove(taskId);
        getScheduledFutures().remove(future);
    }

    /**
     * 重置任务
     */
    public void resetTriggerTask(String taskId, TriggerTask triggerTask) {
        cancelTriggerTask(taskId);
        addTriggerTask(taskId, triggerTask);
    }


    /**
     * 任务编号
     */
    public Set<String> taskIds() {
        return taskFutures.keySet();
    }

    /**
     * 是否存在任务
     */
    public boolean hasTask(String taskId) {
        return this.taskFutures.containsKey(taskId);
    }

    /**
     * 任务调度是否已经初始化完成
     */
    public boolean initialized() {

        System.out.println(taskRegister);
        /*System.out.println(taskRegistrar.getScheduler());*/

        return this.taskRegister != null && this.taskRegister.getScheduler() != null;
    }

}
```



## 3, 测试

```java
package com.example.job_scheduler.controller;

import com.example.job_scheduler.job.DefaultSchedulingConfigurer;
import org.springframework.scheduling.config.TriggerTask;
import org.springframework.scheduling.support.CronTrigger;
import org.springframework.web.bind.annotation.GetMapping;
import org.springframework.web.bind.annotation.RequestMapping;
import org.springframework.web.bind.annotation.RestController;

import javax.annotation.Resource;
import java.util.Date;

/**
 * #Author : ivanl001
 * #Date   : 2019/11/9 11:42
 * #Desc   : TODO
 **/
@RestController
public class IMSchedulingTestController {

    @Resource
    private DefaultSchedulingConfigurer defaultSchedulingConfigurer;

    @RequestMapping("/test")
    public String testRequest(){
        return "OK";
    }

    @GetMapping("add")
    public void addTask() throws InterruptedException {

        System.out.println("开始添加task");

        //如果没有初始化，先进行初始化
        while (!defaultSchedulingConfigurer.initialized())
        {
            Thread.sleep(100);
        }

        defaultSchedulingConfigurer.addTriggerTask("task",
                new TriggerTask(
                        () -> System.out.println("task01" + new Date()),
                        new CronTrigger("0/5 * * * * ? ")));
    }

    @GetMapping("cancel")
    public void cancelTask(String taskId){
        System.out.println("开始取消task");

        if (defaultSchedulingConfigurer.hasTask(taskId)) {
            defaultSchedulingConfigurer.cancelTriggerTask(taskId);
        } else {
            System.out.println("没有该taskid");
        }
    }

}
```

