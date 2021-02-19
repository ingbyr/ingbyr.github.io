# 算法基础-链表


链表基础算法题目

<!--more-->

## [删除排序链表中的重复元素](https://leetcode-cn.com/problems/remove-duplicates-from-sorted-list/description/)

> 给定一个排序链表，删除所有重复的元素，使得每个元素只出现一次。

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



## [删除排序链表中的重复元素 II](https://leetcode-cn.com/problems/remove-duplicates-from-sorted-list-ii/description/)

> 给定一个排序链表，删除所有含有重复数字的节点，只保留原始链表中 *没有重复出现* 的数字。

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
		return deleteDuplicates(head.Next) // 重复的元素不保留，所以不是 head.Next = deleteDuplicates(head.Next)
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


