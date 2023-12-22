# 树形DP
## 树形 DP，即在树上进行的 DP。由于树固有的递归性质，树形 DP 一般都是递归进行的。
## 引入
## 没有上司的舞会
$Ural 大学有 N名职员，编号为 1∼N。$

他们的关系就像一棵以校长为根的树，父节点就是子节点的直接上司。

$每个职员有一个快乐指数，用整数 Hi给出，其中 1≤i≤N$

现在要召开一场周年庆宴会，不过，没有职员愿意和直接上司一起参会。

在满足这个条件的前提下，主办方希望邀请一部分职员参会，使得所有参会职员的快乐指数总和最大，求这个最大值。
```
#include<iostream>
#include<cstring>
#include<algorithm>

using namespace std;

const int N = 6010;
int n, f[N][2], w[N];//f：分成以i为根，选根的树和不选根的树
int h[N], e[N], ne[N], idx;
bool st[N];//看是否有上司

int add(int a, int b) {
    e[idx] = b, ne[idx] = h[a], h[a] = idx ++ ;
}

void dfs(int u) {
    f[u][0] = 0;
    f[u][1] = w[u];
    for(int i = h[u]; i != -1; i = ne[i]) {
        int j = e[i];
        dfs(j);//先dfs子节点，把f子孙更新完后才可以更新f根
        f[u][0] += max(f[j][0], f[j][1]);
        f[u][1] += f[j][0];
    }
}

int main() {
    memset(h, -1, sizeof(h));
    cin >> n;
    for(int i = 1; i <= n; i ++ ) cin >> w[i];
    
    for(int i = 1; i < n; i ++ ) {
        int a, b;
        cin >> a >> b;
        add(b, a);
        st[a] = true;
    }
    
    int root = 1; 
    while(st[root]) root ++ ;
    
    dfs(root);
    
    cout << max(f[root][0], f[root][1]) << endl;
    return 0;
}
```
# 换根 DP
树形 DP 中的换根 DP 问题又被称为二次扫描，通常不会指定根结点，并且根结点的变化会对一些值，例如子结点深度和、点权和等产生影响。

通常需要两次 DFS，第一次 DFS 预处理诸如深度，点权和之类的信息，在第二次 DFS 开始运行换根动态规划。

### 换根DP 一般分为三个步骤
1. 指定任意一个根节点
2. 一次dfs遍历，统计出当前子树内的节点对当前节点的贡献
3. 一次dfs遍历，统计出当前节点的父节点对当前节点的贡献，然后合并统计答案

## 树的中心
### 给定一棵树，树中包含 n个结点（编号1~n）和 n−1条无向边，每条边都有一个权值。请你在树中找到一个点，使得该点到树中其他结点的最远距离最近。

- 那么我们就要先 dfs 一遍，预处理出当前子树对于根的最大贡献（距离）和 次大贡献（距离）

### 处理 次大贡献（距离） 的原因是：

如果 当前节点 是其 父节点子树 的 最大路径 上的点，则 父节点子树 的 最大贡献 不能算作对该节点的贡献

因为我们的路径是 简单路径，不能 走回头路

然后我们再 dfs 一遍，求解出每个节点的父节点对他的贡献（即每个节点往上能到的最远路径

两者比较，取一个 max 即可

