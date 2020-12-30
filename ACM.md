

# ACM模板

## 1	一切的开始

### 1.1	头文件+优化

```cpp
#pragma GCC optimize("Ofast","unroll-loops","omit-frame-pointer","inline")
#pragma GCC optimize(3 , "Ofast" , "inline")
#pragma GCC optimize("Ofast")
#pragma GCC target("avx,avx2,fma")
#pragma GCC optimization("unroll-loops")
#include <iostream>
#include <cstdio>
#include <algorithm>
#include <unordered_map>
#include <vector>
#include <map>
#include <list>
#include <queue>
#include <cstring>
#include <cstdlib>
#include <ctime>
#include <cmath>
#include <stack>
#include <set>
#include <bitset>
#include <deque>
using namespace std ;
#define ios ios::sync_with_stdio(false) , cin.tie(0) , cout.tie(0)
#define x first
#define y second
#define pb push_back
#define ls rt << 1
#define rs rt << 1 | 1
typedef long long ll ;
const double esp = 1e-6 , pi = acos(-1) ;
typedef pair<int , int> PII ;
const int N = 1e6 + 10 , INF = 0x3f3f3f3f , mod = 1e9 + 7;
```

### 1.2	cin同步开关

```cpp
ios ios::sync_with_stdio(false) , cin.tie(0) , cout.tie(0)
// 注意即便关掉cin同步，对于大数据输入输出依旧不如scanf
// 谨记：使用cout 换行功能时，一定不要使用 cout << endl , 这会造成每次都清空缓存，速度超级慢 ，应该使用cout << "\n" ;
```

## 2	图论

### 2.1	图论建图

#### 2.1.1	vector<int> v[N]

```cpp
边a - b , 无边权
v[a].pb(b) , v[b].pb(a) ;
```

#### 2.1.2	vector<PII> v[N]

```cpp
边a - b , 边权c
v[a].pb({b , c}) , v[b].pb({a , c}) ;
```

#### 2.1.3	链式前向星

```cpp
int e[N] , ne[N] , w[N] , h[N] , idx ;
void add(int a , int b , int c) {
  e[idx] = b , ne[idx] = h[a] , w[idx] = c , h[a] = idx ++ ;
}
memset(h , -1 , sizeof h) ; // 初始化
```

### 2.2	经典最短路

#### 2.2.1	堆优化dijkstra

```cpp
// dijkstra 需无负边权
const int N = 1e6 + 10 , INF = 0x3f3f3f3f , mod = 1e9 + 7;
int e[N] , ne[N] , w[N] , h[N] , idx , vis[N] ,  n , m , dis[N] ;
void add(int a , int b , int c) {
  e[idx] = b , ne[idx] = h[a] , w[idx] = c , h[a] = idx ++ ;
}
void dijkstra() {
  priority_queue<PII , vector<PII> , greater<PII>> q ;
  q.push({0 , 1}) ;
  memset(dis , 0x3f , sizeof dis) ;
  dis[1] = 0 ;
  while(q.size()) {
    int u = q.top().y ;
    q.pop() ;
    if(u == n) return dis[u] ;
    if(vis[u]) continue ;
    vis[u] = 1 ;
    for(int i = h[u] ; ~i ; i = ne[i]) {
      int v = e[i] ;
      if(dis[v] > dis[u] + w[i]) {
        dis[v] = dis[u] + w[i] ;
        q.push({dis[v] , v}) ;
      }
    }
  }
    return dis[n] ;
}
```

#### 2.2.2	SPFA

```cpp
// 写最短路时，尽量不要用spfa，有时候会被直接卡掉，尽量用dijkstra
int e[N] , ne[N] , w[N] , h[N] , idx , vis[N] , n , m , dis[N] , cnt[N]  ;
void add(int a , int b , int c) {
  e[idx] = b , ne[idx] = h[a] , w[idx] = c , h[a] = idx ++ ;
}
bool spfa() {
  queue<int> q ;
  for(int i = 1; i <= n ;i ++ ) q.push(i) , vis[i] = 1 , dis[i] = 0  ;
  while(q.size()) {
    int u = q.front() ;
    q.pop() ;
    vis[u] = 0 ;
    if(cnt[u] >= n) return true ;
    for(int i = h[u] ; ~i ; i = ne[i]) {
      int v = e[i] ;
      if(dis[v] > dis[u] + w[i]) {
        dis[v] = dis[u] + w[i] ;
        cnt[v] ++ ;
        if(!vis[v]) {
          q.push(v) ;
          vis[v] = 1 ;
          if(cnt[v] >= n) return true ;
        }
      }
    }
  }
  return false ;
}
```

