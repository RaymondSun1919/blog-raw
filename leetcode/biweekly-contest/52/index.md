---
title: 第52场双周赛 - dianhsu.top
---


## 第52场双周赛
> 返回[:house:首页](../../../index.html)
> 返回[:rocket:LeetCode目录](../../index.html)

---
@import "/mystyle.less"

[TOC]

### 将句子排序

这个题首先拆分单词，因为单词是从1开始索引，所以我们可以用 `word.back() - '1'`获得从零开始的索引。

用桶排序的思想，将单词排序好。

用`word.substr(0, word.size() - 1)`处理掉单词之后末尾的数字。

然后将单词按顺序拼接起来。


```cpp
class Solution {
public:
    string sortSentence(string s) {
        stringstream ss(s);
        vector<string> vec(10);
        string word;
        int cnt = 0;
        while(ss >> word){
            cnt++;
            vec[word.back() - '1'] = word.substr(0, word.size() - 1);
        }
        string ret;
        for(int i = 0; i < cnt; ++i){
            if(i){
                ret += ' ';
            }
            ret += vec[i];
        }
        return ret;
    }
};
```

### 增长的内存泄露

模拟一下内存分配操作。

```cpp
class Solution {
public:
    vector<int> memLeak(int memory1, int memory2) {
        int idx = 1;
        vector<int> ret;
        while(1){
            if(memory1 >= idx or memory2 >= idx){
                if(memory1 >= memory2){
                    memory1 -= idx;
                }else{
                    memory2 -= idx;
                }
            }else{
                ret.push_back(idx);
                ret.push_back(memory1);
                ret.push_back(memory2);
                break;
            }
            ++idx;
        }
        return ret;
    }
};

```

### 旋转盒子

首先旋转一下图形。
旋转图形可以参照这个题[48. 旋转图像](https://leetcode-cn.com/problems/rotate-image/)
通常的做法是先转置，然后在左右翻转。

然后重力掉落。
这个可以考虑每一列单独考虑，从列的最底端向上遍历，找到一个空位之后，再往上找石头。
如果找到了石头，交换石头和空位。
如果找到了障碍物，就跳出循环，往上找一个空格，在进行上一个步骤。


```cpp
class Solution {
public:
    vector<vector<char>> rotateTheBox(vector<vector<char>>& box) {
        vector<vector<char>> ret(box[0].size(), vector<char>(box.size(), '.'));
        // 先旋转90°
        for(int i = 0; i < box.size(); ++i){
            for(int j = 0; j < box[0].size(); ++j){
                ret[j][box.size() - 1 - i] = box[i][j];
            }
        }
        // 然后重力掉落
        for(int j = 0; j < ret[0].size(); ++j){
            for(int i = ret.size() - 1; i > 0; --i){
                if(ret[i][j] == '.'){
                    for(int k = i - 1; k >= 0; --k){
                        if(ret[k][j] == '*'){
                            break;
                        }else if(ret[k][j] == '#'){
                            ret[i][j] = '#';
                            ret[k][j] = '.';
                            break;
                        }
                    }
                }
            }
        }
        return ret;
    }
};

```

### 向下取整数对和

这个题如果直接每个数都和数组中的数计算一次，循环量接近10的10次方，肯定不行。

首先我们需要知道，每个数只需要当不小于它的数的被除数来取模。因为比它小的数，除以它肯定是等于0的。

其次，对于每个数组中的数`x`，数组中其他的数除以这个数的整数部分是在1到1e5之间的。
如果已经找到数组最大的数`maxV`，那么数组中的其他数除以`x`的整数部分是在0到`maxV/x`之间的。

这样的话，我们可以统计小于或等于`i`的数的数目，代码里面用`sum[i]`来表示。
`arr[i]`是代表数字`i`出现的次数。

那么，数组中除以数字`i`的整数部分是`j(1 <= j <= maxV/x)`的数的数量是`sum[(j + 1) * i - 1] - sum[j * i - 1]`，然后把这部分加起来就可以了。

```cpp
class Solution {
public:
    int sumOfFlooredPairs(vector<int>& nums) {
        vector<long long> arr(200010, 0);
        vector<long long> sum(200010, 0);
        int maxV = 0;
        long long ret = 0;
        for(auto it: nums){
            maxV = max(maxV, it);
            arr[it] ++;
        }
        for(int i = 1; i <= maxV * 3; ++i){
            sum[i] = sum[i - 1] + arr[i];
        }
        for(int i = 1; i <= maxV; ++i){
            if(arr[i] > 0){
                for(long long j = 1; j * i <= maxV; ++j){
                    // 后面那部分意义是：数字i的出现次数 乘以 整数部分j 乘以 （数组中除以i的整数部分是j的数字的数量）
                    ret = (ret + (sum[(j + 1) * i - 1] - sum[j * i - 1]) * j * arr[i]) % 1000000007;
                }
            }
            
        }
        return ret;
    }
};
```