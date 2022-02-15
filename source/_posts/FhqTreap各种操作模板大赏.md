---
title: 'FhqTreap各种操作模板大赏'
date: 2019-01-06 18:19:42
categories:
- 学习笔记
tags:
- OI
- 字符串
- 数据结构
- 平衡树
- Fhq_Treap
mathjax: true
---

$\ \ \ \ \ \ \ \,$关于Splay操作的复习笔记：

$\ \ \ \ \ \ \ \,$一些平衡树操作和都是一样的，详见[【Splay各种操作模板大赏】](/2019/01/05/Splay各种操作模板大赏/)。

<!-- more-->

## 结构
``` cpp
struct fhq_Treap{
	#define lson ls[rt]
	#define rson rs[rt]
  	int size[N],key[N],sum[N],val[N];
  	int ls[N],rs[N],cnt;
  	int root;
  	bool lazy[N];
}
```
 1. $\tt root$ ：根；
 2. $\tt cnt$ ：下标大小；
 3. $\tt key[]$ ：关键值；
 4. $\tt ls[]\&rs[]$ ：左右儿子；
 5. $\tt sum[]$ ：子树权值；
 6. $\tt val[]$ ：节点权值；
 7. $\tt size[]$ ：节点大小；
 8. $\tt lazy[]$ ：下传标记。

## 基本操作
- 手写随机($Rand$)
``` cpp
int Rand(){
    static int seed=233;
    return seed=int(seed*48271LL%20020207);
}
```
- 标记上传 ($pushup$)
``` cpp
void pushup(int rt){
	size[rt]=size[lson]+size[rson]+1;
	sum[rt]=sum[lson]+sum[rson]+val[rt];
}
```
- 标记下传 ($pushdown$)，根据情况会不一样
``` cpp
void pushdown(int rt){
  	if(!lazy[rt])return;
    swap(lson,rson);
    lazy[lson]^=1;lazy[rson]^=1;
    lazy[rt]=0;
}
```
- 分裂($split$)
``` cpp
//权值
void split(int rt,int x,int &a,int &b){
    if(!rt) a=0,b=0;
    else{
    	pushdown(rt);
      	if(val[rt]<=x)a=rt,split(rson,x,rs[a],b);
      	else b=rt,split(lson,x,a,ls[b]);
      	pushup(rt);
    }
}
//位置
void split(int rt,int x,int &a,int &b){
    if(!rt) a=0,b=0;
    else{
    	pushdown(rt);
    	if(size[ls[rt]]<x)a=rt,split(rson,x-size[ls[rt]]-1,rs[a],b);
      	else b=rt,split(lson,x,a,ls[b]);
      	pushup(rt);
    }
}
```
- 合并($merge$)
``` cpp
int merge(int x,int y){
    if(!x||!y) return x+y;
    if(key[x]<key[y]){pushdown(x);rs[x]=merge(rs[x],y);pushup(x);return x;}
    else {pushdown(y);ls[y]=merge(x,ls[y]);pushup(y);return y;}
}
```

## 修改
- 新建节点 ($newnode$)
``` cpp
int newnode(int v){
  	sum[++cnt]=val[cnt]=v;size[cnt]=1;key[cnt]=Rand();
  	return cnt;
}
```
- 插入($Insert$)
``` cpp
void Insert(int x,int v){
    int a,b;
    split(root,x,a,b);
    int rt=newnode(v);
    root=merge(merge(a,rt),b);
}
```
- 删除($Delete$)
``` cpp
void Delete(int x){
    int a,b,c;
    split(root,x,a,b);split(a,x-1,a,c);
    c=merge(ls[c],rs[c]);
    root=merge(merge(a,c),b);
}
```
- 区间修改（翻转）($revse$)
``` cpp
void Reverse(int l,int r){
    int x,y,z;
    split(root,r,x,z);
    split(x,l-1,x,y);
    lazy[y]^=1;
    root=merge(merge(x,y),z);
}
```

## 询问
- 查找排名第k的值($QueryRank$)
``` cpp
int QueryRank(int x){
    int rt=root;
    while(rt){
      	if(x==size[lson]+1)return val[rt];
      	if(size[lson]>=x)rt=lson;
      	else x-=size[lson]+1,rt=rson;
    }
}
```

- 查找某个值的排名($Rank$)
``` cpp
int Rank(int x){
    int rt=root,res=1;
    while(rt){
    	if(val[rt]>=x)rt=lson;
    	else res+=size[lson]+1,rt=rson;
  	}
  	return res;
}
```
- 前驱($Numpre$)
``` cpp
int Numpre(int x){return QueryRank(Rank(x)-1);}
```
- 后继($Numnex$)
``` cpp
int Numnex(int x){return QueryRank(Rank(x+1));}
```
- 区间查询（权值和）($Numnex$)
``` cpp
long long Query(int l,int r){
    int x,y,z;
    split(root,r,x,z);
    split(x,l-1,x,y);
    long long ans=sum[y];
    root=merge(merge(x,y),z);
    return ans;
}
```

