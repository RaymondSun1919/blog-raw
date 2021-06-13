---
title: 第245场周赛 - dianhsu.top
---
@import "/mystyle.less"

## 第 245 场周赛 {ignore=True}
> 返回[:house:首页](../../../index.html)，[:rocket:LeetCode目录](../../index.html)

---

[TOC]


### 重新分配字符使所有字符串都相等

#### 简要思路

统计每个字母出现的次数是否能被单词个数整除。

#### 参考代码
```cpp
class Solution {
public:
    bool makeEqual(vector<string>& words) {
        vector<int> alphabets(26, 0);
        for(auto& it: words){
            for(auto& ch: it){
                alphabets[ch - 'a']++;
            }
        }
        for(int i = 0; i < 26; ++i){
            if(alphabets[i] % words.size() != 0) return false;
        }
        return true;
    }
};
```

### 可移除字符的最大数目

#### 简要思路

二分结果，判断结果是否可行。

**PS**: 判断移除之后的字符串是否还能找到子序列p，可以用非字母字符替换字符串s中的字符。之前用unordered_set超时了。

**PPS**: 二分的时候，边界的判断到最后一个区间为2的位置的时候，`mid`一直等于`low`，如果`low`又是满足条件的情况下，`low`就没法更新。这样的处理方式一般有两种：
- 第一种是设置`mid`的时候，使得`mid = (low + high + 1) >> 1`；
- 第二种是判断`mid`是不是之前访问过，如果访问过的话，就使`mid = mid + 1`。

#### 参考代码
```cpp
class Solution {
public:
    bool check(string s, string& p, vector<int>& rv, int x){
        for(int i = 0; i < x; ++i){
            s[rv[i]] = '#';
        }
        int cur = 0;
        for(int i = 0; i < s.length(); ++i){
            if(s[i] == p[cur]){
                ++cur;
                if(cur == p.length()) return true;
            }
        }
        return false;
    }
    int maximumRemovals(string s, string p, vector<int>& removable) {
        int low = 0;
        int high = min(removable.size(), s.length() - p.length());
        int prev = -1;
        while(low < high){
            int mid = (low + high) / 2;
            if(mid == prev){
                mid++;
            }
            prev = mid;
            if(check(s, p, removable, mid)){
                low = mid;
            }else{
                high = mid - 1;
            }
        }
        return low;
    }
};
```

### 合并若干三元组以形成目标三元组
#### 简要思路
将`triplets`中的满足**每个位置的数都不大于`target`对应位置的数**的三元组纳入到合并对象。
比较合并之后的三元组和目标三元组是否一致。
#### 参考代码
```cpp
class Solution {
public:
    bool mergeTriplets(vector<vector<int>>& triplets, vector<int>& target) {
        vector<int> tmp{0, 0, 0};
        for(int i = 0; i < triplets.size(); ++i){
            if(triplets[i][0] <= target[0] and 
                    triplets[i][1] <= target[1] and 
                    triplets[i][2] <= target[2]){
                tmp[0] = max(tmp[0], triplets[i][0]);
                tmp[1] = max(tmp[1], triplets[i][1]);
                tmp[2] = max(tmp[2], triplets[i][2]);
            }
        }
        return tmp[0] == target[0] and 
                tmp[1] == target[1] and 
                tmp[2] == target[2];
    }
};
```

### 最佳运动员的比拼回合

#### 简要思路
记忆化搜索问题。

首先是状态分析，判断某一轮中两个最佳运动员的位置。合并状态之后，最佳运动员的位置实际上只有四种。
1. 第一个最佳运动员和第二最佳运动员刚好在这一轮的对称位置。
2. 第一个最佳运动员在前半部分，第二个最佳运动员在前半部分，且第二个最佳运动员的位置比第一个最佳运动员靠后。
3. 第一个最佳运动员在前半部分，第二个最佳运动员在正中央。这种情况在计算的时候，合并到上面第一种情况里面去了。
4. 第一个最佳运动员在前半部分，第二个最佳运动员在后半部分，且第一个最佳运动员前面的运动员数量小于第二个最佳运动员的后面的运动员数量。

 