#### 2.2.4	Floyd

```cpp
int f[210][210] ;
memset(f , 0x3f ,sizeof f) ;
for(int i = 1 ;i <= n ;i ++ ) f[i][i] = 0 ;  
for(int k = 1; k <= n ;k ++ ) {
    for(int i = 1; i <= n ;i ++ ) {
      for(int j = 1; j <= n ;j ++ )
       if(f[i][k] != INF && f[k][j] != INF)
         f[i][j] = min(f[i][j] , f[i][k] + f[k][j]) ;
    }
}
```

### 2.3	分层最短路

**分层图最短路是指在可以进行分层图的图上解决最短路问题。分层图：可以理解为有多个平行的图。**

**一般模型是：在一个正常的图上可以进行 k 次决策，对于每次决策，不影响图的结构，只影响目前的状态或代价。一般将决策前的状态和决策后的状态之间连接一条权值为决策代价的边，表示付出该代价后就可以转换状态了。**

**一般有两种方法解决分层图最短路问题：**

1. **建图时直接建成k+1层。**
2. **多开一维记录机会信息。也可以想成二维dp维护**     

```cpp
/*
分层最短路的思想，其实就是在原图上建立很多的辅助点，然后用一个现有的算法(比如最短路算法)就可以很简单的解决问题
其中两个最常用的解决问题方法：多层建图、二维dp维护
*/
```

#### 2.3.1	多层建图

**我们建k+1层图。然后有边的两个点，多建一条到下一层边权为0的单向边，如果走了这条边就表示用了一次机会。**

**有N个点时，1~n表示第一层，（1+n）~（n+n）代表第二层，（1+2\*n）~（n+2\*n）代表第三层，（1+i\*n）~（n+i\*n）代表第i+1层。因为要建K+1层图，数组要开到n \* ( k + 1)，点的个数也为n \* ( k + 1 ) 。**

**对于数据：**

**n  =  4，m  =  3， k  =  2**

**0    1     100**

**1    2    100**

**2    3    100**

**建成图之后大概是这样的：**

![image-20201230141051149](C:\Users\spnooyseed\AppData\Roaming\Typora\typora-user-images\image-20201230141051149.png)



意思就是这样的：如果免费使用一次机会的话，如果从u到v使用这次机会，那么就从u -> v + n,例如上图从0节点走到1 + n节点。

这样就相当于免费走了一次。

**注意：这个做法就相当于使用了kn倍的空间**

> 例题：**通信线路**
>
> 在郊区有 N 座通信基站，P 条 **双向** 电缆，第 i 条电缆连接基站AiAi和BiBi。
>
> 特别地，1 号基站是通信公司的总站，N 号基站位于一座农场中。
>
> 现在，农场主希望对通信线路进行升级，其中升级第 i 条电缆需要花费LiLi。
>
> 电话公司正在举行优惠活动。
>
> 农产主可以指定一条从 1 号基站到 N 号基站的路径，并指定路径上不超过 K 条电缆，由电话公司免费提供升级服务。
>
> 农场主只需要支付在该路径上剩余的电缆中，升级价格最贵的那条电缆的花费即可。
>
> 求至少用多少钱可以完成升级。
>
>  **输入格式**
>
> 第1行：三个整数N，P，K。
>
> 第2..P+1行：第 i+1 行包含三个整数Ai,Bi,LiAi,Bi,Li。
>
> **输出格式**
>
> 包含一个整数表示最少花费。
>
> 若1号基站与N号基站之间不存在路径，则输出”-1”。
>
>  **数据范围**
>
> 0≤K<N≤10000≤K<N≤1000,
> 1≤P≤100001≤P≤10000,
> 1≤Li≤10000001≤Li≤1000000
>
> **输入样例：**
>
> ```cpp
> 5 7 1
> 1 2 5
> 3 1 4
> 2 4 8
> 3 2 3
> 5 2 9
> 3 4 7
> 4 5 6
> ```
>
> **输出样例：**
>
> ```cpp
> 4
> ```

