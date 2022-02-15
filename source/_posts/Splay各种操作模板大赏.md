---
title: 'Splay各种操作模板大赏'
date: 2019-01-05 17:01:41
categories:
- 学习笔记
tags:
- OI
- 字符串
- 数据结构
- 平衡树
- Splay
mathjax: true
---

$\ \ \ \ \ \ \ \,$关于Splay操作的复习笔记：

$\ \ \ \ \ \ \ \,$依然的，只记录模板，不讲原理，关于Splay出门左拐百度。

<!-- more-->

## 结构
``` cpp
struct Splay{
	#define lson son[0][rt]
	#define rson son[1][rt]
	int root,cnt;
	int son[2][N],sum[N],val[N],size[N],fa[N];
	int lazy[N];
}
```
 1. $\tt root$ ：根；
 2. $\tt cnt$ ：下标大小；
 3. $\tt son[0/1][]$ ：左右儿子；
 4. $\tt sum[]$ ：子树大小；
 5. $\tt val[]$ ：节点权值；
 6. $\tt size[]$ ：节点大小；
 7. $\tt fa[]$ ：节点父亲；
 8. $\tt lazy[]$ ：下传标记。


## 基本操作
- 标记上传 ($pushup$)
``` cpp
void pushup(int rt){
	if(!rt)return;
	sum[rt]=sum[lson]+sum[rson]+size[rt];
}
```
- 标记下传 ($pushdown$)，根据情况会不一样
``` cpp
void pushdown(int rt){
  	if(!lazy[rt])return;
  	lazy[lson]^=1;lazy[rson]^=1;
  	lazy[rt]=0;
  	swap(lson,rson);
  }
```
- 左旋+右旋 ($rotate$)
``` cpp
 void rotate(int x){
  	int y=fa[x];int z=fa[y];
  	int k=son[1][y]==x;
  	son[son[1][z]==y][z]=x;
  	fa[x]=z;
  	son[k][y]=son[k^1][x];
  	fa[son[k^1][x]]=y;
  	son[k^1][x]=y;
  	fa[y]=x;
	pushup(y);pushup(x);
}
```
- 将$x$旋转到$goal$下 ($splay$)
``` cpp
void splay(int x,int goal){
  	while(fa[x]!=goal){
  		int y=fa[x];int z=fa[y];
  		if(z!=goal)
  		(son[1][z]==y)^(son[1][y]==x)?rotate(x):rotate(y);
  		rotate(x);
    }
    if(goal==0)root=x;
  }
```

## 修改

- 新建节点 ($newnode$)
``` cpp
int newnode(int f,int v){
    int rt=++cnt;
    size[rt]=1;sum[rt]=1;val[rt]=v;
    fa[rt]=f;lson=rson=0;lazy[rt]=0;
    return rt;
}
```
- 清空节点 ($clean$)
``` cpp
void clean(int rt){
    lson=rson=0;fa[rt]=0;
    val[rt]=0;sum[rt]=size[rt]=0;
}
```
- 建立 ($Build$)
``` cpp
void Build(int &rt,int L,int R,int f){
  	if(L>R)return;
  	int mid=(L+R)>>1;
  	rt=newnode(f,a[mid]);
  	Build(lson,L,mid-1,rt);
  	Build(rson,mid+1,R,rt);
  	pushup(rt);
}
```
- 插入($Insert$)
``` cpp
void Insert(int v){
    if(!root){root=newnode(0,v);return;}
    int now=root,rt=0;
    pushdown(now);
    while(1){
      if(val[now]==v){
  			++size[now];
        pushup(now);pushup(rt);
        splay(now,0);
        break;
      }
      rt=now;
      val[rt]<v?now=rson:now=lson;
    	pushdown(now);
      if(!now){
        now=newnode(rt,v);
  	    val[rt]<v?rson=now:lson=now;
        pushup(rt);splay(now,0);
        break;
      }
   }
}
```
- 删除节点($delete\_pos$)
``` cpp
void delete_pos(int rt){
    splay(rt,0);
    if(size[rt]>1){--size[rt];return;}
    int now;
    if(!lson||!rson){
      now=rson+lson;pushdown(now);
      if(!now){clean(rt);root=0;return;}
      clean(rt);root=now;fa[now]=0;
      pushup(now);return;
    }
    now=pre_pos(rt);
    pushdown(now);
    splay(now,rt);
    fa[now]=0;son[1][now]=rson;fa[rson]=now;
    root=now;clean(rt);
    pushup(rson);pushup(now);	
}
```
- 删除($Delete$)
``` cpp
void Delete(int v){
    v=QueryRank_pos(v);
    pushdown(v);
    delete_pos(v);
}
```
- 删除第k($Delete\_QueryRank$)
``` cpp
void Delete_QueryRank(int v){
    v=Rank_pos(v);
    pushdown(v);
    delete_pos(v);
}
```
- 区间修改（翻转）($revse$)
``` cpp
void revse(int a,int b){
	a=find(a);
  	b=find(b+2);
  	splay(a,0);splay(b,a);
  	lazy[son[0][son[1][root]]]^=1;
}
```


