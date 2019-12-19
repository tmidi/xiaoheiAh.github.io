# 

链表需要注意的问题:

* 边界: 头结点尾结点的处理,链表长度为1时的处理
* 多画图,跟一次循环,边界情况也画图试试
* 思路大多都是 **快慢指针**

[No.19 => Remove Nth Node From End of List](https://leetcode.com/problems/remove-nth-node-from-end-of-list/) **Medium**

```java
public ListNode removeNthFromEnd(ListNode head, int n) {
  // 快慢指针
  ListNode fast = head,slow = head,prev = null;
  while(fast.next != null) {
    if (--n <= 0) {
      // 先走 n 步后,slow 再走
      prev = slow;
      slow = slow.next;
    }
    fast = fast.next;
  }
  // fast 走完,slow 刚好到倒数 n 的位置
  // 删除 slow 节点
  if( prev == null) {
    // 说明删除的是头结点
    if(slow.next == null) {
      // 说明链表只有一个节点.且需要删除的就是这个.
      head = null;
    } else {
      // 大于一个节点就需要把 head 指向 slow.next
      head = prev = slow.next;
    }
  } else {
    prev.next = slow.next;
  }
  return head;
}
```

[No.2 => Add Two Numbers](https://leetcode.com/problems/add-two-numbers/) **Medium**

思路: 就按着这个链表顺序加,然后生成一个链表就自然是倒序的了.

```java
public ListNode addTwoNumbers(ListNode l1, ListNode l2) {
  // 一次累加,reverse
  ListNode result = new ListNode(0);
  ListNode head = result;
  int carry = 0;
  while(l1 != null || l2 != null || carry > 0) {
    int v1 = 0;
    int v2 = 0;
    if(l1 != null) {
      v1 = l1.val;
      l1 = l1.next;
    }
    if(l2 != null) {
      v2 = l2.val;
      l2 = l2.next;
    }
    int sum = v1 + v2 + carry;
    result.next = new ListNode( sum % 10);
    result = result.next;
    carry = sum / 10 ;
  }
  return head.next;
}
```

