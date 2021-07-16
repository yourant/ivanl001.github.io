[TOC]

# java-RestAPI操纵elasticSearch

## 1, 使用high rest api



### 01, maven依赖

```xml
<!-- 注意：这里使用高级api的时候需要引入elasticsearch，因为下面的spring-boot-starter-data-elasticsearch使用了较低版本的es，这里需要使用高版本覆盖一下 -->
<dependency>
  <groupId>org.elasticsearch.client</groupId>
  <artifactId>elasticsearch-rest-high-level-client</artifactId>
  <version>6.8.3</version>
</dependency>
<!-- https://mvnrepository.com/artifact/org.elasticsearch/elasticsearch -->
<dependency>
  <groupId>org.elasticsearch</groupId>
  <artifactId>elasticsearch</artifactId>
  <version>6.8.3</version>
</dependency>
```



### 02, 直接测试

```java
package azazie.com.azazieoptimizerjava.elasticsearch;

import azazie.com.azazieoptimizerjava.app.elasticsearch.Employee;
import com.alibaba.fastjson.JSON;
import org.apache.http.HttpHost;
import org.elasticsearch.action.get.GetRequest;
import org.elasticsearch.action.get.GetResponse;
import org.elasticsearch.action.index.IndexRequest;
import org.elasticsearch.action.index.IndexResponse;
import org.elasticsearch.action.update.UpdateRequest;
import org.elasticsearch.action.update.UpdateResponse;
import org.elasticsearch.client.RequestOptions;
import org.elasticsearch.client.RestClient;
import org.elasticsearch.client.RestHighLevelClient;
import org.elasticsearch.common.xcontent.XContentType;
import org.elasticsearch.search.fetch.subphase.FetchSourceContext;
import org.junit.Test;

import java.io.IOException;

/**
 * #Author : ivanl001
 * #Date   : 2019/10/7 16:19
 * #Desc   : TODO
 **/
public class IMHighLevelRestAPI {

    @Test
    public void test() throws IOException {

        RestHighLevelClient client = new RestHighLevelClient(
                RestClient.builder(
                        new HttpHost("centos01", 9200, "http"),
                        new HttpHost("centos02", 9200, "http"),
                        new HttpHost("centos03", 9200, "http")));

        String index = "company";
        String type = "employee";
        String id = "2";

        Employee employee = new Employee();
        employee.setId("2");
        employee.setFirstName("ivanl003");
        employee.setLastName("Bool");
        employee.setAge(3333);
        employee.setAbout("33333333 is the king of world!");
        String msg = JSON.toJSONString(employee);

      	//indexRequest可以插入，如果存在就是更新
        IndexRequest indexRequest = new IndexRequest(index, type, "3").source(msg, XContentType.JSON);
        IndexResponse indexResponse = client.index(indexRequest, RequestOptions.DEFAULT);

        /*System.out.println(msg);
        UpdateRequest request = new UpdateRequest(index, type, id);
        request.doc(msg, XContentType.JSON);
        client.update(request, RequestOptions.DEFAULT);*/

        /*GetRequest getRequest = new GetRequest(
                "company",
                "employee",
                "2");
        GetResponse getResponse = client.get(getRequest, RequestOptions.DEFAULT);
        String sourceAsString = getResponse.getSourceAsString();

        System.out.println("--------------------------------------------------------------ivanl001--------------------------------------------------------------");
        System.out.println(sourceAsString);*/

        client.close();
    }
}
```



