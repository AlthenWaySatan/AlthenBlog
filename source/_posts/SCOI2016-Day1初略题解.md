---
title: '【SCOI2016】Day1初略题解'
date: 2019-02-13 16:41:29
categories:
- 题解
tags:
- OI
- 并查集
- Trie树
- 数据结构
- 异或
- 线性基
mathjax: true
---
$\ \ \ \ \ \,$做一套省选题来练练手（Day1）。

<!-- more -->


## [【T1 背单词】](https://www.luogu.org/problemnew/show/P3294)

### 解法

$\ \ \ \ \ \,$ 由题意可以得到，如果存在一个单词是它的后缀，并且当前没有被填入表内，那他需要吃 n*n 颗泡椒，这显然是很不合算的。要优先填入后缀。

$\ \ \ \ \ \,$第一个问题是如何找后缀，我们可以把串反过来插入 $Trie$ 树，然后按照$Trie$ 树上的父子关系新建树，按照一种神奇的 $DFS$ 序来依次填入。

$\ \ \ \ \ \,$ 现在的问题是如何处理这个 $DFS$ 序，手玩一点小样例可以发现，优先走子树小的较优，现在我们得到了填入顺序，每个点的填入序号减去他父亲的和，就是我们的答案，复杂度 $O(|len|+3n)$

### 代码

``` cpp
#include<queue>
#include<string>
#include<cstdio>
#include<cstring>
#include<iostream>
#include<algorithm>
using namespace std;
typedef long long ll;
inline int read(){
	int x=0,p=1;char c=getchar();
	while(c<'0'||c>'9'){if(c=='-') p=-1;c=getchar();}
	while(c<='9'&&c>='0'){x=(x<<1)+(x<<3)+(c&15);c=getchar();}
	return x*p;
}
const int N=510010;
int son[N][26],appear[N];
int id[N],cnt2,rt,cnt,n,size[N];
char s[N];
vector<int> a[100010];
bool cmp(int x,int y){return size[x]<size[y];}
void insert(char *s,int id){
	int len=strlen(s+1);int now=rt,v;
	for(int i=len;i;i--){
		if(!son[now][s[i]-'a'])son[now][s[i]-'a']=++cnt;
		now=son[now][s[i]-'a'];
	}
	appear[now]=id;
}
void dfs1(int rt,int fa){
	if(appear[rt]){a[fa].push_back(appear[rt]);fa=appear[rt];}
	for(int i=0;i<26;i++)if(son[rt][i])
	id[son[rt][i]]=id[rt],dfs1(son[rt][i],fa);
}
void dfs2(int rt){
	size[rt]=1;
	for(int i=0;i<a[rt].size();i++)
	dfs2(a[rt][i]),size[rt]+=size[a[rt][i]];
}
long long ans;
void get_ans(int rt){
	id[rt]=++cnt2;
	sort(a[rt].begin(),a[rt].end(),cmp);
	for(int i=0;i<a[rt].size();i++)
	ans+=cnt2+1-id[rt],get_ans(a[rt][i]);
}
int main(){
	freopen("word.in","r",stdin);
	freopen("word.out","w",stdout);
	n=read();
	for(int i=1;i<=n;i++)
	scanf("%s",s+1),insert(s,i);
	dfs1(0,0);
	dfs2(0);
	get_ans(0);
	printf("%lld\n",ans);
	fclose(stdin);
	fclose(stdout);
	return 0;
}

```

## [【T2 幸运数字】](https://www.luogu.org/problemnew/show/P3292)

### 解法

$\ \ \ \ \ \,$ 一眼可以看出来是实数异或线性基，不会这个的话就完全不能做这道题的。

$\ \ \ \ \ \,$下面的问题是，如何高效地处理路径问题？

$\ \ \ \ \ \,$我第一个想到的是树链剖分，复杂度为 $O(Q \log^2n)$，算上线性基合并的复杂度 $60\times 60$ 的常数有点吃不消。询问次数很大，我们可以试着多预处理一点。

$\ \ \ \ \ \,$倍增的复杂度为$O(Q \log n)$，加上一些蜜汁卡常数技巧，比如说去掉结构体啊什么的，就过了，空间还算充裕，没有那么卡。代码比较丑：

### 代码

