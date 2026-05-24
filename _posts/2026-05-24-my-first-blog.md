---
layout:post
title:洛谷 P4785 [BalticOI 2016] 交换 (Day2)
date:2026-05-24
---

我的分析与解答：
注意到 x 可以与 ⌊x/2⌋ 交换，即 k,2k,2k+1 之间可以交换，容易联想到完全二叉树。
假设位置 k,2k,2k+1 的值分别是 A,B,C，考虑分类讨论。
对于 A 最小的情况，显然不应该交换。因为字典序可以贪心的选择，如果 k 的位置比 A 大，那么在前面都一样的情况下，显然直接取 A 是最优的。
对于 B 最小的情况，显然应该让 k,2k 交换，原因与上面类似。
对于 C 最小的情况，应该让让 k,2k+1 交换。但是特殊的情况是无论是否交换 k,2k 都可以使 k 这个位置取到最小的 C。
假设 A 为次小值，需要注意将 A 放到 2k 的位置并不是最优。
考虑设 f(x,v) 为 x 这个位置在处理了 ⌊x/2⌋ 之后为 v 时，v 这个值可以移动到的最小位置。
对于上面的情况，我们就只需要考虑 f(2k,a),f(2k+1,a) 的大小关系，a 就应该换到小的那个子树。
考虑证明这个决策的正确性，不妨设放到 2k 得到的序列为 x，反之为 y，且 f(2k,a)<f(2k+1,a)。
要证决策正确性，只需要证明 ∀i∈[1,min(f(2k,a),f(2k+1,a)))∩N 满足 xi=yi且 xf(2k,a)<yf(2k,a)。
要证明 ∀i∈[1,min(f(2k,a),f(2k+1,a)))∩N 满足 xi=yi，只需证 ∀i∈[1,k]∪(k,min(f(2k,a),f(2k+1,a))))∩N 满足 xi=yi。
因为 ∀i∈[1,k]∩N 的选择已经完成，那么显然都有 xi=yi。
因为 a 这个值可以一直移动到 f(2k,a)，说明在在从 2k 向下一直转移到 f(2k,a) 都没有遇到取 a 是最优的情况。
那么如果 2k 这个位置放比 a 还要劣的 b，在从 2k 向下遍历的时候遇到取 b 是最优的一定不早于 f(2k,a) ，对于在 2k+1 的子树也是一样的。
这就证明了 ∀i∈[1,min(f(2k,a),f(2k+1,a)))∩N 满足 xi=yi。
根据定义因为 a 会放到 f(x,a)，那么 2f(x,a),2f(x,a)+1 都没有 a 优秀，所以无论这个位置取 b 还是 2f(x,a),2f(x,a)+1 对应的值都没有 a 优秀，因此策略的正确性得证。
考虑通过深度优先搜索求解 f(x,v)。
如果遇到第 1 种情况或者遇到了根节点，那么显然就可以直接回溯求出答案，因为此时位置已经确定。
如果遇到第 2 种情况，那么显然应该继续进入左子树进行递归。
对于第 3 种情况就模仿现在的情况进行递归两个儿子就可以了。

我的代码：
#include <bits/stdc++.h>
using namespace std;
#define int long long
const int MAXN = 4e5 + 10;
int n,a[MAXN],ans[MAXN];
#define ls id << 1
#define rs id << 1 | 1
map <int,int> mp[MAXN];
inline int dfs(int id,int val) {
	if(mp[id].count(val)) return mp[id][val];
	int res = id,A = val,B = a[ls],C = a[rs];
	if(B == 0 && C == 0) res = id;
	else if(C == 0) {
		if(B < A) res = ls;
		else res = id;
	} else {
		if(A < B && A < C) res = id;
		else if(B < A && B < C) res = dfs(ls,val);
		else {
			int mx = max(A,B),mn = min(A,B);
			int LS = dfs(ls,mn),RS = dfs(rs,mn);
			if(A < B) res = min(LS,RS);
			else {
				if(LS < RS) res = dfs(rs,val);
				else res = dfs(ls,val);
			}
		}
	} return mp[id][val] = res;
}
inline void solve(int id) {
	int A = a[id],B = a[ls],C = a[rs];
	if(B == 0 && C == 0) return;
	else if(C == 0) {
		if(B < A) swap(a[id],a[ls]);
		return;   
	} else {
		if(A < B && A < C) solve(ls),solve(rs);
		else if(B < A && B < C) {
			swap(a[id],a[ls]);
			solve(ls),solve(rs); 
		} else {
			a[id] = C;
			int mx = max(A,B),mn = min(A,B);
			int LS = dfs(ls,mn),RS = dfs(rs,mn);
			if(LS < RS) a[ls] = mn,a[rs] = mx;
			else a[rs] = mn,a[ls] = mx;
			solve(ls),solve(rs);
		}
	} return;
}
signed main() {
	cin >> n;
	for(int i = 1;i <= n;i++) cin >> a[i];
	solve(1);
	for(int i = 1;i <= n;i++) cout << a[i] << " ";	
	return 0; 
} 
