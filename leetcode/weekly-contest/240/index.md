## 第 240 场周赛
> 返回[:house:首页](../../../index.html)
> 返回[:rocket:LeetCode目录](../../index.html)

---

[TOC]

### 人口最多的年份
由于数据范围比较小，直接遍历年份判断
```cpp
class Solution {
public:
    int maximumPopulation(vector<vector<int>>& logs) {
        int ret = 0;
        int cnt = 0;
        for(int i = 1950; i < 2050; ++i){
            int tmp = 0;
            for(auto& it:logs){
                if(i >= it[0] and i < it[1]){
                    tmp ++;
                }
            }
            if(tmp > cnt){
                cnt = tmp;
                ret = i;
            }
        }
        return ret;
    }
};
```

### 下标对中的最大距离
遍历`nums1`，找每个节点对应的`nums2`的节点的最大值。可以使用一个数据标记已经找过`nums2`的节点的位置。这样时间复杂度就可以控制在$O(\cal{Max}(nums1.length, nums2.length))$
```cpp
class Solution {
public:
    int maxDistance(vector<int>& nums1, vector<int>& nums2) {
        int pos = 0;
        int ret = 0;
        for(int i = 0; i < nums1.size(); ++i){
            pos = max(pos, i);
            while(pos + 1 < nums2.size() and nums2[pos + 1] >= nums1[i]){
                ++pos;
            }
            ret = max(ret, pos - i);
        }
        return ret;
    }
};
```
### 子数组最小乘积的最大值
...

### 有向图中最大颜色值
> C++ 拓扑排序

首先整理所有的边的数据，将边`(i, j)`中的`j`存`modEdges[i]`中。意思就是`i`指向的边中，包含`j`节点。同时设置所有点的入度。

取入度为`0`的节点，更新当前节点指向的所有节点的入度和每个颜色的最大值。然后再将更新之后入度为`0`的节点放入到队列中。直到队列为空。

如果处理过的点的数目和总节点数目相同，则输出最大结果。如果处理过的点的数目和总节点数目不同，则存在环。

```cpp

class Solution {
public:
    int largestPathValue(string colors, vector<vector<int>>& edges) {
        /*
            pair:
                int: 当前节点的入度
                pair:
                    vector<int>: 当前节点的颜色值
                    vector<int>: 走到当前节点的每个颜色的最大值，不包含当前节点
        */
        vector<pair<int, pair<vector<int>, vector<int>>>> vec(colors.size(), {0, {vector<int>(26, 0), vector<int>(26, 0)}});
        vector<vector<int>> modEdges(colors.size(), vector<int>{});
        for(auto& it: edges){
            vec[it[1]].first++;
            modEdges[it[0]].push_back(it[1]);
        }
        for(int i = 0; i < colors.length(); ++i){
            vec[i].second.first[colors[i] - 'a'] = 1;
        }
        queue<int> Q;
        int cnt = 0;
        int ret = 0;
        // 检查入度为0的节点
        for(int i = 0; i < vec.size(); ++i){
            if(vec[i].first == 0){
                Q.push(i);
                ++cnt;
            }
        }
        while(!Q.empty()){
            int ptr = Q.front();
            for(int i = 0; i < 26; ++i){
                //更新结果
                ret = max(ret, vec[ptr].second.first[i] + vec[ptr].second.second[i]);    
            }
            Q.pop();
            for(auto it: modEdges[ptr]){
                // 更新指向点的入度
                vec[it].first --;
                // 更新指向点的颜色值
                for(int i = 0; i < 26; ++i){
                    vec[it].second.second[i] = max(vec[it].second.second[i], vec[ptr].second.first[i] + vec[ptr].second.second[i]);
                }
                // 将入度为0的节点加入到队列。
                if(vec[it].first == 0){
                    Q.push(it);
                    ++cnt;
                }
            }
        }
        if(cnt < colors.length()){
            return -1;
        }else{
            return ret;
        }
    }
};
```