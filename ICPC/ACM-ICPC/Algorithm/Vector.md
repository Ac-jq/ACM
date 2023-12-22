# Vector
### 声明
1. vector<int> v(n);
2. vector<vector<int>> v(n, vecotr<int> (m));

### 初始化(k)
1. vector<int> v(n, k);

### 赋值
1. 插入 `v.implace(x);`
2. `cin >> v[i];`
3. 
```
typedef tuple<int, int, int, int> T;
vector<T> v(n);
for (int i = 0; i < n; i ++ ) {
    auto &[a, b, c, d] = v[i];
    cin >> a >> b >> c;
    d = i;
}
```

### 排序
```
sort(v.begin(), v.end(), [&](const type &A, const type &B) {
    return ;
});
```
### 比较
vector 可以直接比较大小，效果等价于按字典序比较。

vector<int> a, b;

return a < b;

### 找某个元素
#### lower_bound()，upper_bound()  函数
函数用于在指定区域内查找不小于目标值的第一个元素 / 用于在指定范围内查找大于目标值的第一个元素。

`int *p = upper_bound(a, a + 5, 3);`

`return lower_bound(v.begin(), v.end(), x) - v.begin();`

#### max_element（）min_element（）函数
```
int maxPosition = max_element(n.begin(),n.end()) - n.begin(); //最大值下标

int minPosition = min_element(n.begin(),n.end()) - n.begin();//最小值下标
```

### 两个vector之间相互赋值
1. 方法1：   vector<int > v1(v2);  //声明
2.  v1 = v2      //最简单

### 将一个容器中的内容追加到另一个容器的后面
```
std::vector<int> v1, v2 ;
v1.insert(v1.end(), v2.begin(), v2.end());
```

### 操作
```
​clear()​:清空向量中所有元素

pop_back 去掉数组的最后一个数据

begin 得到数组头的指针

end 得到数组的最后一个单元+1的指针

front 得到数组头的引用

back 得到数组的最后一个单元的引用

reserve 改变当前vecotr所分配空间的大小

erase 删除指针指向的数据项

empty 判断vector是否为空

swap 与另一个vector交换数据

```
