[toc]

<extoc></extoc>











#### [1. 两数之和](https://leetcode-cn.com/problems/two-sum/)

> 给定一个整数数组 nums 和一个整数目标值 target，请你在该数组中找出 和为目标值 的那 两个 整数，并返回它们的数组下标。
>
> 你可以假设每种输入只会对应一个答案。但是，数组中同一个元素在答案里不能重复出现。
>
> 你可以按任意顺序返回答案。

##### 我的答案

```java
class Solution {
    public static int[] twoSum(int[] nums, int target) {
        int diff;
        int[] arr = new int[2];
        for (int i=0;i<nums.length;i++) {
            diff = target - nums[i];
            for (int j=i+1;j<nums.length;j++) {
                if(diff == nums[j]) {
                    arr[0] = i;
                    arr[1] = j;
                    return arr;
                }
            }
        }
        return arr;
    }
}
```

##### 更好的答案

```java
public static int[] twoSum001(int[] nums, int target) {
    Map<Integer, Integer> hashtable = new HashMap<>();
    int diff;
    for (int i = 0; i < nums.length; i++) {
        diff = target - nums[i];
        if (hashtable.containsKey(diff)) {
            return new int[]{hashtable.get(diff), i};
        }
        hashtable.put(nums[i], i);
    }
    return new int[0];
}
```

##### 值得思考的问题：

> 对于官方第二种采用哈希表的方法，其效率应该是低于采用双循环数组的方式，因为数组遍历的效率大于HashMap，做一个测试：
>
> ```java
> public void findOne(int key) {
>   int size = 1000 * 10000;
>   int[] arr = new int[size];
>   Map<Integer, Integer> map = new HashMap<>(size);
>   for (int i=0;i<size;i++) {
>     arr[i] = i;
>     map.put(i, i);
>   }
>   long l1 = System.currentTimeMillis();
>   for (int i=0;i<arr.length;i++) {
>     if (key == arr[i]) {
>       System.out.println("遍历数组长度"+size+"所需时长:" + (System.currentTimeMillis() - l1));
>       break;
>     }
>   }
>   long l2 = System.currentTimeMillis();
>   if (map.containsValue(key)) {
>     System.out.println("遍历HashMap长度"+size+"所需时长:" + (System.currentTimeMillis() - l2));
>   }
> }
> ```



##### 思考：

用了containsValue， 而不是containKey

如果查找的数字在数组的最前面， 那确实是数组快点，如果是在后面，还是hashmap快一点