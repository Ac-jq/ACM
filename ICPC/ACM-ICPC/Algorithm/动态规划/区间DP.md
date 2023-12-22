# 区间DP
## 定义
区间类动态规划是线性动态规划的扩展，它在分阶段地划分问题时，与阶段中元素出现的顺序和由前一阶段的哪些元素合并而来有很大的关系。

$令状态 f(i,j) 表示将下标位置 i 到 j 的所有元素合并能获得的价值的最大值，那么 f(i,j)=\max\{f(i,k)+f(k+1,j)+cost\}，cost 为将这两组元素合并起来的代价。$

## 性质
区间 DP 有以下特点：

- 合并：
即将两个或多个部分进行整合，当然也可以反过来；

- 特征：能将问题分解为能两两合并的形式；

- 求解：对整个问题设最优值，枚举合并点，将问题分解为左右两个部分，最后合并两个部分的最优值得到原问题的最优值。

## 石子合并
$设有 N堆石子排成一排，其编号为 1,2,3,…,N$
。

每堆石子有一定的质量，可以用一个整数来描述，现在要将这 N
 堆石子合并成为一堆。

每次只能合并相邻的两堆，合并的代价为这两堆石子的质量之和，合并后与这两堆石子相邻的石子将和新堆相邻，合并时由于选择的顺序不同，合并的总代价也不相同。
需要考虑不在环上，而在一条链上的情况。

$令 f(i,j) 表示将区间 [i,j] 内的所有石子合并到一起的最大得分。$

写出 状态转移方程：
$f(i,j)=\max\{f(i,k)+f(k+1,j)+\sum_{t=i}^{j} a_t \}~(i\le k<j)$

$令 sum_i 表示 a 数组的前缀和，状态转移方程变形为 f(i,j)=\max\{f(i,k)+f(k+1,j)+sum_j-sum_{i-1} \}。$
### 代码：
```
#include <iostream>
using namespace std;

const int N = 305;
int dp[N][N], s[N];
int n, len;

int main() {
	cin >> n;
	for (int i = 1; i <= n; i ++ ) {
		cin >> s[i];
		s[i] += s[i - 1];//前缀和
	}
	
	for (int len = 2; len <= n; len ++ ) {
		for (int i = 1; i + len - 1 <= n; i ++ ) {
			int j = i + len - 1;
			dp[i][j] = 0x3f3f3f3f;
			for (int k = i; k <= j - 1; k ++ ) {
				dp[i][j] = min(dp[i][j], dp[i][k] + dp[k + 1][j] + s[j] - s[i - 1]);
			}
		}
	}
	cout << dp[1][n];
	return 0;
}
```
## 环形石子合并
将 n
 堆石子绕圆形操场排放，现要将石子有序地合并成一堆。

规定每次只能选相邻的两堆合并成新的一堆，并将新的一堆的石子数记做该次合并的得分。

请编写一个程序，读入堆数 n
 及每堆的石子数，并进行如下计算：
 1. 选择一种合并石子的方案，使得做 n−1
 次合并得分总和最大。
 2. 选择一种合并石子的方案，使得做 n−1
 次合并得分总和最小。

 ### 分析：
 $我们将这条链延长两倍，变成 2\times n 堆，其中第 i 堆与第 n+i 堆相同，用动态规划求解后，取 f(1,n),f(2,n+1),\dots,f(n-1,2n-2) 中的最优值，即为最后的答案。时间复杂度$

 ```
 #include <iostream>
#include <cstring>
#include <algorithm>

using namespace std;
//int存21亿，1e9是10亿，INF为1061109567
const int N = 410, INF = 0x3f3f3f3f;

int n;
int w[N], s[N];
int f[N][N], g[N][N];

int main() {
    cin >> n;
    for (int i = 1; i <= n; i ++ ) {
        cin >> w[i];
        w[n + i] = w[i];
    }
    for (int i = 1; i <= n * 2; i ++ ) s[i] = s[i - 1] + w[i];// cout << s[i] << endl;
    

    memset(f, 0x3f, sizeof f);
    memset(g, -0x3f, sizeof g);

    for (int len = 1; len <= n; len ++ ) {//len堆石子合并
        for (int l = 1; l + len - 1 <= n * 2; l ++ ) {
            int r = l + len - 1;
            if (len == 1) f[l][r] = g[l][r] = 0;
            else {
                for (int k = l; k < r; k ++ ) {//必须从l开始：（合并最左边和剩下右边）不能为r否则k+1越界
                    f[l][r] = min(f[l][r], f[l][k] + f[k + 1][r] + s[r] - s[l - 1]);
                    g[l][r] = max(g[l][r], g[l][k] + g[k + 1][r] + s[r] - s[l - 1]);
                }
            }
        }
    }
    
    int maxv = -INF, minv = INF;
    for (int i = 1; i <= n; i ++ ) {
        maxv = max(maxv, g[i][i + n - 1]);
        minv = min(minv, f[i][i + n - 1]);
    }

    cout << minv << endl << maxv << endl;

    return 0;
}
```