```
#include <iostream>
#include <cstring>
#include <algorithm>

using namespace std;

const int N = 10010, M = N << 1, INF = 0x3f3f3f3f;

int n;
int h[N], e[M], w[M], ne[M], idx;
int d1[N], d2[N], up[N];
int s[N];

void add(int a, int b, int c) {
    e[idx] = b, w[idx] = c, ne[idx] = h[a], h[a] = idx ++ ;
}
//统计出当前子树内的节点对当前节点的贡献
void dfs1(int u, int father) {
    // d1[u] = d2[u] = -INF;  //这题所有边权都是正的，可以不用初始化为负无穷
    for (int i = h[u]; ~i; i = ne[i]) {
        int j = e[i];
        if (j == father) continue;
        dfs1(j, u);
        if (d1[j] + w[i] >= d1[u]) {
            d2[u] = d1[u];
            d1[u] = d1[j] + w[i];
            s[u] = j;//u的儿子是j（在以u为根的子树的路径中最长路走j的，为了up的时候选择另一条路）
        }
        else if (d1[j] + w[i] > d2[u]) {
            d2[u] = d1[j] + w[i];
        }
    }
    // if (d1[u] == -INF) d1[u] = d2[u] = 0; //特判叶子结点
}

void dfs2(int u, int father) {
    for (int i = h[u]; ~i; i = ne[i]) {
        int j = e[i];
        if (j == father) continue;
        if (s[u] == j) up[j] = w[i] + max(up[u], d2[u]);   //son_u  = j，则用次大更新
        else up[j] = w[i] + max(up[u], d1[u]);              //son_u != j，则用最大更新
        dfs2(j, u);
    }
}
int main() {
    memset(h, -1, sizeof h);
    scanf("%d", &n);
    for (int i = 1; i < n; i ++ ) {
        int a, b, c;
        scanf("%d%d%d", &a, &b, &c);
        add(a, b, c), add(b, a, c);
    }

    dfs1(1, -1);
    dfs2(1, -1);

    int res = INF;
    for (int i = 1; i <= n; i ++ ) res = min(res, max(d1[i], up[i]));
    printf("%d\n", res);

    return 0;
}
```

## 抽象题意（数学）--- 数字转换

如果一个数 x的约数之和 y（不包括他本身）比他本身小，那么 x可以变成 y，y也可以变成 x

例如，4可以变为 3，1可以变为 7。

限定所有数字变换在不超过 n的正整数范围内进行，求不断进行数字变换且不出现重复数字的最多变换步数。

![Alt text](../../../_resources/%E6%95%B0%E5%AD%97%E8%BD%AC%E6%8D%A2.jpeg)

```
#include <iostream>
#include <cstring>
#include <algorithm>
using namespace std;

const int N = 50010;
int h[N], e[N], ne[N], idx;
int n, ans, sum[N], d[N];
bool st[N];

void add (int a, int b) {
    e[idx] = b, ne[idx] = h[a], h[a] = idx ++ ;
}

int dfs(int u) {
    int d1 = 0, d2 = 0;
    for (int i = h[u]; ~i; i = ne[i]) {
        int j = e[i];
        int d = dfs(j) + 1;
        if (d > d1) d2 = d1, d1 = d;
        else if (d > d2) d2 = d;
    }
    ans = max(ans, d1 + d2);
    return d1;
}

int main() {
    cin >> n;

    for (int i = 1; i <= n; i ++ ) {
        for (int j = 2; j <= n / i; j ++ ) {
            sum[i * j] += i;//关键1
        }
    }
    memset(h, -1, sizeof h);
    for (int i = 2; i <= n; i ++ ) {
        if (sum[i] < i) {
            add(sum[i], i);
            st[i] = true;
        }
    }
    for (int i = 1; i <= n; i ++ ) {
        if (!st[i]) dfs(i);
    }
    cout << ans << endl;
    return 0;
}
```

## 二叉苹果树

有一棵二叉苹果树，如果树枝有分叉，一定是分两叉，即没有只有一个儿子的节点。

这棵树共 N
 个节点，编号为 1
 至 N
，树根编号一定为 1
。

我们用一根树枝两端连接的节点编号描述一根树枝的位置。

一棵苹果树的树枝太多了，需要剪枝。但是一些树枝上长有苹果，给定需要保留的树枝数量，求最多能留住多少苹果。

这里的保留是指最终与1号点连通。

