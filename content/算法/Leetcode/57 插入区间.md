[57. 插入区间](https://leetcode.cn/problems/insert-interval/)

给你一个 **无重叠的** _，_按照区间起始端点排序的区间列表。

在列表中插入一个新的区间，你需要确保列表中的区间仍然有序且不重叠（如果有必要的话，可以合并区间）。

intervals = [[1,3],[6,9]], newInterval = [2,5]
[[1,5],[6,9]]

---
### 思路

直接将插入的区间合并到原来的区间里面，然后继续扫描不重叠的区间了

---

### 解答

```c++
class Solution {
public:
    vector<vector<int>> insert(vector<vector<int>>& intervals, vector<int>& newInterval) {
        intervals.emplace_back(newInterval);
        return merge(intervals);
    }


    vector<vector<int>> merge(vector<vector<int>>& intervals) {
        if (intervals.size() == 0 ) {
            return {};
        }

        sort(intervals.begin(),intervals.end());

        vector<vector<int>> merged;

        for(int i =0 ;i<intervals.size();i++) {
            int L = intervals[i][0] , R = intervals[i][1];
            if (!merged.size() || merged.back()[1] < L) {
                merged.push_back({L,R});
            }else {
                merged.back()[1] = max(merged.back()[1],R);
            }
        }

        return merged;
    }
};
```

```rust
impl Solution {
    pub fn insert(intervals: Vec<Vec<i32>>, new_interval: Vec<i32>) -> Vec<Vec<i32>> {
        let mut intervals = intervals;
        // 直接先添加内容
        intervals.push(new_interval);
        intervals.sort();
        let mut ans = vec![];
        let (mut start, mut end) = (intervals[0][0], intervals[0][1]);
        intervals.iter().skip(1).for_each(|x| {
            if x[0] > end {
                ans.push(vec![start, end]);
                start = x[0];
            }
            end = end.max(x[1]);
        });
        ans.push(vec![start, end]);
        ans
    }
}
```

---

### 类似题目


#算法/区间问题