---
title: '关于DP的优化'
date: 2018-12-29 14:21:33
categories:
- 学习笔记
tags:
- OI
- 动态规划
- 矩阵乘法
- 单调队列
- 斜率优化
- WQS二分
mathjax: true
---

$\ \ \ \ \ \ \ \,$动态规划及其相优化方法的复习笔记：

<!-- more -->

## 矩阵快速幂

$\ \ \ \ \ \ \,$很多时候我们的dp式子如下形式，是一个递推形式$f_{(m,n)}$：

$f_{(i,j)}=a_{(1,j)}\cdot f_{(i-1,1)}+a_{(2,j)}\cdot f_{(i-1,2)}+\cdots+a_{(n,j)}\cdot f_{(i-1,n)}$

$\ \ \ \ \ \ \,$显然，若是$a_{(i,j)}$参数确定,复杂度也是$O(mn^2)$的，多数情况下不会达到这个复杂度，但是也是接受不了的，有些题目$m$给的特别大，$n$比较小，我们就考虑矩阵优化：

$\ \ \ \ \ \ \,$首先设定初始矩阵$A$，也就是原dp式子的初始化项：

$A= \begin{bmatrix} f_{(0,1)} & f_{(0,2)} & \cdots & f_{(0,n)}\\ \end{bmatrix} $

$\ \ \ \ \ \ \,$然后设定转移矩阵$B$：

$B= \begin{bmatrix} a_{(1,1)} & a_{(1,2)} & \cdots & a_{(1,n)}\\a_{(2,1)} & a_{(2,2)} & \cdots & a_{(2,n)}\\\vdots\\a_{(n,1)} & a_{(n,2)} & \cdots & a_{(n,n)}\\ \end{bmatrix} $

$\ \ \ \ \ \ \,$根据矩阵乘法的定义，很容易得到：

$A\times B=\begin{bmatrix} f_{(1,1)} & f_{(1,2)} & \cdots & f_{(1,n)}\\ \end{bmatrix}$

$\ \ \ \ \ \ \,$推广得到：

$A\times B^m=\begin{bmatrix} f_{(m,1)} & f_{(m,2)} & \cdots & f_{(m,n)}\\ \end{bmatrix}$

$\ \ \ \ \ \ \,$根据矩阵乘法满足交换律，所以我们可以先算出$B^m$，再$A$乘之，即可得到答案，矩阵快速幂如下，复杂度优化到$O(\log m \cdot n^3)$：

``` cpp
struct matrix{int n,m,a[N][N];};
inline matrix operator *(const matrix &a,const matrix &b){
	matrix ret;ret.n=a.n;ret.m=b.m;
	for(int i=1;i<=a.n;i++)
	for(int j=1;j<=b.m;j++){
	  ret.a[i][j]=0;
	  for(int k=1;k<=a.m;k++)
	  ret.a[i][j]+=a.a[i][k]*b.a[k][j];
	}
	return ret;
}
inline matrix power(matrix A,matrix B,int m){
	for(;m;m>>=1,B=B*B)if(m&1)A=A*B;
	return A;
}
```

### [P1349 广义斐波那契数列](https://www.luogu.org/problemnew/show/P1349)

$\ \ \ \ \ \ \,$模板题目，递推式是：

$a_{n}=p\cdot a_{(n-1)}+q\cdot a_{(n-2)}$

$\ \ \ \ \ \ \,$所以我们令：

$f_{i,2}=[a_{i-1},a_{i}]$

$\ \ \ \ \ \ \,$那么有：

$f_{(i,1)}=0\cdot f_{(i-1,1)}+1\cdot f_{(i-1,2)}$

$f_{(i,2)}=p\cdot f_{(i-1,1)}+q\cdot f_{(i-1,2)}$

$\ \ \ \ \ \ \,$所以我们把$A$，$B$矩阵设置为：

$A= \begin{bmatrix} a_{1} & a_{2} \end{bmatrix} $

$B= \begin{bmatrix} 0 & q\\1 &p \end{bmatrix} $

$\ \ \ \ \ \ \,$那么我们的答案就在$A\times B^{n-2}$的第二项。

