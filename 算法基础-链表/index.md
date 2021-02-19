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