#### 参考代码
```cpp
class Solution {
private:
    pair<int, int> dp[30][30][30];
public:
    pair<int, int> dfs(int n, int firstPlayer, int secondPlayer) {
        // 如果第一个最佳运动员和第二个最佳运动员位置对称。
        if (firstPlayer + secondPlayer == n - 1) {
            return {1, 1};
        }
        int pos1 = firstPlayer;
        int pos2 = secondPlayer;
        // 如果两个运动员的位置都在后半部分，就将两个最佳运动员翻转到前半部分。
        if (pos1 >= (n + 1) / 2 and pos2 >= (n + 1) / 2) {
            pos1 = n - 1 - pos1;
            pos2 = n - 1 - pos2;
        }
        // 保证第一个最佳运动员在第二个最佳运动员之前。
        if (pos1 > pos2) {
            swap(pos1, pos2);
        }
        int li = pos1;
        int ri = n - 1 - pos2;
        // 如果第一个最佳运动员前面的运动员的数量大于第二个最佳运动员后面的运动员的数量，就将两个最佳运动员都调整到他们的对称位置。
        if (n - 1 - pos2 < pos1) {
            pos1 = ri;
            pos2 = n - 1 - li;
        }
        // 如果这种情况之前已经计算过了，就取上次计算的值。并且把这次的结果更新成上次的值。
        if (dp[n][pos1][pos2].first != 0x3f3f3f3f) {
            dp[n][firstPlayer][secondPlayer] = dp[n][pos1][pos2];
            return dp[n][firstPlayer][secondPlayer];
        }
        int nex = (n + 1) / 2;
        pair<int, int> ret = {0x3f3f3f3f, 0};
        if (pos2 < (n + 1) / 2) {
            // 第二个最佳运动员的位置在左侧或中间
            // i是下次第一个最佳运动员为位置，j是下次第二个运动员的位置。
            for (int i = 0; i <= pos1; i++) {
                for (int j = i + 1; j <= i + pos2 - pos1; ++j) {
                    auto it = dfs(nex, i, j);
                    ret.first = min(ret.first, it.first + 1);
                    ret.second = max(ret.second, it.second + 1);
                }
            }
        } else {
            // 第二个最佳运动员的位置在右侧
            /*
                +---+---+---+---+---+---+---+---+---+---+---+---+---+---+---+---+---+---+---+
                | 0 | 1 | 2 | 3 | 4 | 5 | 6 | 7 | 8 | 9 | 10| 11| 12| 13| 14| 15| 16| 17| 18|
                +---+---+---+---+---+---+---+---+---+---+---+---+---+---+---+---+---+---+---+
                |   |   | i |   |   |   | j'|   |   |   |   |   | j |   |   |   |   |   |   |
                +---+---+---+---+---+---+---+---+---+---+---+---+---+---+---+---+---+---+---+

                n = 19
                i = 2
                j = 12
                j'= n - j - 1  (j的对称位置)

                j 到 j' 之前的空格数 j-j'-1=2*j-n ，在下一个状态是 j - n/2
                j'到 i  之间的空格数 j'-i-1=n-j-i-2 ，在下个状态是 [0, n - j - i - 2]

            */
            // i是下次第一个最佳运动员为位置，j是下次第二个运动员的位置。
            for (int i = 0; i <= pos1; ++i) {
                for (int j = i + (pos2 - n / 2) + 1; j <= i + (n+1)/2 - pos1 - 1; ++j) {
                    auto it = dfs(nex, i, j);
                    ret.first = min(ret.first, it.first + 1);
                    ret.second = max(ret.second, it.second + 1);
                }
            }
        }
        dp[n][firstPlayer][secondPlayer] = ret;
        return dp[n][firstPlayer][secondPlayer];
    }
    vector<int> earliestAndLatest(int n, int firstPlayer, int secondPlayer) {
        memset(dp, 0x3f, sizeof dp);
        auto it = dfs(n, firstPlayer - 1, secondPlayer - 1);
        return vector<int>{it.first, it.second};
    }
};
```