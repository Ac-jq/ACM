# 线性DP
## 一、数字三角形
给定一个如下图所示的数字三角形，从顶部出发，在每一结点可以选择移动至其左下方的结点或移动至其右下方的结点，一直走到底层，要求找出一条路径，使路径上的数字的和最大。
```
        7
      3   8
    8   1   0
  2   7   4   4
4   5   2   6   5
```
```
#include<iostream>
#include<cstring>
#include<algorithm>

using namespace std;

const int N = 520;
int dp[N][N];
int n;
int INF = 1e9;

int main()
{
    cin >> n;
    for(int i = 1; i <= n; i ++ )
        for(int j = 1; j <= i; j ++ )
            cin >> dp[i][j];
            
    for(int i = n - 1; i > 0; i -- )//倒数第二行
        for(int j = 1; j <= i; j ++ )//哪行就有几个数
            dp[i][j] += max(dp[i + 1][j], dp[i + 1][j + 1]);//倒叙不断赋最大值，从倒数第二行开始！
            
    cout << dp[1][1] << endl;
    return 0;
}
```
## 二、 最长上升子序列（长度 + 求和）
给定一个长度为 N的数列，求数值严格单调递增的子序列的长度最长是多少。
```
#include<iostream>
#include<algorithm>
#include<cstring>

using namespace std;

const int N = 1010;
int f[N], a[N];
int n;

int main()
{
    cin >> n;
    for(int i = 1; i <= n; i ++ ) cin >> a[i];
    
    int res = 1;//想一想数据1  1；初始为0的话最后输出0而不是1
    for(int i = 1; i <= n; i ++ )
    {
        f[i] = 1;
        //dp[i] = a[i];求序列和
        for(int j = 1; j < i; j ++ )
        {
            if(a[i] > a[j])
            {
                f[i] = max(f[i], f[j] + 1);//纸上模拟一遍很容易
                res = max(res, f[i]);
                //dp[i] = max(dp[i], dp[j] + a[i]);求序列和
            }
        }
    }
    cout << res;
    return 0;
}
```

- ## 二分优化
  
```
#include <bits/stdc++.h>
using namespace std;

const int N = 100005;
int n, a[N];
int f[N], idx;

int find(int x) {
	int l = 1, r = idx;
	while (l < r) {
		int mid = l + r >> 1;
		if (f[mid] >= x) r = mid;
		else l = mid + 1;
	}
	return l;
}

int main() {
	scanf("%d", &n);
	for (int i = 0; i < n; i++) scanf("%d", &a[i]);

	for (int i = 0; i < n; i ++ ) {
		if (idx == 0 || a[i] > f[idx]) {
			f[ ++ idx] = a[i];
		} else {
			int it = find(a[i]);//第一个比ai大的数
			f[it] = a[i]; 
		}
	}
	cout << idx;
	return 0;
}
```

- ## 接龙数列 （应用）

```
#include <iostream>
#include <cstring>
#include <algorithm>
#include <vector>
using namespace std;

const int N = 1e5 + 10;

int n;  
int f[10];


int main() {
    cin >> n;
    
    for (int i = 0; i < n; i ++ ) {
        string s;
        cin >> s;
        int a = s[0] - '0', b = s.back() - '0';

        f[b] = max(f[b], f[a] + 1);

    }
    int res = 0;
    for (int i = 0; i < 10; i ++ ) res = max(res, f[i]);
    cout << n - res << endl;

    return 0;
}
```
## 三、最长公共子序列
给定两个长度分别为 N和 M的字符串 A和 B，求既是 A的子序列又是 B的子序列的字符串长度最长是多少

- $集合表示：f[i][j]表示a的前i个字母，和b的前j个字母的最长公共子序列长度$
- $集合划分：以a[i],b[j]是否包含在子序列当中为依据，因此可以分成四类：$
  - $a[i]不在，b[j]不在,max=f[i−1][j−1]$
  - $a[i]不在，b[j]在$
  - $a[i]在，b[j]不在$
  - $a[i]在，b[j]不在$

其实中间两种情况范围大了，但求最大值，所以没关系
```
#include <bits/stdc++.h>
using namespace std;

const int N = 1010;
int n, m;
char a[N], b[N];
int f[N][N];

int main() {
    cin >> n >> m >> a + 1 >> b + 1;
    
    for (int i = 1; i <= n; i ++ ) {
        for (int j = 1; j <= m; j ++ ) {// 00 01  10  11
            f[i][j] = max(f[i][j - 1], f[i - 1][j]);
            if (a[i] == b[j]) {
                f[i][j] = max(f[i][j], f[i - 1][j - 1] + 1);
            }
        }
    }
    
    cout << f[n][m] << endl;
    return 0;
}
```

