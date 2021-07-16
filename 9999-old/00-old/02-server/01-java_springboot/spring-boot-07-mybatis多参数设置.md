本文链接：https://blog.csdn.net/lixld/article/details/77980443



## 一、单个参数：

```xml
public List<XXBean> getXXBeanList(@param("id")String id);  

<select id="getXXXBeanList" parameterType="java.lang.String" resultType="XXBean">
　　select t.* from tableName t where t.id= #{id}  
</select>  
```



其中方法名和ID一致，#{}中的参数名与方法中的参数名一致， 这里采用的是@Param这个参数，实际上@Param这个最后会被Mabatis封装为map类型的。

select 后的字段列表要和bean中的属性名一致， 如果不一致的可以用 as 来补充。



## 二、多参数：(这个方案满足我需求)

### 方案1

```xml
public List<XXXBean> getXXXBeanList(String xxId, String xxCode);  

<select id="getXXXBeanList" resultType="XXBean">不需要写parameterType参数

　　select t.* from tableName where id = #{0} and name = #{1}  

</select>  

```
由于是多参数那么就不能使用parameterType， 改用#｛index｝是第几个就用第几个的索引，索引从0开始

### 方案2（推荐）基于注解

```xml
public List<XXXBean> getXXXBeanList(@Param("id")String id, @Param("code")String code);  

<select id="getXXXBeanList" resultType="XXBean">

　　select t.* from tableName where id = #{id} and name = #{code}  

</select>  
```

由于是多参数那么就不能使用parameterType， 这里用@Param来指定哪一个



## 三、Map封装多参数：  

```xml
public List<XXXBean> getXXXBeanList(HashMap map);  

<select id="getXXXBeanList" parameterType="hashmap" resultType="XXBean">

　　select 字段... from XXX where id=#{xxId} code = #{xxCode}  

</select>  
```

其中hashmap是mybatis自己配置好的直接使用就行。map中key的名字是那个就在#{}使用那个，map如何封装就不用了我说了吧。 



##  四、List封装in：

```xml
public List<XXXBean> getXXXBeanList(List<String> list);  

<select id="getXXXBeanList" resultType="XXBean">
　　select 字段... from XXX where id in
　　<foreach item="item" index="index" collection="list" open="(" separator="," close=")">  
　　　　#{item}  
　　</foreach>  
</select>  
```

foreach 最后的效果是select 字段... from XXX where id in ('1','2','3','4') 

## 五、selectList()只能传递一个参数，但实际所需参数既要包含String类型，又要包含List类型时的处理方法：

将参数放入Map，再取出Map中的List遍历。如下：

```xml
List<String> list_3 = new ArrayList<String>();
Map<String, Object> map2 = new HashMap<String, Object>();

list.add("1");
list.add("2");
map.put("list", list); //网址id

map.put("siteTag", "0");//网址类型
public List<SysWeb> getSysInfo(Map<String, Object> map2) {
　　return getSqlSession().selectList("sysweb.getSysInfo", map2);
}


<select id="getSysInfo" parameterType="java.util.Map" resultType="SysWeb">
　　select t.sysSiteId, t.siteName, t1.mzNum as siteTagNum, t1.mzName as siteTag, t.url, t.iconPath
  from TD_WEB_SYSSITE t
  left join TD_MZ_MZDY t1 on t1.mzNum = t.siteTag and t1.mzType = 10
  WHERE t.siteTag = #{siteTag } 
  and t.sysSiteId not in 
  <foreach collection="list" item="item" index="index" open="(" close=")" separator=",">
     #{item}
  </foreach>
 </select>
```




