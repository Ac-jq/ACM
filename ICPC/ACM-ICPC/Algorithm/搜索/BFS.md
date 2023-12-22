# $BFS$
## Flood Fill算法
- ### 可以在线性时间复杂度内，找到某个点的连通块
### 思路：
    - 从地图外围走，走到连通块就进入BFS，把连通的地区都标记 + 计数
    - 遍历每一个格子，走到没有标记的连通块就标记 + 计数

# 先遍历到的，一定是最近的

1. ## [记录最短路径](https://www.acwing.com/problem/content/1078/)
给定一个 $n×n$的二维数组其中的1表示墙壁，0表示可以走的路，只能横着走或竖着走，不能斜着走，要求编程序找出从左上角到右下角的最短路线。
```
#include <iostream>
#include <cmath>
#include <cstring>
#include <algorithm>
#include <queue>

#define x first
#define y second
#define inf 0x3f3f3f3f
#define bug cout << "~~~~~here!~~~~~" << endl;

using namespace std;

typedef long long ll;
typedef pair<int, int> PII;

const int N = 1010;
int g[N][N];
int n;
PII pre[N][N];
bool st[N][N];
int dx[4] = {-1, 1, 0, 0}, dy[4] = {0, 0, -1, 1};

void bfs(int x, int y) {
	queue<PII> q;
	q.push({x, y});
	st[x][y] = true;

	while (q.size()) {
		PII t = q.front();
		q.pop();
		
	 for(int i = 0; i < 4; i ++){
            int x = t.x + dx[i], y = t.y + dy[i];
            if(x >= 0 && x < n && y >= 0 && y < n && g[x][y] == 0 && st[x][y] == false){
                st[x][y] = true;
                q.push({x, y});
                pre[x][y] = t;//存储一下当前合理位置的前驱,即该位置由哪个位置引入
            }
        }

	}
}

int main() {
	ios::sync_with_stdio(false);
	cin.tie(0);
	cout.tie(0);

	cin >> n;
    for (int i = 0; i < n; i ++ ) 
		for (int j = 0; j < n; j ++ ) 
			cin >> g[i][j];
	//倒着搜，这样pre（每个点是由哪个点转移来的）（不用担心被更新，先搜到的一定是最近的）	
	bfs(n - 1, n - 1);
 int x = 0, y = 0;
    while(x != n - 1 || y != n - 1){//当x = y = n - 1时,循环截止,因为下标n - 1为第一个元素，它没有前驱
        cout << x << ' ' << y << endl;
        auto t = pre[x][y];
        x = t.x, y = t.y;
    }
    cout << n - 1 << ' ' << n - 1 << endl;//单独打印第一个元素的下标

   system("pause");
   return 0;
}
```
## [理解先遍历的最近---走棋同理](https://www.acwing.com/problem/content/1102/)

农夫知道一头牛的位置，想要抓住它。农夫和牛都位于数轴上，农夫起始位于点 N，牛位于点 K。

农夫有两种移动方式：

1. $从 X移动到 X−1或 X+1，每次移动花费一分钟$

2. $从 X移动到 2∗X，每次移动花费一分钟假设牛没有意识到农夫的行动，站在原地不动。$

农夫最少要花多少时间才能抓住牛？
```
#include <iostream>
#include <cstring>
#include <algorithm>
#include <queue>

#define x first
#define y second

using namespace std;

typedef pair<int, int> PII;

const int N = 2e5 + 10;
int n, k;
int dist[N];

int bfs() {
	memset(dist, -1, sizeof dist);
	dist[n] = 0;
	queue<int> q;
	q.push(n);

	while (q.size()) {
		int t = q.front();
		q.pop();

        //bfs--边权为1-------------------先走到就是最近
		if (t + 1 <= k && dist[t + 1] == -1) {//可以朝k走
			dist[t + 1] = dist[t] + 1;
			q.push(t + 1);
		}
		if (t - 1 >= 0 && dist[t - 1] == -1) {
			dist[t - 1] = dist[t] + 1;
			q.push(t - 1);
		} 
		if (t * 2 < N && dist[t * 2] == -1) {
			dist[t * 2] = dist[t] + 1;
			q.push(t * 2);
		}
	}
	return dist[k];
}

int main() {
	ios::sync_with_stdio(false);
	cin.tie(0);
	cout.tie(0);

	cin >> n >> k;
	cout << bfs();

	system("pause");
	return 0;
}
```

# 多源BFS
### （搜索）改变状态至目标状态（八数码）
## 魔板
这是一张有 8个大小相同的格子的魔板：
```
1 2 3 4
8 7 6 5
```