## 询问
- 查找某个值的节点编号($QueryRank\_pos$)
``` cpp
int QueryRank_pos(int x){
    int rt=root;
    while(1){
    	pushdown(rt);
    	if(val[rt]==x){splay(rt,0);return rt;} 
    	if(val[rt]<x)rt=rson;
    	else if(val[rt]>x)rt=lson;
    }	
}
```
- 查找某个排名的节点编号($Rank\_pos$)
``` cpp
  int Rank_pos(int x){
    int rt=root;
    while(1){
    	pushdown(rt);
    	if(sum[lson]>=x){rt=lson;continue;}
    	x-=sum[lson];
      	if(size[rt]>=x){splay(rt,0);return val[rt];}
      	x-=size[rt];
      	rt=rson;
    }	
  }
```
- 查找排名第k的值($QueryRank$)
``` cpp
int QueryRank(int x){
    int rt=root;
    while(1){
    	pushdown(rt);
    	if(sum[lson]>=x){rt=lson;continue;}
    	x-=sum[lson];
    	if(size[rt]>=x){splay(rt,0);return val[rt];}
    	x-=size[rt];
    	rt=rson;
  }
}
```

- 查找某个值的排名($Rank$)
``` cpp
int Rank(int num){
    int rt=root,ans=0;
    while(1){
    	pushdown(rt);
    	if(val[rt]==num){ans=ans+sum[lson]+1;splay(rt,0);return ans;}
    	if(val[rt]<num)ans+=sum[lson]+size[rt],rt=rson;
     	else if(val[rt]>num)rt=lson;
  	}
}
```
- 集合内前驱的节点编号($pre\_pos$)
``` cpp
int pre_pos(int x){
    splay(x,0);int rt;
    pushdown(x);
    if(!(rt=son[0][x]))return val[x];
    while(rson)pushdown(rt),rt=rson;
    return rt;	
}
```
- 集合内后继的节点编号($nex\_pos$)
``` cpp
int nex_pos(int x){
    splay(x,0);int rt;
    pushdown(x);
    if(!(rt=son[1][x]))return val[x];
    while(lson)pushdown(rt),rt=lson;
    return rt;	
}
```
- 集合内前驱的值($pre$)
``` cpp
int pre(int x){
	splay(x,0);int rt;
    pushdown(x);
    if(!(rt=son[0][x]))return val[x];
    while(rson)pushdown(rt),rt=rson;
    return val[rt];
}
```
- 集合内后继的值($nex$)
``` cpp
int nex(int x){
    splay(x,0);int rt;
    pushdown(x);
    if(!(rt=son[1][x]))return val[x];
    while(lson)pushdown(rt),rt=lson;
    return val[rt];
}
```
- 前驱($Numpre$)
``` cpp
int Numpre(int x){
    Insert(x);
    int y=x;x=QueryRank_pos(x);
    int ls=pre(x);
    Delete(y);
    return ls;
}
```
- 后继($Numnex$)
``` cpp
int Numnex(int x){
    Insert(x);
    int y=x;x=QueryRank_pos(x);
    int ls=nex(x);
    Delete(y);
    return ls;
}
```
- 查找某个值的节点编号($find$)，不改变结构
``` cpp
int find(int k){
  	int rt=root;
  	while(1){
  		pushdown(rt);
  		if(sum[lson]>=k)rt=lson;
  		else if(sum[lson]+1==k)return rt;
  		else k-=sum[lson]+1,rt=rson;
    }
 }
```


