# 算法基础-最小生成树


由图生成权重最小的树

<!--more-->

给你一个points 数组，表示 2D 平面上的一些点，其中 points[i] = [xi, yi] 。

连接点 [xi, yi] 和点 [xj, yj] 的费用为它们之间的 曼哈顿距离 ：|xi - xj| + |yi - yj| ，其中 |val| 表示 val 的绝对值。

请你返回将所有点连接的最小总费用。只有任意两点之间 有且仅有 一条简单路径时，才认为所有点都已连接。

>来源：力扣（LeetCode）
>
>链接：https://leetcode-cn.com/problems/min-cost-to-connect-all-points
>
>著作权归领扣网络所有。商业转载请联系官方授权，非商业转载请注明出处。


## Kruskal

所有边按权重排序，使用并查集选出不成环的边

```go
func NewUF(n int) *UnionFind {
	parent := make([]int, n)
	for i := range parent {
		parent[i] = i
	}
	return &UnionFind{
		parent: parent,
		count:  n,
	}
}

type UnionFind struct {
	parent []int
	count  int
}

func (uf *UnionFind) Union(a, b int) {
	rootA := uf.Find(a)
	rootB := uf.Find(b)
	if rootA == rootB {
		return
	}
	uf.parent[rootA] = rootB
	uf.count--
}

func (uf *UnionFind) Find(a int) int {
	for uf.parent[a] != a {
		uf.parent[a] = uf.parent[uf.parent[a]]
		a = uf.parent[a]
	}
	return a
}

func (uf *UnionFind) Connected(a, b int) bool {
	rootA := uf.Find(a)
	rootB := uf.Find(b)
	return rootA == rootB
}

func minCostConnectPoints(points [][]int) int {
	n := len(points)
    // graph [from, to, weight]
	graph := make([][]int, 0)
	for i := 0; i < n; i++ {
		for j := i + 1; j < n; j++ {
			edge := []int{i, j, dist(points, i, j)}
			graph = append(graph, edge)
		}
	}
	sort.Slice(graph, func(i, j int) bool {
		return graph[i][2] < graph[j][2]
	})

	uf := NewUF(n)
	res := 0
	for _, edge := range graph {
		a, b, w := edge[0], edge[1], edge[2]
		if uf.Connected(a, b) {
			continue
		}
		uf.Union(a, b)
		res += w
	}
	return res
}

func abs(a int) int {
	if a > 0 {
		return a
	}
	return -a
}

func dist(points [][]int, a, b int) int {
	return abs(points[a][0]-points[b][0]) + abs(points[a][1]-points[b][1])
}
```


## Prim

图中每次切分，至少包含一条最小生成树的边（即权重最小的边必为最小生成树路径）

```go
func abs(a int) int {
	if a > 0 {
		return a
	}
	return -a
}

func dist(points [][]int, a, b int) int {
	return abs(points[a][0]-points[b][0]) + abs(points[a][1]-points[b][1])
}

type PriorityQueue struct {
	data [][]int
}

func (p *PriorityQueue) Len() int {
	return len(p.data)
}

func (p *PriorityQueue) Less(i, j int) bool {
	return p.data[i][1] < p.data[j][1]
}

func (p *PriorityQueue) Swap(i, j int) {
	p.data[i], p.data[j] = p.data[j], p.data[i]
}

func (p *PriorityQueue) Push(x interface{}) {
	p.data = append(p.data, x.([]int))
}

func (p *PriorityQueue) Pop() interface{} {
	v := p.data[len(p.data)-1]
	p.data = p.data[:len(p.data)-1]
	return v
}

func minCostConnectPoints(points [][]int) int {
	n := len(points)
    // graph[from] [to, weight]
	graph := make([][][]int, n)
	for i := 0; i < n; i++ {
        // 邻接表表示的无向图，需要双向赋值
		for j := 0; j < n; j++ {
			if i == j {
				continue
			}
			graph[i] = append(graph[i], []int{j, dist(points, i, j)})
		}
	}

	used := make([]bool, n)
	pq := &PriorityQueue{[][]int{}}
	cut := func(a int) {
		for _, edge := range graph[a] {
			if used[edge[0]] {
				continue
			}
			heap.Push(pq, edge)
		}
	}
	used[0] = true
	cut(0)

	w := 0
	for pq.Len() > 0 {
		edge := heap.Pop(pq).([]int)
		if used[edge[0]] {
			continue
		}
		w += edge[1]
		used[edge[0]] = true
		cut(edge[0])
	}

	return w
}
```

---

> 作者: ingbyr  
> URL: https://ingbyr.github.io/posts/%E7%AE%97%E6%B3%95%E5%9F%BA%E7%A1%80-%E6%9C%80%E5%B0%8F%E7%94%9F%E6%88%90%E6%A0%91/  

