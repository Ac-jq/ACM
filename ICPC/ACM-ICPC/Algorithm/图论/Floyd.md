# Floyd算法
### 应用
- $最短路$
- $传递闭包$
- $找最小环$
- $恰好经过k条边的的最短路（倍增）$

## 1.最短路

```
int g[N][N];

void floyd()
{
    for(int k = 1; k <= n; k ++ )
        for(int i = 1; i <= n; i ++ )
            for(int j = 1; j <= n; j ++ )
                g[i][j] = min(g[i][j], g[i][k] + g[k][j]);

}
```
## 2.传递闭包
$给定 n个变量和 m个不等式。其中 n小于等于 26，变量分别用前 n的大写英文字母表示。$

$不等式之间具有传递性，即若 A>B且 B>C，则 A>C。$

$请从前往后遍历每对关系，每次遍历时判断：$

- 如果能够确定全部关系且无矛盾，则结束循环，输出确定的次序；
- 如果发生矛盾，则结束循环，输出有矛盾；
- 如果循环结束时没有发生上述两种情况，则输出无定解。

```
#include <bits/stdc++.h>
using namespace std;

const int N = 26;
bool g[N][N], d[N][N];
int n, m;
bool st[N];

void floyd() {
    memcpy(d, g, sizeof g);
    
    for (int k = 0; k < n; k ++ ) {
        for (int i = 0; i < n; i ++ ) {
            for (int j = 0; j < n; j ++ ) {
                d[i][j] |= d[i][k] && d[k][j];
            }
        }
    }
}

int check() {
    //矛盾情况
    for (int i = 0; i < n; i ++ ) {
        if (d[i][i]) return 2;
    }
    
    //不确定情况
    for (int i = 0; i < n; i ++ ) {
        for (int j = 0; j < i; j ++ ) {//如果j<n，那么第一次循环，d00永为0，找点对，所以j<i就行
            //cout << i << "----" << j << "----" << d[i][j] << "----" << d[j][i] << endl;
            if (!d[i][j] && !d[j][i]) return 0;//找到一对i，j之间的关系不明确
        }
    }
   // cout << endl;
    
    return 1;//整个流程没有问题
}

char get_min() {
    
    
    for (int i = 0; i < n; i ++ ) {
        if (!st[i]) {
            bool flag = false;
            for (int j = 0; j < n; j ++ ) {
                if (!st[j] && d[j][i]) {
                    flag = true;//找到j,还有 j关系i，则i不是最大值
                    break;
                }
            }
            //记得放到大if里面，要每次找完j后看有没有比i大的
            if (!flag) {
                 st[i] = true;
                return 'A' + i;
            }
           
        }
        
    }
}

int main() {
    while (cin >> n >> m, n || m) {
        //memset(g, 0, sizeof g);
        memset(d, 0, sizeof d);
        int flag = 0, t;
        
      for(int i = 1; i <= m; i ++)
        {
            char str[5];
            cin >> str;
            int a = str[0] - 'A', b = str[2] - 'A';

            if(!flag)
            {
                // g[a][b] = 1;
                // floyd();
                d[a][b] = 1;
                for (int x = 0; x < n; x ++ ) {
                    if (d[x][a]) d[x][b] = 1;
                    if (d[b][x]) d[a][x] = 1;
                    for (int y = 0; y < n; y ++ ) {
                        if (d[x][a] && d[b][y]) d[x][y] = 1;
                        if (d[b][y]) d[a][y] = 1;
                    }
                }
                flag = check();
                if(flag) t = i;
            }
        }


     
            if (flag == 0) {//跑完m个关系，仍然没有确定元素之间的关系，check里一直能找到关系不清的
                printf("Sorted sequence cannot be determined.\n");
            } else if (flag == 2) {//有矛盾
                printf("Inconsistency found after %d relations.\n", t);
            } else {//确定关系
                memset(st, 0, sizeof st);
                printf("Sorted sequence determined after %d relations: ", t);
                for (int i = 0; i < n; i ++ ) printf("%c", get_min());
                printf(".\n");
            }    
    }
    
    return 0;
}
```

## 3.找最小环
$给定一张无向图，求图中一个至少包含 3个点的环，环上的节点不重复，并且环上的边的长度之和最小。$

$该问题称为无向图的最小环问题。$

$你需要输出最小环的方案，若最小环不唯一，输出任意一个均可。$

