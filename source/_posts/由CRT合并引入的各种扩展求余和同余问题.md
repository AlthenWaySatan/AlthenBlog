---
title: '由CRT合并引入的各种扩展求余和同余问题'
date: 2018-12-30 12:32:09
categories:
- 学习笔记
tags:
- OI
- 数学
- CRT
- exCRT
- Lucas
- exLucas
- Euler
- exEuler
- BSGS
- exBSGS
- NTT
- 任意模数NTT

mathjax: true
---

$\ \ \ \ \ \ \ \,$扩展中国剩余定理的引入，使得在模特定素数意义下的算法得以扩张为模任意数，这大大方便了我们对于算法的运用。

<!-- more -->

## 大前提[【中国剩余定理】](/2018/12/29/扩展中国剩余定理/)

## 扩展卢卡斯定理（exLucas）
  ### 卢卡斯定理（Lucas）
  
  $\ \ \ \ \ \ \,$卢卡斯定理是快速求组合数余的算法，其中最基本的卢卡斯定理只能解决模数为素数的情况，复杂度是$O(p)$预处理阶乘和逆元，每次回答$O(\log n)$。
  
  $\ \ \ \ \ \ \,$当$n$或$m>p$，套用下面的公式递归处理：

  $C_{n}^{m}\equiv C_{\left\lfloor\frac{n}{P}\right\rfloor}^{\left\lfloor\frac{m}{P}\right\rfloor}\times C_{n\%P}^{m\%P}({\rm mod}\ P)$

  $\ \ \ \ \ \ \,$当$n$或$m\leq p$时，套用定义公式：
  
  $C_{n}^{m}=\frac{n!}{m!(n-m)!}({\rm mod}\ P)$
  
  $\ \ \ \ \ \ \,$那么当模数不为素数的情况下，我们使用中国剩余定理（CRT）合并。
  
  - 将$P$分解质因数：
$P=\prod_{i=1}^rp_i^{c_i}$
    
  - 得到同余方程：
$\begin{cases}x\equiv C_{n}^{m}\ \ ({\rm mod}\ p_1^{c_1})\\x\equiv C_{n}^{m}\ \ ({\rm mod}\ p_2^{c_2})\\x\equiv C_{n}^{m}\ \ ({\rm mod}\ p_3^{c_3})\\\ \ \ \ \ \ \ \ \ \ \ \cdots\\x\equiv C_{n}^{m}\ \ ({\rm mod}\ p_r^{c_r})\end{cases}$
    
$\ \ \ \ \ \ \,$显然可以很方便的用中国剩余定理（CRT）求出$x$的值，现在我们的问题是如何快速求出$C_{n}^{m}\% p_i^{c_i}$的值。
  
  $\ \ \ \ \ \ \,$先忘记卢卡斯定理吧，这里并用不到它。
  
  $\ \ \ \ \ \ \,$观察组合数的定义式：
  
  $C_{n}^{m}=\frac{n!}{m!(n-m)!}=\frac{\prod_{i=1}^ni}{\prod_{i=1}^mi\prod_{i=1}^{n-m}i}$
  
  $\ \ \ \ \ \ \,$现在的问题是如何快速求出$\prod_{i=1}^ni\ (\% p^{c})$的值。
  
  $\ \ \ \ \ \ \,$我们开始拆开它：
  
  $\prod_{i=1}^ni\ (\% p^{c})$
  
  $\prod_{i=1,p|i}^ni\prod_{i=1,p\nmid i}^ni\ (\% p^{c})$
  
  $\prod_{i=1}^{\left\lfloor\frac{n}{p}\right\rfloor}ip\prod_{i=1,p\nmid i}^ni\ (\% p^{c})$
  
  ${\left\lfloor\frac{n}{p}\right\rfloor}!\cdot p\prod_{i=1,p\nmid i}^ni\ (\% p^{c})$
  
  $\ \ \ \ \ \ \,$前面那一块阶乘我们递归处理，后面那一块嘛，我们可以发现他是有循环节的，而且长度最多为$p^{c}$，所以暴力算出循环节然后快速幂咯，至于中间那个$p$，我们最后再计算要方便不少。
  
  $\ \ \ \ \ \ \,$现在阶乘倒是处理完了，我们带回定义式，发现还需要求个逆元。这个还是扩展欧几里得来吧，然后我们发现还有很多$p$没有乘，需要统计一下里面乘$p$的次数再快速幂乘回去。
  
  $\ \ \ \ \ \ \,$说了这么多，终于有下面的模板了，复杂度大约在$O(P\log P)$左右：
  
