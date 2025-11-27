# 算法基础-堆


堆，亦称为优先队列

<!--more-->

## 最小堆

设堆大小为 size，对于下标为k的节点：
- 父节点下标为 `(k - 1) / 2`
- 左子节点下标为 `2 * k + 1`，右子节点下标为 `2 * k + 2`
- 最后一个非叶子节点的下标为 `(size - 2) / 2` (即最后一个节点 `size - 1`的父节点 `((size - 1) - 1) / 2`)

go语言参考实现：
```go
type MinHeap struct {
	heap []int
}

func NewMinHeap() *MinHeap {
	return &MinHeap{heap: []int{}}
}

func (h *MinHeap) up(k int) {
	if k == 0 {
		return
	}
	for pk := (k - 1) / 2; k >= 0 && h.heap[pk] > h.heap[k]; {
		h.heap[pk], h.heap[k] = h.heap[k], h.heap[pk]
		k = pk
		pk = (k - 1) / 2
	}
}

func (h *MinHeap) down(k int) {
	if len(h.heap) == 1 {
		return
	}
	last := (len(h.heap) - 2) / 2
	for k <= last {
		child := 2*k + 1 // left child
		childV := h.heap[child]
		right := child + 1
		if right < len(h.heap) && h.heap[right] < childV {
			child = right
			childV = h.heap[right]
		}
		if h.heap[k] > childV {
			h.heap[k], h.heap[child] = h.heap[child], h.heap[k]
			k = child
		} else {
			break
		}
	}
}

func (h *MinHeap) add(v int) {
	h.heap = append(h.heap, v)
	h.up(len(h.heap) - 1)
}

func (h *MinHeap) pop() (int, bool) {
	if len(h.heap) == 0 {
		return 0, false
	}
	res := h.heap[0]
	h.heap[0] = h.heap[len(h.heap)-1]
	h.heap = h.heap[:len(h.heap)-1]
	if len(h.heap) > 0 {
		h.down(0)
	}
	return res, true
}

func (h *MinHeap) heapify(arr []int) {
	h.heap = arr
	for i := (len(h.heap) - 2) / 2; i >= 0; i-- {
		h.down(i)
	}
}

```

---

> 作者: ingbyr  
> URL: https://ingbyr.github.io/posts/%E7%AE%97%E6%B3%95%E5%9F%BA%E7%A1%80-%E5%A0%86/  

