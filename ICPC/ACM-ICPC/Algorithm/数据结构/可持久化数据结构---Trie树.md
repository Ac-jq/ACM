# 可持久化Trie树
![!\[!\\[!\\\[Alt text\\\](image.png)\\](../../../_resources/image.png)\](../../_resources/image.png)](../../../_resources/image.png)
## [最大异或和](https://www.acwing.com/problem/content/258/)
### 给定一个非负整数序列 a，初始长度为 N有 M个操作，有以下两种操作类型：

`A x：`添加操作，表示在序列末尾添加一个数 x，序列的长度 N增大 1。

`Q l r x：`询问操作，你需要找到一个位置p，满足$（l <= p <= r )$，使得：$a[p] xor a[p+1] xor … xor a[N] xor x$最大，输出这个最大值.
![Alt text](../../../_resources/%E6%9C%80%E5%A4%A7%E5%BC%82%E6%88%96%E5%92%8C.png)

![Alt text](../../../_resources/%E6%9C%80%E5%A4%A7%E5%BC%82%E6%88%96%E5%92%8C-%E6%95%B0%E6%8D%AE%E7%BB%93%E6%9E%84%E7%90%86%E8%A7%A3.png)

```
#include <iostream>
#include <cstring>
#include <algorithm>
#include <cstdio>

using namespace std;

const int N = 600010, M = N * 25, len = 23;
int s[N];
int tr[M][2];//每个数字最多需要25位
int root[N], idx, ver[M];//ver存储历史版本的根节点（也是边界）
int n, m;

//x为当前版本的根，y为上一个版本的根，i是第i个插入的数（版本号）（前i个数的前缀和）
//从高位到低位枚举，取出s[i]的第k位数字c，不是新节点就指向旧版本
//是新节点就添加，然后x走向新儿子，y同步对应走到儿子
void insert(int x, int y, int i) {//插入
    ver[x] = i;
    for (int k = len; k >= 0; k --) {
        int c = s[i] >> k & 1;
        tr[x][!c] = tr[y][!c];
        tr[x][c] = ++ idx;
        x = tr[x][c], y = tr[y][c];
        ver[x] = i;
    }
}
//x为r的根，L为搜索区的左边界，v为定值
//从高位到低位枚举，取出v的第k位数字c
//尽量走当前位的相反位，若不越界则走过去
//并且加上该位的贡献，若越界（空节点）
//则走当前位儿子，异或相同位0，没有贡献值
int query(int l, int r, int v) {//查询（这里的边界是版本）（几个数就有几个版本）
    int res = 0;
    int x = root[r];
    
    // 自带判空功能如果没有点, ver会为-1, 那就肯定不能满足>=l (根据初始化ver[0] = -1)
        // 如果没有这个初始化, ver[0] 默认为0, 而当l=0 的时候就会令到p误跳到空节点
        // 而ver又同时可以限制当前的点是在l r区间内
    for (int k = len; k >= 0; k -- ) {
        int c = v >> k & 1;
        if (ver[tr[x][!c]] >= l) {
            x = tr[x][!c], res += 1 << k;
        } else {
            x = tr[x][c];
        }
    }
    
    return res;
}


int main() {
    cin >> n >> m;
    
    
    //初始化
    
    
    root[0] = ++ idx ;//0号版本的根为1号点
    
    ver[0] = -1;//0号点的版本为-1，会越界
    
    insert(root[0], 0, 0);
    for (int i = 1; i <= n; i ++ ) {
        int x;
        cin >> x;
        s[i] = s[i - 1] ^ x;
        root[i] = ++ idx;
        insert(root[i], root[i - 1], i);
    }
    char op[2];
    int l, r, x;
    while (m--)
    {
        scanf("%s", op);
        if (op[0] == 'A')
        {
            scanf("%d", &x);
            n++;
            s[n] = s[n - 1] ^ x;
            root[n] = ++idx;
            insert(root[n], root[n - 1], n);
        }
        else
        {
            scanf("%d%d%d", &l, &r, &x);
            //从l，r找p使得s[p - 1] ^ s[n] ^ x 最大，就是在l-1,r-1中找p
            printf("%d\n", query(l - 1, r - 1, s[n] ^ x));
        }
    }
    return 0;

}
```
# 主席树
## [第k小数](https://www.acwing.com/problem/content/257/)
### 给定 n 个整数构成的序列 a，将对于指定的闭区间 $[l, r]$ 查询其区间内的第 k 小值。

### 分析：
一种可行的方案是：使用主席树。 主席树的主要思想就是：保存每次插入操作时的历史版本，以便查询区间第 k 小。

怎么保存呢？简单暴力一点，每次开一棵线段树呗。
那空间还不爆掉？

