# LeetCode算法题

## 回文数

[9.回文数 LeetCode]([9. 回文数 - 力扣（LeetCode）](https://leetcode.cn/problems/palindrome-number/))

### **我第一次的解法：**

> 将输入转化为字符串进行操作，简单的使用两个左右两个指针，分别从左边和最右边遍历，如果所有的元素都相同则是回文数，否则，只要有一个指向的元素不相同，就不是回文
>
> leetcode用时6ms，空间占用41MB

```java
class Solution {
    public boolean isPalindrome(int x) {
        int left = 0;
        int right = String.valueOf(x).length()-1;
        String numstr = String.valueOf(x);
        boolean result = false;
        while (left<=right){
            if (numstr.charAt(left)==numstr.charAt(right)){
                result = true;
                left++;
                right--;
            }else {
                result = false;
                return result;
            }
        }
        return result;
    }
}
```

### **解法二**

[算法连接LeetCode]([回文数 - 回文数 - 力扣（LeetCode）](https://leetcode.cn/problems/palindrome-number/solution/hui-wen-shu-by-leetcode-solution/))

第二个想法是将数字本身反转，然后将反转后的数字与原始数字进行比较，如果它们是相同的，那么这个数字就是回文。但是，如果反转后的数字大于 int.MAX，我们将遇到整数溢出问题。

按照第二个想法，为了避免数字反转可能导致的溢出问题，为什么不考虑只反转 int 数字的一半？毕竟，如果该数字是回文，其后半部分反转后应该与原始数字的前半部分相同。

```java
class Solution {
    public boolean isPalindrome(int x) {
        // 特殊情况：
        // 如上所述，当 x < 0 时，x 不是回文数。
        // 同样地，如果数字的最后一位是 0，为了使该数字为回文，
        // 则其第一位数字也应该是 0
        // 只有 0 满足这一属性
        if (x < 0 || (x % 10 == 0 && x != 0)) {
            return false;
        }

        int revertedNumber = 0;
        while (x > revertedNumber) {
            revertedNumber = revertedNumber * 10 + x % 10;
            x /= 10;
        }

        // 当数字长度为奇数时，我们可以通过 revertedNumber/10 去除处于中位的数字。
        // 例如，当输入为 12321 时，在 while 循环的末尾我们可以得到 x = 12，revertedNumber = 123，
        // 由于处于中位的数字不影响回文（它总是与自己相等），所以我们可以简单地将其去除。
        return x == revertedNumber || x == revertedNumber / 10;
    }
}

作者：LeetCode-Solution
链接：https://leetcode.cn/problems/palindrome-number/solution/hui-wen-shu-by-leetcode-solution/
来源：力扣（LeetCode）
著作权归作者所有。商业转载请联系作者获得授权，非商业转载请注明出处。
```

> - 时间复杂度：O(logn),对于每次迭代，我们会将输入除以 10，获取每位数字，因此时间复杂度为O(logn)
> - 空间复杂度：O(1)，只需要常数空间存放若干变量。
> - leetcode用时6ms,内存消耗40MB

## 公因子的数目

[2427.公因子的数目Leetcode]([2427. 公因子的数目 - 力扣（LeetCode）](https://leetcode.cn/problems/number-of-common-factors/))

### 我的解法

```java
class Solution {
    public int commonFactors(int a, int b) {
        if(a==1 || b==1){
            return 1;
        }
        int sum = 0;
        int min = Math.min(a,b);
        for(int i=1;i<=min;i++){
            if(a%i==0 && b%i==0){
                sum++;
            }
        }
        return sum;

    }
}
执行用时：0 ms, 在所有 Java 提交中击败了100.00%的用户
内存消耗：38.5 MB, 在所有 Java 提交中击败了19.86%的用户
通过测试用例：
76 / 76
```

> 简单的暴力破解，枚举从1到最小的那个数之间的数。

### 官方题解一：枚举到较小值

```java
class Solution {
    public int commonFactors(int a, int b) {
        int ans = 0;
        for (int x = 1; x <= Math.min(a, b); ++x) {
            if (a % x == 0 && b % x == 0) {
                ++ans;
            }
        }
        return ans;
    }
}

作者：LeetCode-Solution
链接：https://leetcode.cn/problems/number-of-common-factors/solution/gong-yin-zi-de-shu-mu-by-leetcode-soluti-u9sl/
来源：力扣（LeetCode）
著作权归作者所有。商业转载请联系作者获得授权，非商业转载请注明出处。
```

> 思想一样 。由于 a 和b 的公因子一定不会超过a 和 b，因此我们只需要在[1,min(a,b)]之间枚举x即可，并判断是不是公因数。
>
> - 时间复杂度:O(n),其中n是给定输入a和b的范围。
> - 空间复杂度O(1).

### 方法二：枚举到最大公约数

x是a和b的公因子，当且仅当x是a和b的最大公约数的因子。因此，我们可以首先求出a和b的最大公约数c=gcd(a,b)，在枚举c的因子个数。

我们可以使用类似方法一种的遍历，对于[1,c]中的整数依次进行枚举，时间复杂度为O(c).我们也可以进行一些优化，因子一定是成对出现的：如果x是c的因子，那么c/x一定是c的因子。因此，我们可以仅对[1,√c]中的整数依次进行枚举，如果枚举到x是c的因子，并且x*x!=c，那么答案额外增加1，这样一来，时间复杂度可以降至O(√c)

```java
class Solution {
    public int commonFactors(int a, int b) {
        int c = gcd(a, b), ans = 0;
        for (int x = 1; x * x <= c; ++x) {
            if (c % x == 0) {
                ++ans;
                if (x * x != c) {
                    ++ans;
                }
            }
        }
        return ans;
    }

    public int gcd(int a, int b) {
        while (b != 0) {
            //引用类型不要用这种方法交换
            a %= b;
            a ^= b;
            b ^= a;
            a ^= b;
        }
        return a;
    }
}

作者：LeetCode-Solution
链接：https://leetcode.cn/problems/number-of-common-factors/solution/gong-yin-zi-de-shu-mu-by-leetcode-soluti-u9sl/
来源：力扣（LeetCode）
著作权归作者所有。商业转载请联系作者获得授权，非商业转载请注明出处。
```

> 时间复杂度：O(√n),其中n是给定输入a和b的范围，需要注意的是，求解a和b最大公约数需要O(logn)的时间，但是渐进意义小于O(√n)，因此可以省略。
>
> 空间复杂度:O(1)

其中这个gcd(a,b)是求解最大公因数的方法，利用的是辗转相除法的思想

> ```java
> 引用类型不要用这种方法交换！！！
> a ^= b;a中保存不同的位
> b ^= a;将b赋值为a，因为用b和（a,b不相同的位）进行异或了
> a ^= b;将a赋值为b，因为用(a,b不相同的位)和a进行异或。
> 这三个操作的结果是进行数字交换
>  a^a=0;a^0=a;
> 因此a=a^b就是在a中保存a和b不同的位
>  b=b^a=b^(a^b)=b^b^a=0^a=a;
> 因此b^=a就是将a的值复制给b,此时b=a
>  a^=b=(a^b)^a(因为此时，b中保存的是a的值)=b
> 因此a^=b就是将a的值赋值为b；
>  [参考](https://www.zhihu.com/question/57945220)
> ```

## 负二进制转换

:star2::star2::star2::star2::star2: [1017. 负二进制转换 - 力扣（LeetCode）](https://leetcode.cn/problems/convert-to-base-2/submissions/)

### 我的解法：不会做

### LeetCode

```java
class Solution {
    public String baseNeg2(int n) {
        Stack<String> stringStack = new Stack<>();
        if(n==0) return "0";
        String result = "";
        int temp = 1;
        while (n!=0){
            int remainder = n & 1;
            n -= remainder;
            n /= -2;

            stringStack.add(String.valueOf(remainder));
        }
        while (!stringStack.isEmpty()){
            result+=stringStack.pop();
        }
        return result;

    }
}
```

## 26.删除有序数组中的重复项

我的解法：使用快慢指针

```java
public int removeDuplicates(int[] nums) {
//        快慢指针试试
        int slow = 0;
        for (int fast = 0;fast<nums.length;fast++){
            if (nums[slow]!=nums[fast]) {
                nums[slow+1] = nums[fast];
                slow++;
            }
        }
        return slow+1;

    }
执行用时：0 ms, 在所有 Java 提交中击败了100.00%的用户
内存消耗：42.9 MB, 在所有 Java 提交中击败了82.00%的用户
```

官方解法：

[【双指针】删除重复项-带优化思路 - 删除有序数组中的重复项 - 力扣（LeetCode）](https://leetcode.cn/problems/remove-duplicates-from-sorted-array/solution/shuang-zhi-zhen-shan-chu-zhong-fu-xiang-dai-you-hu/)

解法一与我的做法一致

```java
 public int removeDuplicates(int[] nums) {
    if(nums == null || nums.length == 0) return 0;
    int p = 0;
    int q = 1;
    while(q < nums.length){
        if(nums[p] != nums[q]){
            nums[p + 1] = nums[q];
            p++;
        }
        q++;
    }
    return p + 1;
}
时间复杂度：O(n)
空间复杂度:O(1)

作者：max-LFszNScOfE
链接：https://leetcode.cn/problems/remove-duplicates-from-sorted-array/solution/shuang-zhi-zhen-shan-chu-zhong-fu-xiang-dai-you-hu/
来源：力扣（LeetCode）
著作权归作者所有。商业转载请联系作者获得授权，非商业转载请注明出处。
```

![image](https://pic.leetcode-cn.com/0039d16b169059e8e7f998c618b6c2b269c2d95b02f43415350bde1f661e503a-1.png)

解法二优化了解法一，解决了当数组中没有重复元素时，还要进行赋值的冗余操作

```java
public int removeDuplicates(int[] nums) {
    if(nums == null || nums.length == 0) return 0;
    int p = 0;
    int q = 1;
    while(q < nums.length){
        if(nums[p] != nums[q]){
            if(q - p > 1){
                nums[p + 1] = nums[q];
            }
            p++;
        }
        q++;
    }
    return p + 1;
}
时间复杂度：O(n)
空间复杂度:O(1)
作者：max-LFszNScOfE
链接：https://leetcode.cn/problems/remove-duplicates-from-sorted-array/solution/shuang-zhi-zhen-shan-chu-zhong-fu-xiang-dai-you-hu/
来源：力扣（LeetCode）
著作权归作者所有。商业转载请联系作者获得授权，非商业转载请注明出处。
```

## 283.移动零

[283. 移动零 - 力扣（LeetCode）](https://leetcode.cn/problems/move-zeroes/)

给定一个数组 `nums`，编写一个函数将所有 `0` 移动到数组的末尾，同时保持非零元素的相对顺序。

**请注意** ，必须在不复制数组的情况下原地对数组进行操作。

**示例 1:**

```txt
输入: nums = [0,1,0,3,12]
输出: [1,3,12,0,0]
```

### 我的解法：快慢指针

```java
public void moveZeroes(int[] nums) {
        //双指针
        int slow = 0;
        int temp = 0;
        for (int fast=0;fast<nums.length;fast++){
            if (nums[fast]!=0 ){
                temp = nums[fast];
                nums[fast] = nums[slow];
                nums[slow] = temp;

                slow++;
            }
        }
        for (int i:nums){
            System.out.println(i);
        }

    }
```

### 官方类似解法

```java
class Solution {
    public void moveZeroes(int[] nums) {
        int n = nums.length, left = 0, right = 0;
        while (right < n) {
            if (nums[right] != 0) {
                swap(nums, left, right);
                left++;
            }
            right++;
        }
    }

    public void swap(int[] nums, int left, int right) {
        int temp = nums[left];
        nums[left] = nums[right];
        nums[right] = temp;
    }
}
时间复杂度:O(n)其中n为序列长度。每个位置至多被遍历两次
空间复杂度:O(1)只需要常数的空间存放若干变量。
作者：LeetCode-Solution
链接：https://leetcode.cn/problems/move-zeroes/solution/yi-dong-ling-by-leetcode-solution/
来源：力扣（LeetCode）
著作权归作者所有。商业转载请联系作者获得授权，非商业转载请注明出处。
```

## 884.比较含退格的字符串

[844. 比较含退格的字符串 - 力扣（LeetCode）](https://leetcode.cn/problems/backspace-string-compare/)

给定 s 和 t 两个字符串，当它们分别被输入到空白的文本编辑器后，如果两者相等，返回 true 。# 代表退格字符。

注意：如果对空文本输入退格字符，文本继续为空。

**示例 ：**

```
输入：s = "ab#c", t = "ad#c"
输出：true
解释：s 和 t 都会变成 "ac"。
示例 2：

输入：s = "ab##", t = "c#d#"
输出：true
解释：s 和 t 都会变成 ""。
示例 3：

输入：s = "a#c", t = "b"
输出：false
解释：s 会变成 "c"，但 t 仍然是 "b"。
```

### 我的解法

使用快慢指针，快指针每次检索到一个#，slow就退一格.

```java
public boolean backspaceCompare(String s, String t) {
        String s1 = newStr(s);
        String s2 = newStr(t);
        System.out.println(s1);
        System.out.println(s2);
        return s1.equals(s2)?true:false;
    }
    public static String newStr(String s){
        int slow = 0;
        int fast = 0;
//        bxo#j##tw
        //"y#fo##f"
//        "y#f#o##f"
        StringBuffer result = new StringBuffer("");
        for (int i=0;i<s.length();i++){
            result.append(s.charAt(i));
        }
        while (fast<s.length()){
            if (!('#'==s.charAt(fast))){
                if (slow<0){//为了防止多个###
                    slow=0;
                }
                result.setCharAt(slow,s.charAt(fast));
                slow++;
            }else {
                slow--;
            }
            fast++;
        }
        if (slow<0){
            return "\"\"";
        }

        return String.valueOf(result.substring(0,slow));

    }
```

### Leetcode方法一：重构字符串

[比较含退格的字符串 - 比较含退格的字符串 - 力扣（LeetCode）](https://leetcode.cn/problems/backspace-string-compare/solution/bi-jiao-han-tui-ge-de-zi-fu-chuan-by-leetcode-solu/)

删掉所有的空格字符和应当被删掉的字符，用栈处理即可。

```java
class Solution {
    public boolean backspaceCompare(String S, String T) {
        return build(S).equals(build(T));
    }

    public String build(String str) {
        StringBuffer ret = new StringBuffer();
        int length = str.length();
        for (int i = 0; i < length; ++i) {
            char ch = str.charAt(i);
            if (ch != '#') {
                ret.append(ch);
            } else {
                if (ret.length() > 0) {
                    ret.deleteCharAt(ret.length() - 1);
                }
            }
        }
        return ret.toString();
    }
}

作者：LeetCode-Solution
链接：https://leetcode.cn/problems/backspace-string-compare/solution/bi-jiao-han-tui-ge-de-zi-fu-chuan-by-leetcode-solu/
来源：力扣（LeetCode）
著作权归作者所有。商业转载请联系作者获得授权，非商业转载请注明出处。
```

### LeetCode方法二：双指针

```java
class Solution {
    public boolean backspaceCompare(String S, String T) {
        int i = S.length() - 1, j = T.length() - 1;
        int skipS = 0, skipT = 0;

        while (i >= 0 || j >= 0) {
            while (i >= 0) {
                if (S.charAt(i) == '#') {
                    skipS++;
                    i--;
                } else if (skipS > 0) {
                    skipS--;
                    i--;
                } else {
                    break;
                }
            }
            while (j >= 0) {
                if (T.charAt(j) == '#') {
                    skipT++;
                    j--;
                } else if (skipT > 0) {
                    skipT--;
                    j--;
                } else {
                    break;
                }
            }
            if (i >= 0 && j >= 0) {
                if (S.charAt(i) != T.charAt(j)) {
                    return false;
                }
            } else {
                if (i >= 0 || j >= 0) {
                    return false;
                }
            }
            i--;
            j--;
        }
        return true;
    }
}

作者：LeetCode-Solution
链接：https://leetcode.cn/problems/backspace-string-compare/solution/bi-jiao-han-tui-ge-de-zi-fu-chuan-by-leetcode-solu/
来源：力扣（LeetCode）
著作权归作者所有。商业转载请联系作者获得授权，非商业转载请注明出处。
```



## 1040.移动石子直到连续II :heart::heart::heart::heart::heart::heart::heart:

**根本不会！！！**

[1040. 移动石子直到连续 II - 力扣（LeetCode）](https://leetcode.cn/problems/moving-stones-until-consecutive-ii/)

在一个长度 无限 的数轴上，第 i 颗石子的位置为 stones[i]。如果一颗石子的位置最小/最大，那么该石子被称作 端点石子 。

每个回合，你可以将一颗端点石子拿起并移动到一个未占用的位置，使得该石子不再是一颗端点石子。

值得注意的是，如果石子像 stones = [1,2,5] 这样，你将 无法 移动位于位置 5 的端点石子，因为无论将它移动到任何位置（例如 0 或 3），该石子都仍然会是端点石子。

当你无法进行任何移动时，即，这些石子的位置连续时，游戏结束。

要使游戏结束，你可以执行的最小和最大移动次数分别是多少？ 以长度为 2 的数组形式返回答案：answer = [minimum_moves, maximum_moves] 。

```java
public int[] numMovesStonesII(int[] stones) {
        int n = stones.length;
        Arrays.sort(stones);
        if (stones[n - 1] - stones[0] + 1 == n) {
            return new int[]{0, 0};
        }
        int ma = Math.max(stones[n - 2] - stones[0] + 1, stones[n - 1] - stones[1] + 1) - (n - 1);
        int mi = n;
        for (int i = 0, j = 0; i < n && j + 1 < n; ++i) {
            while (j + 1 < n && stones[j + 1] - stones[i] + 1 <= n) {
                ++j;
            }
            if (j - i + 1 == n - 1 && stones[j] - stones[i] + 1 == n - 1) {
                mi = Math.min(mi, 2);
            } else {
                mi = Math.min(mi, n - (j - i + 1));
            }
        }
        return new int[]{mi, ma};
    }
```

## 2404.出现最频繁的偶数元素

[2404. 出现最频繁的偶数元素 - 力扣（LeetCode）](https://leetcode.cn/problems/most-frequent-even-element/)

给你一个整数数组 `nums` ，返回出现最频繁的偶数元素。

如果存在多个满足条件的元素，只需要返回 **最小** 的一个。如果不存在这样的元素，返回 `-1` 。

### 我的解法

使用hashMap，保存所有的偶数元素，同时存储最大的出现频率以及对应的元素i。

```java
class Solution {
    public int mostFrequentEven(int[] nums) {
        int min = -1;
        int value = 0;
        Map<Integer,Integer> map = new HashMap<>();
        for (int i:nums){
            if (i%2==0){
                map.put(i,map.getOrDefault(i,0)+1);
                if (value < map.get(i)){
                    value = map.get(i);
                    min = i;
                } else if (value == map.get(i)) {
                    min = min>i?i:min;
                }

            }
        }
        return min;

    }
}
```

### LeetCode官方一:哈希表计数

遍历数组 nums，并且使用哈希表 count 记录偶数元素的出现次数。使用 res和 ct 分别记录当前出现次数多的元素值以及对应的出现次数。遍历哈希表中的元素，如果元素的出现次数大于 ct 或者出现次数等于 ct 且元素值小 res，那么用res 记录当前遍历的元素值，并且用 ct 记录当前遍历的元素的出现次数。

```java
class Solution {
    public int mostFrequentEven(int[] nums) {
        Map<Integer, Integer> count = new HashMap<Integer, Integer>();
        for (int x : nums) {
            if (x % 2 == 0) {
                count.put(x, count.getOrDefault(x, 0) + 1);
            }
        }
        int res = -1, ct = 0;
        for (Map.Entry<Integer, Integer> entry : count.entrySet()) {
            if (res == -1 || entry.getValue() > ct || entry.getValue() == ct && res > entry.getKey()) {
                res = entry.getKey();
                ct = entry.getValue();
            }
        }
        return res;
    }
}

作者：LeetCode-Solution
链接：https://leetcode.cn/problems/most-frequent-even-element/solution/chu-xian-zui-pin-fan-de-ou-shu-yuan-su-b-cxeo/
来源：力扣（LeetCode）
著作权归作者所有。商业转载请联系作者获得授权，非商业转载请注明出处。
时间复杂度O(n),其中n是数组的长度，遍历数组和哈希表都需要O(n)
空间复杂度O(n),保存哈希表需要O(n)
```

## 2409.统计共同度过的日子数

LeetCode解法:分别计算出每个日子是一年中的第几天后求差

[2409. 统计共同度过的日子数 - 力扣（LeetCode）](https://leetcode.cn/problems/count-days-spent-together/)

```java
class Solution {
    public int countDaysTogether(String arriveAlice, String leaveAlice, String arriveBob, String leaveBob) {
        int[] datesOfMonths = {31, 28, 31, 30, 31, 30, 31, 31, 30, 31, 30, 31};
        int[] prefixSum = new int[13];
        for (int i = 0; i < 12; i++) {
            prefixSum[i + 1] = prefixSum[i] + datesOfMonths[i];
        }

        int arriveAliceDay = calculateDayOfYear(arriveAlice, prefixSum);
        int leaveAliceDay = calculateDayOfYear(leaveAlice, prefixSum);
        int arriveBobDay = calculateDayOfYear(arriveBob, prefixSum);
        int leaveBobDay = calculateDayOfYear(leaveBob, prefixSum);
        return Math.max(0, Math.min(leaveAliceDay, leaveBobDay) - Math.max(arriveAliceDay, arriveBobDay) + 1);
    }

    public int calculateDayOfYear(String day, int[] prefixSum) {
        int month = Integer.parseInt(day.substring(0, 2));
        int date = Integer.parseInt(day.substring(3));
        return prefixSum[month - 1] + date;
    }
}

作者：LeetCode-Solution
链接：https://leetcode.cn/problems/count-days-spent-together/solution/tong-ji-gong-tong-du-guo-de-ri-zi-shu-by-1iwp/
来源：力扣（LeetCode）
著作权归作者所有。商业转载请联系作者获得授权，非商业转载请注明出处。
O(1)
O(1)
```

# 滑动窗口

## 904.水果成篮

[904. 水果成篮 - 力扣（LeetCode）](https://leetcode.cn/problems/fruit-into-baskets/)



### 我的方法：不会

### LeetCode:滑动窗口

```java
public int totalFruit(int[] fruits){
        int start = 0;
        int temp = 0;
        HashMap<Integer, Integer> map = new HashMap<>();
        for (int end = 0;end<fruits.length;end++){
            map.put(fruits[end],map.getOrDefault(fruits[end],0)+1);
            while (map.size()>2){
                //品种数量大于2,令左边指针所对应的数目-1，当大小为0的时候，移出集合
                map.put(fruits[start],map.get(fruits[start])-1);
                if (map.get(fruits[start])==0){
                    map.remove(fruits[start]);
                }
                start++;
            }
            temp = Math.max(temp,end-start+1);
        }
        return temp;
    }
```

> 时间复杂度O(n)
>
> 空间复杂度O(1)

## 76.最小滑动字串
