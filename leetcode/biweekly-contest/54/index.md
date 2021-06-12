---
title: 第54场双周赛 - dianhsu.top
---
@import "/mystyle.less"

## 第54场双周赛 {ignore=True}
> 返回[:house:首页](../../../index.html)，[:rocket:LeetCode目录](../../index.html)

---

[TOC]

### 检查是否区域内所有整数都被覆盖

#### 简要思路
遍历从`left`到`right`中的每个数字，检查是否被区间覆盖。

#### 参考代码
```cpp
class Solution {
public:
    bool isCovered(vector<vector<int>>& ranges, int left, int right) {
        for(int i = left; i <= right; ++i){
            bool flag = false;
            for(auto& it: ranges){
                if(i >= it[0] and i <= it[1]){
                    flag = true;
                    break;
                }
            }
            if(!flag) return false;
        }
        return true;
    }
};
```
### 找到需要补充粉笔的学生编号

#### 简要思路
首先是统计一轮下来，学生需要的粉笔的总和。

然后将总粉笔数对一轮学生需要的粉笔数取模，保证剩余的粉笔不能完成一轮。

统计粉笔的前缀和，求第一个粉笔前缀和大于剩余的粉笔和的位置，这里可以使用二分查找。

#### 参考代码
```cpp
class Solution {
public:
    int chalkReplacer(vector<int>& chalk, int k) {
        long long sum = 0;
        for(auto it: chalk) sum += it;
        k %= sum;
        vector<long long> vec{0};
        // 求前缀和
        for(auto it: chalk) vec.push_back(vec.back()+it);
        // 二分查找第一个前缀和大于剩余粉笔的位置。
        return upper_bound(vec.begin(), vec.end(), k) - vec.begin() - 1;
    }
};
```
### 最大的幻方

#### 简要思路

分别横向，纵向，斜向和反斜向求前缀和。

幻方默认的大小是`1`，从`2`开始检查遍历每个位置的幻方是否可行。

使用前缀和来判断是否可行。

时间复杂度应该是 $O(n^4)$。

#### 参考代码
```cpp
class Solution {
public:
    int largestMagicSquare(vector<vector<int>>& grid) {
        // 行前缀和
        vector<vector<int>> row(grid.size() + 1, vector<int>(grid[0].size() + 1, 0));
        // 列前缀和
        vector<vector<int>> col(grid.size() + 1, vector<int>(grid[0].size() + 1, 0));
        // 斜向前缀和
        vector<vector<int>> xxx(grid.size() + 1, vector<int>(grid[0].size() + 1, 0));
        // 反斜向前缀和
        vector<vector<int>> yyy(grid.size() + 1, vector<int>(grid[0].size() + 1, 0));
        for(int i = 0; i < grid.size(); ++i){
            for(int j = 0; j < grid[0].size(); ++j){
                row[i+1][j] = row[i][j] + grid[i][j];
                col[i][j+1] = col[i][j] + grid[i][j];
                xxx[i+1][j+1] = xxx[i][j] + grid[i][j];
            }
        }
        for(int i = 0; i < grid.size(); ++i){
            for(int j = grid[0].size() - 1; j >= 0; --j){
                yyy[i+1][j] = yyy[i][j+1] + grid[i][j];
            }
        }

        int ret = 1;
        for(int dir = 2; dir <= min(grid[0].size(), grid.size()); ++ dir){
            bool ok = false;
            for(int i = 0; i <= grid.size() - dir; ++i){
                for(int j = 0; j <= grid[0].size() - dir; ++j){
                    // 以斜向前缀和为标准，检查其他的前缀和是否与斜向前缀和相同
                    int tmp = xxx[i+dir][j+dir] - xxx[i][j];

                    if(yyy[i+dir][j] - yyy[i][j+dir] != tmp){
                        continue;
                    }
                    bool test = true;
                    for(int tp = 0; tp < dir; ++tp){
                        if(col[i + tp][j + dir] - col[i + tp][j] != tmp){
                            test = false;
                            break;
                        }
                        if(row[i + dir][j + tp] - row[i][j + tp] != tmp){
                            test = false;
                            break;
                        }
                    }
                    if(!test) continue;
                    ok = true;
                    break;
                }
                if(ok){
                    ret = dir;
                }
            }
        }
        return ret;
    }
};
```
### 反转表达式值的最少操作次数

