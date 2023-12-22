# $DFS$
## 对搜索的理解：：：：：
# 收集宝石(蓝桥14.3）

## 题目背景

聪聪在玩冒险岛游戏，为了召唤法力更强大的神龙，他必须尽可能收集更多的魔法宝石，每颗宝石都有不同的功效。不过在游戏里，几乎每一颗魔法宝石都会和另外一颗宝石相冲。相冲表示这两颗宝石不能同时拥有。例如，宝石A和宝石B相冲，那么，你可以选择两颗宝石都不收集，也可以只收集宝石A或者只收集宝石B，但不能同时拥有宝石A和宝石B。

## 题目描述

现在给定了游戏里宝石的数量N(2≤N≤100)，宝石从1到N依次编号，并给出M对(2≤M≤2000)相冲的宝石编号，请帮聪聪计算出最多能够收集到多少颗宝石。

## 输入格式

第一行输入两个正整数N和M(2≤N≤100，2≤M≤2000)，分别表示游戏里的宝石数量和M对相冲的宝石，两个正整数之间用一个空格隔开

接下来输入M行，每行两个正整数，分别表示相冲的两颗宝石的编号，两个正整数之间用一个空格隔开

## 输出格式

输出一个整数，表示聪聪在游戏里最多能够收集到的宝石数量

### 样例输入 #1

```
6 8
1 2
2 3
2 4
2 5
2 6
3 4
4 5
5 6
```
### 样例输出 #1
```
3
```
## Code:
```
#include <iostream>	
#include <algorithm> 
#include <cstring>
#include <queue>

using namespace std;
const int N = 110；

int n, m;
bool st[N];
bool g[N][N];
int ans;

void dfs(int step, int num) {
	if (ans > num + n - step) return ;
	if (step > n) {
		ans = max(ans, num);
		return ;
	}

	bool flag = false;
	for (int i = 1; i < step; i ++ ) {
		if (st[i] && g[i][step]) {//当前step跟前面有冲突
			flag = true;
			break;
		}
	}
	if (flag) {//有冲突必不选
		dfs(step + 1, num);
	} else {//没冲突可选可不选
		st[step] = true;
		dfs(step + 1, num + 1);
		st[step] = false;
		dfs(step + 1, num);
	}
}

int main() {
	cin >> n >> m;

	while (m -- ) {
		int a, b;
		cin >> a >> b;
		g[a][b] = g[b][a] = true;
	}

	dfs(1, 0);
	cout << ans << endl;

	system("pause");
	return 0;
}
```
## 分成互质组
给定 n个正整数，将它们分组，使得每组中任意两个数互质。至少要分成多少个组？
```
#include <bits/stdc++.h>
using namespace std;

const int N = 15;
int a[N];// 存放数
int g[N][N]; // 存放每个组
bool st[N];// 标记每个数是否被放进了组内
int n, ans;// 当前所存在的最优解

int gcd(int a, int b) {//注意别写成bool
    return b? gcd(b, a % b) : a;
}

bool check(int g[], int ed, int x) {// 判断当前组中的数是否和该数都互质(即该数能否放进该组)
    for (int i = 0; i < ed; i ++ ) // 枚举此组组内每个数
        if (gcd(g[i], x) > 1) // 只要组内有数和该数不互质了就 return false
            return false;
    return true;
}

void dfs(int group, int group_end_id, int num, int start) {
    // group为当前的最后一组的组的序号, group_end_id为当前组内搜索到的数的序号；
    // num为当前搜索过的数的数量, start为当前组开始搜索的数的序号
    if (group >= ans) return ;// 如果有比当前最优解所需的组数更多的解法说明此解不为最优解-->直接return即可
    if (num == n) ans = group;// 如果搜完了所有点了,说明此解为当前的最优解,更新最优解
    
    bool flag = true; // flag标记是否能新开一组
    for (int i = start; i < n; i ++) {// 枚举每个数
        if (!st[i] && check(g[group], group_end_id, a[i])) {// 如果当前数还未被放进组里 且 与当前的组中的数都互质
            st[i] = true;// 将该数标记为被放进组里的状态
            g[group][group_end_id] = a[i]; // 将该数放进该组
              // 继续搜索该组,组内数的数量gc + 1,总的数的数量tc + 1,搜索的数的序号i + 1
            dfs(group, group_end_id + 1, num + 1, start + 1);
            flag = false;// 如果能放进当前最后一组,则不用新开一组,故flag标记为false
            st[i] = false; // 恢复
        }
    }
    if (flag) dfs(group + 1, 0, num, 0);
    // 如果无法放进最后一组,则新开一组来搜索
    // 当前最后一组的组的序号group + 1, 组内搜索的数的序号group_end_id为0；
    // 搜索到的数tc未变, 当前组内开始搜索的数的序号start为0
    /* 此时的dfs操作其实就相当于开了一个组开始搜索的操作,还没有放数进来 */
}

int main() {
    cin >> n;
    for (int i = 0; i < n; ++ i) cin >> a[i];
    ans =  n;// 还未搜索时,假定最优解为最坏的情况每个数都分一组
    
    dfs(1, 0, 0, 0);// 从第一个组, 组内没有数；还没搜索到点 当前组还未开始遍历数start = 0的初始状态开始搜索
    cout << ans << endl;
    
    return 0;
}
```