``` cpp
long long inv(long long a,long long p){
  long long x,y;exgcd(a,p,x,y);
  return (x+p)%p==0?p:(x+p)%p;
}
long long fac(long long n,long long p,long long pc){
  if(n==1||n==0)return 1;
  long long res=1;
  for(long long i=2;i<=pc;i++)if(i%p)res=(res*i)%pc;
  res=power(res,n/pc,pc);
  for(long long i=2;i<=n%pc;i++)if(i%p)res=(res*i)%pc;
  return (res*fac(n/p,p,pc))%pc;
}
long long C(long long n,long long m,long long p,long long pc,long long Mod){
  if(n<m)return 0;
  long long a=fac(n,p,pc),b=fac(m,p,pc),c=fac(n-m,p,pc);
  long long k=0;
  for(long long i=n;i;i/=p)k+=i/p;
  for(long long i=m;i;i/=p)k-=i/p;
  for(long long i=n-m;i;i/=p)k-=i/p;
  long long res=a*inv(b,pc)%pc*inv(c,pc)%pc*power(p,k,pc)%pc;
}
long long exLucas(long long n,long long m,long long Mod){
  long long res=0,P=Mod;
  for(long long i=2;i<=P;i++)
    if(P%i==0){
      long long pc=1;
      while(P%i==0)P/=i,pc*=i;
      res=(res+C(n,m,i,pc,Mod)*(Mod/pc)%Mod*inv(Mod/pc,pc)%Mod)%Mod;
    }
  return res;
}

```

### [【P4720 【模板】扩展卢卡斯】](https://www.luogu.org/problemnew/show/P4720)
  $\ \ \ \ \ \ \,$既然是模板题，就不多废话了：
  
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
inline long long read(){
    long long x=0,f=1;char ch;ch=getchar();
    while(ch<'0'||ch>'9'){if(ch=='-') f=0;ch=getchar();}
    while(ch>='0'&&ch<='9'){x=(x<<1)+(x<<3)+(ch&15);ch=getchar();}
    if(f)return x;else return -x;
}
long long power(long long a,long long b,long long c){
  long long ans=1;a=a%c;
  for(;b;b>>=1,a=(a*a)%c)if(b&1)ans=(ans*a)%c;
  return ans;
}
void exgcd(long long a,long long b,long long &x,long long &y){
  if(!b){x=1ll;y=0ll;return;}
  exgcd(b,a%b,y,x);y-=x*(a/b);
}
long long inv(long long a,long long p){
  long long x,y;exgcd(a,p,x,y);
  return (x+p)%p==0?p:(x+p)%p;
}
long long fac(long long n,long long p,long long pc){
  if(n==1||n==0)return 1;
  long long res=1;
  for(long long i=2;i<=pc;i++)if(i%p)res=(res*i)%pc;
  res=power(res,n/pc,pc);
  for(long long i=2;i<=n%pc;i++)if(i%p)res=(res*i)%pc;
  return (res*fac(n/p,p,pc))%pc;
}
long long C(long long n,long long m,long long p,long long pc,long long Mod){
  if(n<m)return 0;
  long long a=fac(n,p,pc),b=fac(m,p,pc),c=fac(n-m,p,pc);
  long long k=0;
  for(long long i=n;i;i/=p)k+=i/p;
  for(long long i=m;i;i/=p)k-=i/p;
  for(long long i=n-m;i;i/=p)k-=i/p;
  return (a*inv(b,pc)%pc*inv(c,pc)%pc*power(p,k,pc)%pc)*(Mod/pc)%Mod*inv(Mod/pc,pc)%Mod;
}
long long exLucas(long long n,long long m,long long Mod){
  long long res=0,P=Mod;
  for(long long i=2;i<=P;i++)
    if(P%i==0){
      long long pc=1;
      while(P%i==0)P/=i,pc*=i;
      res=(res+C(n,m,i,pc,Mod))%Mod;
    }
  return res;
}
int main()
{
  long long n=read(),m=read(),P=read();
  printf("%lld\n",exLucas(n,m,P));
  return 0;
}

