---

title: 线段树 - 算法 - huangfeihu.xyz
---
@import "/mystyle.less"

## 线段树

> 返回:house:[首页](../../index.html)，:rocket:[算法](../index.html)

---


```cpp
template<typename T>
class SegmentTree {
private:
    /**
     * @brief 建立一棵线段树的步骤
     * @param lPosQ 建树区间的左端点
     * @param rPosQ 建树区间的右端点
     * @param pos 当前根的编号
     * @param data 输入数据
     * @param shift 输入数据和建树的序号匹配
     * */
    void buildStep(int lPosQ, int rPosQ, std::vector<T>& data, int shift, int pos) {
        if (lPosQ == rPosQ) {
            node[pos] = data[lPosQ + shift];
            minVal[pos] = data[lPosQ + shift];
            maxVal[pos] = data[lPosQ + shift];
            return;
        }
        int mPosP = lPosQ + ((rPosQ - lPosQ) >> 1);
        buildStep(lPosQ, mPosP, data, shift, pos << 1);
        buildStep(mPosP + 1, rPosQ, data, shift, (pos << 1) | 1);
        node[pos] = node[pos << 1] + node[(pos << 1) | 1];
        minVal[pos] = min(minVal[pos << 1], minVal[(pos << 1) | 1]);
        maxVal[pos] = max(maxVal[pos << 1], maxVal[(pos << 1) | 1]);
    }

    /**
     * @brief 区间更新线段树
     * @param lPosQ 被修改的区间的左端点
     * @param rPosQ 被修改的区间的右端点
     * @param diff 被修改的区间元素的变化量
     * @param sPos 当前结点包含的区间的左端点
     * @param ePos 当前结点包含的区间的右端点
     * @param pos 当前结点的编号
     * */
    void updateStep(int lPosQ, int rPosQ, int diff, int sPos, int ePos, int pos) {
        if (lPosQ > ePos or rPosQ < sPos) return;
        if (lPosQ <= sPos and ePos <= rPosQ) {
            node[pos] += (ePos - sPos + 1) * diff;
            lazy[pos] += diff;
            minVal[pos] += diff;
            maxVal[pos] += diff;
            return;
        }
        int mPos = sPos + ((ePos - sPos) >> 1);
        lazyUpdate(pos, sPos, mPos, ePos);
        updateStep(lPosQ, rPosQ, diff, sPos, mPos, pos << 1);
        updateStep(lPosQ, rPosQ, diff, mPos + 1, ePos, (pos << 1) | 1);
        node[pos] = node[pos << 1] + node[(pos << 1) | 1];
        maxVal[pos] = max(maxVal[pos << 1], maxVal[(pos << 1) | 1]);
        minVal[pos] = min(minVal[pos << 1], minVal[(pos << 1) | 1]);
    }

    /**
     * @brief 区间求和的步骤
     * @param lPosQ 查询区间左端点
     * @param rPosQ 查询区间右端点
     * @param sPos 当前结点包含的区间的左端点
     * @param ePos 当前结点包含的区间的右端点
     * @param pos 当前结点的编号
     **/
    T querySumStep(int lPosQ, int rPosQ, int sPos, int ePos, int pos = 1) {
        if (lPosQ > ePos or rPosQ < sPos) return 0;
        if (lPosQ <= sPos and ePos <= rPosQ) return node[pos];
        int mPos = sPos + ((ePos - sPos) >> 1);
        lazyUpdate(pos, sPos, mPos, ePos);
        T sum = 0;
        sum = querySumStep(lPosQ, rPosQ, sPos, mPos, pos << 1);
        sum += querySumStep(lPosQ, rPosQ, mPos + 1, ePos, (pos << 1) | 1);
        return sum;
    }

    T queryMinStep(int lPosQ, int rPosQ, int sPos, int ePos, int pos = 1) {
        if (lPosQ > ePos or rPosQ < sPos) return MAX_VAL;
        if (lPosQ <= sPos and ePos <= rPosQ) return minVal[pos];
        int mPos = sPos + ((ePos - sPos) >> 1);
        lazyUpdate(pos, sPos, mPos, ePos);
        T ret = MAX_VAL;
        ret = min(ret, queryMinStep(lPosQ, rPosQ, sPos, mPos, pos << 1));
        ret = min(ret, queryMinStep(lPosQ, rPosQ, mPos + 1, ePos, (pos << 1) | 1));
        return ret;
    }

    T queryMaxStep(int lPosQ, int rPosQ, int sPos, int ePos, int pos = 1) {
        if (lPosQ > ePos or rPosQ < sPos) return MIN_VAL;
        if (lPosQ <= sPos and ePos <= rPosQ) return maxVal[pos];
        int mPos = sPos + ((ePos - sPos) >> 1);
        lazyUpdate(pos, sPos, mPos, ePos);
        T ret = MIN_VAL;
        ret = max(ret, queryMaxStep(lPosQ, rPosQ, sPos, mPos, pos << 1));
        ret = max(ret, queryMaxStep(lPosQ, rPosQ, mPos + 1, ePos, (pos << 1) | 1));
        return ret;
    }
    /**
     * @brief 懒更新
     * @param pos 当前区间节点
     * @param sPos 区间左端点
     * @param ePos 区间右端点
     * @param mPos 区间中间端点
     * */
    void lazyUpdate(int pos, int sPos, int mPos, int ePos) {
        if (lazy[pos] == 0) return;
        node[pos << 1] += lazy[pos] * (mPos - sPos + 1);
        maxVal[pos << 1] += lazy[pos];
        minVal[pos << 1] += lazy[pos];

        node[(pos << 1) | 1] += lazy[pos] * (ePos - mPos);
        minVal[(pos << 1) | 1] += lazy[pos];
        maxVal[(pos << 1) | 1] += lazy[pos];

        lazy[pos << 1] += lazy[pos];
        lazy[(pos << 1) | 1] += lazy[pos];
        lazy[pos] = 0;
    }


public:
    explicit SegmentTree(int n) :
        lPos(1), rPos(n), node(4 * n + 10), lazy(4 * n + 10), minVal(4 * n + 10), maxVal(4 * n + 10) {}

    /**
     * @brief 建立一棵线段树
     * @param data 输入数据
     * @param shift 调整偏移量使得数据和树匹配，因为树中的结点是从1开始计数
     * */
    void build(std::vector<T>& data, int shift) {
        buildStep(lPos, rPos, data, shift, 1);
    }

    /**
     * @brief 区间更新线段树
     * @param diff 被修改的区间元素的变化量
     * @param lPosQ 被修改的区间的左端点
     * @param rPosQ 被修改的区间的右端点
     *
     * */
    void update(int diff, int lPosQ, int rPosQ) {
        updateStep(lPosQ, rPosQ, diff, lPos, rPos, 1);
    }

    /**
     * @brief 区间求和
     * @param lPosQ 当前结点包含的区间的左端点
     * @param rPosQ 当前结点包含的区间的右端点
     *
     **/
    T querySum(int lPosQ, int rPosQ) {
        return querySumStep(lPosQ, rPosQ, lPos, rPos, 1);
    }

    /**
     * @brief 区间最小值查询
     * @param lPosQ 当前结点包含的区间的左端点
     * @param rPosQ 当前结点包含的区间的右端点
     **/
    T queryMin(int lPosQ, int rPosQ) {
        return queryMinStep(lPosQ, rPosQ, lPos, rPos, 1);
    }

    /**
     * @brief 区间最大值查询
     * @param lPosQ 当前结点包含的区间的左端点
     * @param rPosQ 当前结点包含的区间的右端点
     **/
    T queryMax(int lPosQ, int rPosQ) {
        return queryMaxStep(lPosQ, rPosQ, lPos, rPos, 1);
    }

private:
    int lPos, rPos;
    std::vector<T> node, lazy, minVal, maxVal;
    const static T MAX_VAL = INT_MAX;
    const static T MIN_VAL = INT_MIN;
};

```


### 参考例题

@import "../oj/CF_1555E.md"