```
#include <iostream>
#include <cstring>
#include <algorithm>

using namespace std;

const int N = 110, INF = 0x3f3f3f3f;

int n, m;
int d[N][N], g[N][N];  // d[i][j] 是不经过点
int pos[N][N];  // pos存的是中间点k
int path[N], cnt;  // path 当前最小环的方案, cnt环里面的点的数量

// 递归处理环上节点
void get_path(int i, int j) {
    if (pos[i][j] == 0) return;  // i到j的最短路没有经过其他节点

    int k = pos[i][j];  // 否则,i ~ k ~ j的话,递归处理 i ~ k的部分和k ~ j的部分
    get_path(i, k);
    path[cnt ++] = k;  // k点放进去
    get_path(k, j);
}

int main() {
    cin >> n >> m;

    memset(g, 0x3f, sizeof g);
    for (int i = 1; i <= n; i ++) g[i][i] = 0;

    while (m --) {
        int a, b, c;
        cin >> a >> b >> c;
        g[a][b] = g[b][a] = min(g[a][b], c);
    }

    int res = INF;
    memcpy(d, g, sizeof g);
    // dp思路, 假设k是环上的最大点, i ~ k ~ j(Floyd的思想)
    for (int k = 1; k <= n; k ++) {

        // 求最小环, 
        //至少包含三个点的环所经过的点的最大编号是k
        for (int i = 1; i < k; i ++)  // 至少包含三个点，i，j，k不重合
            for (int j = i + 1; j < k; j ++)  
            // 由于是无向图,
            // ij调换其实是跟翻转图一样的道理
            // 直接剪枝, j从i + 1开始就好了
            // 更新最小环, 记录一下路径
                if ((long long)d[i][j] + g[j][k] + g[k][i] < res) {
                    // 注意,每当迭代到这的时候, 
                    // d[i][j]存的是上一轮迭代Floyd得出的结果
                    // d[i][j] : i ~ j 中间经过不超过k - 1的最短距离(k是不在路径上的)
                    res = d[i][j] + g[j][k] + g[k][i];  
                    cnt = 0;
                    path[cnt ++] = k;  // 先把k放进去
                    path[cnt ++] = i;  // 从k走到i(k固定的)
                    get_path(i ,j);  // 递归求i到j的路径
                    path[cnt ++] = j;  // j到k, k固定
                }

        // Floyd, 更新一下所有ij经过k的最短路径
        for (int i = 1; i <= n; i ++) 
            for (int j = 1; j <= n; j ++)   
                if (d[i][j] > d[i][k] + d[k][j]) {
                    d[i][j] = d[i][k] + d[k][j];  
                    pos[i][j] = k;
            }
    }

    if (res == INF) puts("No solution.");
    else {
        for (int i = 0; i < cnt; i ++) cout << path[i] << ' ';
        cout << endl;
    }

    return 0;
}
```

## 3.恰好经过K条边的最短路
$给定一张由 T条边构成的无向图，点的编号为 1∼1000之间的整数。$

$求从起点 S到终点 E恰好经过 N条边（可以重复经过）的最短路。$

$注意: 数据保证一定有解。$
```
#include <iostream>
#include <cstring>
#include <algorithm>
#include <map>

using namespace std;

const int N = 210;
int g[N][N], res[N][N];
int k, m, S, E;
int n;
map<int, int> ids;



void mul(int c[][N], int a[][N], int b[][N]) {//矩阵乘法
    static int temp[N][N];//每次qmi函数temp保存使用
    memset(temp, 0x3f, sizeof temp);//只由a，b确定可达矩阵，自己设置为不可达，否则有影响

    for (int k = 1; k <= n; k ++ ) {
        for (int i = 1; i <= n; i ++ ) {
            for (int j = 1; j <= n; j ++ ) {
                temp[i][j] = min(temp[i][j], a[i][k] + b[k][j]);
            }
        }
    }

    memcpy(c, temp, sizeof temp);
}

void qmi() {
    memset(res, 0x3f, sizeof res);
    //第一次，i到i只有一条边的距离为0
    for (int i = 1; i <= n; i ++ ) res[i][i] = 0;//相当于吧数值初始化为1，才能和后面相乘

    //这里计算走边数（幂级增长）（怎么增看k的二进制数哪里为1）
    while (k) {
        if (k & 1) mul(res, res, g);//res=res*g;根据k决定是否用当前g的结果去更新res
        k >>= 1;
        mul(g, g, g);//g=g*g;g的更新
    }

}

int main() {
    cin >> k >> m >> S >> E;
    //虽然点数较多，但由于边数少，所以我们实际用到的点数也很少，可以使用map来离散化来赋予
    //他们唯一的编号

    //不用保序，只需要知道哪两个点的距离就好
    if (!ids.count(S)) ids[S] = ++ n;
    if (!ids.count(E)) ids[E] = ++ n;
    S = ids[S], E = ids[E];

    memset(g, 0x3f, sizeof g);
    //这里我们来解释一下为什么不去初始化g[i][i]=0呢？
    //我们都知道在类Floyd算法中有严格的边数限制，如果出现了i->j->i的情况其实在i->i中我们是有2条边的
    //要是我们初始化g[i][i]=0,那样就没边了，影响了类Floyd算法的边数限制！
    while (m -- ) {
        int a, b, c;
        cin >> c >> a >> b;

        if (!ids.count(a)) ids[a] = ++ n;
        if (!ids.count(b)) ids[b] = ++ n;
        a = ids[a], b = ids[b];
        g[a][b] = g[b][a] = min(g[a][b], c);
    }

    qmi();

    cout << res[S][E] << endl;

    system("pause");
    return 0;
}
//https://www.acwing.com/solution/content/36603/
```