```

## 扩展欧拉算法（exEuler）

  ### 欧拉算法（Euler）

$\ \ \ \ \ \,$欧拉算法是快速求$a^b\%P$的值的算法，其核心是欧拉定理：

$\ \ \ \ \ \,$当$a$,$P$互质时,有 $a^{\varphi(P)}\equiv1({\rm mod}\ P)$

$\ \ \ \ \ \,$那么就有，欧拉算法：$a^b\%P=a^{b\%{\varphi(P)}}\%P$

$\ \ \ \ \ \,$当模数不为素数的情况下，我们依然使用中国剩余定理（CRT）合并。
  
  - 将$P$分解质因数：
$P=\prod_{i=1}^rp_i^{c_i}$
    
  - 得到同余方程：
$\begin{cases}x\equiv a^b\ \ ({\rm mod}\ p_1^{c_1})\\x\equiv a^b\ \ ({\rm mod}\ p_2^{c_2})\\x\equiv a^b\ \ ({\rm mod}\ p_3^{c_3})\\\ \ \ \ \ \ \ \ \ \ \ \cdots\\x\equiv a^b\ \ ({\rm mod}\ p_r^{c_r})\end{cases}$
  
  $\ \ \ \ \ \ \,$好吧其实这个结论挺简单的，证明比较麻烦，所以我们就开心地记结论吧，[证明在这里哦](https://blog.csdn.net/synapse7/article/details/19610361)。
  
  $a^b\%P=\begin{cases}a^{b\%{\varphi(P)}}\%P, & {[\gcd(a,P)=1]}\\a^{b\%{\varphi(P)}+\varphi(P)}\%P, & {[\gcd(a,P)\neq 1,b>\varphi(P)]}\\a^b\%P, & {[\gcd(a,P)\neq 1,b\leq\varphi(P)]}\end{cases}$

``` cpp
int Euler_Power(int a,int b,int n){
  if(n==1)return 0;
  if(gcd(a,n)==1)return power(a,b%phi[n],n);
  if(b>phi[n])return power(a,b%phi[n]+phi[n],n)%n;
  if(b<=phi[n])return power(a,b,n);
}
```

### [【P4139 上帝与集合的正确用法】](https://www.luogu.org/problemnew/show/P4139)
  $\ \ \ \ \ \ \,$这道题是只用了欧拉算法呢，我们不断递归过去，一定会遇到指数为$1$的时候，退出就行了：
  
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
const int N=1e7+10;
int phi[N];
void phi_table(int n){
  for(int i=2;i<=n;i++)phi[i]=0;
  phi[1]=1;
  for(int i=2;i<=n;i++)if(!phi[i])
  for(int j=i;j<=n;j+=i){
    if(!phi[j])phi[j]=j;
    phi[j]=phi[j]/i*(i-1);
  }
}
int gcd(int a,int b){return b?gcd(b,a%b):a;}
long long power(long long a,long long b,long long mod){
  long long ans=1ll;
  while(b){
    if(b&1)ans=(1ll*ans*a)%mod;
    a=(1ll*a*a)%mod;b>>=1;
  }
  return ans;
}
int solve(int mod){
  if(mod==1)return 0;
  return power(2,solve(phi[mod])+phi[mod],mod);
}
int main()
{
  phi_table(N-10);
  int T=read();
  while(T--){
    int mod=read();
    printf("%d\n",solve(mod));
  }
  return 0;
}

```

