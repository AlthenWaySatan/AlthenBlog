---
title: 'Codeforces Round #536 (Div. 2)【己亥年农历新年赛】简略题解'
date: 2019-02-15 21:37:26
categories:
- 题解
tags:
- OI
- 修勾勾
- 暴力模拟
- 贪心
- 动态规划
- 矩阵乘法
- 数据结构
mathjax: true
---

## [【题目地址】](https://codeforces.com/contest/1106)
---

## 写在前面

$\ \ \ \ \ \ \,$这场比赛是wc2019回家那天晚上举办的，从8点到10点刚好在动车上，饥寒交迫，还拉肚子（吃不惯粤菜），就没有参加，是后面写的。

$\ \ \ \ \ \ \,$这套题在洛谷上面五颜六色的，很有意思啊（除了没有红的），题目也算可做，感觉很过年很快乐呢（~~嘤嘤~~

<!-- more -->

![在这里插入图片描述](https://img-blog.csdnimg.cn/20190215201821828.jpg?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dlaXhpbl80Mzk3Mzk2Ng==,size_16,color_FFFFFF,t_70)


---

## [A. Lunar New Year and Cross Counting](https://codeforces.com/contest/1106/problem/A)
$\ \ \ \ \ \ \,$模拟？暴力？可以不解释吗……

![在这里插入图片描述](https://img-blog.csdnimg.cn/20190215202145632.jpg)

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
const int N=510;
int n;
char mp[N][N];
int check(int x,int y){
	if(x+2>n||y+2>n)return 0;
	if(mp[x][y]!='X')return 0;if(mp[x][y+2]!='X')return 0;
	if(mp[x+1][y+1]!='X')return 0;
	if(mp[x+2][y]!='X')return 0;if(mp[x+2][y+2]!='X')return 0;
	return 1;
}
int main()
{
	n=read();
	for(int i=1;i<=n;i++)scanf("%s",mp[i]+1);
	int ans=0;
	for(int i=1;i<=n;i++)
	for(int j=1;j<=n;j++)
	ans+=check(i,j);
	printf("%d\n",ans);
	return 0;
}

```

## [B. Lunar New Year and Food Ordering](https://codeforces.com/contest/1106/problem/B)

$\ \ \ \ \ \ \,$这个也是模拟吧，我们把菜品排个序，用一个指针跳就好了吧……（敷衍
![在这里插入图片描述](https://img-blog.csdnimg.cn/20190215203257557.jpg)
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
const int N=1e5+10;
int n,m,z=1;
int rk[N];
struct node{int a,c,id;}di[N];
inline bool operator <(const node &a,const node &b)
{return a.c<b.c||(a.c==b.c&&a.id<b.id);}
int main()
{
	n=read();m=read();
	for(int i=1;i<=n;i++)di[i].a=read();
	for(int i=1;i<=n;i++)di[i].c=read();
	for(int i=1;i<=n;i++)di[i].id=i;
	sort(di+1,di+n+1);
	for(int i=1;i<=n;i++)rk[di[i].id]=i;
	while(m--){
		int kind=read(),cnt=read();
		if(di[rk[kind]].a>=cnt){
			di[rk[kind]].a-=cnt;
			printf("%lld\n",1ll*di[rk[kind]].c*cnt);
			continue;
		}
		long long ls=0ll;
		if(di[rk[kind]].a){
			cnt-=di[rk[kind]].a;
			ls+=1ll*di[rk[kind]].c*di[rk[kind]].a;
			di[rk[kind]].a=0;
		}
		while(cnt){
			if((!di[z].a)&&z<=n)z++;
			if(z>n){ls=0;break;}
			else{
				if(di[z].a>=cnt){
					di[z].a-=cnt;
					ls+=1ll*di[z].c*cnt;
					cnt=0;
				}
				else{
					cnt-=di[z].a;
					ls+=1ll*di[z].c*di[z].a;
					di[z].a=0;
				}
			}
		}
		printf("%lld\n",ls);
	}
	return 0;
}

```

## [C. Lunar New Year and Number Division](https://codeforces.com/contest/1106/problem/C)
$\ \ \ \ \ \ \,$根据二项式定理，当然是两个两个分为一组最合算了（$n$ 范围明示

$\ \ \ \ \ \ \,$我们展开可得：

$(a+b)^2=a^2+b^2+2ab$

$\ \ \ \ \ \ \,$那么我们就想要两个成积较小的分一组最好，就是排序过后，最小的和最大的分一组好了呀。

![在这里插入图片描述](https://img-blog.csdnimg.cn/20190215204003943.jpg)
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
const int N=3e5+10;
long long ans;
int a[N],n;
int main()
{
	n=read();
	for(int i=1;i<=n;i++)a[i]=read();
	sort(a+1,a+n+1);
	for(int i=1,j=n;i<=j;i++,j--)
	ans+=1ll*(a[i]+a[j])*(a[i]+a[j]);
	printf("%lld\n",ans);
	return 0;
}

```

## [D. Lunar New Year and a Wander](https://codeforces.com/contest/1106/problem/D)
$\ \ \ \ \ \ \,$BFS……

$\ \ \ \ \ \ \,$并不是，其实也差不多吧，当前可以走到的点，我们把他放进堆里面，然后每次走堆里最小的这个样子。

$\ \ \ \ \ \ \,$（因为题意没看懂翻车了几次

![在这里插入图片描述](https://img-blog.csdnimg.cn/20190215204454189.jpg?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dlaXhpbl80Mzk3Mzk2Ng==,size_16,color_FFFFFF,t_70)
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
const int N=1e5+10;
int n,m;
bool used[N];
vector<int> G[N];
priority_queue<int> Q;
int main()
{
	n=read();m=read();
	while(m--){
		int a=read(),b=read();
		G[a].push_back(b);
		G[b].push_back(a);
	}
	for(int i=1;i<=n;i++)
	sort(G[i].begin(),G[i].end());
	Q.push(-1);used[1]=1;
	while(!Q.empty()){
		int u=-Q.top();Q.pop();
		printf("%d ",u);
		for(auto v:G[u])if(!used[v])
		Q.push(-v),used[v]=1;
	}
	return 0;
}

```

## [E. Lunar New Year and Red Envelopes](https://codeforces.com/contest/1106/problem/E)

$\ \ \ \ \ \ \,$后面两道题就开始有讲的意思了，反正这道题我并没有独自写出来（我怀疑题都没有怎么读懂（我好菜呀
![在这里插入图片描述](https://img-blog.csdnimg.cn/20190215204926766.jpg)

$\ \ \ \ \ \ \,$题目做法是DP，我们定义$f_{i,j}$，表示被打扰了 $i$ 次，现在时间是 $j$ 的最小收益。转移方程呢就是：

$f_{i,j+1}=f_{i-1,j}$

$f_{i,a_l.d+1}=f_{i-1,k}+a_l.w$

$\ \ \ \ \ \ \,$我们预处理数当前时间用那个红包好，就可以降低复杂度到$O(nm)$，具体来说，就是哪个钱多哪个好，钱一样多的话就是哪个冷却时间长哪个好。具体操作看的[**这里**](https://blog.csdn.net/g21glf/article/details/86743023)，其实很多地方没有必要这么麻烦，但是自己确实是太菜了，没有自己独立做出来。
![在这里插入图片描述](https://img-blog.csdnimg.cn/20190215210921828.jpg)

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
int n,m,k;
struct node{int d,w,t;}sta[N];
inline bool operator <(const node &a,const node &b)
{return a.w>b.w||(a.w==b.w&&a.d>b.d);}
vector<node> e[N];
map<node,int> mp;
void Insert(node a){
	if(mp.count(a))mp[a]++;
	else mp[a]=1;
}
void Delete(node a){
	mp[a]--;
	if(!mp[a])mp.erase(a);
}
long long f[2][N],ans=(1ll<<62);
int main()
{
	n=read();m=read();k=read();
	for(int i=1,s,t,d,w;i<=k;i++){
		s=read(),t=read(),d=read(),w=read();
		e[s].push_back((node){d,w,1});
		e[t+1].push_back((node){d,w,-1});
	}
	for(int i=1;i<=n;i++){
		for(auto p:e[i])
			if(~p.t)Insert(p);
			else Delete(p);
		if(mp.size())sta[i]=(*mp.begin()).first;
		else sta[i]=(node){i,0,0};
	}
	memset(f,0x3f,sizeof(f));f[0][1]=0;
	int cas=1;
	for(int j=0;j<=m;j++){
		memset(f[cas],0x3f,sizeof(f[cas]));
		for(int i=1;i<=n;i++){
			f[cas][i+1]=min(f[cas][i+1],f[cas^1][i]);
			f[cas^1][sta[i].d+1]=min(f[cas^1][sta[i].d+1],f[cas^1][i]+sta[i].w);
		}
		ans=min(ans,f[cas^1][n+1]);cas^=1;
	}
	printf("%lld\n",ans);
	return 0;
}

```

## [F. Lunar New Year and a Recursive Sequence](https://codeforces.com/contest/1106/problem/F)

$\ \ \ \ \ \ \,$感觉这道题操作比E题麻烦一点，但是确实比E题好想呢。

![在这里插入图片描述](https://img-blog.csdnimg.cn/20190215213423624.jpg)

$\ \ \ \ \ \ \,$看到是一个有 $k$ 项的递推式，马上就可以想到矩乘，而前 $k-1$ 项已经确定了是 $1$，我们不妨设要求的 $f_k$ 为 $a$ 。根据他给的式子啊，我们就容易发现，这个递推式的每一项都应该是 $a^x$ 的形式，知道第 $n$ 项是 $a$ 的多少次方就要好处理一些了。

$\ \ \ \ \ \ \,$这样子稍微观察一下矩阵乘法就定义好了：

$\ \ \ \ \ \ \,$转移矩阵：$A=$

$
\begin{bmatrix}0&0&\cdots&0&b_k\\ 1&0&\cdots&0&b_{k-1}\\0&1&\cdots&0&b_{k-2}\\\vdots&\vdots&\ddots&\vdots&\vdots\\0&0&\cdots&1&b_1\end{bmatrix}
$

$\ \ \ \ \ \ \,$初始矩阵：$S=$

$
\begin{bmatrix}0,0,\cdots,0,1\end{bmatrix}
$

$\ \ \ \ \ \ \,$那么第 $n$ 项的指数，就是 $S\cdot A^{n-k}$ 的第 $k$ 项，矩阵乘法取模的时候，根据欧拉定理，因为模数是素数，直接每次模 $mod-1$ 就好了。

$\ \ \ \ \ \ \,$现在问题是，我们知道 $x$，$m$，$mod$，$a^x\%mod=m$，如何求 $a$ 呢？

$\ \ \ \ \ \ \,$好在他给我们的模数很特殊，我们很清楚他的原根为 $3$ ，那么我们可以重新把 $a$ 定义为 $3^s\%mod$，所以原式化为:

$3^{sx}\%mod=m$

$\ \ \ \ \ \ \,$我们可以很轻松用 BSGS 算法知道 $sx\%(mod-1)$的取值，而我们又知道 $x\%(mod-1)$ 的取值，扩展GCD处理一下就好咯~

![在这里插入图片描述](https://img-blog.csdnimg.cn/20190215213335919.jpg)

$\ \ \ \ \ \ \,$然后我们就知道 $s$ 的取值了（也有可能无解），那么答案也就出来了：$f_k=3^s$。

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
const int mod=998244353;
struct matrix{
	int x,y;
	int a[101][101];
};
int K,m,n;
matrix S,A,T;
matrix operator *(matrix m1,matrix m2){
	matrix t;t.x=m1.x;t.y=m2.y;
	for(int i=1;i<=m1.x;i++)
	for(int j=1;j<=m2.y;j++){
	  	t.a[i][j]=0;
	  	for(int k=1;k<=m1.y;k++)
	  	t.a[i][j]=(1ll*t.a[i][j]+1ll*m1.a[i][k]*m2.a[k][j])%(mod-1);
	}
	return t;
}
matrix power(matrix a,int b){
	matrix ans=a;b--;
	while(b){
		if(b&1ll)ans=ans*a;
		a=a*a;b>>=1;
	}
	return ans;
}
int power(int a,int b,int mod){
	int ans=1;
	while(b){
		if(b&1)ans=(1ll*ans*a)%mod;
		a=(1ll*a*a)%mod;
		b>>=1;
	}
	return ans;
}
long long BSGS(long long a,long long b,long long c){
  	map<int,int>hash;hash.clear();b%=c;
  	int t=(int)sqrt(c)+1;
  	for(int j=0;j<t;j++){
    	int val=(int)(b*power(a,j,c)%c);
    	hash[val]=j;
  	}
  	a=power(a,t,c);
  	if(a==0){
  		if(b==0)return 1;
    	else return -1;
  	}
  	for(int i=0;i<=t;i++){
    	int val=power(a,i,c);
    	int j=hash.find(val)==hash.end()?-1:hash[val];
    	if(j>=0&&i*t-j>=0)return i*t-j;
  	}
  	return -1;
}
void exgcd(long long a,long long b,long long &d,long long &x,long long &y){
	if(!b){d=a;x=1;y=0;return;}
	exgcd(b,a%b,d,y,x);y-=x*(a/b);
}
int main()
{
	K=read();S.x=1;
	A.x=A.y=S.y=K;S.a[1][K]=1;
	for(int i=2;i<=K;i++)A.a[i][i-1]=1;
	for(int i=K;i>=1;i--)A.a[i][K]=read();
	n=read();m=read();
	T=S*power(A,n-K);
	long long t=BSGS(3ll,1ll*m,mod);
	long long g,x,y;
	exgcd(T.a[1][K],mod-1,g,x,y);
  	if(t%g)puts("-1");
  	else{
   		x=(t/g*x%(mod-1)+mod-1)%(mod-1);
    	printf("%d\n",power(3,x,mod));
  	}
	return 0;
}

```

## 完结撒花
![在这里插入图片描述](https://img-blog.csdnimg.cn/20190215213506253.gif)