``` cpp
#include<queue>
#include<string>
#include<cstdio>
#include<cstring>
#include<iostream>
#include<algorithm>
using namespace std;
typedef long long ll;
inline long long read(){
	long long x=0,p=1;char c=getchar();
	while(c<'0'||c>'9'){if(c=='-') p=-1;c=getchar();}
	while(c<='9'&&c>='0'){x=(x<<1)+(x<<3)+(c&15);c=getchar();}
	return x*p;
}
const int N=20010;
int n,m;
long long V[N],res;
int head[N],p;
struct ss{int last,v;}G[N<<1];
void add(int a,int b){
	G[++p]=(ss){head[a],b};head[a]=p;
	G[++p]=(ss){head[b],a};head[b]=p;
}
long long lb[N][19][61],ans[61];
void insert(long long *a,long long x){
	for(register int i=60;i>=0;i--)
	if(x&(1ll<<i))
	if(!a[i]){a[i]=x;return;}
	else x^=a[i];
}
long long querymax(long long *a){
	long long res=0;
	for(register int i=60;i>=0;--i)
	if((res^a[i])>res)res^=a[i];
	return res;
}
void merge(long long *a,long long *b)
{for(register int i=0;i<=60;i++)if(b[i])insert(a,b[i]);}
int fa[N][19],deep[N];
void dfs(int a,int f){
	fa[a][0]=f;deep[a]=deep[f]+1;
	for(register int i=head[a];i;i=G[i].last)
	if(G[i].v!=f)dfs(G[i].v,a);
}
void query(int a,int b){
	memset(ans,0,sizeof(ans));
	if(deep[a]>deep[b])swap(a,b);
	for(register int i=18;i>=0;i--)
	if(deep[a]<=deep[b]-(1<<i))
    merge(ans,lb[b][i]),b=fa[b][i];
	if(a==b){merge(ans,lb[a][0]);return ;}
	for(register int i=18;i>=0;i--)if(fa[a][i]!=fa[b][i]){
  		merge(ans,lb[a][i]),merge(ans,lb[b][i]);
    	a=fa[a][i],b=fa[b][i];
	}
	merge(ans,lb[a][0]),merge(ans,lb[b][0]);
	merge(ans,lb[fa[a][0]][0]);
	return ;
}
int main()
{
	freopen("lucky.in","r",stdin);
	freopen("lucky.out","w",stdout);
	n=(int)read();m=(int)read();
	for(register int i=1;i<=n;i++)V[i]=read(),insert(lb[i][0],V[i]);
	for(register int i=1,a,b;i<n;i++)
    a=read(),b=read(),add(a,b);
    dfs(1,0);
    for(register int j=1;j<=18;j++)
    for(register int i=1;i<=n;i++){
    	fa[i][j]=fa[fa[i][j-1]][j-1];
  		memcpy(lb[i][j],lb[i][j-1],sizeof(lb[i][j-1]));
    	merge(lb[i][j],lb[fa[i][j-1]][j-1]);
	}
  	for(register int i=1,a,b;i<=m;i++){
    	a=(int)read();b=(int)read();
    	query(a,b);
    	printf("%lld\n",querymax(ans));
	}
	return 0;
}

```

## [【T3 萌萌哒】](https://www.luogu.org/problemnew/show/P3295)

### 解法

$\ \ \ \ \ \,$ 这道题前$30$分做法很显然吧，两段相关联的各个元素，我们可以用并查集并在一起，因为他们必须一定是同一种。最后统计并查集个数就好了，设个数为$cnt$，那么我们的答案是$9\times 10^{cnt-1}$，因为包含第一个数位的只有$1$-$9$，其他可以取$0$-$9$。复杂度$O(n^2)$

$\ \ \ \ \ \,$ 我们可以通过类似于势能分析来操作，来降低复杂度到$O(n\log n)$，操作很骚，一开始没想到，也没有在其他地方见到过。

### 代码

``` cpp
#include<queue>
#include<string>
#include<cstdio>
#include<cstring>
#include<iostream>
#include<algorithm>
using namespace std;
typedef long long ll;
inline int read(){
	int x=0,p=1;char c=getchar();
	while(c<'0'||c>'9'){if(c=='-') p=-1;c=getchar();}
	while(c<='9'&&c>='0'){x=(x<<1)+(x<<3)+(c&15);c=getchar();}
	return x*p;
}
const int N=1e5+10;
int cnt,n,m;
const int mod=1e9+7;
int power(int a,int b){
	int ans=1;
	for(;b;b>>=1,a=1ll*a*a%mod)if(b&1)ans=1ll*a*ans%mod;
	return ans;
}
int f[20][N];
#define inv(x) power(x,mod-2)
inline int find(int k,int x)
{return f[k][x]==x?x:f[k][x]=find(k,f[k][x]);}
int log_2(int x){
	int ret=0;
	for(ret=0;x;x>>=1,ret++);
	return ret-1;
}
void solve(int k,int xx,int yy){
	int r1=find(k,xx),r2=find(k,yy);
	if(r1==r2) return;
	f[k][r1]=r2;
	if(!k){cnt--;return;}
	solve(k-1,xx,yy);
	solve(k-1,xx+(1<<k-1),yy+(1<<k-1));
}
int main(){
	freopen("moe.in","r",stdin);
	freopen("moe.out","w",stdout);
	cnt=n=read();m=read();
	for(int i=0;i<=18;i++)
	for(int j=1;j<=n;j++)
	f[i][j]=j;
	while(m--){
		int l1=read(),r1=read(),l2=read(),r2=read();
		if(l1>l2) swap(l1,l2),swap(r1,r2);
		int t=log_2(r1-l1+1);
    	solve(t,l1,l2);
    	solve(t,r1-(1<<t)+1,r2-(1<<t)+1);
	}
	printf("%d\n",9ll*power(10,cnt-1)%mod);
	fclose(stdin);
	fclose(stdout);
	return 0;
}

```