## 扩展大步小步算法（exBSGS）

  ### 大步小步算法（BSGS）
    
$\ \ \ \ \ \ \,$大步小步算法，是求解指数方程同余最小自然数解的算法，其中必须保证模数与$a$互质,形如:
    
$a^x\equiv b\ \ ({\rm mod}\ P)$
    
$\ \ \ \ \ \ \,$其实想法还是挺简单的，因为$p$是素数，所以我们可以知道解 $x<P$，那么我们$\sqrt P$ 分块。
    
$\ \ \ \ \ \ \,$我们不妨假设我们的答案$x=k\sqrt P+l$，显然 $l<\sqrt P$，$k<\sqrt P$：
    
$a^x\equiv b\ \ ({\rm mod}\ P)$
    
$a^{k\sqrt p+l}\equiv b\ \ ({\rm mod}\ P)$
$(a^{\sqrt p})^{k}a^l\equiv b\ \ ({\rm mod}\ P)$
    
$\left(a^{\sqrt p}\right)^{k}\equiv \left(a^l\right)^{-1}b\ \ ({\rm mod}\ P)$
    
$\left(a^{\sqrt p}\right)^{k}\equiv a^{-l}b\ \ ({\rm mod}\ P)$
    
$\ \ \ \ \ \ \,$那么我们枚举一个 $k$,就可以确定那个唯一的 $l$了,前提是我们要把所有$a^{-l}b\ \ ({\rm mod}\ P)$预处理出来，当然了，这不是问题，我们可以用$\rm map$存一下，复杂度$O(\sqrt P)$：

    
  ``` cpp
  long long BSGS(long long a,long long b,long long p){
    map<int,int>hash;hash.clear();b%=p;
    int t=(int)sqrt(p)+1;
    for(int j=0;j<t;j++){
      int val=(int)(b*power(a,j,p)%p);
      hash[val]=j;
    }
    a=power(a,t,p);
    if(a==0){
  	  if(b==0)return 1;
      else return -1;
    }
    for(int i=0;i<=t;i++){
      int val=power(a,i,p);
      int j=hash.find(val)==hash.end()?-1:hash[val];
      if(j>=0&&i*t-j>=0)return i*t-j;
    }
    return -1;
  }

  ```
  
  $\ \ \ \ \ \ \,$那么当模数与$a$不互质的情况下，我们依然使用中国剩余定理（CRT）合并。
    
  $\ \ \ \ \ \ \,$由扩展欧拉定理可以得到，当$\gcd (a,P)\nmid b$并且$b\neq1$的时候，方程是无解的，我们先把他判掉。
  
  $\ \ \ \ \ \ \,$首先我们的想法是，如何一步一步把它化简成模数与$a$互质，再BSGS就行了：
    
  $a^x\equiv b\ \ ({\rm mod}\ P)$
  
  $\frac{a^x}{\gcd (a,P)} \equiv \frac{b}{\gcd (a,P)}\ \ ({\rm mod}\ \frac{P}{\gcd (a,P)})$
  
  ${a^{x-1}} \frac{a}{\gcd (a,P)}\equiv \frac{b}{\gcd (a,P)}\ \ ({\rm mod}\ \frac{P}{\gcd (a,P)})$
  
  ${a^{x-1}} \equiv \frac{b}{a}\ \ ({\rm mod}\ \frac{P}{\gcd (a,P)})$
  
  $\ \ \ \ \ \ \,$如此递归下去，化简成模数与$a$互质，再BSGS，返回值记住加上递归层数：~~（- 然而并没有用到CRT啊，它是怎么混进来的？？- ex嘛，一样的）~~
  
