## 1 问题

最长回文子串

## 2 经典写法

中心扩散法：

1. 逐个遍历字符
2. 每一个字符向两侧扩散
3. 记录最长子串

> 问题：会错过偶数长度的回文子串

解决方法：

1. 在字符间插入特殊符号，比如`#`
2. 再单独逐两个遍历字符，寻找一遍偶数长度的回文子串

```c++
int countSubstrings(string s) {
	int countL = 0, n = s.size();
	int i,l,r;
	for(i = 0; i < n;++i){// 奇数
		++countL; // 字符本身算是一个回文子串
		l = i-1;
		r = i+1;
		while(l>=0&&r<n){
			if(s[l--] != s[r++]){
				break;
			}else{
			++countL;
			}
		}
	}
	for(i = 0; i < n-1;++i){//偶数
		if(s[i] == s[i+1]){
			++countL;
		}else{
			continue;
		}
		l = i-1;
		r = i+2;
		while(l>=0&&r<n){
			if(s[l--] != s[r++]){
				break;
			}
			else{
			++countL;
			}
		}
	}
	return countL;
}
```

## 3 manacher算法

### 3. 1 四个关键概念

#### 回文半径、回文直径

回文子串的长度为回文直径，回文半径就是其$直径/2$

#### 回文半径数组

在遍历的过程中将扩充的回文半径数组记录下来

#### 变量 R

设置变量`R`，负责记录之前扩的所有位置中，所到达的最右回文子串右边界，只记录最大值，只增不减

#### 变量 C

取得最远边界时，变量C记录中心点位置，C跟着R一起更新

## 3.2 代码思路

回文座标系如图：

![[回文坐标|1000]]

### 情况1 当前中心点`i`未在最右边界内

也就是，$i >= R$，暴力遍历，没有优化

### 情况2 当前中心点在最右边界内

也就是$i \in (C, R)$，则此时，必存在$i' \in (L,C)$

根据以 $i'$ 为中心点的最长回文情况，又存在三种情况

#### （1）以 $i'$ 为中心点的最长回文在最长回文内部

也就是以 $i'$ 为中心点的最长回文在$(L,L')$中

此时，$i$的回文情况与$i'$一致

#### （2）以 $i'$ 为中心点的最长回文不全在最长回文内部

也就是以 $i'$ 为中心点的最长回文不全在$(L,L')$范围内

此时，$i$的最长回文子串一定为$[R',R]$

#### （3）以 $i'$ 为中心点的最长回文与最长回文相切

也就是以 $i'$ 为中心点的最长回文为$(L,L')$中

此时，$i$的最长回文子串一定大于$[R',R]$

> [!BUG] 扩充字符
> 补充：第一步需要扩充字符，要不然回文子串是双数个数的是无法被捕捉到的

### 3.3 代码

```C++
string longestPalindrome(string s) {

	int n = s.size();
	if(n == 0|n==1) return s;
	
	string str = "#"; // 扩充源字符为 #a#b#c#
	
	for(int i = 0; i < n; ++i){
		str+=s[i];
		str+='#';
	}
	n = str.size();// 获取新字符串的长度
	string sRes = "";// 结果
	vector<int> pArr; // 回文半径数组
	for(int i = 0; i < n; ++i){
		pArr.push_back(0);
	}
	int R = -1, C = -1; // 记录最右边界以及最右边界时的中心点位置
	for(int i = 0; i < n; ++i){
		if(i > R){// 情况1 
			int li = i-1, ri = i+1;
			while(li>=0 && ri<n && str[li] == str[ri]){
				++pArr[i];
				--li;
				++ri;
			}
			ri = pArr[i] + i;// 更新 R C
			if(ri > R){
				R = ri;
				C = i;
			}
		}else{// 情况2
			int ii = 2*C -i; // i'坐标, i'与i以C为对称轴
			int RrI = R - i;
			pArr[i] == min(pArr[ii], RrI);
			if(pArr[ii]<RrI){//情况2 - 1
				pArr[i] == pArr[ii];
			}else if(pArr[ii] > RrI){ // 情况2 - 2
				pArr[i] = RrI;
			}else{// 情况2 - 3
				int li = 2*i-R-1, ri = R+1;
				pArr[i] = RrI;
				while(li>=0 && ri <n && str[li] == str[ri]){
					++pArr[i];
					--li;
					++ri;
				}
				ri = pArr[i] + i;//更新 R C
				if(ri > R){
					R = ri;
					C = i;
				}
			}
		}
	}
	int maxL = pArr[0], maxI = 0;
	for(int i = 1;i<n;++i ){
		if(pArr[i] > maxL){
			maxL = pArr[i];
			maxI = i;
		}
	}
	for(int i = maxI-maxL;i<=maxI+maxL;++i ){
		if(str[i] == '#'){
			continue;
		}
		sRes+=str[i];
	}
	return sRes;
}


```

时间复杂度 $O(n)$

### 3.4优化代码

```c++
string longestPalindrome(string s) {
	int n = s.size();
	if(n == 0|n==1) return s;
	
	string str = "#"; // 扩充源字符为 #a#b#c#
	for(int i = 0; i < n; ++i){
	str+=s[i];
	str+='#';
	}
	n = str.size();// 获取新字符串的长度
	
	string sRes = "";// 结果
	
	vector<int> pArr; // 记录每一个字符为核心时，最长回文子串
	for(int i = 0; i < n; ++i){
		pArr.push_back(0);
	}
	
	int R = -1, C = -1; // 记录最右边界以及最右边界时的中心点位置
	
	for(int i = 0; i < n; ++i){
			int ii = 2*C -i; // i' 坐标, i' 与 i 以C为对称轴
			int RrI = R - i; // R - i i' - L
		pArr[i] = i>R ? 0 : min(pArr[ii], RrI);// 不用验证的区域，回文半径数组
		int li = i-pArr[i]-1, ri = i+pArr[i]+1;
		while(li>=0 && ri <n && str[li] == str[ri]){ // 需要验证的区域
			++pArr[i];
			--li;
			++ri;
		}
		ri = pArr[i] + i;// 更新 R C
		if(ri > R){
			R = ri;
			C = i;
		}
	}
	int maxL = pArr[0], maxI = 0;
	for(int i = 1;i<n;++i ){
		if(pArr[i] > maxL){
			maxL = pArr[i];
			maxI = i;
		}
	}
	for(int i = maxI-maxL;i<=maxI+maxL;++i ){
		if(str[i] == '#'){
			continue;
		}
		sRes+=str[i];
	}
	return sRes;
}
```
