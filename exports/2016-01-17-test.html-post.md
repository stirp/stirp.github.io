~~~~{cpp}

//#include <iostream>
#include <cstdio>
#include <cstdlib>
#include <cstring>
#include <cmath>
#include <algorithm>
#include <cctype>
using namespace std;
typedef long long ll;
typedef unsigned long long ull;

int iRead()
{
	int ret = 0;
	int ch;
	while(!isdigit(ch = getchar()) && ch != '-');
	bool bm = (ch == '-'); if(bm) ch = getchar();
	while(ret = ret * 10 + (ch - '0'), isdigit(ch = getchar()));
	return bm ? -ret : ret;
}

const int N = 300003;
const int STP = 100000;
struct iT;
iT *q[N];
struct iT
{
    iT *fa, *ch[2];
    int val, s;
    bool rev;
    #define iDir(c) ((c)->fa->ch[1] == (c))
    void up()
    {
        s = 1 + 
            (ch[0] ? ch[0]->s : 0) + 
            (ch[1] ? ch[1]->s : 0);
    }
    void down()
    {
        if(!rev) return;
        rev = 0;
        swap(ch[0], ch[1]);
        if(ch[0]) ch[0]->rev ^= 1;
        if(ch[1]) ch[1]->rev ^= 1;
    }
    void rot() // 把this转到fa的位置 
    {
        iT *f = fa;
        if(f->fa) f->fa->ch[iDir(f)] = this;
        int dr = iDir(this);
        fa = f->fa, f->fa = this;
        if(ch[!dr]) ch[!dr]->fa = f;
        f->ch[dr] = ch[!dr], ch[!dr] = f;
        f->up(), up();      // !!!! 此时f已经是this的孩子了 
    }
    iT *splay(iT *tag = NULL) // 把this转到tag下，返回this 
    {
        iT **p = q;
        for(iT *i = this; i != tag; i = i->fa)
            *p++ = i;
        while(p != q)
            (*--p)->down();
        for(; fa != tag; rot())
            if(fa->fa != tag)   // 双旋的情况 
                (iDir(this) == iDir(fa) ? fa : this)->rot();
        return this;
    }
    iT *select(int k) // 取rank k 从0开始 
    {
        down();
        int sl = ch[0] ? ch[0]->s : 0;
        if(sl == k)
            return this;
        else if(k < sl)
            return ch[0]->select(k);
        else
            return ch[1]->select(k - sl - 1);
    }
    iT *min()
    {
        iT *c = this;
        while(c->down(), c->ch[0])
            c = c->ch[0];
        return c;
    }
}ss[N], *sp = ss;
int arr[N];

// 建[l,r)的高度平衡splay 像线段树一样 感觉快一点 当然也可以一个个insert & splay 
iT *iMake(int l, int r, iT *f = NULL)
{
    if(l >= r) return NULL;
    int m = (l + r) >> 1;
    iT *c = sp++;
    *c = (iT){
        f, 
        {iMake(l, m, c), iMake(m + 1, r, c)}, 
        arr[m], r - l, false
    };
    return c;
}

int n;

int main()
{
    n = iRead();
    for(int i = 0; i < n; ++i)
        arr[i] = iRead();
    iT *rt = iMake(0, n);
    int i;
	for(i = 0; i <= STP; ++i)
	{
        int c = rt->min()->val;
        if(c == 1)
            break;
        if(c == n)          // !!!! 特判翻转全部的情况 
            rt->rev ^= 1;   // 否则下面会select(n) 然后炸掉 
        else
        {
            rt = rt->select(c)->splay(); // splay完，根已经换了 
            rt->ch[0]->rev ^= 1;
        }
	}
	printf("%d\n", i <= STP ? i : -1);
	fclose(stdout);
	return 0;
}

~~~~