```
#include <iostream>
#include <cstring>
#include <algorithm>
using namespace std;

const int N = 210;
int h[N], e[N], ne[N], idx, w[N];
int n, m, f[N][N];//以i为根，保留j条边的最大价值

void add (int a, int b, int c) {
    e[idx] = b, ne[idx] = h[a], w[idx] = c; h[a] = idx ++ ;
}


void dfs(int u, int father) {
    for (int i = h[u]; ~i; i = ne[i]) {
        if (e[i] == father) continue;
        dfs(e[i], u);

        for (int j = m; j >= 0; j -- ) {
            for (int k = 0; k < j; k ++ )  {
                //选了儿子j，但不选以j为根的捆绑，所以体积至少要减一，即留给父节点，必选u——j
                f[u][j] = max(f[u][j], f[u][j - k - 1] + f[e[i]][k] + w[i]);
            }
        }
    }
}

int main() {
    cin >> n >> m;
    memset(h, -1, sizeof h);
    for (int i = 0; i < n - 1; i ++ ) {
        int a, b, c;
        cin >> a >> b >> c;
        add(a, b, c), add(b, a, c);
    }

    dfs(1, -1);
    
    cout << f[1][m] << endl;

    system("pause");
    return 0;
}
```

## 战略游戏
他必须保护一座中世纪城市，这条城市的道路构成了一棵树。

每个节点上的士兵可以观察到所有和这个点相连的边。

他必须在节点上放置最少数量的士兵，以便他们可以观察到所有的边。

例如，下面的树：
![Alt text](../../../_resources/%E6%88%98%E7%95%A5%E6%B8%B8%E6%88%8F.jpg.gif)

只需要放置 1
 名士兵（在节点 1
 处），就可观察到所有的边。

 ```
 #include <iostream>
#include <cstring>
#include <algorithm>
using namespace std;

const int N = 1510;
int h[N], e[N], ne[N], idx;
int n, f[N][2];
bool st[N];

void add (int a, int b) {
    e[idx] = b, ne[idx] = h[a], h[a] = idx ++ ;
}

void dfs(int u) {
    f[u][0] = 0;//!!!!!!!!!!!!!!!!!!!!!!!!!!必须要写，多组测试数据的初始化
    f[u][1] = 1;
    for (int i = h[u]; ~i; i = ne[i]) {
        int j = e[i];
        dfs(j);

        f[u][0] += f[j][1];
        f[u][1] += min(f[j][0], f[j][1]);
    }
}

int main() {
    while (scanf("%d", &n) == 1) {//没有读到元素返回-1，读到元素返回个数
        memset(h, -1, sizeof h);
        idx = 0;
        memset(st, 0, sizeof st);
        for (int i = 0; i < n; i ++ ) {
            int id, cnt;
            scanf("%d:(%d)", &id, &cnt);
            while (cnt -- ) {
                int ver;
                scanf("%d", &ver);
                add(id, ver);
                st[ver] = true;
            }
        }

        int root = 0;
        while (st[root]) root ++ ;//找根
        dfs(root);
        printf("%d\n", min(f[root][1], f[root][0]));
    }

    system("pause");
    return 0;
}
```

# 皇宫看守
太平王世子事件后，陆小凤成了皇上特聘的御前一品侍卫。

皇宫各个宫殿的分布，呈一棵树的形状，宫殿可视为树中结点，两个宫殿之间如果存在道路直接相连，则该道路视为树中的一条边。

已知，在一个宫殿镇守的守卫不仅能够观察到本宫殿的状况，还能观察到与该宫殿直接存在道路相连的其他宫殿的状况。

大内保卫森严，三步一岗，五步一哨，每个宫殿都要有人全天候看守，在不同的宫殿安排看守所需的费用不同。

可是陆小凤手上的经费不足，无论如何也没法在每个宫殿都安置留守侍卫。

帮助陆小凤布置侍卫，在看守全部宫殿的前提下，使得花费的经费最少。

--- 

输入格式
输入中数据描述一棵树，描述如下：

第一行 n
，表示树中结点的数目。

第二行至第 n+1行，每行描述每个宫殿结点信息，依次为：该宫殿结点标号 i，在该宫殿安置侍卫所需的经费 k，该结点的子结点数 m，接下来 m个数，分别是这个结点的 m个子结点的标号 $r1,r2,…,rm$。

