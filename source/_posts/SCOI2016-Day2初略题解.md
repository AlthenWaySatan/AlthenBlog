---
title: '【SCOI2016】Day2初略题解'
date: 2019-02-15 16:18:00
categories:
- 题解
tags:
- OI
- 计算几何
- 凸壳
- 数形结合
- 数据结构
- 主席树
- 贪心
mathjax: true
---
$\ \ \ \ \ \,$做一套省选题来练练手（Day2）。

<!-- more -->

## [【T1 妖怪】](https://www.luogu.org/problemnew/show/P3291)

### 解法

$\ \ \ \ \ \,$ 对于一个妖怪的两个属性，为了方便我们把它定义为 $x$，$y$，而要求的一个妖怪的战斗力应该为：

$(\frac{b}{a}+1)x+(\frac{a}{b}+1)y$

$\ \ \ \ \ \,$ 既：

$\frac{b}{a}x+\frac{a}{b}y+x+y$

$\ \ \ \ \ \,$ 由于 $x$，$y$ 已经确定，所以我们需要找的是 $\frac{b}{a}x+\frac{a}{b}y$ 最大的最小。

$\ \ \ \ \ \,$容易得到这是个**对勾函数**，对于一个怪物，当$\frac{b}{a}=\sqrt{\frac{y}{x}}$ 时，战斗力最小。

$\ \ \ \ \ \,$所以我们以 $x$，$y$ 为横纵坐标做个上凸壳，那么答案就一定是在凸壳上面，就会存在下面两种情况：

- 在点上：$\frac{b}{a}=\sqrt{\frac{y}{x}}$
- 在边上：需要满足：
  $(\frac{b}{a}+1)x_1+(\frac{a}{b}+1)y_1=(\frac{b}{a}+1)x_2+(\frac{a}{b}+1)y_2$
  解得：$\frac{b}{a}=\frac{y_1-y_2}{x_2-x_1}$
  
$\ \ \ \ \ \,$就这样扫一遍过去就行了，复杂度 $O(n \log n)$。

### 代码

``` cpp
#include<queue>
#include<cmath>
#include<string>
#include<cstdio>
#include<cstring>
#include<iostream>
#include<algorithm>
using namespace std;
const int inf=0x7fffffff;
const int eps=1e-10;
const double pi=acos(-1.0);
inline int read(){
	int x=0,p=1;char c=getchar();
	while(c<'0'||c>'9'){if(c=='-')p=-1;c=getchar();}
	while(c<='9'&&c>='0'){x=(x<<1)+(x<<3)+(c&15);c=getchar();}
	return x*p;
}
const int N=1e6+20;
int n,m;
struct Point{long long x,y;}p[N],f[N];
inline Point operator +(const Point &a,const Point &b)
{return (Point){a.x+b.x,a.y+b.y};}
inline Point operator -(const Point &a,const Point &b)
{return (Point){a.x-b.x,a.y-b.y};}
inline bool operator <(const Point &a,const Point &b)
{return a.x==b.x?a.y<b.y:a.x<b.x;}
inline long long Cross(const Point &a,const Point &b)
{return a.x*b.y-a.y*b.x;}
int Solve(Point *P,int n,Point *F){
	sort(P+1,P+n+1);int top=0;
	for(register int i=1;i<=n;++i){
		for(;top>1&&Cross(F[top]-F[top-1],P[i]-F[top-1])>=0;top--);
    	F[++top]=P[i];
  	}
  	return top;
}
inline double solve(const Point &a,double k)
{return k<=0?inf:(double)a.x+a.y+k*a.x+a.y/k;}
inline double Getval_point(const Point &a)
{return sqrt((double)a.y/a.x);}
inline double Getval_line(const Point &a,const Point &b)
{return a.x==b.x?-inf:((double)(a.y-b.y)/(double)(b.x-a.x));}
int main(){
	freopen("monster.in","r",stdin);
	freopen("monster.out","w",stdout);
	n=read();
	for(register int i=1;i<=n;++i)p[i].x=1ll*read(),p[i].y=1ll*read();
	m=Solve(p,n,f);
	if(m<2){
		printf("%.4lf",solve(f[1],Getval_point(f[1])));
	}
	else{
		double k1,k2,k3,ans=inf;
		k1=Getval_point(f[1]),k2=Getval_line(f[1],f[2]);
		if(k1<=k2)ans=min(ans,solve(f[1],k1));
		k1=Getval_point(f[m]),k2=Getval_line(f[m-1],f[m]);
		if(k1>=k2)ans=min(ans,solve(f[m],k1));
  		ans=min(ans,solve(f[m],k2));
  		for(register int i=2;i<m;++i){
  			k1=Getval_line(f[i-1],f[i]);
  			k2=Getval_line(f[i],f[i+1]);
  			k3=Getval_point(f[i]);
  			ans=min(ans,solve(f[i],k1));
  			if(k1<=k3&&k3<=k2)
  			ans=min(ans,solve(f[i],k3));
  		}
  		printf("%.4lf\n",ans);
	}
	fclose(stdin);
	fclose(stdout);
	return 0;
}

```