可以用颜色的序列来表示一种魔板状态，规定从魔板的左上角开始，沿顺时针方向依次取出整数，构成一个颜色序列。

$对于上图的魔板状态，我们用序列 (1,2,3,4,5,6,7,8)来表示，这是基本状态。$

这里提供三种基本操作，分别用大写字母 A，B，C 来表示（可以通过这些操作改变魔板的状态）：

- A：交换上下两行；
```
8 7 6 5
1 2 3 4
```
- B：将最右边的一列插入到最左边；
```
4 1 2 3
5 8 7 6
```
- C：魔板中央对的4个数作顺时针旋转。
```
1 7 2 4
8 6 3 5
```

你要编程计算用最少的基本操作完成基本状态到特殊状态的转换，输出基本操作序列。

```
#include <iostream>
#include <cstring>
#include <algorithm>
#include <queue>
#include <unordered_map>

using namespace std;

unordered_map<string, int> dist;
unordered_map<string, pair<char, string>>  pre;
queue<string> q;

string A(string s) {
    reverse(s.begin(),s.end());
    return s;
}
string B(string s) {
    string t1 = s[3] + s.substr(0, 3);
    string t2 = s.substr(5, 3) + s[4];
    return t1 + t2;
}
string C(string s) {
    char c1 = s[1], c2 = s[2], c5 = s[5], c6 = s[6];
    s[1] = c6, s[2] = c1, s[5] = c2, s[6] =c5;
    return s;
}

void bfs(string start, string end) {
    if (start == end) return ;
    q.push(start);
    dist[start] = 0;

    while (q.size()) {
        string t = q.front();
        q.pop();

        string m[3];
        m[0] = A(t);
        m[1] = B(t);
        m[2] = C(t);

        for (int i = 0; i < 3; i ++ ) {
            string s = m[i];
            if (dist.count(s) == 0) {
                dist[s] = dist[t] + 1;
                q.push(s);
                pre[s] = {(char)(i + 'A'), t};//s是由t经过操作（i + ‘A’）转移而来
            }
        }
    }
}

int main() {
    ios::sync_with_stdio(false);
    cin.tie(0);
    cout.tie(0);

    int x;
    string start = "12345678", end;
    for (int i = 0; i < 8; i ++ ) {
        cin >> x;
        end += char(x + '0');
    }
    //for (int i = 0; i < 8; i ++ ) start += char(i + '1');

    bfs(start, end);

    cout << dist[end] << endl;
    string res;
    while (end != start) {//遍历前驱
        res += pre[end].first;
        end = pre[end].second;
    }
    if (res.length()) {//存在的话遍历顺序是倒着的
        reverse(res.begin(), res.end());
        cout << res << endl;
    }

    system("pause");
    return 0;
}

```

# 双向广搜
## 字串变换

已知有两个字串 A
, B
 及一组字串变换的规则（至多 6
 个规则）:

$A1→B1，A2→B2…$

$规则的含义为：在 A
 中的子串 A1
 可以变换为 B1
、A2
 可以变换为 B2…
。$

$例如：A
＝abcd B
＝xyz$

变换规则为：

$abc →
 xu ud →
 y y →
 yz$

则此时，A
 可以经过一系列的变换变为 B
，其变换的过程为：

$abcd →
 xud →
 xy →
 xyz$

共进行了三次变换，使得 A
 变换为 B
。

$注意，一次变换只能变换一个子串，例如 A
＝aa B
＝bb$

变换规则为：

a →b

此时，不能将两个 a 在一步中全部转换为 b，而应当分两步完成。