## 完整模板
``` cpp
const int N=1e5+10;
int a[N],n,m;
struct Splay{
	#define lson son[0][rt]
	#define rson son[1][rt]
	int root,cnt;
	int son[2][N],sum[N],val[N],size[N],fa[N];
  	bool lazy[N];
	void pushdown(int rt){
  		if(!lazy[rt])return;
  		lazy[lson]^=1;lazy[rson]^=1;
  		lazy[rt]=0;
  		swap(lson,rson);
  	}
	void pushup(int rt){
	  	if(!rt)return;
	  	sum[rt]=sum[lson]+sum[rson]+size[rt];
	}
  	void rotate(int x){
	  	int y=fa[x];int z=fa[y];
	  	int k=son[1][y]==x;
	  	son[son[1][z]==y][z]=x;
	  	fa[x]=z;
	  	son[k][y]=son[k^1][x];
	  	fa[son[k^1][x]]=y;
	  	son[k^1][x]=y;
	  	fa[y]=x;
	  	pushup(y);pushup(x);
	}
  	void splay(int x,int goal){
	  	while(fa[x]!=goal){
	  		int y=fa[x];int z=fa[y];
	  		if(z!=goal)
	  		(son[1][z]==y)^(son[1][y]==x)?rotate(x):rotate(y);
	  		rotate(x);
	    }
	    if(goal==0)root=x;
  	}
  	int newnode(int f,int v){
	    int rt=++cnt;
	    size[rt]=1;sum[rt]=1;val[rt]=v;
	    fa[rt]=f;lson=rson=0;lazy[rt]=0;
	    return rt;
		}
		void clean(int rt){
	    lson=rson=0;fa[rt]=0;
	    val[rt]=0;sum[rt]=size[rt]=0;
  	}
	void Build(int &rt,int L,int R,int f){
	  	if(L>R)return;
	  	int mid=(L+R)>>1;
	  	rt=newnode(f,a[mid]);
	  	Build(lson,L,mid-1,rt);
	  	Build(rson,mid+1,R,rt);
	  	pushup(rt);
	}
  	void Insert(int v){
    	if(!root){root=newnode(0,v);return;}
    	int now=root,rt=0;
    	pushdown(now);
   		while(1){
      		if(val[now]==v){
	  			++size[now];
	        	pushup(now);pushup(rt);
	        	splay(now,0);
	        	break;
	      	}
	      	rt=now;
	      	val[rt]<v?now=rson:now=lson;
	    	pushdown(now);
	      	if(!now){
	        	now=newnode(rt,v);
	  	    	val[rt]<v?rson=now:lson=now;
	        	pushup(rt);splay(now,0);
	        	break;
	      	}
   	 	}
  	}
  	int QueryRank_pos(int x){
    	int rt=root;
    	while(1){
    		pushdown(rt);
      		if(val[rt]==x){splay(rt,0);return rt;} 
      		if(val[rt]<x)rt=rson;
      		else if(val[rt]>x)rt=lson;
    	}	
  }
  int Rank_pos(int x){
    int rt=root;
    while(1){
    	pushdown(rt);
      if(sum[lson]>=x){rt=lson;continue;}
      x-=sum[lson];
      if(size[rt]>=x){splay(rt,0);return val[rt];}
      x-=size[rt];
      rt=rson;
    }	
  }
  	int QueryRank(int x){
    	int rt=root;
    	while(1){
    		pushdown(rt);
     		if(sum[lson]>=x){rt=lson;continue;}
     		x-=sum[lson];
      		if(size[rt]>=x){splay(rt,0);return val[rt];}
      		x-=size[rt];
      		rt=rson;
    	}
  	}
  	int Rank(int num){
    	int rt=root,ans=0;
    	while(1){
    		pushdown(rt);
      		if(val[rt]==num){ans=ans+sum[lson]+1;splay(rt,0);return ans;}
     		if(val[rt]<num)ans+=sum[lson]+size[rt],rt=rson;
      		else if(val[rt]>num)rt=lson;
    	}
  	}
  	int pre_pos(int x){
    	splay(x,0);int rt;
    	pushdown(x);
    	if(!(rt=son[0][x]))return val[x];
    	while(rson)pushdown(rt),rt=rson;
    	return rt;	
  	}
  	int nex_pos(int x){
    	splay(x,0);int rt;
    	pushdown(x);
    	if(!(rt=son[1][x]))return val[x];
    	while(lson)pushdown(rt),rt=lson;
    	return rt;	
  	}
  	int pre(int x){
    	splay(x,0);int rt;
    	pushdown(x);
    	if(!(rt=son[0][x]))return val[x];
    	while(rson)pushdown(rt),rt=rson;
    	return val[rt];
  	}
  	int nex(int x){
    	splay(x,0);int rt;
    	pushdown(x);
    	if(!(rt=son[1][x]))return val[x];
    	while(lson)pushdown(rt),rt=lson;
    	return val[rt];
  	}
  	void delete_pos(int rt){
    	splay(rt,0);
    	if(size[rt]>1){--size[rt];return;}
    	int now;
    	if(!lson||!rson){
      		now=rson+lson;pushdown(now);
      		if(!now){clean(rt);root=0;return;}
      		clean(rt);root=now;fa[now]=0;
      		pushup(now);return;
    	}
    	now=pre_pos(rt);
    	pushdown(now);
    	splay(now,rt);
    	fa[now]=0;son[1][now]=rson;fa[rson]=now;
    	root=now;clean(rt);
    	pushup(rson);pushup(now);	
  	}
  	void Delete(int v){
    	v=QueryRank_pos(v);
    	pushdown(v);
    	delete_pos(v);
  	}
  	void Delete_QueryRank(int v){
    	v=Rank_pos(v);
    	pushdown(v);
    	delete_pos(v);
  	}
  	int Numpre(int x){
    	Insert(x);
    	int y=x;x=QueryRank_pos(x);
    	int ls=pre(x);
    	Delete(y);
    	return ls;
  	}
  	int Numnex(int x){
    	Insert(x);
    	int y=x;x=QueryRank_pos(x);
    	int ls=nex(x);
    	Delete(y);
    	return ls;
  	}
  	int find(int k){
  		int rt=root;
  		while(1){
  			pushdown(rt);
  			if(sum[lson]>=k)rt=lson;
  			else if(sum[lson]+1==k)return rt;
  			else k-=sum[lson]+1,rt=rson;
    	}
  	}
	void revse(int a,int b){
		a=find(a);
  		b=find(b+2);
  		splay(a,0);splay(b,a);
  		lazy[son[0][son[1][root]]]^=1;
	}
}Spl;
```