``` cpp
#include<algorithm>
#include<iostream>
#include<cstdlib>
#include<cstring>
#include<cctype>
#include<cstdio>
#include<vector>
#include<string>
#include<queue>
#include<stack>
#include<cmath>
#include<map>
#include<set>
using namespace std;
const int inf=0x7fffffff;
const double eps=1e-10;
const double pi=acos(-1.0);
//char buf[1<<15],*S=buf,*T=buf;
//char getch(){return S==T&&(T=(S=buf)+fread(buf,1,1<<15,stdin),S==T)?0:*S++;}
inline int read(){
	int x=0,f=1;char ch;ch=getchar();
	while(ch<'0'||ch>'9'){if(ch=='-') f=0;ch=getchar();}
	while(ch>='0'&&ch<='9'){x=(x<<1)+(x<<3)+(ch&15);ch=getchar();}
	if(f)return x;else return -x;
}
const int N=5;
long long n,m;
int p,q;
struct matrix{int n,m;long long a[N][N];}A,B;
inline matrix operator *(const matrix &a,const matrix &b){
	matrix ret;ret.n=a.n;ret.m=b.m;
	for(int i=1;i<=a.n;i++)
	for(int j=1;j<=b.m;j++){
	  ret.a[i][j]=0ll;
	  for(int k=1;k<=a.m;k++)
	  ret.a[i][j]=(ret.a[i][j]+a.a[i][k]*b.a[k][j]%m)%m;
	}
	return ret;
}
inline matrix power(matrix A,matrix B,int m){
	for(;m;m>>=1,B=B*B)if(m&1)A=A*B;
	return A;
}
int main()
{
	A.n=1;A.m=B.m=B.n=2;
	p=read();q=read();
	A.a[1][1]=read();A.a[1][2]=read();
	B.a[1][1]=0;B.a[1][2]=q;
	B.a[2][1]=1;B.a[2][2]=p;
	scanf("%lld%lld",&n,&m);
	A=power(A,B,n-2);
	printf("%lld\n",A.a[1][2]);
	return 0;
}

```


## 单调队列

$\ \ \ \ \ \ \,$对于一个dp转移方程式，其中取最小或最大：

$f_{i}={\rm solve}(i,j)[l<j<r]$

$\ \ \ \ \ \ \,$若是可以化成如下形式：

$f_(i)=F(j)+g(i)[l<j<r]$

$\ \ \ \ \ \ \,$既 $j$ 造成的贡献与 $i$ 没有关系，并且 $j$ 造成的贡献我们需要取最大或者最小时，并且范围$[l<j<r]$有单调性时，我们可以利用单调队列来优化dp，从$O(n^2)$优化到$O(n)$。

$\ \ \ \ \ \ \,$核心想法是，建立一个容器，我们把 $F(j)$ 造成的贡献按照单调性加入该容器，若是现在需要新加入一个元素，那么就从队尾开始，把比他造成贡献不优的踢出去（原dp取 $\rm Min$ 的话就是需要 $F(j)$ 比队尾小，反之就是要大）。

$\ \ \ \ \ \ \,$当然这个时候队首将会是最优秀的 $j$，不过我们的范围限制还可能单调变化，于是我们又需要把队首那些范围不对的都踢掉，于是现在队首就是我们要的 $j$ 了，带入原dp即可。

$\ \ \ \ \ \ \,$由于所有元素都最多进入容器一次，又最多被踢一次，所以复杂度是$O(n)$的，模板如下：

``` cpp
int q=1,p=0,Q[N];
for(int i=1;i<=n;i--){
  while(q<=p&&better(F(i),F(Q[p])))p--;
  Q[++p]=i;
  while(q<=p&&!in_lim(i,Q[q]))q++;
  if(q<=p)f[i]=F(Q[q])+g(i); 
}
```

### [P2569 [SCOI2010]股票交易](https://www.luogu.org/problemnew/show/P2569)

$\ \ \ \ \ \ \,$这题显然会有一个dp方程，$f_{(i,j)}$表示在第$i$天手里有$j$张股票的最大收益：

- 直接购买：

  $f_{(i,j)}=-aP_i\times j[0\leq j\leq aS_i]$

- 不行动：

  $f_{(i,j)}=f_{(i-1,j)}$

- 买入：

  $f_{(i,j)}={\rm Max}_{k=j-aS_i}^{j-1}\left(f_{(i-w-1,k)}-(j-k)\times aP_i\right)$

