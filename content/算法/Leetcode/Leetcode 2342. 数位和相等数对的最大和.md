[2342. 数位和相等数对的最大和](https://leetcode.cn/problems/max-sum-of-a-pair-with-equal-sum-of-digits/)

给你一个下标从 **0** 开始的数组 `nums` ，数组中的元素都是 **正** 整数。请你选出两个下标 `i` 和 `j`（`i != j`），且 `nums[i]` 的数位和 与  `nums[j]` 的数位和相等。

请你找出所有满足条件的下标 `i` 和 `j` ，找出并返回 `nums[i] + nums[j]` 可以得到的 **最大值** 

例子：
nums = [18,43,36,13,7]
满足条件的数对 (i, j) 为：
- (0, 2) ，两个数字的数位和都是 9 ，相加得到 18 + 36 = 54 。
- (1, 4) ，两个数字的数位和都是 7 ，相加得到 43 + 7 = 50 。
所以可以获得的最大和是 54 。

---

枚举右，维护左：

不妨设 $i<ji<ji<j$。枚举 $nums[j]\textit{nums}[j]nums[j]$，设 n$ums[j]\textit{nums}[j]nums[j]$ 的数位和为 sss，我们需要知道 jjj 左边的数位和也等于 sss 的最大的 $nums[i]\textit{nums}[i]nums[i]$。可以用一个哈希表（或者数组）$mx[s]\textit{mx}[s]mx[s]$ 记录数位和为 sss 对应的最大的 $nums[i$]。

用 $mx[s]+nums[j]\textit{mx}[s]+\textit{nums}[j]mx[s]+nums[j]$ 更新答案的最大值

如果不存在两个数位和相同的数，返回 −1-1−1。

---

```rust
impl Solution {
    pub fn maximum_sum(nums: Vec<i32>) -> i32 {
        let mut ans = -1;
        let mut mx = [0; 82]; // 至多 9 个 9 相加
        for num in nums {
            let mut s = 0; // num 的数位和
            let mut x = num;
            while x > 0 { // 枚举 num 的每个数位
                s += x % 10;
                x /= 10;
            }
            let s = s as usize;
            if mx[s] > 0 { // 说明左边也有数位和等于 s 的元素
                ans = ans.max(mx[s] + num); // 更新答案的最大值
            }
            mx[s] = mx[s].max(num); // 维护数位和等于 s 的最大元素
        }
        ans
    }
}
```


---

对于这种一左一右两个数的题目，通用套路是「枚举右，维护左」

#算法/模拟  