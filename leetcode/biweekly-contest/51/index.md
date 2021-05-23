
## 第51场双周赛
> 返回[:house:首页](../../../index.html)
> 返回[:rocket:LeetCode目录](../../index.html)
---
[TOC]

### 将所有数字用字符替换
遍历一遍字符串，将奇数位置替换成前一个字母加上当前位置的数字即可。
```cpp
class Solution {
public:
    string replaceDigits(string s) {
        for(int i = 1; i < s.length(); i += 2){
            s[i] = s[i - 1] + s[i] - '0';
        }
        return s;
    }
};

```


### 座位预约管理系统
因为每次预约都取最小，我们使用小顶堆来维护这个结果。
初始状态下，堆里面有n个数。
锁定一个座位，就从堆里面取最小的数。
释放一个座位，就插入一个数进入堆。
```cpp
class SeatManager {
public:
    priority_queue<int, vector<int>, greater<int>> PQ;
    SeatManager(int n) {
        for(int i = 1; i <= n; ++i){
            PQ.push(i);
        }
    }
    int reserve() {
        int ret = PQ.top();
        PQ.pop();
        return ret;
    }
    void unreserve(int seatNumber) {
        PQ.push(seatNumber);
    }
};

```

### 减小和重新排列数组后的最大元素

排序之后，如果两个相邻的数之间的差距大于1的话，就将后一个数削减到前一个数+1。
```cpp
class Solution {
public:
    int maximumElementAfterDecrementingAndRearranging(vector<int>& arr) {
        sort(arr.begin(), arr.end());
        int val = 1;
        arr[0] = 1;
        for(int i = 1; i < arr.size(); ++i){
            if(arr[i] - arr[i-1] > 1){
                arr[i] = arr[i-1] + 1;
            }
        }
        return arr.back();
    }
};

```

### 最近的房间


分别正向遍历和反向遍历所有的query，同时使用单调栈保存遍历当前房间之前的容量情况。如果是反向就是比当前的房间号大的房间容量。
在单调栈中二分查找每个询问的值。

时间复杂度为 $ \mathcal{O}(max(n, m) \times 2 \times \log n)$

```cpp
struct Query{
    int prefer;
    int minV;
    int lValue;
    int rValue;
    int originIndex;
};
struct Room{
    int id;
    int size;
    bool operator < (const Room& r) const{
        return id < r.id;
    }
};
struct SortByPrefer{

    bool operator () (const Query& a, const Query& b) const{
        if(a.prefer == b.prefer){
            return a.minV < b.minV;
        }
        return a.prefer < b.prefer;
    }
}sort1;
struct SortByIndex{
    bool operator() (const Query& a, const Query& b) const{
        return a.originIndex < b.originIndex;
    }
}sort2;
class Solution {
public:
    vector<int> closestRoom(vector<vector<int>>& rooms, vector<vector<int>>& queries) {
        vector<int> dec;
        unordered_map<int, int> mp;
        vector<Query> qVec;
        vector<Room> rVec;
        for(auto &it: rooms){
            rVec.push_back({it[0], it[1]});
        }
        sort(rVec.begin(), rVec.end());
        for(int i = 0; i < queries.size(); ++i){
            qVec.push_back({queries[i][0], queries[i][1], 0x3f3f3f3f, 0x3f3f3f3f, i});
        }
        sort(qVec.begin(), qVec.end(), sort1);
        int vecPos = 0;
        for(int i = 0; i < qVec.size(); ++i){
            while(vecPos < rVec.size() and rVec[vecPos].id <= qVec[i].prefer){
                auto &it = rVec[vecPos++];
                mp[it.size] = it.id;
                while(!dec.empty() and dec.back() <= it.size){
                    dec.pop_back();
                }
                dec.push_back(it.size);
            }
            auto pos = lower_bound(dec.rbegin(), dec.rend(), qVec[i].minV);
            if(pos != dec.rend()){
                    qVec[i].lValue = mp[*pos];
            }
        }
        dec.clear();
        vecPos = rVec.size() - 1;
        for(int i = qVec.size() - 1; i >= 0; --i){
            while(vecPos >= 0 and rVec[vecPos].id >= qVec[i].prefer){
                auto &it = rVec[vecPos--];
                mp[it.size] = it.id;
                while(!dec.empty() and dec.back() <= it.size){
                    dec.pop_back();
                }
                dec.push_back(it.size);
            }
            auto pos = lower_bound(dec.rbegin(), dec.rend(), qVec[i].minV);
            if(pos != dec.rend()){
                qVec[i].rValue = mp[*pos];
            }
        }
        
        sort(qVec.begin(), qVec.end(), sort2);
        vector<int> ret{};
        
        for(auto &it : qVec){
            if(it.lValue != 0x3f3f3f3f and it.rValue != 0x3f3f3f3f){
                if(it.prefer - it.lValue <= it.rValue - it.prefer){
                    ret.push_back(it.lValue);
                }else{
                    ret.push_back(it.rValue);
                }
            }else if(it.lValue != 0x3f3f3f3f){
                ret.push_back(it.lValue);
            }else if(it.rValue != 0x3f3f3f3f){
                ret.push_back(it.rValue);
            }else{
                ret.push_back(-1);
            }
        }
        return ret;
    }
};
```
