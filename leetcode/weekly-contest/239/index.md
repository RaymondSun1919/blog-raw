---
title: 第239场周赛 - dianhsu.top
---


## 第 239 场周赛
> 返回[:house:首页](../../../index.html)
> 返回[:rocket:LeetCode目录](../../index.html)

---
@import "/mystyle.less"
[TOC]


### 到目标元素的最小距离
遍历数组，找到和target相同的数，计算位置差的绝对值的最小值。
```cpp
class Solution {
public:
    int getMinDistance(vector<int>& nums, int target, int start) {
        int minD = 0x3f3f3f3f;
        for(int i = 0; i < nums.size(); ++i){
            if(nums[i] == target){
                minD = min(abs(i - start), minD);
            }
        }
        return minD;
    }
};
```
### 将字符串拆分为递减的连续值
分别设定第一个数的长度为`1`到`n - 1`。在第一个数确定的情况下，dfs查找下一个数。为了控制数字大小，第一个数不超过`9999999999`。注意`1000`和`0000`的处理即可。
```cpp
class Solution {
public:
    bool dfs(conststring& s, int pos, longlong val){
        if(pos == s.length()) return true;
        if(val == 0){
            bool allZero = true;
            for(int i = pos; i < s.length(); ++i){
                if(s[i] != '0'){                    
                    allZero = false;break;                
                }            
            }
            if(allZero) 
                return true;        
        }
        int nexPos = pos + 1;
        long long tmp = s[pos] - '0';
        while(nexPos < s.length() and tmp < val - 1){
            tmp = tmp * 10 + s[nexPos++] - '0';        
        }
        if(tmp == val - 1){
            return dfs(s, nexPos, tmp);        
        }
        return false;
    }
    bool splitString(string s){
        bool allZero = true;
        for(auto& it: s){
            if(it != '0'){
                allZero = false;
                break;
            }        
        }
        if(allZero) return false;
        for(int i = 1; i < s.length(); ++i){
            long long cnt = 0;
            for(int j = 0; j < i; ++j){
                cnt = cnt * 10 + s[j] - '0';
            }
            if(cnt > 999999999999LL){
                break;            
            }
            if(dfs(s, i, cnt)){
                return true;
            }
        }
        return false;
    }
};
```
### 邻位交换的最小次数
先求当前序列的第`k`个后续序列。然后冒泡排序，统计交换次数。

```cpp

classSolution {
public:
    int getMinSwaps(string num, int k){
        string nex = num;
        while(k--){            
            next_permutation(nex.begin(), nex.end());        
        }
        int ret = 0;
        int pos = 0;
        for(int i = 0; i < nex.length(); ++i){
            if(num[i] != nex[i]){                
                pos = i;
                break;            
            }        
        }
        for(int i = pos; i < nex.length(); ++i){
            auto nexPos = find(nex.begin() + i, nex.end(), num[i]);
            for(int j = nexPos - nex.begin(); j > i; --j){
                swap(nex[j], nex[j - 1]);         
                ++ret;            
            }        
        }
        return ret;    
    }
};
```

### 包含每个查询的最小区间

#### 预处理排序
对于查询，需要离线处理。
首先将查询列表从小向大排序，同时需要记录查询列表原本的位置信息。
将区间列表按照左端点从小到大排序，右端点从小到大排序。

#### 遍历查询，计算结果
对于每一个查询：
首先将左端点符合条件的区间（即左端点小于或等于当前查询的值）插入到优先队列，优先队列按照区间长度，从小到大排序。
然后检查优先队列的堆顶，如果堆顶的区间的右端点满足条件（即右端点大于或者等于当前查询的值），即查询到结果。
如果不满足就弹出堆顶，继续查询下一个堆顶。
直到堆顶满足条件或者优先队列为空为止。

#### 整理并返回结果
遍历完所有的查询之后，将查询列表按照原有的位置信息还原。
输出结果即可。

```cpp
struct Query{
    int val;
    int index;
    int len;
    bool operator < (const Query& q) const{
        return val < q.val;
    }
};
struct Interval{
    int tail;
    int len;
    bool operator < (const Interval& i) const{
        return len > i.len;
    }
};
class Solution {
public:
    vector<int> minInterval(vector<vector<int>>& intervals, vector<int>& queries) {
        sort(intervals.begin(), intervals.end(), [](const vector<int>& a, const vector<int>& b){
            if(a[0] == b[0]){
                return a[1] < b[1];
            }else{
                return a[0] < b[0];
            }
        });
        int tail = 0;
        vector<Query> qVec;
        for(int i = 0; i < queries.size(); ++i){
            qVec.push_back({queries[i], i, 0});
        }
        sort(qVec.begin(), qVec.end());
        priority_queue<Interval> PQ;
        for(auto& it: qVec){
            while(tail < intervals.size()  and intervals[tail][0] <= it.val){                
                PQ.push({intervals[tail][1], intervals[tail][1] - intervals[tail][0] + 1});
                ++tail;
            }
            while(!PQ.empty() and PQ.top().tail < it.val){
                PQ.pop();
            }
            if(PQ.empty()){
                it.len = -1;
            }else{
                it.len = PQ.top().len;    
            }
        }
        vector<int> ret(qVec.size());
        for(auto& it:qVec){
            ret[it.index] = it.len;
        }
        return ret;
    }
};

```