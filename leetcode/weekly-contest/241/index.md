## 第 241 场周赛
> 返回[:house:首页](../../../index.html)
> 返回[:rocket:LeetCode目录](../../index.html)

---
@import "/mystyle.less"

[TOC]


### 找出所有子集的异或总和再求和

位操作找到所有的子序列，统计即可。

```cpp
class Solution {
public:
    int subsetXORSum(vector<int>& nums){
        int ret = 0;
        for(int i = 1; i < (1 << nums.size()); ++i){
            int tmp = 0;
            for(int j = 0; j < nums.size(); ++j){
                if(i & (1 << j)){                    
                    tmp ^= nums[j];               
                }            
            }            
            ret += tmp;        
        }
        return ret;    
    }
};
```

### 构成交替字符串需要的最小交换次数

注意字符串长度不为偶数的情况

```cpp
class Solution {
public:
    int minSwaps(string s) {
        int val[2] = {0, 0};
        int cnt = 0;
        for(int i = 0; i < s.length(); ++i){
            if(s[i] == '1'){
                val[i%2]++;
            }else{
                ++cnt;
            }
        }
        if(val[0] + val[1] == cnt){
            return min(val[0], val[1]);      
        }else if(val[0] + val[1] == cnt - 1){
            return val[0];
        }else if(val[0] + val[1] == cnt + 1){
            return val[1];
        }else{
            return -1;
        }
        
    }
};
```

### 找出和为指定值的下标对
用hash表保存`nums2`中的每个数，因为`nums1`的数据范围只有`1000`，遍历`nums1`中的每个数，统计`nums2`中`tot - nums1[i]`的数目。

```cpp
class FindSumPairs {
public:
    vector<int> n1, n2;
    unordered_map<int, int> mp;
    FindSumPairs(vector<int>& nums1, vector<int>& nums2): n1(nums1), n2(nums2) {
        for(auto it: nums2){
            if(mp.count(it)){
                mp[it]++;
            }else{
                mp[it] = 1;
            }
        }
    }
    void add(int index, int val) {
        mp[n2[index]]--;
        n2[index] += val;
        if(mp.count(n2[index])){
            mp[n2[index]]++;
        }else{
            mp[n2[index]] = 1;
        }
    }
    int count(int tot) {
        int ret = 0;
        for(auto it: n1){
            if(mp.count(tot - it)){
                ret += mp[tot - it];
            }
        }
        return ret;
    }
};
```

### 恰有 K 根木棍可以看到的排列数目