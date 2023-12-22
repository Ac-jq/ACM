# 数位DP数位dp的题目一般会问，某个区间内，满足某种性质的数的个数。

## 引入
数位是指把一个数字按照个、十、百、千等等一位一位地拆开，关注它每一位上的数字。如果拆的是十进制数，那么每一位数字都是 0~9，其他进制可类比十进制。

数位 DP：用来解决一类特定问题，这种问题比较好辨认，一般具有这几个特征：

- 要求统计满足一定条件的数的数量（即，最终目的为计数）；

- 这些条件经过转化后可以使用「数位」的思想去理解和判断；

- 输入会提供一个数字区间（有时也只提供上界）来作为统计的限制；

- 上界很大（比如 10^{18}），暴力枚举验证会超时。

## 分析
统计答案可以选择 记忆化搜索，也可以选择 循环迭代递推。

为了不重不漏地统计所有不超过上限的答案，要 从高到低 枚举每一位

再考虑每一位都可以 填哪些数字，最后利用上述 前缀和思想 统计答案

记忆化搜索 中要引入的参数通常有：

- $当前枚举到的数位 pos（搜索的深度）$
- $前一位数（或是前几位数）的情况 st（诸如 前一位是什么、前几位总和是多少、某个数出现了几次 等）$
- $前几位的数字是否等于上界的前几位数字 op (0/1)（限制本次搜索的数位范围）$
- $是否有前导零 lead (0/1)$

## 对应的数位dp问题有相应的解题技巧：

1. $利用前缀和，比如求区间[l, r]中的个数，转化成求dp[r] - dp[l - 1]的个数$
2. $利用树的结构来考虑（按位分类讨论）$

- ## 数位DP解法基本一致，框架都一样
- $一个数的B进制数有K个1，其他都是0$
- $不下降数$
- $不含前导零，且相邻数字的差值不小于2$
- $数字之和 模 N位0$

## [数的度量](https://www.acwing.com/problem/content/1083/)

$求给定区间 [X,Y]中满足下列条件的整数个数：这个数恰好等于 K个互不相等的 B的整数次幂之和。例如，设 X=15,Y=20,K=2,B=2，则有且仅有下列三个数满足题意：$

$17=2^4+2^0$

$18=2^4+2^1$

$20=2^4+2^2$

### 在一个区间[x,y]内，有多少个符合题意的数，这里的符合题意是指：这个数的B进制表示中，其中有K位上是1、其他位上全是0。

### 在具体些：在[1, y]中有多少个数满足：它的B进制数有K个1，其他全是0
- eg：1—65中。K = 2，B = 4;
1. 65的4进制数为 1001
2. 从高位开始枚举，第一个数是1，则分两种情况
   1. 这个位置放0，那么剩下三个位置中随便找两个位置放1，得到的数一定可以满足KB要求而且还小于65
   2. 这个位置放1，后面不能用组合数（万一放1放出界）,把1固定，后面只能放$k - last - 1$个1


## 思路如下：
### 1. 先将数x转化成为B进制，记为N。
```
//这是一段将n转化为B进制的代码
//输入：n 和进制B
#include<bits/stdc++.h>
using namespace std;

int main(){
    int n,B;
    cin >> n >> B;
    vector<int> a;
    while(n) a.push_back(n % B) , n /= B;
    for(int i = a.size()-1; i>= 0; i--) cout<< a[i]<<" ";
}
```
对于B进制数N（共n位），从最高位（第n位）开始遍历每1位，对于第 i 位，last表示第i位之前的所有位取过的1的个数。第i位上的数字x有三种情况：

- $限定值x=0：此时第i位只能取0，不影响1的个数，后面所有位可以取k-last个1。$
- $限定值x=1：此时可以取0或1.当此位取0时，后面的所有位可以取k -last个1；当此位取1时，后面所有位只能取k -last -1个1.$
- $限定值x>1： 当前位取0或1时，处理同2；其他值直接不能取，在本题中没意义。$
  

$这里用记忆化搜索来求组合数。存在数组f中，f[i][j]表示 从i个数中选出j个数的组合数。比如f[5][2]表示从5个数中选出2个数，结果就是f[5][2]=10.$

