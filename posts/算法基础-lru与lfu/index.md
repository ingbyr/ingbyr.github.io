# 算法设计-LRU与LFU


go实现LRU和LFU

<!--more-->

## LRU

```go
type LRUCache struct {
	size, capacity int
	head, tail     *KVNode
	cache          map[int]*KVNode
}

type KVNode struct {
	prev, next *KVNode
	key, value int
}

func NewKVNode(key, value int) *KVNode {
	return &KVNode{
		key:   key,
		value: value,
	}
}

func Constructor(capacity int) LRUCache {
	lru := LRUCache{
		size:     0,
		capacity: capacity,
		head:     NewKVNode(0, 0),
		tail:     NewKVNode(0, 0),
		cache:    map[int]*KVNode{},
	}
	lru.head.next = lru.tail
	lru.tail.prev = lru.head
	return lru
}

func (lru *LRUCache) Get(key int) int {
	if node, ok := lru.cache[key]; ok {
		lru.MoveToHead(node)
		return node.value
	}
	return -1
}

func (lru *LRUCache) Put(key int, value int) {
	if node, ok := lru.cache[key]; ok {
		node.value = value
		lru.MoveToHead(node)
		return
	}
	newNode := &KVNode{key: key, value: value}
	lru.InsertToHead(newNode)
	lru.size++
	lru.cache[key] = newNode
	if lru.size > lru.capacity {
		lastNode := lru.RemoveTail()
		lru.size--
		delete(lru.cache, lastNode.key)
	}
}

func (lru *LRUCache) MoveToHead(node *KVNode) {
	lru.RemoveNode(node)
	lru.InsertToHead(node)
}

func (lru *LRUCache) InsertToHead(node *KVNode) {
	node.prev = lru.head
	node.next = lru.head.next
	lru.head.next.prev = node
	lru.head.next = node
}

func (lru *LRUCache) RemoveTail() *KVNode {
	lastNode := lru.tail.prev
	lru.RemoveNode(lru.tail.prev)
	return lastNode
}

func (lru *LRUCache) RemoveNode(node *KVNode) {
	node.prev.next = node.next
	node.next.prev = node.prev
}
```



## LFU

```go
type KVNode struct {
	prev, next *KVNode
	key, value int
	freq       int
}

type DoubleList struct {
	head, tail *KVNode
	size       int
}

func NewDoubleList() *DoubleList {
	dl := &DoubleList{
		head: &KVNode{ key: 0, value: 0},
		tail: &KVNode{ key: 0, value: 0},
		size: 0,
	}
	dl.head.next = dl.tail
	dl.tail.prev = dl.head
	return dl
}

func (dl *DoubleList) PushFront(node *KVNode) {
	node.prev = dl.head
	node.next = dl.head.next
	dl.head.next.prev = node
	dl.head.next = node
	dl.size++
}

func (dl *DoubleList) RemoveBack() *KVNode {
	lastNode := dl.tail.prev
	dl.Remove(lastNode)
	return lastNode
}

func (dl *DoubleList) Remove(node *KVNode) {
	node.prev.next = node.next
	node.next.prev = node.prev
	dl.size--
}

type LFUCache struct {
	cache                   map[int]*KVNode
	freq                    map[int]*DoubleList
	capacity, size, minFreq int
}

func Constructor(capacity int) LFUCache {
	return LFUCache{
		cache:    map[int]*KVNode{},
		freq:     map[int]*DoubleList{},
		capacity: capacity,
		size:     0,
		minFreq:  0,
	}
}

func (lfu *LFUCache) IncFreq(node *KVNode) {
	preFreq := node.freq
	lfu.freq[preFreq].Remove(node)
	if preFreq == lfu.minFreq && lfu.freq[preFreq].size == 0 {
		lfu.minFreq++
		delete(lfu.freq, preFreq)
	}
	node.freq++
	if lfu.freq[node.freq] == nil {
		lfu.freq[node.freq] = NewDoubleList()
	}
	lfu.freq[node.freq].PushFront(node)
}

func (lfu *LFUCache) Get(key int) int {
	if node, ok := lfu.cache[key]; ok {
		lfu.IncFreq(node)
		return node.value
	}
	return -1
}

func (lfu *LFUCache) Put(key int, value int) {
	if lfu.capacity == 0 {
		return
	}

	if node, ok := lfu.cache[key]; ok {
		node.value = value
		lfu.IncFreq(node)
		return
	}

	if lfu.size == lfu.capacity {
		node := lfu.freq[lfu.minFreq].RemoveBack()
		delete(lfu.cache, node.key)
		lfu.size--
		// No need to update minFreq because it will be set 1 below
	}

	node := &KVNode{ key: key, value: value, freq: 1}
	lfu.cache[key] = node
	if lfu.freq[1] == nil {
		lfu.freq[1] = NewDoubleList()
	}
	lfu.freq[1].PushFront(node)
	lfu.minFreq = 1
	lfu.size++
}
```



---

> 作者: ingbyr  
> URL: https://ingbyr.github.io/posts/%E7%AE%97%E6%B3%95%E5%9F%BA%E7%A1%80-lru%E4%B8%8Elfu/  