## 四、最长公共上升子序列
对于两个数列 A和 B，如果它们都包含一段位置不一定连续的数，且数值是严格递增的，那么称这一段数是两个数列的公共上升子序列，而所有的公共上升子序列中最长的就是最长公共上升子序列了。

奶牛半懂不懂，小沐沐要你来告诉奶牛什么是最长公共上升子序列。

不过，只要告诉奶牛它的长度就可以了。

数列 A和 B的长度均不超过 3000

![Alt text](https://staic.oss-cn-beijing.aliyuncs.com/typora/%E6%9C%80%E9%95%BF%E5%85%AC%E5%85%B1%E4%B8%8A%E5%8D%87%E5%AD%90%E5%BA%8F%E5%88%97.png)

```
#include <bits/stdc++.h>
using namespace std;

const int N = 3010;
int a[N], b[N];
int n, f[N][N];

int main() {
    cin >> n;
    for (int i = 1; i <= n; i ++ ) cin >> a[i];
    for (int i = 1; i <= n; i ++ ) cin >> b[i];
    
    // for (int i = 1; i <= n; i ++ ) {
    //     for (int j = 1; j <= n; j ++ ) {
    //         f[i][j] = f[i - 1][j];
            
    //         if (a[i] == b[j]) {
    //             for (int k = 0; k < j; k ++ ) {//从0开始就是让1走一遍，
    //                 if (b[j] > b[k]) {//因为最短公共序列长度为1
    //                     f[i][j] = max(f[i][j], f[i - 1][k] + 1);
    //                 }
    //             }
    //         }
    //     }
    // }
    
    for (int i = 1; i <= n; i ++ ) {
        int maxn = 1;
        for (int j = 1; j <= n; j ++ ) {
            f[i][j] = f[i - 1][j];
            
            if (a[i] == b[j]) f[i][j] = max(f[i][j], maxn);
            if (a[i] > b[j]) maxn = max(maxn, f[i - 1][j] + 1);
        }
    }
    
    int res = 0;
    for (int i = 0; i <= n; i ++ ) {
        res = max(res, f[n][i]);
    }
    cout << res << endl;
    return 0;
}
```

## 编辑距离（应用）
给定 n
 个长度不超过 10
 的字符串以及 m
 次询问，每次询问给出一个字符串和一个操作次数上限。

对于每次询问，请你求出给定的 n
 个字符串中有多少个字符串可以在上限操作次数内经过操作变成询问给出的字符串。

#### - $每个对字符串进行的单个字符的插入、删除或替换算作一次操作。$
![Alt text](https://staic.oss-cn-beijing.aliyuncs.com/typora/%E7%BC%96%E8%BE%91%E8%B7%9D%E7%A6%BB.jpg)

```
#include<iostream>
#include<algorithm>
#include<string.h>
using namespace std;

const int N = 15, M = 1010;
int n, m;
int f[N][N];//[i][j]表示从s1前i个字符转换到s2前j个字符最少的步骤数。
char str[M][N];

int edit_distance(char a[], char b[])//将a转换成b
{
    int la = strlen(a + 1), lb = strlen(b + 1);
    
    for(int i = 1; i <= la; i ++ ) f[i][0] = i;//删i次
    for(int i = 1; i <= lb; i ++ ) f[0][i] = i;//加i次
    
    for(int i = 1; i <= la; i ++ )
        for(int j = 1; j <= lb; j ++ )
        {
            f[i][j] = min(f[i - 1][j] + 1, f[i][j - 1] + 1);
            f[i][j] = min(f[i][j], f[i - 1][j - 1] + (a[i] != b[j]));
        }
        
    return f[la][lb];
}

int main()
{
    cin >> n >> m;
    for(int i = 0; i < n; i ++ ) cin >> str[i] + 1;
    
    
    while(m --)
    {
        char s[N];
        int x;
        int res = 0;
        
        cin >> s + 1 >> x;
        for(int i = 0; i < n; i ++ )
            if(edit_distance(str[i], s) <= x) res ++ ;
        
        cout << res << endl;
    }
    
    
    return 0;
}
```
