给你一个整数数组 `nums` ，请你找出一个具有最大和的连续子数组（子数组最少包含一个元素），返回其最大和。

**子数组** 是数组中的一个连续部分。


---

动态规划的做法  

```rust
impl Solution {
    pub fn max_sub_array(nums: Vec<i32>) -> i32 {
        let ( mut max , mut res) = (nums[0], nums[0]);
        for num in nums.into_iter().skip(1) {
            max = num.max(num + max);
            res = res.max(max);
        }
        res
    }
}
```

#算法/动态规划
