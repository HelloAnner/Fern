[56. 合并区间](https://leetcode.cn/problems/merge-intervals/)

以数组 `intervals` 表示若干个区间的集合，其中单个区间为 `intervals[i] = [starti, endi]`请你合并所有重叠的区间，并返回 _一个不重叠的区间数组，该数组需恰好覆盖输入中的所有区间_ 

intervals = [[1,3],[2,6],[8,10],[15,18]]
输出：[[1,6],[8,10],[15,18]]
解释：区间 [1,3] 和 [2,6] 重叠, 将它们合并为 [1,6].

---
### 思路

先排序首位置，然后直接一次遍历完成

---

### 解答

```c++
class Solution {
public:
    vector<vector<int>> merge(vector<vector<int>>& intervals) {
        if (intervals.size() == 0 ) {
            return {};
        }
        sort(intervals.begin() , intervals.end());
        vector<vector<int>> merged;

        for (int i=0;i<intervals.size();i++) {
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
    pub fn merge(intervals: Vec<Vec<i32>>) -> Vec<Vec<i32>> {
        let mut intervals = intervals;
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