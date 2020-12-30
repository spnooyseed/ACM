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
  memset(dis , 0x3f , sizeof dis) ;
  dis[1] = 0 ;
  for(int i = 1; i <= n ;i ++ ) q.push(i) , vis[i] = 1 ;
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
       // a -- , b -- ;
       add(a , b , c) ;
       add(b , a , c) ;
       for(int j = 0 ;j < k ;j ++ ) {
         add(a + j * n , b + (j + 1) * n , 0) ;
         add(b + j * n , a + (j + 1) * n , 0) ;
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

   #### 2.3.1	dp维护