```cpp

int e[N * 2] , ne[N * 2] , h[N * 2] , idx , w[N * 2] , n , k , p;
void add(int a , int b , int c) {
  e[idx] = b, ne[idx] = h[a] , w[idx] = c , h[a] = idx ++ ;
}
int dis[N] ;
bool vis[N] ;
void dijkstra() {
  priority_queue<PII , vector<PII> , greater<PII> > q ;
  memset(dis , 0x3f , sizeof dis) ;
  dis[1] = 0 ;
  q.push({0 , 1}) ;
  while(q.size()) {
    int u = q.top().y ;
    q.pop() ;
    if(vis[u]) continue ;
    vis[u] = 1 ;
    // cout << " u ==  " << u << "\n" ;
    for(int i = h[u] ; ~i ; i = ne[i]) {
      int v = e[i] ;
      int maxn = max(dis[u] , w[i]) ;
      if(dis[v] > maxn) {
        // cout << " v == " << v << " " << maxn << "\n" ;
        dis[v] = maxn;
        q.push({dis[v] , v}) ;
      }
    }
  }
}
int work()
{
  memset(h , -1 , sizeof h) ;
  cin >> n >> p >> k ;
  for(int i = 1 ;i <= p ; i ++ ) {
    int a , b , c ;
    cin >> a >> b >> c ;
    // 当前层建图
    add(a , b , c) ;
    add(b , a , c) ;
    for(int j = 0 ;j < k ;j ++ ) {
      add(a + j * n , b + (j + 1) * n , 0) ; // 当前层和下一层建图、这个是单向，表示a->b免费使用一次
      add(b + j * n , a + (j + 1) * n , 0) ; // 当前层和下一层建图、这个也是单向，表示b -> a免费使用一次，切记建立双边
      add(b + (j + 1) * n , a + (j + 1) * n , c) ;
      add(a + (j + 1) * n , b + (j + 1) * n , c) ;
    }
  }
  dijkstra() ;
  int ans = INF ;
  for(int i = 0 ;i <= k + 1; i ++ )
   ans = min(ans , dis[i * n]) ;
  if(ans == INF) ans = -1 ;
  cout << ans << "\n" ;
  return 0 ;
}
```

#### 2.3.2	二维dp维护

```cpp
// dp[i][j] 表示到u这个点使用了j次免费机会，那么转移就是
// dp[v][0] = max(dp[u][0] , w[i]) ;
// dp[v][j] = min(dp[u][j - 1] , max(dp[u][j] , w[i])) ;
int dp[1100][1100] ;
int n , m , k , e[N] , ne[N] , w[N] , h[N] , idx ;
void add(int a , int b , int c) {
  e[idx] = b , ne[idx] = h[a] , w[idx] = c , h[a] = idx ++ ;
}
bool vis[N] ;
void dijkstra() {
  queue<int> q ;
  q.push(1) ;
  memset(dp , 0x3f , sizeof dp) ;
  dp[1][0] = 0 ;
  vis[1] = 1 ;
  while(q.size()) {
    int u = q.front() ;
    q.pop() ;
    vis[u] = 0 ;
    for(int i = h[u] ; ~i ; i = ne[i]) {
      int v = e[i] ;
      int maxn = max(dp[u][0] , w[i]) ;
      if(dp[v][0] > maxn) {
        dp[v][0] = maxn ;
        if(!vis[v]) q.push(v) , vis[v] = 1 ;
      }
      for(int j = 1 ;j <= k ;j ++ ) {
        int maxn = min(dp[u][j - 1] , max(dp[u][j] , w[i])) ;
        if(dp[v][j] > maxn) {
          dp[v][j] = maxn ;
          if(!vis[v]) vis[v] = 1 , q.push(v) ;
        }
      }
    }
  }
}
int work()
{
  cin >> n >> m >> k ;
  memset(h , -1 , sizeof h) ;
  for(int i = 1; i <= m ;i ++ ) {
    int a , b , c ;
    cin >> a >> b >> c ;
    add(a , b , c) , add(b , a , c) ;
  }
  dijkstra() ;
  int ans = INF ;
  for(int i = 0 ; i <= k ;i ++ )
   ans = min(ans , dp[n][i]) ;
  if(ans == INF) ans = -1 ;
  cout << ans << "\n" ;
  return 0 ;
}
```

