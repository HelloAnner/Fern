[2760. 最长奇偶子数组](https://leetcode.cn/problems/longest-even-odd-subarray-with-threshold/)


给你一个下标从 **0** 开始的整数数组 `nums` 和一个整数 `threshold` 。

请你从 `nums` 的子数组中找出以下标 `l` 开头、下标 `r` 结尾 `(0 <= l <= r < nums.length)` 且满足以下条件的 **最长子数组** ：

- `nums[l] % 2 == 0`
- 对于范围 `[l, r - 1]` 内的所有下标 `i` ，`nums[i] % 2 != nums[i + 1] % 2`
- 对于范围 `[l, r]` 内的所有下标 `i` ，`nums[i] <= threshold`

以整数形式返回满足题目要求的最长子数组的长度。

**注意：子数组** 是数组中的一个连续非空元素序列。

---

标准的一个双层循环单次遍历的逻辑

```rust
impl Solution {
    pub fn longest_alternating_subarray(nums: Vec<i32>, threshold: i32) -> i32 {
        let n = nums.len();
        let mut ans = 0;
        let mut i = 0;
        while i < n {
            if nums[i] > threshold || nums[i] % 2 != 0 {
                i += 1; // 直接跳过
                continue;
            }
            let start = i; // 记录这一组的开始位置
            i += 1; // #开始位置已经满足要求，从下一个位置开始判断
            while i < n && nums[i] <= threshold && nums[i] % 2 != nums[i - 1] % 2 {
                i += 1;
            }
            // 从 start 到 i-1 是满足题目要求的（并且无法再延长的）子数组
            ans = ans.max(i - start);
        }
        ans as i32
    }
}
```

可以说是一个非常简单的模版了

#算法/模拟  