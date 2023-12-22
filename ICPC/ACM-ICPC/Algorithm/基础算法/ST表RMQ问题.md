# ST表RMQ问题

### RMQ:Range Maximum/Minimum Query 的缩写，即区间最值问题
对于长度为 n 的数列 A ，回答若干查询 $RMQ(A,i,j)(i,j<=n)$，
返回数列 A 中下标在 i,j 里的最大(小)值。
对于RMQ问题，最容易想到的解决方案是遍历，复杂度是O(n)。但当数据量非常大且查询很频繁时，该算法无法在有效的时间内查询出正解。

处理RMQ问题的算法有很多，ST算法 RMQ标准算法 线段树以及一些高级数据结构

当题目是离线的时侯使用ST算法更快，当题目是在线的时候直接使用线段树维护即可

在线和离线可以简单的理解为对于所有的操作是否需要读入完毕。

在线的要求是可以不用先知道所有的操作（类似询问、修改），边读入边执行，类似“走一步，做一步”的思想。

离线则与在线相反，要求必须知道所有的操作，类似"记录所有步，回头再做”的思想，一般用$Query[]$记录所有操作。

【在线可以理解为动态，离线理解为静态】
### ST（Sparse Table）算法是一个非常有名的在线处理RMQ问题的算法 【稀疏表】
所谓在线算法，是指用户每输入一个查询便马上处理一个查询。
该算法一般用较长的时间做预处理，待信息充足以后便可以用较少的时间回答每个查询。

所谓离线算法，是指在来了非常多的请求之后，一次性处理多个请求。
能够不依赖于预处理数据，却能一次完成多个请求的结果。
$ST算法实际上就是 DP + 位运算$

ST算法分成两部分：离线预处理 （nlogn）和 在线查询（O(1)）
## 给定一个序列，输出[l, r]中的最大值

![Alt text](../../../_resources/ST%E8%A1%A8%E9%A2%84%E5%A4%84%E7%90%86.png)

![Alt text](../../../_resources/ST%E8%A1%A8-%E8%AF%A2%E9%97%AE.png)

```
#include <iostream>
#include <algorithm>
#include <cmath>

using namespace std;

const int N = 2e5 + 10, M = 20;

int n, m;
int f[N][M];//以i为起点，的长度为2 ^ j 的区间

void init() {
    for (int i = 1; i <= n; i ++ ) cin >> f[i][0];//本身
    for (int j = 1; j < M; j ++ ) {//枚举区间长度指数，0已经输入
        for (int i = 1; i + (1 << j) - 1 <= n; i ++ ) {//枚举起点
            f[i][j] = max(f[i][j - 1], f[i + (1 << (j - 1))][j - 1]);//mid = 2^{j-1}
        }
    }
}

int query(int l, int r) {//找到一个k <= (r - l + 1) 的最大k，所查询区间由两个区间覆盖
    int k = log2(r - l + 1);
    return max(f[l][k], f[r - (1 << k) + 1][k]);
}

int main() {
    cin >> n;
    init();
    cin >> m;
    while (m -- ) {
        int l, r;
        cin >> l >> r;
        cout << query(l, r) << endl;
    }
    return 0;
}
```
