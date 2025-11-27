# 算法基础-排序


经典排序算法

<!--more-->


## 冒泡排序

```go
func sortBubble1(arr []int) []int {
	n := len(arr)
	// 有序区间 [0, i]
	for i := 0; i < n; i++ {
		// 从最右侧两两交换，将无序区间中的最小值放置到i
		for j := n - 1; j > i; j-- {
			if arr[j] < arr[j-1] {
				swap(arr, j, j-1)
			}
		}
	}
    return arr
}

func sortBubble2(arr []int) {
	n := len(arr)
	exchanged := true
	// 有序区间 [0, i]
	for i := 0; i < n && exchanged; i++ {
        // 当 exchanged 为 true 时，表明剩余无序区间已有序，结束排序
		exchanged = false
		// 从最右侧两两交换，将无序区间中的最小值放置到i
		for j := n - 1; j > i; j-- {
			if arr[j] < arr[j-1] {
				exchanged = true
				swap(arr, j, j-1)
			}
		}
	}
}
```


## 快速排序

```go
func sortQuick(arr []int) []int {
	return _sortQuick(arr, 0, len(arr) - 1)
}

func _sortQuick(arr []int, left, right int) []int {
	if left < right {
		p := partition(arr, left, right)
		_sortQuick(arr, left, p-1)
		_sortQuick(arr, p+1, right)
	}
	return arr
}

func partition(arr []int, l int, r int) int {
	// l位空出
	v := arr[l]
	for l < r {
		for l < r && arr[r] >= v {
			r--
		}
		// r位保存至l位，r位空出
		arr[l] = arr[r]
		for l < r && arr[l] <= v {
			l++
		}
		// l位保存至r位，l位空出
		arr[r] = arr[l]
	}
	// 空出的l位填入基准值v
	arr[l] = v
	return l
}
```

```java
   class Solution {

    public int[] sortArray(int[] nums) {
        if (nums == null || nums.length <= 1) {
            return nums;
        }
        quickSort(nums, 0, nums.length - 1);
        return nums;
    }

    private void quickSort(int[] nums, int l, int r) {
        if (l >= r) {
            return;
        }
        int p = partition(nums, l, r);
        quickSort(nums, l, p - 1);
        quickSort(nums, p + 1, r);
    }

    private int partition(int[] nums, int l, int r) {
        int v = randomPiv(nums, l, r);
        while (l < r) {
            while (l < r && nums[r] >= v) {
                r--;
            }
            nums[l] = nums[r];
            while (l < r && nums[l] <= v) {
                l++;
            }
            nums[r] = nums[l];
        }
        nums[l] = v;
        return l;
    }

    private int randomPiv(int[] nums, int l, int r) {
        int p = ThreadLocalRandom.current().nextInt(r - l + 1) + l;
        int temp = nums[l];
        nums[l] = nums[p];
        nums[p] = temp;
        return nums[l];
    }

}  
```

## 插入排序

```go
func sortInsert(arr []int) []int{
	// 有序区间 [0, i]，无序区间[i+1, +00]
	for i := 1; i < len(arr); i++ {
		for j := i; j > 0 && arr[j] < arr[j-1]; j-- {
			// 小数前移
			swap(arr, j, j - 1)
		}
	}
	return arr
}
```


## 希尔排序（插入排序改进版）

> 希尔为该算法发明人

```go
func sortShell(arr []int) []int {
	for step := len(arr) / 2; step > 0; step /= 2 {
		for i := step; i < len(arr); i++ {
			for j := i; j >= step && arr[j] < arr[j-step]; j -= step {
				// 同组内小数前移
				swap(arr, j, j-step)
			}
		}
	}
	return arr
}
```


## 选择排序

```go
func sortSelect(arr []int) {
	for i := 0; i < len(arr); i++ {
		m := i
        // 无序区间内选择最小值下标 m
		for j := i; j < len(arr); j++ {
			if arr[j] < arr[m] {
				m = j
			}
		}
		if m != i {
			swap(arr, i, m)
		}
	}
}
```


## 堆排序