### 2.3	差分约束

**学习链接：http://www.cppblog.com/menjitianya/archive/2015/11/19/212292.html**

> 给定 n 个区间 [ai,biai,bi]和 n 个整数 cici。
>
> 你需要构造一个整数集合 Z，使得∀i∈[1,n]∀i∈[1,n],Z 中满足ai≤x≤biai≤x≤bi的整数 x 不少于 cici 个。
>
> 求这样的整数集合 Z 最少包含多少个数。

设f[x] 为前x个数里面最少选择多少个数，那么每个条件就是f[b[i]] - f[a[i] - 1] >= c[i]  , 然后这就是差分约束

1. f[b[i]] - f[a[i] - 1] >= c[i]   --------  加入边 a[i] - 1  ---- >  b[i] ,, 边权为c[i] 
2. 对于每个点都和0点连边 0 - > i ，边权为0 
3. 对于f[i] - f[i - 1] >= 0 , i - 1 --> i ,边权为0 
4. 对于f[i] - f[i - 1] <= 1 --- > f[i - 1] - f[i] >= -1 , 所以建立边 i  --> i - 1 , 边权为 - 1 
5. 最后求解最长边

```cpp
int n , e[N] , ne[N] , w[N] , h[N] , idx ;
void add(int a,  int b , int c) {
  e[idx] = b , ne[idx] = h[a] , w[idx] = c , h[a] = idx ++ ;
}
int vis[N] ;
ll dis[N] ;
void spfa() {
  queue<int> q ;
  memset(dis , -0x3f , sizeof dis) ;
  q.push(0) ;
  dis[0] = 0 ;
  vis[0] = 1 ;
  while(q.size()) {
    int u = q.front() ;
    q.pop() ;
    vis[u] = 0 ;
    for(int i = h[u] ; ~i ; i = ne[i]) {
      int v = e[i] ;
      if(dis[v] < dis[u] + w[i]) {
        dis[v] = dis[u] + w[i] ;
        if(!vis[v]) {
          vis[v] = 1 ;
          q.push(v) ;
        }
      }
    }
  }
}
int work()
{
  cin >> n ;
  memset(h , -1 , sizeof h) ;
  for(int i = 1; i <= n ;i ++ ) {
    int a , b , c ;
    cin >> a >> b >> c ;
    add(a + 1 , b + 2 , c) ;
  }
  for(int i = 1 ;i <= 50003 ;i ++ )
   add(i , i + 1 , 0) , add(i + 1 , i , -1) , add(0 , i , 0) ;
  ll ans = 0 ;
  spfa() ;
  for(int i = 1; i <= 50003 ;i ++ )
   ans = max(ans , dis[i]) ;
  cout << ans << "\n" ;
  return 0 ;
}
```



## 6	数论

### 6.6	0/1分数规划

所谓的0/1分数规划，就是求解
$$
\frac{\sum_i^n{a[i]}*x[i]}{\sum_i^n{b[i]}*x[i]} , x[i] = 0 , 1
$$
这个公式最值。

也就是对于一系列数组a 和 b，其中每个数组都有选择与不选。然后求解其公式最值。

如果我们假设其最小值等于L ，那么得到
$$
\frac{\sum_i^n{a[i]}*x[i]}{\sum_i^n{b[i]}*x[i]} >= L
$$
移向得到
$$
{\sum_i^n{a[i]}*x[i]} >= L * {\sum_i^n{b[i]}*x[i]}\\=>{\sum_i^n{a[i]}*x[i]} - L * {\sum_i^n{b[i]}*x[i]} >= 0
$$
可以发现如果我们枚举L 这个最小值，那么可以判断这个公式是否成立，也就相当于我们可以枚举L时，判断L是否合法，符合单调性。

然后我们就可以二分求解。