```
#include<bits/stdc++.h>
using namespace std;
const int N = 6;
int n;
string a[N],b[N];
// 扩展函数
// 参数：扩展的队列，到起点的距离，到终点的距离，规则，规则
//返回值：满足条件的最小步数
int extend(queue<string>& q, unordered_map<string,int>& da, unordered_map<string, int>& db,
        string a[], string b[]){
    // 取出队头元素
    string t = q.front();
    q.pop();

    for(int i = 0; i < t.size(); i ++)  // t从哪里开始扩展
        for(int j = 0; j < n; j ++) // 枚举规则
            //如果t这个字符串的一段= 规则，比如= xyz，才可以替换
            if(t.substr(i, a[j].size()) == a[j]){
                // 变换之后的结果state:前面不变的部分+ 变化的部分 + 后面不变的部分
                // 比如abcd ，根据规则abc--> xu，变成 xud，这里的state就是xud
                string state = t.substr(0,i) +b[j] + t.substr(i + a[j].size());
                // state状态是否落到b里面去，两个方向会师，返回最小步数
                if(db.count(state)) return da[t] + 1 + db[state];
                // 如果该状态之前已扩展过，
                if(da.count(state)) continue;
                da[state] = da[t] + 1;
                q.push(state);
            }
    return 11;

}
// 从起点和终点来做bfs
int bfs(string A, string B){
    if (A == B) return 0;
    queue<string> qa, qb; // 两个方向的队列
    //每个状态到起点的距离da（哈希表），每个状态到终点的距离db哈希表
    unordered_map<string, int> da, db; 
    // qa从起点开始搜，qb从终点开始搜
    qa.push(A), da[A] = 0; // 起点A到起点的距离为0
    qb.push(B), db[B] = 0; // 终点B到终点B的距离为0

    // qa和qb都有值，说明可以扩展过来，否则说明是不相交的
    while(qa.size() && qb.size()){
        int t; // 记录最小步数
        // 哪个方向的队列的长度更小一些，空间更小一些，从该方向开始扩展，
        // 时间复杂度比较平滑,否则有1个点会超时
        if(qa.size() <= qb.size()) 
            t = extend(qa, da, db, a, b);
        else t = extend(qb, db, da, b, a);
        // 如果最小步数在10步以内

        if( t <= 10) return t;
    }

    return 11; // 如果不连通或者最小步数>10，则返回大于10的数

}

int main(){
    string A, B;
    cin >> A >> B;
    // 读入扩展规则，分别存在a数组和b数组
    while( cin >> a[n] >> b[n]) n ++;
    int step = bfs(A,B);
    if(step > 10) puts("NO ANSWER!");
    else cout << step << endl;
}

```
# A_star（A*）
- ### 当状态非常庞大的时候才有效果,A*是堆优化Dijkstra的进一步优化
###  第K短路

给定一张 N
 个点（编号 1,2…N
），M
 条边的有向图，求从起点 S
 到终点 T
 的第 K
 短路的长度，路径允许重复经过点或边。

注意： 每条最短路中至少要包含一条边。

![Alt text](https://staic.oss-cn-beijing.aliyuncs.com/typora/A-start%E7%AC%ACk%E7%9F%AD%E8%B7%AF.png)

![Alt text](https://staic.oss-cn-beijing.aliyuncs.com/typora/A-star%E5%88%86%E6%9E%90.png)

```
#include <iostream>
#include <cstring>
#include <algorithm>
#include <queue>

#define x first
#define y second

using namespace std;

typedef pair<int, int> PII;
typedef pair<int, PII> PIII;

const int N = 1010, M = 2e5 + 10;
int h[N], rh[N], e[M], ne[M], w[M], idx;
int n, m;
int f[N];//估价函数，相当于dist数组（反向）
bool st[N];
int S, T, K;
int cnt[N];

void add(int h[], int a, int b, int c) {
    e[idx] = b, ne[idx] = h[a], w[idx] = c, h[a] = idx ++ ;
}

void Djkstra() {
    priority_queue<PII, vector<PII>, greater<PII> > q;
    q.push({0, T});
    memset(f, 0x3f, sizeof f);
    f[T] = 0;
    
    while (q.size()) {
        PII t = q.top();
        q.pop();
        
        int ver = t.y;
        if (st[ver]) continue;
        st[ver] = true;
        for (int i = rh[ver]; ~i; i = ne[i]) {
            int j = e[i];
            if (f[j] > f[ver] + w[i]) {
                f[j] = f[ver] + w[i];
                q.push({f[j], j});
            }
        }
    }
}

int A_Star() {
    //估价值（）         真实距离（到起点）             节点编号
    priority_queue<PIII, vector<PIII>, greater<PIII>> heap;
    heap.push({f[S], {0, S}});
    
    while (heap.size()) {
      auto t = heap.top ();
        heap.pop ();
        int ver = t.y.y,distance = t.y.x;
        cnt[ver]++;
        if (cnt[T] == K) return distance;
        for (int i = h[ver];~i;i = ne[i]) {
            int j = e[i];
            if (cnt[j] < K) heap.push ({distance+w[i]+f[j],{distance+w[i],j}});
        }

    }
    return -1;
}

int main() {
    ios::sync_with_stdio(false);
    cin.tie(0);
    cout.tie(0);
    cin >> n >> m;
    memset(h, -1, sizeof h);
    memset(rh, -1, sizeof rh);
    while (m -- ) {
        int a, b, c;
        cin >> a >> b >> c;
        add(h, a, b, c), add(rh, b, a, c);
    }
    
    
    cin >> S >> T >> K;
    if (S == T) K ++ ;
    Djkstra();//预处理估计函数
    cout << A_Star() << endl;
    
    return 0;
}
```