# DFS剪枝
## 常用策略：
### 1. 优化搜索顺序：
大部分情况下，我们应该优先搜索 分支较少的节点。
### 2. 排除等效冗余
不要搜索重复状态
### 3. 可行性剪枝
当前状态已经不合法了
### 4. 最优性剪枝
方案已不是最优，无需后续搜索
### 5. 记忆化搜索
保留搜索记录

## 木棒
乔治拿来一组等长的木棒，将它们随机地砍断，使得每一节木棍的长度都不超过 50
 个长度单位。

然后他又想把这些木棍恢复到为裁截前的状态，但忘记了初始时有多少木棒以及木棒的初始长度。

请你设计一个程序，帮助乔治计算木棒的可能最小长度。

每一节木棍的长度都用大于零的整数表示。

### 输入格式 #1
输入包含多组数据，每组数据包括两行。

第一行是一个不超过 64的整数，表示砍断之后共有多少节木棍。

第二行是截断以后，所得到的各节木棍的长度。

在最后一组数据之后，是一个零。
```
#include <iostream>
#include <cmath>
#include <cstring>
#include <algorithm>

using namespace std;

const int N = 3250;
int a[N];
int n, ans = 1e9;
int length, sum;
bool st[N];

bool cmp(int x, int y) {
    return x > y;
}
//小喵爬山是遍历所有的喵，for循环找能放下的cab，所以不用标记哪只喵用过，不会重复
//此题有些区别，首先，小喵一定有解，我们要找最优，这题不一定有解
//这个题的槽必须要填满

bool dfs(int u, int len, int start) {
    if (u * length == sum) return true;

     if (len == length) {
        if (dfs(u + 1, 0, 0)) return true;
     }
     
    //剪枝2：排除重复
    for (int i = start; i <= n; i ++ ) {
        if (st[i]) continue;
        if (len + a[i] <= length) {
            st[i] = true;
            if (dfs(u, len + a[i], i + 1)) return true;
            st[i] = false;

            //剪枝3：可行性
            if (len == 0) return false;
            if (len + a[i] == length) return false;

            //剪枝
            int j = i;
            while (j <= n && a[i] == a[j]) j ++ ;
            i = j - 1;  
        }
    }
    return false;
}

int main() {
    ios::sync_with_stdio(false);
    cin.tie(0);
    cout.tie(0);

    while (cin >> n, n) {
        memset(st, 0, sizeof st);
        int l = 0;
        sum = 0;
        for (int i = 1; i <= n; i ++ ) {
            cin >> a[i];
            l = max(l, a[i]);
            sum += a[i];
        }
        sort(a + 1, a + n + 1, cmp);//剪枝1，搜索循序优化

        length = l;
       
        while (1) {
            if (sum % length == 0 && dfs(1, 0, 1)) {
                cout << length << endl;
                break;
            }
            length ++ ;
        }
    }
    return 0;

}
```
## 双向DFS（dp超时）
达达帮翰翰给女生送礼物，翰翰一共准备了 N
 个礼物，其中第 i
 个礼物的重量是 G[i]
。

达达的力气很大，他一次可以搬动重量之和不超过 W
 的任意多个物品。

达达希望一次搬掉尽量重的一些物品，请你告诉达达在他的力气范围内一次性能搬动的最大重量是多少

![Alt text](../../../_resources/%E5%8F%8C%E5%90%91DFS.png)