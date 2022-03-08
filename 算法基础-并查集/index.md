# 算法基础-并查集


用于判断连通性、等效等问题

<!--more-->

## 实现

```go
func NewUF(n int) *UnionFind {
	parent := make([]int, n)
	for i := range parent {
		parent[i] = i
	}
	return &UnionFind{parent: parent, count: n}
}

type UnionFind struct {
	parent []int
    count int
}

func (uf *UnionFind) Connect(a, b int) {
	rootA := uf.FindRoot(a)
	rootB := uf.FindRoot(b)
	if rootA == rootB {
		return
	}
	uf.parent[rootA] = rootB
    uf.count--
}

func (uf *UnionFind) Connected(a, b int) bool {
	rootA := uf.FindRoot(a)
	rootB := uf.FindRoot(b)
	return rootA == rootB
}

func (uf *UnionFind) FindRoot(a int) int {
	for uf.parent[a] != a {
        // 压缩路径
		uf.parent[a] = uf.parent[uf.parent[a]]
		a = uf.parent[a]
	}
	return a
}
```