#### 简要思路

这是一个区间dp的问题。

在求得表达式的值的时候，同时需要保存当前值取反的最少操作次数。

当进行`|`或者`&`操作的适合，统计最少的操作次数，关系如下表：
> x代表的是第一部分的值和它取反的操作数
> y代表的是第二部分的值和它取反的操作数
> c代表x和y的操作关系


| x          | y          | c   | op(取反的操作数)    |
| ---------- | ---------- | --- | ------------------- |
| `{0, opx}` | `{0, opy}` | `|` | `min(opx, opy)`     |
| `{0, opx}` | `{0, opy}` | `&` | `min(opx, opy) + 1` |
| `{0, opx}` | `{1, opy}` | `|` | `1`                 |
| `{0, opx}` | `{1, opy}` | `&` | `1`                 |
| `{1, opx}` | `{0, opy}` | `|` | `1`                 |
| `{1, opx}` | `{0, opy}` | `&` | `1`                 |
| `{1, opx}` | `{1, opy}` | `|` | `min(opx, opy) + 1` |
| `{1, opx}` | `{1, opy}` | `&` | `min(opx, opy)`     |



PS：虽然LeetCode不考虑代码中标准输入输出，但是打印过程参数可能会因为IO导致结果超时。:cry::cry::cry:
#### 参考代码
```cpp
class Solution {
public:
    // 如上表所示
    int ops(pair<int, int>& x, pair<int, int>& y, char c){
        if(x.first == 0 and y.first == 0){
            if(c == '|'){
                return min(x.second, y.second);
            }else{
                return min(x.second, y.second) + 1;
            }
        }else if(x.first == 1 and y.first == 0){
            return 1;
        }else if(x.first == 0 and y.first == 1){
            return 1;
        }else{
            if(c == '|'){
                return min(x.second, y.second) + 1;
            }else{
                return min(x.second, y.second);
            }
        }
    }
    int minOperationsToFlip(string expression) {
        stack<char> stc;
        stack<pair<int, int>> sti;
        for(auto& ch: expression){
            if(ch == '('){
                stc.push('(');
            }else if(ch == ')'){
                if(stc.top() == '('){
                    stc.pop();
                    if(!stc.empty() and (stc.top() == '|' or stc.top() == '&')){
                        auto x = sti.top(); sti.pop();
                        auto y = sti.top(); sti.pop();
                        int nex = x.first;
                        char tmpc = stc.top();stc.pop();
                        if(tmpc == '|'){
                            nex |= y.first;
                        }else{
                            nex &= y.first;
                        }
                        sti.push({nex, ops(x, y, tmpc)});
                    }
                }else{
                    auto x = sti.top(); sti.pop();
                    auto y = sti.top(); sti.pop();
                    int nex = x.first;
                    char tmpc = stc.top();stc.pop();
                    if(tmpc == '|'){
                        nex |= y.first;
                    }else{
                        nex &= y.first;
                    }
                    sti.push({nex, ops(x, y, tmpc)});
                }
            }else if(ch == '|' or ch == '&'){
                stc.push(ch);
            }else{
                auto x = make_pair((int)(ch - '0'), 1);
                if(!stc.empty() and (stc.top() == '|' or stc.top() == '&')){
                    auto y = sti.top(); 
                    sti.pop();
                    
                    char tmpc = stc.top();stc.pop();
                    int nex = x.first;
                    if(tmpc == '|'){
                        nex |= y.first;
                    }else{
                        nex &= y.first;
                    }
                    sti.push({nex, ops(x, y, tmpc)});
                }else{
                    sti.push(x);
                }
            }
        }
        return sti.top().second;
    }
};
```