> **观光奶牛**
>
> 给定一张L个点、P条边的有向图，每个点都有一个权值f[i]，每条边都有一个权值t[i]。
>
> 求图中的一个环，使“环上各点的权值之和”除以“环上各边的权值之和”最大。
>
> 输出这个最大值。
>
> **注意**：数据保证至少存在一个环。

**即需要找到一个环**
$$
max \frac{\sum f[i]}{\sum t[i]} 
$$
假设最大值为ans , 那么如果设mid = 当前值，如果ans >= mid ， 即
$$
\frac{\sum f[i]}{\sum t[i]} >= mid\\->
mid * \sum t[i] - \sum f[i] <= 0\\->
\sum (mid*t[i]-f[i]) <= 0
$$
如果我们将每条边全设置为mid * 原本的边权- 出点的点权， 那么这个公式就表示出现负环。

现在我们就直接枚举mid ， 如果符合上述条件，出现负环，则l = mid , 否则r = mid ;



```cpp
/*
 *@author spnooyseed
 */
#pragma GCC optimize("Ofast","unroll-loops","omit-frame-pointer","inline")
#pragma GCC optimize(3 , "Ofast" , "inline")
#pragma GCC optimize("Ofast")
#pragma GCC target("avx,avx2,fma")
#pragma GCC optimization("unroll-loops")
#include <iostream>
#include <cstdio>
#include <algorithm>
#include <unordered_map>
#include <vector>
#include <map>
#include <list>
#include <queue>
#include <cstring>
#include <cstdlib>
#include <ctime>
#include <cmath>
#include <stack>
#include <set>
#include <bitset>
#include <deque>
using namespace std ;
#define ios ios::sync_with_stdio(false) , cin.tie(0)
#define x first
#define y second
#define pb push_back
#define ls rt << 1
#define rs rt << 1 | 1
typedef long long ll ;
const double esp = 1e-6 , pi = acos(-1) ;
typedef pair<int , int> PII ;
const int N = 10000 + 10 , INF = 0x3f3f3f3f , mod = 1e9 + 7;
int n , m ;
double f[N] ;
struct node {
  int u , v ;
  double w ;
}edge[N];
int e[N] , ne[N] , h[N] , idx ;
double w[N] ;
void add(int a , int b , double c) {
  e[idx] = b , ne[idx] = h[a] , w[idx] = c , h[a] = idx ++ ;
}
int cnt[N] , vis[N] ;
double dis[N] ;
bool check(double mid) {
  idx = 0 ;
  memset(h , -1 , sizeof h) ;
  for(int i = 1; i <= m ;i ++ )
    add(edge[i].u , edge[i].v , edge[i].w * mid - f[edge[i].u]) ;
  queue<int> q ;
  for(int i = 1; i <= n ;i ++ )  dis[i] = 0 , vis[i] = 1 , q.push(i) , cnt[i] = 0;
  while(q.size()) {
    int u = q.front() ;
    q.pop() ;
    vis[u] = 0 ;
    if(cnt[u] >= n) return true ;
    for(int i = h[u] ; ~i ; i = ne[i]) {
      int v = e[i] ;
      if(dis[v] > dis[u] + w[i]) {
        dis[v] = dis[u] + w[i] ;
        cnt[v] ++ ;
        if(!vis[v]) {
          vis[v] = 1 ;
          q.push(v) ;
          if(cnt[v] >= n) return true ;
        }
      }
    }
  }
  return false ;
}
int work()
{
  cin >> n >> m ;
  for(int i = 1; i <= n ;i ++ ) cin >> f[i] ;
  for(int i = 1; i <= m ;i ++ )
    cin >> edge[i].u >> edge[i].v >> edge[i].w ;
  double l = 0.0 , r = INF , ans = 0 ;
  while(fabs(r - l) > esp) {
    double mid = (l + r) / 2.0 ;
    if(check(mid)) l = mid , ans = mid ;
    else r = mid ;
  }
  printf("%.2lf\n" , ans) ;
  return 0 ;
}
int main()
{
  //   freopen("C://Users//spnooyseed//Desktop//in.txt" , "r" , stdin) ;
  //   freopen("C://Users//spnooyseed//Desktop//out.txt" , "w" , stdout) ;
  work() ;
  return 0 ;
}
/*
*/

```