## 两个模板题：
- [P3369 【模板】普通平衡树](https://www.luogu.org/problemnew/show/P3369)
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
	if(f)return x;else return-x;
}
const int N=1e5+10;
int a[N],n,m;
struct Splay{
	
}Spl;
int main()
{
  	int n=read();
  	while(n--){
    	int opt=read(),x=read();
    	if(opt==1){Spl.Insert(x);continue;}
    	if(opt==2){Spl.Delete(x);continue;}
    	if(opt==3){printf("%d\n",Spl.Rank(x));continue;}
    	if(opt==4){printf("%d\n",Spl.QueryRank(x));continue;}
    	if(opt==5){printf("%d\n",Spl.Numpre(x));continue;}
    	if(opt==6){printf("%d\n",Spl.Numnex(x));continue;}
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
	if(f)return x;else return-x;
}
const int N=1e5+10;
int a[N],n,m;
struct Splay{
	
}Spl;
void write(int rt){
  	Spl.pushdown(rt);
  	if(Spl.lson)write(Spl.lson);
  	if(Spl.val[rt]>1&&Spl.val[rt]<n+2)
  	printf("%d ",Spl.val[rt]-1);
  	if(Spl.rson)write(Spl.rson); 
}
int main()
{
	n=read();m=read();
 	for(int i=1;i<=n+2;i++)a[i]=i;
 	Spl.Build(Spl.root,1,n+2,0);
 	while(m--){
    	int l=read(),r=read();
  		Spl.revse(l,r);
  	}
  	write(Spl.root);
  	printf("\n");
	return 0;
}

```