```go
func sortHeap(arr []int) []int {
	h := MinHeap{}
	h.build(arr)
	res := make([]int, 0, len(arr))
	for len(h.heap) > 0 {
		res = append(res, h.pop())
	}
	return res
}

type MinHeap struct {
	heap []int
}

// 上浮操作：节点与大于节点值的父节点交换
func (h *MinHeap) up(k int) {
	if k == 0 {
		return
	}
    // 节点 k 比 父节点 pk 小时，交换两节点，实现节点上浮
	pk := (k - 1) / 2
	for k >= 0 && h.heap[pk] > h.heap[k] {
		h.heap[pk], h.heap[k] = h.heap[k], h.heap[pk]
		k = pk
		pk = (k - 1) / 2
	}
}


// 下沉操作：节点与子节点中较小值交换
func (h *MinHeap) down(k int) {
	if len(h.heap) <= 1 {
		return
	}
	lastNonLeaf := (len(h.heap) - 1 - 1) / 2
	for k <= lastNonLeaf {
		// 默认左侧节点为子节点中值最小节点
		child := 2*k + 1
		childV := h.heap[child]
        // 检查右侧节点，若比默认节点小，替换之
		right := child + 1
		if right < len(h.heap) && h.heap[right] < childV {
			child = right
			childV = h.heap[right]
		}
        // 若节点比子节点大，下沉
		if h.heap[k] > childV {
			h.heap[k], h.heap[child] = h.heap[child], h.heap[k]
			k = child
		} else {
			break
		}
	}
}

// 添加操作：加至末尾节点，后上浮尾节点
func (h *MinHeap) add(v int) {
	h.heap = append(h.heap, v)
	h.up(len(h.heap) - 1)
}

// 弹出操作：头节点弹出，尾节点放置头节点，后下沉头节点
func (h *MinHeap) pop() int {
	if len(h.heap) == 0 {
		panic("empty heap")
	}
	res := h.heap[0]
	// swap last and first
	h.heap[0] = h.heap[len(h.heap)-1]
	// remove last
	h.heap = h.heap[:len(h.heap)-1]
	if len(h.heap) > 0 {
		// re-heap
		h.down(0)
	}
	return res
}

// 构建树：从最后一个非叶子节点开始，执行下沉操作
func (h *MinHeap) build(arr []int) {
	h.heap = arr
	for i := (len(h.heap) - 2) / 2; i >= 0; i-- {
		h.down(i)
	}
}
```


## 归并排序

```go
func sortMerge(arr []int) []int {
	if len(arr) <= 1 {
		return arr
	}
	mid := len(arr) / 2
	return merge(sortMerge(arr[:mid]), sortMerge(arr[mid:]))
}

func merge(left, right []int) []int {
	res := make([]int, 0, len(left)+len(right))
    // 每次处理一个元素
	i, j := 0, 0
	for i < len(left) && j < len(right) {
		if left[i] < right[j] {
			res = append(res, left[i])
			i++
		} else {
			res = append(res, right[j])
			j++
		}
	}
	if i < len(left) {
		res = append(res, left[i:]...)
	}
	if j < len(right) {
		res = append(res, right[j:]...)
	}
	return res
}
```

```java
class Solution {

    public int[] sortArray(int[] nums) {
        if (nums == null || nums.length <= 1) {
            return nums;
        }
        sort(nums, 0, nums.length);
        return nums;
    }

    // [left, right)
    private void sort(int[] nums, int left, int right) {
        if (right - left <= 1) {
            return;
        }
        int mid = left + (right - left) / 2;
        sort(nums, left, mid);
        sort(nums, mid, right);
        merge(nums, left, mid, right);
    }

    private void merge(int[] nums, int left, int mid, int right) {
        int[] arr = new int[right - left];
        int p = 0;
        int i = left, j = mid;
        while (i < mid && j < right) {
            if (nums[i] < nums[j]) {
                arr[p] = nums[i++];
            } else {
                arr[p] = nums[j++];
            }
            p++;
        }
        while (i < mid) {
            arr[p++] = nums[i++];
        }
        while (j < right) {
            arr[p++] = nums[j++];
        }
        System.arraycopy(arr, 0, nums, left, right-left);
    }

}
```

## 计数排序

```go
func sortCount(arr []int) []int {
	// 建立排序桶 [min, max]
	minV, maxV := arr[0], arr[0]
	for _, v := range arr {
		if v < minV {
			minV = v
		}
		if v > maxV {
			maxV = v
		}
	}
	buckets := make([]int, maxV-minV+1)
	for _, v := range arr {
        // 需要 min 偏移
		buckets[v-minV]++
	}

	// 桶中提取数据
	res := make([]int, 0, len(arr))
	for i, count := range buckets {
		for ; count > 0; count-- {
            // 加回偏移量 minV
			res = append(res, i+minV)
		}
	}
	return res
}
```


## 基数排序（桶排序的一种）

- 基数排序：根据键值的每位数字来分配桶
- 计数排序：每个桶只存储单一键值
- 桶排序：每个桶存储一定范围的数值，桶内使用其他排序算法

```go
func sortRadix(arr []int) []int {
    // 默认 10 进制，故桶为 [0, 9] 
    // 最大值位数默认
	maxV := arr[0]
	for _, v := range arr {
		if v > maxV {
			maxV = v
		}
	}
	maxLen := len(strconv.Itoa(maxV))

    // 按低位至高位排序
	step := 1
	for i := 0; i < maxLen; i++ {
		var buckets [10][]int
		// 插入排序桶
		for _, v := range arr {
			bitV := v / step % 10
			buckets[bitV] = append(buckets[bitV], v)
		}
		// 更新数组
		idx := 0
		for _, bucket := range buckets {
			for _, v := range bucket {
				arr[idx] = v
				idx++
			}
		}
		step *= 10
	}
	return arr
}
```

---

> 作者: ingbyr  
> URL: https://ingbyr.github.io/posts/%E7%AE%97%E6%B3%95%E5%9F%BA%E7%A1%80-%E6%8E%92%E5%BA%8F/  

