# 贪心
#### 本页面将简要介绍贪心算法。

## 引入
贪心算法（英语：greedy algorithm），是用计算机来模拟一个「贪心」的人做出决策的过程。这个人十分贪婪，每一步行动总是按某种指标选取最优的操作。而且他目光短浅，总是只看眼前，并不考虑以后可能造成的影响。

可想而知，并不是所有的时候贪心法都能获得最优解，所以一般使用贪心法的时候，都要确保自己能证明其正确性。

解释适用范围贪心算法在有最优子结构的问题中尤为有效。最优子结构的意思是问题能够分解成子问题来解决，子问题的最优解能递推到最终问题的最优解。

## 证明
贪心算法有两种证明方法：反证法和归纳法。一般情况下，一道题只会用到其中的一种方法来证明。

- 反证法：如果交换方案中任意两个元素/相邻的两个元素后，答案不会变得更好，那么可以推定目前的解已经是最优解了。
- 归纳法：先算得出边界情况$（例如 n = 1）的最优解 F_1，然后再证明：对于每个 n，F_{n+1} 都可以由 F_{n}$ 推导出结果。
要点
## 常见题型
「我们将 XXX 按照某某顺序排序，然后按某种顺序（例如从小到大）选择。」。
「我们每次都取 XXX 中最大/小的东西，并更新 XXX。」（有时「XXX 中最大/小的东西」可以优化，比如用优先队列维护）
二者的区别在于一种是离线的，先处理后选择；一种是在线的，边处理边选择。

- 排序解法:用排序法常见的情况是输入一个包含几个（一般一到两个）权值的数组，通过排序然后遍历模拟计算的方法求出最优值。
- 后悔解法:思路是无论当前的选项是否最优都接受，然后进行比较，如果选择之后不是最优了，则反悔，舍弃掉这个选项；否则，正式接受。如此往复。
### 区别
与动态规划的区别
贪心算法与动态规划的不同在于它对每个子问题的解决方案都做出选择，不能回退。动态规划则会保存以前的运算结果，并根据以前的结果对当前进行选择，有回退功能。

## 邻项交换法的例题

![Alt text](https://staic.oss-cn-beijing.aliyuncs.com/typora/%E8%B4%AA%E5%BF%83%E9%A2%98.png)
```
struct uv {
  int a, b;

  bool operator<(const uv &x) const {
    return max(x.b, a * b) < max(b, x.a * x.b);
  }
};
```
## 后悔法的例题

![Alt text](https://staic.oss-cn-beijing.aliyuncs.com/typora/%E8%B4%AA%E5%BF%83%E9%A2%982.png)
```
#include <algorithm>
#include <cmath>
#include <cstdio>
#include <cstring>
#include <iostream>
#include <queue>
using namespace std;

struct f {
  long long d;
  long long p;
} a[100005];

bool cmp(f A, f B) { return A.d < B.d; }

priority_queue<long long, vector<long long>, greater<long long> >
    q;  // 小根堆维护最小值

int main() {
  long long n, i;
  cin >> n;
  for (i = 1; i <= n; i++) {
    scanf("%lld%lld", &a[i].d, &a[i].p);
  }
  sort(a + 1, a + n + 1, cmp);
  long long ans = 0;
  for (i = 1; i <= n; i++) {
    if (a[i].d <= (int)q.size()) {  // 超过截止时间
      if (q.top() < a[i].p) {       // 后悔
        ans += a[i].p - q.top();
        q.pop();
        q.push(a[i].p);
      }
    } else {  // 直接加入队列
      ans += a[i].p;
      q.push(a[i].p);
    }
  }
  cout << ans << endl;
  return 0;
}
```
## [增减序列](https://www.acwing.com/problem/content/102/)

给定一个长度为 n的数列 a1,a2,…,an，每次可以选择一个区间 [l,r]，使下标在这个区间内的数都加一或者都减一。

求至少需要多少次操作才能使数列中的所有数都一样，并求出在保证最少次数的前提下，最终得到的数列可能有多少种。

![Alt text](https://staic.oss-cn-beijing.aliyuncs.com/typora/%E8%B4%AA%E5%BF%83%E6%9E%84%E9%80%A0.png)

```
#include <iostream>	
#include <algorithm> 

using namespace std;

typedef long long ll;

const int N = 100010;
int a[N], b[N];
int n;

int main() {
	cin >> n;
	for (int i = 1; i <= n; i ++ ) cin >> a[i];

    ll p = 0, q = 0;
	for (int i = 2; i <= n; i ++ ) {
		b[i] = a[i] - a[i - 1];
		if (b[i] > 0) p += b[i];
		else q -= b[i];
	}

	cout << max(p, q) << endl << abs(p - q) + 1 << endl;
}
```