对于一个 n
 个结点的树，结点标号在 1
 到 n
 之间，且标号不重复。

 ```
 #include <iostream>
#include <cstring>
#include <algorithm>

using namespace std;

/*
以下注释为早期笔记，希望对你有所帮助

状态机 + 树形Dp问题
状态表示:
    f(i, 0)：第i号结点被他的父结点安排的守卫看住的方案数
    f(i, 1)：第i号结点被他的子结点安排的守卫看住的方案数
    f(i, 2)：第i号结点自己安排守卫看住的方案数
状态计算:(j是i的子结点)
    f(i, 0) = sum{min(f(j,1), f(j,2))}
        i是被他父结点看住的，那他的子结点要么自己看自己，要么被自己的子结点看住
    f(i, 1) = min{w(k) + f(k, 2) - sum{min(f(j,1), f(j,2))}}
        i如果是被子结点看住的，那么就要枚举他是被哪个子结点看住的所有方案，对所有方案求最小值
        这里的sum不包括j==k的情况，因此需要手动额外减去
    f(i, 2) = sum{min(f(j,0), f(j,1), f(j,2))} + w(u)
        i是被自己看住的，那他的子结点可以被父结点看住，可以自己看自己，也可以被自己的子结点看住
*/

const int N = 1510;

int n;
int h[N], e[N], ne[N], idx, w[N];
int f[N][3];
bool st[N];

void add(int a, int b) {
    e[idx] = b, ne[idx] = h[a], h[a] = idx ++ ;
}

void dfs(int u) {
    f[u][2] = w[u];//初始化放置自己的代价
    f[u][0] = 0;
    f[u][1] = 1e9; //f[u][1]求最小值，初始化为最大值

    for (int i = h[u]; ~i; i = ne[i]) {
        int j = e[i];
        dfs(j);

        f[u][0] += min(f[j][1], f[j][2]);//被父节点看到
        f[u][2] += min(min(f[j][0], f[j][1]), f[j][2]);//自己放置守卫
    }

    for (int i = h[u]; ~i; i = ne[i]) {
        int j = e[i];
        //f[j][2] 表达的是当前 f[u][1] 可以从子节点j放置守卫这一状态转移过来
        //f[u][0] - min(f[j][1], f[j][2]) 表达的是其他(除j外)各个子节点最少可以放置守卫的数量
        
        //f[u][1]表示u被子节点看到，那么被哪个看到呢？（可以有多个），那么不妨被j看到了（j放置守卫）
        //那么，除了j之外，u的其他儿子就可放可不放，f[u][0]正是表示u的所有儿子可放可不放的最小代价（包括了j）
        //j既然放了，就要在其“所有儿子可放可不放的最小代价”中把j的价值去掉，看u的儿子除了j之外的节点的情况
        
        //j放了，代价就要加上f[j][2],而其他的点的情况需要总代价把j点的相关代价去掉（j无论是自己亮，还是被子节点照亮）
        
        //如果减去的min是j自己亮，那么f[u][1]就是总情况（j亮了，其他不管亮不亮，反正u节点已经被照亮了，直接f[u][0]就行）
        //如果减去的min是j被子节点照亮，但现在还必须要让j是亮起来（用来照u）
        
        //一句话，第45行 ，j的代价就是min(f[j][1], f[j][2])，f[u][0]当时加的是这个，现在减的也要是这个
        f[u][1] = min(f[u][1], f[j][2] + f[u][0] - min(f[j][1], f[j][2]));
    }
}

int main() {
    cin >> n;
    memset(h, -1, sizeof h);

    for (int i = 1; i <= n; i ++ ) {
        int id, cost, cnt;
        cin >> id >> cost >> cnt;
        w[id] = cost;
        while (cnt -- ) {
            int ver;
            cin >> ver;
            add(id, ver);
            st[ver] = true;
        }
    }

    int  root = 1;
    while (st[root]) root ++ ;

    dfs(root);

    cout << min(f[root][2], f[root][1]) << endl;

    return 0;
}
```