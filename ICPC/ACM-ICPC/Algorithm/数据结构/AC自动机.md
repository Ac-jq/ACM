# AC自动机（Trie + KMP）
## 前言字典树（Trie）
```
void insert(string s)//爬树
{
    int p = 0; //根节点
    for(int i = 0; s[i] != '\0'; i ++ )
    {
//从根开始26个分支，每个分支又有26个分支，以此类推...
        int u = s[i] - 'a';
        if(!son[p][u]) son[p][u] = ++ idx;//0→s[i]的分支
        p = son[p][u];//p从根爬到s[i]，以s[i]再为根
    }
    cnt[p] ++ ;//以p结尾的字符串++
}
```
### 1.AC自动机简介
AC自动机，不是会自动 AC的机器，而是由 Alfred.V.Aho和 Margaret.J.Corasick发明的 Aho−Corasick自动机，它能解决多模式串的匹配问题，相当于多模式串的 KMP

### 2.AC自动机的基本概念
AC自动机 =Trie+KMP所以 AC自动机本质是 Trie树，但加了 KMP的失配边，使得AC
自动机的效率和 KMP几乎一样快。

### 3.AC自动机的存储方法
上文已经提到，直接用 Trie树的存储方式，还要再加一个 KMP的失配边数组
Trie树一般要开字符串总个数 ×字符串最大长度， AC自动机也是如此。

我们对应过来就是某一个字符串的第 i个字符，就是 AC自动机中第 i 层！！！（ Trie中字符串的第几个字母就在第几层）我们不难发现，每次 KMP中算 $ne$指针，都是用前一个字母的 $ne$值计算出来的，对应到 AC自动机上就是每层的 $ne$值，都是由上一层节点的 $ne$值算出来的。所以要一层一层的搜，我们很容易想到 BFS因为算 ne指针是从第 $2$层开始算的，所以我们就把所有第 1层的所有点，扔到队列里，这样扩展出来的所有点就是第 $2$层及以下更低层。我们这里解释一下如果 $ne[i]=j$，假设从根节点到 $j$的字符串的长度是 w，则从根节点到 j的字符串和从 i往上走的第 w父亲到 i的字符串相等可以对应这图看一下，虚线是失配边。

## AC自动机：二维Trie树

- $ne[v]存节点v的回跳边的终点$
- $tr[u][i]存节点u的树边或转移边的终点$

- $回跳边指向父节点的回跳边所指节点的儿子$
- $转移边指向当前节点的回跳边所指节点的儿子$

### 建树：
![Alt text](https://staic.oss-cn-beijing.aliyuncs.com/typora/AC%E8%87%AA%E5%8A%A8%E6%9C%BA1.png)

### 查询：
![Alt text](https://staic.oss-cn-beijing.aliyuncs.com/typora/AC%E8%87%AA%E5%8A%A8%E6%9C%BA2.png)

![Alt text](https://staic.oss-cn-beijing.aliyuncs.com/typora/Trie.png)