---
title: 经典算法 - huangfeihu.xyz
---
@import "/mystyle.less"

# 经典算法 {ignore=True}
> 返回:house:[首页](./index.html)

-----------------------------------


# vector
## 自定义排序
```c++
#include<vector>
#include<algorithm>
using namespace std;

struct data{
    int x;
    int y;
    data(int _x,int _y){
        x=_x;y=_y;
    }
};

class CompLess{
public:
    bool operator() (const data& d1,const data& d2){
        return d1.x<d2.x;
    }
};

int main(){
    vector <data> vec;

    sort(vec.begin(),vec.end(),CompLess());

}
```

# 堆排序

## 基础数据类型申明
```c++ 
#include<iostream>
#include<queue>
using namespace std;

int main(){
    //小顶堆，top() 是最小值
    priority_queue<int,vector<int>,greater<int>> q;
    
    //大顶堆，top()是最大值
    priority_queue<int,vector<int>,less<int>> p;

    q.push(1);
    q.push(2);
    q.top();
    q.pop();
}
```

## 自定义类型数据
```c++ 
#include<iostream>
#include<queue>
using namespace std;

struct data{
    int x;
    int y;
}

struct tmp{
    bool operator() (data a,data b){
        if (a.x>b.x){
            return true;
        }else if (a.x==b.x){
            return false;
        }else{
            return a.y>b.y;
        }
    }
}

int main(){
    priority_queue<data,vector<data>,tmp> q;

    q.push(1);
    q.push(2);
    q.top();
    q.pop();
}
```