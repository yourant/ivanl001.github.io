[toc]

<extoc></extoc>



#### [7. 整数反转](https://leetcode-cn.com/problems/reverse-integer/)

> 给你一个 32 位的有符号整数 x ，返回将 x 中的数字部分反转后的结果。
>
> 如果反转后整数超过 32 位的有符号整数的范围 [−231,  231 − 1] ，就返回 0。
>
> 假设环境不允许存储 64 位整数（有符号或无符号）。
>

##### 我的答案

```java
// 没想出来怎么把余数加和，看了答案
```



##### 官方答案

```java
public int reverse_v1(int x) {
        int rev = 0;
        while (x != 0) {
            int pop = x%10;
            x /= 10;
            /*if (rev > Integer.MAX_VALUE/10 || (rev == Integer.MAX_VALUE / 10 && pop > 7)) {
                return 0;
            }
            if (rev < Integer.MIN_VALUE/10 || (rev == Integer.MIN_VALUE / 10 && pop < -8)) {
                return 0;
            }*/
            if (rev>214748364 || (rev==214748364 && pop>7)) {
                return 0;
            }
            //判断是否 小于 最小32位整数
            if (rev<-214748364 || (rev==-214748364 && pop<-8)) {
                return 0;
            }
            rev = rev * 10 + pop;
        }
        System.out.println("result是：" + rev);
        return rev;
}
```





#### [5. 最长回文子串](https://leetcode-cn.com/problems/longest-palindromic-substring/)

> 给你一个字符串 `s`，找到 `s` 中最长的回文子串。

##### 我的答案

> 答案应该没错，但是因为是暴力遍历，效率低，超出了时间限制

```java
/**
 * 我的答案：
 * 暴力解法(这个方法不太好，1，直接使用字符串的方法，效率低，最好把字符串转化成数组，方便直接获取)
 * @param s
 * @return
 */
public String longestPalindrome_v0(String s) {

    int n = s.length();
    if (n == 1) {
        System.out.println("答案是:" + s);
        return s;
    }

    int rk;
    StringBuilder tmpAns;
    String answer = "";
    for (int i = 0; i < n; i++) {
        for (rk = i; rk < n; rk++) {

            tmpAns = new StringBuilder();
            for (int j = i; j <= rk; j++) {
                tmpAns.append(s.charAt(j));
            }

            String words = tmpAns.toString();
            StringBuilder reversed = new StringBuilder();
            for (int m = words.length()-1; m >= 0; m--) {
                reversed.append(words.charAt(m));
            }

            if (tmpAns.toString().equals(reversed.toString())) {
                if (answer.length() < tmpAns.length()) {
                    answer = tmpAns.toString();
                }
            }
        }
    }
    System.out.println("答案是:" + answer);

    return answer;
}
```



##### 官方答案1： 暴力解法

> 官方的暴力解法

```java
/**
 * 官方的暴力解法
 * @param s
 * @return
 */
public String longestPalindrome_v0(String s) {

    int length = s.length();
    // 小于2的时候，本身就是回文串，直接返回即可
    if (length < 2) {
        return s;
    }

    char[] chars = s.toCharArray();

    int startIndex = 0, endIndex=0;

    for (int i = 0; i < length - 1; i++) {
        for (int j = i + 1; j <= length - 1; j++) {

            // 这里是判断字符串是不是回文子串
            int left = i, right = j;
            boolean isPalindrom = true;
            while (left < right) {
                if (chars[left] == chars[right]) {
                    left++;
                    right--;
                } else {
                    isPalindrom = false;
                    break;
                }
            }

            // 说明是回文子串
            if (isPalindrom && ((j - i) > (endIndex-startIndex))) {
                startIndex = i;
                endIndex = j;
            }
        }
    }

    // substring是前包后不包的
    System.out.println("结果是：" + s.substring(startIndex, endIndex+1));

    return s.substring(startIndex, endIndex+1);
}
```





##### 官方答案2: 中心扩散法

> 从内向外

```java
// 暂时未总结，看答案
```



##### 官方答案3: 动态规划

> 从外向内

```java
/**
 * 官方的动态规划解法
 * 从外部往内部循环
 *
 * 时间复杂度:O(n^2)
 * 空间复杂度:O(n^2)
 * @param s
 * @return
 */
public String longestPalindrome_v3(String s) {

    int length = s.length();
    // 小于2的时候，本身就是回文串，直接返回即可
    if (length < 2) {
        return s;
    }

    int maxLen = 1;
    int begin = 0;
    boolean[][] dp = new boolean[length][length];

    // 先把对角线赋值为true，因为所有的单个字符串肯定都是回文
    for (int i = 0; i < length; i++) {
        dp[i][i] = true;
    }

    char[] charArray = s.toCharArray();

    // 先从最左列，然后再后边的列
    // 先计算列，再计算行
    // 状态转移
    for (int j = 0; j < length; j++) {
        for (int i = 0; i < j; i++) {
            if (charArray[i] != charArray[j]) {
                dp[i ][j] = false;
            } else {
                if (j - i < 3) {
                    dp[i][j] = true;
                } else {
                    // 这里因为是查当前dp[i][j] 对应的dp[i + 1][j - 1],也就是左下角位置。对应的j-1的值在前一轮已经算过并且赋值了的
                    // 所以这里是可以直接使用的
                    dp[i][j] = dp[i + 1][j - 1];
                }
            }

            if (dp[i][j] && j - i + 1 > maxLen) {
                // 因为字符串的subString是前包后不包的，所以需要加1
                maxLen = j - i + 1;
                begin = i;
            }
        }
    }

    return s.substring(begin, begin + maxLen);
}
```



