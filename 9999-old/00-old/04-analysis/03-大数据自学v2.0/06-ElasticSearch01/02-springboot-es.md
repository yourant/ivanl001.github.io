[TOC]

# SpringBoot 集成Elasticsearch

## 1, 使用jpa方式



### 01, maven依赖

```xml
<!-- spring-boot elasticsearch -->
<dependency>
  <groupId>org.springframework.boot</groupId>
  <artifactId>spring-boot-starter-data-elasticsearch</artifactId>
</dependency>
```



### 02, 相关配置

```properties
# -------------------------------------设置elasticSearch相关配置-------------------------------------
# elasticsearch集群名称，默认的是elasticsearch
spring.data.elasticsearch.cluster-name=centos-es
#节点的地址 注意api模式下端口号是9300，千万不要写成9200
spring.data.elasticsearch.cluster-nodes=centos01:9300
#是否开启本地存储
#spring.data.elasticsearch.repositories.enable=true
```



### 03, 根据es中type创建实体类

```java
package azazie.com.azazieoptimizerjava.app.elasticsearch;

import lombok.Getter;
import lombok.Setter;
import lombok.ToString;
import org.springframework.data.annotation.Id;
import org.springframework.data.elasticsearch.annotations.Document;
import org.springframework.data.elasticsearch.annotations.Field;

/**
 * #Author : ivanl001
 * #Date   : 2019/10/7 14:56
 * #Desc   : 
 **/
//这里是getter和setter是lombok中的注解
@Getter
@Setter
@ToString
@Document(indexName = "company",type = "employee", shards = 1,replicas = 0, refreshInterval = "-1")
public class Employee {
    @Id
    private String id;
    @Field
    private String firstName;
    @Field
    private String lastName;
    @Field
    private Integer age = 0;
    @Field
    private String about;
}
```



### 04, dao类EmployeeRepository

```java
package azazie.com.azazieoptimizerjava.dao;

import azazie.com.azazieoptimizerjava.app.elasticsearch.Employee;
import org.springframework.data.elasticsearch.repository.ElasticsearchRepository;

/**
 * #Author : ivanl001
 * #Date   : 2019/10/7 14:58
 * #Desc   : TODO
 * ElasticsearchRepository --> ElasticsearchCrudRepository --> PagingAndSortingRepository --> CrudRepository
 **/
public interface EmployeeRepository extends ElasticsearchRepository<Employee, String> {

    /**
     * 查询雇员信息
     */
    Employee queryEmployeeById(String id);

    //这里其实内部是使用代理，根据方法中的字段进行自行实现的
    Employee findByFirstName(String firstName);
}

```



### 05, 进行测试

```java
package azazie.com.azazieoptimizerjava.dao;

import azazie.com.azazieoptimizerjava.app.elasticsearch.Employee;
import azazie.com.azazieoptimizerjava.app.sql.IMJdbcTemplateTest;
import org.junit.Test;
import org.junit.runner.RunWith;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.boot.test.context.SpringBootTest;
import org.springframework.test.context.ActiveProfiles;
import org.springframework.test.context.junit4.SpringRunner;

import static org.junit.Assert.*;


@RunWith(SpringRunner.class)
@SpringBootTest
@ActiveProfiles("dev")
public class EmployeeRepositoryTest {

    @Autowired
    private EmployeeRepository employeeRepository;


    @Test
    public void queryquery(){
        Employee byFirstName = employeeRepository.findByFirstName("dfaksladsfladskfjdajjjjjjjjjjjj");
        System.out.println(byFirstName);
    }

    @Test
    public void save() {
        Employee employee = new Employee();
        employee.setId("1");
        employee.setFirstName("dfaksladsfladskfjdajjjjjjjjjjjj");
        employee.setLastName("zh");
        employee.setAge(26);
        employee.setAbout("i am in peking");
        employeeRepository.save(employee);
        System.err.println("add a obj");
    }


    /**
     * 删除
     * @return
     */
    @Test
    public void delete() {
        Employee employee = employeeRepository.queryEmployeeById("1");
        employeeRepository.delete(employee);
    }

    /**
     * 局部更新
     * @return
     */
    @Test
    public void update() {
        Employee employee = employeeRepository.queryEmployeeById("1");
        employee.setFirstName("哈哈");
        employeeRepository.save(employee);
        System.err.println("update a obj");

    }
    /**
     * 查询
     * @return
     */
    @Test
    public void query() {
        Employee accountInfo = employeeRepository.queryEmployeeById("1");
//        System.err.println(new Gson().toJson(accountInfo));
//        return accountInfo;
    }
}
```