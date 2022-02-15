---
title: 'OI中常见的线性代数矩阵问题'
date: 2018-12-29 14:18:02
categories:
- 学习笔记
tags:
- OI
- 数学
- 矩阵乘法
- 矩阵求逆
- 高斯消元
mathjax: true
---

$\ \ \ \ \ \ \ \,$关于OI中常见的线性代数矩阵问题的复习笔记：


<!-- more -->

$\ \ \ \ \ \ \,$对于一个 $n\times m$ 的矩阵 $A$，我们这样定义：

$ A=\begin{bmatrix} a_{(1,1)} &  a_{(1,2)} &\cdots &a_{(1,n)} \\  a_{(2,1)}&  a_{(2,2)} &\cdots &a_{(2,n)} \\\vdots &\vdots &\ddots &\vdots\\  a_{(m,1)}&  a_{(m,2)} &\cdots &a_{(m,n)}  \end{bmatrix} $


``` cpp
struct matrix{	
	int n,m;
	double a[N][N];
}A;
```

## 高斯消元

$\ \ \ \ \ \ \,$回忆初中学的多元一次方程（线性方程），我们可以把它当做一个矩阵，然后等号的后面为 $n\times 1$ 的增广矩阵。

$\ \ \ \ \ \ \,$我们解答的过程，就是用方程之间的加减，或者乘一个常数，来消去一些未知数，直到可以直接解出答案。这个消元过程，就是高斯消元。

$\ \ \ \ \ \ \,$变换到矩阵上面，就是通过一些行与行之间的加减，或者乘一个常数，来使得一些位置的值变为 $0$ 。

$\ \ \ \ \ \ \,$我们一般会画成下面两种形态：

1. 行阶梯式

  $ \begin{bmatrix} a &  a &a \\  0&  a &a \\0&  0 &a  \end{bmatrix} $
  
  $\ \ \ \ \ \ \,$代码如下，复杂度$O(n^3)$
  
``` cpp
void Gauss(matrix &A){
	for(int i=1;i<=A.n;i++){
		for(int j=i;j<=A.m;j++)
		if(fabs(A.a[j][i])){swap(A.a[j],A.a[i]);break;}
		for(int j=i+1;j<=A.m;j++){
		if(fabs(A.a[i][i])<eps)continue;
		double f=A.a[j][i]/A.a[i][i];
		for(int k=i;k<=A.n;k++)A.a[j][k]-=f*A.a[i][k];
		}
	}
}
```
  
2. 行最简式

  $ \begin{bmatrix} a &  0 &0 \\  0&  a &0 \\0&  0 &a\end{bmatrix} $
  
  $\ \ \ \ \ \ \,$代码如下，在化成行阶梯式之后，再化成行最简式，复杂度$O(n^3)$
  
  
``` cpp
void Gauss(matrix &A){
	for(int i=1;i<=A.n;i++){
 		for(int j=i;j<=A.m;j++)
 		if(fabs(A.a[j][i])){swap(A.a[j],A.a[i]);break;}
 		for(int j=i+1;j<=A.m;j++){
 			if(fabs(A.a[i][i])<eps)continue;
 			double f=A.a[j][i]/A.a[i][i];
      for(int k=i;k<=A.n;k++)A.a[j][k]-=f*A.a[i][k];
		}
	}
	for(int i=A.n;i>=1;i--)
 	for(int j=i-1;j>=1;j--){
 		if(fabs(A.a[j][i])<eps)continue;
 		double f=-A.a[i][i]/A.a[j][i];
    for(int k=j;k<=A.n;k++)
    (A.a[j][k]*=f)+=A.a[i][k];
	}
}
```

## 矩阵的秩和行列式的值

1. 矩阵的秩

$\ \ \ \ \ \ \,$矩阵的秩是A的线性独立的纵（横）列的极大数目，感性地理解，就是在线性方程里的非自由元数目，也就是有用的为多少，我们可以在高斯消元的同时，来求矩阵的秩。

