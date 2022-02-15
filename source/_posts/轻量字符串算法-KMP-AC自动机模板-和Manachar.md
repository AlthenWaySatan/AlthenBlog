---
title: '轻量字符串算法——KMP（AC自动机模板）和Manachar'
date: 2018-12-29 14:28:15
categories:
- 学习笔记
tags:
- OI
- 字符串
- 数据结构
- KMP
- AC自动机
- Manachar
mathjax: true
---

$\ \ \ \ \ \ \ \,$一些简单字符串相关算法的复习笔记：

<!-- more -->

## KMP

$\ \ \ \ \ \ \,$kmp是用来处理字符串匹配的常见简单算法，网上可以找到很多讲解，这里就不细讲了，一笔带过。

$\ \ \ \ \ \ \,$我们知道，暴力匹配两个字符串的复杂度是$O(n^2)$的，很多时候我们都不能接受这个复杂度，考虑如何减小复杂度，我们发现在暴力匹配的过程中，会重复匹配很多地方，所以我们从这里下手，进行优化。

$\ \ \ \ \ \ \,$引入kmp算法最核心的东西—— $ next$ 数组：

$\ \ \ \ \ \ \,$代表当前字符之前的字符串中，有多大长度的相同前缀。例如如果 $next [j] = k$，代表位置 $j$ 之前的字符串中有最大长度为 $k $ 的相同前缀。

$\ \ \ \ \ \ \,$意味着在某个字符失配时，告诉你下一步匹配中，模式串应该跳到哪个位置（$next [j]$ ）。如果 $next [j]$ 等于$-1$，则跳到模式串的开头字符，若 $next [j] = k$ 且 $k > 0$，代表下次匹配跳到 $j$ 之前的某个字符，而不是跳到开头，跳过了 $k$ 个曾经匹配过的字符。

$\ \ \ \ \ \ \,$为什么这 $k$ 个字符就这样逃过了？我们可以这样感性地理解：

$\ \ \ \ \ \ \,$若是在 $j$ 这个位置失配，那么说明在这个位置之前，模式串和文本串是可以匹配的，也就是一样的，那么我们要是可以预处理下次跳到的地方就好了，这个就是我们预处理的结果：$ next$ 数组。

$\ \ \ \ \ \ \,$这样我们的匹配复杂度就降为$O(n)$：

$\ \ \ \ \ \ \,$假设现在文本串$S$匹配到 $i$ 位置，模式串$P$匹配到 $j$ 位置

- 如果$j = -1$，或者当前字符匹配成功（即$S[i] = P[j]$），$i$，$j$都加一，继续匹配下一个字符；

- 如果$j \neq -1$，且当前字符匹配失败（即$S[i] \neq P[j]$），则令 $i$ 不变，$j = next[j]$。此举意味着失配时，模式串P相对于文本串S向右移动了$j - next [j]$ 位。

``` cpp
while(i<slen&&j<plen){
  if(j==-1||s[i]==p[j])i++,j++;
  else j=Next[j];
  if(j==plen)j=Next[j];//到这里就匹配到了一个模式串了。
}

```

$\ \ \ \ \ \ \,$那么我们如何快速地求出$ next$ 数组？

$\ \ \ \ \ \ \,$这个过程相当于自己与自己匹配，假设现在对于字符串$p$，已经处理到了 $k$ 位置，和自己匹配到了 $j$ 位置（显然$k>j$）：

- 如果$j>0$，并且$p[k]\neq p[j]$，那么就是和自己失配了，$j = next[j]$；

- 如果$p[k]= p[j]$，那么就是是适配了，$k$，$j$都加一，同时如果下次匹配的时候在 $k+1$ 处失配了，那么我们就跳过枚举前面 $k$ 个元素，直接匹配 $j+1$，所以 $next[k+1]=j+1$。

``` cpp
int j=0;next[0]=-1;
for(int k=1;k<n;k++){
  while(j>0&&(p[k]!=p[j])) j=next[j];
  j+=(p[k]==p[j]);next[k+1]=j;
}
```
$\ \ \ \ \ \ \,$复杂度$O(n)$。