- 卖出：

  $f_{(i,j)}={\rm Max}_{k=j+1}^{j+bS_i}\left(f_{(i-w-1,k)}+(k-j)\times bP_i\right)$
  
$\ \ \ \ \ \ \,$复杂度为$O(n^3)$，主要是后面两个操作花时间了，所幸，后面两个都可以斜率优化，复杂度优化为$O(n^2)$

``` cpp
#include<algorithm>
#include<iostream>
#include<cstdlib>
#include<cstring>
#include<cctype>
#include<cstdio>
#include<vector>
#include<string>
#include<queue>
#include<stack>
#include<cmath>
#include<map>
#include<set>
using namespace std;
const int inf=0x7fffffff;
const double eps=1e-10;
const double pi=acos(-1.0);
//char buf[1<<15],*S=buf,*T=buf;
//char getch(){return S==T&&(T=(S=buf)+fread(buf,1,1<<15,stdin),S==T)?0:*S++;}
inline int read(){
	int x=0,f=1;char ch;ch=getchar();
	while(ch<'0'||ch>'9'){if(ch=='-') f=0;ch=getchar();}
	while(ch>='0'&&ch<='9'){x=(x<<1)+(x<<3)+(ch&15);ch=getchar();}
	if(f)return x;else return -x;
}
const int N=2010;
int n,m,w;
int f[N][N];
int aP[N],bP[N],aS[N],bS[N];
int q,p,Q[N];
int main()
{
	n=read();m=read();w=read();
	for(int i=1;i<=n;i++)
	aP[i]=read(),bP[i]=read(),
	aS[i]=read(),bS[i]=read();
  memset(f,128,sizeof(f));
  for(int i=1;i<=n;i++){
  	for(int j=0;j<=aS[i];j++)f[i][j]=-j*aP[i];
  	for(int j=0;j<=m;j++)f[i][j]=max(f[i][j],f[i-1][j]);
  	if(i<=w)continue;
    q=1,p=0; 
    for(int j=0;j<=m;j++){
      while(q<=p&&f[i-w-1][Q[p]]+Q[p]*aP[i]<=f[i-w-1][j]+j*aP[i])p--;
      Q[++p]=j;
      while(q<=p&&Q[q]<j-aS[i])q++;
      if(q<=p)f[i][j]=max(f[i][j],f[i-w-1][Q[q]]+(Q[q]-j)*aP[i]); 
    }
	  q=1,p=0;
		for(int j=m;j>=0;j--){
      while(q<=p&&f[i-w-1][Q[p]]+Q[p]*bP[i]<=f[i-w-1][j]+j*bP[i])p--;
      Q[++p]=j;
      while(q<=p&&Q[q]>j+bS[i])q++;
      if(q<=p)f[i][j]=max(f[i][j],f[i-w-1][Q[q]]+(Q[q]-j)*bP[i]); 
    }
	}
	int ans=0;
	for(int i=0;i<=m;i++)ans=max(ans,f[n][i]);
  printf("%d\n",ans);
	return 0;
}

```


## 斜率优化

$\ \ \ \ \ \ \,$对于一个dp式子，我们尝试将它化成3个部分：

- 只与 $i$ 有关的（$A_i$）；
- 只与 $j$ 有关的（$D_j$）；
- 与 $i$ 和 $j$ 同时有关的（$B_i\cdot C_j$）；

$\ \ \ \ \ \ \,$所以我们会得到形如这样子的式子：

${B_i}\cdot C_j+{A_i}={D_j}$

$\ \ \ \ \ \ \,$那么这个时候，我们把$B_i$当做斜率，$C_j$是横坐标，$D_j$是纵坐标，我们想要的是$A_i$最大或者是最小，那么答案一定是下面的那一层点里面诞生，需要满足的是：

- 两个点之间的斜率必须优于$B_i$，若是取$\rm Max$就是需要大于，否者就是小于。

- 最优的$j$取值一定是斜率尽量优的。

$\ \ \ \ \ \ \,$那么我们的想法是维护这个斜率组成的凸壳，若是$B_i$单调，我们可以考虑单调队列，若是不单调，那么我们在单调队列的基础上面，就不能从队首踢，就是维护一整个凸壳，每次询问在上面二分。