$\ \ \ \ \ \ \ \,$也就是统计行阶梯式的对角线上，不为 $0$ 的数目。

2. 行列式的值

$\ \ \ \ \ \ \,$行列式虽然与矩阵不同，但是也可以用矩阵表示，暴力求行列式的值复杂度太高，但是我们可以高斯消元过后再求。

$\ \ \ \ \ \ \,$就是行阶梯式的对角线的乘积。



## 矩阵乘法

$\ \ \ \ \ \ \,$那么对于矩阵之间的乘法 $B\times A$，我们这样定义：

$\ \ \ \ \ \ \,$首先，我们需要确定，$B$ 的列数等于$A$ 的行数，既$m_A=n_B$。

$\ \ \ \ \ \ \,$令$S_{(i,j)}=\sum_{k=1}^{m_A}a_{(k,j)}\times b_{(i,k)}$，那么有：

$B\times A=\begin{bmatrix} S_{(1,1)} &  S_{(1,2)} &\cdots &S_{(1,n_A)} \\  S_{(2,1)}&  S_{(2,2)} &\cdots &S_{(2,n_A)} \\\vdots &\vdots &\ddots &\vdots\\  S_{(m_B,1)}&  S_{(m_B,2)} &\cdots &S_{(m_B,n_A)}  \end{bmatrix} $

$\ \ \ \ \ \ \,$显然是一定满足结合律，不一定满足交换律的，复杂度为$O(n^3)$，直接模拟的，代码如下：

``` cpp
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
```


## 矩阵逆元

$\ \ \ \ \ \ \,$有矩阵乘法，那么就会存在单位元，那么就自然会有逆元的存在。

$\ \ \ \ \ \ \,$单位矩阵，既对角线上面都是 $1$，其余都是 $0$ 的矩阵：

$ I=\begin{bmatrix} 1 &  0 &\cdots &0 \\  0&  1 &\cdots &0 \\\vdots &\vdots &\ddots &\vdots\\  0&  0 &\cdots &1  \end{bmatrix} $

$\ \ \ \ \ \ \,$根据矩阵乘法的定义，很容易知道$A\times I=A$。

$\ \ \ \ \ \ \,$那么对于矩阵$A$，若是$A\times A'=I$，我们就称 $A'$ 为 $A$ 的逆矩阵。

$\ \ \ \ \ \ \,$对于一个矩阵$A$，肯定可以通过一些行与行之间的加减，或者乘一个常数，既高斯消元，变成 $I$。那么我们令$B=I$，同时和$A$做高斯消元，那么当$A=I$时，$B=A'$了。

$\ \ \ \ \ \ \,$模板如下，返回$0$是无逆元情况：

``` cpp
bool Inv(matrix A,matrix &B){
	B.n=A.n;B.m=A.m;
	memset(B.a,0,sizeof(B.a));
	for(int i=1;i<=A.n;i++)B.a[i][i]=1;
	for(int i=1;i<=A.n;i++){
    for(int j=i;j<=A.n;j++)
    if(A.a[j][i]){
      swap(A.a[i],A.a[j]);
			swap(B.a[i],B.a[j]);
      break;
    }
    if(!A.a[i][i])return 0;
    long long r=inv(A.a[i][i]);
    for(int j=i;j<=A.n;j++)A.a[i][j]=A.a[i][j]*r%mod;
    for(int j=1;j<=A.n;j++)B.a[i][j]=B.a[i][j]*r%mod;
    for(int j=1;j<=A.n;j++)
    if(j!=i){
      long long f=A.a[j][i];
      for(int k=i;k<=A.n;k++)
      A.a[j][k]=(A.a[j][k]-f*A.a[i][k]%mod+mod)%mod;
      for(int k=1;k<=A.n;k++)
      B.a[j][k]=(B.a[j][k]-f*B.a[i][k]%mod+mod)%mod;
    }
  }
  return 1;
}
```

