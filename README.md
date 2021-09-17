# Segment-tree-
线段树入门


## 引子
子问题--1
给出n个数，m次问询，每次问询给出一个范围l、r，求区间l--r的总和
很容易想到用前缀和可以解决  O(n)

子问题--2
给出n个数，m次修改，每次给出一个范围l、r和一个值k，将区间l---r的数全部+k，m次修改后，再问询1次求区间l--r的总和
也可以想到这用差分就能解决  O(n)

最后将这两个问题结合起来，给出n个数，m次操作，操作有两种一个是将区间l--r的值+k,一个是询问区间l--r的总和。

这个问题与之前不同的地方是，之前想要求一个范围只需处理一次数据算前缀和，然后数据不变，可以一劳永逸算出多次查询，但现在用原本的方法的话，每当数据更新一次，就必须再算一次前缀和 O(n^2)。

那有没有一些方法能够优化最后一种问题呢？

现在就到了线段树出场的时候啦~

## 线段树基本概念

**1.线段树是一颗二叉树**

**2.线段树的每一个结点由左端点，右端点，范围值组成、懒标记（子树的待加值）**

**3.线段树核心思想是二分**

**下面来看看线段树长啥样**

![在这里插入图片描述](https://img-blog.csdnimg.cn/20210310131300502.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L2FzYmJ2,size_16,color_FFFFFF,t_70)特征：仔细观察一下会发现这其实是一个二分的一个过程，对于一个结点k的左子树它的左儿子的序号为2*k,右儿子的序号为2*k+1。设mid=(l+r)/2,然后左儿子的范围为[父节点l,父节点mid]，右儿子的范围为[父节点mid,父节点r]。

**在这着重介绍下懒标记**

每当我们修改一个范围内的值时，为了维护线段树，我们需要将每个与该范围相关的范围都修改一遍。如我要将1--8全部+3,那我需要将所有结点的w值都改变，这的操作数（15）已经比直接将8个元素+3的操作数（8）还多了。而懒标记就是来优化这个过程的，可以将懒标记当作一个待加值。如果我们暂时还用不到后面的值，这个待加值先暂时存储在他的父节点上，如果后面有相关的问询或修改要用到后面的结点，那这待加值顺带就给它加上，这样会避免每改一个值都将相关的范围结点改一遍，而一些根本用不到的结点也去改变就会浪费很多时间，所以先把这个待加值记在父结点上，如果后续操作需用到父节点的儿子，那时在加上也不迟，这样就确保了我们每次改变的结点都是我们需要用到的，不会有多余的浪费。
![在这里插入图片描述](https://img-blog.csdnimg.cn/2021031013464565.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L2FzYmJ2,size_16,color_FFFFFF,t_70)


## 线段树基本操作

**一个结点**

```cpp
struct node
{
	int l, r, f, w;//l表示左端，r右端，w范围内的和，f是懒标记后文解释
}tree[500005*4];//开的空间为题目给出元素数目的四倍，可以模拟一下建树的过程加以理解
```

**建树**

按着二分的思路往下递归，直到范围为一个元素赋值

```cpp
inline void build(int k, int ll, int rr)//k是结点序号、ll、rr分别是左右范围
{
	tree[k].l = ll, tree[k].r = rr;
	if (tree[k].l == tree[k].r) {cin >> tree[k].w; return ;}
	int mid = (tree[k].l + tree[k].r) / 2;
	build(2*k,ll,mid);
	build(2 * k + 1,mid+1,rr);
	tree[k].w = tree[2 * k].w + tree[2 * k + 1].w;
}
```

**单点查询**

和二分一样发现点在左边查询左子树，点在右边查询右子树直到找到该点


```cpp
inline int ask_point(int k, int x)  //返回x这个点的值
{
	if (tree[k].l == tree[k].r)
	{
		return tree[k].w;
	}
	if (tree[k].f)down(k);
	int mid = (tree[k].l + tree[k].r) / 2;
	if (x <= mid)return ask_point(2 * k, x);
	else return ask_point(2 * k + 1, x);
}
```

**单点修改**

```cpp
inline void change_point(int k, int x, int add)//k是结点序号 给第x个元素加add
{
	if (tree[k].l == tree[k].r)
	{
		tree[k].w += add;return;
	}
	if (tree[k].f)down(k);
	int mid = (tree[k].l + tree[k].r) / 2;
	if (x <= mid) change_point(2 * k, x, add);
	else change_point(2 * k + 1, x, add);
	tree[k].w = tree[2 * k].w + tree[2 * k + 1].w;
}
```

**范围查询**
对区域[x,y]的和的查询
当前结点表示的范围[l,r]
发现要查询的区域完全包含当前结点(x <= tree[k].l && y >= tree[k].r)  直接加上当前结点的值

令mid = (tree[k].l + tree[k].r) / 2

如果(x<=mid) 则表示在当前结点左儿子表示的范围中包含查询区域的部分或全部值，对左儿子查询加上其返回的值

如果(y > mid)  则表示在当前结点右儿子表示的范围中包含查询区域的部分或全部值，对右儿子查询加上其返回的值

如：


构造的线段树还是原来这个
![在这里插入图片描述](https://img-blog.csdnimg.cn/20210310142134225.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L2FzYmJ2,size_16,color_FFFFFF,t_70)
查询[2,6]的值

![在这里插入图片描述](https://img-blog.csdnimg.cn/20210310142320417.png)此时在1号结点，不满足全部包含
mid=(1+8)/2=4
x<=mid  左段包含
y>mid+1  右段包含

分成两段再次问询


![在这里插入图片描述](https://img-blog.csdnimg.cn/20210310142855153.png)
此时在1*2=2号结点，不满足全部包含
mid=(1+4)/2=2
x<=mid  左段包含
y>mid+1  右段包含

继续模拟即可
......

```cpp
inline int ask_interval(int k, int x, int y)//求[x,y]范围的和
{
	int ans = 0;
	if (x <= tree[k].l && y >= tree[k].r)
	{
		return tree[k].w;
	}
	if (tree[k].f)down(k);
	int mid = (tree[k].l + tree[k].r) / 2;
	if (x <= mid)ans += ask_interval(2 * k, x, y);
	if (y > mid)ans += ask_interval(2 * k + 1, x, y);
	return ans;
}
```

**范围修改**

```cpp
inline void change_interval(int k, int x, int y, int add)//x、y范围内+add
{
	if (x <= tree[k].l && y >= tree[k].r)
	{
		tree[k].w += add * (tree[k].r - tree[k].l + 1);
		tree[k].f += add;
		return;
	}
	if (tree[k].f)down(k);
	int mid = (tree[k].l + tree[k].r)/2;
	if (x <= mid)change_interval(2 * k, x, y, add);
	if (y > mid)change_interval(2 * k + 1, x, y, add);
	tree[k].w = tree[2 * k].w + tree[2 * k + 1].w;
}
```

## 线段树模板

```cpp
#include<iostream>
#define FAST ios::sync_with_stdio(false),cin.tie(0),cout.tie(0)
typedef long long ll;
using namespace std;

struct node
{
	ll l, r, f, w;
}tree[5000005];

inline void build(int k, int ll, int rr)
{
	tree[k].l = ll, tree[k].r = rr;
	if (tree[k].l == tree[k].r) {cin >> tree[k].w; return ;}
	int mid = (tree[k].l + tree[k].r) / 2;
	build(2*k,ll,mid);
	build(2 * k + 1,mid+1,rr);
	tree[k].w = tree[2 * k].w + tree[2 * k + 1].w;
}

inline void down(int k)
{
	tree[2 * k].f += tree[k]. f;
	tree[2 * k + 1].f += tree[k].f;
	tree[2 * k].w += tree[k].f * (tree[2 * k].r - tree[2 * k].l + 1);
	tree[2 * k + 1].w += tree[k].f * (tree[2 * k + 1].r - tree[2 * k + 1].l + 1);
	tree[k].f = 0;
}

inline void change_point(int k, int x, int add)
{
	if (tree[k].l == tree[k].r)
	{
		tree[k].w += add;return;
	}
	if (tree[k].f)down(k);
	int mid = (tree[k].l + tree[k].r) / 2;
	if (x <= mid) change_point(2 * k, x, add);
	else change_point(2 * k + 1, x, add);
	tree[k].w = tree[2 * k].w + tree[2 * k + 1].w;
}

inline ll ask_point(int k, int x)
{
	if (tree[k].l == tree[k].r)
	{
		return tree[k].w;
	}
	if (tree[k].f)down(k);
	int mid = (tree[k].l + tree[k].r) / 2;
	if (x <= mid)return ask_point(2 * k, x);
	else return ask_point(2 * k + 1, x);
}

inline ll ask_interval(int k, int x, int y)
{
	ll ans = 0;
	if (x <= tree[k].l && y >= tree[k].r)
	{
		return tree[k].w;
	}
	if (tree[k].f)down(k);
	int mid = (tree[k].l + tree[k].r) / 2;
	if (x <= mid)ans += ask_interval(2 * k, x, y);
	if (y > mid)ans += ask_interval(2 * k + 1, x, y);
	return ans;
}

inline void change_interval(int k, int x, int y, int add)
{
	if (x <= tree[k].l && y >= tree[k].r)
	{
		tree[k].w += add * (tree[k].r - tree[k].l + 1);
		tree[k].f += add;
		return;
	}
	if (tree[k].f)down(k);
	int mid = (tree[k].l + tree[k].r)/2;
	if (x <= mid)change_interval(2 * k, x, y, add);
	if (y > mid)change_interval(2 * k + 1, x, y, add);
	tree[k].w = tree[2 * k].w + tree[2 * k + 1].w;
}

int main()
{
	FAST;
	int n, m;cin >> n >> m;//n个元素m个操作
	build(1, 1, n);//建n个元素的线段树
	for (int i = 1;i <= m;i++)
	{
		int sec;cin >> sec;
		if (sec == 1)//询问某个点的值
		{
			int x;cin >> x;
			cout << ask_point(1, x) << endl;
		}
		else if(sec==2)//修改某个点的值
		{
			int x, add;cin >> x >> add;
			change_point(1, x, add);
		}
		else if (sec == 3)//询问一段区间
		{
			int x, y;cin >> x >> y;
			cout << ask_interval(1, x, y) << endl;
		}
		else if (sec == 4)//改变一段区间的值
		{
			int x, y, add;cin >> x >> y >> add;
			change_interval(1, x, y, add);
		}
	}
}
```

## 线段树入门题

[https://www.luogu.com.cn/problem/P3374](https://www.luogu.com.cn/problem/P3374)

[https://www.luogu.com.cn/problem/P3368](https://www.luogu.com.cn/problem/P3368)

[https://www.luogu.com.cn/problem/P3372](https://www.luogu.com.cn/problem/P3372)

[https://www.luogu.com.cn/problem/P3373](https://www.luogu.com.cn/problem/P3373)

**第一题源码**

```cpp
#include<iostream>
typedef long long ll;
using namespace std;

struct node
{
	ll l, r, f, w;
}tree[5000005];

inline void build(int k, int ll, int rr)
{
	tree[k].l = ll, tree[k].r = rr;
	if (tree[k].l == tree[k].r) {cin >> tree[k].w; return ;}
	int mid = (tree[k].l + tree[k].r) / 2;
	build(2*k,ll,mid);
	build(2 * k + 1,mid+1,rr);
	tree[k].w = tree[2 * k].w + tree[2 * k + 1].w;
}

inline void down(int k)
{
	tree[2 * k].f += tree[k]. f;
	tree[2 * k + 1].f += tree[k].f;
	tree[2 * k].w += tree[k].f * (tree[2 * k].r - tree[2 * k].l + 1);
	tree[2 * k + 1].w += tree[k].f * (tree[2 * k + 1].r - tree[2 * k + 1].l + 1);
	tree[k].f = 0;
}

inline void change_point(int k, int x, int add)
{
	if (tree[k].l == tree[k].r)
	{
		tree[k].w += add;return;
	}
	if (tree[k].f)down(k);
	int mid = (tree[k].l + tree[k].r) / 2;
	if (x <= mid) change_point(2 * k, x, add);
	else change_point(2 * k + 1, x, add);
	tree[k].w = tree[2 * k].w + tree[2 * k + 1].w;
}

inline ll ask_point(int k, int x)
{
	if (tree[k].l == tree[k].r)
	{
		return tree[k].w;
	}
	if (tree[k].f)down(k);
	int mid = (tree[k].l + tree[k].r) / 2;
	if (x <= mid)return ask_point(2 * k, x);
	else return ask_point(2 * k + 1, x);
}

inline ll ask_interval(int k, int x, int y)
{
	ll ans = 0;
	if (x <= tree[k].l && y >= tree[k].r)
	{
		return tree[k].w;
	}
	if (tree[k].f)down(k);
	int mid = (tree[k].l + tree[k].r) / 2;
	if (x <= mid)ans += ask_interval(2 * k, x, y);
	if (y > mid)ans += ask_interval(2 * k + 1, x, y);
	return ans;
}

inline void change_interval(int k, int x, int y, int add)
{
	if (x <= tree[k].l && y >= tree[k].r)
	{
		tree[k].w += add * (tree[k].r - tree[k].l + 1);
		tree[k].f += add;
		return;
	}
	if (tree[k].f)down(k);
	int mid = (tree[k].l + tree[k].r)/2;
	if (x <= mid)change_interval(2 * k, x, y, add);
	if (y > mid)change_interval(2 * k + 1, x, y, add);
	tree[k].w = tree[2 * k].w + tree[2 * k + 1].w;
}

int main()
{
	int n, m;cin >> n >> m;
	build(1, 1, n);
	for (int i = 1;i <= m;i++)
	{
		int sec;cin >> sec;
		if (sec == 1)
		{
			int x,k;cin >> x >> k;
			change_point(1, x,k);
		}
		else
		{
			int x, y;cin >> x >> y;
			cout << ask_interval(1, x, y) << endl;
		}
	}
}
```

**第二题源码**

```cpp
#include<iostream>
#define FAST ios::sync_with_stdio(false),cin.tie(0),cout.tie(0)
typedef long long ll;
using namespace std;

ll ans, a, b, x, y;

struct node
{
	ll l, r, f, w;
}tree[5000005];

inline void build(int k, int ll, int rr)
{
	tree[k].l = ll, tree[k].r = rr;
	if (tree[k].l == tree[k].r) {cin >> tree[k].w; return ;}
	int mid = (tree[k].l + tree[k].r) / 2;
	build(2*k,ll,mid);
	build(2 * k + 1,mid+1,rr);
	tree[k].w = tree[2 * k].w + tree[2 * k + 1].w;
}

inline void down(int k)
{
	tree[2 * k].f += tree[k]. f;
	tree[2 * k + 1].f += tree[k].f;
	tree[2 * k].w += tree[k].f * (tree[2 * k].r - tree[2 * k].l + 1);
	tree[2 * k + 1].w += tree[k].f * (tree[2 * k + 1].r - tree[2 * k + 1].l + 1);
	tree[k].f = 0;
}

inline void change_point(int k, int x, int add)
{
	if (tree[k].l == tree[k].r)
	{
		tree[k].w += add;return;
	}
	if (tree[k].f)down(k);
	int mid = (tree[k].l + tree[k].r) / 2;
	if (x <= mid) change_point(2 * k, x, add);
	else change_point(2 * k + 1, x, add);
	tree[k].w = tree[2 * k].w + tree[2 * k + 1].w;
}

inline ll ask_point(int k, int x)
{
	if (tree[k].l == tree[k].r)
	{
		return tree[k].w;
	}
	if (tree[k].f)down(k);
	int mid = (tree[k].l + tree[k].r) / 2;
	if (x <= mid)return ask_point(2 * k, x);
	else return ask_point(2 * k + 1, x);
}

inline ll ask_interval(int k, int x, int y)
{
	ll ans = 0;
	if (x <= tree[k].l && y >= tree[k].r)
	{
		return tree[k].w;
	}
	if (tree[k].f)down(k);
	int mid = (tree[k].l + tree[k].r) / 2;
	if (x <= mid)ans += ask_interval(2 * k, x, y);
	if (y > mid)ans += ask_interval(2 * k + 1, x, y);
	return ans;
}

inline void change_interval(int k, int x, int y, int add)
{
	if (x <= tree[k].l && y >= tree[k].r)
	{
		tree[k].w += add * (tree[k].r - tree[k].l + 1);
		tree[k].f += add;
		return;
	}
	if (tree[k].f)down(k);
	int mid = (tree[k].l + tree[k].r)/2;
	if (x <= mid)change_interval(2 * k, x, y, add);
	if (y > mid)change_interval(2 * k + 1, x, y, add);
	tree[k].w = tree[2 * k].w + tree[2 * k + 1].w;
}

int main()
{
	FAST;
	int n, m;cin >> n >> m;
	build(1, 1, n);
	for (int i = 1;i <= m;i++)
	{
		int sec;cin >> sec;
		if (sec == 2)
		{
			int x;cin >> x;
			cout << ask_point(1, x) << endl;
		}
		else
		{
			int x, y, k;cin >> x >> y >> k;
			change_interval(1, x, y, k);
		}
	}
}
```

**第三题源码**

```cpp
#include<iostream>
typedef long long ll;
using namespace std;

ll ans, a, b, x, y;

struct node
{
	ll l, r, f, w;
}tree[5000005];

inline void build(int k, int ll, int rr)
{
	tree[k].l = ll, tree[k].r = rr;
	if (tree[k].l == tree[k].r) {cin >> tree[k].w; return ;}
	int mid = (tree[k].l + tree[k].r) / 2;
	build(2*k,ll,mid);
	build(2 * k + 1,mid+1,rr);
	tree[k].w = tree[2 * k].w + tree[2 * k + 1].w;
}

inline void down(int k)
{
	tree[2 * k].f += tree[k]. f;
	tree[2 * k + 1].f += tree[k].f;
	tree[2 * k].w += tree[k].f * (tree[2 * k].r - tree[2 * k].l + 1);
	tree[2 * k + 1].w += tree[k].f * (tree[2 * k + 1].r - tree[2 * k + 1].l + 1);
	tree[k].f = 0;
}

inline void change_point(int k, int x, int add)
{
	if (tree[k].l == tree[k].r)
	{
		tree[k].w += add;return;
	}
	if (tree[k].f)down(k);
	int mid = (tree[k].l + tree[k].r) / 2;
	if (x <= mid) change_point(2 * k, x, y);
	else change_point(2 * k + 1, x, y);
	tree[k].w = tree[2 * k].w + tree[2 * k + 1].w;
}

inline ll ask_point(int k, int x)
{
	if (tree[k].l == tree[k].r)
	{
		return tree[k].w;
	}
	if (tree[k].f)down(k);
	int mid = (tree[k].l + tree[k].r) / 2;
	if (x <= mid)return ask_point(2 * k, x);
	else return ask_point(2 * k + 1, x);
}

inline ll ask_interval(int k, int x, int y)
{
	ll ans = 0;
	if (x <= tree[k].l && y >= tree[k].r)
	{
		return tree[k].w;
	}
	if (tree[k].f)down(k);
	int mid = (tree[k].l + tree[k].r) / 2;
	if (x <= mid)ans += ask_interval(2 * k, x, y);
	if (y > mid)ans += ask_interval(2 * k + 1, x, y);
	return ans;
}

inline void change_interval(int k, int x, int y, int add)
{
	if (x <= tree[k].l && y >= tree[k].r)
	{
		tree[k].w += add * (tree[k].r - tree[k].l + 1);
		tree[k].f += add;
		return;
	}
	if (tree[k].f)down(k);
	int mid = (tree[k].l + tree[k].r)/2;
	if (x <= mid)change_interval(2 * k, x, y, add);
	if (y > mid)change_interval(2 * k + 1, x, y, add);
	tree[k].w = tree[2 * k].w + tree[2 * k + 1].w;
}

int main()
{
	int n, m;cin >> n >> m;
	build(1, 1, n);
	for (int i = 1;i <= m;i++)
	{
		int sec;cin >> sec;
		if (sec == 1)
		{
			int x, y, k;cin >> x >> y >> k;
			change_interval(1, x, y, k);
		}
		else
		{
			int x, y;cin >> x >> y;
			cout << ask_interval(1, x, y) << endl;
		}
	}
}
```

**第四题源码**

```cpp
#include<iostream>
#define FAST ios::sync_with_stdio(false),cin.tie(0),cout.tie(0)
typedef long long ll;
using namespace std;
int Mod = 571373;

struct node
{
	ll l, r, a = 0, m = 1, w;
}tree[500005*4];

inline void build(int k, int ll, int rr)
{
	tree[k].l = ll, tree[k].r = rr;
	if (tree[k].l == tree[k].r) { cin >> tree[k].w; return; }
	int mid = (tree[k].l + tree[k].r) / 2;
	build(2 * k, ll, mid);
	build(2 * k + 1, mid + 1, rr);
	tree[k].w = (tree[2 * k].w + tree[2 * k + 1].w) % Mod;
}

inline void down(int k)
{
	tree[2 * k].w = (tree[2 * k].w * tree[k].m + (tree[2 * k].r - tree[2 * k].l + 1) * tree[k].a) % Mod;
	tree[2 * k + 1].w = (tree[2 * k+1].w * tree[k].m + (tree[2 * k + 1].r - tree[2 * k + 1].l + 1) * tree[k].a) % Mod;
	tree[2 * k].a = (tree[k].m * tree[2 * k].a + tree[k].a) % Mod;
	tree[2 * k + 1].a = (tree[k].m * tree[2 * k + 1].a + tree[k].a) % Mod;
	tree[2 * k].m = (tree[2 * k].m * tree[k].m) % Mod;
	tree[2 * k + 1].m = (tree[k].m * tree[2 * k + 1].m) % Mod;
	tree[k].m = 1;
	tree[k].a = 0;
}



inline ll ask_interval(int k, int x, int y)
{
	ll ans = 0;
	if (x <= tree[k].l && y >= tree[k].r)
	{
		return tree[k].w;
	}
	down(k);
	int mid = (tree[k].l + tree[k].r) / 2;
	if (x <= mid)ans += ask_interval(2 * k, x, y);
	if (y > mid)ans += ask_interval(2 * k + 1, x, y);
	return ans;
}

inline void change1_interval(int k, int x, int y, int add)
{
	if (x <= tree[k].l && y >= tree[k].r)
	{
		tree[k].w += add * (tree[k].r - tree[k].l + 1);
		tree[k].a += add;
		tree[k].w %= Mod;
		tree[k].a %= Mod;
		return;
	}
	down(k);
	int mid = (tree[k].l + tree[k].r) / 2;
	if (x <= mid)change1_interval(2 * k, x, y, add);
	if (y > mid)change1_interval(2 * k + 1, x, y, add);
	tree[k].w = (tree[2 * k].w + tree[2 * k + 1].w) % Mod;
}

inline void change2_interval(int k, int x, int y, int mul)
{
	if (x <= tree[k].l && y >= tree[k].r)
	{
		tree[k].w *= mul;
		tree[k].m *= mul;
		tree[k].a *= mul;
		tree[k].m %= Mod;
		tree[k].a %= Mod;
		return;
	}
	down(k);
	int mid = (tree[k].l + tree[k].r) / 2;
	if (x <= mid)change2_interval(2 * k, x, y, mul);
	if (y > mid)change2_interval(2 * k + 1, x, y, mul);
	tree[k].w = (tree[2 * k].w + tree[2 * k + 1].w) % Mod;
}

int main()
{
	FAST;
	int n, m;cin >> n >> m >> Mod;
	build(1, 1, n);
	for (int i = 1;i <= m;i++)
	{
		int sec;cin >> sec;
		if (sec == 1)
		{
			int x, y, k;cin >> x >> y >> k;
			change2_interval(1, x, y, k);
		}
		else if(sec==2)
		{
			int x, y, k;cin >> x >> y >> k;
			change1_interval(1, x, y, k);
		}
		else
		{
			int x, y;cin >> x >> y;
			cout << ask_interval(1, x, y) % Mod << endl;
		}
	}
}
```