``` cpp
long long exBSGS(long long a,long long b,long long p){
	if(p==1)return 0;a%=p,b%=p;
	if(b==1)return 0;
  if(a==0){
  	if(b==0)return 1;
    else return -1;
  }
	int k=0,d=1;
  for(int g=gcd(a,p);g!=1;g=gcd(a,p)){
    if(b%g) return -1;
    p/=g;b/=g;d=d*(a/g)%p;k++;
    if(d==b) return k;
  }
  a%=p;
	map<int,int>hash;hash.clear();
	int t=(int)sqrt(p)+1;
  for(int j=0;j<t;j++){
    int val=(int)(b*power(a,j,p)%p);
    hash[val]=j;
  }
  a=power(a,t,p);
  for(int i=1;i<=t;i++){
    int val=d*power(a,i,p)%p;
    int j=hash.find(val)==hash.end()?-1:hash[val];
    if(j>=0&&i*t-j+k>=0)return i*t-j+k;
  }
  return -1;
}
```
### [【P4195 【模板】exBSGS/Spoj3105 Mod】](https://www.luogu.org/problemnew/show/P4195)
  $\ \ \ \ \ \ \,$模板题，不废话：
  
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
int gcd(int a,int b){return b?gcd(b,a%b):a;}
int power(int a,int b,int mod){
	int ans=1ll;
	for(;b;a=1ll*a*a%mod,b>>=1)
	if(b&1)ans=1ll*ans*a%mod;
	return ans;
}
const int mod=100208;
struct HaHashsh{
	int head[mod],p;
	struct ss{int v;int w,last;}G[100000];
	void clear(){memset(head,0,sizeof(head));p=0;}
	int find(int a){
		int A=a%mod;
		for(int i=head[A];i;i=G[i].last)
		if(G[i].v==a)return G[i].w;
		return -1;
	}
	void insert(int a,int b){
		int A=a%mod;
		G[++p]=(ss){a,b,head[A]};head[A]=p;
	}
}hash;
int exBSGS(int a,int b,int c){
	if(c==1)return 0;
	a%=c,b%=c;
	if(b==1)return 0;
	if(a==0)return (b==0)?1:-1;
	int k=0,d=1;
	for(int g=gcd(a,c);g^1;g=gcd(a,c)){
		if(b%g)return -1;
		c/=g;b/=g;d=1ll*d*(a/g)%c;k++;
		if(d==b)return k;
	}
	a%=c;
	hash.clear();
	int t=(int)sqrt(c)+1;
	int val=b;
	for(int j=0;j<t;j++)
	hash.insert(val,j),val=1ll*val*a%c;
	val=a=power(a,t,c);val=1ll*d*val%c;
	for(int i=1,j;i<=t;i++){
		j=hash.find(val);val=1ll*val*a%c;
		if((~j)&&1ll*i*t-j+k>=0)return 1ll*i*t-j+k;
	}
	return -1;
}
int a,b,c,ans;
int main()
{
	while(scanf("%d%d%d",&a,&c,&b)==3&&(a||b||c))
	if(~(ans=exBSGS(a,b,c)))cout<<ans<<endl;
	else cout<<"No Solution"<<endl;
	return 0;
}