我们分析一下，发现每次修改操作修改的点的个数是一样的。
$（例如下图，修改了 [1,8] 中对应权值为 1 的结点，红色的点即为更改的点）$
![
!\[!\\[Alt text\\](../../../_resources/persistent-seg.png)\](../../_resources/persistent-seg.png)](../../../_resources/persistent-seg.png)


$只更改了 O(\log{n}) 个结点，形成一条链，也就是说每次更改的结点数 = 树的高度。
注意主席树不能使用堆式存储法，就是说不能用 x\times 2，x\times 2+1 来表示左右儿子，而是应该动态开点，并保存每个节点的左右儿子编号。$
$所以我们只要在记录左右儿子的基础上，保存插入每个数的时候的根节点就可以实现持久化了。$

$我们把问题简化一下：每次求 [1,r] 区间内的 k 小值。
怎么做呢？$$只需要找到插入 r 时的根节点版本，然后用普通权值线段树（有的叫键值线段树/值域线段树）做就行了。$

这个相信大家都能理解，回到原问题——求 [l,r] 区间 k 小值。
这里我们再联系另外一个知识：前缀和。
这个小东西巧妙运用了区间减法的性质，通过预处理从而达到 O(1) 回答每个询问。

$我们可以发现，主席树统计的信息也满足这个性质。
所以……如果需要得到 [l,r] 的统计信息，只需要用 [1,r] 的信息减去 [1,l - 1] 的信息就行了。$

$至此，该问题解决！$

$关于空间问题，我们分析一下：由于我们是动态开点的，所以一棵线段树只会出现 2n-1 个结点。$
$然后，有 n 次修改，每次至多增加 \lceil\log_2{n}\rceil+1 个结点。因此，最坏情况下 n 次修改后的结点总数会达到 2n-1+n(\lceil\log_2{n}\rceil+1)。 此题的 n \leq 10^5，单次修改至多增加 \lceil\log_2{10^5}\rceil+1 = 18 个结点，故 n 次修改后的结点总数为 2\times 10^5-1+18\times 10^5，忽略掉 -1，大概就是 20\times 10^5。$

最后给一个忠告：千万不要吝啬空间（大多数题目中空间限制都较为宽松，因此一般不用担心空间超限的问题）！大胆一点，直接上个 $2^5\times 10^5$，接近原空间的两倍（即 n << 5）。

## 实现
```
#include <cstdio>
#include <iostream>
#include <algorithm>
using namespace std;
const int N = 100005;
//d 为离散化数组
int n, m, len, a[N], d[N];

//T[i] 为 [1, i] 区间的权值线段树的根节点
int T[N], tot = 0;

//线段树的每个点
struct SegTree{
    int l, r, v;
}t[N * 20];

//建树
int build(int l, int r){
    int p = ++tot, mid = (l + r) >> 1;
    if(l < r) {
        t[p].l = build(l, mid);
        t[p].r = build(mid + 1, r);
    }
    t[p].v = 0; return p;
}

//增加一个数 pre 为上一个的根节点。
int update(int pre, int l, int r, int v){
    int p = ++tot, mid = (l + r) >> 1;
    t[p].l = t[pre].l, t[p].r = t[pre].r, t[p].v = t[pre].v + 1;
    if(l < r){
        //应该更新哪一个值域区间
        if(v <= mid) t[p].l = update(t[pre].l, l, mid, v);
        else t[p].r = update(t[pre].r, mid + 1, r, v); 
    }
    return p;
}

//查询
int query(int x, int y, int l, int r, int k){
    //找到了
    if(l == r) return l;
    //对位相减
    int sum = t[t[y].l].v - t[t[x].l].v, mid = (l + r) >> 1;
    if(k <= sum) return query(t[x].l, t[y].l, l, mid, k);
    else return query(t[x].r, t[y].r, mid + 1, r, k - sum);
}

int main(){
    scanf("%d%d", &n, &m);
    for (int i = 1; i <= n; i++)
        scanf("%d", a + i), d[i] = a[i];
    //离散化
    sort(d + 1, d + 1 + n);
    len = unique(d + 1, d + 1 + n) - (d + 1);
    for(int i = 1; i <= n; i++) 
        a[i] = lower_bound(d + 1, d + 1 + len, a[i]) - d;


    T[0] = build(1, len);
    for(int i = 1; i <= n; i++)
        T[i] = update(T[i - 1], 1, len, a[i]);

    //回答
    while(m--){
        int l, r, k; scanf("%d%d%d", &l, &r, &k);
        int ans = query(T[l - 1], T[r], 1, len, k);
        printf("%d\n", d[ans]);
    }
    return 0;
}
```
