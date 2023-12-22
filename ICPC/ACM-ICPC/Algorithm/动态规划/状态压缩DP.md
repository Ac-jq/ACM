# 状态压缩DP
- ### 状压 DP 是动态规划的一种，通过将状态压缩为整数来达到优化转移的目的。
## 引入
求把 N×M的棋盘分割成若干个 1×2的长方形，有多少种方案。

例如当 N=2，M=4时，共有 5种方案。当 N=2，M=3时，共有 3种方案。

![Alt text](https://staic.oss-cn-beijing.aliyuncs.com/typora/%E8%92%99%E5%BE%B7%E9%87%8C%E5%AE%89%E7%9A%84%E6%A2%A6%E6%83%B3.jpg)

```
#include<iostream>
#include<algorithm>
#include<cstring>
using namespace std;

const int N = 12, M = 1 << N;
long long f[N][M];//i-1列已经确定，伸出到i列的状态是j
int n, m;
bool st[M];

int main() {
    while(cin >> n >> m, n || m)//行列 {//对于每种状态，先预处理每列不能有奇数个连续的0
        for(int i = 0; i < 1 << n; i ++ ) {
            st[i] = true;
            int cnt = 0;//记录一列中0的个数
            
            for(int j = 0; j < n; j ++ ) {//i状态下j行是否放置空格，0不放，1放
                if(i >> j & 1)//i(一种状态)的二进制数的第j位是不是1
                {
                    if(cnt & 1)//这一位是1，看前面连续0的个数
                    {
                        st[i] = false;
                        break;
                    }
                }
                else cnt ++ ;//这一位是0，空格加一
            }
            if(cnt & 1) st[i] = false;//最后一段那一段判断连续0的个数
        }
        memset(f, 0, sizeof(f));
        f[0][0] = 1;
        
        for(int i = 1; i <= m; i ++ )
            for(int j = 0; j < 1 << n; j ++ )
                for(int k = 0; k < 1 << n; k ++ )
                    if((j & k) == 0 && st[j | k])//j+k状态合法
                        f[i][j] += f[i - 1][k];
                        
        cout << f[m][0] << endl;
    }
    return 0;
}
```
## 小国王
$在 n×n的棋盘上放 k个国王，国王可攻击相邻的 8个格子，求使它们无法互相攻击的方案总数。$
```
#include <iostream>
#include <cstring>
#include <vector>
#include <algorithm>

using namespace std;

const int N = 12, M = 1 << 10, K = 110;

int n, m;
vector<int> state;
vector<int> head[M];
long long f[N][K][M];

bool check(int state) {//看该层状态是否合法
    for (int i = 0; i < n; i ++ ) {
        if ((state >> i & 1) && state >> i + 1 & 1) {
            return false;
        }
    }
    return true;
}

int count(int state) {//状态里面有多少个1
    int res = 0;
    for (int i = 0; i < n; i ++ ) res += state >> i & 1;
    return res;
}

int main() {
    cin >> n >> m;

    for (int i = 0; i < 1 << n; i ++ ) {
        if (check(i)) {//state存的是所有合法状态
            state.push_back(i);
        }
    }

    //哪两个状态之间可以无向连接
    for (int i = 0; i < state.size(); i ++ ) {
        for (int j = 0; j < state.size(); j ++ ) {
            int a = state[i], b = state[j];
            if ((a & b) == 0 && check(a | b)) {
                head[i].push_back(j);
            }
        }
    }

    f[0][0][0] = 1;//前0行，每个国王都不能摆，只有0是合法的
    for (int i = 1; i <= n + 1; i ++ ) {//n+1小技巧
        for (int j = 0; j <= m; j ++ ) {//j的话是从0到m对吧，m表示的是国王数量
            for (int a = 0; a < state.size(); a ++ ) {//a表示第i行的状态
                for (auto b : head[a]) {//然后来枚举所有a能到的状态
                    //求一下我们a里面的一个1的个数
                    int c = count(state[a]);
                    if (j >= c) {//因为我们这个表示我们当前这行摆放的国王数量一定要小于等于我们整个的上限对吧
                        f[i][j][a] += f[i - 1][j - c][b];
                    }
                }
            }
        }
    }

    cout << f[n + 1][m][0] << endl;

    system("pause");
    return 0;
}
```
## 玉米田
农夫约翰的土地由 M×N非常遗憾，部分土地是不育的，无法种植。

而且，相邻的土地不能同时种植玉米，也就是说种植玉米的所有方格之间都不会有公共边缘。

现在给定土地的大小，请你求出共有多少种种植方法。

土地上什么都不种也算一种方法。个小方格组成，现在他要在土地里种植玉米
```
#include <iostream>
#include <cstring>
#include <algorithm>
#include <vector>

using namespace std;

const int N = 15, MOD = 1e8, M = 1 << 12;
int n, m;
int g[N];
int f[N][M];
vector<int> state;
vector<int> head[M];


//a, b不包含连续两个1
//a&b为0
bool check(int state) {
    for (int i = 0; i < m; i ++ ) {
        if (state >> i & 1 && state >> i + 1 & 1) {
            return false;
        }
    }
    return true;
}

int main() {
    cin >> n >> m;

    //看哪些土地gi不能种，1不能种，0可以
    for (int i = 1; i <= n; i ++ ) {
        for (int j = 0; j < m; j ++ ) {
            int t;
            cin >> t;
            g[i] += !t << j;//！！！！！！！！！！！！！！！！！
        }
    }

    for (int i = 0; i < 1 << m; i ++ ) 
        if (check(i)) 
            state.push_back(i);
        
    for (int i = 0; i < state.size(); i ++ ) 
        for (int j = 0; j < state.size(); j ++ ) {
            int a = state[i], b = state[j];
            if ((a & b) == 0) //a和b各自已经判断过了
                head[i].push_back(j);
        }

    f[0][0] = 1;
    for (int i = 1; i <= n + 1; i ++ )
        for (int a = 0; a < state.size(); a ++ )
            for (auto b : head[a]) {
                //只有我能种玉米的状态中，一个都没有在不育土地上才行
                if (g[i] & state[a]) continue;
                f[i][a] = (f[i][a] + f[i - 1][b]) % MOD;
            }
    
    cout << f[n + 1][0] << endl;

    system("pause");
    return 0;
}
```