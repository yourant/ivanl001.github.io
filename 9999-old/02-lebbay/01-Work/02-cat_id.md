[TOC]

```shell
WD: 2
sod: 3 
Wedding Veils: 6   
BD: 7 
MOB: 8
JBD: 9  
Flower Girl Dresses: 10 
Wraps: 13  
Sashes: 15   
swatch: 33
fabric: 36
MaternityBD: 45  
Groomsmen Accessories: 138
separates： 139   
Modest BD: 158  
sample/clearance : 206
```





## 1， 商品类别划分

订单表：order_info

订单商品表： order_goods

事实商品表： goods

order ----> order_goods ----> goods(cat_id)

| 类别                                         | cat_id(在goods表中) |
| -------------------------------------------- | ------------------- |
| WD (Wedding Dresses, 新娘装)                 | 2                   |
| SOD (Special Occasions, )                    | 3                   |
| Wedding Veils(面纱)                          | 6                   |
| BD (Bridemaid Dresses, 伴娘裙)               | 7                   |
| MOB (Mother of the Bride, 妈妈装)            | 8                   |
| JBD (Junior Bridesmaid Dresses, 少女装)      | 9                   |
| Flower Girl Dresses（花童装）                | 10                  |
| Wraps（披肩）                                | 13                  |
| Sashes （腰带）                              | 15                  |
| swatch （色卡, color swatch）                | 33                  |
| fabric（布料）                               | 36                  |
| MaternityBD （孕妇婚纱）                     | 45                  |
| Groomsmen Accessories （新郎配饰）           | 138                 |
| separates（单裙）                            | 139                 |
| Modest BD （常规婚纱）                       | 158                 |
| sample/clearance (样衣/清仓)                 | 206                 |
| ---> 取这两个可以不用cat_id                  |                     |
| ---> 直接用order_good表中的goods_id = 101199 |                     |
| clearance现在也叫ready to ship               |                     |

*　裙子： 2 3 7 8 9 10  45 139 158
*　 正式裙子：wd,bd,mob,jbd     2,7,8,9





### 注意01：区分sample和clearance：

* 1，先通过goods_id = 101199找出sample和clearance(也可以通过goods表的cat_id=206来判断)
* 2，然后根据order_goods表中的styles字段来判断，如果isClearance = 1  则是清仓商品，否则是样衣



### 注意02：BD 是bridemaid dresses

* 伴娘装， 在网站中没有单独列出来