## 两个模板题：
- [P3369 【模板】普通平衡树](https://www.luogu.org/problemnew/show/P3369)
``` cpp
// luogu-judger-enable-o2
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
struct fhq_Treap{
    #define lson ls[rt]
    #define rson rs[rt]
  int Rand(){
    static int seed=233;
    return seed=int(seed*48271LL%20020207);
  }
  int size[N],key[N],val[N];
  int ls[N],rs[N],cnt,sum[N];
  int root;
  bool lazy[N];
  void pushup(int rt){
        size[rt]=size[lson]+size[rson]+1;
        sum[rt]=sum[lson]+sum[rson]+val[rt];
    }
  int clone(int b){
  	++cnt;
    size[cnt]=size[b];key[cnt]=key[b];val[cnt]=val[b];
    ls[cnt]=ls[b];rs[cnt]=rs[b],lazy[cnt]=lazy[b];sum[cnt]=sum[b];
    return cnt;
  }
  void pushdown(int rt) {
    if(!lazy[rt])return;
    swap(lson,rson);
    lazy[lson]^=1;lazy[rson]^=1;
    lazy[rt]=0;
  }
  int merge(int x,int y){
    if(!x||!y) return x+y;
    if(key[x]<key[y]){pushdown(x);rs[x]=merge(rs[x],y);pushup(x);return x;}
    else {pushdown(y);ls[y]=merge(x,ls[y]);pushup(y);return y;}
  }
  //权值
  void split(int rt,int x,int &a,int &b){
    if(!rt) a=0,b=0;
    else{
    	pushdown(rt);
      if(val[rt]<=x)a=rt,split(rson,x,rs[a],b);
      else b=rt,split(lson,x,a,ls[b]);
      pushup(rt);
    }
  }
  void Insert(int x){
    int a,b;
    split(root,x,a,b);
    val[++cnt]=x;size[cnt]=1;key[cnt]=Rand();
    root=merge(merge(a,cnt),b);
  }
  void Delete(int x){
    int a,b,c;
    split(root,x,a,b);split(a,x-1,a,c);
    c=merge(ls[c],rs[c]);
    root=merge(merge(a,c),b);
  }
  int Rank(int x){
    int rt=root,res=1;
    while(rt){
    	if(val[rt]>=x)rt=lson;
    	else res+=size[lson]+1,rt=rson;
  	}
  	return res;
  }
  int QueryRank(int x){
    int rt=root;
    while(rt){
      if(x==size[lson]+1)return val[rt];
      if(size[lson]>=x)rt=lson;
      else x-=size[lson]+1,rt=rson;
    }
  }
  int Numpre(int x){return QueryRank(Rank(x)-1);}
  int Numnex(int x){return QueryRank(Rank(x+1));}
  void Reverse(int l,int r){
    int x,y,z;
    split(root,r,x,z);
    split(x,l-1,x,y);
    lazy[y]^=1;
    root=merge(merge(x,y),z);
  }
  long long Query(int l,int r){
    int x,y,z;
    split(root,r,x,z);
    split(x,l-1,x,y);
    long long ans=sum[y];
    root=merge(merge(x,y),z);
    return ans;
  }
}Fhq;

int main()
{
  int n=read();
  while(n--){
    int opt=read(),x=read();
    if(opt==1){Fhq.Insert(x);continue;}
    if(opt==2){Fhq.Delete(x);continue;}
    if(opt==3){printf("%d\n",Fhq.Rank(x));continue;}
    if(opt==4){printf("%d\n",Fhq.QueryRank(x));continue;}
    if(opt==5){printf("%d\n",Fhq.Numpre(x));continue;}
    if(opt==6){printf("%d\n",Fhq.Numnex(x));continue;}
  }
  return 0;
}
```
- [P3391 【模板】文艺平衡树（Splay）](https://www.luogu.org/problemnew/show/P3391)
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
int a[N],n,m;
struct fhq_Treap{
	#define lson ls[rt]
	#define rson rs[rt]
  int Rand(){
    static int seed=233;
    return seed=int(seed*48271LL%20020207);
  }
  int size[N],key[N],val[N],w[N];
  int ls[N],rs[N],cnt;
  int root;
  bool lazy[N];
  void pushup(int rt){
		size[rt]=size[lson]+size[rson]+1;
	}
  void pushdown(int rt) {
    if(!lazy[rt])return;
    swap(lson,rson);
    lazy[lson]^=1;lazy[rson]^=1;
    lazy[rt]=0;
  }
  int merge(int x,int y){
    if(!x||!y) return x+y;
    if(key[x]<key[y]){pushdown(x);rs[x]=merge(rs[x],y);pushup(x);return x;}
    else {pushdown(y);ls[y]=merge(x,ls[y]);pushup(y);return y;}
  }
  //位置
  void split(int rt,int x,int &a,int &b){
    if(!rt) a=0,b=0;
    else{
    	pushdown(rt);
    	if(size[ls[rt]]<x)a=rt,split(rson,x-size[ls[rt]]-1,rs[a],b);
      else b=rt,split(lson,x,a,ls[b]);
      pushup(rt);
    }
  }
	void Insert(int x,int v){
    int a,b;
    split(root,x,a,b);++cnt;
    val[cnt]=v;size[cnt]=1;key[cnt]=Rand();
    root=merge(merge(a,cnt),b);
  }
  void Reverse(int l,int r){
    int x,y,z;
    split(root,r,x,z);
    split(x,l-1,x,y);
    lazy[y]^=1;
    root=merge(merge(x,y),z);
  }
}Fhq;
void write(int rt){
  Fhq.pushdown(rt);
  if(Fhq.lson)write(Fhq.lson);
  printf("%d ",Fhq.val[rt]);
  if(Fhq.rson)write(Fhq.rson); 
}
int main()
{
  n=read();m=read();
 	for(int i=1;i<=n;i++)Fhq.Insert(i-1,i);
 	while(m--){
    int l=read(),r=read();
  	Fhq.Reverse(l,r);
  }
  write(Fhq.root);
  printf("\n");
	return 0;
}

```
