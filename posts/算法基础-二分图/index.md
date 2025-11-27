# 算法基础-二分图


用于将节点分为两组

<!--more-->

## 判断是否为二分图

### BFS 实现
```go
func isBipartite(graph [][]int) bool {
	n := len(graph)
	visited := make([]bool, n)
	color := make([]bool, n)
	valid := true

	var bfs func(int)
	bfs = func(root int) {
		visited[root] = true
		queue := []int{root}
		for len(queue) > 0 {
			node := queue[0]
			queue = queue[1:]
			for _, next := range graph[node] {
				if !visited[next] {
					color[next] = !color[node]
					visited[next] = true
					queue = append(queue, next)
				} else {
					if color[next] == color[node] {
						valid = false
						return
					}
				}
			}
		}
	}

	for i := 0; i < n; i++ {
		if !visited[i] && valid {
			bfs(i)
		}
	}

	return valid
}
```


### DFS 实现
```go
func possibleBipartition(n int, dislikes [][]int) bool {
	graph := make([][]int, n)
	for _, dislike := range dislikes {
		a, b := dislike[0]-1, dislike[1]-1
		graph[a] = append(graph[a], b)
		graph[b] = append(graph[b], a)
	}

	valid := true
	visited := make([]bool, n)
	color := make([]bool, n)
	var dfs func(node int)
	dfs = func(node int) {
		if !valid {
			return
		}
		visited[node] = true
		for _, next := range graph[node] {
			if !visited[next] {
				color[next] = !color[node]
				dfs(next)
			} else {
				if color[next] == color[node] {
					valid = false
					return
				}
			}
		}
	}

	for i := 0; i < n; i++ {
		if !visited[i] && valid {
			dfs(i)
		}
	}
	return valid
}

```

---

> 作者: ingbyr  
> URL: https://ingbyr.github.io/posts/%E7%AE%97%E6%B3%95%E5%9F%BA%E7%A1%80-%E4%BA%8C%E5%88%86%E5%9B%BE/  