$\ \ \ \ \ \ \,$下面给出单调队列版的模板：

``` cpp
int Q[N],q,p;
long double Slope(int i){};//B(i)
long long X(int i){}//C(j)
long long Y(int j){}//D(j)
long double slope(int i,int j){
	if(X(i)==X(j))return 1.0*inf;//max_inf,min_-inf;
	return (1.0*(Y(i)-Y(j)))/(1.0*(X(i)-X(j)));
}
int main()
{
	q=p=1;
	for(int i=1;i<=n;i++){
		while(q<p&&better(slope(Q[q],Q[q+1]),Slope))++q;
		f[i]=F(Q[q]);//原dp方程
		while(q<p&&better(slope(i,Q[p-1]),slope(Q[p],Q[p-1])))--p;
		Q[++p]=i;
	}
	return 0;
}
```
### [P3195 [HNOI2008]玩具装箱TOY](https://www.luogu.org/problemnew/show/P3195)

$\ \ \ \ \ \ \,$本题dp方程如下：

$f_i={\rm Min}_{j=1}^{i-1}\left(f_j+(Sum_i-Sum_j+i-j-L-1)^2\right)$

$\ \ \ \ \ \ \,$化成如下形式：

${2\times(Sum_i+i)}(Sum_j+j+L+1)+{\left(f_i-(Sum_i+i)^2\right)}={\left(f_j+(Sum_j+j+L+1)^2\right)}$

$\ \ \ \ \ \ \,$我们发现${2\times(Sum_i+i)}$是单调的，于是我们想用单调队列来维护这个凸壳：

``` cpp
#include<iostream>
#include<cstdio>
#include<algorithm>
#include<cstdlib>
#include<cstring>
#include<string>
#include<queue>
#include<map>
#include<set>
#include<stack>
#include<cmath>
#include<cctype>
using namespace std;
const int inf=0x7fffffff;
const double eps=1e-10;
const double pi=acos(-1.0);
//char buf[1<<15],*S=buf,*T=buf;
//char getch(){return S==T&&(T=(S=buf)+fread(buf,1,1<<15,stdin),S==T)?0:*S++;}
inline long long read(){
	long long x=0,f=1;char ch;ch=getchar();
	while(ch<'0'||ch>'9'){if(ch=='-') f=0;ch=getchar();}
	while(ch>='0'&&ch<='9'){x=(x<<1)+(x<<3)+(ch&15);ch=getchar();}
	if(f)return x;else return -x;
}
const int N=50010;
double f[N],sum[N],L;
int q,p,Q[N];
int n;
double fn(int i){return sum[i]+(double)i;}
double X(int i){return fn(i)+L;}
double Y(int i){return f[i]+(fn(i)+L)*(fn(i)+L);}
double slope(int i,int j){return (Y(i)-Y(j))/(X(i)-X(j));}
int main()
{
	n=(int)read();L=(double)read()+1.0;
	for(int i=1;i<=n;i++)
	sum[i]=(double)read(),sum[i]+=sum[i-1];
	q=p=1;
	for(int i=1;i<=n;i++){
		while(q<p&&slope(Q[q],Q[q+1])<2*fn(i))++q;
		f[i]=f[Q[q]]+(fn(i)-fn(Q[q])-L)*(fn(i)-fn(Q[q])-L);
		while(q<p&&slope(i,Q[p-1])<slope(Q[p-1],Q[p])) --p;
    	Q[++p]=i;
	}
	printf("%lld\n",(long long)f[n]);
	return 0;
}

```

## WQS二分

$\ \ \ \ \ \ \,$WQS二分和斜率优化很相似，多了一个宽度限制，既需要干好取$m$个。和斜率优化一样，我们需要把式子化成下面的形式：

${B_i}\cdot C_j+{A_i}={D_j}$

$\ \ \ \ \ \ \,$在二分之前，我们二分一个特殊的值 $v$，取$k$个物品时，总贡献会多$kv$，在进行dp的同时，若$v$越大，取的物品越少，那么我们检查在$v$，的时候有多少个物品被选就好了，**注意使用WQS二分需要满足$v$越大，取的物品越少**，既需要满足取$x$个时的总贡献斜率不增，说白了，就是需要满足选的越多越好。

