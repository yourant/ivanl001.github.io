[toc]

<extoc></extoc>







#### [2. 两数相加](https://leetcode-cn.com/problems/add-two-numbers/)

> 给你两个 非空 的链表，表示两个非负的整数。它们每位数字都是按照 逆序 的方式存储的，并且每个节点只能存储 一位 数字。
>
> 请你将两个数相加，并以相同形式返回一个表示和的链表。
>
> 你可以假设除了数字 0 之外，这两个数都不会以 0 开头。

##### 我的答案

`暂时还有问题，还需要调整`

```java
/**
     * 这里还有bug，输入两个5，会有问题
     * @param l1
     * @param l2
     * @return
     */
    // TODO: 这里还有问题：输入两个5，会有问题
    public ListNode addTwoNumbers_v1(ListNode l1, ListNode l2) {

        // 先判断长度
        int l1Length = 1;
        ListNode current1 = l1;
        while (current1.next != null) {
            l1Length += 1;
            current1 = current1.next;
        }
        System.out.println("l1的长度是:" + l1Length);

        int l2Length = 1;
        ListNode current2 = l2;
        while (current2.next != null) {
            l2Length += 1;
            current2 = current2.next;
        }
        System.out.println("l2的长度是:" + l2Length);


        boolean tenPlus = false;
        ListNode current ;
        if (l1Length >= l2Length) {
            current = l1;
            int tenPlusNum = 0;

            int i=0;
            while (current.next != null) {
                System.out.println(i++);

                if (tenPlus) {
                    tenPlusNum = 1;
                }
                tenPlus = (current.val + l2.val + tenPlusNum) >= 10;
                current.val = (current.val + l2.val + tenPlusNum)%10 ;

                current = current.next;
                l2 = l2.next == null ? new ListNode(0, null) : l2.next;
            }


            if (tenPlus) {
                tenPlusNum = 1;
                if (current.val + l2.val + tenPlusNum >= 10) {
                    current.next = new ListNode(1, null);
                }
                current.val = (current.val + l2.val + tenPlusNum)%10;
            } else {
                current.val = current.val + l2.val;
            }

            return l1;


        } else {

            current = l2;
            int tenPlusNum = 0;

            int i=0;
            while (current.next != null) {
                System.out.println(i++);

                if (tenPlus) {
                    tenPlusNum = 1;
                }
                tenPlus = (current.val + l1.val + tenPlusNum) >= 10;
                current.val = (current.val + l1.val + tenPlusNum)%10 ;

                current = current.next;
                l1 = l1.next == null ? new ListNode(0, null) : l1.next;
            }

            if (tenPlus) {
                tenPlusNum = 1;

                if (current.val + l1.val + tenPlusNum >= 10) {
                    current.next = new ListNode(1, null);
                }

                current.val = (current.val + l1.val + tenPlusNum)%10;

            } else {
                current.val = current.val + l1.val;
            }

            return l2;
        }
    }
```

##### 官方答案

```java
public ListNode addTwoNumbers_v2(ListNode l1, ListNode l2) {

        ListNode head = null, tail = null;
        // 进位为0
        int carry = 0;

        while (l1 != null || l2 != null) {
            int n1 = l1 == null ? 0 : l1.val;
            int n2 = l2 == null ? 0 : l2.val;

            int sum = n1 + n2 + carry;

            if (head == null) {
                head = tail = new ListNode(sum%10);
            } else {
                tail.next = new ListNode(sum%10);
                tail = tail.next;
            }

            carry = sum/10;

            if (l1 != null) {
                l1 = l1.next;
            }
            if (l2 != null) {
                l2 = l2.next;
            }
        }

        // carry最大应该只能为1吧
        if (carry > 0) {
            tail.next = new ListNode(carry);
        }

        return head;
    }
```



##### 更好的答案

```java
/**
     * https://leetcode-cn.com/problems/add-two-numbers/solution/ji-bai-100di-gui-jie-fa-shuai-zha-by-tan-vzaj/
     * @param l1
     * @param l2
     * @return
     * 这个算法确实非常巧妙， 把整体加和分散到单个位数加和。通过迭代实现单个位数加和
     */
    public ListNode addTwoNumbers_v3(ListNode l1, ListNode l2) {
        if (l1 == null) {
            return l2;
        }
        if (l2 == null) {
            return l1;
        }

        // 直接修改l2，最后返回l2
        l2.val = l1.val + l2.val;

        if (l2.val >= 10) {
            l2.val = l2.val % 10;
            if (l2.next != null) {
                l2.next.val = l2.next.val + 1;
                if (l2.next.val == 10) {
                    l2.next = addTwoNumbers_v3(new ListNode(0, null), l2.next);
                }
            } else {
                l2.next = new ListNode(1, null);
            }
        }
        l2.next = addTwoNumbers_v3(l1.next, l2.next);
        return l2;
    }
```











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

##### 官方答案

> 注意到方法一的时间复杂度较高的原因是寻找 target - x 的时间复杂度过高。因此，我们需要一种更优秀的方法，能够快速寻找数组中是否存在目标元素。如果存在，我们需要找出它的索引。
>
> 使用哈希表，可以将寻找 target - x 的时间复杂度降低到从 O(N)O(N) 降低到 O(1)O(1)
>
> 作者：LeetCode-Solution
> 链接：https://leetcode-cn.com/problems/two-sum/solution/liang-shu-zhi-he-by-leetcode-solution/
> 来源：力扣（LeetCode）
> 著作权归作者所有。商业转载请联系作者获得授权，非商业转载请注明出处。

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