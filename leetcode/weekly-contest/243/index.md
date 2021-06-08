---
title: 第243场周赛 - dianhsu.top
---
@import "/mystyle.less"

## 第 243 场周赛 {ignore=True}
> 返回[:house:首页](../../../index.html)，[:rocket:LeetCode目录](../../index.html)

---

[TOC]

### 检查某单词是否等于两单词之和
转化成数字之后，比较相加之和
```cpp
class Solution {
public:
    bool isSumEqual(string firstWord, string secondWord, string targetWord) {
        int firstVal = 0;
        for(auto ch: firstWord){
            firstVal = firstVal * 10 + (ch - 'a');
        }
        int secondVal = 0;
        for(auto ch: secondWord){
            secondVal = secondVal * 10 + (ch - 'a');
        }
        int thirdVal = 0;
        for(auto ch: targetWord){
            thirdVal = thirdVal * 10 + (ch - 'a');
        }
        return firstVal + secondVal == thirdVal;
    }
};
```

### 插入后的最大值

- 如果是正整数，从前往后找到第一个字符数字小于x的位置，在之前插入x的字符；找不到就插入到末尾。
- 如果是负整数，从前往后找到第一个字符数字大于x的位置，在之前插入x的字符；找不到就插入到末尾。


```cpp
class Solution {
public:
    string maxValue(string n, int x) {
        bool inserted = false;
        string ret = "";
        char ch = (char)(x + '0');
        if(n[0] == '-'){
            ret += n[0];
            for(int i = 1; i < n.size(); ++i){
                if(n[i] > ch){
                    if(inserted == false){
                        ret += ch;
                        inserted = true;
                    }
                }
                ret += n[i];
            }
            if(inserted == false){
                ret += ch;
            }
        }else{
            for(int i = 0; i < n.size(); ++i){
                if(n[i] < ch){
                    if(inserted == false){
                        ret += ch;
                        inserted = true;
                    }
                }
                ret += n[i];
            }
            if(inserted == false){
                ret += ch;
            }
        }
        return ret;
    }
};
```

### 使用服务器处理任务

模拟服务器操作过程，用优先队列动态排序，选择需要的服务器和最先完成的任务。

```cpp
class Cmp{
public:
    bool operator () (const pair<int, int>& a, const pair<int, int>& b) const {
        if(a.first == b.first){
            return a.second > b.second;
        }
        return a.first > b.first;
    }  
};
struct Processor{
    pair<int, int> p;
    int endTime;
    bool operator < (const Processor& a) const{
        return endTime > a.endTime;
    }
};
class Solution {
public:
    vector<int> assignTasks(vector<int>& servers, vector<int>& tasks) {
        // 空闲服务器队列
        priority_queue<pair<int, int>, vector<pair<int, int>>, Cmp> PQ;
        for(int i = 0; i < servers.size(); ++i){
            PQ.push({servers[i], i});
        }
        // 任务队列
        priority_queue<Processor> Q;
        vector<int> ret;
        int cur = 0;
        int idx = 0;
        while(idx < tasks.size()){
            while(!Q.empty() and Q.top().endTime <= cur){
                PQ.push(Q.top().p);
                Q.pop();
            }
            if(!PQ.empty()){
                ret.push_back(PQ.top().second);
                Q.push({PQ.top(), cur + tasks[idx++]});
                cur = max(cur, idx);
                PQ.pop();
            }else{
                cur = max(Q.top().endTime, cur);
            }
        }
        return ret;
    }
};
```
### 准时抵达会议现场的最小跳过休息次数