```

## 任意模数NTT

  $\ \ \ \ \ \ \,$在[【求多项式卷积的变换】](/2018/12/29/求多项式卷积的变换/)中我们知道，一般NTT只在模数为NTT模数的时候才合法。那么当模数为任意数的时候怎么办呢？
  
  **$\ \ \ \ \ \ \,$ 一种解法，3模NTT**
  
  $\ \ \ \ \ \ \,$假设在求卷积过后，我们的系数最大为 $A_{max}$，那么我们找若干个NTT模数，求一次NTT变换过后，我们对于这一项，可以得到一个同余方程组：
  
   $\begin{cases}A_{max}\equiv A_x\ \ ({\rm mod}\ p_1)\\A_{max}\equiv A_x\ \ ({\rm mod}\ p_2)\\A_{max}\equiv A_x\ \ ({\rm mod}\ p_3)\\\ \ \ \ \ \ \ \ \ \ \ \cdots\\A_{max}\equiv A_x\ \ ({\rm mod}\ p_r)\end{cases}$
   
   $\ \ \ \ \ \ \,$显然，当$\prod_{i-1}^rp_r\geq P$的时候，可以解出$A_{max}$的具体值。这个时候用CRT合并，再模$P$就出来了。
   
   $\ \ \ \ \ \ \,$下面还是给出模板，这里取的NTT模数为$469762049$，$998244353$，$1004535809$，他们的原根都是 $3$：
   
``` cpp
const long long p1=469762049ll,p2=998244353ll,p3=1004535809ll;
void NTT(long long *a,int f,int n,int mod){
  for(int i=0;i<n;i++)if(i<R[i])swap(a[i],a[R[i]]);
  for(register int i=1;i<n;i<<=1){
    int gn=power(3,(mod-1)/(i<<1),mod);
    for(register int j=0;j<n;j+=(i<<1)){
      int g=1,x,y;
      for(register int n=0;n<i;++n,g=1ll*g*gn%mod){
        x=a[j+n],y=1ll*g*a[j+n+i]%mod;
        a[j+n]=(x+y)%mod;a[j+n+i]=(x-y+mod)%mod;
      }
    }
  }
	if(f==-1){
    reverse(a+1,a+n);
    int inv=power(n,mod-2,mod);
    for(register int i=0;i<n;i++)a[i]=1ll*a[i]*inv%mod;
  }
}
void merge_ntt(long long *a,long long *b,int n,int mod){
  NTT(a,1,n,mod);NTT(b,1,n,mod);
  for(register int i=0;i<=n;i++)a[i]=1ll*a[i]*b[i]%mod;
}
void mcpy(int d){
  for(int i=0;i<=n;i++) ans[d][i]=lsa[i];
  if(d==2) return;
  memset(lsa,0,sizeof(lsa));memset(lsb,0,sizeof(lsb));
  for(int i=0;i<=n;i++) lsa[i]=a[i];
  for(int i=0;i<=m;i++) lsb[i]=b[i];
}
int ex_merge_NTT(long long *a,long long *b,int la,int lb,int p){
	n=la,m=lb;
  for(int i=0;i<=la;i++) lsa[i]=a[i];
  for(int i=0;i<=lb;i++) lsb[i]=b[i];
	int L=0;for(m+=n,n=1;n<=m;n<<=1)L++;
  for(register int i=0;i<n;i++)
  R[i]=(R[i>>1]>>1)|((i&1)<<(L-1));
  merge_ntt(lsa,lsb,n,p1);mcpy(0);
  merge_ntt(lsa,lsb,n,p2);mcpy(1);
  merge_ntt(lsa,lsb,n,p3);mcpy(2);
  NTT(ans[0],-1,n,p1);
  NTT(ans[1],-1,n,p2);
  NTT(ans[2],-1,n,p3);
  for(int i=0;i<=la+lb;i++){
    long long x=((mul(1ll*ans[0][i]*p2%(p1*p2),power(p2%p1,p1-2,p1),(p1*p2)))+(mul(1ll*ans[1][i]*p1%(p1*p2),power(p1%p2,p2-2,p2),(p1*p2))))%(p1*p2);
    long long y=((((ans[2][i]-x)%p3+p3)%p3)*power((p1*p2)%p3,p3-2,p3))%p3;
    a[i]=(1ll*(y%p)*((p1*p2)%p)%p+x%p)%p;
  }
  return la+lb;
}

```