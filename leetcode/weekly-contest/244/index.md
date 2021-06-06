---
title: 第244场周赛 - dianhsu.top
---
@import "/mystyle.less"

## 第 244 场周赛 {ignore=True}
> 返回[:house:首页](../../../index.html)，[:rocket:LeetCode目录](../../index.html)

---

[TOC]


### 判断矩阵经轮转后是否一致

#### 简要思路
关于矩阵的旋转，在LeetCode上面有很多道类似的题目 [https://leetcode-cn.com/problems/rotate-image/](https://leetcode-cn.com/problems/rotate-image/)

顺时针旋转，通常用的方法是先转置，然后再左右镜像。

#### 参考代码

```cpp
class Solution {
public:
    bool isEqual(vector<vector<int>>& mat, vector<vector<int>>& target){
        for(int i = 0; i < mat.size(); ++i){
            for(int j = 0; j < mat.size(); ++j){
                if(mat[i][j] != target[i][j]){
                    return false;
                }
            }
        }
        return true;
    }
    bool findRotation(vector<vector<int>>& mat, vector<vector<int>>& target) {
        for(int kase = 0; kase < 4; ++kase){
            if(isEqual(mat, target)){
                return true;
            }
            // 转置
            for(int i = 0; i < mat.size(); ++i){
                for(int j = i; j < mat.size(); ++j){
                    int c = mat[i][j];
                    mat[i][j] = mat[j][i];
                    mat[j][i] = c;
                }
            }
            // 左右镜像
            for(int i = 0; i < mat.size(); ++i){
                for(int j = 0; j < mat.size()/2; ++j){
                    int c = mat[i][mat.size() - 1 - j];
                    mat[i][mat.size() - 1 - j] = mat[i][j];
                    mat[i][j] = c;
                }
            }
        }
        return false;
    }
};
```
### 使数组元素相等的减少操作次数

#### 简要思路

分析一下得出一个结论，就是某个数需要的操作次数等于小于它的不同的数的个数

| 原始数据 | 1   | 1   | 2   | 2   | 3   |
| -------- | --- | --- | --- | --- | --- |
| 操作次数 | 0   | 0   | 1   | 1   | 2   |

这样的话，我们只需要排序一下，然后统计下比当前数小的不同的数的个数就可以了

#### 参考代码
```cpp
class Solution {
public:
    int reductionOperations(vector<int>& nums) {
        sort(nums.begin(), nums.end());
        int cur = 0;
        int ans = 0;
        for(int i = 1; i < nums.size(); ++i){
            if(nums[i] != nums[i-1]){
                ++cur;
            }
            ans += cur;
        }
        return ans;
    }
};
```
### 使二进制字符串字符交替的最少反转次数

#### 简要思路

模拟这个操作1，将字符串头部的字符移动到末尾，然后统计出当前需要操作的次数。

移动字符可以考虑用两个相同的字符串拼接起来，然后取中间一段和原本长度相同的字符串，如果它的位置相对于原本的位置向右移动了 $n$ 位，那么就相当于把前面 $n$ 个字符挪到了后面。

统计当前字符串需要进行操作2的次数，初始化的时候，将开头为 $1$ 和开头为 $0$ 的情况都计算一遍。
然后在考虑进行操作1之后，头部的字符挪到了末尾，这样就需要更新头部为 $1$ 和头部为 $0$ 的情况，然后比较这样的操作2的进行次数就可以了。

时间复杂度是 $O(n)$。


#### 参考代码

```cpp
class Solution {
public:

    int minFlips(string s) {
        auto tmp = s + s;
        int arr[2] = {0, 0};
        for(int i = 0; i < s.length(); ++i){
            if(s[i] - '0' == (i & 1)){
                arr[0] ++;
            }else{
                arr[1] ++;
            }
        }
        int ans = min((int)s.length() - arr[0], (int)s.length() - arr[1]);
        for(int i = s.length(); i < tmp.length(); ++i){
            if(tmp[i] - '0' == (i & 1)){
                arr[0] ++;
            }else{
                arr[1] ++;
            }
            if(s[i - (int)s.length()] - '0' == ((i - (int)s.length()) & 1)){
                arr[0] --;
            }else{
                arr[1] --;
            }
            ans = min(ans, min((int)s.length() - arr[0], (int)s.length() - arr[1]));
        }
        return ans;
    }
};
```

### 装包裹的最小浪费空间


#### 简要思路

对于每个供应商提供的箱子，每个包裹需要选择不小于包裹尺寸的最小的箱子。

用示例3的供应商3举例：

| packages | 3   | 5   | 8   | 10  | 11  | 12  |
| -------- | --- | --- | --- | --- | --- | --- |
| box      | 5   | 5   | 10  | 10  | 14  | 14  |

这里考虑二分查找，对于每个供应商提供的一组箱子，从小到大遍历箱子的尺寸，然后尽可能放入更多的包裹。如果包裹已经被之前的箱子放入了，那么就不在考虑放到当前箱子里面。

C++的STL库中有个`upper_bound`，就是给你一个值，找到大于当前值的第一个位置。如果这个值是箱子尺寸，那么找到的位置就是当前箱子放不下的第一个包裹的位置。

这里还需要用前缀和来处理一下包裹的容量和，方便计算

一些情况的处理：
如果供应商提供的箱子中的最大值小于最大包裹的尺寸，就是无法将包裹放进箱子里面。

时间复杂度： $O(boxes \times log_2(packages))$
#### 参考代码

```cpp
class Solution {
public:
    int minWastedSpace(vector<int>& packages, vector<vector<int>>& boxes) {
        // 排序包裹
        sort(packages.begin(), packages.end());

        // 包裹容量和前缀
        vector<long long> prefix{0};
        for(auto it: packages){
            prefix.push_back(prefix.back() + it);
        }
        int maxBox = 0;
        long long ret = LONG_LONG_MAX;
        // 遍历每一个供应商
        for(auto& box: boxes){
            // 排序箱子
            sort(box.begin(), box.end());
            if(box.back() < packages.back()){
                continue;
            }
            long long tmp = 0;
            long long prev = 0;
            // 遍历供应商的每一个箱子
            for(int i = 0; i < box.size(); ++i){
                // 找到箱子放不下的包裹的第一个位置
                auto pos = upper_bound(packages.begin(), packages.end(), box[i]) - packages.begin();
                // 用当前箱子的总尺寸 减去 当前箱子放下的包裹的总尺寸
                tmp += 1ll * (pos - prev) * box[i] - 1ll * (prefix[pos] - prefix[prev]); 
                prev = pos;
            }
            ret = min(ret, tmp);
        }
        if(ret == LONG_LONG_MAX){
            return -1;
        }else{
            return ret % 1000000007;
        }
    }
};
```