---
title: 第246场周赛 - dianhsu.top
---
@import "/mystyle.less"

## 第 246 场周赛 {ignore=True}
> 返回[:house:首页](../../../index.html)，[:rocket:LeetCode目录](../../index.html)

---

[TOC]

### 字符串中的最大奇数
#### 简要思路

题目已经给出了这个字符串不含前导0，只需要找到这个字符串中最后一个奇数的位置就可以了。这样就能保证奇数是最长的一个奇数，从而保证了这个奇数最大。

#### 实例代码
```cpp
class Solution {
public:
    string largestOddNumber(string num) {
        int pos = -1;
        for(int i = 0; i < num.length(); ++i){
            if((num[i] - '0') % 2){
                pos = i;
            }
        }
        if(pos == -1){
            return "";
        }else{
            return num.substr(0, pos + 1);
        }
    }
};
```

### 你完成的完整对局数
#### 简要思路
1个小时内有4个15分钟，按0点开始算起，计算当前在第几个15分钟的位置。开始时间需要向上取整，结束时间向下取整。另外注意通宵的情况，需要用取整之前的时间计算。
#### 参考代码
```cpp
class Solution {
public:
    int cal(string& t){
        int ret =ret = ((t[0] - '0') * 10 + t[1] - '0') * 60 + (t[3] - '0') * 10 + t[4] - '0';
        return ret;
    }
    int numberOfRounds(string startTime, string finishTime) {
        int t1 = (cal(startTime)+14) / 15;
        int t2 = cal(finishTime)/15;
        if(cal(finishTime) < cal(startTime)){
            t2 += 24 * 4;
        }
        return max(0, t2 - t1);
    }
};
```

### 统计子岛屿

#### 简要思路
经典BFS问题，遍历grid2中的每一块岛屿，判断这个岛屿上的土地是不是grid1上面的岛屿的土地完全覆盖。遍历每个岛屿上面的土地的时候，需要把访问过的土地设一个访问过的标记，避免重复访问。另外注意一下不要访问出这个地图的边界就可以了。
#### 参考代码
```cpp
int dx[] = {1, -1, 0, 0};
int dy[] = {0, 0, 1, -1};
queue<pair<int, int>> Q;
class Solution {
public:
    void bfs(bool& valid, int x, int y, vector<vector<int>>& grid1, vector<vector<int>>& grid2, vector<vector<int>>& vis){
        Q.push({x, y});
        while(!Q.empty()){
            auto it = Q.front();
            Q.pop();
            for(int i = 0; i < 4; ++i){
                int nx = it.first + dx[i];
                int ny = it.second + dy[i];
                if(nx >= 0 and nx < grid1.size() and ny >= 0 and ny < grid1[0].size() and grid2[nx][ny] == 1 and vis[nx][ny] == 0){
                    vis[nx][ny] = 1;
                    if(grid1[nx][ny] == 0) valid = false;
                    Q.push({nx, ny});
                }
            }
        }
    }
    int countSubIslands(vector<vector<int>>& grid1, vector<vector<int>>& grid2) {
        vector<vector<int>> vis(grid1.size(), vector<int>(grid1[0].size(), 0));
        int ret = 0;
        for(int i = 0; i < grid1.size(); ++i){
            for(int j = 0; j < grid1[0].size(); ++j){
                if(grid2[i][j] == 1 and vis[i][j] == false){
                    bool valid = true;
                    if(grid1[i][j] == 0) valid = false;
                    bfs(valid, i, j, grid1, grid2, vis);
                    vis[i][j] = 1;
                    if(valid){
                        ++ret;
                    }
                }
            }
        }
        return ret;
    }
};
```

### 查询差绝对值的最小值

#### 简要思路
这个题看到queries的区间重叠的可能性很大，考虑离线处理queries。
离线处理的意思是现将queries的数据按照某种序列排序之后，在进行处理，这样通过最小化相邻query的查询区间的差异，使得每次查询的时候，只需要修改少量的数据，便可以得到查询的结果。排序的时候，按照左区间从小到大，然后右区间从小达到的顺序排序。

统计结果的时候，因为`nums`中的数据在 $1$ 到 $100$ 之间，只需要遍历一遍这个里面的数据即可。如果只找到一个数据，那么就说明这个区间内的数据都相同。返回`-1`。

#### 参考代码
```cpp
struct Node{
    int l, r;
    int idx;
    bool operator < (const Node& b){
        if(l != b.l){
            return l < b.l;    
        }
        return r < b.r;
    }
};

class Solution {
public:
    vector<int> minDifference(vector<int>& nums, vector<vector<int>>& queries) {
        // 存放的是区间内nums[i]出现的次数
        int arr[102];
        memset(arr, 0, sizeof arr);
        // 离线处理用的排序列表
        vector<Node> vec;
        for(int i = 0; i < queries.size(); ++i){
            vec.push_back({queries[i][0], queries[i][1], i});
        }
        sort(vec.begin(), vec.end());
        // 最终求出的queries对应的结果
        vector<int> ret(queries.size(), -1);
        for(int i = 0; i < vec.size(); ++i){
            if(i == 0){
                // 这个区间之前没有
                for(int j = vec[i].l; j <= vec[i].r; ++j){
                    arr[nums[j]]++;
                }
            }else{
                // 新的区间左边大于旧区间左边的部分
                for(int j = vec[i-1].l; j < vec[i].l; ++j){
                    arr[nums[j]]--;
                }
                // 如果新区间右边大于或者等于旧区间的右边
                if(vec[i].r >= vec[i-1].r){
                    // 添加一些元素
                    for(int j = vec[i-1].r + 1; j <= vec[i].r; ++j){
                        arr[nums[j]]++;
                    }
                }else{
                    // 删除一些元素
                    for(int j = vec[i].r + 1; j <= vec[i-1].r; ++j){
                        arr[nums[j]]--;
                    }
                }
            }
            int pre = -220;
            int tmp = 20000;
            for(int i = 1; i <= 100; ++i){
                if(arr[i] > 0){
                    tmp = min(i - pre, tmp);
                    pre = i;
                }
            }
            // 如果统计的区间内元素只有一个
            if(tmp <= 100){
                ret[vec[i].idx] = tmp;    
            }
        }
        return ret;
    }
};
```