## [【T2 美味】](https://www.luogu.org/problemnew/show/P3293)

### 解法

$\ \ \ \ \ \,$又是异或最大呢，不是线性基就是贪心了，day1才搞了线性基，可以排除，我们看看怎么贪心。

$\ \ \ \ \ \,$首先可以看到他有一个取值范围的限制，我们可以用到可持久化数据结构维护。

$\ \ \ \ \ \,$然后我们可以贪心地想，从高位到低为维护 $a_i$ 的存在性。这个可持久化数据结构需要满足下面的操作：

- 插入一个数；
- 删除一个数；
- 统计某个取值范围的数数量是多少。

$\ \ \ \ \ \,$我最后选择了权值主席树。

$\ \ \ \ \ \,$现在对于每一次询问，我们贪心一下，从高位到低位枚举，如果$b$这一位为$1$，我们就找 $0$ ，反之找 $1$。

$\ \ \ \ \ \,$怎么找呢？我们令当前找到第$i$位， $ans$ 等于当前最优的 $a_i+x$，那么我们就找当前 $[l,r]$ 范围内，是否存在有数次在区间（$1/0$为当前要找的数）：

$[ans+(1/0<<i)-x,ans+(1/0<<i)-x+(1<<i)-1]$

$\ \ \ \ \ \,$存在的话就更新$ans$为 $ans+(1/0<<i)$，不然退而求其次，取 $ans+(0/1<<i)$。

$\ \ \ \ \ \,$这样我们可以保证在完成贪心，取到第0位之时，$ans$ 等于最优的 $a_i+x$。（并不关心是哪个$a_i$，反正是拼出来了。

$\ \ \ \ \ \,$复杂度$O(n \log a_{max}+m\log^2 a_{max})$

### 代码

``` cpp
#include<queue>
#include<cmath>
#include<string>
#include<cstdio>
#include<cstring>
#include<iostream>
#include<algorithm>
using namespace std;
inline int read(){
	int x=0,p=1;char c=getchar();
	while(c<'0'||c>'9'){if(c=='-') p=-1;c=getchar();}
	while(c<='9'&&c>='0'){x=(x<<1)+(x<<3)+(c&15);c=getchar();}
	return x*p;
}
const int N=5e5+5;
int a[N],root[N];
int n,m,nn;
struct CM_Tree{
	#define lson l,mid,ls[rt]
	#define rson mid+1,r,rs[rt]
	int ls[N*20],rs[N*20],sum[N*20],size;
	int copy(int rt){
		ls[++size]=ls[rt];
		rs[size]=rs[rt];
		sum[size]=sum[rt];
		return size;
	}
	void update(int id,int l,int r,int &rt){
    	rt=copy(rt);sum[rt]++;
    	if(l==r)return;
    	int mid=(l+r)>>1;
    	if(id<=mid)update(id,lson);
    	else update(id,rson);
	}
	int query(int L,int R,int l,int r,int rt,int rt2){
    if(L<=l&&r<=R)return sum[rt2]-sum[rt];
    	int mid=(l+r)>>1,ret=0;
    	if(L<=mid)ret+=query(L,R,lson,ls[rt2]);
    	if(mid<R)ret+=query(L,R,rson,rs[rt2]);
    	return ret;
	}
}tree;
bool check(int i,int j,int L,int R){
	L=max(0,L);R=min(R,nn);
	if(L>R)return 0;
	return tree.query(L,R,0,nn,root[i],root[j])>0;
}
#define getbit(a,i) ((a>>i)&1)
int main(){
	freopen("food.in","r",stdin);
	freopen("food.out","w",stdout);
	n=read();m=read();
	for(register int i=1;i<=n;++i)a[i]=read(),nn=max(nn,a[i]);
  	for(register int i=1;i<=n;++i)root[i]=root[i-1],tree.update(a[i],0,nn,root[i]);
  	for(register int i=1;i<=m;++i){
    	int b=read(),x=read(),l=read(),r=read(),ans=0;
    	for(register int i=17,ls;i>=0;--i){
    		if(!getbit(b,i))ls=ans+(1<<i)-x;
    		else ls=ans-x;
      		if(check(l-1,r,ls,ls+(1<<i)-1))ans=ls+x;
      		else ans+=getbit(b,i)<<i;
    	}
    	printf("%d\n",ans^b);
  	}
	fclose(stdin);
	fclose(stdout);
	return 0;
}

```


## [【T3 围棋】](https://www.luogu.org/problemnew/show/P3290)
$\ \ \ \ \ \,$插头dp是不会做插头dp的，这辈子不可能做插头dp的。写起来又怪麻烦，就是打打傻逼暴力，才能骗得了分这样子。

$\ \ \ \ \ \,$（逃

