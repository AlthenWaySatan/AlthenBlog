---
title: '【CF487E】 Tourists 简易题解'
date: 2019-03-20 09:49:00
categories:
- 题解
tags:
- OI
- 图论
- 圆方树
mathjax: true
---
题目传送门：[【CF487E】 Tourists](https://www.luogu.com.cn/problem/CF487E)

<!-- more -->

## 题目大意

$\ \ \ \ \ \ \,$ 给你 $n$ 个点和 $m$条边的无向图，没有自环，没有重边，每个点上面有点权。

$\ \ \ \ \ \ \,$ 每次可能有两种操作：修改一个点的点权，或者询问两个点之间的路径上最小可能的点权是多少。

## 想法
$\ \ \ \ \ \ \,$有一个很显然的贪心想法，询问的时候肯定优先走较小权值路径，也就是在有分叉（点双）的时候走较小权值的那一侧，而只有可能最小的会造成贡献。

$\ \ \ \ \ \ \,$所以说我们可以尝试将每个点双之间建一个堆，记录最小的点权。

$\ \ \ \ \ \ \,$也就是建出一棵圆方树，方点上面一个堆，记录与他向连的圆点的权值最小值。

$\ \ \ \ \ \ \,$每次询问就只需要考虑经过唯一路径上面 **圆点点权** 和 **方点堆顶** 的最小值就行了，这个可以用树链剖分搞。

$\ \ \ \ \ \ \,$然后考虑修改，便是修改与这个 **圆点** 相连的方点上面的堆就好了，删除原来的点权，加入新点权。但是这样子我们不能保证其复杂度，要是圆方树建成一个菊花图就会 $T$ 飞了。

$\ \ \ \ \ \ \,$我们再考虑一下，我们树链剖分往上跳的时候，其实已经计算过  **走过的方点** 的父亲了（圆点），会有计算重复的地方。所以说 **方点的堆** 里面不需要记录其父亲的权值，换句话说，对于一个圆点，他的值只需要被他的 **方点父亲** 的堆记录，这样子每次修改，只需要修改他父亲的，加上线段树和堆的复杂度也不过一次 $O(\log n)$。

$\ \ \ \ \ \ \,$但是每次询问的是两个圆点，其$LCA$有可能是方点，这个时候少计算了一个 **方点$LCA$** 的父亲，需要注意加入判断一下。~~（第一个样第一次例询问输出为2有可能就是这样死的）~~

$\ \ \ \ \ \ \,$然后就是 **圆方树+树链剖分+线段树+可删堆** 套模板了，可删堆我是用 $fhq\_Treap$ 实现的，所以说代码比较丑。

$\ \ \ \ \ \ \,$预处理复杂度$O(n\log n)$，修改操作复杂度$O(\log n)$，询问操作复杂度$O(\log^2 n)$。

## 代码

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
#include<ctime>
#include<map>
#include<set>
using namespace std;
const int inf=0x3f3f3f3f;
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
const int N=4e5+200;
vector<int> G[N],E[N];
void add_E(int x,int y){E[x].push_back(y);E[y].push_back(x);}
void add_G(int x,int y){G[x].push_back(y);G[y].push_back(x);}
int sta[N],top,Size;
int tim,dfn[N],low[N];
void tarjan(int u){
  	dfn[u]=low[u]=++tim;
  	sta[++top]=u;
  	for(auto v:E[u]){
    	if(dfn[v]) low[u]=min(low[u],dfn[v]);
    	else{
      		tarjan(v),low[u]=min(low[u],low[v]);
      		if(low[v]>=dfn[u]){
        		++Size;int p;
        		while((p=sta[top])!=v)add_G(p,Size),top--;
        		p=sta[top],add_G(p,Size),top--;
        		add_G(u,Size);
      		}
    	}
  	}
}

struct node{int ls,rs,key,size,val;}T[N<<1];
#define lson T[rt].ls
#define rson T[rt].rs
int node_cnt;
queue<int>Rub;
int newnode(int x){
	int rt;
	if(!Rub.empty())rt=Rub.front(),Rub.pop();
	else rt=++node_cnt;
	lson=rson=0;
	T[rt].key=rand();T[rt].val=x;T[rt].size=1;
	return rt;
}
struct fhq_Treap{
	int root;
	void pushup(int rt){T[rt].size=T[lson].size+T[rson].size+1;}
	int merge(int a,int b){
		if(!a||!b)return a|b;
		if(T[a].key<T[b].key){T[a].rs=merge(T[a].rs,b);pushup(a);return a;}
		else{T[b].ls=merge(a,T[b].ls);pushup(b);return b;}
	}
	void split(int rt,int x,int &a,int &b){
		if(!rt){a=b=0;return;}
		if(T[rt].val<=x){a=rt;split(rson,x,rson,b);}
		else{b=rt;split(lson,x,a,lson);}
		pushup(rt);
	}
	void Insert(int x){
		int a,b;
		split(root,x,a,b);
		int rt=newnode(x);
		root=merge(merge(a,rt),b);
	}
	void Delete(int x){
		int a,b,c;
		split(root,x,a,c);
		split(a,x-1,a,b);
		b=merge(T[b].ls,T[b].rs);
		root=merge(merge(a,b),c);
	}
	int Min(){
		int rt=root;
		while(lson)rt=lson;
		return T[rt].val;
	}
}Treap[N];

int pos[N],w[N];
struct Segment_Tree{
  	#define Lson l,mid,rt<<1
  	#define Rson mid+1,r,rt<<1|1
  	int sum[N<<2];
  	void pushup(int rt){sum[rt]=min(sum[rt<<1],sum[rt<<1|1]);}
  	void build(int l,int r,int rt){
  		if(l==r){sum[rt]=w[pos[l]];return;}
  		int mid=(l+r)>>1;
  		build(Lson);build(Rson);
  		pushup(rt);
	}
  	void Update(int id,int c,int l,int r,int rt){
    	if(l==r){sum[rt]=c;return;}
    	int mid=(l+r)>>1;
    	if(id<=mid)Update(id,c,Lson);
    	else Update(id,c,Rson);
		pushup(rt);
  	}
  	int Query(int L,int R,int l,int r,int rt){
    	if(L<=l&&r<=R)return sum[rt];
    	int ret=1e9,mid=(l+r)>>1;
    	if(L<=mid)ret=min(ret,Query(L,R,Lson));
    	if(R>mid)ret=min(ret,Query(L,R,Rson));
    	return ret;
  	}
}Seg;

int n,m,Q;
struct Tree_Chain_Dissection{
	int idx[N];
	int deep[N],fa[N],son[N],size[N];
	int cnt,top[N];
	int dfs1(int u,int f,int dep){
		deep[u]=dep;fa[u]=f;size[u]=1;
		Treap[fa[u]].Insert(w[u]);
		int maxson=-1;
	  	for(auto v:G[u])if(v!=f){
		  	size[u]+=dfs1(v,u,dep+1);
		  	if(size[v]>maxson)maxson=size[v],son[u]=v;
		}
		return size[u];
	}
	void dfs2(int u,int topf){
		idx[u]=++cnt;top[u]=topf;
		pos[cnt]=u;
		if(!son[u])return;
		dfs2(son[u],topf);
	  	for(auto v:G[u])if(!idx[v])dfs2(v,v);
	}
	void init(){
		dfs1(1,0,1);
		dfs2(1,1);
	  	for(int i=n+1;i<=Size;++i)w[i]=Treap[i].Min();
		Seg.build(1,Size,1);
	}
	void Update(int u,int val){
	  	if(fa[u]){
	    	Treap[fa[u]].Delete(w[u]);
	    	Treap[fa[u]].Insert(val);
	    	Seg.Update(idx[fa[u]],Treap[fa[u]].Min(),1,Size,1);
	  	}
	  	w[u]=val;
		Seg.Update(idx[u],val,1,Size,1);
	}
	int Query(int x,int y){
	  	int ans=inf;
	  	while(top[x]!=top[y]){
	    	if(deep[top[x]]<deep[top[y]])swap(x,y);
	    	ans=min(ans,Seg.Query(idx[top[x]],idx[x],1,Size,1));
	    	x=fa[top[x]];
	  	}
	 	if(deep[x]>deep[y])swap(x,y);
	  	ans=min(ans,Seg.Query(idx[x],idx[y],1,Size,1));
    	if(x<=n)return ans;
    	else return min(ans,w[fa[x]]);
	}
}TCD;

char op[2];
int a,b;

int main()
{
	srand(time(NULL));
	Size=n=read();m=read();Q=read();w[0]=inf;
  	for(int i=1;i<=n;++i)w[i]=read();
  	for(int i=1,a,b;i<=m;++i)
	a=read(),b=read(),add_E(a,b);
  	tarjan(1);
  	TCD.init();
  	while(Q--){
    	scanf("%s",op);a=read(),b=read();
    	if(op[0]=='C')TCD.Update(a,b);
    	if(op[0]=='A')printf("%d\n",TCD.Query(a,b));
  	}
	return 0;
}


```
