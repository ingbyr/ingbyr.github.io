# 算法基础-前缀树


字符串前缀相关操作

<!--more-->

## 实现

```go
package main

type TrieNode struct {
	isWord   bool
	children []*TrieNode
}

func NewTrieNode() *TrieNode {
	return &TrieNode{
		isWord:   false,
		children: make([]*TrieNode, 26),
	}
}

type Trie struct {
	root *TrieNode
}

func NewTrie() Trie {
	return Trie{NewTrieNode()}
}

func (t *Trie) Insert(word string) {
	node := t.root
	for i := range word {
		idx := int(word[i] - 'a')
		if node.children[idx] == nil {
			node.children[idx] = NewTrieNode()
		}
		node = node.children[idx]
	}
	node.isWord = true
}

func (t *Trie) Search(word string) bool {
	node := t.GetNode(word)
	if node == nil || !node.isWord {
		return false
	}
	return true
}

func (t *Trie) StartsWith(prefix string) bool {
	return t.GetNode(prefix) != nil
}

func (t *Trie) GetNode(word string) *TrieNode {
	node := t.root
	for i := range word {
		idx := int(word[i] - 'a')
		child := node.children[idx]
		if child == nil {
			return nil
		}
		node = child
	}
	return node
}

func (t *Trie) Scan(prefix string) []string {
	node := t.GetNode(prefix)
	res := make([]string, 0)
	if node == nil {
		return res
	}
	var dfs func(node *TrieNode, path []byte)
	dfs = func(node *TrieNode, path []byte) {
		if node == nil {
			return
		}
		if node.isWord {
			res = append(res, prefix+string(path))
		}
		for i := 0; i < 26; i++ {
			child := node.children[i]
			if child != nil {
				path = append(path, byte(i+'a'))
				dfs(child, path)
				path = path[:len(path)-1]
			}
		}
	}
	dfs(node, []byte{})
	return res
}
```


---

> 作者: ingbyr  
> URL: https://ingbyr.github.io/posts/%E7%AE%97%E6%B3%95%E5%9F%BA%E7%A1%80-%E5%89%8D%E7%BC%80%E6%A0%91/  

