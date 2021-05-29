---
title: 第53场双周赛 - dianhsu.top
---
@import "/mystyle.less"

## 第53场双周赛 {ignore=True}
> 返回[:house:首页](../../../index.html)，[:rocket:LeetCode目录](../../index.html)

---


[TOC]

### 长度为三且各字符不同的子字符串

遍历所有的长度为3子字符串，然后判断三个字符是否互不相等

```cpp
class Solution {
public:
    int countGoodSubstrings(string s) {
        if(s.length() < 3) return 0;
        int ret = 0;
        for(int i = 0; i < s.length() - 2; ++i){
            if((s[i] != s[i + 1]) and (s[i] != s[i + 2]) and (s[i + 1] != s[i + 2])){
                ++ret;
            }
        }
        return ret;
    }
};
```

### 数组中最大数对和的最小值

排序之后，将正序的排列和反序的排列依次相加，找到最大的数对和

```cpp
class Solution {
public:
    int minPairSum(vector<int>& nums) {
        sort(nums.begin(), nums.end());
        int ret = 0;
        for(int i = 0; i < nums.size(); ++i){
            ret = max(ret, nums[i] + nums[nums.size() - 1 - i]);
        }
        return ret;
    }
};
```

### 矩阵中最大的三个菱形和

首先，分析可以知道，最大的可能的存在的菱形的边长为$\frac{min(n, m)}{2} + 1$

我们可以对左下和右上两条对角线，分别计算前缀和

然后再遍历每个菱形，可以用$O(1)$的时间根据前缀和计算出该菱形的边长

![image](https://cdn.dianhsu.top/PicGo/20210529235849.png)

我们用对角前缀可以很方便计算这个菱形的边长

```cpp
class Solution {
public:
    bool check(int x, int y, int n, int m){
        return x >= 0 and x < n and y >= 0 and y < m;
    }
    vector<int> getBiggestThree(vector<vector<int>>& grid) {
        int n = grid.size();
        int m = grid[0].size();
        int border = min(n, m)/2 + 1;
        vector<vector<int>> mp1(n, vector<int>(m, 0)), mp2(n, vector<int>(m, 0));
        for(int i = 0; i < n; ++i){
            for(int j = 0; j < m; ++j){
                if(i > 0 and j > 0){
                    mp1[i][j] = grid[i][j] + mp1[i-1][j-1];
                }else{
                    mp1[i][j] = grid[i][j];
                }
            }
        }
        for(int i = 0; i < n; ++i){
            for(int j = m - 1; j >= 0; --j){
                if(i > 0 and j < m - 1){
                    mp2[i][j] = grid[i][j] + mp2[i - 1][j + 1];
                }else{
                    mp2[i][j] = grid[i][j];
                }
            }
        }
        set<int> st;
        for(int i = 0; i < n; ++i){
            for(int j = 0; j < m; ++j){
                for(int k = 0; k <= border; ++k){
                    if(k == 0){
                        st.insert(grid[i][j]);
                        continue;
                    }
                    if(i - k >= 0 and i + k < n and j - k >= 0 and j + k < m){
                        int ux = i - k;
                        int dx = i + k;
                        int ly = j - k;
                        int ry = j + k;
                        
                        int ans = mp2[i][ly] + mp2[dx][j] + mp1[dx][j] + mp1[i][ry];
                        if(check(i - 1, ly - 1, n, m)){
                            ans -= mp1[i - 1][ly - 1];
                        }
                        if(check(ux - 1, j - 1, n, m)){
                            ans -= mp1[ux - 1][j - 1];
                        }
                        if(check(ux - 1, j + 1, n, m)){
                            ans -= mp2[ux - 1][j + 1];
                        }
                        if(check(i - 1, ry + 1, n, m)){
                            ans -= mp2[i - 1][ry + 1];
                        }
                        ans = ans - grid[i][ly] - grid[i][ry] - grid[dx][j] - grid[ux][j];
                        st.insert(ans);
                    }
                }
            }
        }
        vector<int> ret;
        for(auto it = st.rbegin(); it != st.rend(); ++it){
            ret.push_back(*it);
            if(ret.size() == 3) break;
        }
        return ret;
    }
};

```