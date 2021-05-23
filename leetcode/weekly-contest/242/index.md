## 第 242 场周赛
> 返回[:house:首页](../../../index.html)
> 返回[:rocket:LeetCode目录](../../index.html)


---

[TOC]

### 哪种连续子字符串更长

数据比较小，我们可以用`stringstream`分割字符串

分割的原理是这样，比如我们要统计1的连续字符的最大长度。对于这样一个字符串`101010010101`，我们使用空格来替换字符串中的`0`，就得到了一个`1 1 1 1 1 1`这样的字符串。用流读入，然后依次读出到`string`类型中，直接比较字符串的长度就可以了
```cpp
class Solution {
public:
    bool checkZeroOnes(string s) {
        string zeros = s;
        for(auto& it:zeros){
            if(it == '1'){
                it = ' ';
            }
        }
        string ones = s;
        for(auto& it:ones){
            if(it == '0'){
                it = ' ';
            }
        }
        int cntZeros = 0;
        int cntOnes = 0;
        stringstream ssz(zeros), sso(ones);
        string tmp;
        while(ssz >> tmp){
            cntZeros = max(cntZeros, (int)tmp.length());
        }
        while(sso >> tmp){
            cntOnes = max(cntOnes, (int)tmp.length());
        }
        return cntOnes > cntZeros;
    }
};
```

### 准时到达的列车最小时速

二分查找一下，判断每个速度是否满足条件就可以了

注意几个细节：
- 最后一趟列车的运行时间不需要向上取整。
- 因为时间有两位小数，所以在考虑二分上限的时候，要设定二分上限至少为$Max(dist_i) \times 100$。

```cpp
class Solution {
public:
    double check(double speed, vector<int>& dist){
        double ret = 0;
        for(int i = 0; i < dist.size(); ++i){
            if(i != dist.size() - 1){
                ret += ceil((double)dist[i]/speed);
            }else{
                ret += (double)dist[i]/speed;
            }
            
        }
        return ret;
    }
    int minSpeedOnTime(vector<int>& dist, double hour) {
        if((double)dist.size() - 1 > hour){
            return -1;
        }
        int low = 1;
        int high = 0;
        for(auto& it: dist){
            high = max(high, it);
        }
        high *= 1000;
        while(low < high){
            int mid = low + (high - low) / 2;
            if(check((double)mid, dist) <= hour){
                high = mid;
            }else{
                low = mid + 1;
            }
        }
        return low;
    }
};
```

### 跳跃游戏 VII

宽度优先搜索（BFS）是一个很容易想到的算法。但是数据范围比较大，要考虑剪枝才能在时间和空间上满足要求。

我这里设定了一个参数`pos`，代表目前访问到的最远的字符串的位置，当然包不包含`pos`这个点都没关系。BFS拓展的时候，不访问低于`pos`的位置，并且实时更新`pos`的位置。在这样的剪枝方法下，对`string s`的访问几乎是每个字符都只访问了一次。

```cpp
class Solution {
public:
    bool canReach(string s, int minJump, int maxJump) {
        if(s.back() == '1') return false;
        unordered_set<int> V; // 存放已经放入到队列之中的位置
        queue<int> Q;
        int pos = 0; // 剪枝用的，访问到的最大位置
        Q.push(0);
        V.insert(0);
        while(!Q.empty()){
            auto it = Q.front();
            Q.pop();
            for(int i = max(it + minJump, pos); i <= min(it + maxJump, (int)s.length() - 1); ++i){
                pos = i;
                if(s[i] == '0' and V.count(i) == 0){
                    Q.push(i);
                    V.insert(i);
                }
            }
        }
        return V.count(s.length() - 1) > 0;
    }
};
```

### 石子游戏 VIII

看到 Alice 和 Bob，就知道是博弈问题。:cry:

仔细考虑下，实际上每个人所取的石子的和是前缀和

然后提炼下问题的核心：
- Alice和Bob都是想在序列中取到最大的前缀和
- Alice先手，Bob后手
- 两人依次进行，当前取到的前缀和的位置总大于上一次取前缀和的位置

| name      | 0   | 1   | 2   | 3   | 4   | 5   | 6   |
| :-------- | --- | --- | --- | --- | --- | --- | --- |
| stones    | 7   | -6  | 5   | 10  | 5   | -2  | -6  |
| prefixSum | 1   | 6   | 16  | 21  | 19  | 13  | -   |
```cpp
prefixSum[i] = prefixSum[i - 1] + stones[i + 1]
```
这里前缀和是按照这样的方式存储的，没有存放0位置的前缀和是因为取石子的时候最少要取两个，所以不用考虑0这个位置。


我们假定Alice取第`i`号位置的话，Bob就会从`i + 1`开始的位置里面选择差值最大的位置。这里用`ret`保存最后的结果
```cpp
ret = max(ret, prefixSum[i] - *suffix.rbegin());
suffix.insert(prefixSum[i] - *suffix.rbegin());
```
| name      | 0   | 1   | 2   | 3   | 4   | 5   | 6   |
| :-------- | --- | --- | --- | --- | --- | --- | --- |
| stones    | 7   | -6  | 5   | 10  | 5   | -2  | -6  |
| prefixSum | 1   | 6   | 16  | 21  | 19  | 13  | -   |
| ret       | -12 | -7  | 3   | 8   | 6   | 13  | -   |

```cpp
class Solution {
public:
    int stoneGameVIII(vector<int>& stones) {
        // 统计前缀和
        vector<int> prefixSum(stones.size(), 0);
        
        prefixSum[0] = stones[0] + stones[1];
        for(int i = 1; i < stones.size() - 1; ++i){
            prefixSum[i] = prefixSum[i - 1] + stones[i + 1];
        }
        set<int> suffix;
        int ret = prefixSum[stones.size() - 2];
        suffix.insert(prefixSum[stones.size() - 2]);

        for(int i = stones.size() - 3; i >= 0; --i){
            ret = max(ret, prefixSum[i] - *suffix.rbegin());
            suffix.insert(prefixSum[i] - *suffix.rbegin());
        }
        return ret;
    }
};
```
