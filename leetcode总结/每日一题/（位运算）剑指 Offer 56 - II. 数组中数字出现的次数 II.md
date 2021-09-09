https://leetcode-cn.com/problems/shu-zu-zhong-shu-zi-chu-xian-de-ci-shu-ii-lcof/

在一个数组 `nums` 中除一个数字只出现一次之外，其他数字都出现了三次。请找出那个只出现一次的数字。

![image-20210510213959407](D:/Program%20Files/typora/Notebook/source/image-20210510213959407.png)



思路：建立一个长度为 32 的数组 c**o**u**n**t**s* ，通过以上方法可记录所有数字的各二进制位的 1 的出现次数。然后将每个位置上的数%3，得到的就是出现一次的那个数的二进制表示。最后用或操作将整个加起来。

```java
class Solution {
    public int singleNumber(int[] nums) {
        int[] counts = new int[32];
        for(int num : nums) {
            for(int j = 0; j < 32; j++) {
                counts[j] += num & 1;
                num >>>= 1;
            }
        }
        int res = 0, m = 3;
        for(int i = 0; i < 32; i++) {
            res <<= 1;
            res |= counts[31 - i] % m;
        }
        return res;
    }
}
```