![Alt text](https://staic.oss-cn-beijing.aliyuncs.com/typora/%E6%95%B0%E4%BD%8DDP%E6%A8%A1%E6%8B%9F.png)

![Alt text](https://staic.oss-cn-beijing.aliyuncs.com/typora/%E6%95%B0%E4%BD%8DDP.png)

- AC代码
```
#include<bits/stdc++.h>
using namespace std;
const int N = 35; //位数

int f[N][N];// f[a][b]表示从a个数中选b个数的方案数，即组合数

int K, B; //K是能用的1的个数，B是B进制

//求组合数：预处理
void init(){
    for(int i=0; i< N ;i ++)
        for(int j =0; j<= i ;j++)
            if(!j) f[i][j] = 1;
            else f[i][j] =f[i-1][j] +f[i-1][j-1];
}



 //求区间[0,n]中的 “满足条件的数” 的个数
 //“满足条件的数”是指：一个数的B进制表示，其中有K位是1、其他位全是0
int dp(int n){
    
    if(n == 0) return 0; //如果上界n是0，直接就是0种
    
    vector<int> nums; 
    //把n在B进制下的每一位单独拿出来
    while(n) nums.push_back(n % B) , n/= B;
    
    int res = 0;//答案：[0,n]中共有多少个合法的数
    int last = 0; //已经有多少个1
    
    //从最高位开始遍历每一位
    for(int i = nums.size()-1; i>= 0; i--){
        int x = nums[i]; //取当前位上的数
        if(x>0) { 
        
        	//当前位填0，从剩下的所有位（共有i位）中选K-last个数。
            //size = 6, （6个位置）第一次i=5（末尾下标）填0
            //剩下0，1，2，3，4五个位置，正好是五个位置选K - last
            //注意思维转换一下
            res += f[i][K - last];
            
            if(x > 1) {
                //当前位填1，从剩下的所有位（共有i位）中选K-last-1个数。
                if(K - last -1 >= 0) res += f[i][K -last -1];
                break;
            } else {
            //上面统计完了**左分支**的所有情况，和右分支大于1的情况，

            //这个else 是x==1，
            //对应于：右分支为1的情况，即限定值为1的情况，也就是左分支只能取0
            //此时的处理是，直接放到下一位来处理
            //只不过下一位可使用的1的个数会少1，体现在代码上是last+1
                last ++;
                if(last > K) break;
            }
            
        }
        if(i==0 && last == K) res++; 
    }
    return res;
}

int main(){
    init();
    int l,r;
    cin >>  l >> r >> K >>B;
    cout<< dp(r) - dp(l-1) <<endl;  
}
```

## [数字游戏](https://www.acwing.com/problem/content/1084/)

某人命名了一种不降数，这种数字必须满足从左到右各位数字呈非下降关系，如 123，446。

现在大家决定玩一个游戏，指定一个整数闭区间 [a,b]，问这个区间内有多少个不降数。

![Alt text](https://staic.oss-cn-beijing.aliyuncs.com/typora/%E6%95%B0%E5%AD%97%E6%B8%B8%E6%88%8F.png)

![Alt text](https://staic.oss-cn-beijing.aliyuncs.com/typora/%E6%95%B0%E4%BD%8DDP%E7%8A%B6%E6%80%81.png)

![Alt text](https://staic.oss-cn-beijing.aliyuncs.com/typora/%E6%95%B0%E5%AD%97%E6%B8%B8%E6%88%8F%E2%80%94%E2%80%94%E6%A8%A1%E6%8B%9F.png)

```
#include <bits/stdc++.h>
using namespace std;

const int N = 12;//2^31 == 2e9;
int f[N][N];
int a[N];

void init() {//预处理不降数的个数
    for (int i = 0; i <= 9; i ++ ) f[1][i] = 1;//一位数
    for (int i = 2; i < N; i ++ ) {//阶段，枚举位数
        for (int j = 0; j <= 9; j ++ ) {//状态，枚举最高位
            for (int k = j; k <= 9; k ++ ) {//决策：枚举次高位
                f[i][j] += f[i - 1][k];//累加方案数
            }
        }
    }
}

int dp(int n) {
   if(n == 0) return 1; //0也是一个不降数

    int cnt = 0;
    //把n在B进制下的每一位单独拿出来
    while(n) a[ ++ cnt] = n % 10, n /= 10;

    int res = 0;//答案
    int last = 0; //上一位数字

    //从最高位开始遍历每一位
    for(int i = cnt; i >= 1; i--){
        int now = a[i]; //取当前位上的数
        for (int j = last; j < now; j ++ ) {
            res += f[i][j];
        }
        if (now < last) break;
        last = now;
        if (i == 1) res ++ ;
    }
    return res;
}

int main() {
    init();
    int l, r;
    while (cin >> l >> r) cout << dp(r) - dp(l - 1) << endl;
    return 0;
}
```
## [windy数](https://www.acwing.com/problem/content/1085/)
Windy 定义了一种 Windy 数：不含前导零且相邻两个数字之差至少为 2的正整数被称为 Windy 数。

Windy 想知道，在 A和 B之间，包括 A和 B，总共有多少个 Windy 数？
```
#include <bits/stdc++.h>
using namespace std;

const int N = 12;//2^31 == 2e9;
int f[N][N];
int a[N];

void init() {//预处理windy个数
    for (int i = 0; i <= 9; i ++ ) f[1][i] = 1;//一位数
    for (int i = 2; i < N; i ++ ) {//阶段，枚举位数
        for (int j = 0; j <= 9; j ++ ) {//状态，枚举第i位
            for (int k = 0; k <= 9; k ++ ) {//决策：枚举次高位
                if (abs(k - j) >= 2) 
                    f[i][j] += f[i - 1][k];//累加方案数
            }
        }
    }
}

int dp(int n) {
   if(n == 0) return 1; //0也是一个不降数
    
    int cnt = 0;
    //把n在B进制下的每一位单独拿出来
    while(n) a[ ++ cnt] = n % 10, n /= 10;
    
    int res = 0;//答案
    int last = -2; //上一位数字
    
    //从最高位开始遍历每一位
    for(int i = cnt; i >= 1; i-- ){
        int now = a[i]; //取当前位上的数
        for (int j = (i == cnt); j < now; j ++ ) {//最高位从1开始，没有前导0
            if (abs(j - last) >= 2) {
                res += f[i][j];
                //cout << j << ' ' << last << endl;
            } 
        }
        if (abs(now - last) < 2) break;//相邻两数差值小于2,固定这俩后，后面都不合法
        last = now;
        if (i == 1) res ++ ;
    }
    //上面都是有cnt位的，这里记录小于cnt位的
    for (int i = 1; i < cnt; i ++ ) 
        for (int j = !(i == 1); j <= 9; j ++ ) {//只有1位可以前导0，大于1位不能有前导0
            res += f[i][j];
        }
    return res;
}

int main() {
    init();
    int l, r;
    cin >> l >> r;
    cout << dp(r) - dp(l - 1) << endl;
    return 0;
}
```