$\ \ \ \ \ \ \,$单字符串匹配的话，就是模板题不说了，kmp最重要的，还是对$ next$ 数组的运用。~~（多字符串匹配我还是信仰Sam，XD）~~，还是贴一个AC自动机的模板XD：

``` cpp
struct AC_Automata{
	int son[26][N],fail[N],appear[N];
	int size;
	void Get_trie(char s[],int id){
		int len=strlen(s),now=0;
		for(int i=0;i<len;i++){
			if(!son[s[i]-'a'][now])son[s[i]-'a'][now]=++size;
			now=son[s[i]-'a'][now]; 
		}
		appear[now]=id;
	}
	void Get_fail(){
		queue<int> Q;
		for(int i=0;i<26;i++)if(son[i][0]!=0)
		fail[son[i][0]]=0,Q.push(son[i][0]);
		while(!Q.empty()){
			int u=Q.front();Q.pop();
			for(int i=0;i<26;i++)
			if(son[i][u])fail[son[i][u]]=son[i][fail[u]],Q.push(son[i][u]);
			else son[i][u]=son[i][fail[u]];
		}
	}
	int query(char s[]){
		int len=strlen(s),now=0;
    for(int i=0;i<len;i++){
      now=son[s[i]-'a'][now];
      for(int t=now;t;t=fail[t])
			//something about appear[t];
    }
	}
};
```

### [P3193 [HNOI2008]GT考试](https://www.luogu.org/problemnew/show/P3193)

$\ \ \ \ \ \ \,$很明显的，我们会得到一个DP方程式：

$\ \ \ \ \ \ \,$我们令$f(i,j)$表示我们$X$已经处理到了$i$，其中出现了长度为$j$的连续不吉利数字。

$\ \ \ \ \ \ \,$答案显然就是$\sum_{i=0}^{m-1}f(n,i)$，现在考虑如何转移。

$\ \ \ \ \ \ \,$令$g(i,j)$表示，在长度为$j$的连续不吉利数字后，跟上数字$j$后，不吉利数字的长度。

$\ \ \ \ \ \ \,$那么就有：

$f(i,j)=\sum_{b=0}^{9} f(i,a)[j=g(a,b)]$

$\ \ \ \ \ \ \,$其中$g(a,b)$就可以用kmp的$ next$ 数组找到：

``` cpp
int g(int j,int k){
  while(j!=-1&&a[j]!=k)j=next[j];
  return j+1;
}
```

$\ \ \ \ \ \ \,$我们可以花$O(10\cdot m^2)$的时间把$g$预处理出来，如何$O(nm)$来dp，但是$n$特别大，于是我们用矩阵来优化，完整代码如下：

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
const int N=30;
int n,m,mod;
char s[N];
int a[N];
int f[N][N],ans;
struct Matrix{int n,m;int a[N][N];}A,B;
inline Matrix operator *(const Matrix &a,const Matrix &b){
  Matrix ret;
  ret.n=a.n;ret.m=b.m;
  for(int i=0;i<=a.n;i++)
  for(int j=0;j<=b.m;j++){
    ret.a[i][j]=0;
    for(int k=0;k<=a.m;k++)
    ret.a[i][j]=(ret.a[i][j]+a.a[i][k]*b.a[k][j]%mod)%mod;
  }
  return ret;
}
int next[N];
int g(int j,int k){
  while(j!=-1&&a[j+1]!=k)j=next[j];
  return j+1;
}
int main()
{
  n=read();m=read();mod=read();
  scanf("%s",s+1);
  for(int i=1;i<=m;i++)a[i]=s[i]-'0';
  next[0]=-1;
  for(int j=-1,i=1;i<=m;next[i++]=++j)
  while(j!=-1&&a[i]!=a[j+1])j=next[j];
  A.n=0;A.m=m-1;
  B.m=B.n=m-1;
  A.a[0][0]=1;
  for(int i=0;i<m;i++)
  for(int j=0;j<=9;j++)
  B.a[i][g(i,j)]++;
  for(;n;n>>=1,B=B*B)if(n&1)A=A*B;
  for(int i=0;i<m;i++) ans=(ans+A.a[0][i])%mod;
  printf("%d\n",ans);
  return 0;
}

