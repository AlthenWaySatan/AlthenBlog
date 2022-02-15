---
title: 'tarjian算法的运用'
date: 2019-01-02 19:23:06
categories:
- 学习笔记
tags:
- OI
- 图论
- Tarjian
- 连通分量
- 圆方树
- 割点
mathjax: true
---

$\ \ \ \ \ \ \ \,$tarjian算法及其相关算法的复习笔记：

<!-- more -->

## 边双连通分量

$\ \ \ \ \ \ \ \,$对于一个有向图，能互相到达的点在一个连通分量，很多时候一个连通分量的点对答案没有影响，或者可以统一安排，那么我们就用tarjan算法把它们缩在一起。

$\ \ \ \ \ \ \ \,$这样就把有向图转换为了一个DAG，会方便处理很多。

``` cpp
int U[N],V[N],p;
vector<int> G[N];
void add(int a,int b){U[++p]=a;V[p]=b;G[a].push_back(b);}
int sta[N],top;
int low[N],dfn[N],tim;
int col[N],mark;
void tarjan(int x){
  sta[++top]=x;
  low[x]=dfn[x]=++tim;
  for(auto v:G[x]){
  	if(!dfn[v])tarjan(v);
    if(!col[v])low[x]=min(low[x],low[v]);
	}
  if(low[x]==dfn[x]){
    mark++; 
    while(sta[top+1]!=x)col[sta[top]]=mark,top--;
  }
}
//main():
//  for(int i=1;i<=n;i++)
//  if(!dfn[i]) tarjan(i);
```


## 割点

$\ \ \ \ \ \ \ \,$在一个无向图中，去掉一个点，使得图不连通，这个点就叫割点（sta[]中）：

``` cpp
vector<int> G[N];
void add(int a,int b){G[a].push_back(b);G[b].push_back(a);}
int sta[N],top;
int low[N],dfn[N],tim;
bool used[N];
void tarjan(int x,int rt){
  int cnt=0;
  low[x]=dfn[x]=++tim;
  for(auto v:G[x]){
  	if(!dfn[v]){
      tarjan(v,rt);
      low[x]=min(low[x],low[v]);
      if(low[v]>=dfn[x]&&x!=rt&&used[x]==0)
      {sta[++top]=x;used[x]=1;}
      if(x==rt)cnt++;
    }
    low[x]=min(low[x],dfn[v]);
  }
  if(x==rt&&cnt>=2&&used[x]==0)
  {sta[++top]=x;used[x]=1;}
}
//main():
//  for(int i=1;i<=n;i++)
//  if(!dfn[i]) tarjan(i,i);
```


## 圆方树

$\ \ \ \ \ \ \ \,$专门处理仙人掌的做法，把一个环转换为一个方点，把一个仙人掌转换为一棵树，方便处理：

$\ \ \ \ \ \ \ \,$（方点之间连接的圆点为割点）

``` cpp
int sta[N],top,size,tim,dfn[N],low[N];
vector<int> G[N],E[N<<1];
void add1(int a,int b){G[a].push_back(b);G[b].push_back(a);}
void add2(int a,int b){E[a].push_back(b);E[b].push_back(a);}
void tarjan(int x){
  dfn[x]=low[x]=++tim;
  sta[++top]=x;
  for(auto v:G[x]){
    if(!dfn[v]){
      tarjan(v),low[x]=min(low[x],low[v]);
      if(low[v]>=dfn[x]){
        ++size;
        while(sta[top+1]!=v)add2(sta[top],size),top--;
        add2(x,size);
      }
    }
		low[x]=min(low[x],dfn[v]);
  }
}
```