##### 官方答案4: manacher解法

```java
// 暂时看不懂，后面再看
```









#### [3. 无重复字符的最长子串](https://leetcode-cn.com/problems/longest-substring-without-repeating-characters/)

> 给定一个字符串，请你找出其中不含有重复字符的 **最长子串** 的长度。

##### 我的答案

> 这里没答对，只考虑了直线，没考虑窗口，而且效率不行。

```java
/**
 * 如果是相对较短的话， 还可以， 但是如果是较长的字符串，效果不太好，会超时。
 * @param s
 * @return
 */
public int lengthOfLongestSubstring_v0(String s) {

    int maxLeng = 0;

    for (int m = 0; m < s.length(); m++) {
        char[] chars = s.substring(m).toCharArray();

        // Map<Integer, Integer> data = new HashMap<>();
        Map<String, Integer> charData = new HashMap<>();

        // 这个地方刚开始是用0，后来看了官方答案是用m合适点
        for (int i = m; i < chars.length; i++) {
            // System.out.println(chars[i]);
            if (!charData.containsKey(String.valueOf(chars[i]))) {
                if (charData.size() > maxLeng) {
                    maxLeng = charData.size();
                }
            } else {
                if (charData.size() > maxLeng) {
                    maxLeng = charData.size();
                }
                charData.clear();
            }
            charData.put(String.valueOf(chars[i]), i);
        }

        if (charData.size() > maxLeng) {
            maxLeng = charData.size();
        }

        if (chars.length == 1 && maxLeng < 1) {
            maxLeng = 1;
        }
    }

    return maxLeng;
}
```

##### 官方答案

```java
/**
 * 官方的解题方法
 * @param s
 * @return
 */
public int lengthOfLongestSubstring_v1(String s) {
    // 哈希集合，记录每个字符是否出现过
    Set<Character> occ = new HashSet<Character>();

    int n = s.length();
    // 右指针，初始值为 -1，相当于我们在字符串的左边界的左侧，还没有开始移动
    int rk = -1, ans = 0;

    for (int i = 0; i < n; ++i) {
        occ.clear();
        occ.add(s.charAt(i));
        for (rk = i ; rk + 1 < n && !occ.contains(s.charAt(rk + 1)); rk++) {
            occ.add(s.charAt(rk + 1));
        }
        ans = Math.max(ans, rk + 1 - i);

        /*if (i != 0) {
            // 左指针向右移动一格，移除一个字符
            occ.remove(s.charAt(i - 1));
        }
        while (rk + 1 < n && !occ.contains(s.charAt(rk + 1))) {
            // 不断地移动右指针
            occ.add(s.charAt(rk + 1));
            ++rk;
        }*/
      
        // 第 i 到 rk 个字符是一个极长的无重复字符子串
        ans = Math.max(ans, rk - i + 1);
    }

    return ans;
}
```



##### 我看了官方答案优化的一版

```java
public int lengthOfLongestSubstring_v2(String s) {
        // 哈希集合，记录每个字符是否出现过
        Set<Character> occ = new HashSet<Character>();

        int n = s.length();
        // 右指针，初始值为 -1，相当于我们在字符串的左边界的左侧，还没有开始移动
        int rk = -1, ans = 0;

        for (int i = 0; i < n; ++i) {
          
           // 这里主要优化了一个一个循环删除数据的流程
            occ.clear();
            occ.add(s.charAt(i));
          
          	// 注意：这里rk=i哈，加数据的时候加rk+1
            for (rk = i; rk + 1 < n && !occ.contains(s.charAt(rk + 1)); rk++) {
                occ.add(s.charAt(rk + 1));
            }
            ans = Math.max(ans, rk + 1 - i);
          
            // 第 i 到 rk 个字符是一个极长的无重复字符子串
            ans = Math.max(ans, rk - i + 1);
        }

        return ans;
    }
```



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
public ListNode addTwoNumbers_v0(ListNode l1, ListNode l2) {

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
/**
 * 官方解题方法
 * @param l1
 * @param l2
 * @return
 */
public ListNode addTwoNumbers_v1(ListNode l1, ListNode l2) {

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
/**
 * 我的答案：
 * 暴力解法， 两次遍历，直接找数据即可
 * @param nums
 * @param target
 * @return
 */
public int[] twoSum_v0(int[] nums, int target) {
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
/**
 * 官方答案:
 * 一次遍历， 遍历之前， 先准备一个hashtable，遍历时候把数据存储进去， 直接从这里找对应的值即可。这样不用第二次遍历
 *
 * 注意到方法一的时间复杂度较高的原因是寻找 target - x 的时间复杂度过高。因此，我们需要一种更优秀的方法，能够快速寻找数组中是否存在目标元素。如果存在，我们需要找出它的索引。
 *
 * 使用哈希表，可以将寻找 target - x 的时间复杂度降低到从 O(N)O(N) 降低到 O(1)O(1)
 *
 * 作者：LeetCode-Solution
 * 链接：https://leetcode-cn.com/problems/two-sum/solution/liang-shu-zhi-he-by-leetcode-solution/
 * 来源：力扣（LeetCode）
 * 著作权归作者所有。商业转载请联系作者获得授权，非商业转载请注明出处。
 *
 * @param nums
 * @param target
 * @return
 */
public int[] twoSum_v1(int[] nums, int target) {
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