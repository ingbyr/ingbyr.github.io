# 算法基础-链表


链表基础算法题目

<!--more-->

## 删除排序链表中的重复元素

> [删除排序链表中的重复元素](https://leetcode-cn.com/problems/remove-duplicates-from-sorted-list/description/) 给定一个排序链表，删除所有重复的元素，使得每个元素只出现一次。\

递归

```go
func deleteDuplicates(head *ListNode) *ListNode {
	if head == nil || head.Next == nil {
		return head
	}
	next := head
	for next != nil && next.Val == head.Val {
		next = next.Next
	}
	head.Next = deleteDuplicates(next) // head的下个一节点是与head值不同的节点
	return head
}
```

迭代

```go
func deleteDuplicates(head *ListNode) *ListNode {
	if head == nil || head.Next == nil {
		return head
	}
	pre, cur := head, head.Next
	for cur != nil {
		if pre.Val == cur.Val { // 若与前一个元素相同，则跳过
			cur = cur.Next
		} else {
			pre.Next = cur
			pre = cur
		}
	}
	pre.Next = nil // pre之后的有可能是与pre相同的若干节点，故断开后续节点
	return head
}
```



## 删除排序链表中的重复元素 II

> [删除排序链表中的重复元素 II](https://leetcode-cn.com/problems/remove-duplicates-from-sorted-list-ii/description/) 给定一个排序链表，删除所有含有重复数字的节点，只保留原始链表中 *没有重复出现* 的数字。

递归

```go
func deleteDuplicates(head *ListNode) *ListNode {
	if head == nil || head.Next == nil {
		return head
	}
	if head.Val == head.Next.Val {
		for head != nil && head.Next != nil && head.Val == head.Next.Val {
			head = head.Next // head 与 head.Next值相同时，head指针无需保留，故向后移动
		}
		return deleteDuplicates(head.Next) // 重复的元素不保留
	}
	head.Next = deleteDuplicates(head.Next)
	return head
}
```

迭代

```go
func deleteDuplicates(head *ListNode) *ListNode {
	if head == nil || head.Next == nil {
		return head
	}
  dummyHead := &ListNode{0, head} // 头节点可能会被删除，故引入虚拟头节点
	head = dummyHead
	rmV := 0 // 重复值
	for head.Next != nil && head.Next.Next != nil {
		if head.Next.Val == head.Next.Next.Val {
			rmV = head.Next.Val // 删除 head 后的所有与rmV相同的节点
			for head.Next != nil && head.Next.Val == rmV {
				head.Next = head.Next.Next
			}
		} else {
			head = head.Next
		}
	}
	return dummyHead.Next
}
```



## 反转链表

> [反转链表](https://leetcode-cn.com/problems/reverse-linked-list/description/) 反转一个单链表。

递归

```go
func reverseList(head *ListNode) *ListNode {
	if head == nil || head.Next == nil {
		return head	// 返回反转后的头节点
	}
	newHead := reverseList(head.Next) // 递归获取新头节点
	head.Next.Next = head // 反转当前节点和其后继节点的指向
	head.Next = nil // 反转后的最后一个节点指向nil
	return newHead
}
```

迭代

```go
func reverseList(head *ListNode) *ListNode {
	if head == nil || head.Next == nil {
		return head
	}
	var pre *ListNode
	for head != nil {
		next := head.Next
		head.Next = pre
		pre = head
		head = next
	}
	return pre
}
```



## 反转链表 II

> [反转链表 II](https://leetcode-cn.com/problems/reverse-linked-list-ii/description/) 反转从位置 *m* 到 *n* 的链表。请使用一趟扫描完成反转。

递归

```go
func reverseBetween(head *ListNode, m int, n int) *ListNode {
	if m == 1 {
		return reverseN(head, n)
	}
	head.Next = reverseBetween(head.Next, m-1, n-1) // 递归移动到翻转区间
	return head
}

var next *ListNode // 保存翻转区间后的第一个节点
// 翻转链表的前n个节点
func reverseN(head *ListNode, n int) *ListNode {
	if n == 1 {
		next = head.Next
		return head
	}
	last := reverseN(head.Next, n-1)
	head.Next.Next = head
	head.Next = next // 反转后的最后一个节点会指向next，其他的会被上一行覆盖
	return last
}
```

迭代

```go
func reverseBetween(head *ListNode, m int, n int) *ListNode {
	if head == nil {
		return nil
	}
	dummyHead := &ListNode{0, head} // 头节点可能被反转
	// 反转区间表示为[l, r]，pre为区间外左侧第一个节点，next为右侧第一个节点
	// head 移动到 l
	head = dummyHead
	var pre *ListNode // 区间左侧第一个节点
	l := 0
	for l < m {
		pre = head
		head = head.Next
		l++
	}
	lNode := head // 位于 l 的节点
	var rNode *ListNode // 位于 r 的节点 （下面for循环结束后）
	r := l
	for r <= n {
		next := head.Next
		head.Next = inPre
		rNode = head // 循环中代表head的前一个节点
		head = next
		r++
	} // 结束时 head=next 位于r+1
	lNode.Next = next
	pre.Next = inPre
	return dummyHead.Next
}
```



## 合并两个有序链表

> [合并两个有序链表](https://leetcode-cn.com/problems/merge-two-sorted-lists/description/) 将两个升序链表合并为一个新的 **升序** 链表并返回。新链表是通过拼接给定的两个链表的所有节点组成的。 

递归

```go
func mergeTwoLists(l1 *ListNode, l2 *ListNode) *ListNode {
	if l1 == nil {
		return l2
	}
	if l2 == nil {
		return l1
	}
	var head *ListNode
	if l1.Val < l2.Val {
		head = l1
		head.Next = mergeTwoLists(l1.Next, l2)
	} else {
		head = l2
		head.Next = mergeTwoLists(l1, l2.Next)
	}
	return head
}
```

迭代

```go
func mergeTwoLists(l1 *ListNode, l2 *ListNode) *ListNode {
	dummyHead := &ListNode{0, nil}
	curr := dummyHead
	for l1 != nil && l2 != nil {
		if l1.Val < l2.Val {
			curr.Next = l1
			l1 = l1.Next
		} else {
			curr.Next = l2
			l2 = l2.Next
		}
		curr = curr.Next
	}
	if l1 == nil && l2 == nil {
		return dummyHead.Next
	}
	if l1 == nil {
		curr.Next = l2
	}
	if l2 == nil {
		curr.Next = l1
	}
	return dummyHead.Next
}
```



## 分隔链表

> [分隔链表](https://leetcode-cn.com/problems/partition-list/description/) 给你一个链表的头节点 `head` 和一个特定值 `x` ，请你对链表进行分隔，使得所有 **小于** `x` 的节点都出现在 **大于或等于** `x` 的节点之前。
>
> 你应当 **保留** 两个分区中每个节点的初始相对位置。

```go
func partition(head *ListNode, x int) *ListNode {
	if head == nil {
		return nil
	}
	smallHead := &ListNode{0, nil} // 保存小于x的节点
	small := smallHead
	largeHead := &ListNode{0, nil}	// 保存大于等于x的节点
	large := largeHead
	for node := head; node != nil; node = node.Next {
		if node.Val < x {
			small.Next = node
			small = small.Next
		} else {
			large.Next = node
			large = large.Next
		}
	}
	small.Next = largeHead.Next // 合并节点
	large.Next = nil	// 断开后续节点
	return smallHead.Next
}
```



## 排序链表

>[排序链表](https://leetcode-cn.com/problems/sort-list/description/) `O(n log n)` 时间复杂度和常数级空间复杂度下对链表进行排序。

```go
// 基于分治思想，使用归并排序
func sortList(head *ListNode) *ListNode {
	return sort(head, nil)
}

// 对区间 [left, right) 排序
func sort(left, right *ListNode) *ListNode {
	if left == nil {
		return nil
	}
	if left.Next == right { // 只有一个left节点
		left.Next = nil // 断开连接所有节点的链接，在merge中会排序后再次连接
		return left
	}
	// 快慢指针寻找中间节点
	slow, fast := left, left
	for fast != right && fast.Next != right { // 右侧结尾是right节点
		slow = slow.Next
		fast = fast.Next.Next
	}
	mid := slow
	return merge(sort(left, mid), sort(mid, right))
}

func merge(head1, head2 *ListNode) *ListNode {
	dummyHead := &ListNode{0, nil}
	cur, cur1, cur2 := dummyHead, head1, head2
	for cur1 != nil && cur2 != nil {
		if cur1.Val < cur2.Val {
			cur.Next = cur1
			cur1 = cur1.Next
		} else {
			cur.Next = cur2
			cur2 = cur2.Next
		}
		cur = cur.Next
	}
	if cur1 != nil {
		cur.Next = cur1
	} else if cur2 != nil {
		cur.Next = cur2
	}
	return dummyHead.Next
}
```



## 两两交换链表中的节点

> [两两交换链表中的节点](https://leetcode-cn.com/problems/swap-nodes-in-pairs/description/) 给定一个链表，两两交换其中相邻的节点，并返回交换后的链表。

递归

```go
func swapPairs(head *ListNode) *ListNode {
	if head == nil || head.Next == nil {
		return head
	}
	pre, cur, next := head, head.Next, head.Next.Next
	cur.Next = pre
	pre.Next = swapPairs(next)
	return cur
}
```



## 环形链表

> [环形链表](https://leetcode-cn.com/problems/linked-list-cycle/description/) 给定一个链表，判断链表中是否有环。

```go
func hasCycle(head *ListNode) bool {
	if head == nil || head.Next == nil {
		return false
	}
	slow, fast := head, head
	for fast != nil && fast.Next != nil {
		slow = slow.Next
		fast = fast.Next.Next
		if slow == fast { // 在环内每执行一次快慢指针距离减1
			return true
		}
	}
	return false
}
```



## 环形链表 II

> [环形链表 II](https://leetcode-cn.com/problems/linked-list-cycle-ii/description/)  给定一个链表，返回链表开始入环的第一个节点。 如果链表无环，则返回 `null`。
>
> [原始题解](https://leetcode-cn.com/problems/linked-list-cycle-ii/solution/linked-list-cycle-ii-kuai-man-zhi-zhen-shuang-zhi-/)

```
         a            入环点          
+----------------------#------------------X 相遇点
                       |                  |
                       |                  |
                       |                  |
                       +------------------+
                                 b
```

入环前节点个数为`a`，环内节点个数为`b`，快指针移动距离`f`，慢指针移动距离`s`，快慢指针**相遇**时有：

- `f = 2s`  快指针移动距离是慢指针的两倍
- `f = s + nb`  双指针都走过 `a` 步，然后在环内绕圈直到重合，重合时 `fast` 比 `slow` 多走 **环的长度整数倍** 

由上可得：

- `f = 2nb` `fast`移动了`2n`个环长
- `s = nb` `slow`移动了n个环长

又当指针`ptr`处于入环点时有`k = a + nb`，因此只需要使`slow`移动`a`个长度则可使slow指向入环点，而`a`长度可以由头节点移动到入环点得到，此时`ptr = slow = #`，即都指向入环点。

```go
func detectCycle(head *ListNode) *ListNode {
	if head == nil || head.Next == nil {
		return nil
	}
	slow, fast := head, head
	for fast != nil && fast.Next != nil {
		slow = slow.Next
		fast = fast.Next.Next
		if slow == fast { // 相遇点
			p := head // p指向头节点开始移动a个距离
			for p != slow {
				p = p.Next
				slow = slow.Next
			}
			return p
		}
	}
	// 无环
	return nil
}
```



## 回文链表

> [回文链表](https://leetcode-cn.com/problems/palindrome-linked-list/description/) 判断一个链表是否为回文链表。

```go
func isPalindrome(head *ListNode) bool {
	if head == nil || head.Next == nil {
		return true
	}
	// 快慢指针找中点
	slow, fast := head, head
	for fast != nil && fast.Next != nil {
		slow = slow.Next
		fast = fast.Next.Next
	}
	// 递归反转链表
	right := reverse(slow)
	// 判断回文
	for left := head; left != nil && right != nil; left, right = left.Next, right.Next {
		if left.Val != right.Val {
			return false
		}
	}
	return true
}

func reverse(head *ListNode) *ListNode {
	if head == nil || head.Next == nil {
		return head
	}
	last := reverse(head.Next)
	head.Next.Next = head
	head.Next = nil
	return last
}
```



---

> 作者: ingbyr  
> URL: https://ingbyr.github.io/posts/%E7%AE%97%E6%B3%95%E5%9F%BA%E7%A1%80-%E9%93%BE%E8%A1%A8/  

