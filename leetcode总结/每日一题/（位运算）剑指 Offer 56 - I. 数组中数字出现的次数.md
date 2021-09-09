https://leetcode-cn.com/problems/shu-zu-zhong-shu-zi-chu-xian-de-ci-shu-lcof/

一个整型数组 `nums` 里除两个数字之外，其他数字都出现了两次。请写程序找出这两个只出现一次的数字。要求时间复杂度是O(n)，空间复杂度是O(1)。

![image-20210510212023653](D:/Program%20Files/typora/Notebook/source/image-20210510212023653.png)



思路： 遍历一遍将整个数组异或，得到的值是两个数字的异或值，异或值的二进制表示两个数在二进制中相同为0，不同为1,。

拿这个值可以判断在某个位上两个数字产生不同，故此根据这个位数来将数组中的数字分为两组(相同的数一定被分到一组)，遍历，与得到某位置上不一样的那个值再来异或，为1的是一个数，为0的是另外一个数。

```java
class Solution {
    public int[] singleNumbers(int[] nums) {
        int re = 0;
        int a = 0,b = 0;
        for (int n:nums){
            re ^=n;
        }
        int h = 1;
        while ((re & h) == 0){
            h <<= 1;
        }
        for (int num:nums){
            if((num&h)==0){
                a^=num;
            }
            else b^=num;
        }
        return new int[]{a,b};

    }
}
```