```


## Manachar

$\ \ \ \ \ \ \,$manachar算法特别单一，就是求回文串用的，题一般也特别裸，比较套路。

$\ \ \ \ \ \ \,$首先，我们知道的，回文串分奇偶，但是如果我们在这个串中每一个字符之间都插入一个特殊字符，那么偶回文串就变成奇回文串了，这样我们就可以只处理奇回文串就行了。

$\ \ \ \ \ \ \,$然后就是$RL$数组，表示以这个字符为中点，回文串的最大半径是多少，manachar算法就是$O(n)$求$RL$数组的算法。下面直接将做法，不讲原理了：

$\ \ \ \ \ \ \,$记我们已经处理的回文串已经处理到的最右端为$MR$，他的对称轴为$pos$，现在要处理的位置为$i$，显然$pos<i$。

$\ \ \ \ \ \ \,$如果$MR>i$，那么我们可以确定，已$i$为中点，回文串的最大半径至少是${\rm min}(RL[2\cdot pos-i],MR-i)$。

$\ \ \ \ \ \ \,$剩下的就是暴力扩展，记得到时候更新一下$MR$和$pos$，模板如下：


``` cpp
int RL[N<<1];
void Manacher(char *s,int n){
  int MR=0,pos=0;
  memset(RL,0,sizeof(RL));
  for(int i=0;i<n;i++){
	if(MR>i)RL[i]=min(RL[2*pos-i],MR-i);
	else RL[i]=1;
    while(s[i+RL[i]]==s[i-RL[i]])RL[i]++;
    if(i+RL[i]>MR)MR=i+RL[i],pos=i;
  }
}
void insert(char *s,int len){
  len=strlen(s);P[0]='$';
  for(int i=1;i<=len*2+1;i+=2) P[i]='#';
  for(int i=0;i<len;i++) P[i*2+2]=s[i];
}
```

### [P4555 [国家集训队]最长双回文串](https://www.luogu.org/problemnew/show/P4555)

$\ \ \ \ \ \ \,$在我们求好$RL$后，很明显的有一个dp，我们把对称轴上的信息移动到起始点和终点：

``` cpp
for(int i=0;i<n;i++){
  L[i+RL[i]-1]=max(L[i+RL[i]-1],RL[i]-1);
  R[i-RL[i]+1]=max(R[i-RL[i]+1],RL[i]-1);
}
```

$\ \ \ \ \ \ \,$那么，显然的，答案等于：
${\rm Max}_{i=0}^{n-1}L_i+R_i[L_i\neq0,R_i\neq0]$

$\ \ \ \ \ \ \,$代码如下：
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
int RL[N<<1],n;
char s[N],P[N<<1];
int L[N<<1],R[N<<1];
void Manacher(char *s,int n){
  int MR=0,pos=0;
  for(int i=0;i<n;i++){
    if(MR>i)RL[i]=min(RL[2*pos-i],MR-i);
    else RL[i]=1;
    while(s[i+RL[i]]==s[i-RL[i]])RL[i]++;
    if(i+RL[i]>MR)MR=i+RL[i],pos=i;
    L[i+RL[i]-1]=max(L[i+RL[i]-1],RL[i]-1);
  	R[i-RL[i]+1]=max(R[i-RL[i]+1],RL[i]-1);
  }
}
void insert(char *s,int len){
  len=strlen(s);P[0]='$';
  for(int i=1;i<=len*2+1;i+=2) P[i]='#';
  for(int i=0;i<len;i++) P[i*2+2]=s[i];
}
int main()
{
  scanf("%s",s);n=strlen(s);
  insert(s,n);
  Manacher(P,n*2+2);
  for(int i=1;i<=n*2+1;i+=2)R[i]=max(R[i],R[i-2]-2);
  for(int i=n*2+1;i>=1;i-=2)L[i]=max(L[i],L[i+2]-2);
  int ans=0;
  for(int i=1;i<=2*n+1;i+=2)
  if(R[i]&&L[i])ans=max(ans,L[i]+R[i]);
  printf("%d\n",ans);
  return 0;
}

```
