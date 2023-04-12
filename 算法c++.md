- [算法思想](#算法思想)
  - [二分](#二分)
    - [STL 二分库函数](#stl-二分库函数)
  - [前缀和与差分](#前缀和与差分)
  - [dp](#dp)
    - [线性dp（字符串编辑距离）](#线性dp字符串编辑距离)
    - [区间dp（合并石子，最长回文子序列）](#区间dp合并石子最长回文子序列)
    - [树形DP(打家劫舍III，没有上司的舞会 )](#树形dp打家劫舍iii没有上司的舞会-)
    - [状态压缩DP（最短Hamilton路径，1723. 完成所有工作的最短时间, 1125. 最小的必要团队）](#状态压缩dp最短hamilton路径1723-完成所有工作的最短时间-1125-最小的必要团队)
    - [数位DP](#数位dp)
  - [dfs（回溯）](#dfs回溯)
  - [bfs（拓扑排序）](#bfs拓扑排序)
    - [平衡树+BFS](#平衡树bfs)
    - [并查集+BFS](#并查集bfs)
  - [图论](#图论)
    - [无向图（bfs，dfs）](#无向图bfsdfs)
    - [有向图（拓扑排序）](#有向图拓扑排序)
    - [最短路](#最短路)
      - [单源最短路](#单源最短路)
        - [边权为正（dijkstra）](#边权为正dijkstra)
        - [存在负边](#存在负边)
      - [多源汇最短路（Floyd）](#多源汇最短路floyd)
    - [最小生成树（prim，Kruskal）](#最小生成树primkruskal)
    - [二分图（染色，匈牙利算法）](#二分图染色匈牙利算法)
  - [数学](#数学)
    - [快速幂](#快速幂)
    - [组合数](#组合数)
    - [约瑟夫问题](#约瑟夫问题)
  - [贪心](#贪心)
  - [拓扑排序（图论）](#拓扑排序图论)
  - [滑动窗口](#滑动窗口)
- [数据结构](#数据结构)
  - [树](#树)
  - [Trie树](#trie树)
  - [并查集](#并查集)
  - [单调栈](#单调栈)
  - [树状数组](#树状数组)
  - [线段树](#线段树)
- [输入输出](#输入输出)
  - [更快的输入输出](#更快的输入输出)
- [典型问题](#典型问题)
  - [背包问题](#背包问题)
    - [01背包](#01背包)
    - [完全背包](#完全背包)
    - [完全背包求 最值 方案](#完全背包求-最值-方案)
    - [01背包求方案数](#01背包求方案数)
    - [多重背包问题](#多重背包问题)
    - [容积、重量双限制](#容积重量双限制)
  - [重写排序](#重写排序)
  - [最大公约数](#最大公约数)
- [C++ 头文件](#c-头文件)
# 算法思想
## 二分
二分思想比较场景，模板就不写了。分析一下二分的场景。

首先是在某种有序数列中查找对应的位置，这类问题需要注意最后返回左右端点的问题。以及考虑是不是会越界。

其次是一类查找能满足xxx条件的最大/小值的问题，这类问题的特点在于**有单调性，并且容易验证可行性**。

另外，就是**基于二分的最长上升子序列问题**。
```c++
// 最长上升子序列
class Solution {
public:
    int lengthOfLIS(vector<int>& nums) {
        // 为什么要做替换，看起来只有最后一个元素真正起到作用了？
        // [0,8,4,12,2,3,4]为例子
        // [0] -> [0,8] -> [0,4] -> [0,4,12]
        // 关键替换 [0,2,12] -> [0,2,3,4]
        // 如果不替换后面的3，4都无法进来
         int n = nums.size();
         vector<int> list;
        // 二分查找思考的点：1. 相等如何处理 2. 位于边界怎么处理
        auto bs = [&](int target){
            int n = list.size();
            int l = 0, r = n-1;
            // 抉择1. 要不要加等号
            while(l<=r){
                int mid = (r+l)/2;
                if(list[mid] < target){
                    l = mid+1;
                }else if (list[mid] > target){
                    r = mid-1;
                }else{
                    // 抉择2. 如果相等，如何处理。
                    return mid;
                }
            }
            // 抉择3. 返回l还是r (如果是大于target的最小值，就是l。小于target的最大值就是r)
            return l;
        };

         for (int i = 0;i < n;i++){
             int cur = nums[i];
             if(list.empty() || list[list.size()-1] < cur){
                 list.push_back(cur);
                 continue;
             }
             // 二分查找 大于 cur的最小值
             int index = bs(cur);
             list[index] = cur;
         }
         return list.size();
    }
};
```

### STL 二分库函数

- `upper_bound()`大于target的最小值，如果找不到返回`nums.end()`。
- `lower_bound()`大于等于target的最小值。

同样的，对于小于target的最大值，我们需要调用`lower_boud`并对得到的it--，这样可以得到正确的数值，但是得不到正确的迭代器或者数值。

```c++
    vector<int> nums = {3,3,4,4,5};
    cout << lower_bound(nums.begin(), nums.end(), 4)-nums.begin() << endl; // nums[1] = 4 第一个大于等于 target的 index
    cout << upper_bound(nums.begin(), nums.end(), 4)-nums.begin() << endl; // nums[3] = 5 第一大于target的index
    
    cout << lower_bound(nums.begin(), nums.end(), 4)-1-nums.begin() << endl; // nums[1] =3  第一个小于target的数值，但是不是最靠前的
    cout << upper_bound(nums.begin(), nums.end(), 4)-1-nums.begin() << endl; // nums[3] = 4 第一个小于等于target的数值，但是不是最靠前的
```

## 前缀和与差分
前缀和的思路往往还是在代码中部分被使用。一般是通过预处理出来前缀和的方法，实现降低复杂度的目的。
[2602. 使数组元素全部相等的最少操作次数](https://leetcode.cn/problems/minimum-operations-to-make-all-array-elements-equal/)
```c++
int n = nums.size();
vector<int> presum(n+1, 0);
for(int i = 0;i<n;i++){
    presum[i+1] = presum[i]+nums[i];
}
```


差分的主要思路是**利用差分统计区间的覆盖频次问题**。维护了一个差分数组，对于每次的覆盖区间，**区间头位置+1，区间结尾+1的位置-1**。最后在进行累加，这样数组每个位置就对应了相应位置的频次。
[1893. 检查是否区域内所有整数都被覆盖](https://leetcode-cn.com/problems/check-if-all-the-integers-in-a-range-are-covered/)
```java
//ranges = [[1,2],[3,4],[5,6]], left = 2, right = 5
class Solution {
    public boolean isCovered(int[][] ranges, int left, int right) {
        int[] diff = new int[52];   // 差分数组
        for (int[] range : ranges) {
            ++diff[range[0]];
            --diff[range[1] + 1];
        }
        // 前缀和
        int curr = 0;
        for (int i = 1; i <= 50; ++i) {
            curr += diff[i];
            if (left <= i && i <= right && curr <= 0) {
                return false;
            }
        }
        return true;
    }
}

```

差分数组更新的信息也都是区间信息，但是我们只需要进行单点查询。这一个是与树状数组和线段树不太一样的一点。

[1589. 所有排列中的最大和](https://leetcode-cn.com/problems/maximum-sum-obtained-of-any-permutation/)

```c++
    int maxSumRangeQuery(vector<int>& nums, vector<vector<int>>& requests) {
        sort(nums.begin(), nums.end());
        int n = nums.size();
        vector<long long> f(n+1);
        // 差分数组 处理好每个位置的头尾跳变
        for(auto &r:requests){
            f[r[0]]++;
            f[r[1]+1]--;
        }
        // f成为 [1,n]的前缀和  从头开始进行累加 这样其实是巧妙的利用了跳变
        for(int i = 1;i<=n;i++){
            f[i] += f[i-1];
        }
        sort(f.begin(), f.end());
        int mod = 1e9+7;
        long long ans = 0;
        for(int i = 0;i<n;i++){
            ans = (ans + f[i+1]*nums[i])%mod;
        }
        return ans;
    }
```
## dp
### 线性dp（字符串编辑距离）

状态设计思路：二维矩阵**dp[i][j]表示word1前i个字符与word2的前j个字符下的子问题**，在本问题中就是代表 word1 到i位置转换成 word2 到j 位置需要最少步数。

状态转移方程：如果word1[i] == word[j]，说明当前两个字符串一样，可以从对角线转移得到。否则考虑其上方，前方，和对角线三个元素的最值转移得到。在本题中dp[i][j] = min(dp[i-1][j-1], dp[i][j-1], dp[i-1][j])+1。

```c++
    int minDistance(string word1, string word2) {
        int n = word1.size();
        int m = word2.size();
        vector<vector<int>> dp(n+1, vector<int>(m+1));
        // dp[i][j]表示指向第i，第j个char时候的最加情况
        for(int i = 0;i<=n;i++) dp[i][0] = i;
        for(int j = 0;j<=m;j++) dp[0][j] = j;
        for(int i = 1;i<=n;i++){
            for(int j = 1;j<=m;j++){
                if(word1[i-1] == word2[j-1]) {
                    // 如果正好匹配，最佳情况一定是匹配
                    dp[i][j] = dp[i-1][j-1];  
                }else{
                    // 否则是三个操作之一，word1替换 dp[i-1][j-1], 删除dp[i-1][j], 插入dp[i][j-1]
                    dp[i][j] = min(min(dp[i-1][j], dp[i][j-1]), dp[i-1][j-1])+1;
                }
            }
        }
        return dp[n][m];
        }
```

### 区间dp（合并石子，最长回文子序列）
时间复杂度一般是O(n^3)。也就是三个循环，首先外层倒叙遍历起始点i = [n-2, 0]，第二层循环终点j = [i+1, n-1]，第三层是分割点k = [i, j-1] (因为一般转移方程是dp[i][j] = max(dp[i][k],dp[k+1][j]) )。
[戳气球](https://leetcode-cn.com/problems/burst-balloons/),[合并石子](https://leetcode-cn.com/problems/minimum-cost-to-merge-stones/)
```java
class Solution {
    public int maxCoins(int[] nums) {
        int n = nums.length;
        int[] arr = new int[n+2];
        n = arr.length;
        arr[0] = 1;
        arr[n-1] = 1;
        for (int i = 1; i<n-1;i++){
            arr[i] = nums[i-1];
        }
        int[][] dp = new int[n][n]; // dp[i][j]表示扎破，开区间
        // n-1是1
        for(int i = n-2;i>=0;i--){
            // 因为是开区间，因此跨越一个没什么意义
            for(int j = i+2;j<n;j++){
                // 必须先+1,这样才是真实的可以戳破的
                for(int k = i+1; k<j;k++){
                    dp[i][j] = Math.max(dp[i][j], dp[i][k]+dp[k][j]+arr[k]*arr[i]*arr[j]);
                }
            }
        
        }
        return dp[0][n-1];
    }
}

```
### 树形DP(打家劫舍III，没有上司的舞会 )
结合了DFS的思路，核心思路在于，维护一个字典，key是root，val是全部的字节点。然后二维的dp[root][0],dp[root][1]表示选取或者不选取的情况。

```python
N = int(input())
w = [0]
for i in range(N):
    w.append(int(input()))
dic = {}
root = sum([i for i in range(1,N+1)])
# 首先构建出来当前的子孙关系
for i in range(N-1):
    son, par = map(int, input().split())
    if par not in dic:
        dic[par] = []
    dic[par].append(son)
    root -= son
dp = [[0]*2 for _ in range(N+1)]   
##  -----------------树形dfs模板------------------------------ ##
def dfs(root):
    dp[root][1] = w[root]
    if root not in dic:
        return
    # 遍历每一个字节点
    for son in dic[root]: 
        dfs(son)
        # 选取当前root节点，因此所有的儿子都不能选取
        dp[root][1] += dp[son][0]
        # 不选择当前的节点，儿子节点可选可不选
        dp[root][0] += max(dp[son][0], dp[son][1])
        
dfs(root)
print(max(dp[root]))

```
### 状态压缩DP（最短Hamilton路径，1723. 完成所有工作的最短时间, 1125. 最小的必要团队）
核心是将多个并存的状态转换为二进制的思想。这个方法的特点在于数据量一般不能很大，因为最大就是31。否则枚举不开。并且这个问题的特点一般在于如何转移得到当前的状态。

两类方法比较多，一个是子集枚举，`for(int p = i; i>0;p = (p-1)&i)`；对于状态为 i 的情况，拆分成 p + （i-p）的情况，将拆出来的 p 独立计算. $dp[i] = dp[i-p]+cal(p)$

另外一个思路是，枚举某个位置的转移`dp[i +(1<<j)]`。一般都是二维的DP。

[统计子树中城市之间最大距离](https://leetcode-cn.com/problems/count-subtrees-with-max-distance-between-cities/submissions/)

### 数位DP
这个Dp思路我感觉更为少见，一般就是求一个方案的方案数。比如整数拆分问题，求解N=n1+n2+n3...的拆分数量。核心思想是知道树的搜索思路。典型题目 [不含连续1的非负整数](https://leetcode-cn.com/problems/non-negative-integers-without-consecutive-ones/)

[blog解析](https://blog.csdn.net/zcz5566719/article/details/120028832)


## dfs（回溯）
两类问题容易使用dfs，一类是需要类似枚举的，比如八皇后的问题，另外就是返回全部的方案的，比如全排列问题。

**dfs问题需要注意不能重复，以及每次传进函数的都应该是深复制。**

回溯算法需要注意一点，**在dfs退出以后，记着恢复现场**。
## bfs（拓扑排序）
bfs是我非常喜欢的一个算法，非常的清晰。使用的场合较多，**树的遍历，可行性的宽搜，以及典型的拓扑排序问题**，这个我会下面仔细分析下。

### 平衡树+BFS
[网格图中最少访问的格子数](https://leetcode.cn/problems/minimum-number-of-visited-cells-in-a-grid/)

```c++
class Solution {
public:
    int minimumVisitedCells(vector<vector<int>>& grid) {
        // 朴素BFS，需要遍历到每一个点。并且需要维护一个visited标记已经被加入queue的点
        // 我们希望可以得到一个更好的方案，可以直接得到还没被加入队列的点。
        // 我们可以注意到，每次在进行bfs时，我们是将一个范围内的点都加入队列里。
        // 1. 平衡树，可以得到符合范围的区间的第一个的iterator
        // 2. 并查集，类似链表，每个点都指向自己的下一个可行节点。
        int n = grid.size();
        int m = grid[0].size();
        if(n == 1 && m ==1)return 1;
        vector<set<int>> row(n+1); // 每一个行都建立一个平衡树，得到这一行的点的访问情况。
        vector<set<int>> col(m+1); // 每一个列都建立一个平衡树，得到这一列的点的访问情况。
        // 初始平衡树。加入n.m比较越界
        for(int i = 0;i<=n;i++){
            for(int j = 0;j<=m;j++){
                row[i].insert(j);
                col[j].insert(i);
            }
        }

        queue<pair<int, int>> q;
        q.emplace(0,0);
        int ans = 1;
        while(!q.empty()){
            ans++;
            int size = q.size();
            for (int i = 0;i<size;i++){
                auto [x,y] = q.front();
                
                q.pop();
                int step = grid[x][y];
                int max_x = min(n-1, x+step);
                int max_y = min(m-1, y+step);
                // y
                for(auto it = row[x].upper_bound(y); *it<= max_y; it = row[x].erase(it)){
                    q.emplace(x, *it);
                    if( x == n-1 && *it == m-1) return ans;
                }
                //x
                for(auto it = col[y].upper_bound(x); *it<= max_x; it = col[y].erase(it)){
                    q.emplace(*it, y);
                    if( *it == n-1 && y == m-1) return ans;
                }
            }
        }
        return -1;

    }
};

```
### 并查集+BFS
使用一个特殊的并查集。每个节点的初始父节点指向自己，被使用以后修改父节点指向到下一个节点。

```c++
class UF{
private:
    vector<int> fa;
public:
    UF(int n){
        for(int i =0;i<n;i++) fa.push_back(i);
    }

    int find(int x){
        if (fa[x] == x){
            return x;
        }
        fa[x] = find(fa[x]);
        return fa[x];
    }

    void merge(int x) {
        fa[x] = x+1;
    }
};
class Solution {
  using Node = tuple<int, int, int>;
  
 public:
  int minimumVisitedCells(vector<vector<int>>& grid) {
    int n = grid.size();
    int m = grid[0].size();

    vector<UF> row_uf(n+1, UF(m+1));
    vector<UF> col_uf(m+1, UF(n+1));

    queue<Node> q;
    q.emplace(1, 0, 0);
    while (!q.empty()) {
      auto [d, x, y] = q.front();
      q.pop();
      if (x == n - 1 && y == m - 1) {
        return d;
      }
      
      int g = grid[x][y];
      
      // right
      UF &row = row_uf[x];
      int right_bound = min(m, g + y + 1);
      // 注意迭代方案，每次需要merge当前点，然后迭代到下一个坐标点
      for (int p = row.find(y+1); p < right_bound; p = row.find(p+1)) {
        q.emplace(d + 1, x, p);
        row.merge(p);

      }
      
      // down
      UF &col = col_uf[y];
      int down_bound = min(n, g + x + 1);
      for (int p = col.find(x + 1); p < down_bound; p = col.find(p+1)) {
        q.emplace(d + 1, p, y);
        col.merge(p);
      }
    }
    return -1;
  }
};

```

## 图论
### 无向图（bfs，dfs）
图的问题一个难点在于建图上，对于无向图，采用字典或者二维列表建图都可以。如果还有权重，那可能需要嵌套的比较麻烦 `List<List<int[]>>`。

这类题目我比较喜欢bfs方法，典型的题目包括，寻找树的重心，从一个点到某个点的最短距离（最短路问题）等。

### 有向图（拓扑排序）
有向图是点之间存在依赖关系。还是采用bfs比较好解决。我们需要维护一个节点的入度，也就是看这个节点上有“几把锁”。在建图时候，我们维护一个依赖数组，key是父节点，val是子节点。然后在遍历时候，将入度为0的点加入队列，按照bfs依次解锁其他点即可。

通用解法：维护一个入度数组indegree[], 邻接表graph[pre][cur]想要解锁cur，需要先学习pre。

### 最短路
给定一幅图，求解点于点之间的最短距离。
#### 单源最短路
从一号点到n号点的最短路径。点的个数是n，边的个数是m。
##### 边权为正（dijkstra）
> 稠密图
朴素的dijkstra算法适合稠密图，只与点的数量有关。O(n^2)。并且由于的稠密图，边比较多，建议**采用邻接矩阵的方法，存储点与点之间的最短距离。**
```java
import java.util.*;
public class Main{
    // ！！！！！！这里不可以是最大整数
    static int INF = Integer.MAX_VALUE/2;

    public static void main(String[] args){
        Scanner sc = new Scanner(System.in);
        int n = sc.nextInt();
        int m = sc.nextInt();
        int[][] dis = new int[n+1][n+1];
        for(int i = 1; i<=n;i++){
            Arrays.fill(dis[i], INF);
        }
        while(sc.hasNext()){
            int a = sc.nextInt();
            int b = sc.nextInt();
            int c = sc.nextInt();
            dis[a][b] = Math.min(dis[a][b],c);
        }
        int ans = dijkstra(dis);
        if (ans == INF)System.out.println(-1);
        else System.out.println(ans);
        return;
        
    }
    
    private static int dijkstra(int[][] dis){
        int n = dis.length;
        HashSet<Integer> set = new HashSet<>();
        int[] ans = new int[n];
        Arrays.fill(ans, INF);
        ans[1] = 0;
        for (int i = 1; i<n;i++){
            int t = -1;
            // 开始进行枚举 
            // 找到没有被考虑过的点到当前点的最小距离。
            for(int j = 1;j<n;j++){
                if (!set.contains(j) && (t == -1 || ans[j]<ans[t])){
                    t = j;
                }
            }
            set.add(t);
            // 全图更新
            for(int j = 1;j<n;j++){
                ans[j] = Math.min(ans[j], ans[t]+dis[t][j]);
            }
        }
        return ans[n-1];
        
    } 
}
```
> 稀疏图 点的数量大于边的数量。O(mlogn)。**采用邻接表的存储，点之间的距离。**

```java
import java.util.*;
public class Main{
    // 这里不可以是最大整数
    static int n;
    static int INF = Integer.MAX_VALUE/2;
    public static void main(String[] args){
        Scanner sc = new Scanner(System.in);
        n = sc.nextInt();
        int m = sc.nextInt();
        List<List<int[]>> dis = new ArrayList<>();
        for (int i = 0;i<n+1;i++) dis.add(new ArrayList<int[]>());
        while(sc.hasNext()){
            int a = sc.nextInt();
            int b = sc.nextInt();
            int c = sc.nextInt();
            dis.get(a).add(new int[]{b,c});
        }
        int ans = dijkstra(dis);
        if (ans == INF)System.out.println(-1);
        else System.out.println(ans);
        return;
        
    }
    
    
    private static int dijkstra(List<List<int[]>> dis){
        int[] ans = new int[n+1];
        Arrays.fill(ans, INF);
        
        HashSet<Integer> set = new HashSet<>();
        
        PriorityQueue<int[]> pq = new PriorityQueue<>(new Comparator<int[]>(){
           //@Override
           public int compare(int[] a, int[] b){
               return a[1]-b[1];
           }
        });
        
        pq.offer(new int[]{1,0});

        while(!pq.isEmpty()){
            int[] now = pq.poll();
            int cur = now[0];
            int distance = now[1];
            
            if (set.contains(cur)) continue;
            
            set.add(cur);
            ans[cur] = distance;
            
            for(int[] next:dis.get(cur)){
                if (set.contains(next[0])) continue;
                if (distance+next[1]<ans[next[0]]){
                    pq.offer(new int[]{next[0], distance+next[1]});
                    ans[next[0]] = distance+next[1];
                }
            }
        }
        
        
        return ans[n];
        
    } 
}
```

##### 存在负边
可以使用Bellman-Ford算法，这个可以解决**限制了最多经过 K 条边到达 n 的最短路径问题**。需要注意，存在负权边时候，如果存在**负权重环**，可能无最短距离。如果第n次迭代，依然有更新最短边，说明存在一个至少为n+1的最短路径，存在负环。

```python
## bellman算法
# 注意这里考虑的是有向边
N,M = map(int, input().split())
g = [[0,0]]
for i in range(M):
    a,b,c = map(int, input().split())
    g.append([a,b,c])

def bellman():
    dis = [float('inf')]*(N+1)
    dis[1] = 0
    
    for i in range(N):
        backup = dis.copy() # 这里需要注意，进行了复制，防止迭代出现混乱
        for j in range(M):
            a,b,c = g[j]
            dis[b] = min(backup[a]+c, dis[b])
            
    if dis[N] == float('inf'):
        return 'impossible'
    return dis[N]
ans = bellman()
print(ans)

```

另外还有，SPFA算法，使用了一个FIFO队列只存储了节点没有存储边的信息，并且使用了标识数组，如果是已经在队列里的将不会再次加入。可以检测负环。除了维护dis以外，还需要维护一个cnt，每次进行状态转移时候，cnt(next) = cnt(cur)+1，如果cnt>N表示存在负环。

```java
import java.util.*;
public class Main{
    static int INF = Integer.MAX_VALUE/2;
    static int n;
    public static void main(String[] args){
        Scanner sc = new Scanner(System.in);
        n = sc.nextInt();
        int m = sc.nextInt();
        List<List<int[]>> graph = new ArrayList<>();
        for(int i = 0;i<=n;i++ ){
            graph.add(new ArrayList<int[]>());
        }
        while(sc.hasNext()){
            int a = sc.nextInt();
            int b = sc.nextInt();
            int c = sc.nextInt();
            graph.get(a).add(new int[]{b,c});
        }
        
        int ans = SPFA(graph);
        if (ans == INF)System.out.print("impossible");
        else System.out.print(ans);
    }
    
    private static int SPFA(List<List<int[]>> graph){
        int[] dist = new int[n+1];
        boolean[] st = new boolean[n+1];
        Arrays.fill(dist, INF);
        Queue<Integer> queue = new LinkedList<Integer>();
        dist[1] = 0;
        queue.add(1);
        st[1] = true;//标记1号点在队列中
        while(!queue.isEmpty()){
            int t = queue.poll();
            st[t] = false;
            for(int[] cur:graph.get(t)){
                int next = cur[0];
                int w = cur[1];
                if(dist[next] > dist[t] + w){
                    dist[next] = dist[t] + w;
                    //判断该点是否已经在队列中
                    if(!st[next]){
                        queue.add(next);
                        st[next] = true; //标记已加入队列
                    }
                }
            }
        }
        return dist[n];
        
    }
}
```
#### 多源汇最短路（Floyd）
多个起点，多个终点。从x号点，到y号点的最短距离。是可以处理**重边，自环和负权边的**。但是因为研究的是最短路问题，因此不能出现负环。

注意这个方法一定是枚举顺序，k，i，j。存储用邻接矩阵

```python
# N,M,Q分别为点的个数，边的个数，和查询的个数
# 注意这里考虑的是有向边
N,M,Q = map(int, input().split())
# 采用邻接矩阵进行存储
dis = [[float('inf')]*(N+1) for _ in range(N+1)]

for i in range(1,N+1):
    dis[i][i] = 0   
for i in range(M):
    a,b,c = map(int, input().split())
    dis[a][b] = min(dis[a][b], c)

def Foldy():
    for k in range(1, N+1):
        for i in range(1,N+1):
            for j in range(1, N+1):
                dis[i][j] = min(dis[i][j], dis[i][k]+dis[k][j])
    return 
    
Foldy()
for i in range(Q):
    a,b = map(int, input().split())
    if dis[a][b] == float('inf'):
        print(-1)
    else:
        print(dis[a][b])

```
### 最小生成树（prim，Kruskal）
https://leetcode.cn/problems/min-cost-to-connect-all-points/

Prim算法是维护了一个dis数组，表示当前的树距离其余各个点的最小距离。每次把最小距离的那个点添加进来；O(n^2)，适用于稠密图
```c++
class Solution {
public:
    int minCostConnectPoints(vector<vector<int>>& points) {
        // 最小生成树：只给了点，边的数量远大于点的数量，因此用prim算法
        int n = points.size();
        vector<vector<int>> g(n, vector<int>(n, 0));
        for (int i = 0;i<n;i++){
            int x1 = points[i][0]; 
            int y1 = points[i][1]; 
            for (int j = i+1;j<n;j++){
               int x2 = points[j][0]; 
               int y2 = points[j][1]; 
               int dis = abs(x1-x2) + abs(y1-y2);
               g[i][j] = g[j][i] = dis;
            }            
        }


        auto prim = [&](){
            int ans = 0;
            vector<int> dis(n, INT_MAX);
            vector<bool> visited(n, false);
            for(int i = 0;i<n;i++){
                int cur = -1;
                for (int j = 0;j<n;j++){
                    // 找到未被访问过的最短的边
                    if(!visited[j] && (cur == -1 || dis[cur]> dis[j])){
                        cur = j;
                    }
                }
                // 无解，无法成树
                if (i != 0 && dis[cur] == INT_MAX) return -1;
                // 这里一定要注意，先累加，再更新，否则会错在自环上
                if (i != 0) ans += dis[cur];
                // 更新每个点到集合的距离 
                for(int j = 0;j<n;j++){
                    dis[j] = min(dis[j], g[j][cur]);
                }
                visited[cur] = true;
            }
            return ans;
        };

        return prim();

    }
};
```

kruskal算法使用了并查集，每次检查距离最小的两个点有没有连通，如果没有就连通，最后会成为一个树。O(mlogm)，是适用于稀疏图，首选。

```c++ 
class Solution {
public:
    int minCostConnectPoints(vector<vector<int>>& points) {
        int n = points.size();
        // 实测这样很慢，性能不如vector<struct>
        vector<vector<int>> g;
        for (int i = 0;i<n;i++){
            int x1 = points[i][0], y1 = points[i][1]; 
            for (int j = i+1;j<n;j++){
               int x2 = points[j][0], y2 = points[j][1]; 
               int dis = abs(x1-x2) + abs(y1-y2);
               g.push_back({dis, i, j});
            }            
        }

        sort(g.begin(), g.end(), [](vector<int> a, vector<int> b) -> int {return a[0] < b[0];});
        UF uf(n);
        
        auto kruskal = [&](){
            int ans = 0;
            int num = 0;
            for(int i = 0;i<g.size() && num < n;i++){
                if (uf.isUnion(g[i][2], g[i][1])) continue;
                ans += g[i][0];
                num++;
                uf.unin(g[i][2],g[i][1]);
            }
            return ans;
        };

        return kruskal();

    }
};
```
### 二分图（染色，匈牙利算法）
二分图的判断：**当且仅当图中不存在奇数环**。其余的都可以染色的方法实现二分。

染色法的思路很简单，BFS的方法，最外面一个循环，如果没染色就染白色，然后放入FIFO队列，邻接的都是黑色的。如果不出现矛盾就是成功的。

匈牙利算法目的，在两个集合中，**寻找到数量最多的一一匹配**。考虑男生与女生配对的问题，依次考虑每个男生，去匹配每个女生，并考虑冲突的女生配对的男生是否存在别的可能。
```python
# 匈牙利算法，求解左右两个图的最大匹配度
# 输入a,b,c，分别是左半侧，右半侧的点和边的数量
n1, n2, m = map(int, input().split())
g = [[]for _ in range(n1+1)] # 只需要存储左边指向右边的边的个数
for _ in range(m):
    a,b = map(int, input().split())
    g[a].append(b)

has = set()  
# 存储当前girl已经匹配的对象
match = [-1]*(n2+1)

def find(x):
    for c in g[x]: # 枚举目前的男生可以选择的全部女生
        if c not in has: # 每个女生只考虑一次，防止嵌套
            has.add(c)
            if match[c] == -1 or find(match[c]): # 如果当前女生还未被匹配，或匹配的男生可以修改
                match[c] = x
                return True
    return False # 只有一切可能都不行才返回False
            
res = 0
for i in range(1,n1+1):
    # -------注意：每次新的循环需要初始化girls的序列---
    has.clear()
    # 匹配成功就+1 从前往后匹配，前面成功就不可修改了。
    if find(i):
        res += 1
print(res)

```
## 数学
### 快速幂

快速幂其实就是个模板，一般用不到。核心就是利用的位运算。
```java
static long mod = 100000007;
static long quick_pow(long a,long b){
        long res=1;
        while(b>0){
           if((b&1)==1) res = res * a % mod;
           a = a * a % mod; // a翻倍
           b >>= 1; // 移位
        }
        return res%mod;
}

```
### 组合数
对于不同的复杂度有不同的要求。
1. O(n^2)要求n,m小于2000
![在这里插入图片描述](https://img-blog.csdnimg.cn/20210317204052852.png)
```java
static int N = 2000;
static int c[][]=new int[N][N];
for(int i=0;i<N;i++)
    for(int j=0;j<=i;j++)
        if(j==0) c[i][j]=1;
        else  c[i][j]=(c[i-1][j]+c[i-1][j-1])%mod;

```
2. 利用逆元求解组合数。时间复杂度O(nlogn) ，nm可以到10^5


![](https://img-blog.csdnimg.cn/20210317204328635.png)

```java

public class Main{
    static final int N=100005;
    static final int mod=(int)1e9+7;
    static long fact[]=new long[N];
    static long infact[]=new long[N]; // 逆元的阶乘
    static long quick_pow(long a,long b){
        long res=1;
        while(b>0){
                if((b&1)==1) res=res*a%mod;
                a=a*a%mod;
                b>>=1;
        }
        return res%mod;
    }
    public static void main(String[] args) {
        Scanner scan=new Scanner(System.in);
        fact[0]=infact[0]=1;
        for(int i=1;i<N;i++){
            fact[i]=fact[i-1]*i%mod;
            infact[i]=quick_pow(fact[i],mod-2)%mod;
        }
        int t=scan.nextInt();
        while(t-->0){
            int a=scan.nextInt();
            int b=scan.nextInt();
            System.out.println(fact[a]*infact[a-b]%mod*infact[b]%mod);
        }
    }
}

```

3. 还可以更快，就是使用lucas定理。但是一般使用不到。
4. 判断组合数的奇偶。也是可以很简单的判断，对于C_n^m，如果`(n&m) == m`则为奇数，否则偶数
### 约瑟夫问题
直接背诵模板吧。共有 n 名小伙伴一起做游戏。小伙伴们围成一圈，按 顺时针顺序 从 1 到 n 编号。确切地说，从第 i 名小伙伴顺时针移动一位会到达第 (i+1) 名小伙伴的位置，其中 1 <= i < n ，从第 n 名小伙伴顺时针移动一位会回到第 1 名小伙伴的位置。
```java
    public int findTheWinner(int n, int k) {
        return find(n,k)+1;// 注意这里的+1不是必须的，只是因为题目的编号是从1开始的。因此我们需要加上1.
    }
    private int find(int n, int k){
        if(n == 1)return 0;
        return (find(n-1,k)+k)%n;
    }
```

## 贪心
世上贪心绝无相同，非常靠经验。

提供几道题作为参考。[完全平方数](https://leetcode-cn.com/problems/perfect-squares/submissions/) 采用记录第一次出现作为贪心。

## 拓扑排序（图论）
bfs的分支思路，维护一个入度数组，当某个节点的入度为0时候。将该节点加入队列。
## 滑动窗口
类似于双指针的思路吧，只是固定了大小。

# 数据结构
## 树
主要是各类遍历，以及涉及到递归和迭代的思路。比较关键的在于分析清楚当前根节点的作用，然后对两侧的子树进行递归的时候可以得到什么。
## Trie树
这个算是一个树的变种吧。解决的问题包括 [最大异或对](https://leetcode-cn.com/problems/maximum-xor-with-an-element-from-array/) [单词压缩编码](https://leetcode-cn.com/problems/short-encoding-of-words/)

```java
class Solution {
    // 最大异或值，典型的Trie树的模板问题，每次都存储数组的每一位的存在情况。
    int[][] son; // Trie树的本体
    int index; // 全局的一个索引
    int[] cnt; // 作为Trie树的模板，还可以维护一个
    public int[] maximizeXor(int[] nums, int[][] queries) {
        int n = nums.length;
        son = new int[n*31][2]; // 两条道路，异或一样或者不一样
        Arrays.sort(nums);
        
        node[] q = new node[queries.length];
        for (int i = 0; i<queries.length;i++){
            q[i] = new node(queries[i][0], queries[i][1], i);
        }
        // 离线查询的思路
        Arrays.sort(q,(a,b)->a.bar-b.bar);
        int cur = 0;
        int[] ans = new int[queries.length];
        for(int i = 0;i<queries.length;i++){
            while(cur<n && nums[cur]<=q[i].bar){
                insert(nums[cur]);
                cur++;
            }
            if(index == 0) ans[q[i].index] = -1;
            else{
                int t = query(q[i].num);
                ans[q[i].index] = t^q[i].num;
            }
            
        }
        return ans;
    }

    private void insert(int num){
        int p = 0;
        for(int i = 30;i>=0;i--){
            int cur = (num>>i)&1; 
            if(son[p][cur] == 0){
                index++;
                son[p][cur] = index; // 存储了下一个trie节点的索引
            }
            p = son[p][cur];
        }
    }

    private int query(int num){
        int p = 0;
        int res = 0;
        for(int i = 30;i>=0;i--){
            int cur = (num>>i)&1;
            // 如果有逆反的位，就去逆反位上
            if (son[p][cur^1] != 0){ // 1^1 = 0 0^1 = 1 按位取反
                res = res*2+(cur^1);
                p = son[p][cur^1];
            }else{
                // 否则只能当前位
                res = res*2+cur;
                p = son[p][cur];
            }
        }
        return res;
    }

    class node{
        int bar; // 题目给定了一个限制，只能和不超过bar的数字异或
        int index;
        int num;
        public node(int num, int bar, int index){
            this.num = num;
            this.index = index;
            this.bar = bar;
        }
    }
}

```
```java
后缀trie树
class Solution {
    public int minimumLengthEncoding(String[] words) {
        TrieNode trie = new TrieNode();
        Map<TrieNode, Integer> nodes = new HashMap<TrieNode, Integer>();

        for (int i = 0; i < words.length; ++i) {
            String word = words[i];
            TrieNode cur = trie;
            for (int j = word.length() - 1; j >= 0; --j) {
                cur = cur.get(word.charAt(j));
            }
            // 字典中存储了头的那个位置
            nodes.put(cur, i);
        }

        int ans = 0;
        for (TrieNode node: nodes.keySet()) {
            if (node.count == 0) {
                ans += words[nodes.get(node)].length() + 1; // 表示是结尾
            }
        }
        return ans;

    }
}

class TrieNode {
    TrieNode[] children;
    // count表示当前节点被使用的次数，为0表示是根节点
    int count;

    TrieNode() {
        children = new TrieNode[26];
        count = 0;
    }

    public TrieNode get(char c) {
        if (children[c - 'a'] == null) {
            count++;
            children[c - 'a'] = new TrieNode();
            
        }
        return children[c - 'a'];
    }
}


```

经典题目 [单词搜索II]（https://leetcode-cn.com/problems/word-search-ii/）
## 并查集
一般可以使用在查找连通性的问题上，包括最小生成树，具有传递性的一些连通等。

最后上模板
```c++
class UF{
private:
    vector<int> fa;
    vector<int> sz;
    int num = 0;
public:
    UF(int n){
        num = n;
        for(int i =0;i<n;i++){
            fa.push_back(i);
            sz.push_back(1);
        }
    }

    int find(int x){
        if (fa[x] == x){
            return x;
        }
        fa[x] = find(fa[x]);
        return fa[x];
    }

    bool isUnion(int x, int y){
        return find(x) == find(y);
    }

    void unin(int x, int y){
        int xfa = find(x);
        int yfa = find(y);
        if (xfa==yfa) return;
        num--;
        if(sz[xfa] < sz[yfa]){
            fa[xfa] = yfa;
            sz[yfa] += sz[xfa];
        }else{
            fa[yfa] = xfa;
            sz[xfa] += sz[yfa];
        }
    }

    int count() const{
        return num;
    }
};

```

## 单调栈
单调栈结构一定是一个双段队列，每次维护其中一段，然后最值是另外一段。
```java
class Solution {
    public int[] maxSlidingWindow(int[] nums, int k) {
        if(nums.length == 0 || k == 0) return new int[0];
        Deque<Integer> queue = new LinkedList<>();
        int n = nums.length;
        int[] ans = new int[n-k+1];
        
        for(int i = 0;i<n;i++){
            int cur = nums[i];
            while(!queue.isEmpty() && nums[queue.peekLast()] <= nums[i]){
                queue.pollLast();//右侧不断弹出
            }
            queue.offerLast(i);
            if (i-k>=-1){
                while(!queue.isEmpty() && (i-queue.peekFirst()>=k)){
                    queue.pollFirst(); // 左侧计算最大值
                }
                ans[i-k+1] = nums[queue.peekFirst()];
            }
        }
        return ans;
    }
}
```
## 树状数组
树状数组的特点在于可以快速的求解某个区间的前缀和，并且可以修改某个数字。时间复杂度都是O(logn)。核心的函数有三个`lowbit(x)`,`query(index)`,`add(index, val)`。
需要注意的是，**树状数组的插入是从1开始的**。
![](https://img-blog.csdnimg.cn/20200707170445981.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3pjejU1NjY3MTk=,size_16,color_FFFFFF,t_70)
[通过指令创建有序数组](https://leetcode-cn.com/problems/create-sorted-array-through-instructions/)
```java
class Solution {
    public int createSortedArray(int[] instructions) {
        int N = 100010;
        BIT ta = new BIT(N);
        int ans = 0;
        int mod = 1000000007;
        for (int i = 0 ; i<instructions.length;i++){
            int l = ta.query(instructions[i]-1);
            int r = ta.query(N-1)-ta.query(instructions[i]);
            ta.add(instructions[i],1); // 在index这个位置加1权重
            ans = (int)((1L*ans + 1L*Math.min(r,l))%mod);
        }
        return ans;
    }
}

// 以下是树状区间的代码，Java实现
// 需要注意是从1开始的。

class BIT{
    int[] tree;
    // 传入树的大小
    public BIT(int N){
        tree = new int[N];
    }
	// 内部函数，帮助计算需要修改的位置，得到二进制的最后一个1
    // 利用了负数的补码，反转后最后一位+1
    private int lowbit(int x){
        return (x & -x);
    }
    // 对x位置的数字加c
    public void add(int x, int c){
        int n = tree.length;
        while (x < n){
            tree[x] += c;
            x += lowbit(x);
        }
    }
    // 查询[1,x]的区间和
    public int query(int x){
        int res = 0;
        while(x>0){
            res += tree[x];
            x -= lowbit(x) ;
        }
        return res;
    }
    public
}

```

## 线段树
线段树可以说是对树状数组的升级了。可以很明显的看出来线段树和树状数组的区别，线段是可以对区间整体抬升复杂度也是O(logn)，树状数组只能是对单点提升。
```java
// 通过线段树保存区间的最大值，最小值，区间和。
public static class Result {
    public int minv;
    public int maxv;
    public int sumv;
}
    //query(L, R) 查询区间 [L, R] 的最大值、最小值、区间和；
    // 用线段树实现动态区间最小值、最大值、区间和查询，对区间A[0..N-1]支持更新操作和查询操作：
    // update(int L, int R, int v) 将区间[L,R]所有元素值设置为v
    // query(int L, int R ) 查询数组区间[L,R]的最小值、最大值、区间和
public static class RangeTree {
    //N为源数组总长度，可查询区间为[0,N)
    int N = 0;
    //M为最底层叶子节点数目，M = min { x | x = 2^k && x >= N }
    int M = 0;
    //线段树的数组表示
    int[] setv = null;	//各结点的设置标记
    int[] sumv = null;	//各结点的区间和
    int[] minv = null;	//各结点的最小值
    int[] maxv = null;	//各结点的最大值

    public RangeTree(int[] A) {
        build(A);
    }

    //构造线段树，从叶子节点递推，复杂度O(N)
    public void build(int[] A) {
        N = A.length; // [1,2,3,4]
        M = calculate(N); //4
        setv = new int[2*M-1]; // 7
        sumv = new int[2*M-1]; 
        minv = new int[2*M-1];
        maxv = new int[2*M-1];

        //初始化所有叶子结点
        for ( int i=0; i<N; i++ ) {
            setv[i+M-1] = Integer.MAX_VALUE;
            sumv[i+M-1] = minv[i+M-1] = maxv[i+M-1] = A[i];
        }
        // 多余的废弃的 这里的max和min都初始为0 是有点小问题的。
        for ( int i=N; i<M; i++ ) {
            sumv[i+M-1] = 0;
            minv[i+M-1] = Integer.MAX_VALUE;
            maxv[i+M-1] = Integer.MIN_VALUE;
            //sumv[i+M-1] = minv[i+M-1] = maxv[i+M-1]= 0;
            setv[i+M-1] = Integer.MAX_VALUE;
        }

        //初始化非叶子结点
        for ( int i=M-2; i>=0; i-- ) {
            setv[i] = Integer.MAX_VALUE;
            minv[i] = Math.min(minv[2*i+1], minv[2*i+2]);
            maxv[i] = Math.max(maxv[2*i+1], maxv[2*i+2]);
            sumv[i] = sumv[2*i+1] + sumv[2*i+2];
        }

    }

    // 区间最小值查询  // L : 待查询区间左边界  // R : 待查询区间右边界
    public Result query(int L, int R) {
        Result res = new Result();
        res.minv = Integer.MAX_VALUE;// 这里解决了我的疑问
        res.maxV = Integer.MIN_VALUE; 
        query(L, R, 0, 0, M-1, res);
        return res;
    }

    //区间最小值查询 c : 当前结点在线段树中的编号 //CUR_L : 当前结点区间的左边界 //CUR_R : 当前结点区间的右边界
    private void query(int L, int R, int c, int CL, int CR, Result res) {

        if ( setv[c] != Integer.MAX_VALUE ) {// 懒标记存在，需要更新
            res.sumv += setv[c] * (Math.min(R,CR)-Math.max(L,CL)+1);
            res.maxv = Math.max(res.maxv, maxv[c]);
            res.minv = Math.min(res.minv, minv[c]);
        } else if ( L <= CL && CR <= R ) {
            //待查询区间完全覆盖当前结点区间
            res.maxv = Math.max(res.maxv, maxv[c]);
            res.minv = Math.min(res.minv, minv[c]);
            res.sumv += sumv[c];
        } else {
            int CM = ( CL + CR ) / 2;
            //查询区间与左半区间有重叠
            if ( L <= CM ) query(L, R, 2*c+1, CL, CM, res);
            //查询区间与右半区间重叠
            if ( R > CM ) query(L, R, 2*c+2, CM+1, CR, res);
        }

    }

    //更新一个区间，将 [L,R] 区间所有值更新为 v
    public void update(int L, int R, int v) {
        update(L,R,v,0,0,M-1);
    }

    //更新区间最小值//c : 当前结点在线段树中的编号//CL : 当前结点区间的左边界 //CR : 当前结点区间的右边界
    private void update(int L, int R, int v, int c, int CL, int CR) {
        if ( L == R ) {
            setv[c] = sumv[c] = maxv[c] = minv[c] = v;
        } else {
            if ( L <= CL && R >= CR ) {
                setv[c] = v;
            } else {
                // 首先下移标记
                pushdown(c);
                int CM = (CR+CL)/2, lc = 2*c+1, rc = 2*c+2;
                if ( L <= CM ) {
                    update(L, R, v, lc, CL, CM);
                } else {
                    maintain(lc, CL, CM);
                }
                if ( R > CM ) {
                    update(L, R, v, rc, CM+1, CR);
                } else {
                    maintain(rc, CM+1, CR);
                }
            }
            maintain(c,CL,CR);
        }
    }

    // 将当前结点的设置标记下移
    private void pushdown(int c) {
        if ( setv[c] != Integer.MAX_VALUE ) {
            setv[2*c+1] = setv[c];
            setv[2*c+2] = setv[c];
            setv[c] = Integer.MAX_VALUE;
        }
    }

    // 计算当前结点区间的最小最大值及区间和
    private void maintain(int c, int CL, int CR) {
        int lc = 2*c+1, rc = 2*c+2;
        if ( setv[c] == Integer.MAX_VALUE ) {
            sumv[c] = sumv[lc] + sumv[rc];
            maxv[c] = Math.max(maxv[lc],maxv[rc]);
            minv[c] = Math.min(minv[lc],minv[rc]);

        } else {// 懒标记存在，发生了区间更新
            sumv[c] = setv[c]*(CR-CL+1);
            maxv[c] = minv[c] = setv[c];
        }
    }

    // 计算最底层的叶子结点数目 大于个数的第一个二次方
    private int calculate(int n) {
        int y = 1;
        while ( y < n ) {
            y <<= 1;
        }
        return y;
    }

    // 测试
    public static void main(String[] args) {
        int[] v = { 7, 8, 9, 5, 6, 4, 3, 2 };
        RangeTree inst = new RangeTree(v);
        for ( int i=0; i<v.length; i++ ) {
            Result res = inst.query(0, i);
            System.out.printf("%d,%d,%d  ", res.minv, res.maxv, res.sumv);
        }
        System.out.println();
        for ( int i=0; i<v.length; i++ ) {
            Result res = inst.query(i, v.length-1);
            System.out.printf("%d,%d,%d  ", res.minv, res.maxv, res.sumv);
        }
        System.out.println();
        inst.update(0, 3, 1);
        inst.update(2, 7, 2);
        for ( int i=0; i<v.length; i++ ) {
            Result res = inst.query(i, v.length-1);
            System.out.printf("%d,%d,%d  ", res.minv, res.maxv, res.sumv);
        }

    }

}
```

# 输入输出
对于面试问题需要有一个规范的输入输出模板。以下格式需要熟记。
```java
import java.util.*;
public class Main{
    public static void main(String[] args){
        Scanner sc = new Scanner(System.in);
    }
}

```


需要注意输入的格式，有没有给定每行的个数。如果没有的话就需要当成字符串处理。
```java
import java.util.*;
public class Main{
    public static void main(String[] args){
        Scanner sc = new Scanner(System.in);
        while(sc.hasNext()){
            String[] a = sc.nextLine().split(" ");
            int len = a.length;
            int ans = 0;
            for(int i = 0; i<len;i++){
                ans += Integer.valueOf(a[i]);
            }
            System.out.println(ans);
        }
    }
}
```
如果需要输出特定的小数点，可以是使用`System.out.printf("%.6f\n", ans1);`
## 更快的输入输出
```java
public static void main(String[] args) throws IOException {
    Reader.init( System.in );
}

class Reader {
    static BufferedReader reader;
    static StringTokenizer tokenizer;

    /** 调用这个方法来初始化reader，即InputStream*/
    static void init(InputStream input) {
        reader = new BufferedReader(
                new InputStreamReader(input) );
        tokenizer = new StringTokenizer("");
    }

    /** 获取下一段文本 */
    static String next() throws IOException {
        while ( ! tokenizer.hasMoreTokens() ) {
            //TODO add check for eof if necessary
            tokenizer = new StringTokenizer(
                    reader.readLine() );
        }
        return tokenizer.nextToken();
    }

    static int nextInt() throws IOException {
        return Integer.parseInt( next() );
    }

    static double nextDouble() throws IOException {
        return Double.parseDouble( next() );
    }
}

```
# 典型问题
## 背包问题
[参考文献](https://blog.csdn.net/zcz5566719/article/details/106932292)
可以细分为01背包，完全背包，多重背包。01背包的优化包括从后往前遍历剩余容量，完全背包是从前往后遍历剩余容量，多重背包是可以采用二进制转换为01背包。

另外会有类似的，分组背包->其实就是多重背包，二维背包->其实就是额外多了一个可以倒序遍历的维度。

在进一步是求解背包的方案数问题，最简单的方案数问题，与权值无关。**求醉最优方案的方案数**我们需要额外的维护dp数组和方案数数组，我们需要知道当前的最优方案是如何转换得到的，并且进行加和得到最优解的方案数。

有依赖的树型背包问题，需要使用dfs，首先完善出来子树的dp情况，在计算根节点的最优。自下而上的01背背包。

**核心：状态压缩——外循环是正向，内循环是倒序m~v[i]**

状态转移方程：
1、最值问题: dp[i] = max/min(dp[i], dp[i-nums]+1) 或 dp[i] = max/min(dp[i], dp[i-num]+nums);
2、存在问题(bool)：dp[i]=dp[i]||dp[i-num];
3、组合问题：dp[i]+=dp[i-num];

### 01背包
`dp[i][j] = max(dp[i-1][j], dp[i-1][j-w[i]]+v[i])`
```c++
#include<bits/stdc++.h>
using namespace std;

const int N = 1010;

int n, m;
int value[N], weight[N];
int dp[N];

int main() {
    cin >> n >> m;
    for(int i = 0; i < n; i++) cin >> value[i] >> weight[i];
    for(int i = 0; i < n; i++) 
        for(int j = m; j >= weight[i]; j--) 
            dp[j] = max(dp[j], dp[j-weight[i]]+value[i]);
    cout << dp[m] << endl;
 return 0;    
}
```
### 完全背包

**核心：状态压缩——外循环是正向，内循循环正序序v[i]~m**  `dp[i][j] = max(dp[i][j], dp[i][j-w[i]]+v[i])`
```c++
#include<bits/stdc++.h>
using namespace std;

const int N = 1010;

int n, m;
int value[N], weight[N];
int dp[N];

int main() {
    cin >> n >> m;
    for(int i = 0; i < n; i++) cin >> weight[i]  >> value[i] ;
    for(int i = 0; i < n; i++) 
        for(int j = weight[i]; j <= m; j++) 
            dp[j] = max(dp[j], dp[j-weight[i]]+value[i]);
    cout << dp[m] << endl;
 return 0;    
}
```

### 完全背包求 最值 方案
求出符合某个方案的最X的条件。关键因素在于如何实现转移方程。
```c++
class Solution {
public:
    int coinChange(vector<int>& coins, int amount) {
        int n = coins.size();
        // 注意这里需要使用long long，否则爆INT
        vector<long long> dp(amount+1, INT_MAX);
        dp[0] = 0;
        for(auto &c : coins){
            // 因为是完全背包，所有内循环正向枚举，
            for(int i = 0;i<=amount;i++){
                if(i>=c){
                    dp[i] = min(dp[i], dp[i-c]+1);
                }
            }
        }
        return dp[amount] == INT_MAX ? -1:dp[amount];
    }
};
```

### 01背包求方案数
注意在定义转移矩阵的时候，**恰好**和**小于等于**的区别。
```c++
#include<bits/stdc++.h>
using namespace std;

const int N = 1010;

int n, m;
int value[N], weight[N];
int dp[N];
int mod = 1e9+7;
long long path[N];
// 方法1：容积恰好为 i 时的最佳解的方案个数
int main() {
    cin >> n >> m;
    for(int i = 0; i < n; i++) cin >> weight[i]  >> value[i];
    path[0] = 1; // 容积恰好为 0 的最佳解的方案个数
    for(int i = 0; i < n; i++) {
        for(int j = m; j >= weight[i]; j--) {
            long long cur = dp[j-weight[i]]+value[i];
            // 新的方案更好
            if(cur > dp[j]){
                dp[j] = cur%mod;
                path[j] = path[j-weight[i]];
            }else if(dp[j] == cur){ // 新的方案和之前的一样好,两种方案
                path[j] = (path[j-weight[i]] + path[j])%mod;
            }  
        }
    }
    int max_ans = dp[m];
    // 此时的path[i]: 在n个物品下，容积恰好i时，最佳方案的方案数。
    int ans = 0;
    for(int i = 0;i<=m;i++){
        // 因为可能容积 x~m 内的，得到的最佳结果时一样的，这些方案都需要累加。
        if(dp[i] == max_ans) ans = (ans+path[i])%mod;
    }
    cout << ans << endl;
 return 0;    
}


// 方案二： path[i]定义为容量最大为i时的最优解的方案数
int main() {
    cin >> n >> m;
    for(int i = 0; i < n; i++) cin >> weight[i]  >> value[i];
    // !!! 注意！！！ 这里是两种方法唯一的区别。对于恰好，只有dp[0]=1，对于最大，dp[i]=1；
    for(int i = 0; i <= m; i ++)  path[i] = 1;
    for(int i = 0; i < n; i++) {
        for(int j = m; j >= weight[i]; j--) {
            long long cur = dp[j-weight[i]]+value[i];
            if(cur > dp[j]){
                dp[j] = cur%mod;
                path[j] = path[j-weight[i]];
            }else if(dp[j] == cur){
                path[j] = (path[j-weight[i]] + path[j])%mod;
            }  
        }
    }
    cout << path[m] << endl;
 return 0;    
}
```
### 多重背包问题
第 i 种物品最多有 si 件，每件体积是 vi，价值是 wi。
方法其实就是把s件那个物品，重复多次；对于无限次使用的，最多也只有V/vi。退化成01背包。

枚举的时候可以采用二进制的思路。 复杂度 $O(nlog(s))$

更进一步可以用单调队列优化复杂度更低 https://www.acwing.com/solution/content/53507/
```c++
int main()
{
    cin >> n >> m;
    int cnt = 0; 
    for(int i = 1;i <= n;i ++){
        int a,b,s;
        cin >> a >> b >> s; // 体积，价值，个数
        int k = 1; 
        if(s<0)s=1; // 表示 只能用1此
        else if(s==0)s=m/a; // 随便用，最多也 m/a

        while(k<=s){
            cnt ++ ; //组别先增加
            w[cnt] = a * k ; 
            v[cnt] = b * k; 
            s -= k; k *= 2;
        }
        //剩余的一组
        if(s>0){
            cnt ++ ;
            w[cnt] = a*s; 
            v[cnt] = b*s;
        }
    }
    //01背包
    for(int i = 1;i <= cnt ;i ++)
        for(int j = m ;j >= w[i];j --)
            f[j] = max(f[j],f[j-w[i]] + v[i]);
    cout << f[m] << endl;
    return 0;
}
```
### 容积、重量双限制
```c++
int main () {
    cin >> n >> V >> M;
    for (int i = 1; i <= n; i ++) {
        cin >> v[i] >> m[i] >> w[i];//体积，重量，价值
    }
    for (int i = 1; i <= n; i ++)
        for (int j = V; j >= v[i]; j --)
            for (int k = M; k >= m[i]; k --)
                f[j][k] = max (f[j - v[i]][k - m[i]] + w[i], f[j][k]); //01 背包
    cout << f[V][M] << endl;
    return 0;
} 

```


## 重写排序
```java
Arrays.sort(arr, new Comparator<int[]>(){
    public int compare(int[] a, int[] b){
        return a[0]-b[0];
    }
});


List<int[]> list = new ArrayList<>(map.values());
Collections.sort(list, (a, b)->{
    if (a[0] != b[0]) return a[0] - b[0];
    if (a[1] != b[1]) return a[1] - b[1];
    return a[2] - b[2];
});

```
## 最大公约数
```java
public int gcd(int x, int y) {
    return x == 0 ? y : gcd(y % x, x);
}
```
```c++
int x = gcd(a, b);
```
# C++ 头文件
https://www.acwing.com/blog/content/17174/
```c++
#include<bits/stdc++.h>
using namespace std;
```