$\ \ \ \ \ \ \,$模板如下：

``` cpp
int Q[N],q,p;
long double Slope(int i){};//B(i)
long long X(int i){}//C(j)
long long Y(int j){}//D(j)
long double slope(int i,int j){
	if(X(i)==X(j))return 1.0*inf;//max_inf,min_-inf;
	return (1.0*(Y(i)-Y(j)))/(1.0*(X(i)-X(j)));
}
bool cheak(long long v){
	q=p=1;
	for(int i=1;i<=n;i++){
		while(q<p&&better(slope(Q[q],Q[q+1]),Slope))++q;
		f[i]=F(Q[q])+v;//原dp方程
		tot[i]=tot[Q[q]]+1;
		while(q<p&&better(slope(i,Q[p-1]),slope(Q[p],Q[p-1])))--p;
		Q[++q]=i;
	}
  return tot[n]<=m;
}
int main()
{
  while(l<r){
    long long mid=l+r>>1;
    if(cheak(mid))r=mid;
    else l=mid+1;
  }
  cheak(l);
  printf("%lld\n",f[n]-l*m);
	return 0;
}
```

### [P4983 忘情](https://www.luogu.org/problemnew/show/P4983)

$\ \ \ \ \ \ \,$这个题是我们团队准备的，原式是来恶心人的，化简下来就是这个东西：

$\left(\sum_{i=1}^nx_i+1\right)^2$

$\ \ \ \ \ \ \,$dp方程显然就是这个东西：

$f_i={\rm Min}_{j=1}^{i-1}f_j+\left(Sum_i-Sum_j+1\right)^2$

$\ \ \ \ \ \ \,$化成如下形式：

${2\times Sum_i}\cdot Sum_j+{\left(f_i-(Sum_i+1)^2\right)}={f_j+Sum_j^2-2\times Sum_j}$

$\ \ \ \ \ \ \,$感性思考一下，当$m$越大，答案一定小越小，那么我们就可以使用WQS二分：

``` cpp
#include<algorithm>
#include<iostream>
#include<cstdlib>
#include<cstring>
#include<cctype>
#include<cstdio>
#include<vector>
#include<string>
#include<queue>
#include<stack>
#include<cmath>
#include<map>
#include<set>
using namespace std;
const int inf=0x7fffffff;
const double eps=1e-10;
const double pi=acos(-1.0);
//char buf[1<<15],*S=buf,*T=buf;
//char getch(){return S==T&&(T=(S=buf)+fread(buf,1,1<<15,stdin),S==T)?0:*S++;}
inline int read(){
	int x=0,f=1;char ch;ch=getchar();
	while(ch<'0'||ch>'9'){if(ch=='-') f=0;ch=getchar();}
	while(ch>='0'&&ch<='9'){x=(x<<1)+(x<<3)+(ch&15);ch=getchar();}
	if(f)return x;else return -x;
}
const int N=100010;
int n,m,tot[N];
long long sum[N],f[N],Q[N];
long long X(int a){return sum[a];}
long long Y(int a){return f[a]+sum[a]*sum[a]-2ll*sum[a];}
double solpe(int i,int j){return 1.0*(Y(i)-Y(j))/(X(i)-X(j));}
bool cheak(long long v){
	int q=1,p=1;
  for(int i=1;i<=n;i++){
  	while(q<p&&solpe(Q[q+1],Q[q])<2.0*sum[i])q++;
    f[i]=f[Q[q]]+(sum[i]-sum[Q[q]]+1)*(sum[i]-sum[Q[q]]+1)+v;
		tot[i]=tot[Q[q]]+1;
    while(q<p&&solpe(i,Q[p])<solpe(Q[p-1],Q[p]))p--;
    Q[++p]=i;
	}
  return tot[n]<=m;
}
int main()
{
	n=read();m=read();
	for(int i=1;i<=n;i++)sum[i]=sum[i-1]+1ll*read();
	long long l=1,r=(sum[n]*sum[n])/2,mid;
	while(l<r){
    long long mid=l+r>>1;
    if(cheak(mid))r=mid;
    else l=mid+1;
  }
  cheak(l);
  printf("%lld\n",f[n]-l*m);
	return 0;
}

```


