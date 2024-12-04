---
layout: post
title: "leetcode"
subtitle: "leetcode"
author: "DYC"
header-img: "img/post-bg-linux.jpg"
header-mask: 0.3
catalog: true
tags:
- java

---

### 链表

###### [LRU 缓存](https://leetcode.cn/problems/lru-cache/)
get方法实现的时候，需要注意因为是lru，所以在获取之后需要把该节点移动到链表的头节点位置，添加也是

```java
class DLinkedNode {
        int key;
        int value;
        DLinkedNode prev;
        DLinkedNode next;
        public DLinkedNode() {}
        public DLinkedNode(int _key, int _value) {
          key = _key; 
          value = _value;
        }
    }
public class LRUCache {
    private Map<Integer, DLinkedNode> cache = new HashMap<Integer, DLinkedNode>();
    private int size;
    private int capacity;
      //head tail只是代表链表的头和尾 不代表具体的值
    private DLinkedNode head, tail;

    public LRUCache(int capacity) {
        this.size = 0;
        this.capacity = capacity;
        // 使用伪头部和伪尾部节点
        head = new DLinkedNode();
        tail = new DLinkedNode();
        head.next = tail;
        tail.prev = head;
    }

    public int get(int key) {
        DLinkedNode node = cache.get(key);
        if (node == null) {
            return -1;
        }
        // 如果 key 存在，先通过哈希表定位，再移到头部
        moveToHead(node);
        return node.value;
    }

    public void put(int key, int value) {
        DLinkedNode node = cache.get(key);
        if (node == null) {
            // 如果 key 不存在，创建一个新的节点
            DLinkedNode newNode = new DLinkedNode(key, value);
            // 添加进哈希表
            cache.put(key, newNode);
            // 添加至双向链表的头部
            addToHead(newNode);
            ++size;
            if (size > capacity) {
                // 如果超出容量，删除双向链表的尾部节点
                DLinkedNode tail = removeTail();
                // 删除哈希表中对应的项
                cache.remove(tail.key);
                --size;
            }
        }
        else {
            // 如果 key 存在，先通过哈希表定位，再修改 value，并移到头部
            node.value = value;
            moveToHead(node);
        }
    }

    private void addToHead(DLinkedNode node) {
        node.prev = head;
        node.next = head.next;
        head.next.prev = node;
        head.next = node;
    }

    private void removeNode(DLinkedNode node) {
        node.prev.next = node.next;
        node.next.prev = node.prev;
    }

    private void moveToHead(DLinkedNode node) {
        removeNode(node);
        addToHead(node);
    }

    private DLinkedNode removeTail() {
        DLinkedNode res = tail.prev;
        removeNode(res);
        return res;
    }
}

```

###### [用栈实现队列](https://leetcode.cn/problems/implement-queue-using-stacks/)
```java
class MyQueue {
    Stack<Integer>inStack;
    Stack<Integer>outStack;
    public MyQueue() {
        inStack=new Stack<>();
        outStack=new Stack<>();
    }
    
    public void push(int x) {
        inStack.push(x);
    }
    
    public int pop() {
        if(outStack.isEmpty())
            in2out();
        return outStack.pop();
    }
    
    public int peek() {
        if(outStack.isEmpty())
            in2out();
        return outStack.peek();
    }
    
    public boolean empty() {
        return inStack.isEmpty()&&outStack.isEmpty();
    }

    public void in2out(){
        while(!inStack.isEmpty())
            outStack.push(inStack.pop());
    }
}

/**
 * Your MyQueue object will be instantiated and called as such:
 * MyQueue obj = new MyQueue();
 * obj.push(x);
 * int param_2 = obj.pop();
 * int param_3 = obj.peek();
 * boolean param_4 = obj.empty();
 */
```

###### [用队列实现栈](https://leetcode.cn/problems/implement-stack-using-queues/)
```java
class MyStack {
    Queue<Integer>queue1;
    Queue<Integer>queue2;
    public MyStack() {
        queue1=new LinkedList<>();
        queue2=new LinkedList<>();
    }
    
    public void push(int x) {
        queue2.add(x);
        while(!queue1.isEmpty()){
            queue2.add(queue1.poll());
        }
        Queue<Integer>temp=queue1;
        queue1=queue2;
        queue2=temp;
    }
    
    public int pop() {
        return queue1.poll();
    }
    
    public int top() {
        return queue1.peek();
    }
    
    public boolean empty() {
        return queue1.isEmpty();
    }
}

/**
 * Your MyStack object will be instantiated and called as such:
 * MyStack obj = new MyStack();
 * obj.push(x);
 * int param_2 = obj.pop();
 * int param_3 = obj.top();
 * boolean param_4 = obj.empty();
 */
```

###### [排序链表](https://leetcode.cn/problems/7WHec2/)
方法1️⃣：逐个插入 超时

```java
/**
 * Definition for singly-linked list.
 * public class ListNode {
 *     int val;
 *     ListNode next;
 *     ListNode() {}
 *     ListNode(int val) { this.val = val; }
 *     ListNode(int val, ListNode next) { this.val = val; this.next = next; }
 * }
 */
class Solution {
    public ListNode sortList(ListNode head) {
        ListNode res=new ListNode(-1);
        while(head!=null){
            ListNode temp=head.next;
            ListNode r=res;
            while(r!=null){
                if(r.next==null){
                    head.next=r.next;
                    r.next=head;
                    break;
                }
                if(r.next.val>head.val){
                    head.next=r.next;
                    r.next=head;
                    break;
                }
                r=r.next;
            }
            head=temp;
        }
        return res.next;
    }
}
```

方法2️⃣：使用有序队列

```java
/**
 * Definition for singly-linked list.
 * public class ListNode {
 *     int val;
 *     ListNode next;
 *     ListNode() {}
 *     ListNode(int val) { this.val = val; }
 *     ListNode(int val, ListNode next) { this.val = val; this.next = next; }
 * }
 */
class Solution {
    public ListNode sortList(ListNode head) {
        PriorityQueue<ListNode>queue=new PriorityQueue<>(
            (a,b)->{
                return a.val-b.val;
            }
        );
        while(head!=null){
            queue.add(head);
            head=head.next;
        }
        ListNode res=new ListNode(-1);
        ListNode node=res;
        while(!queue.isEmpty()){
            node.next=queue.poll();
            node=node.next;
        }
        node.next=null;
        return res.next;
    }
}
```

注意有序队列的定义：

```java
        PriorityQueue<ListNode>queue=new PriorityQueue<>(
            (a,b)->{
                return a.val-b.val;
            }
        );
```

##### 双指针
###### 排序奇升偶降链表
```java
class ListNode {
    int val;
    ListNode next;

    ListNode(int x) {
        val = x;
        next = null;
    }
}

public class Solution {
    public ListNode sortOddEvenList(ListNode head) {
        if (head == null || head.next == null) {
            return head;
        }
        ListNode[] lists = partition(head);
        ListNode oddList = lists[0];
        ListNode evenList = lists[1];
        evenList = reverse(evenList);
        return merge(oddList, evenList);
    }

    private ListNode[] partition(ListNode head) {
        ListNode evenHead = head.next;
        ListNode odd = head, even = evenHead;
        while (even != null && even.next != null) {
            odd.next = even.next;
            odd = odd.next;
            even.next = odd.next;
            even = even.next;
        }
        odd.next = null; // important to terminate the odd list
        return new ListNode[] { head, evenHead };
    }

    private ListNode reverse(ListNode head) {
        ListNode dummy = new ListNode(-1);
        ListNode p = head;
        while (p != null) {
            ListNode temp = p.next;
            p.next = dummy.next;
            dummy.next = p;
            p = temp;
        }
        return dummy.next;
    }

    private ListNode merge(ListNode p, ListNode q) {
        ListNode dummy = new ListNode(-1);
        ListNode r = dummy;
        while (p != null && q != null) {
            if (p.val <= q.val) {
                r.next = p;
                p = p.next;
            } else {
                r.next = q;
                q = q.next;
            }
            r = r.next;
        }
        if (p != null) {
            r.next = p;
        }
        if (q != null) {
            r.next = q;
        }
        return dummy.next;
    }
}
```

###### [重排链表](https://leetcode.cn/problems/reorder-list/)
不能只是单纯的改变节点内部的值，而是需要实际的进行节点交换

```java
class Solution {
    public void reorderList(ListNode head) {
        if (head == null) {
            return;
        }
        ListNode mid = middleNode(head);
        ListNode l1 = head;
        ListNode l2 = mid.next;
        mid.next = null;
        l2 = reverseList(l2);
        mergeList(l1, l2);
    }

    public ListNode middleNode(ListNode head) {
        ListNode slow = head;
        ListNode fast = head;
        while (fast.next != null && fast.next.next != null) {
            slow = slow.next;
            fast = fast.next.next;
        }
        return slow;
    }

    public ListNode reverseList(ListNode head) {
        ListNode prev = null;
        ListNode curr = head;
        while (curr != null) {
            ListNode nextTemp = curr.next;
            curr.next = prev;
            prev = curr;
            curr = nextTemp;
        }
        return prev;
    }

    public void mergeList(ListNode l1, ListNode l2) {
        ListNode l1_tmp;
        ListNode l2_tmp;
        while (l1 != null && l2 != null) {
            l1_tmp = l1.next;
            l2_tmp = l2.next;

            l1.next = l2;
            l1 = l1_tmp;

            l2.next = l1;
            l2 = l2_tmp;
        }
    }
}

```

###### [分隔链表](https://leetcode.cn/problems/partition-list/)
思路 定义两个链表，一个存储小的，一个存储大于等于的，最后进行拼接

```java
/**
 * Definition for singly-linked list.
 * public class ListNode {
 *     int val;
 *     ListNode next;
 *     ListNode() {}
 *     ListNode(int val) { this.val = val; }
 *     ListNode(int val, ListNode next) { this.val = val; this.next = next; }
 * }
 */
class Solution {
    public ListNode partition(ListNode head, int x) {
        ListNode bigger=new ListNode(-1);
        ListNode biggertemp=bigger;
        ListNode smaller=new ListNode(-1);
        ListNode smallertemp=smaller;
        while(head!=null){
            if(head.val>=x){
                biggertemp.next=head;
                head=head.next;
                biggertemp=biggertemp.next;
            }else{
                smallertemp.next=head;
                head=head.next;
                smallertemp=smallertemp.next;
            }
        }
        biggertemp.next=null;
        smallertemp.next=bigger.next;
        return smaller.next;
    }
}
```

###### [合并两个有序链表](https://leetcode.cn/problems/merge-two-sorted-lists/)
合并有序链表

```java
/**
 * Definition for singly-linked list.
 * public class ListNode {
 *     int val;
 *     ListNode next;
 *     ListNode() {}
 *     ListNode(int val) { this.val = val; }
 *     ListNode(int val, ListNode next) { this.val = val; this.next = next; }
 * }
 */
class Solution {
    public ListNode mergeTwoLists(ListNode list1, ListNode list2) {
        if(list1==null||list2==null){
            if(list1!=null)
                return list1;
            else
                return list2;
        }
        ListNode result=new ListNode(-1);
        ListNode temp=result;
        while(list1!=null&&list2!=null){
            if(list1.val>list2.val){
                temp.next=list2;
                temp=temp.next;
                list2=list2.next;
            }else{
                temp.next=list1;
                temp=temp.next;
                list1=list1.next;
            }
        }
        if(list1==null)
            temp.next=list2;
        else if(list2==null)
            temp.next=list1;
        return result.next;
    }
}
```

###### [ 合并 K 个升序链表](https://leetcode.cn/problems/merge-k-sorted-lists/)
解法1️⃣

合并k个有序链表

思路：循环遍历，以第一个链表为标准，不断把后面的链表插入第一个链表中

```java
/**
 * Definition for singly-linked list.
 * public class ListNode {
 *     int val;
 *     ListNode next;
 *     ListNode() {}
 *     ListNode(int val) { this.val = val; }
 *     ListNode(int val, ListNode next) { this.val = val; this.next = next; }
 * }
 */
class Solution {
    public ListNode mergeKLists(ListNode[] lists) {
        if(lists==null||lists.length==0)
            return null;
        ListNode res=new ListNode(-1);
        res.next=lists[0];
        for(int i=1;i<lists.length;i++){
            ListNode p1=res;
            ListNode p2=lists[i];
            while(p1.next!=null&&p2!=null){
                if(p1.next.val<p2.val){
                    p1=p1.next;
                }else{
                    ListNode temp=p2.next;
                    p2.next=p1.next;
                    p1.next=p2;
                    p2=temp;
                    p1=p1.next;
                }
            }
            if(p2!=null){
                p1.next=p2;
            }
        }
        return res.next;
    }
}
```

解法2️⃣

思路：将数组中各个链表头节点加入有序队列中，然后定义一个新的链表加入其中最小的头节点，如果被加入的那个链表不为空，把它的下一个加入有序队列中，继续进行比较，知道队列为空

```java
/**
 * Definition for singly-linked list.
 * public class ListNode {
 *     int val;
 *     ListNode next;
 *     ListNode() {}
 *     ListNode(int val) { this.val = val; }
 *     ListNode(int val, ListNode next) { this.val = val; this.next = next; }
 * }
 */
class Solution {
    public ListNode mergeKLists(ListNode[] lists) {
        if(lists.length==0)
            return null;
        PriorityQueue<ListNode>queue=new PriorityQueue<>(
            lists.length,(a,b)->(a.val-b.val)
        );
        for(ListNode head:lists){
            if(head!=null)
                queue.add(head);
        }
        ListNode res=new ListNode(-1);
        ListNode temp=res;
        while(!queue.isEmpty()){
            ListNode node=queue.poll();
            temp.next=node;
            if(node.next!=null)
                queue.add(node.next);
            temp=temp.next;
        }
        return res.next;
    }
}
```

###### [删除链表的倒数第 N 个结点](https://leetcode.cn/problems/SLwz0R/)
删除链表倒数第n个节点

双链表，一个快指针，一个慢指针，让快指针先走n个，慢指针再走

```java
class Solution {
    public ListNode removeNthFromEnd(ListNode head, int n) {
        if(head==null)
            return null;
        int index=0;
        ListNode fast=new ListNode(-1);
        ListNode slow=new ListNode(-1);
        fast.next=head;
        slow.next=head;
        ListNode slowtemp=slow;
        while(fast.next!=null&&index<n){
            fast=fast.next;
            index++;
        }
        if(fast.next==null&&index<n){
            return null;
        }
        while(fast.next!=null){
            fast=fast.next;
            slowtemp=slowtemp.next;
        }
        slowtemp.next=slowtemp.next.next;
        return slow.next;
        
    }
}
```

方法2️⃣：

```java
/**
 * Definition for singly-linked list.
 * public class ListNode {
 *     int val;
 *     ListNode next;
 *     ListNode() {}
 *     ListNode(int val) { this.val = val; }
 *     ListNode(int val, ListNode next) { this.val = val; this.next = next; }
 * }
 */
class Solution {
    public ListNode removeNthFromEnd(ListNode head, int n) {
        ListNode temp=head;
        while(n>0&&temp!=null){
            temp=temp.next;
            n--;
        }
        if(temp==null&&n==0)
            return head.next;
        if(temp==null)
            return null;
        ListNode slow=head;
        while(temp.next!=null){
            slow=slow.next;
            temp=temp.next;
        }
        slow.next=slow.next.next;
        return head;
    }
}
```

###### [链表的中间结点](https://leetcode.cn/problems/middle-of-the-linked-list/)
方法1️⃣

直接首先while循环一遍计数，然后for循环

```java
class Solution {
    public ListNode middleNode(ListNode head) {
        ListNode temp=head;
        int num=0;
        while(temp!=null){
            temp=temp.next;
            num++;
        }
        ListNode res=head;
        for(int i=0;i<num/2;i++){
            res=res.next;
        }
        return res;
    }
}
```

方法2️⃣

设置快指针

```java
ListNode middleNode(ListNode head) {
    // 快慢指针初始化指向 head
    ListNode slow = head, fast = head;
    // 快指针走到末尾时停止
    while (fast != null && fast.next != null) {
        // 慢指针走一步，快指针走两步
        slow = slow.next;
        fast = fast.next.next;
    }
    // 慢指针指向中点
    return slow;
}
```

###### [环形链表](https://leetcode.cn/problems/linked-list-cycle/)
思路：定义一个快指针和一个慢指针，如果两个能相遇，那就说明是环形

```java
public class Solution {
    public boolean hasCycle(ListNode head) {
        ListNode fast=head;
        ListNode slow=head;
        while(fast!=null&&fast.next!=null){
            fast=fast.next.next;
            slow=slow.next;
            if(fast==slow)
                return true;
        }
        return false;
    }
}
```

###### [环形链表 II](https://leetcode.cn/problems/linked-list-cycle-ii/)
思路：**当快慢指针相遇时，让其中任意一个指针指向头节点，然后让它俩以相同速度前进，再次相遇时所在的节点位置就是环开始的位置**

原理：我们假设快慢指针相遇时，慢指针 `slow` 走了 `k` 步，那么快指针 `fast` 一定走了 `2k` 步：

![](https://cdn.jsdelivr.net/gh/ddyycc123/imageloader@main/image-20231101175243112.png)

`fast` 一定比 `slow` 多走了 `k` 步，这多走的 `k` 步其实就是 `fast` 指针在环里转圈圈，所以 `k` 的值就是环长度的「整数倍」。

假设相遇点距环的起点的距离为 `m`，那么结合上图的 `slow` 指针，环的起点距头结点 `head` 的距离为 `k - m`，也就是说如果从 `head` 前进 `k - m` 步就能到达环起点。

巧的是，如果从相遇点继续前进 `k - m` 步，也恰好到达环起点。因为结合上图的 `fast` 指针，从相遇点开始走k步可以转回到相遇点，那走 `k - m` 步肯定就走到环起点了：

![](https://cdn.jsdelivr.net/gh/ddyycc123/imageloader@main/image-20231101175916495.png)

```java
public class Solution {
    public ListNode detectCycle(ListNode head) {
        ListNode fast=head;
        ListNode slow=head;
        while(fast!=null&&fast.next!=null){
            fast=fast.next.next;
            slow=slow.next;
            if(fast==slow)
                break;
        }
        if(fast==null||fast.next==null)
            return null;
        slow=head;
        while(fast!=slow){
            fast=fast.next;
            slow=slow.next;
        }
        return slow;

    }
}
```

###### [相交链表](https://leetcode.cn/problems/intersection-of-two-linked-lists/)
解法1️⃣

思路：将一个链表加入hashset中进行判断

```java
public class Solution {
    public ListNode getIntersectionNode(ListNode headA, ListNode headB) {
        HashSet<ListNode>hashset1=new HashSet<>();
        while(headA!=null){
            hashset1.add(headA);
            headA=headA.next;
        }
        while(headB!=null){
            if(hashset1.contains(headB))
                return headB;
            headB=headB.next;
        }
        return null;
    }
}
```

解法2️⃣

思路：将headA遍历完A之后遍历B，让headB遍历完B之后遍历A，这样如果两者能相遇就说明有交点，如果没有交点，那就都遍历完都为空，退出循环

![](https://cdn.jsdelivr.net/gh/ddyycc123/imageloader@main/image-20231102160749946.png)

```java
public class Solution {
    public ListNode getIntersectionNode(ListNode headA, ListNode headB) {
        ListNode p1=headA;
        ListNode p2=headB;
        while(p1!=p2){
            if(p1!=null)
                p1=p1.next;
            else
                p1=headB;
            if(p2!=null)
                p2=p2.next;
            else
                p2=headA;
        }
        return p1;
    }
}
```

###### k个有序数组求交集
k个数组求交集

思路：首先定义k个指针指向每个数组的头部，接着求每个指针当前元素中最大值，接着其余指针通过二分查找在其余数组中大于或等于该元素的位置，如果都能找到就加入集合，如果不能就继续之前的操作，直到某一个数组遍历完即可

```java
import java.util.*;

public class IntersectionOfSortedArrays {
    public static List<Integer> findIntersection(int[][] arrays) {
        // 检查输入的有效性
        if (arrays == null || arrays.length == 0) {
            return new ArrayList<>();
        }

        // 初始化指针数组
        int k = arrays.length;
        int[] indices = new int[k];
        List<Integer> result = new ArrayList<>();

        while (true) {
            int maxElement = Integer.MIN_VALUE;
            boolean isIntersection = true;

            // 找到所有数组中当前指针所指元素的最大值
            for (int i = 0; i < k; i++) {
                if (indices[i] == arrays[i].length) {
                    // 如果任意一个数组已经被遍历完，返回结果
                    return result;
                }
                maxElement = Math.max(maxElement, arrays[i][indices[i]]);
            }

            // 检查所有数组的当前指针是否都指向这个最大值
            for (int i = 0; i < k; i++) {
                // 如果当前数组的指针不指向最大值，则移动指针
                if (arrays[i][indices[i]] != maxElement) {
                    indices[i] = findFirstNotLessThan(arrays[i], maxElement, indices[i]);
                    isIntersection = false;
                }
            }

            // 如果所有指针都指向同一个元素，添加到结果中
            if (isIntersection) {
                result.add(maxElement);
                // 移动所有指针
                for (int i = 0; i < k; i++) {
                    indices[i]++;
                }
            }
        }
    }

    // 使用二分查找找到第一个不小于目标值的元素的索引
    private static int findFirstNotLessThan(int[] array, int target, int start) {
        int left = start, right = array.length;
        while (left < right) {
            int mid = left + (right - left) / 2;
            if (array[mid] < target) {
                left = mid + 1;
            } else {
                right = mid;
            }
        }
        return left;
    }

    public static void main(String[] args) {
        int[][] arrays = {
            {1, 2, 3, 4, 5},
            {3, 4, 5, 6, 7},
            {5, 6, 7, 8, 9}
        };

        List<Integer> intersection = findIntersection(arrays);
        System.out.println("Intersection: " + intersection);
    }
}

```

###### [删除排序链表中的重复元素](https://leetcode.cn/problems/remove-duplicates-from-sorted-list/)
给定一个已排序的链表的头 `head` ， 删除所有重复的元素，使每个元素只出现一次 。

返回 已排序的链表 

删除链表重复元素，这个是重复元素只保留一个

```java
class Solution {
    public ListNode deleteDuplicates(ListNode head) {
        if(head==null)
            return null;
        ListNode slow=head,fast=head;
        while(fast!=null){
            if(fast.val!=slow.val){
                slow=slow.next;
                slow.val=fast.val;
            }
            fast=fast.next;
        }
        slow.next=null;
        return head;
    }
}
```

###### [删除排序链表中的重复元素 II](https://leetcode.cn/problems/remove-duplicates-from-sorted-list-ii/)
给定一个已排序的链表的头 `head` ， 删除原始链表中所有重复数字的节点，只留下不同的数字 。

返回 已排序的链表 

这个是重复元素都需要删除

```java
/**
 * Definition for singly-linked list.
 * public class ListNode {
 *     int val;
 *     ListNode next;
 *     ListNode() {}
 *     ListNode(int val) { this.val = val; }
 *     ListNode(int val, ListNode next) { this.val = val; this.next = next; }
 * }
 */
class Solution {
    public ListNode deleteDuplicates(ListNode head) {
        ListNode res=new ListNode(-1);
        ListNode slow=res;
        ListNode fast=head;
        while(fast!=null){
            if(fast.next!=null&&fast.val==fast.next.val){
                while(fast.next!=null&&fast.val==fast.next.val)
                    fast=fast.next;
                fast=fast.next;
            }else{
                slow.next=new ListNode(fast.val);
                slow=slow.next;
                fast=fast.next;
            }
        }
        return res.next;
    }
}
```

###### [删除链表的节点](https://leetcode.cn/problems/shan-chu-lian-biao-de-jie-dian-lcof/)
```java
/**
 * Definition for singly-linked list.
 * public class ListNode {
 *     int val;
 *     ListNode next;
 *     ListNode(int x) { val = x; }
 * }
 */
class Solution {
    public ListNode deleteNode(ListNode head, int val) {
        if(head.val==val)
            return head.next;
        ListNode pre=head;
        ListNode node=head.next;
        while(node!=null){
            if(node.val==val){
                pre.next=node.next;
                break;
            }
            pre=pre.next;
            node=node.next;
        }
        return head;
    }
}
```

###### [复杂链表的复制](https://leetcode.cn/problems/fu-za-lian-biao-de-fu-zhi-lcof/)
请实现 `copyRandomList` 函数，复制一个复杂链表。在复杂链表中，每个节点除了有一个 `next` 指针指向下一个节点，还有一个 `random` 指针指向链表中的任意节点或者 `null`。

链表复制

难点在于如何处理random

思路：分成三步，第一步将原链表的结点对应的拷贝节点连在其后，最后链表变成 原1 -> 拷1 -> 原2 -> 拷2 -> … -> null 的形式。第二步将一个指针指向原链表的节点， 一个指向拷贝链表的节点，那么就有 拷->random = 原->random->next (random不为空)。第三步用双指针将两条链表拆分。

```java
/*
// Definition for a Node.
class Node {
    int val;
    Node next;
    Node random;

    public Node(int val) {
        this.val = val;
        this.next = null;
        this.random = null;
    }
}
*/
class Solution {
    public Node copyRandomList(Node head) {
        if(head==null)
            return null;
        Node node=head;
        while(node!=null){
            Node temp=new Node(node.val);
            temp.next=node.next;
            node.next=temp;
            node=node.next.next;
        }
        Node fast=head;
        while(fast!=null){
            if(fast.random!=null)
                fast.next.random=fast.random.next;
            fast=fast.next.next;
        }
        Node slow=head.next;
        Node res=slow;
        fast=head;
        while(fast!=null){
            if(fast.next!=null)
                fast.next=fast.next.next;
            if(slow.next!=null)
                slow.next=slow.next.next;
            fast=fast.next;
            slow=slow.next;
        }
        return res;
    }
}
```

###### [两数相加](https://leetcode.cn/problems/add-two-numbers/)
链表相加

代码写的很冗杂，就是最基础的想法挨个相加，再考虑进位的问题

```java
/**
 * Definition for singly-linked list.
 * public class ListNode {
 *     int val;
 *     ListNode next;
 *     ListNode() {}
 *     ListNode(int val) { this.val = val; }
 *     ListNode(int val, ListNode next) { this.val = val; this.next = next; }
 * }
 */
class Solution {
    public ListNode addTwoNumbers(ListNode l1, ListNode l2) {
        ListNode res=new ListNode(-1);
        ListNode s=res;
        int i=0;
        while(l1!=null&&l2!=null){
            int sum=l1.val+l2.val+i;
            ListNode node=new ListNode(sum%10);
            i=sum/10;
            s.next=node;
            s=s.next;
            l1=l1.next;
            l2=l2.next;
        }
        if(l1==null&&l2==null){
            if(i==0)
                return res.next;
            ListNode n=new ListNode(i);
            s.next=n;
            return res.next;
        }
        if(l1!=null){
            if(i==0){
                s.next=l1;
                return res.next;
            }
            while(i!=0&&l1!=null){
                int sum=l1.val+i;
                ListNode n=new ListNode(sum%10);
                i=sum/10;
                s.next=n;
                s=s.next;
                l1=l1.next;
            }
            if(i!=0){
                ListNode n=new ListNode(i);
                s.next=n;
            }
            if(l1!=null){
                s.next=l1;
            }
            return res.next;
        }
        if(l2!=null){
            if(i==0){
                s.next=l2;
                return res.next;
            }
            while(i!=0&&l2!=null){
                int sum=l2.val+i;
                ListNode n=new ListNode(sum%10);
                i=sum/10;
                s.next=n;
                s=s.next;
                l2=l2.next;
            }
            if(i!=0){
                ListNode n=new ListNode(i);
                s.next=n;
            }
            if(l2!=null){
                s.next=l2;
            }
            return res.next;
        }
        return null;
    }

}
```

###### [旋转链表](https://leetcode.cn/problems/rotate-list/)
给你一个链表的头节点 `head` ，旋转链表，将链表每个节点向右移动 `k` 个位置

思路：先遍历一遍链表，记录链表的个数，并找到最后一个节点，接着把链表变成环形链表（让最后一个节点指向头节点），接着根据k找到新的头节点，并在该位置断开环形链表即可

```java
/**
 * Definition for singly-linked list.
 * public class ListNode {
 *     int val;
 *     ListNode next;
 *     ListNode() {}
 *     ListNode(int val) { this.val = val; }
 *     ListNode(int val, ListNode next) { this.val = val; this.next = next; }
 * }
 */
class Solution {
    public ListNode rotateRight(ListNode head, int k) {
        if(head==null||k==0)
            return head;
        ListNode temp=head;
        int count=1;
        while(temp.next!=null){
            temp=temp.next;
            count++;
        }
        temp.next=head;
        k=k%count;
        k=count-k;
        while(k>0){
            temp=temp.next;
            head=head.next;
            k--;
        }
        temp.next=null;
        return head;

    }
}
```

##### 反转链表
###### [反转链表](https://leetcode.cn/problems/reverse-linked-list/)
```java
class Solution {
    public ListNode reverseList(ListNode head) {
        ListNode res=new ListNode(-1);
        while(head!=null){
            ListNode temp=head.next;
            head.next=res.next;
            res.next=head;
            head=temp;
        }
        return res.next;
    }
}
```

方法2️⃣：递归的方式

```java
class Solution {
    public ListNode reverseList(ListNode head) {
        if(head==null||head.next==null){
         return head;
       }
      ListNode last=reverseList(head.next);
      head.next.next=head;
      head.next=null;
      return last;
    }
}
```

使用递归进行求解，首先考虑的是reverseList这个函数的作用，那就是进行反转，那么将reverseList(head.next)之后，链表情况

![](https://cdn.jsdelivr.net/gh/ddyycc123/imageloader@main/image-20231102165719950.png)

![](https://cdn.jsdelivr.net/gh/ddyycc123/imageloader@main/image-20231102165707289.png)

所以在这之后，只需要将head加入链表即可

```java
    head.next.next = head;
    head.next = null;
```

这个之后last的反转之后的根节点，返回last即可

###### [反转链表 II](https://leetcode.cn/problems/reverse-linked-list-ii/)
反转链表

思路：temp是一个完全空的链表，然后通过for循环，在left之前逐个插入，在left-right之间倒叙插入，然后再将right之后的直接插入

```java
class Solution {
    public ListNode reverseBetween(ListNode head, int left, int right) {
        ListNode res=new ListNode(-1);
        ListNode temp=res;
        for(int i=1;i<left;i++){
            temp.next=head;
            temp=temp.next;
            head=head.next;
        }
        ListNode node2=temp;
        for(int i=left;i<=right;i++){
            ListNode node=head.next;
            head.next=temp.next;
            temp.next=head;
            head=node;
        }
        for(int i=left;i<=right;i++)
            temp=temp.next;
        temp.next=head;
        return res.next;
    }
}
```

###### [两两交换链表中的节点](https://leetcode.cn/problems/swap-nodes-in-pairs/)
两两反转链表

思路：和下一题思路一样，首先用last指针判断接下来是否有2个，如果没有就直接顺序返回了，如果有的话，就进行反转，反转是在原链表上进行反转

```java
/**
 * Definition for singly-linked list.
 * public class ListNode {
 *     int val;
 *     ListNode next;
 *     ListNode() {}
 *     ListNode(int val) { this.val = val; }
 *     ListNode(int val, ListNode next) { this.val = val; this.next = next; }
 * }
 */
class Solution {
    public ListNode swapPairs(ListNode head) {
        ListNode res=new ListNode(-1,head);
        ListNode temp=res;
        while(true){
            ListNode last=temp;
            for(int i=0;i<2;i++){
                last=last.next;
                if(last==null)
                    return res.next;
            }
            ListNode cur=temp.next;
            ListNode node=cur.next;
            cur.next=node.next;
            node.next=cur;
            temp.next=node;
            temp=cur;
        }
    }
}
```

###### [K 个一组翻转链表](https://leetcode.cn/problems/reverse-nodes-in-k-group/)
k个一组反转链表

```java
class Solution {
    public ListNode reverseKGroup(ListNode head, int k) {
        ListNode res=new ListNode(-1,head);
        ListNode temp=res;
        while(true){
            ListNode last=temp;
            for(int i=0;i<k;i++){
                last=last.next;
                if(last==null){
                    return res.next;
                }
            }
            ListNode cur=temp.next,node;
            for(int i=0;i<k-1;i++){
                node=cur.next;
                cur.next=node.next;
                node.next=temp.next;
                temp.next=node;
            }
            temp=cur;
        }
    }
}
```

之前反转链表的时候，因为是将一个链表插到另一个链表上，可以采用插的方式是

```java
            ListNode node=head.next;
            head.next=temp.next;
            temp.next=head;
            head=node;
```

这样不会产生循环，如果在同一条链表上进行反转插值，这样会产生循环，因为没有对head的前一个指针进行指向的更改，所以对于同一条链表的反转可以使用下面的方式进行插值

```java
                node=cur.next;
                cur.next=node.next;
                node.next=temp.next;
                temp.next=node;
```

##### 回文链表
###### [回文链表](https://leetcode.cn/problems/palindrome-linked-list/)
回文链表是指正序和逆序遍历链表得到的序列完全相同。也就是说，如果一个链表从头到尾和从尾到头遍历的节点值都一样，那么这个链表就是回文链表。

解法1️⃣

使用栈的方式

```java
class Solution {
    public Boolean isPalindrome(ListNode head) {
        Stack<Integer>stack=new Stack<>();
        ListNode temp=head;
        while(temp!=null){
            stack.push(temp.val);
            temp=temp.next;
        }
        while(head!=null){
            if(head.val!=stack.pop()){
                return false;
            }
            head=head.next;
        }
        return true;
    }
};
```

解法2️⃣

使用双链表的方式

```java
    public static boolean isPalindrome(ListNode head) {
        if (head == null || head.next == null) {
            // 空链表或只有一个节点时为回文链表
            return true;
        }
        // 遍历链表并将节点值存储在列表中
        List<Integer> values = new ArrayList<>();
        ListNode current = head;
        while (current != null) {
            values.add(current.val);
            current = current.next;
        }
        // 使用双指针进行比较
        int left = 0;
        int right = values.size() - 1;
        while (left < right) {
            if (!values.get(left).equals(values.get(right))) {
                return false;
            }
            left++;
            right--;
        }
        return true;
    }
```

### 数组
###### [比较版本号](https://leetcode.cn/problems/compare-version-numbers/)
给你两个 版本号字符串 `version1` 和 `version2` ，请你比较它们。版本号由被点 `'.'` 分开的修订号组成。修订号的值 是它 转换为整数 并忽略前导零。

比较版本号时，请按 从左到右的顺序 依次比较它们的修订号。如果其中一个版本字符串的修订号较少，则将缺失的修订号视为 `0`。

```java
class Solution {
    public int compareVersion(String version1, String version2) {
        String[] v1 = version1.split("\\.");
        String[] v2 = version2.split("\\.");
        for (int i = 0; i < v1.length || i < v2.length; ++i) {
            int x = 0, y = 0;
            if (i < v1.length) {
                x = Integer.parseInt(v1[i]);
            }
            if (i < v2.length) {
                y = Integer.parseInt(v2[i]);
            }
            if (x > y) {
                return 1;
            }
            if (x < y) {
                return -1;
            }
        }
        return 0;
    }
}

```

###### [直角三角形](https://leetcode.cn/problems/right-triangles/)
首先统计出每行每列1的个数，因为这里直角三角形是可以不相邻的，所以直接相乘即可

```java
class Solution {
    public long numberOfRightTriangles(int[][] grid) {
        int m=grid.length;
        int n=grid[0].length;
        long count=0;
        int []rows=new int[m];
        int []columns=new int[n];
        for(int i=0;i<m;i++){
            for(int j=0;j<n;j++){
                if(grid[i][j]==1){
                    rows[i]++;
                    columns[j]++;
                }
            }
        }
        for(int i=0;i<m;i++){
            for(int j=0;j<n;j++){
                if(grid[i][j]==1){
                    count+=(long)(rows[i]-1)*(columns[j]-1);
                }
            }
        }
        return count;
    }
}
```

###### [文本左右对齐](https://leetcode.cn/problems/text-justification/)
文本对齐

题意是将数组分成若干行，每行最大长度为maxWidth，思路是首先用while循环判断当前行最多能放多少个单词，记录字符数量，单词数量，然后分为三种情况

+ 如果单词用完了，即最后一行，则这一行的单词之间空格只有一个空格，后面补空格
+ 如果没用完，但这一行只放了一个单词，那么就后面补空格
+ 如果没有完，且这一行有多个单词，就需要计算总空格数量，然后对单词数量求平均，算出平均每个单词之间的空格avg，这里不一定能除尽，所以还需要求余k，接着前k个单词间隙就是avg+1，剩余的才是k

```java
class Solution {
    public List<String> fullJustify(String[] words, int maxWidth) {
        List<String>res=new LinkedList<>();
        int n=words.length,right=0;
        while(true){
            int left=right;
            int sumLen=0;
            //这里right是不可达状态，还要加right-left是因为单词间至少要留一个空格
            while(right<n&&sumLen+words[right].length()+right-left<=maxWidth){
                sumLen+=words[right++].length();
            }
            //如果最后一行
            if(right==n){
                StringBuffer sb=join(words,left,n," ");
                sb.append(blank(maxWidth-sb.length()));
                res.add(sb.toString());
                return res;
            }
            int numWords=right-left;
            int space=maxWidth-sumLen;
            //如果只有一个单词
            if(numWords==1){
                StringBuffer sb=new StringBuffer(words[left]);
                sb.append(blank(space));
                res.add(sb.toString());
                continue;
            }
            //有多个单词
            int avgSpace=space/(numWords-1);
            int extra=space%(numWords-1);
            StringBuffer sb=new StringBuffer();
            sb.append(join(words,left,left+extra+1,blank(avgSpace+1)));
            sb.append(blank(avgSpace));
            sb.append(join(words,left+extra+1,right,blank(avgSpace)));
            res.add(sb.toString());
        

        }
    }
    //返回n个空格的字符串
    public String blank(int n){
        StringBuffer sb=new StringBuffer();
        for(int i=0;i<n;i++){
            sb.append(' ');
        }
        return sb.toString();
    }
    //对left到right之间的单词进行拼接，间隙是seq
    public StringBuffer join(String[]words,int left,int right,String seq){
        StringBuffer sb=new StringBuffer(words[left++]);
        while(left<right){
            sb.append(seq);
            sb.append(words[left++]);
        }
        return sb;

    }

}
```

###### k个偶数积
![](https://cdn.jsdelivr.net/gh/ddyycc123/imageloader@main/image-20240508162318691.png)

思路：考虑到只有奇数和奇数相乘才为奇数，4所以可以倒过来先求奇数的数量放到最前面，再将剩余元素按顺序排上去

```java
import java.util.Scanner;

public class Main {
    static final int N = 100007;
    static int n, k;
    static int[] a = new int[N];
    static boolean[] st = new boolean[N];
    public static void main(String[] args) {
        Scanner scanner = new Scanner(System.in);
        n = scanner.nextInt();
        k = scanner.nextInt();
        scanner.close();
        int od = n - 1 - k;
        if (od < 0) {
            System.out.println(-1);
            return;
        }
        int cnt = 0;
        for (int i = 1; i <= n; i += 2) {
            a[cnt++] = i;
            st[i] = true;
            //要放od+1个才可以
            if (cnt == od + 1) {
                break;
            }
        }
        //奇数不够
        if (cnt <= od) {
            System.out.println(-1);
            return;
        }
        //st[i]如果为false就可以按顺序直接填入
        for (int i = 1; i <= n; i++) {
            if (!st[i]) {
                a[cnt++] = i;
            }
        }
        for (int i = 0; i < cnt; i++) {
            System.out.print(a[i] + " ");
        }
    }
}

```

###### 求翻倍数组权值
![](https://cdn.jsdelivr.net/gh/ddyycc123/imageloader@main/image-20240508163405647-20240508163445479.png)

思路：最开始就求得数组的和，最大值，最小值和次小值，然后对每个元素翻倍之后进行判断，接着进行对应的极值计算

```java
import java.util.Scanner;

public class Main {
    static final int N = 100007;
    static int n;
    static long sum;
    static int[] a = new int[N];

    public static void main(String[] args) {
        Scanner scanner = new Scanner(System.in);
        n = scanner.nextInt();

        int maxa = 0;
        int mina = Integer.MAX_VALUE;
        int cmina = Integer.MAX_VALUE;

        for (int i = 0; i < n; i++) {
            a[i] = scanner.nextInt();
            sum += a[i];
            maxa = Math.max(maxa, a[i]);

            if (a[i] < mina) {
                cmina = mina;
                mina = a[i];
            } else if (a[i] < cmina) {
                cmina = a[i];
            }
        }
        scanner.close();
        if (n == 1) {
            System.out.println(0);
            return;
        }

        for (int i = 0; i < n; i++) {
            if (a[i] == maxa) {
                System.out.print((long) (2 * maxa - mina) * (sum + a[i]) + " ");
            } else if (a[i] == mina) {
                int mi = (2 * a[i] > cmina) ? cmina : 2 * a[i];
                int ma = (2 * a[i] > maxa) ? 2 * a[i] : maxa;
                System.out.print((long) (ma - mi) * (sum + a[i]) + " ");
            } else {
                int ma = (2 * a[i] > maxa) ? 2 * a[i] : maxa;
                System.out.print((long) (ma - mina) * (sum + a[i]) + " ");
            }
        }
    }
}
```

###### 求数组异或为0的子序列个数
![](https://cdn.jsdelivr.net/gh/ddyycc123/imageloader@main/image-20240508170842946.png)

思路：通过遍历法➕快速幂来进行求解

```java
import java.util.Scanner;

public class Main {
    static final int MOD = 1000000007;
    static long res = 0;
    static int[] num = new int[25];

    // 快速幂算法：计算 (a^k) % MOD
    static int qmi(int a, int k) {
        long result = 1;
        while (k != 0) {
            if ((k & 1) != 0) {
                result = result * a % MOD;
            }
            a = (int) ((long) a * a % MOD);
            k >>= 1;
        }
        return (int) result;
    }

    public static void main(String[] args) {
        Scanner scanner = new Scanner(System.in);
        int n = scanner.nextInt();

        // 读取数组数据
        for (int i = 1; i <= n; i++) {
            num[i] = scanner.nextInt();
        }
        scanner.close();

        // 枚举所有子集的组合
          //(1 << n)表示2^n个数，这是因为考虑每个数字要么算上要么不算，所以就两种状态，所以所有组合个数2^n
        for (int i = 0; i < (1 << n); i++) {
            int ans = 0;
            // 内层循环判断子集的元素并计算异或和
            for (int j = 0; j < n; j++) {
                  //判断元素在第j个位置是否为1，如果为1则说明使用了，就异或
                if ((i >> j & 1) != 0) {
                    ans ^= (j + 1);
                }
            }

            // 判断子集的异或和是否为0
            if (ans == 0) {
                long t = 1;
                // 计算该子集的组合数，因为每个位置的元素可能存在num[k] - 1个
                for (int k = 1; k <= n; k++) {
                    t = t * qmi(2, num[k] - 1) % MOD;
                }
                // 将结果累加到最终答案中
                res = (res + t) % MOD;
            }
        }

        // 输出最终答案
        System.out.println((res - 1 + MOD) % MOD);
    }
}
```

###### [最长有效括号](https://leetcode.cn/problems/longest-valid-parentheses/)
最长括号

```java
class Solution {
    public int longestValidParentheses(String s) {
        int maxans = 0;
        Deque<Integer> stack = new LinkedList<Integer>();
        stack.push(-1);
        for (int i = 0; i < s.length(); i++) {
            if (s.charAt(i) == '(') {
                stack.push(i);
            } else {
                stack.pop();
                if (stack.isEmpty()) {
                    stack.push(i);
                } else {
                    maxans = Math.max(maxans, i - stack.peek());
                }
            }
        }
        return maxans;
    }
}

```

###### [一手顺子](https://leetcode.cn/problems/hand-of-straights/)
```java
class Solution {
    public boolean isNStraightHand(int[] hand, int groupSize) {
        Arrays.sort(hand);
        Map<Integer,Integer>map=new HashMap<>();
        for(int n:hand)
            map.put(n,map.getOrDefault(n,0)+1);
        for(int i=0;i<hand.length;i++){
            if(map.get(hand[i])==0)
                continue;
            for(int j=0;j<groupSize;j++){
                if(map.get(hand[i]+j)==null||map.get(hand[i]+j)==0)
                    return false;
                map.put(hand[i]+j,map.getOrDefault(hand[i]+j,0)-1);
            }
        }
        return true;
    }
}
```

###### [矩阵置零](https://leetcode.cn/problems/set-matrix-zeroes/)
思路：首先遍历一边矩阵找到所有0元素的行和列，在分别对行列进行归零

```java
class Solution {
    public void setZeroes(int[][] matrix) {
        ArrayList<Integer>row=new ArrayList<>();
        ArrayList<Integer>colum=new ArrayList<>();
        for(int i=0;i<matrix.length;i++){
            for(int j=0;j<matrix[0].length;j++){
                if(matrix[i][j]==0){
                    row.add(i);
                    colum.add(j);
                }
            }
        }
        for(Integer r:row){
            for(int i=0;i<matrix[0].length;i++)
                matrix[r][i]=0;
        }
        for(Integer c:colum){
            for(int i=0;i<matrix.length;i++){
                matrix[i][c]=0;
            }
        }
        
    }
}
```

###### [轮转数组](https://leetcode.cn/problems/rotate-array/)
给定一个整数数组 `nums`，将数组中的元素向右轮转 `k` 个位置，其中 `k` 是非负数。

旋转数组

解法1️⃣：

```java
class Solution {
    public void rotate(int[] nums, int k) {
        int length=nums.length;
        k=k%length;
        ArrayList<Integer>arr=new ArrayList<>();
        for(int i=length-k;i<length;i++){
            arr.add(nums[i]);
        }
        for(int i=length-k-1;i>=0;i--){
            nums[i+k]=nums[i];
        }
        for(int i=0;i<k;i++)
            nums[i]=arr.get(i);
        
    }
}
```

解法2️⃣：思路参照下表

![](https://cdn.jsdelivr.net/gh/ddyycc123/imageloader@main/image-20240117222212255.png)

```java
class Solution {
    public void rotate(int[] nums, int k) {
        k=k%nums.length;
        reverse(nums,0,nums.length-1);
        reverse(nums,0,k-1);
        reverse(nums,k,nums.length-1);
    }
    public void reverse(int []nums,int start,int end){
        while(start<end){
            int temp=nums[start];
            nums[start]=nums[end];
            nums[end]=temp;
            start++;
            end--;
        }
    }
}
```

###### [生命游戏](https://leetcode.cn/problems/game-of-life/)
1为活细胞，0为死细胞，所有细胞都遵循一些生存定律

思路：就按照题目规则进行遍历技术，因为是同时发生，所以需要提前复制一份数组，用新数组判断，对旧数组进行改造

```java
class Solution {
    public void gameOfLife(int[][] board) {
        int [][]old=new int[board.length][board[0].length];
        int []neighbors={-1,0,1};
        for(int i=0;i<board.length;i++){
            for(int j=0;j<board[0].length;j++){
                old[i][j]=board[i][j];
            }
        }
        for(int row=0;row<board.length;row++){
            for(int col=0;col<board[0].length;col++){
                int count=0;
                for(int i=0;i<3;i++){
                    for(int j=0;j<3;j++){
                        if(!(i==1&&j==1)){
                            int r=row+neighbors[i];
                            int c=col+neighbors[j];
                            if((r<board.length&&r>=0)&&(c<board[0].length&&c>=0)&&old[r][c]==1){
                                count++;
                            }
                        }
                    }
                }
                if(old[row][col]==1&&(count<2||count>3)){
                    board[row][col]=0;
                }
                if(board[row][col]==0&&count==3)
                    board[row][col]=1;
            }
        }
    }
}
```

###### [逆波兰表达式求值](https://leetcode.cn/problems/evaluate-reverse-polish-notation/)
```java
class Solution {
    public int evalRPN(String[] tokens) {
        Deque<Integer> stack = new LinkedList<Integer>();
        int n = tokens.length;
        for (int i = 0; i < n; i++) {
            String token = tokens[i];
            if (isNumber(token)) {
                stack.push(Integer.parseInt(token));
            } else {
                int num2 = stack.pop();
                int num1 = stack.pop();
                switch (token) {
                    case "+":
                        stack.push(num1 + num2);
                        break;
                    case "-":
                        stack.push(num1 - num2);
                        break;
                    case "*":
                        stack.push(num1 * num2);
                        break;
                    case "/":
                        stack.push(num1 / num2);
                        break;
                    default:
                }
            }
        }
        return stack.pop();
    }

    public boolean isNumber(String token) {
        return !("+".equals(token) || "-".equals(token) || "*".equals(token) || "/".equals(token));
    }
}

```

###### [多数元素](https://leetcode.cn/problems/majority-element/)
给定一个大小为 `n` 的数组 `nums` ，返回其中的多数元素。多数元素是指在数组中出现次数 大于 `⌊ n/2 ⌋` 的元素。

你可以假设数组是非空的，并且给定的数组总是存在多数元素。

```java
class Solution {
    public int majorityElement(int[] nums) {
        HashMap<Integer,Integer>map=new HashMap<>();
        int n=nums.length/2;
        for(int i=0;i<nums.length;i++){
            map.put(nums[i],map.getOrDefault(nums[i],0)+1);
            if(map.get(nums[i])>n)
                return nums[i];
        }
        return -1;
    }
}
```

###### [H 指数](https://leetcode.cn/problems/h-index/)
思路：这个定义有点绕，思路是首先对citations进行排序，让引用的数从小到大，方便更快的算出每个h满足条件的引用的论文数量。接着就利用for循环挨个计算每个h值对应的引用大于它的论文的数量 然后对这个数组最后开始遍历，找到满足条件的h

这里需要注意的是nums数组的长度是citations.length+1，因为nums的索引i对应的是论文数量，所以需要相对应

```java
class Solution {
    public int hIndex(int[] citations) {
        Arrays.sort(citations);
        int []nums=new int[citations.length+1];
        for(int i=0;i<=citations.length;i++){
            int count=0;
            for(int j=0;j<citations.length;j++){
                if(citations[j]>=i){
                    nums[i]=citations.length-count;
                    break;
                }else{
                    count++;
                }
            }
        }
        for(int i=nums.length-1;i>=0;i--){
            if(nums[i]>=i)
                return i;
        }
        return 0;
    }
}
```

###### [O(1) 时间插入、删除和获取随机元素](https://leetcode.cn/problems/insert-delete-getrandom-o1/)
```java
class RandomizedSet {
    List<Integer> nums;
    Map<Integer, Integer> indices;
    Random random;

    public RandomizedSet() {
        nums = new ArrayList<Integer>();
        indices = new HashMap<Integer, Integer>();
        random = new Random();
    }

    public boolean insert(int val) {
        if (indices.containsKey(val)) {
            return false;
        }
        int index = nums.size();
        nums.add(val);
        indices.put(val, index);
        return true;
    }

    public boolean remove(int val) {
        if (!indices.containsKey(val)) {
            return false;
        }
        int index = indices.get(val);
        int last = nums.get(nums.size() - 1);
        nums.set(index, last);
        indices.put(last, index);
        nums.remove(nums.size() - 1);
        indices.remove(val);
        return true;
    }

    public int getRandom() {
        int randomIndex = random.nextInt(nums.size());
        return nums.get(randomIndex);
    }
}

```

###### [加油站](https://leetcode.cn/problems/gas-station/)
思路1️⃣：挨个判断，超时

```java
class Solution {
    public int canCompleteCircuit(int[] gas, int[] cost) {
        int n=gas.length;
        for(int i=0;i<n;i++){
            if(gas[i]<cost[i])
                continue;
            int cur=0;
            for(int j=0;j<n;j++){
                int k=(i+j)%n;
                cur+=gas[k]-cost[k];
                if(cur<0)
                    break;
            }
            if(cur>=0)
                return i;
        }
        return -1;
    }
}
```

思路2️⃣：剪枝，比如说从i点出发，到j的位置发现不能继续往下走了，那么这一段路的所有站点都是不行的，因为i这个位置至少是大于等于0的，i到j这一段路的点加了一个正数，到j这个位置都还是不能继续往下走，说明这一段的站点都是不行的 

虽然是两个循环，实际时间复杂度为O(N)

```java
class Solution {
    public int canCompleteCircuit(int[] gas, int[] cost) {
        int n=gas.length;
        int i=0;
        while(i<n){
            int cnt=0;
            int gasum=0;int costsum=0;
            while(cnt<n){
                int j=(i+cnt)%n;
                gasum+=gas[j];
                costsum+=cost[j];
                if(gasum<costsum)
                    break;
                cnt++;
            }
            if(cnt==n)
                return i;
            else
                i=i+cnt+1;
        }
        return -1;
    }
}
```

###### [罗马数字转整数](https://leetcode.cn/problems/roman-to-integer/)
罗马转数字

```java
class Solution {
    public int romanToInt(String s) {
        int num=0;
        for(int i=0;i<s.length();i++){
            if(s.charAt(i)=='I'){
                if(i<s.length()-1){
                    if(s.charAt(i+1)=='V'||s.charAt(i+1)=='X'){
                        num=num-2;
                    }
                }
                num+=1;
            }else if(s.charAt(i)=='V'){
                num+=5;
            }else if(s.charAt(i)=='X'){
                if(i<s.length()-1){
                    if(s.charAt(i+1)=='L'||s.charAt(i+1)=='C'){
                        num=num-20;
                    }
                }
                num+=10;
            }else if(s.charAt(i)=='L'){
                num+=50;
            }else if(s.charAt(i)=='C'){
                if(i<s.length()-1){
                    if(s.charAt(i+1)=='D'||s.charAt(i+1)=='M'){
                        num=num-200;
                    }
                }
                num+=100;
            }else if(s.charAt(i)=='D'){
                num+=500;
            }else if(s.charAt(i)=='M'){
                num+=1000;
            }
        }
        return num;
    }
}
```

###### [整数转罗马数字](https://leetcode.cn/problems/integer-to-roman/)
数字转罗马

思路：从num最大位数开始判断，逐渐减values中对应的位数，每减一次就StringBuffer append一次

```java
class Solution {
    int []values={1000,900,500,400,100,90,50,40,10,9,5,4,1};
    String []symbols={"M","CM","D","CD","C","XC","L","XL","X","IX","V","IV","I"};
    public String intToRoman(int num) {
        StringBuffer raman=new StringBuffer();
        for(int i=0;i<values.length;i++){
            int value=values[i];
            String symbol=symbols[i];
            while(num>=value){
                num-=value;
                raman.append(symbol);
            }
            if(num==0)
                break;
        }
        return raman.toString();
    }
}
```

###### [整数转换英文表示](https://leetcode.cn/problems/integer-to-english-words/)
数字转英文

```java
class Solution {
    String[] singles = {"", "One", "Two", "Three", "Four", "Five", "Six", "Seven", "Eight", "Nine"};
    String[] teens = {"Ten", "Eleven", "Twelve", "Thirteen", "Fourteen", "Fifteen", "Sixteen", "Seventeen", "Eighteen", "Nineteen"};
    String[] tens = {"", "Ten", "Twenty", "Thirty", "Forty", "Fifty", "Sixty", "Seventy", "Eighty", "Ninety"};
    String[] thousands = {"", "Thousand", "Million", "Billion"};

    public String numberToWords(int num) {
        if (num == 0) {
            return "Zero";
        }
        StringBuffer sb = new StringBuffer();
        //分成几个阶段，因为英文数字是三个数为一组
        for (int i = 3, unit = 1000000000; i >= 0; i--, unit /= 1000) {
            int curNum = num / unit;
            //说明在当前这个区间内有值
            if (curNum != 0) {
                num -= curNum * unit;
                StringBuffer curr = new StringBuffer();
                recursion(curr, curNum);
                curr.append(thousands[i]).append(" ");
                sb.append(curr);
            }
        }
        return sb.toString().trim();
    }

    public void recursion(StringBuffer curr, int num) {
        if (num == 0) {
            return;
        } else if (num < 10) {
            curr.append(singles[num]).append(" ");
        } else if (num < 20) {
            curr.append(teens[num - 10]).append(" ");
        } else if (num < 100) {
            curr.append(tens[num / 10]).append(" ");
            recursion(curr, num % 10);
        } else {
            curr.append(singles[num / 100]).append(" Hundred ");
            recursion(curr, num % 100);
        }
    }
}
```

###### [最后一个单词的长度](https://leetcode.cn/problems/length-of-last-word/)
```java
class Solution {
    public int lengthOfLastWord(String s) {
        String []res=s.split(" +");
        return res[res.length-1].length();
    }
}
```

###### [最长公共前缀](https://leetcode.cn/problems/longest-common-prefix/)
思路：先求第一个字符串的各个长度的前缀，再从大到小挨个取出与其他字符串进行比较

```java
class Solution {
    public String longestCommonPrefix(String[] strs) {
        String []prefix=new String[strs[0].length()];
        for(int i=0;i<strs[0].length();i++){
            prefix[i]=strs[0].substring(0,strs[0].length()-i);
        }
        for(int i=0;i<prefix.length;i++){
            String pre=prefix[i];
            boolean flag=true;
            for(int j=1;j<strs.length;j++){
                if(pre.length()>strs[j].length()||!pre.equals(strs[j].substring(0,pre.length()))){
                    flag=false;
                    break;
                }
            }
            if(flag==true){
                return pre;
            }
        }
        return "";
    }
}
```

###### [Z 字形变换](https://leetcode.cn/problems/zigzag-conversion/)
将一个给定字符串 `s` 根据给定的行数 `numRows` ，以从上往下、从左到右进行 Z 字形排列

思路：定义一个List< StringBuffer>rows数组，由题意可以看出，整个z字形，就是行号i从上到下，再从下到上的一个循环过程，所以就模拟i的这个过程，在期间不断append字符即可

```java
class Solution {
    public String convert(String s, int numRows) {
        if(numRows<2)
            return s;
        List<StringBuffer>rows=new ArrayList<>();
        for(int i=0;i<numRows;i++)
            rows.add(new StringBuffer());
        int i=0;int flag=-1;
        for(char c:s.toCharArray()){
            rows.get(i).append(c);
            if(i==0||i==numRows-1)
                flag=-flag;
            i+=flag;
        }
        StringBuffer res=new StringBuffer();
        for(int j=0;j<rows.size();j++)
            res.append(rows.get(j));
        return res.toString();
    }
}
```

###### [找出字符串中第一个匹配项的下标](https://leetcode.cn/problems/find-the-index-of-the-first-occurrence-in-a-string/)
给你两个字符串 `haystack` 和 `needle` ，请你在 `haystack` 字符串中找出 `needle` 字符串的第一个匹配项的下标（下标从 0 开始）。如果 `needle` 不是 `haystack` 的一部分，则返回  `-1` 。

```java
class Solution {
    public int strStr(String haystack, String needle) {
        int n=needle.length();
        for(int i=0;i<haystack.length();i++){
            if(i<=haystack.length()-n&&haystack.substring(i,i+n).equals(needle))
                return i;
        }
        return -1;
    }
}
```

###### 翻倍元素
思路：首先求一下每个元素要乘的倍数，接着求和即可。这道题的难点在于数字太大了，所以需要在进行每一步操作的时候都要%mod，并且不能使用math的pow方法，因为不能防止溢出，所以需要使用**快速幂**

```java
import java.util.*;
public class Solution {

    public static void main(String[] args) {
        int mod=1000000007;
        Scanner sc=new Scanner(System.in);
        int n=sc.nextInt();
        int q=sc.nextInt();
        int[]nums=new int[n];
        int[]tims=new int[n];
        for(int i=0;i<n;i++){
            nums[i]=sc.nextInt();
            tims[i]=q;
        }
        for(int i=0;i<q;i++){
            int x=sc.nextInt();
            tims[x-1]--;
        }
        long sum=0;
        for(int i=0;i<n;i++){
            long val=(long) nums[i]*pow(2,tims[i],mod)%mod;
            sum=(sum+val)%mod;
        }
        System.out.println(sum);
    }
      //快速幂的原理就是把n不断除2分解，减少运算量
    public static long pow(long x,int n,int mod){
        long result=1;
        x=x%mod;
        while(n>0){
              //判断是否为奇数
            if((n&1)==1){
                  //如果是奇数就先拿一个出来相乘
                result=x*result%mod;
            }
              //分解成两个小的
            x=(x*x)%mod;
              //n除以2
            n>>=1;
        }
        return result;
    }
}

```

快速幂

```java
public static long pow(long x,int n,int mod){
        long result=1;
        x=x%mod;
        while(n>0){
              //判断是否为奇数
            if((n&1)==1){
                  //如果是奇数就先拿一个出来相乘
                result=x*result%mod;
            }
              //分解成两个小的
            x=(x*x)%mod;
              //n除以2
            n>>=1;
        }
        return result;
    }
```

###### [腐烂的橘子](https://leetcode.cn/problems/rotting-oranges/)
思路：题目很简单，不过需要copy一个数组作为前一分钟的数组进行检查

```java
class Solution {
    public int orangesRotting(int[][] grid) {
        int count=0;
        int n=grid.length;
        int m=grid[0].length;
        while(true){
            boolean flag=change(grid);
              //flag用于判断是否还有更新
            if(flag==false)
                break;
            count++;
        }
      //判断是否还有新橘子
        for(int i=0;i<n;i++)
            for(int j=0;j<m;j++){
                if(grid[i][j]==1)
                    return -1;
            }
        return count;
    }
    public boolean change(int [][]grid){
        int n=grid.length;
        int m=grid[0].length;
        boolean flag=false;
        int [][]copy=new int[n][m];
        for (int i = 0; i < grid.length; i++) {
            copy[i] = Arrays.copyOf(grid[i], grid[i].length);
        }
        for(int i=0;i<n;i++){
            for(int j=0;j<m;j++){
                if(copy[i][j]==2){
                    if(i>0&&copy[i-1][j]==1){
                        grid[i-1][j]=2;
                        flag=true;
                    }
                    if(i<n-1&&copy[i+1][j]==1){
                        grid[i+1][j]=2;
                        flag=true;
                    }
                    if(j>0&&copy[i][j-1]==1){
                        grid[i][j-1]=2;
                        flag=true;
                    }
                    if(j<m-1&&copy[i][j+1]==1){
                        grid[i][j+1]=2;
                        flag=true;
                    }
                }
            }
        }
        return flag;
    }
}
```

###### 塔子哥打周赛
![](https://cdn.jsdelivr.net/gh/ddyycc123/imageloader@main/image-20240329152511582.png)

思路：很简单的一道题，就用有序队列存数据，不过要单独用一个变量存最大值

```java
import java.util.*;

public class Solution{
    public static void main(String[] args) {
        Scanner in=new Scanner(System.in);
        int n=in.nextInt();
        int m=in.nextInt();
        PriorityQueue<Integer>queue=new PriorityQueue<>(
                (a,b)->{
                    return a-b;
                }
        );
        int max=-1;
        for(int i=0;i<n;i++){
            int x=in.nextInt();
            queue.add(x);
            if(x>max)
                max=x;
        }
        for(int i=0;i<m;i++){
            int x=in.nextInt();
            int y=queue.poll();
            y+=x;
            if(y>max){
                max=y;
            }
            queue.add(y);
            System.out.println(max);
        }
    }
}
```

###### [缺失的第一个正数](https://leetcode.cn/problems/first-missing-positive/)
给你一个未排序的整数数组 `nums` ，请你找出其中没有出现的最小的正整数

第一个缺失的正数

思路1️⃣：如果能用hashmap就可以直接进行判断，但这道题不允许，就想着造一个hashmap来，数组长度为n，所以缺失的正值一定在1-n+1之间，正好对应了数组的大小（数组最多放了1-n，那么就是n+1），那现在要做的就是标记数组中1-n的数，将对应的0-n-1位置的值标为负数，这样再遍历哪个位置是正数，那么该位置+1就是缺失的正数

但要考虑一些情况，比如本来就是负数，对于这些数首先就要排出范围，即第一次遍历把数组中负数变成n+1，直接排除范围，第二遍再遍历数组，将数组中1-n的元素对应索引位置的元素标为负数，这样最后遍历的时候，看到这个索引位置的元素为负，就知道这个索引是存在的，通过这样的方式来找

```java
class Solution {
    public int firstMissingPositive(int[] nums) {
        int n=nums.length;
        for(int i=0;i<n;i++)
            if(nums[i]<=0)
                nums[i]=n+1;
        for(int i=0;i<n;i++){
              //考虑到可能前面的数标记负数的时候把后面的数也标记为了负数，这样后面的数就进不了if，对应的这些数对应的索引位置元素也不能标为负，就会导致结果错误，所以要用abs，因为只要在1-n范围内，用abs一样可以操作
            int k=Math.abs(nums[i]);
            if(k>=1&&k<=n){
                if(nums[k-1]>0)
                    nums[k-1]=-nums[k-1];
                
            }
        }
        for(int i=0;i<n;i++){
            if(nums[i]>0){
                return i+1;
            }
        }
          //如果没找到，说明数组中刚好存放的1-n，就返回n+1
        return n+1;
    }
}
```

思路2️⃣：使用置换的思路，即将数组中值为1-n的元素放到其应该在的位置，不断的置换。然后再遍历数组，找到元素不对应的索引，不过要注意的是因为有重复，所以要进行判断，不然会造成死循环

```java
class Solution {
    public int firstMissingPositive(int[] nums) {
        int n=nums.length;
        for(int i=0;i<n;i++){
            while(nums[i]>0&&nums[i]<=n&&nums[i]!=nums[nums[i]-1]){
                int temp=nums[i];
                nums[i]=nums[temp-1];
                nums[temp-1]=temp;
            }
        }
        for(int i=0;i<n;i++){
            if(nums[i]!=i+1)
                return i+1;
        }
        return n+1;
    }
}
```

###### [数组中重复的数据](https://leetcode.cn/problems/find-all-duplicates-in-an-array/)
给你一个长度为 `n` 的整数数组 `nums` ，其中 `nums` 的所有整数都在范围 `[1, n]` 内，且每个整数出现 一次 或 两次 。请你找出所有出现 两次 的整数，并以数组形式返回

找到所有出现两次的数进行返回

重复数

```java
class Solution {
    public List<Integer> findDuplicates(int[] nums) {
        int n = nums.length;
        for (int i = 0; i < n; ++i) {
            while (nums[i] != nums[nums[i] - 1]) {
                swap(nums, i, nums[i] - 1);
            }
        }
        List<Integer> ans = new ArrayList<Integer>();
        for (int i = 0; i < n; ++i) {
            if (nums[i] - 1 != i) {
                ans.add(nums[i]);
            }
        }
        return ans;
    }

    public void swap(int[] nums, int index1, int index2) {
        int temp = nums[index1];
        nums[index1] = nums[index2];
        nums[index2] = temp;
    }
}

```

###### **变换字符串**
![](https://cdn.jsdelivr.net/gh/ddyycc123/imageloader@main/image-20240416212044357.png)

思路：用hashmap记录每个字符应该编程哪一个，或者用一个字符串也可以

```java
import java.util.Scanner;

public class Main {
    public static void main(String[] args) {
        Scanner scanner = new Scanner(System.in);
        String t = "red";
        String s = scanner.nextLine();  // 读取一行输入
        
        StringBuilder result = new StringBuilder();  // 创建一个可变字符串来构建结果
        for (char i : s.toCharArray()) {  // 遍历输入字符串的每个字符
            int index = t.indexOf(i);  // 查找字符在 t 中的索引
            if (index != -1) {  // 如果字符存在于 t 中
                result.append(t.charAt((index + 1) % 3));  // 将其替换为 t 中下一个字符
            } else {
                result.append(i);  // 如果不在 t 中，保持原样（可以根据实际需求调整行为）
            }
        }
        
        System.out.println(result.toString());  // 输出结果字符串
    }
}

```



##### 树状数组
```java
import java.util.Arrays;

public class Solution {
    public static void main(String[] args) {
        int []record={7,5,6,4};
        reversePairs(record);

    }
    public static int reversePairs(int[] record) {
        int n = record.length;
        int[] tmp = new int[n];
        System.arraycopy(record, 0, tmp, 0, n);
        // 离散化
        Arrays.sort(tmp);
        for (int i = 0; i < n; ++i) {
            record[i] = Arrays.binarySearch(tmp, record[i]) + 1;
        }
        // 树状数组统计逆序对
        BIT bit = new BIT(n);
        int ans = 0;
        for (int i = n - 1; i >= 0; --i) {
            ans += bit.query(record[i] - 1);
            bit.update(record[i]);
        }
        return ans;
    }
}

class BIT {
    private int[] tree;
    private int n;

    public BIT(int n) {
        this.n = n;
        this.tree = new int[n + 1];
    }

    public static int lowbit(int x) {
        return x & (-x);
    }

    public int query(int x) {
        int ret = 0;
        while (x != 0) {
            ret += tree[x];
            x -= lowbit(x);
        }
        return ret;
    }

    public void update(int x) {
        while (x <= n) {
            ++tree[x];
            x += lowbit(x);
        }
    }
}


```

###### 小美的众数
![](https://cdn.jsdelivr.net/gh/ddyycc123/imageloader@main/image-20240317220826385.png)

由于它的数值只有1和2，所以记录一个前缀和。如果说当前值是1就前缀和加 1，如果说当前值是2，就前缀和减 1，对于一个前缀和sum[i]来说，他前面有多少个sum[j]的值比sum[i]小，就说明，1的个数会大于等于2的个数，那么这些数量的区间，他的众数肯定是一。i减去这些区间的数量，也就是剩下的区间数量。就是众数为 2 了

```java
import java.util.Scanner;

public class Main {
    static final int N = 200010;
    static int n;
    static int[] a = new int[N];
    static int[] tr = new int[N * 2];
    static int[] sum = new int[N];

    static int lowbit(int x) {
        return x & -x;
    }

    static void modify(int x, int k) {
        for (int i = x; i <= 2 * n + 5; i += lowbit(i)) tr[i] += k;
    }

    static int query(int x) {
        int res = 0;
        for (int i = x; i > 0; i -= lowbit(i)) res += tr[i];
        return res;
    }

    public static void main(String[] args) {
        Scanner scanner = new Scanner(System.in);
        n = scanner.nextInt();
        for (int i = 1; i <= n; i++) a[i] = scanner.nextInt();
        long ans = 0;
        modify(0 + n + 1, 1);
        for (int i = 1; i <= n; i++) {
            sum[i] = sum[i - 1];
            if (a[i] == 1) sum[i] += 1;
            else sum[i] -= 1;
            ans += query(sum[i] + n + 1) + (i - query(sum[i] + n + 1)) * 2;
            modify(sum[i] + n + 1, 1);
        }
        System.out.println(ans);
        scanner.close();
    }
}

```

###### 小美的逆序对
![](https://cdn.jsdelivr.net/gh/ddyycc123/imageloader@main/image-20240317220754217.png)

```java
import java.util.Scanner;

public class Main {
    static final int N = 200010;
    static int n;
    static int[] a = new int[N];
    static int[] tr = new int[N];
    static long[] sum = new long[N];
    static long[] f = new long[N];

    static int lowbit(int x) {
        return x & -x;
    }

    static void modify(int x, int k) {
        for (int i = x; i <= n; i += lowbit(i)) tr[i] += k;
    }

    static int query(int x) {
        int res = 0;
        for (int i = x; i > 0; i -= lowbit(i)) res += tr[i];
        return res;
    }

    public static void main(String[] args) {
        Scanner scanner = new Scanner(System.in);
        n = scanner.nextInt();
        for (int i = 1; i <= n; i++) a[i] = scanner.nextInt();
        long ans = 0;
        for (int i = 1; i <= n; i++) {
            modify(a[i], 1);
            sum[i] = sum[i - 1] + i - query(a[i]);
        }
        for (int i = 0; i < tr.length; i++) tr[i] = 0; // Reset the tr array
        for (int i = n; i >= 1; i--) {
            f[i] = sum[i - 1] + sum[n] - sum[i] + i - 1 - query(a[i]);
            modify(a[i], 1);
        }
        for (int i = 1; i <= n; i++)
            System.out.print(f[i] + " ");
        System.out.println();
        scanner.close();
    }
}

```

##### 逆序对
###### [交易逆序对的总数](https://leetcode.cn/problems/shu-zu-zhong-de-ni-xu-dui-lcof/)
在股票交易中，如果前一天的股价高于后一天的股价，则可以认为存在一个「交易逆序对」。请设计一个程序，输入一段时间内的股票交易记录 `record`，返回其中存在的「交易逆序对」总数。

思路：寻找逆序对，使用归并排序的思路，考虑归并排序的过程，在两段数组中挨个挑较小的元素进行合并，在这个过程中，如果左边i位置的元素大于右边j位置的元素，那么左边数组i及其之后的元素都对j位置元素形成逆序对，通过这样的方式在归并排序的过程中不断找逆序对

```java
class Solution {
    public int reversePairs(int[] record) {
        int n=record.length;
        if(n<2)
            return 0;
        int []nums=new int[n];
        for(int i=0;i<n;i++)
            nums[i]=record[i];
        int []temp=new int[n];
        return mergesort(nums,0,n-1,temp);
    }

    public int mergesort(int []record,int left,int right,int []temp){
        if(left==right)
            return 0;
        int mid=left+(right-left)/2;
        int leftres=mergesort(record,left,mid,temp);
        int rightres=mergesort(record,mid+1,right,temp);
        if(record[mid]<=record[mid+1])
            return leftres+rightres;
        int mergesum=merge(record,left,mid,right,temp);
        return leftres+rightres+mergesum;
    }

    public int merge(int []record,int left,int mid,int right,int []temp){
        for(int i=left;i<=right;i++){
            temp[i]=record[i];
        }
        int i=left,j=mid+1;
        int count=0;
        for(int k=left;k<=right;k++){
            if(i==mid+1){
                record[k]=temp[j];
                j++;
            }else if(j==right+1){
                record[k]=temp[i];
                i++;
            }else if(temp[i]<=temp[j]){
                record[k]=temp[i];
                i++;
            }else if(temp[i]>temp[j]){
                record[k]=temp[j];
                j++;
                count+=mid-i+1;
            }
        }
        return count;
    }
}
```

###### [计算右侧小于当前元素的个数](https://leetcode.cn/problems/count-of-smaller-numbers-after-self/)
给你一个整数数组 `nums` ，按要求返回一个新数组 `counts` 。数组 `counts` 有该性质： `counts[i]` 的值是  `nums[i]` 右侧小于 `nums[i]` 的元素的数量。

思路：和上题思路大致相同，区别在于在合并的时候，因为要求每一个元素右边的值，所以不能像上一题那样当左边的更大来直接计算左边有多少个，应该反过来，对于左边每个元素看他小于右边元素时，那他是大于右边当前索引以左的元素

```java
import java.util.ArrayList;
import java.util.List;

class Solution {
    private static int[] index;
    private static int[] temp;
    private static int[] tempIndex;
    private static int[] ans;

    public static List<Integer> countSmaller(int[] nums) {
        index = new int[nums.length];
        temp = new int[nums.length];
        tempIndex = new int[nums.length];
        ans = new int[nums.length];
        for (int i = 0; i < nums.length; ++i) {
            index[i] = i;
        }
        int l = 0, r = nums.length - 1;
        mergeSort(nums, l, r);
        List<Integer> list = new ArrayList<Integer>();
        for (int num : ans) {
            list.add(num);
        }
        return list;
    }

    public static void mergeSort(int[] a, int l, int r) {
        if (l >= r) {
            return;
        }
        int mid = (l + r) >> 1;
        mergeSort(a, l, mid);
        mergeSort(a, mid + 1, r);
        merge(a, l, mid, r);
    }

    public static void merge(int[] a, int l, int mid, int r) {
        for (int k = l; k <= r; ++k) {
            temp[k]=a[k];
            tempIndex[k]=index[k];
        }
        int i = l, j = mid + 1;
        for(int p=l;p<=r;p++){
            if(i==mid+1){
                a[p] = temp[j];
                index[p] = tempIndex[j];
                ++j;
            }else if(j==r+1){
                a[p] = temp[i];
                index[p] = tempIndex[i];
                ans[tempIndex[i]] += (j - mid - 1);
                ++i;

            }else if(temp[i]>temp[j]){
                a[p] = temp[j];
                index[p] = tempIndex[j];
                ++j;

            }else if(temp[i]<=temp[j]){
                a[p] = temp[i];
                index[p] = tempIndex[i];
                ans[tempIndex[i]] += (j - mid - 1);
                ++i;
            }
        }

    }
}

```

###### [每日温度](https://leetcode.cn/problems/daily-temperatures/)
给定一个整数数组 `temperatures` ，表示每天的温度，返回一个数组 `answer` ，其中 `answer[i]` 是指对于第 `i` 天，下一个更高温度出现在几天后。如果气温在这之后都不会升高，请在该位置用 `0` 来代替。

思路：寻找每个位置最近逆序对可以考虑使用栈来处理，栈里存的都是越来越大的值，栈低存的是最大值，如果栈底为空，说明当前值是目前的最大值

```java
class Solution {
    public int[] dailyTemperatures(int[] temperatures) {
        Stack<Integer>stack=new Stack<>();
        int []res=new int[temperatures.length];
        for(int i=temperatures.length-1;i>=0;i--){
            while(!stack.isEmpty()){
                int index=stack.peek();
                if(temperatures[i]<temperatures[index]){
                    res[i]=index-i;
                    stack.push(i);
                    break;
                }
                stack.pop();
            }
            if(stack.isEmpty()){
                res[i]=0;
                stack.push(i);
            }
        }
        return res;
    }
}
```

###### 求左边第一个小于的数字
```java
import java.util.Stack;

public class LeftFirstSmaller {
    public static int[] findLeftFirstSmaller(int[] nums) {
        int[] result = new int[nums.length];  // 用于存储结果
        Stack<Integer> stack = new Stack<>(); // 用于存储索引

        for (int i = 0; i < nums.length; i++) {
            // 弹出所有大于等于当前元素的栈顶元素
            while (!stack.isEmpty() && nums[stack.peek()] >= nums[i]) {
                stack.pop();
            }

            // 如果栈为空，说明左边没有比当前元素小的数字
            if (stack.isEmpty()) {
                result[i] = -1; // 使用 -1 表示不存在
            } else {
                result[i] = nums[stack.peek()]; // 栈顶元素即为左边第一个小于当前元素的数字
            }

            // 将当前元素的索引压入栈
            stack.push(i);
        }

        return result;
    }

    public static void main(String[] args) {
        int[] nums = {3, 7, 8, 4, 2, 6, 5};
        int[] result = findLeftFirstSmaller(nums);

        // 输出结果
        for (int res : result) {
            System.out.print(res + " ");
        }
    }
}
```

##### TopK
###### [奖励最顶尖的 K 名学生](https://leetcode.cn/problems/reward-top-k-students/)
给你两个字符串数组 `positive_feedback` 和 `negative_feedback` ，分别包含表示正面的和负面的词汇。不会 有单词同时是正面的和负面的。

一开始，每位学生分数为 `0` 。每个正面的单词会给学生的分数 加 `3` 分，每个负面的词会给学生的分数 减  `1` 分。

给你 `n` 个学生的评语，用一个下标从 0 开始的字符串数组 `report` 和一个下标从 0 开始的整数数组 `student_id` 表示，其中 `student_id[i]` 表示这名学生的 ID ，这名学生的评语是 `report[i]` 。每名学生的 ID 互不相同。

给你一个整数 `k` ，请你返回按照得分 从高到低 最顶尖的 `k` 名学生。如果有多名学生分数相同，ID 越小排名越前。

思路1️⃣：先使用hashmap存储每个学生的id和分数，再放入PriorityQueue进行排序，再放入list中，这么做的原因是因为刚开始以为report可能几个索引对应一个student_id，即student_id中可能含有相同的id 但实际是student_id含有的是n个学生各自的id

```java
import java.util.*;

class Student{
    int id;
    int score;
    Student(int id,int score){
        this.id=id;
        this.score=score;
    }
}

class Solution {
    public List<Integer> topStudents(String[] positive_feedback, String[] negative_feedback, String[] report, int[] student_id, int k) {
        PriorityQueue<Student> student=new PriorityQueue<>(
                (a,b)->{
                    if(a.score==b.score)
                        return a.id- b.id;
                    else
                        return b.score-a.score;
                }
        );
        HashMap<Integer,Integer>map=new HashMap<>();
        HashSet<String>positive=new HashSet<>();
        HashSet<String>negative=new HashSet<>();
        for(String s:positive_feedback)
            positive.add(s);
        for(String s:negative_feedback)
            negative.add(s);
        for(int i=0;i<report.length;i++){
            String []p=report[i].trim().split(" ");
            for(String s:p){
                if(negative.contains(s))
                    map.put(student_id[i],map.getOrDefault(student_id[i],0)-1);
                else if(positive.contains(s))
                    map.put(student_id[i],map.getOrDefault(student_id[i],0)+3);
            }
        }
        for(int i=0;i<student_id.length;i++){
            if(map.containsKey(student_id[i])){
                student.add(new Student(student_id[i],map.get(student_id[i])));
            }else{
                student.add(new Student(student_id[i],0));
            }
            
        }
        List<Integer>list=new LinkedList<>();
        while(!student.isEmpty()&&k>0){
            list.add(student.poll().id);
            k--;
        }
        return list;
    }
}
```

思路2️⃣：因此其实是不需要hashmap的，直接PriorityQueue就够了，每一轮add一个student类即可，或者也可以直接对数组进行排序

```java
import java.util.*;


class Solution {
    public List<Integer> topStudents(String[] positive_feedback, String[] negative_feedback, String[] report, int[] student_id, int k) {
        int n=report.length;
        int ans[][]=new int[n][2];
        HashSet<String>positive=new HashSet<>();
        HashSet<String>negative=new HashSet<>();
        for(String s:positive_feedback)
            positive.add(s);
        for(String s:negative_feedback)
            negative.add(s);
        for(int i=0;i<report.length;i++){
            String []p=report[i].trim().split(" ");
            int score=0;
            for(String s:p){
                if(negative.contains(s))
                    score-=1;
                else if(positive.contains(s))
                    score+=3;
            }
            ans[i]=new int[]{student_id[i],score};
        }
        Arrays.sort(ans,
                    (a,b)->a[1]==b[1]?a[0]-b[0]:b[1]-a[1]
                   );
        List<Integer>list=new LinkedList<>();
        for(int i=0;i<k;i++){
            list.add(ans[i][0]);
        }
        return list;
    }
}
```

###### [前K个高频单词](https://leetcode.cn/problems/top-k-frequent-words/)
给定一个单词列表 `words` 和一个整数 `k` ，返回前 `k` 个出现次数最多的单词。

返回的答案应该按单词出现频率由高到低排序。如果不同的单词有相同出现频率， 按字典顺序 排序

相同思路

两个字符串按字典顺序排序的函数是a.word.compareTo(b.word)

```java
import java.util.*;

class Word{
    String word;
    int count;
    Word(String word,int count){
        this.word=word;
        this.count=count;
    }
}
class Solution {
    public List<String> topKFrequent(String[] words, int k) {
        PriorityQueue<Word>queue=new PriorityQueue<>(
                (a,b)->{
                    if(a.count== b.count)
                        return a.word.compareTo(b.word);
                    else
                        return b.count-a.count;
                }
        );
        HashMap<String,Integer>map=new HashMap<>();
        for(String w:words){
            map.put(w,map.getOrDefault(w,0)+1);
        }
        for(Map.Entry<String,Integer>entry: map.entrySet()){
            queue.add(new Word(entry.getKey(),entry.getValue()));
        }
        List<String>list=new LinkedList<>();
        while(!queue.isEmpty()&&k>0){
            list.add(queue.poll().word);
            k--;
        }
        return list;
    }
}
```

###### [前 K 个高频元素](https://leetcode.cn/problems/top-k-frequent-elements/)
```java
class Element{
    Integer index;
    Integer value;
    Element(int index,int value){
        this.index=index;
        this.value=value;
    }
}
class Solution {
    public int[] topKFrequent(int[] nums, int k) {
        HashMap<Integer,Integer>map=new HashMap<>();
        for(int n:nums){
            map.put(n,map.getOrDefault(n,0)+1);
        }
        PriorityQueue<Element>queue=new PriorityQueue<>(
            (a,b)->{
                return b.value-a.value;
            }
        );
        for(Map.Entry<Integer,Integer>entry:map.entrySet()){
            queue.add(new Element(entry.getKey(),entry.getValue()));
        }
        int []res=new int[k];
        int i=0;
        while(!queue.isEmpty()&&k>0){
            res[i]=queue.poll().index;
            i++;
            k--;
        }
        return res;
    }
}
```

###### [数组中的第K个最大元素](https://leetcode.cn/problems/kth-largest-element-in-an-array/)
思路：使用快速选择法，因为整个排序求的是升序的数组，而要求的是第k的元素，所以传入的时候是n-k

时间复杂度O(N)

```java
class Solution {
    public int findKthLargest(int[] nums, int k) {
        return quickSelect(nums,0,nums.length-1,nums.length-k);
    }
    public int quickSelect(int []nums,int l,int r,int k){
        if(l==r)
            return nums[k];
        int x=nums[l],i=l-1,j=r+1;
        while(i<j){
            do i++;
            while(nums[i]<x);
            do j--;
            while(nums[j]>x);
            if(i<j){
                int temp=nums[i];
                nums[i]=nums[j];
                nums[j]=temp;
            }
        }
        if(k<=j)return quickSelect(nums,l,j,k);
        else return quickSelect(nums,j+1,r,k);
    }
}
```

快排代码  

快排

```java
class Solution {
    public int[] sortArray(int[] nums) {
        quickSort(nums, 0, nums.length - 1);
        return nums;
    }
     void quickSort(int[] nums, int l, int r) {
        if (l >= r) return;
        int x = nums[l], i = l - 1, j = r + 1;
        while (i < j) {
            do i++; while (nums[i] < x);
            do j--; while (nums[j] > x);
            if (i < j){
                int tmp = nums[i];
                nums[i] = nums[j];
                nums[j] = tmp;
            }
        }
        quickSort(nums, l, j);
        quickSort(nums, j + 1, r);
    }
}
```

###### [查找和最小的 K 对数字](https://leetcode.cn/problems/find-k-pairs-with-smallest-sums/)
给定两个以 非递减顺序排列 的整数数组 `nums1` 和 `nums2` , 以及一个整数 `k` 。

定义一对值 `(u,v)`，其中第一个元素来自 `nums1`，第二个元素来自 `nums2` 。

请找到和最小的 `k` 个数对 `(u1,v1)`, ` (u2,v2)`  ...  `(uk,vk)` 

思路：就是用有序队列来存，不过肯定不能全存，这样会超内存，所以只能存一部分，因为两个数组都是有序数组，所以(0,0)的时候肯定是最小的，那第二小的一定是他的下面或者右边

所以最开始的思路就是从队列中取出一个后，添加这一个下面和右边到有序队列中，但这又有一个问题，会重复，即有可能一个右边正好是之前添加的其他的下面，所以可以先添加第一列，然后只向右扩展

```java
class Node{
    int left;
    int right;
    Node(int l,int r){
        left=l;
        right=r;
    }
}
class Solution {
    public List<List<Integer>> kSmallestPairs(int[] nums1, int[] nums2, int k) {
        PriorityQueue<Node>queue=new PriorityQueue<>(
            (a,b)->{
                return (nums1[a.left]+nums2[a.right])-(nums1[b.left]+nums2[b.right]);
            }
        );
        List<List<Integer>>res=new LinkedList<>();
        for(int i=0;i<Math.min(nums1.length,k);i++)
            queue.add(new Node(i,0));
        while(k-->0&&!queue.isEmpty()){
            List<Integer>list=new LinkedList<>();
            Node node=queue.poll();
            list.add(nums1[node.left]);list.add(nums2[node.right]);
            res.add(list);
            //只向右扩展
            if(node.right+1<nums2.length)
                queue.add(new Node(node.left,node.right+1));
        }
        return res;
    }
}

```

###### [有序矩阵中第 K 小的元素](https://leetcode.cn/problems/kth-smallest-element-in-a-sorted-matrix/)
给你一个 `n x n` 矩阵 `matrix` ，其中每行和每列元素均按升序排序，找到矩阵中第 `k` 小的元素。  
请注意，它是 排序后 的第 `k` 小元素，而不是第k个不同的元素

和上题一样的思路可以做

```java
class Node{
    int left;
    int right;
    Node(int l,int r){
        left=l;
        right=r;
    }
}
class Solution {
    public int kthSmallest(int[][] matrix, int k) {
        PriorityQueue<Node>queue=new PriorityQueue<>(
            (a,b)->{
                return matrix[a.left][a.right]-matrix[b.left][b.right];
            }
        );
        for(int i=0;i<matrix.length&&i<k;i++)
            queue.add(new Node(i,0));
        while(k-->0&&!queue.isEmpty()){
            Node node=queue.poll();
            if(k==0)
                return matrix[node.left][node.right];
            if(node.right+1<matrix[0].length)
                queue.add(new Node(node.left,node.right+1));

        }
        return -1;
    }
}
```

这道题也可以用二分来做，思路是由于是二维有序数组，所以数组的最小值一定是左上角，最大值是右下角，所以可以找到一个mid值，然后从矩阵左下角开始计算小于等于mid值的元素数量，和k进行对比，如果数量更多的话，说明mid值比目标大了，反之mid值比目标小了

```java
class Solution {
    public int kthSmallest(int[][] matrix, int k) {
        int m=matrix.length;
        int n=matrix[0].length;
        int left=matrix[0][0],right=matrix[m-1][n-1];
        while(left<=right){
            int mid=left+(right-left)/2;
            if(check(matrix,k,mid))
                right=mid-1;
            else
                left=mid+1;
        }
        return left;
    }
    public boolean check(int [][]matrix,int k,int mid){
        int m=matrix.length-1;
        int n=0;
        int num=0;
        while(m>=0&&n<matrix[0].length){
            //因为是从左下角进行比较的，所以要是matrix[m][n]小的话，那上面的一列都是小的
            if(matrix[m][n]<=mid){
                num=num+m+1;
                n++;
            }else{
                m--;
            }
        }
        return num>=k;
    }
}
```

##### 堆排序
###### [IPO](https://leetcode.cn/problems/ipo/)
这道题的思路是成本是w，现在需要在满足成本w之下拿下k个利润最大的项目，并且拿下一个项目w是不用减去对应成本的

所以思路是首先将成本和利润对应放到一个数组中，按照成本由小到大对数组进行排序，定义一个有序队列用于承载当前成本下能拿下的项目，队列是按照利润由大到小进行排序的

因为数组是按成本从小到大进行排序的，所以定义一个指针cur一直对数组往后推进即可，接着for循环每次取当前成本满足下利润最高的一个项目，直到k结束

```java
class Solution {
    public int findMaximizedCapital(int k, int w, int[] profits, int[] capital) {
        int n=profits.length;
        int cur=0;
        int [][]arr=new int[n][2];
        for(int i=0;i<n;i++){ 
            arr[i][0]=capital[i];
            arr[i][1]=profits[i];
        }
        Arrays.sort(arr,(a,b)->a[0]-b[0]);
        PriorityQueue<Integer>queue=new PriorityQueue<>(
            (a,b)->{
                return b-a;
            }
        );
        for(int i=0;i<k;i++){
            while(cur<n&&arr[cur][0]<=w){
                queue.add(arr[cur][1]);
                cur++;
            }
            if(!queue.isEmpty()){
                w+=queue.poll();
            }else{
                break;
            }
        }
        return w;
    }
}
```

###### [数据流的中位数](https://leetcode.cn/problems/find-median-from-data-stream/)
中位数是有序整数列表中的中间值。如果列表的大小是偶数，则没有中间值，中位数是两个中间值的平均值

用两个有序队列来做，maxHeap放的是有序数组中更小的那一半，minHeap放的是有序数组中更大的那一半

思路是需要经过两个有序队列都过滤一遍，得到中间值，具体步骤是首先放到maxHeap中进行过滤，得到目前的最大值给minHeap进行过滤，如果maxHeap长度小于minHeap，就将minHeap的最小值给maxHeap

```java
class MedianFinder {

    PriorityQueue<Integer> maxHeap;
    PriorityQueue<Integer> minHeap;

    public MedianFinder() {
        maxHeap = new PriorityQueue<>(
            (a,b)->{
                return b-a;
            }
        );
        minHeap = new PriorityQueue<>();
    }
    
    public void addNum(int num) {
        maxHeap.offer(num);
        minHeap.offer(maxHeap.poll());

        if (maxHeap.size() < minHeap.size()) {
            maxHeap.offer(minHeap.poll());
        }
    }
    
    public double findMedian() {
        double median;
        if (maxHeap.size() > minHeap.size()) {
            median = maxHeap.peek();
        } else {
            median = (maxHeap.peek() + minHeap.peek()) / 2.0;
        }
        return median;
    }
}

```

###### <font style="color:rgb(89, 89, 89);">购房之旅</font>
![](https://cdn.nlark.com/yuque/0/2024/png/42839395/1722568220857-af1cb89f-20b2-450d-8eae-b7b8129d390b.png)

思路：因为是要求最大舒适度之和，所以可以使用贪心的思路，即对房子按照舒适度由大到小进行排序，对每个房子找到最接近能拿下它价格的朋友，一直遍历即可

提升时间的方式：对于找对应价格的朋友，可以使用二分查找来找，这样更快，这里本来是想用treeset的，但考虑到有可能有钱重复的朋友，所以用**TreeMap**来存储所有的金额，`TreeMap` 是通过红黑树实现的，然后使用**ceilingKey(key)**来找最接近目标的朋友，这个方法**是返回找到的最小的大于或等于key的键**，如果没有就返回null

```java
import java.util.*;

public class Solution {
    public static void main(String[] args) {
        Scanner sc=new Scanner(System.in);
        int m=sc.nextInt();
        int n=sc.nextInt();
        int res=0;
        TreeMap<Integer,Integer>money=new TreeMap<>();
        for(int i=0;i<m;i++){
            int k=sc.nextInt();
            money.put(k,money.getOrDefault(k,0)+1);
        }
        List<int[]>house=new LinkedList<>();
        for(int i=0;i<n;i++){
            int k=sc.nextInt();
            int j=sc.nextInt();
            house.add(new int[]{k,j});
        }
        Collections.sort(house,(a,b)->(b[0]-a[0]));
        for(int []h:house){
            int k=h[0];
            int j=h[1];
            Integer key=money.ceilingKey(j);
            if(key!=null){
                if(money.get(key)==1){
                    money.remove(key);
                }else{
                    money.put(key,money.get(key)-1);
                }
                res+=k;
                n--;
            }
            if(n==0)
                break;
        }
        System.out.println(res);
    }
}
```

###### 小美的密码
![](https://cdn.nlark.com/yuque/0/2024/png/42839395/1723343889094-fa3b9d0b-33e6-4cf4-9d77-013cf256dbd8.png)

思路1️⃣：使用hashmap来存储密码，然后对hashmap进行排序，这里hashmap的key是字符串长度，value是set集合，存储对应字符串长度下的字符串

```java
import java.util.*;

public class Main {
    public static void main(String[] args) {
        Scanner scanner=new Scanner(System.in);
        int n= scanner.nextInt();
        String res=scanner.next();
        HashMap<Integer, Set<String>>map=new HashMap<>();
        for(int i=0;i<n;i++){
            String p=scanner.next();
            map.computeIfAbsent(p.length(),k->new HashSet<>()).add(p);
        }
        List<Map.Entry<Integer,Set<String>>>sortedMap=new LinkedList<>(map.entrySet());
        Collections.sort(sortedMap,(a,b)->(a.getKey()-b.getKey()));
        int min=0,max=0;
        for(Map.Entry<Integer,Set<String>>m:sortedMap){
            Set<String>set=m.getValue();
            if(set.contains(res)){
                min=min+1;
                max=max+set.size();
                break;
            }else{
                min+=set.size();
                max+=set.size();
            }
        }
        System.out.println(min+" "+max);
    }
}
```

思路2️⃣：其实只需要知道小于密码长度的字符串数量和等于密码的字符串数量即可，即只需要一个set只加入大于长度的即可，并且在set加入的时候进行判断

```java
import java.util.*;

public class Main {
    public static void main(String[] args) {
        Scanner scanner=new Scanner(System.in);
        int n=scanner.nextInt();
        String res=scanner.next();
        Set<String>set=new HashSet<>();
        int shorter=0,equal=0;
        for(int i=0;i<n;i++){
            String p=scanner.next();
            if(p.length()<res.length()&&!set.contains(p))
                shorter++;
            else if(p.length()==res.length()&&!set.contains(p))
                equal++;
            set.add(p);
        }
        System.out.println((shorter+1)+" "+(shorter+equal));
    }
}
```

###### 小美的数组删除
![](https://cdn.nlark.com/yuque/0/2024/png/42839395/1723347690719-a57807b5-d8a0-4b57-bf7f-18376abe1831.png)

思路：先求从后向前求每个位置的最小未出现正整数，然后在从前往后求最小值

```java
import java.util.*;

public class Main {
    public static void main(String[] args) {
        Scanner sc=new Scanner(System.in);
        int N=sc.nextInt();
        while(N-->0){
            int n=sc.nextInt();
            int k=sc.nextInt();
            int x=sc.nextInt();
            int []arr=new int[n];
            for(int i=0;i<n;i++)
                arr[i]=sc.nextInt();
            int []prix=new int[n+1];
            Set<Integer>set=new HashSet<>();
            int index=0;
            for(int i=n-1;i>=0;i--){
                set.add(arr[i]);
                while(set.contains(index)){
                    index++;
                }
                prix[i]=index;
            }
            int res=k*prix[0];
            for(int i=1;i<=n;i++){
                res=Math.min(res,i*x+prix[i]*k);
            }
            System.out.println(res);
        }
    }
}

```

##### 字典树
###### [实现 Trie (前缀树)](https://leetcode.cn/problems/implement-trie-prefix-tree/)
```java
class Trie {
    
    private Trie[]children;
    private boolean isEnd;
    public Trie() {
        children=new Trie[26];
        isEnd=false;
    }
    
    public void insert(String word) {
        Trie node=this;
        for(int i=0;i<word.length();i++){
            char ch=word.charAt(i);
            int index=ch-'a';
            if(node.children[index]==null){
                node.children[index]=new Trie();
            }
            node=node.children[index];
        }
        node.isEnd=true;
    }
    
    public boolean search(String word) {
        Trie node=searchPrefix(word);
        return node!=null&&node.isEnd;
    }
    
    public boolean startsWith(String prefix) {
        return searchPrefix(prefix)!=null;
    }

    private Trie searchPrefix(String prefix){
        Trie node=this;
        for(int i=0;i<prefix.length();i++){
            char ch=prefix.charAt(i);
            int index=ch-'a';
            if(node.children[index]==null)
                return null;
            node=node.children[index];
        }
        return node;
    }
}
```

###### [添加与搜索单词 - 数据结构设计](https://leetcode.cn/problems/design-add-and-search-words-data-structure/)
```java
class WordDictionary {
    private WordDictionary[]items;
    boolean isEnd;

    public WordDictionary() {
        items=new WordDictionary[26];
    }
    
    public void addWord(String word) {
        WordDictionary curr=this;
        int n=word.length();
        for(int i=0;i<n;i++){
            int index=word.charAt(i)-'a';
            if(curr.items[index]==null)
                curr.items[index]=new WordDictionary();
            curr=curr.items[index];
        }
        curr.isEnd=true;
    }
    
    public boolean search(String word) {
        return search(this,word,0);
    }

    private boolean search(WordDictionary curr,String word,int start){
        int n=word.length();
        if(start==n)
            return curr.isEnd;
        char c=word.charAt(start);
        if(c!='.'){
            WordDictionary item=curr.items[c-'a'];
            return item!=null&&search(item,word,start+1);
        }
        for(int j=0;j<26;j++){
            if(curr.items[j]!=null&&search(curr.items[j],word,start+1))
                return true;
        }
        return false;
    }
}
```

##### 双指针
###### [删除最短的子数组使剩余数组有序](https://leetcode.cn/problems/shortest-subarray-to-be-removed-to-make-array-sorted/)
给你一个整数数组 `arr` ，请你删除一个子数组（可以为空），使得 `arr` 中剩下的元素是 非递减 的。

一个子数组指的是原数组中连续的一个子序列。

```java
class Solution {
    public int findLengthOfShortestSubarray(int[] arr) {
        int n = arr.length, j = n - 1;
        //首先找到数组右边的递增序列 j就是右边递增序列的头位置
        while (j > 0 && arr[j - 1] <= arr[j]) {
            j--;
        }
        if (j == 0) {
            return 0;
        }
        //res现在代表最多要去除前面j个
        int res = j;
        //左指针从0开始 
        for (int i = 0; i < n; i++) {
            //要保证中间删去后是递增的，就要保证i位置小于等于j位置
            //否则j++
            while (j < n && arr[j] < arr[i]) {
                j++;
            }
            res = Math.min(res, j - i - 1);
            //也需要保证左边是递增的，如果不递增就直接退出
            if (i + 1 < n && arr[i] > arr[i + 1]) {
                break;
            }
        }
        return res;
    }
}
```

###### [下一个排列](https://leetcode.cn/problems/next-permutation/)
找一个更大的排列

整数数组的 下一个排列 是指其整数的下一个字典序更大的排列。更正式地，如果数组的所有排列根据其字典顺序从小到大排列在一个容器中，那么数组的 下一个排列 就是在这个有序容器中排在它后面的那个排列。如果不存在下一个更大的排列，那么这个数组必须重排为字典序最小的排列（即，其元素按升序排列）。

思路：找比当前排列大的最小值，基本思想是从后往前找一个大数和前面的小数进行交换，并且这个增加的幅度尽可能小，所以要找刚好比小数大的数，这样进行交换，比如123465，那首先就是4和5进行交换，变成123564，但这不是最小的，还需要将5之后的进行顺序排序，因为后面的肯定是逆序，所以把后面的部分反转即可

具体步骤：

![](https://cdn.jsdelivr.net/gh/ddyycc123/imageloader@main/image-20240405112802659.png)

```java
class Solution {
    public void nextPermutation(int[] nums) {
        for(int i=nums.length-1;i>0;i--){
              //找到相邻顺序
            if(nums[i]>nums[i-1]){
                for(int k=nums.length-1;k>=i;k--){
                      //找到最接近小数的大数
                    if(nums[k]>nums[i-1]){
                        swap(nums,i-1,k);
                        reverse(nums,i,nums.length-1);
                        break;
                    }
                }
                return;
            }
        }
          //如果到这 说明当前数已经是最大的了，那就按照题意 变成最小的
        reverse(nums,0,nums.length-1);
    }
    public void swap(int []nums,int i,int j){
        int temp=nums[i];
        nums[i]=nums[j];
        nums[j]=temp;
    }
    public void reverse(int []nums,int i,int j){
        while(i<j){
            swap(nums,i,j);
            i++;
            j--;
        }
    }
}
```

###### [颜色分类](https://leetcode.cn/problems/sort-colors/)
给定一个包含红色、白色和蓝色、共 `n` 个元素的数组 `nums` ，[原地](https://baike.baidu.com/item/%E5%8E%9F%E5%9C%B0%E7%AE%97%E6%B3%95) 对它们进行排序，使得相同颜色的元素相邻，并按照红色、白色、蓝色顺序排列。

思路：双指针，p0指向当前排好的0的下一个位置，p1指向当前排好1的下一个位置，当遇到是1的时候，就将1进行交换，然后只用p1++即可，当遇到0的时候，就需要进行判断当前有没有已经排好的1，如果有，那p0<p1，这个时候就需要交换两次，因为 swap(nums,i,p0);实际上会把已经排好的1交换到后面，所以还需要交换回来，最后p0++;p1++是因为后面加了一个数，所以要进行++

```java
class Solution {
    public void sortColors(int[] nums) {
        int n=nums.length;
        int p0=0,p1=0;
        for(int i=0;i<n;i++){
            if(nums[i]==1){
                swap(nums,i,p1);
                p1++;
            }else if(nums[i]==0){
                swap(nums,i,p0);
                if(p0<p1)
                    swap(nums,i,p1);
                p0++;
                p1++;
            }
        }
    }
    public void swap(int []nums,int i,int j){
        int temp=nums[i];
        nums[i]=nums[j];
        nums[j]=temp;
    }
}
```

思路2️⃣：首先记录0，1，2的个数，然后在原数组上进行填充即可

```java
class Solution {
    public void sortColors(int[] nums) {
        int[]count=new int[3];
        for(int n:nums){
            count[n]++;
        }
        int k=0;
        for(int i=0;i<3;i++){
            for(int j=0;j<count[i];j++){
                nums[k]=i;
                k++;
            }
        }
        
    }
}
```

###### [合并两个有序数组](https://leetcode.cn/problems/merge-sorted-array/)
合并有序数组

思路：合并两个数组有很多方法，最简单的就是直接将nums2填到nums1中，然后sort，或者也可以创建一个新的大小为m+n的数组，然后用双指针将元素插到新数组中，再填充到nums1中，但这两种方法都没有把条件都用完，并且第一种时间复杂度大，第二种空间复杂度大

下面这种解法是首先分别找到两个数组目前最大值的位置，然后挨个找最大值填到nums1的最后，从后往前进行填充

```java
class Solution {
    public void merge(int[] nums1, int m, int[] nums2, int n) {
        int p1=m-1;int p2=n-1;
        int tail=m+n-1;
        int cur;
        while(p1>=0||p2>=0){
            if(p1==-1){
                cur=nums2[p2--];
            }else if(p2==-1){
                cur=nums1[p1--];
            }else if(nums1[p1]>nums2[p2]){
                cur=nums1[p1--];
            }else{
                cur=nums2[p2--];
            }
            nums1[tail--]=cur;
        }
    }
}
```

###### 合并两个有序数组加强版
和上题区别nums1，nums2两个数组均不知道是升序还是降序

思路：这样的话需要额外定义一个res数组用于存储合并之后的值，再覆盖nums1，因为考虑这样一个情况nums1是降序，并且nums2的值都比nums1的值大，那么当将nums2的值填完，nums1进行填的时候，就会存在自己覆盖自己的情况，会吃掉自己一部分值，这个时候只能交换，不过判断条件比较多，所以额外定义一个数组

```java
import java.util.Arrays;
import java.util.List;
import java.util.PriorityQueue;

public class Solution {
        public static void merge(int[] A, int m, int[] B, int n) {
            boolean Ais=A[0]>A[1]?true:false;
            boolean Bis=B[0]>B[1]?true:false;
            int p1=Ais==true?0:m-1;
            int p2=Bis==true?0:n-1;
            int tail=m+n-1;
            int []res=new int[m+n];
            int cur;
            while(tail>=0){
                if(p1==m||p1==-1){
                    if(Bis)
                        cur=B[p2++];
                    else
                        cur=B[p2--];
                }else if(p2==n||p2==-1){
                    if(Ais)
                        cur=A[p1++];
                    else
                        cur=A[p1--];
                }else if(A[p1]>B[p2]){
                    if(Ais)
                        cur=A[p1++];
                    else
                        cur=A[p1--];
                }else{
                    if(Bis)
                        cur=B[p2++];
                    else
                        cur=B[p2--];
                }
                res[tail--]=cur;
            }
            for(int i=0;i<res.length;i++)
                A[i]=res[i];
        }

        public static void main(String[] args) {
            int[] A = {1, 2, 3, 4, 7, 0, 0, 0};
            int[] B = {3, 2, 1};
            int m = 5, n = 3;
            merge(A, m, B, n);
            System.out.println(Arrays.toString(A));
        }
}

```

###### 合并三个有序数组
```java
import java.util.Arrays;

public class MergeSortedArrays {

    public static int[] mergeSortedArrays(int[] array1, int[] array2, int[] array3) {
        int[] result = new int[array1.length + array2.length + array3.length];
        int i = 0, j = 0, k = 0, r = 0;

        while (i < array1.length || j < array2.length || k < array3.length) {
            // 假设数组已经用最大整数填充到无穷
            int a = i < array1.length ? array1[i] : Integer.MAX_VALUE;
            int b = j < array2.length ? array2[j] : Integer.MAX_VALUE;
            int c = k < array3.length ? array3[k] : Integer.MAX_VALUE;

            // 比较这三个值，找到最小的，将其加入结果数组，并移动相应数组的索引
            if (a <= b && a <= c) {
                result[r++] = a;
                i++;
            } else if (b <= a && b <= c) {
                result[r++] = b;
                j++;
            } else {
                result[r++] = c;
                k++;
            }
        }

        return result;
    }

    public static void main(String[] args) {
        int[] array1 = {1, 3, 5};
        int[] array2 = {2, 4, 6};
        int[] array3 = {0, 7, 8, 9};

        int[] mergedArray = mergeSortedArrays(array1, array2, array3);
        System.out.println(Arrays.toString(mergedArray));
    }
}
```

###### [删除有序数组中的重复项](https://leetcode.cn/problems/remove-duplicates-from-sorted-array/)
删除数组重复元素

给你一个 非严格递增排列 的数组 `nums` ，请你[原地](http://baike.baidu.com/item/%E5%8E%9F%E5%9C%B0%E7%AE%97%E6%B3%95) 删除重复出现的元素，使每个元素 只出现一次 ，返回删除后数组的新长度。元素的 相对顺序 应该保持 一致 。然后返回 `nums` 中唯一元素的个数

设置快慢指针，快慢指针从0出发，快指针一直走，当遇到快慢指针值不同时，就要进行更新，让慢指针前进一步，然后更新慢指针所在位置的值

前后指针

```java
class Solution {
    public int removeDuplicates(int[] nums) {
        if(nums.length==0)
            return 0;
        int slow=0,fast=0;
        while(fast<nums.length){
            if(nums[slow]!=nums[fast]){
                slow++;
                nums[slow]=nums[fast];
            }
            fast++;
        }
        return slow+1;
    }
}
```

###### [删除有序数组中的重复项 II](https://leetcode.cn/problems/remove-duplicates-from-sorted-array-ii/)
删除数组重复元素

对于可以保存两个的这种情况，slow代表将要更新的位置，进行判断的是**slow-2和fast**

这个是先赋值，在slow++，是因为对于上一题来讲，两个指针都是从0开始的，所以slow代表只能是已经存在的位置，这个位置的点是这个值第一次出现，所以得slow先前进一格，再进行操作，对于这道题而言，因为是两个，所以从2开始出发的，这个时候slow代表的是重复值的下一个，这个时候就应该是赋新的值，再前进

前后指针

```java
class Solution {
    public int removeDuplicates(int[] nums) {
        if(nums.length==0)
            return 0;
        int slow=2,fast=2;
        while (fast < nums.length) {
            if (nums[slow - 2] != nums[fast]) {
                nums[slow] = nums[fast];
                ++slow;
            }
            ++fast;
        }
        return slow;
    }
}
```

###### [移除元素](https://leetcode.cn/problems/remove-element/)
给你一个数组 `nums` 和一个值 `val`，你需要 [原地](https://baike.baidu.com/item/%E5%8E%9F%E5%9C%B0%E7%AE%97%E6%B3%95) 移除所有数值等于 `val` 的元素。元素的顺序可能发生改变。然后返回 `nums` 中与 `val` 不同的元素的数量。

和上面的思路一样，移除数组元素

前后指针

```java
class Solution {
    public int removeElement(int[] nums, int val) {
        if(nums.length==0){
            return 0;
        }
        int slow=0,fast=0;
        while(fast<nums.length){
            if(nums[fast]!=val){
                nums[slow]=nums[fast];
                slow++;
            }
            fast++;
        }
        return slow;
    }
}
```

###### [移动零](https://leetcode.cn/problems/move-zeroes/)
给定一个数组 `nums`，编写一个函数将所有 `0` 移动到数组的末尾，同时保持非零元素的相对顺序

和上面的一个思路，只不过要在最后进行补0

前后指针

```java
class Solution {
    public void moveZeroes(int[] nums) {
        int slow=0,fast=0;
        while(fast<nums.length){
            if(nums[fast]!=0){
                nums[slow]=nums[fast];
                slow++;
            }
            fast++;
        }
        for(int i=slow;i<nums.length;i++)
            nums[i]=0;
        return;
    }
}
```

###### [柱状图中最大的矩形](https://leetcode.cn/problems/largest-rectangle-in-histogram/)
求在该柱状图中，能够勾勒出来的矩形的最大面积

使用单调栈的方式进行计算，**往栈中加入的是序号**，加入栈的规则是从数组中依次挑出数字，遇到当前值大于栈顶元素的，就加入，遇到当前值小于栈顶元素的，说明当前栈顶元素左右两边能计算的矩形的范围已经出来了，就把栈顶元素拿出来，计算面积

计算面积的步骤是拿出来的栈顶元素对应的高度为矩形高度，长度是**i-1-Stack.peek()**，Stack.peek()表示新的栈顶，这么计算的原因是防止这中间其实是有更高的被拿出去的，算长度的时候也要计算在内，这样的话就要在**最开始的时候加入-1进入栈**，因为考虑栈中正常的最后一个元素计算矩形面积的时候，如果不提前加入一个值，当pop出最后一个高度时，计算peek的时候，如果此时栈为空，就会报错，因此while的循环条件也必须是**Stack.size()>1**，因为后面那个判断条件，如果是不为0的话，heights[-1]也会报错，**右边需要多的一个值0**是用于最后把栈中所有元素都拿出来进行计算判断

```java
class Solution {
    public int largestRectangleArea(int[] heights) {
        Stack<Integer>Stack=new Stack<>();
        int max=0;
        Stack.push(-1);
        for(int i=0;i<=heights.length;i++){
            int curHeight=i==heights.length?0:heights[i];
            while(Stack.size()>1&&curHeight<heights[Stack.peek()])            
            {
                int h=heights[Stack.pop()];
                int size=(i-1-Stack.peek())*h;
                max=Math.max(size,max);
            }
            Stack.push(i);
        }
        return max;
    }
}
```

###### [两数之和 II - 输入有序数组](https://leetcode.cn/problems/two-sum-ii-input-array-is-sorted/)
解法1️⃣

这种解法有点像冒泡排序的方式

```java
class Solution {
    public int[] twoSum(int[] numbers, int target) {
        int []res=new int[2];
        for(int i=0;i<numbers.length;i++){
            for(int j=i+1;j<numbers.length;j++){
                if(numbers[i]+numbers[j]==target){
                    res[0]=i+1;
                    res[1]=j+1;
                    return res;
                }
            }
        }
        return res;
    }
}
```

解法2️⃣

双数之和，这种解法利用双指针，有点像二分查找的方式

左右指针

```java
class Solution {
    public int[] twoSum(int[] numbers, int target) {
        int slow=0;int fast=numbers.length-1;
        int sum;
        while(slow!=fast){
            sum=numbers[slow]+numbers[fast];
            if(sum==target)
                return new int[]{slow+1,fast+1};
            else if(sum<target)
                slow++;
            else
                fast--;
        }
        return new int[]{-1,-1};
    }
}
```

###### [反转字符串](https://leetcode.cn/problems/reverse-string/)
左右指针

```java
class Solution {
    public void reverseString(char[] s) {
        if(s.length==0||s==null)
            return ;
        int slow=0;int fast=s.length-1;
        char temp;
        while(slow<fast){
            temp=s[fast];
            s[fast]=s[slow];
            s[slow]=temp;
            slow++;
            fast--;
        }
    }
}
```

###### [最长回文子串](https://leetcode.cn/problems/longest-palindromic-substring/)
思路：使用for遍历的方式，for的每个i就是一个回文判断的出发点，从两边扩展看有没有，因为不知道该串是奇数还是偶数，所以就两个都进行查找，取最长长度

左右指针

```java
class Solution {
    public String longestPalindrome(String s) {
        String res="";
        for(int i=0;i<s.length();i++){
            String s1=judge(i,i,s);
            String s2=judge(i,i+1,s);
            res=res.length()>s1.length()?res:s1;
            res=res.length()>s2.length()?res:s2;
        }
        return res;
    }

    public String judge(int i,int j,String s){
        while(i>=0&&j<s.length()&&s.charAt(i)==s.charAt(j)){
            i--;
            j++;
        }
          //取i+1是因为断开循环要么是越界，要么是不相等，所以当前的是不能要的
        return s.substring(i+1,j);
    }

}
```

###### [回文子串](https://leetcode.cn/problems/a7VOhD/)
这个是计算回文子串个数

思路：找到字符串中每一组可能是回文子串的起始位置，然后进行逐个遍历，对于回文子串的中心位置就要考虑子串是奇数还是偶数

假设子串的长度为n，则会有2n-1组不同的回文子串中心位置，比如n=4

![](https://cdn.jsdelivr.net/gh/ddyycc123/imageloader@main/image-20240307232721272.png)

因此l=i/2，r=i/2+i%2

```java
class Solution {
    public int countSubstrings(String s) {
        int n=s.length();
        int nums=0;
        for(int i=0;i<2*n-1;i++){
            int l=i/2;int r=i/2+i%2;
            while(l>=0&&r<n&&s.charAt(l)==s.charAt(r)){
                nums++;
                l--;
                r++;
            }
        }
        return nums;
    }
}
```

###### [验证回文串](https://leetcode.cn/problems/valid-palindrome/)
如果在将所有大写字符转换为小写字符、并移除所有非字母数字字符之后，短语正着读和反着读都一样。则可以认为该短语是一个 回文串

```java
class Solution {
    public boolean isPalindrome(String s) {
        int left=0;
        int right=s.length()-1;
        while(left<right){
            while(!Character.isLetterOrDigit(s.charAt(left))&&left<right)
                left++;
            char ch_left=s.charAt(left);
            if(Character.isUpperCase(ch_left))
                ch_left=Character.toLowerCase(ch_left);
            while(!Character.isLetterOrDigit(s.charAt(right))&&left<right)
                right--;
            char ch_right=s.charAt(right);
            if(Character.isUpperCase(ch_right))
                ch_right=Character.toLowerCase(ch_right);
            if(ch_left!=ch_right)
                return false; 
            left++;
            right--;
        }
        return true;
    }
}
```

总结：

1. 判断字符是否是字母  Character.isLetter(ch)
2. 判断字符是否是字母或者数字  Character.isLetterOrDigit(ch)
3. 判断字符是否是大写字母 Character.isUpperCase(ch)
4. 将大写字母转化成小写字母 ch=Character.toLowerCase(ch);

###### [跳跃游戏](https://leetcode.cn/problems/jump-game/)
给你一个非负整数数组 `nums` ，你最初位于数组的 第一个下标 。数组中的每个元素代表你在该位置可以跳跃的最大长度

思路：一直找最远能到那里，i代表当前位置，j代表目前最远能到什么位置

```java
class Solution {
    public boolean canJump(int[] nums) {
        if(nums.length<=1)
            return true;
        int i=0;
        int j=nums[0];
        while(i!=j){
            if(j>=nums.length-1)
                return true;
            i++;
            if(nums[i]+i>j)
                j=nums[i]+i;
        }
        return false;
    }
}
```

###### [跳跃游戏 II](https://leetcode.cn/problems/jump-game-ii/)
思路：题目的意思已经说了可以到达，那么就使用贪心算法，从最后的位置往前看，看每次最远能到哪里，一直到初始位置就是最短跳数了

```java
class Solution {
    public int jump(int[] nums) {
        int index=nums.length-1;
        int res=0;
        while(index>0){
            for(int i=0;i<index;i++){
                if(nums[i]+i>=index){
                    index=i;
                    res++;
                    break;
                }
            }
        }
        return res;
    }
}
```

###### [三数之和](https://leetcode.cn/problems/3sum/)
排序+双指针

每次固定目前最左边的数，然后双指针从剩下的右边部分的两边开始遍历，在查找的时候要不断判断是否有重复的，重复的就跳过

```java
class Solution {
    public List<List<Integer>> threeSum(int[] nums) {
          //先排序
        Arrays.sort(nums);
        List<List<Integer>> res = new ArrayList<>();
          //如果nums[i]>0，也没必要继续，因为nums[i]三元组中最小值
        for (int i = 0; i < nums.length - 2&&nums[i]<=0; i++) {
            //和前一个相等就没必要浪费时间
            if (i > 0 && nums[i] == nums[i - 1]) {
                continue; 
            }
            int left = i + 1, right = nums.length - 1;
            while (left < right) {
           //如果nums[right]<0，也没必要继续，因为nums[right]三元组中最大值
                if(nums[right]<0)
                    break;
                int sum = nums[i] + nums[left] + nums[right];
                if (sum == 0) {
                    res.add(Arrays.asList(nums[i], nums[left], nums[right]));
                    //去掉重复值
                    while (left < right && nums[left] == nums[left + 1]) left++; 
                    while (left < right && nums[right] == nums[right - 1]) right--; 
                    left++;
                    right--;
                } else if (sum < 0) {
                    left++;
                } else {
                    right--;
                }
            }
        }
        return res;
    }
}

```

###### [最接近的三数之和](https://leetcode.cn/problems/3sum-closest/)
```java
class Solution {
    public int threeSumClosest(int[] nums, int target) {
        Arrays.sort(nums);
        int n = nums.length;
        int best = 10000000;

        // 枚举 a
        for (int i = 0; i < n; ++i) {
            // 保证和上一次枚举的元素不相等
            if (i > 0 && nums[i] == nums[i - 1]) {
                continue;
            }
            // 使用双指针枚举 b 和 c
            int j = i + 1, k = n - 1;
            while (j < k) {
                int sum = nums[i] + nums[j] + nums[k];
                // 如果和为 target 直接返回答案
                if (sum == target) {
                    return target;
                }
                // 根据差值的绝对值来更新答案
                if (Math.abs(sum - target) < Math.abs(best - target)) {
                    best = sum;
                }
                if (sum > target) {
                    // 如果和大于 target，移动 c 对应的指针
                    int k0 = k - 1;
                    // 移动到下一个不相等的元素
                    while (j < k0 && nums[k0] == nums[k]) {
                        --k0;
                    }
                    k = k0;
                } else {
                    // 如果和小于 target，移动 b 对应的指针
                    int j0 = j + 1;
                    // 移动到下一个不相等的元素
                    while (j0 < k && nums[j0] == nums[j]) {
                        ++j0;
                    }
                    j = j0;
                }
            }
        }
        return best;
    }
}
```

###### [除自身以外数组的乘积](https://leetcode.cn/problems/product-of-array-except-self/)
给你一个整数数组 `nums`，返回 数组 `answer` ，其中 `answer[i]` 等于 `nums` 中除 `nums[i]` 之外其余各元素的乘积 。

方法1️⃣：超时

```java
class Solution {
    public int[] productExceptSelf(int[] nums) {
        int []answer=new int[nums.length];
        for(int i=0;i<nums.length;i++){
            int res=1;
            for(int j=0;j<nums.length;j++){
                if(i==j)
                    continue;
                res*=nums[j];
            }
            answer[i]=res;
        }
        return answer;
    }
}
```

方法2️⃣：使用双指针，并且一个前缀积和一个后缀积

这个思路是forward和back两个指针分别带着对应的两个方向的前缀积，每个answer的积是forward和back到该位置的积，很妙的是指针到该位置的时候正好是没有结合该位置的元素的，离开之后才会与该位置元素积，用于下一个元素

```java
class Solution {
    public int[] productExceptSelf(int[] nums) {
        int []answer=new int[nums.length];
        Arrays.fill(answer,1);
        int forward=1;int back=1;
        for(int i=0,j=nums.length-1;i<nums.length;i++,j--){
            answer[i]*=forward;
            answer[j]*=back;
            forward*=nums[i];
            back*=nums[j];
        }
        return answer;
    }
}
```

###### [快乐数](https://leetcode.cn/problems/happy-number/)
思路：用快慢指针来判断是否会重逢来判断循环

```java
class Solution {
    public int getNext(int n){
        int sum=0;
        while(n>0){
            int k=n%10;
            n/=10;
            sum+=k*k;
        }
        return sum;
    }

    public boolean isHappy(int n) {
        int slow=n;
        int fast=getNext(n);
        while(fast!=1&&fast!=slow){
            slow=getNext(slow);
            fast=getNext(getNext(fast));
        }
        return fast==1;
    }
}
```

###### [搜索二维矩阵 II](https://leetcode.cn/problems/search-a-2d-matrix-ii/)
编写一个高效的算法来搜索 `m x n` 矩阵 `matrix` 中的一个目标值 `target` 。该矩阵具有以下特性：

+ 每行的元素从左到右升序排列。
+ 每列的元素从上到下升序排列。

思路：从右上方进行查找，因为是要找目标值，所以最好是找大小在中间的值，进行左右移动，所以用左下角和右上角作为起点都可以

```java
class Solution {
    public boolean searchMatrix(int[][] matrix, int target) {
        int m=matrix.length,n=matrix[0].length;
        int i=0,j=n-1;
        while(i<m&&j>=0){
            if(matrix[i][j]==target)
                return true;
            else if(matrix[i][j]>target)
                j--;
            else 
                i++;
        }
        return false;
    }
}
```

###### [分发糖果](https://leetcode.cn/problems/candy/)
思路1：题目说的相邻元素中更大的需要获取更多的糖果，所以可以分成两次遍历

+ 从左到右遍历，只判断ratings[i]和ratings[i-1]的关系，如果ratings[i]>ratings[i-1]，那么i就需要比i-1获取更多的糖果
+ 从右到左遍历，只判断ratings[i]和ratings[i+1]的关系，如果ratings[i]>ratings[i+1]，那么i就需要比i+1获取更多的糖果

```java
class Solution {
    public int candy(int[] ratings) {
        int n=ratings.length;
        int []left=new int[n];
        for(int i=0;i<n;i++){
            if(i>0&&ratings[i]>ratings[i-1])
                left[i]=left[i-1]+1;
            else
                left[i]=1;
        }
        int res=0,right=0;
        for(int i=n-1;i>=0;i--){
            if(i<n-1&&ratings[i]>ratings[i+1])
                right=right+1;
            else
                right=1;
            res+=Math.max(left[i],right);
        }
        return res;

    }
}
```

解法2

如果ratings[i] >= ratings[i - 1]，那么在此之前的递增序列全部加1，否则在此之前的递减序列全部加1

```java
class Solution {
    public int candy(int[] ratings) {
        int n = ratings.length;
        int ret = 1;
        int inc = 1, dec = 0, pre = 1;
        for (int i = 1; i < n; i++) {
            if (ratings[i] >= ratings[i - 1]) {
                dec = 0;
                pre = ratings[i] == ratings[i - 1] ? 1 : pre + 1;
                ret += pre;
                inc = pre;
            } else {
                dec++;
                if (dec == inc) {
                    dec++;
                }
                ret += dec;
                pre = 1;
            }
        }
        return ret;
    }
}
```

##### 前缀和
###### [ 二维区域和检索 - 矩阵不可变](https://leetcode.cn/problems/range-sum-query-2d-immutable/)
二维前缀和

可以使用双重for循环来做，不过时间复杂度高了，可以考虑利用前缀的思想，用相减的方式解决

![](https://cdn.jsdelivr.net/gh/ddyycc123/imageloader@main/image-20231108153054765.png)

```java
class NumMatrix {
    // 定义：preSum[i][j] 记录 matrix 中子矩阵 [0, 0, i-1, j-1] 的元素和
    private int[][] preSum;
    
    public NumMatrix(int[][] matrix) {
        int m = matrix.length, n = matrix[0].length;
        if (m == 0 || n == 0) return;
        // 构造前缀和矩阵
        preSum = new int[m + 1][n + 1];
        for (int i = 1; i <= m; i++) {
            for (int j = 1; j <= n; j++) {
                // 计算每个矩阵 [0, 0, i, j] 的元素和
                preSum[i][j] = preSum[i-1][j] + preSum[i][j-1] + matrix[i - 1][j - 1] - preSum[i-1][j-1];
            }
        }
    }
    
    // 计算子矩阵 [x1, y1, x2, y2] 的元素和
    public int sumRegion(int x1, int y1, int x2, int y2) {
        // 目标矩阵之和由四个相邻矩阵运算获得
        return preSum[x2+1][y2+1] - preSum[x1][y2+1] - preSum[x2+1][y1] + preSum[x1][y1];
    }
}

```

###### 平衡矩阵
![](https://cdn.jsdelivr.net/gh/ddyycc123/imageloader@main/image-20240309172521089.png)

思路：使用二维数组前缀和来求解，**对于数组这种求和的都要考虑使用这种方法**

```java
import java.lang.reflect.Array;
import java.util.Arrays;
import java.util.LinkedList;
import java.util.List;
import java.util.Scanner;

public class Solution {
    public static void main(String[] args) {
        Scanner sc=new Scanner(System.in);
        int n=sc.nextInt();
        int [][]matrix=new int[n][n];
        for(int i=0;i<n;i++){
            String s=sc.next();
            for(int j=0;j<s.length();j++){
                matrix[i][j]=s.charAt(j)-'0';
            }
        }
        int[][]presum=new int[n+1][n+1];
          //求每个位置的前缀和
        for(int i=1;i<=n;i++){
            for(int j=1;j<=n;j++){
                presum[i][j]=presum[i-1][j]+presum[i][j-1]-presum[i-1][j-1]+matrix[i-1][j-1];
            }
        }
        int []results=new int[n];
        for(int i=0;i<n;i++){
            for(int j=0;j<n;j++){
                  //k代表矩形的边长
                for(int k=0;k<n;k++){
                      //越界
                    if(i+k==n||j+k==n){
                        break;
                    }
                    int res=submatrix(presum,i,j,i+k,j+k);
                    if(res*2==(k+1)*(k+1))
                        results[k]++;
                }
            }
        }
        System.out.println(Arrays.toString(results));

    }
    public static int submatrix(int[][]presum,int x1,int y1,int x2,int y2){
          //求方块的前缀和
        return presum[x2+1][y2+1]-presum[x2+1][y1]-presum[x1][y2+1]+presum[x1][y1];
    }


}

```

###### 区间删除
![](https://cdn.jsdelivr.net/gh/ddyycc123/imageloader@main/image-20240309235012133.png)

思路：要使最后的乘积有k个0，那么就需要至少k个2和5，所以首先对数组各数字求因子2和5的数量

因为最后比较的还是2和5的数量，所以先求数组关于2和5数量的前缀和，然后就可以找删除的区间，这里可以用滑动窗口，也可以用二分查找找到那个位置

二分查找的思路是考虑到剩下的要有k个2和5，所以删除的区间最多可以有sum2-k个因子2，所以区间在位置i最大面积的count就是pre[i]+sum2-k，区间起始位置为i时，面积小于等于pre[i]+sum2-k就都是满足情况的

```java
import java.util.Scanner;

public class Solution {
    public static void main(String[] args) {
        Scanner sc=new Scanner(System.in);
        int n=sc.nextInt();
        int k=sc.nextInt();
        int []nums=new int[n];
        for(int i=0;i<n;i++){
            nums[i]=sc.nextInt();
        }
        int []nums2=new int[n];
        int []nums5=new int[n];
        int sum2=0;
        int sum5=0;
        for(int i=0;i<n;i++){
            //求每个数字的因子，可以想象多个数相乘，实际上是其因子相乘
            // 想得到多个0，就需要多个2和5
            nums2[i]=getfactor(nums[i],2);
            nums5[i]=getfactor(nums[i],5);
            sum2+=nums2[i];
            sum5+=nums5[i];
        }
        int []pre2=new int[n+1];
        int []pre5=new int[n+1];
        //求因子数量的前缀和
        for(int i=1;i<=n;i++){
            pre2[i]=pre2[i-1]+nums2[i-1];
            pre5[i]=pre5[i-1]+nums5[i-1];
        }
        int count=0;
        for(int i=0;i<n;i++){
            //查找从每个位置开始的要减去的区间，最大能到什么位置
            //那么小于等于这些位置的都是满足情况的，就可以算出从这个位置开始删去区间可以成功的方案数
            //对于每个区间而言，要剩下的要至少k个，也就是说删除的区间可以占用sum2-k个
            //那在pre2的中的位置就是pre[i]+sum2-k
            int p2=binaryselect(pre2,i+1,n,sum2+pre2[i]-k);
            int p5=binaryselect(pre5,i+1,n,sum5+pre5[i]-k);
            count+=Math.min(p2,p5)-i-1;
        }
        System.out.println(count);

    }

    public static int binaryselect(int []nums,int start,int end,int target){
        while(start<end){
            int mid=start+(end-start)/2;
            //因为不一定只有一个，所以这样找最右边的点
            if(nums[mid]<=target)
                start=mid+1;
            else
                end=mid;
        }
        return start;
    }

    //求因子就是对factor进行求余操作
    public static int getfactor(int n,int factor){
        int cnt=0;
        while(n!=0&&n%factor==0){
            cnt++;
            n/=factor;
        }
        return cnt;
    }
}

```

###### [和为 K 的子数组](https://leetcode.cn/problems/subarray-sum-equals-k/)
给你一个整数数组 `nums` 和一个整数 `k` ，请你统计并返回 该数组中和为k的子数组的个数

思路：由于是连续数组，所以不能使用dfs遍历组合，由于存在负数，所以不好使用滑动窗口，考虑到是连续子数组，任何一段连续子数组之和两段前缀和之差，所以可以一直求前缀和，然后查找之前有没有pre-k大小的前缀和，如果有的话，就说明中间的大小为k

```java
class Solution {
    public int subarraySum(int[] nums, int k) {
        HashMap<Integer,Integer>map=new HashMap<>();
        map.put(0,1);
        int count=0;int pre=0;
        for(int i=0;i<nums.length;i++){
            pre+=nums[i];
            if(map.containsKey(pre-k))
                count+=map.get(pre-k);
            map.put(pre,map.getOrDefault(pre,0)+1);
        }
        return count;
    }
}
```

###### [使数组元素全部相等的最少操作次数](https://leetcode.cn/problems/minimum-operations-to-make-all-array-elements-equal/)
同时给你一个长度为 `m` 的整数数组 `queries` 。第 `i` 个查询中，你需要将 `nums` 中所有元素变成 

`queries[i]` 。你可以执行以下操作 任意 次：

+ 将数组里一个元素 增大 或者 减小 `1` 。

请你返回一个长度为 `m` 的数组 `answer` ，其中 `answer[i]`是将 `nums` 中所有元素变成 `queries[i]` 的 最少 操作次数。

思路：相当于要求数组每一个元素和target之间的差值，所以首先对数组排序，对排序后的数组求前缀和，接着在排序后的数组中用二分查找target的位置index，对于index前面的元素就用index*target-pre[index]，对于index后面的就用后面这一段的总和减这一段个数和target的积

需要注意的点是有可能target不在nums中，这个时候就要注意pre数组中的位置，不能再index+1，否则会越界

```java
class Solution {
    public List<Long> minOperations(int[] nums, int[] queries) {
        Arrays.sort(nums);
        long []pre=new long[nums.length+1];
        List<Long>list=new LinkedList<>();
        int n=nums.length;
        for(int i=1;i<=nums.length;i++)
            pre[i]=nums[i-1]+pre[i-1];
        for(int query:queries){
            int index=Arrays.binarySearch(nums,query);
            long count;
            if(index<0){
                index=-(index+1);
                count=((long)index*query-pre[index])+(pre[n]-pre[index]-(long)query*(n-index));
            }else{
                count=((long)index*query-pre[index])+(pre[n]-pre[index+1]-(long)(n-index-1)*query);
            }
            list.add(count);
        }
        return list;
    }
}
```

二分查找调用库：如果target不在数组中，就会返回负数，-(index+1)就是其应该在数组中的位置，如果有多个相同会返回最左边的索引

```java
            int index=Arrays.binarySearch(nums,target);
            if(index<0){
                index=-(index+1);
            }
```

###### 种花
![](https://cdn.nlark.com/yuque/0/2024/png/42839395/1723603070174-f807478d-2556-4285-8d1f-391f107934b8.png)

```java
import java.util.*;

public class Main {
    public static void main(String[] args) {
        Scanner scanner = new Scanner(System.in);
        int n = scanner.nextInt();
        int[] a = new int[n];

        // 读取并转换输入数组
        for (int i = 0; i < n; i++) {
            a[i] = scanner.nextInt();
            if (a[i] == 0) {
                a[i] = -1;
            }
        }

        // 计算总和
        int s = Arrays.stream(a).sum();

        // 初始化最大和最小前缀和
        int max_sum = 0;
        int min_sum = 0;
        int left = 0;
        int right = 0;
        //滑动窗口的求法来最最大区间和 和 最小区间和
        for (int i = 0; i < n; i++) {
            max_sum = Math.max(max_sum + a[i], a[i]);
            min_sum = Math.min(min_sum + a[i], a[i]);
            left = Math.min(left, min_sum);
            right = Math.max(right, max_sum);
        }

        // 使用set存储所有可能的结果
        Set<Integer> res = new HashSet<>();
        // 最后的结果是在sum-2*left和sum-2*right之间
        for (int i = left; i <= right; i++) {
            res.add(Math.abs(s - 2 * i));
        }

        // 打印结果大小
        System.out.println(res.size());
    }
}
```

##### 差分数组
利用差分数组的思想进行解决

查分数组应用场景：多次对数组中的一部分进行增值减值的操作，这个时候就可以使用差分数组，一般就是题目给定nums[ n ]3的时候 

差分数组的原理：**差分数组中每一个值代表了原数组中i-1和i之间的差值，所以当对数组中一部分进行值的修改时，使用差分数组，只需要对差分数组那个范围的两边进行修改就行了，因为中间都加，对于差分数组来说等于没加**

差分数组的基本框架：首先创建一个全为0的差分数组，然后根据题目中给的值对差分数组对应节点进行赋值，注意题目一般从1开始，所以差分数组添加的时候记得减1，返回原数组也只需要查分数组从0开始一直累加即可

###### [航班预订统计](https://leetcode.cn/problems/corporate-flight-bookings/)
```java
class Solution {
    public int[] corpFlightBookings(int[][] bookings, int n) {
        int []num=new int[n];
        for(int []booking:bookings){
            num[booking[0]-1]+=booking[2];
            if(booking[1]<n){
                num[booking[1]]-=booking[2];
            }
        }
        for(int i=1;i<n;i++){
            num[i]+=num[i-1];
        }
        return num;
    }
}
```

###### [拼车](https://leetcode.cn/problems/car-pooling/)
车上最初有 `capacity` 个空座位。车 只能 向一个方向行驶（也就是说，不允许掉头或改变方向）

给定整数 `capacity` 和一个数组 `trips` ,  `trip[i] = [numPassengersi, fromi, toi]` 表示第 `i` 次旅行有 `numPassengersi` 乘客，接他们和放他们的位置分别是 `fromi` 和 `toi` 。这些位置是从汽车的初始位置向东的公里数。

主要是考虑在什么位置进行添加和删除值，如果是从0开始就直接i和j+1，如果是从1开始就是i-1和j，如果在范围点的不要就要减1

```java
class Solution {
    public boolean carPooling(int[][] trips, int capacity) {
        int[] nums=new int[1001];
        for(int[]trip:trips){
            nums[trip[1]]+=trip[0];
            nums[trip[2]]-=trip[0];
        }
        for(int i=1;i<=1000;i++){
            nums[i]+=nums[i-1];
        }
        for(int i=0;i<=1000;i++){
            if(nums[i]>capacity)
                return false;
        }
        return true;


    }
}
```

##### 花式遍历数组
###### [反转字符串中的单词](https://leetcode.cn/problems/reverse-words-in-a-string/)
解法1️⃣

将字符串使用trim和split方法转化成只有非空字符串的数组，然后加入栈中，然后再从栈中提取出来进行拼接，然后返回

```java
class Solution {
    public String reverseWords(String s) {
        Stack<String>stack=new Stack<>();
        String []ss=s.trim().split(" +");
        for(String word:ss){
            stack.push(word);
        }
        String res="";
        while(!stack.isEmpty()){
            res=res.concat(" ").concat(stack.pop());
        }
        return res.substring(1);
    }
}
```

解法2️⃣

```java
class Solution {
    public String reverseWords(String s) {
        List<String>wordlist=Arrays.asList(s.trim().split(" +"));
        Collections.reverse(wordlist);
        return String.join(" ",wordlist);
    }
}
```

###### [旋转图像](https://leetcode.cn/problems/rotate-image/)
给定一个 n × n 的二维矩阵 `matrix` 表示一个图像。请你将图像顺时针旋转 90 度。

你必须在[原地](https://baike.baidu.com/item/%E5%8E%9F%E5%9C%B0%E7%AE%97%E6%B3%95) 旋转图像，这意味着你需要直接修改输入的二维矩阵。请不要 使用另一个矩阵来旋转图像

顺时针九十度反转数组，可以想到先将矩阵按照对角线进行对称，再将每一行进行反转

```java
class Solution {
    public void rotate(int[][] matrix) {
        int n=matrix.length;
          //按照对角线进行对称
        for(int i=0;i<n;i++)
            for(int j=i;j<n;j++){
                int temp=matrix[i][j];
                matrix[i][j]=matrix[j][i];
                matrix[j][i]=temp;
            }
          //每一行进行反转
        for(int i=0;i<n;i++){
            reverse(matrix[i]);
        }
    }
    public void reverse(int []nums){
        int i=0;int j=nums.length-1;
          //用while循环对一维数组进行反转
        while(i<j){
            int temp=nums[j];
            nums[j]=nums[i];
            nums[i]=temp;
            i++;j--;
        }
    }
}
```

同理也可以写逆时针反转矩阵

```java
// 将二维矩阵原地逆时针旋转 90 度
void rotate2(int[][] matrix) {
    int n = matrix.length;
    // 沿左下到右上的对角线镜像对称二维矩阵
    for (int i = 0; i < n; i++) {
        for (int j = 0; j < n - i; j++) {
            // swap(matrix[i][j], matrix[n-j-1][n-i-1])
            int temp = matrix[i][j];
            matrix[i][j] = matrix[n - j - 1][n - i - 1];
            matrix[n - j - 1][n - i - 1] = temp;
        }
    }
    // 然后反转二维矩阵的每一行
    for (int[] row : matrix) {
        reverse(row);
    }
}

void reverse(int[] arr) { /* 见上文 */}

```

###### [螺旋矩阵](https://leetcode.cn/problems/spiral-matrix/)
给你一个 `m` 行 `n` 列的矩阵 `matrix` ，请按照 顺时针螺旋顺序 ，返回矩阵中的所有元素。

螺旋遍历矩阵，可以想象为上右下左遍历矩阵，使用一个大的while循环，当list满就退出循环，在while循环里用4个if来进行遍历，就根据边界来进行判断，然后用for循环挨个把每个值加入list中

```java
class Solution {
    public List<Integer> spiralOrder(int[][] matrix) {
        int m=matrix.length;int n=matrix[0].length;
        int upperbound=0;int buttonbound=m-1;
        int leftbound=0;int rightbound=n-1;
        ArrayList<Integer>list=new ArrayList<>();
        while(list.size()<m*n){
            if(upperbound<=buttonbound){
                for(int i=leftbound;i<=rightbound;i++){
                    list.add(matrix[upperbound][i]);
                }
                upperbound++;
            }
            if(leftbound<=rightbound){
                for(int i=upperbound;i<=buttonbound;i++){
                    list.add(matrix[i][rightbound]);
                }
                rightbound--;
            }
            if(upperbound<=buttonbound){
                for(int i=rightbound;i>=leftbound;i--){
                    list.add(matrix[buttonbound][i]);
                }
                buttonbound--;
            }
            if(leftbound<=rightbound){
                for(int i=buttonbound;i>=upperbound;i--){
                    list.add(matrix[i][leftbound]);
                }
                leftbound++;
            }
    }
    return list;
}}
```

###### [螺旋矩阵 II](https://leetcode.cn/problems/spiral-matrix-ii/)
和上题类似，只要按照螺旋的轨迹走一遍就行了

```java
class Solution {
    public int[][] generateMatrix(int n) {
        int upper=0;int button=n-1;
        int left=0;int right=n-1;
        int count=1;
        int [][]res=new int[n][n];
        while(count<n*n+1){
            if(upper<=button){
                for(int i=left;i<=right;i++){
                    res[upper][i]=count++;                
                }
                upper++;
            }
            if(left<=right){
                for(int i=upper;i<=button;i++){
                    res[i][right]=count++;                
                }
                right--;
            }
            if(upper<=button){
                for(int i=right;i>=left;i--){
                    res[button][i]=count++;                
                }
                button--;
            }
            if(left<=right){
                for(int i=button;i>=upper;i--){
                    res[i][left]=count++;                
                }
                left++;
            }
        }
        return res;
    }
}
```

##### 滑动窗口
###### [最大连续1的个数 III](https://leetcode.cn/problems/max-consecutive-ones-iii/)
给定一个二进制数组 `nums` 和一个整数 `k`，如果可以翻转最多 `k` 个 `0` ，则返回 数组中连续 `1` 的最大个数 。

思路：使用滑动窗口，保持一个窗口内的0数量小于等于k前提下，求这个窗口最大即可

```java
class Solution {
    public int longestOnes(int[] nums, int k) {
        int n = nums.length;
        int left = 0, lsum = 0, rsum = 0;
        int ans = 0;
        for (int right = 0; right < n; ++right) {
            rsum += 1 - nums[right];
            while (lsum < rsum - k) {
                lsum += 1 - nums[left];
                ++left;
            }
            ans = Math.max(ans, right - left + 1);
        }
        return ans;
    }
}

```

###### [最小覆盖子串](https://leetcode.cn/problems/minimum-window-substring/)
给你一个字符串 `s` 、一个字符串 `t` 。返回 `s` 中涵盖 `t` 所有字符的最小子串。如果 `s` 中不存在涵盖 `t` 所有字符的子串，则返回空字符串 `""` 

思路：

1、我们在字符串 `S` 中使用双指针中的左右指针技巧，初始化 `left = right = 0`，把索引**左闭右开**区间 `[left, right)` 称为一个「窗口」。

2、我们先不断地增加 `right` 指针扩大窗口 `[left, right)`，直到窗口中的字符串符合要求（包含了 `T` 中的所有字符）。

3、此时，我们停止增加 `right`，转而不断增加 `left` 指针缩小窗口 `[left, right)`，直到窗口中的字符串不再符合要求（不包含 `T` 中的所有字符了）。同时，每次增加 `left`，我们都要更新一轮结果。

4、重复第 2 和第 3 步，直到 `right` 到达字符串 `S` 的尽头

**第 2 步相当于在寻找一个「可行解」，然后第 3 步在优化这个「可行解」，最终找到最优解**，也就是最短的覆盖子串。左右指针轮流前进，窗口大小增增减减，窗口不断向右滑动，这就是「滑动窗口」这个名字的来历。

具体实施：

+ 使用HashMap来存储每个字符出现的次数，need表示需要哪些字符，每个字符出现了多少次，window表示目前字符串中有哪些字符了，出现了多少次，两者的字符出现数量相等，就表示某字符匹配成功了，num++，当num的数量和need的大小相等时，说明匹配成功
+ 使用getOrDefault(tt,0)+1方法对hashmap某一字符出现的次数进行更新
+ 当匹配成功后就要进行优化，left进行++，这个时候每次也要进行判断left所在位置的那个字符是否存在与need中，如果存在就要对window中字符的数量进行修改，因为不要它了，并且在此之前还要判断window和need该字符的数量是否相等，如果相等说明去掉该字符之后，就不能匹配成功了，就需要num--

```java
class Solution {
    public String minWindow(String s, String t) {
        Map<Character,Integer>window=new HashMap<>();
        Map<Character,Integer>need=new HashMap<>();
        for(char tt:t.toCharArray()){
            need.put(tt,need.getOrDefault(tt,0)+1);
        }
        int start=0;int len=Integer.MAX_VALUE;
        int left=0;int right=0;
        int num=0;
        while(right<s.length()){
            char c=s.charAt(right);
            right++;
            if(need.containsKey(c)){
                window.put(c,window.getOrDefault(c,0)+1);
                if(need.get(c).equals(window.get(c))){
                    num++;
                }
            }
            while(num==need.size()){
                if(right-left<len){
                    start=left;
                    len=right-left;
                }
                char d=s.charAt(left);
                left++;
                if(need.containsKey(d)){
                    if(need.get(d).equals(window.get(d))){
                        num--;
                    }
                    window.put(d,window.getOrDefault(d,0)-1);
                }
            }
        }
        return len==Integer.MAX_VALUE?"":s.substring(start,start+len);
    }
}
```

###### [无重复字符的最长子串](https://leetcode.cn/problems/longest-substring-without-repeating-characters/)
给定一个字符串 `s` ，请你找出其中不含有重复字符的最长子串的长度

一样的使用滑动窗口，只不过这个计算长度的时候，在循环之后计算长度

```java
class Solution {
        public static int lengthOfLongestSubstring(String s) {
        Map<Character,Integer> window=new HashMap<>();
        int left=0;int right=0;
        int len=Integer.MIN_VALUE;
        while(right<s.length()){
            char c=s.charAt(right);
            right++;
            while(window.containsKey(c)){
                char d=s.charAt(left);
                left++;
                window.remove(d);
            }
            if(right-left>len){
                len=right-left;
            }
            window.put(c,1);
        }
        return len==Integer.MIN_VALUE?s.length():len;
    }
}
```

###### [字符串的排列](https://leetcode.cn/problems/permutation-in-string/)
给你两个字符串 `s1` 和 `s2` ，写一个函数来判断 `s2` 是否包含 `s1` 的排列。如果是，返回 `true` ；否则，返回 `false` 。

换句话说，`s1` 的排列之一是 `s2` 的 子串

子序列

判断一个字符串是否包含另一个字符串的排列。如果是，返回 `true` ；否则，返回 `false` 。

```java
class Solution {
    public boolean checkInclusion(String s1, String s2) {
        Map<Character,Integer>need=new HashMap<Character,Integer>();
        Map<Character,Integer>window=new HashMap<>();
        for(char c:s1.toCharArray()){
            need.put(c,need.getOrDefault(c,0)+1);
        }
        int left=0;int right=0;
        int num=0;
        while(right<s2.length()){
            char c=s2.charAt(right);
            right++;
            if(need.containsKey(c)){
                //这里如果能进去，说明c是多的，那么只能left往左进
                while(need.get(c).equals(window.get(c))){
                    char d=s2.charAt(left);
                    left++;
                    if(need.containsKey(d)){
                        if(need.get(d).equals(window.get(d)))
                            num--;
                        window.put(d,window.getOrDefault(d,0)-1);
                    }
                }
                window.put(c,window.getOrDefault(c,0)+1);
                if(need.get(c).equals(window.get(c)))
                            num++;

            }else{
                window.clear();
                num=0;
                left=right;
            }
            if(num==need.size())
                return true;
        }
        return false;

    }
}
```

###### [找到字符串中所有字母异位词](https://leetcode.cn/problems/find-all-anagrams-in-a-string/)
给定两个字符串 `s` 和 `p`，找到 `s` 中所有 `p` 的 异位词 的子串，返回这些子串的起始索引。不考虑答案输出的顺序。

异位词 指由相同字母重排列形成的字符串（包括相同的字符串）。

这道题是在上题的基础上再求索引

```java
class Solution {
    public List<Integer> findAnagrams(String s, String p) {
        Map<Character,Integer>need=new HashMap<>();
        Map<Character,Integer>window=new HashMap<>();
        for(char c:p.toCharArray()){
            need.put(c,need.getOrDefault(c,0)+1);
        }
        int left=0;int right=0;
        int num=0;
        List <Integer>res=new ArrayList<>();
        while(right<s.length()){
            char c=s.charAt(right);
            right++;
            if(need.containsKey(c)){
                window.put(c,window.getOrDefault(c,0)+1);
                if(need.get(c).equals(window.get(c))){
                    num++;
                }
            }
            while(right-left>=p.length()){
                if(num==need.size()){
                    res.add(left);
                }
                char ss=s.charAt(left);
                left++;
                if(need.containsKey(ss)){
                    if(need.get(ss).equals(window.get(ss)))
                        num--;
                    window.put(ss,window.getOrDefault(ss,0)-1);
                }
            }
        }
        return res;
    }
}
```

###### [长度最小的子数组](https://leetcode.cn/problems/minimum-size-subarray-sum/)
长度最小的子数组和

使用滑动窗口

给定一个含有 `n` 个正整数的数组和一个正整数 `target` 。

找出该数组中满足其总和大于等于 `target` 的长度最小的 子数组 `[numsl, numsl+1, ..., numsr-1, numsr]` ，并返回其长度。如果不存在符合条件的子数组，返回 `0` 

总和大于等于target的长度最小的子数组

```java
class Solution {
    public int minSubArrayLen(int target, int[] nums) {
        int start=0;int len=Integer.MAX_VALUE;
        int sum=0;
        int left=0;int right=0;
        while(right<nums.length){
            int n=nums[right];
            sum+=n;
            right++;
            while(sum>=target){
                if(right-left<len){
                    len=right-left;
                }
                int n_left=nums[left];
                left++;
                sum-=n_left;
            }
        }
        return len==Integer.MAX_VALUE?0:len;
    }
}
```

###### [滑动窗口最大值](https://leetcode.cn/problems/sliding-window-maximum/)
给你一个整数数组 `nums`，有一个大小为 `k` 的滑动窗口从数组的最左侧移动到数组的最右侧。你只可以看到在滑动窗口内的 `k` 个数字。滑动窗口每次只向右移动一位。

这个是求每个窗口的最大值

思路1️⃣：利用有序队列来做，最先想到的是考虑每次移动的时候怎么把最左边的值给移除有序队列但其实是不用考虑的，因为最后**只要窗口内的最大值**，所以**只要目前有序队列的最大值不是目前窗口之外的即可**

具体做法：用一个二维数组来记录给个值的大小和索引位置，每移动一次窗口的时候就判断目前有序队列的最大值是不是窗口之外的，如果是就poll

```java
import java.util.PriorityQueue;

class Solution {
    public int[] maxSlidingWindow(int[] nums, int k) {
        PriorityQueue<int[]>queue=new PriorityQueue<>(
                (int []a,int []b)->{
                    if(a[0]!=b[0])
                        return b[0]-a[0];
                    else
                        return b[1]-a[1];
                }
        );
        int []res=new int[nums.length-k+1];
        for(int i=0;i<k;i++){
            queue.add(new int[]{nums[i],i});
        }
        res[0]=queue.peek()[0];
        for(int i=1;i<nums.length-k+1;i++){
            queue.add(new int[]{nums[i+k-1],i+k-1});
            while(queue.peek()[1]<i)
                queue.poll();
            res[i]=queue.peek()[0];
        }
        return res;

    }

}
```

思路2️⃣：利用单调队列，整体思路是考虑一个情况，在窗口中i<j，并且nums[i]<=nums[j]在这样的情况下，i是永远不可能成为窗口的最大值的，所以就可以不用存i

具体做法：使用一个双向队列，队列存储的是索引的值，考虑到前面的思路，队列中索引值是单调递增的，但其对应的nums的值是单调递减的，这样才能满足前面的思路，而每个窗口的最大值就是当前队列头中索引在窗口范围内的索引

```java
import java.util.PriorityQueue;

class Solution {
    public int[] maxSlidingWindow(int[] nums, int k) {
        int n=nums.length;
        Deque<Integer>dq=new LinkedList<>();
        for(int i=0;i<k;i++){
            while(!dq.isEmpty()&&nums[i]>nums[dq.peekLast()]){
                dq.pollLast();
            }
            dq.offerLast(i);
        }
        int []res=new int[n-k+1];
        res[0]=nums[dq.peekFirst()];
        for(int i=k;i<n;i++){
            while(!dq.isEmpty()&&nums[i]>nums[dq.peekLast()]){
                dq.pollLast();
            }
            dq.offerLast(i);
            while(dq.peekFirst()<=i-k)
                dq.pollFirst();
            res[i-k+1]=nums[dq.peekFirst()];
        }
        return res;

    }

}
```

###### [环形子数组的最大和](https://leetcode.cn/problems/maximum-sum-circular-subarray/)
给定一个长度为 `n` 的环形整数数组 `nums` ，返回 `nums` 的非空子数组的最大可能和 。

环形数组 意味着数组的末端将会与开头相连呈环状。形式上， `nums[i]` 的下一个元素是 `nums[(i + 1) % n]` ， `nums[i]` 的前一个元素是 `nums[(i - 1 + n) % n]` 。

子数组 最多只能包含固定缓冲区 `nums` 中的每个元素一次。形式上，对于子数组 `nums[i], nums[i + 1], ..., nums[j]` ，不存在 `i <= k1, k2 <= j` 其中 `k1 % n == k2 % n` 。

这个是环行数组，所以子数组有可能不是环形，也有可能是环行，对于不是环形的情况，按照之前的思路即可

对于是环形的情况，就需要拆成两部分，前面一部分，后面一部分，然后对后面的部分进行遍历rightSum[i]，i等于1，2，....，接着与左边对应的最大值求和，找到最大值

```java
class Solution {
    public int maxSubarraySumCircular(int[] nums) {
        int []leftSum=new int[nums.length];
        leftSum[0]=nums[0];
        int pre=nums[0];
        int res=nums[0];
        int left=nums[0];
        for(int i=1;i<nums.length;i++){
            pre=Math.max(nums[i],pre+nums[i]);
            res=Math.max(res,pre);
            left+=nums[i];
            leftSum[i]=Math.max(leftSum[i-1],left);
        }
        int right=0;
        for(int i=nums.length-1;i>0;i--){
            right+=nums[i];
            res=Math.max(res,leftSum[i-1]+right);
        }
        return res;
    }
}
```

###### [串联所有单词的子串](https://leetcode.cn/problems/substring-with-concatenation-of-all-words/)
给定一个字符串 `s` 和一个字符串数组 `words`。 `words` 中所有字符串 长度相同。

`s` 中的 串联子串 是指一个包含  `words` 中所有字符串以任意顺序排列连接起来的子串。

+ 例如，如果 `words = ["ab","cd","ef"]`， 那么 `"abcdef"`， `"abefcd"`，`"cdabef"`， `"cdefab"`，`"efabcd"`， 和 `"efcdab"` 都是串联子串。 `"acdbef"` 不是串联子串，因为他不是任何 `words` 排列的连接。

返回所有串联子串在 `s` 中的开始索引。你可以以 任意顺序 返回答案

最简单的思路就是建立一个长度为len的窗口，将其按照长度为d进行切片，接着放入一个hashmap中，然后将words也放入hashmap中，进行比较，接着整体往前移一位，但这会超时

所以建立多起点滑动窗口，因为是固定长度，所以可以初始化多个窗口，从0到d，放入一个list中，当窗口进行移动的时候就会移动d个位置，只需要增加一个单词，和减去一个单词，

这样能省去上一个思路中每次前进一步就要重新初始化窗口的时间

```java
class Solution {
    public List<Integer> findSubstring(String s, String[] words) {
        List<Integer> ans = new ArrayList<>();
        Map<String, Integer> key = new HashMap<>();

        // 建立wordsmap        
        for(String word:words){
            key.put(word, key.getOrDefault(word, 0) + 1);
        }
        int n = words.length;
        int d = words[0].length();
        int slen = s.length();
        int len = n * d;

        // 初始化多起点的滑动窗口
        List<Map<String, Integer>> ma = new ArrayList<>();
        for (int i = 0; i < d; i++) {
            ma.add(new HashMap<>());
        }
        //一共建立d个窗口
        for (int i = 0; i < d && i + len <= slen; i++) {
            for (int j = i; j < i + len; j += d) {
                String str = s.substring(j, j + d);
                ma.get(i).put(str, ma.get(i).getOrDefault(str, 0) + 1);
            }
            //如果相等就可以添加到ans中
            if (ma.get(i).equals(key)) {
                ans.add(i);
            }
        }

        //遍历d即后面的位置，首先判断这个位置是属于哪一个窗口的起点
        //然后对这个窗口删除一个单词，添加一个单词，然后判断新窗口是否和key窗口相等
        // 滑动，先根据i%d判断当前是哪个窗口。然后滑动d格
        for (int i = d; i + len <= slen; i++) {
            int r = i % d;
            String del = s.substring(i - d, i - d + d);
            String add = s.substring(i + len - d, i + len - d + d);
            Map<String, Integer> window = ma.get(r);
            window.put(del, window.getOrDefault(del, 0) - 1);
            if (window.get(del) == 0) {
                window.remove(del);
            }
            window.put(add, window.getOrDefault(add, 0) + 1);

            if (window.equals(key)) {
                ans.add(i);
            }
        }

        return ans;
    }
}
```

##### 二分查找
###### [猜数字大小](https://leetcode.cn/problems/guess-number-higher-or-lower/)
```java
/** 
 * Forward declaration of guess API.
 * @param  num   your guess
 * @return 	     -1 if num is higher than the picked number
 *			      1 if num is lower than the picked number
 *               otherwise return 0
 * int guess(int num);
 */

public class Solution extends GuessGame {
    public int guessNumber(int n) {
        int left=1,right=n;
        while(left<=right){
            int mid=left+(right-left)/2;
            if(guess(mid)==0)
                return mid;
            else if(guess(mid)>0){
                left=mid+1;
            }else{
                right=mid-1;
            }
        }
        return -1;
    }
}
```

###### [搜索插入位置](https://leetcode.cn/problems/search-insert-position/)
给定一个排序数组和一个目标值，在数组中找到目标值，并返回其索引。如果目标值不存在于数组中，返回它将会被按顺序插入的位置

```java
class Solution {
    public int searchInsert(int[] nums, int target) {
        int left=0,right=nums.length-1;
        while(left<=right){
            int mid=left+(right-left)/2;
            if(nums[mid]==target){
                return mid;
            }else if(nums[mid]>target){
                right=mid-1;
            }else{
                left=mid+1;
            }
        }
        return left;
    }
}
```

###### [搜索二维矩阵](https://leetcode.cn/problems/search-a-2d-matrix/)
思路：从两个对角进行搜索

```java
class Solution {
    public boolean searchMatrix(int[][] matrix, int target) {
        int m=matrix.length,n=matrix[0].length;
        int i=0,j=n-1;
        while(i<m&&j>=0){
            if(matrix[i][j]==target)
                return true;
            else if(matrix[i][j]>target)
                j--;
            else 
                i++;
        }
        return false;
    }
}
```

###### [寻找旋转排序数组中的最小值](https://leetcode.cn/problems/find-minimum-in-rotated-sorted-array/)
循环有序数组找最小值

```java
class Solution {
    public int findMin(int[] nums) {
        int n=nums.length;
        int left=0,right=nums.length-1;
        while(left<=right){
            int mid=left+(right-left)/2;
            //特殊情况
            if(left==right){
                return nums[left];
            }
            //处理命中情况，即找最小值
            if((mid!=0&&nums[mid-1]>nums[mid])
            ||(mid==0&&nums[mid]<nums[right]))
                return nums[mid];
            //处理非命中情况
            //判断最小值归属区间
            else if(nums[mid]<nums[right]){
                right=mid-1;
            }else
                left=mid+1;
        }
        return -1;
    }
}
```

###### [搜索旋转排序数组](https://leetcode.cn/problems/search-in-rotated-sorted-array/)
循环有序数组找目标值

思路：这道题由于不知道数组是从什么位置开始旋转的且要求时间复杂度为logn，所以只能进行判断，考虑到旋转一次，所以肯定是有一半的区域是完全升序的，所以判断的时候只用判断target是否在完全升序这一边，如果不在就算在else这边，通过这样的方式进行求解

```java
class Solution {
    public int[] searchX(int[] nums, int target) {
        int n=nums.length;
        int left=0,right=n-1;
        while(left<=right){
            int mid=left+(right-left)/2;
            //先处理命中
            if(nums[mid]==target){
                return mid;
            //再处理不命中
            //找target在哪个区间 
            //首先就要判断哪个区间是有序的
            }else if(nums[left]<=nums[mid]){
                //左边有序
                if(nums[left]<=target&&target<=nums[mid]){
                    right=mid-1;
                }else{
                    left=mid+1;
                }
            }else{
                //右边有序
                if(nums[mid]<=target&&target<=nums[right]){
                    left=mid+1;
                }else{
                    right=mid-1;
                }
            }
        }
        return -1;
    }
        
}
```

###### [统计目标成绩的出现次数](https://leetcode.cn/problems/zai-pai-xu-shu-zu-zhong-cha-zhao-shu-zi-lcof/)
某班级考试成绩按非严格递增顺序记录于整数数组 `scores`，请返回目标成绩 `target` 的出现次数。

搜索目标范围

```java
class Solution {
    public int countTarget(int[] scores, int target) {
        return bis(scores,target,0,scores.length-1);
    }
    public int bis(int []scores,int target,int first,int last){
        if(first>last)
            return 0;
        if(first==last){
            if(scores[first]==target)
                return 1;
            else
                return 0;
        }
        int mid=first+(last-first)/2;
        if(scores[mid]<target){
            return bis(scores,target,mid+1,last);
        }else if(scores[mid]>target){
            return bis(scores,target,first,mid-1);
        }else{
            return bis(scores,target,first,mid-1)+bis(scores,target,mid+1,last)+1;
        }
    }
}
```

###### [二分查找](https://leetcode.cn/problems/binary-search/)
给定一个 `n` 个元素有序的（升序）整型数组 `nums` 和一个目标值 `target`  ，写一个函数搜索 `nums` 中的 `target`，如果目标值存在返回下标，否则返回 `-1`。

```java
class Solution {
    public int search(int[] nums, int target) {
        int left=0;
        int right=nums.length-1;
        while(left<=right){
            int mid=left+(right-left)/2;
            if(target==nums[mid])
                return mid;
            else if(target>nums[mid])
                left=mid+1;
            else
                right=mid-1;
        }
        return -1;
    }
}
```

###### [在排序数组中查找元素的第一个和最后一个位置](https://leetcode.cn/problems/find-first-and-last-position-of-element-in-sorted-array/)
找第一个位置和最后一个位置

用二分查找的通用框架，使用两次，找到左边界和右边界

```java
class Solution {
    public int[] searchRange(int[] nums, int target) {
        int left=getfirst(nums,target);
        int right=getlast(nums,target);
        return new int[]{left,right};
    }

    public int getfirst(int []nums,int target){
        int left=0,right=nums.length-1;
        while(left<=right){
            int mid=left+(right-left)/2;
            //先处理命中
            if(nums[mid]==target){
                //真命中   前后探测法确定搜索空间
                if(mid==0||nums[mid-1]!=target)
                    return mid;
                //伪命中
                else
                    right=mid-1;
            }else if(nums[mid]>target){
                right=mid-1;
            }else{
                left=mid+1;
            }
        }
        return -1;
    }

    public int getlast(int []nums,int target){
        int left=0,right=nums.length-1;
        while(left<=right){
            int mid=left+(right-left)/2;
            if(nums[mid]==target){
                if(mid==nums.length-1||nums[mid+1]!=target){
                    return mid;
                }else{
                    left=mid+1;
                }
            }else if(nums[mid]>target){
                right=mid-1;
            }else{
                left=mid+1;
            }
        }
        return -1;
    }
}
```

###### 查找第一个大于等于x和最后一个小于等于x的数
一样的思路：前面的步骤一样，先处理命中情况，再处理不命中，在命中情况下处理真命中和伪命中

```java
class Solution {
    public int[] searchRange(int[] nums, int target) {
        int left=getfirst(nums,target);
        int right=getlast(nums,target);
        return new int[]{left,right};
    }

    public int getfirstge(int []nums,int target){
        int n=nums.length;
        int left=0,right=n-1;
        while(left<=right){
            int mid=left+(right-left)/2;
            //先处理命中
            if(nums[mid]>=target){
                if(mid==0||nums[mid-1]<target){
                    return mid;
                }else{
                    right=mid-1;
                }
            }else{
                left=mid+1;
            }
        }
        return -1;
    }

    public int getlastle(int []nums,int target){
        int n=nums.length;
        int left=0,right=n-1;
        while(left<=right){
            int mid=left+(right-left)/2;
            if(nums[mid]<=target){
                if(mid==n-1||nums[mid+1]>target){
                    return mid;
                }else{
                    left=mid+1;
                }
            }else{
                right=mid-1;
            }
        }
        return -1;
    }
        
}
```

###### [寻找比目标字母大的最小字母](https://leetcode.cn/problems/find-smallest-letter-greater-than-target/)
给你一个字符数组 `letters`，该数组按非递减顺序排序，以及一个字符 `target`。`letters` 里至少有两个不同的字符。

返回 `letters` 中大于target的最小的字符。如果不存在这样的字符，则返回 `letters` 的第一个字符。

```java
class Solution {
    public char nextGreatestLetter(char[] letters, char target) {
        int left=0,right=letters.length-1;
        while(left<=right){
            int mid=left+(right-left)/2;
            //处理命中 题目要求的是大于，那这里的判断条件就是大于
            if(letters[mid]>target){
                //真命中
                if(mid==0||letters[mid-1]<=target)
                    return letters[mid];
                else
                    right=mid-1;
            }else{
                left=mid+1;
            }
        }
        return letters[0];
    }
}
```

###### [判断子序列](https://leetcode.cn/problems/is-subsequence/)
给定字符串 s 和 t ，判断 s 是否为 t 的子序列。

字符串的一个子序列是原始字符串删除一些（也可以不删除）字符而不改变剩余字符相对位置形成的新字符串。（例如，`"ace"`是`"abcde"`的一个子序列，而`"aec"`不是）。

方法1️⃣：直接进行判断

```java
class Solution {
    public boolean isSubsequence(String s, String t) {
        if(s.length()==0)
            return true;
        int k=0;
        for(int i=0;i<t.length();i++){
            if(s.charAt(k)==t.charAt(i))
                k++;
            if(k==s.length())
                return true;
        }
        if(k==s.length())
            return true;
        else
            return false;
    }
}
```

方法2️⃣：二分法

```java
class Solution {
    boolean isSubsequence(String s, String t) {
        int m = s.length(), n = t.length();
        ArrayList<Integer>[] index = new ArrayList[256];
        for (int i = 0; i < n; i++) {
            char c = t.charAt(i);
            if (index[c] == null) 
                index[c] = new ArrayList<>();
            index[c].add(i);
        }
        int j = 0;
        for (int i = 0; i < m; i++) {
            char c = s.charAt(i);
            if (index[c] == null) 
                return false;
            int pos = left_bound(index[c], j);
            if (pos == -1) 
                return false;
            j = index[c].get(pos) + 1;
        }
        return true;
    }

    int left_bound(ArrayList<Integer> arr, int target) {
        int left = 0, right = arr.size();
        while (left < right) {
            int mid = left + (right - left) / 2;
            if (target > arr.get(mid)) {
                left = mid + 1;
            } else {
                right = mid;
            } 
        }
        if (left == arr.size()) {
            return -1;
        }
        return left;
    }
}
```

###### [匹配子序列的单词数](https://leetcode.cn/problems/number-of-matching-subsequences/)
给定字符串 `s` 和字符串数组 `words`, 返回  `words[i]` 中是`s`的子序列的单词个数 。

字符串的子序列是从原始字符串中生成的新字符串，可以从中删去一些字符(可以是none)，而不改变其余字符的相对顺序

```java
class Solution {
    int numMatchingSubseq(String s, String[] words) {
    // 对 s 进行预处理
    // char -> 该 char 的索引列表
    ArrayList<Integer>[] index = new ArrayList[256];
    for (int i = 0; i < s.length(); i++) {
        char c = s.charAt(i);
        if (index[c] == null) {
            index[c] = new ArrayList<>();
        }
        index[c].add(i);
    }
    
    int res = 0;
    for (String word : words) {
        // 字符串 word 上的指针
        int i = 0;
        // 串 s 上的指针
        int j = 0;
        // 借助 index 查找 word 中每个字符的索引
        for (; i < word.length(); i++) {
            char c = word.charAt(i);
            // 整个 s 压根儿没有字符 c
            if (index[c] == null) {
                break;
            }
            int pos = left_bound(index[c], j);
            // 二分搜索区间中没有找到字符 c
            if (pos == -1) {
                break;
            }
            // 向前移动指针 j
            j = index[c].get(pos) + 1;
        }
        // 如果 word 完成匹配，则是子序列
        if (i == word.length()) {
            res++;
        }
    }
    
    return res;
}

    int left_bound(ArrayList<Integer> arr, int target) {
        int left = 0, right = arr.size();
        while (left < right) {
            int mid = left + (right - left) / 2;
            if (target > arr.get(mid)) {
                left = mid + 1;
            } else {
                right = mid;
            } 
        }
        if (left == arr.size()) {
            return -1;
        }
        return left;
    }

}
```

###### 塔子哥删数组
![](https://cdn.jsdelivr.net/gh/ddyycc123/imageloader@main/image-20240330123129109.png)

方法1️⃣：使用遍历➕二分查找的方式

```java
import java.util.Scanner;

public class Solution {
    public static void main(String[] args) {
        Scanner in=new Scanner(System.in);
        int n=in.nextInt();
        int []nums=new int[n+2];
        nums[0]=0;
        for(int i=1;i<=n;i++){
            nums[i]=in.nextInt();
        }
        nums[n+1]=Integer.MAX_VALUE;
        int min_r=n;
        //保证右边是有序的
        for(int i=n;i>=0;i--){
            if(nums[i+1]>=nums[i]){
                min_r=i;
            }else
                break;
        }
        int ans=0;
        //要删除的区间是left-right的区间
        for(int left=1;left<=n;left++){
            //右边min_r-1，是因为就算最后right的位置是min_r-1也是正确的
            int l=Math.max(left,min_r-1),r=n;
            //通过二分查找 找到右边最左大于左边的位置
            while (l<r){
                int right=(l+r)>>1;
                if(nums[left-1]<=nums[right+1]){
                    r=right;
                }else{
                    l=right+1;
                }
            }
            ans+=n-l+1;
            //保证左边是有序的
            if(nums[left]<nums[left-1])
                break;
        }
        System.out.println(ans);
    }
}

```

方法2️⃣：一样的思路，只不过在找right的时候就直接从min_r-1的位置向右开始找

```java
import java.util.Scanner;

public class Solution {
    public static void main(String[] args) {
        Scanner scanner = new Scanner(System.in);
        int n = scanner.nextInt();
        int[] a = new int[n + 2];
        a[0] = 0;
        a[n + 1] = 2 * (int)Math.pow(10, 9);

        for (int i = 1; i <= n; i++) {
            a[i] = scanner.nextInt();
        }

        int min_r = n;
        for (int i = n; i > 0; i--) {
            if (a[i] <= a[i + 1]) {
                min_r = i;
            } else {
                break;
            }
        }

        int ans = 0;
        int right = min_r - 1;
        for (int left = 1; left <= n; left++) {
            right = Math.max(right, left);
            while (right <= n && a[right + 1] < a[left - 1]) {
                right++;
            }
            ans += n - right + 1;

            if (a[left] < a[left - 1]) {
                break;
            }
        }

        System.out.println(ans);
    }
}
```

###### 塔子哥修改区间
思路：就是一个遍历的过程，k的取值在1-n之间，所以用二分法每次从中挑选一个k，来判断这个k是否成立，通过这样不断压缩区间，得到最小k

为了方便查找原数组，用数组nextW来记录每个位置i的下一个W的位置，然后每次使用一个k用check方法来判断是否成立

```java
import java.util.*;

public class Main {
    public static void main(String[] args) {
        Scanner sc = new Scanner(System.in);
        int n = sc.nextInt();
        int m = sc.nextInt();
        sc.nextLine();  // 消耗换行符
        String s = sc.nextLine();

        if (!s.contains("W")) {
            System.out.println(0);
            return;
        }

        int[] nextW = new int[n];
        Arrays.fill(nextW, n);
        for (int i = n - 1; i >= 0; i--) {
            if (s.charAt(i) == 'W') {
                nextW[i] = i;
            } else if (i < n - 1) {
                nextW[i] = nextW[i + 1];
            }
        }

        int l = 1, r = n;
        while (l < r) {
            int mid = (l + r) >> 1;
            if (check(mid, nextW, n, m)) {
                r = mid;
            } else {
                l = mid + 1;
            }
        }

        System.out.println(l);
    }

    private static boolean check(int mid, int[] nextW, int n, int m) {
        int p = nextW[0];
        //c是记录分割次数，mid表示当前的k，m是最多可以分割m次
        int c = 0;
        while (p < n) {
            c++;
            if (p + mid < n) {
                p = nextW[p + mid];
            } else {
                p = n;
            }
        }
        return c <= m;
    }
}

```

###### [寻找重复数](https://leetcode.cn/problems/find-the-duplicate-number/)
给定一个包含 `n + 1` 个整数的数组 `nums` ，其数字都在 `[1, n]` 范围内（包括 `1` 和 `n`），可知至少存在一个重复的整数。

假设 `nums` 只有 一个重复的整数 ，返回这个重复的数 。

思路1️⃣：考虑找循环链表的方式。采用双指针的方式，考虑如果数组每个都是1-n的不同元素，那么如果让i等于0，从num[0]往后走，会把每个位置走一遍，但如果存在重复的元素，就会一直在一个范围内遍历，那这就和循环链表一样了，就可以使用快慢指针

```java
class Solution {
    public int findDuplicate(int[] nums) {
        int slow=0,fast=0;
        do{
            slow=nums[slow];
            fast=nums[nums[fast]];
        }while(slow!=fast);
        slow=0;
        while(slow!=fast){
            slow=nums[slow];
            fast=nums[fast];
        }
        return slow;
    }
}
```

思路2️⃣：最简单的方式是挨个遍历1-n中每个数，用一个变量cnt统计当前这个数在数组中出现的次数，大于1则返回，时间复杂度更低的就是考虑使用二分查找来找1-n中的数，考虑首先对数组中的元素进行计数

如果nums =[3,1,3,4,2]，可以统计为下面的表格，cnt统计的是小于上面num的个数

![](https://cdn.jsdelivr.net/gh/ddyycc123/imageloader@main/image-20240405163258074.png)

通过上表可以看出，按理来说num=2时，cnt应该小于等于2的，但大于了2，则说明num在1-2之间，通过这样的思路来找num

```java
class Solution {
    public int findDuplicate(int[] nums) {
        int n=nums.length;
        int l=1,r=n-1,res=-1;
        while(l<=r){
            int mid=(l+r)>>1;
            int cnt=0;
            for(int i=0;i<n;i++){
                if(nums[i]<=mid)
                    cnt++;
            }
            if(mid<cnt){
                r=mid-1;
                res=mid;
            }else{
                l=mid+1;
            }
        }
        return res;
    }
}
```

###### [寻找两个正序数组的中位数](https://leetcode.cn/problems/median-of-two-sorted-arrays/)
给定两个大小分别为 `m` 和 `n` 的正序（从小到大）数组 `nums1` 和 `nums2`。请你找出并返回这两个正序数组的 中位数 。

思路1️⃣：因为两个数组都是有序的，可以直接合并，然后找到中位数即可

```java
class Solution {
    public double findMedianSortedArrays(int[] nums1, int[] nums2) {
        int[] nums;
        int m = nums1.length;
        int n = nums2.length;
        nums = new int[m + n];
        if (m == 0) {
            if (n % 2 == 0) {
                return (nums2[n / 2 - 1] + nums2[n / 2]) / 2.0;
            } else {

                return nums2[n / 2];
            }
        }
        if (n == 0) {
            if (m % 2 == 0) {
                return (nums1[m / 2 - 1] + nums1[m / 2]) / 2.0;
            } else {
                return nums1[m / 2];
            }
        }

        int count = 0;
        int i = 0, j = 0;
        while (count != (m + n)) {
            if (i == m) {
                while (j != n) {
                    nums[count++] = nums2[j++];
                }
                break;
            }
            if (j == n) {
                while (i != m) {
                    nums[count++] = nums1[i++];
                }
                break;
            }

            if (nums1[i] < nums2[j]) {
                nums[count++] = nums1[i++];
            } else {
                nums[count++] = nums2[j++];
            }
        }

        if (count % 2 == 0) {
            return (nums[count / 2 - 1] + nums[count / 2]) / 2.0;
        } else {
            return nums[count / 2];
        }
    }
}
```

思路2️⃣：不用完全合并，找到中间位置即可

```java
public double findMedianSortedArrays(int[] A, int[] B) {
    int m = A.length;
    int n = B.length;
    int len = m + n;
    int left = -1, right = -1;
    int aStart = 0, bStart = 0;
    for (int i = 0; i <= len / 2; i++) {
          //用left防止长度为偶数
        left = right;
        if (aStart < m && (bStart >= n || A[aStart] < B[bStart])) {
            right = A[aStart++];
        } else {
            right = B[bStart++];
        }
    }
    if ((len & 1) == 0)
        return (left + right) / 2.0;
    else
        return right;
}
```

思路3️⃣：[使用二分法](https://leetcode.cn/problems/median-of-two-sorted-arrays/solutions/8999/xiang-xi-tong-su-de-si-lu-fen-xi-duo-jie-fa-by-w-2/?envType=study-plan-v2&envId=top-100-liked)

```java
public double findMedianSortedArrays(int[] nums1, int[] nums2) {
    int n = nums1.length;
    int m = nums2.length;
    int left = (n + m + 1) / 2;
    int right = (n + m + 2) / 2;
    //将偶数和奇数的情况合并，如果是奇数，会求两次同样的 k 。
    return (getKth(nums1, 0, n - 1, nums2, 0, m - 1, left) + getKth(nums1, 0, n - 1, nums2, 0, m - 1, right)) * 0.5;  
}
    
    private int getKth(int[] nums1, int start1, int end1, int[] nums2, int start2, int end2, int k) {
        int len1 = end1 - start1 + 1;
        int len2 = end2 - start2 + 1;
        //让 len1 的长度小于 len2，这样就能保证如果有数组空了，一定是 len1 
        if (len1 > len2) return getKth(nums2, start2, end2, nums1, start1, end1, k);
        if (len1 == 0) return nums2[start2 + k - 1];

        if (k == 1) return Math.min(nums1[start1], nums2[start2]);

        int i = start1 + Math.min(len1, k / 2) - 1;
        int j = start2 + Math.min(len2, k / 2) - 1;

        if (nums1[i] > nums2[j]) {
            return getKth(nums1, start1, end1, nums2, j + 1, end2, k - (j - start2 + 1));
        }
        else {
            return getKth(nums1, i + 1, end1, nums2, start2, end2, k - (i - start1 + 1));
        }
    }
```

###### [找出出现至少三次的最长特殊子字符串 II](https://leetcode.cn/problems/find-longest-special-substring-that-occurs-thrice-ii/)
给你一个仅由小写英文字母组成的字符串 `s` 。

如果一个字符串仅由单一字符组成，那么它被称为 特殊 字符串。例如，字符串 `"abc"` 不是特殊字符串，而字符串 `"ddd"`、`"zz"` 和 `"f"` 是特殊字符串。

返回在 `s` 中出现 至少三次 的 最长特殊子字符串 的长度，如果不存在出现至少三次的特殊子字符串，则返回 `-1` 。

这道题是想找出现过至少三次的最长子字符串，且字符串是由相同字符组成

思路：使用一个map将每个字符出现的连续长度进行记录，比如abbba，会记录a有两个1，b有一个3，当进行统计的时候，统计长度为2的字符的时候b就有3-2+1个，再记录长度之后就可以挨个对每个字符统计每个长度出现的次数，找到出现次数大于等于3中长度最长的即可

这里可以用二分法进行统计，这个二分的是长度，因为对于子字符串来说，如果要满足出现次数大于3，其长度区间是1~n-2，这样用二分法每次找一个mid长度，来看每个字符中大于等于mid的count是否大于3，如果大于就可以更新结果，并且再增长长度看是否满足，如果小于就减少长度

```java
class Solution {
    public int maximumLength(String s) {
        int n=s.length();
        Map<Character,List<Integer>>map=new HashMap<>();
        for(int i=0,j=0;i<n;i=j){
            while(j<n&&s.charAt(i)==s.charAt(j)){
                j++;
            }
            map.computeIfAbsent(s.charAt(i),k->new ArrayList()).add(j-i);
        }
        int res=-1;
        for(List<Integer>list:map.values()){
            //特殊字符串长度区间1~n-2
            int short_length=1,long_length=n-2;   
            while(short_length<=long_length){
                int count=0;
                int mid=(short_length+long_length)>>1;
                //遍历某一个字符的特殊串集合，找大于mid长度的个数
                for(int l:list){
                    if(l>=mid){
                        count+=l-mid+1;
                    }
                }
                //如果个数大于3，就可以记录这个长度
                if(count>=3){
                    res=Math.max(res,mid);
                    short_length=mid+1;
                }else{
                    long_length=mid-1;
                }      
            }
        }
        return res;
    }
}
```

###### [寻找峰值](https://leetcode.cn/problems/find-peak-element/)
峰值元素是指其值严格大于左右相邻值的元素。

给你一个整数数组 `nums`，找到峰值元素并返回其索引。数组可能包含多个峰值，在这种情况下，返回 任何一个峰值 所在位置即可。

思路：找山峰峰值，因为左右两边-1，length位置都是最小值，所以只要找单调递增的一边，就一定能找到峰值

```java
class Solution {
    public int findPeakElement(int[] nums) {
        int left=0,right=nums.length-1;
        while(left<=right){
            int mid=(left+right)/2;
            //处理命中
            if(compare(nums,mid-1,mid)<0&&compare(nums,mid,mid+1)>0)
                return mid;
            //处理非命中
            //判断分区
            if(compare(nums,mid,mid+1)<0){
                left=mid+1;
            }else{
                right=mid-1;
            }
        }
        return -1;
    }

    public int[]get(int []num,int index){
        if(index==-1||index==num.length)
            return new int[]{0,0};
        return new int[]{1,num[index]};
    }

    public int compare(int []nums,int index1,int index2){
        int[]num1=get(nums,index1);
        int []num2=get(nums,index2);
        if(num1[0]!=num2[0]){
            return num1[0]>num2[0]?1:-1;
        }
        if(num1[1]==num2[1])
            return 0;
        return num1[1]>num2[1]?1:-1;
    }
}
```

###### [山脉数组的峰顶索引](https://leetcode.cn/problems/peak-index-in-a-mountain-array/)
这道题题意说明了峰值是在数组中间，不会在边界处，所以在进行边界判断的时候直接越过，上道题就需要额外判断边界，使用compare方法

```java
class Solution {
    public int peakIndexInMountainArray(int[] arr) {
        int left=0,right=arr.length-1;
        while(left<=right){
            int mid=left+(right-left)/2;
            if(mid==0)
                left=mid+1;
            else if(mid==arr.length-1)
                right=mid-1;
            //处理命中
            else if(arr[mid]>arr[mid-1]&&arr[mid]>arr[mid+1])
                return mid;
            //处理非命中
            //寻找分区
            else if(arr[mid]<arr[mid+1]){
                left=mid+1;
            }else{
                right=mid-1;
            }
        }
        return -1;
    }
}
```

###### [稀疏数组搜索](https://leetcode.cn/problems/sparse-array-search-lcci/)
难点在于当索引到空字符串的时候，不知道往哪边移动，这个时候就可以用边界来判断，对边界进行移动

```java
class Solution {
    public int findString(String[] words, String s) {
        int left=0,right=words.length-1;
        while(left<=right){
            int mid=left+(right-left)/2;
            if(words[mid].equals(s)){
                return mid;
            }else if(words[mid].equals("")){
                //因为当位空的时候不知道往哪边动，所以用边界判断，让边界动，这样整体就又动起来了
                if(words[left].equals(s))
                    return left;
                else
                    left++;
            }else if(words[mid].compareTo(s)<0){
                left=mid+1;
            }else{
                right=mid-1;
            }
        }
        return -1;
    }
}
```

###### [x 的平方根](https://leetcode.cn/problems/sqrtx/)
这是要求整数版

```java
class Solution {
    public int mySqrt(int x) {
        if(x==0)
            return 0;
        int left=0,riht=x/2+1;
        while(left<=riht){
            int mid=left+(riht-left)/2;
            long product=(long)mid*mid;
            if(product==x)
                return mid;
            else if(product>x){
                riht=mid-1;
            }else{
                //存在
                long product2=(long)(mid+1)*(mid+1);
                if(product2<=x)
                    left=mid+1;
                else
                    return mid;
            }
        }
        return -1;
    }
}
```

如果是要求平方根，并且要求保留5位小数

```java
public class Solution {

    public double mySqrt(double x) {
        if (x == 0) return 0;
        double left = 0, right = x;
        double epsilon = 0.000001; // 设置精度阈值
        
        if (x < 1) {
            right = 1;
        }
        
        while (right - left > epsilon) {
            double mid = left + (right - left) / 2;
            double square = mid * mid;
            if (Math.abs(square - x) < epsilon) {
                return mid; // 当差距小于epsilon时返回结果
            } else if (square < x) {
                left = mid;
            } else {
                right = mid;
            }
        }
        return left + (right - left) / 2; // 取中值作为最终结果
    }
    
}

```

###### [有效的完全平方数](https://leetcode.cn/problems/valid-perfect-square/description/)
```java
class Solution {
    public boolean isPerfectSquare(int num) {
        int left=0,right=num/2+1;
        while(left<=right){
            int mid=left+(right-left)/2;
            long product=(long)mid*mid;
            if(product==num)
                return true;
            else if(product<num){
                long product2=(long)(mid+1)*(mid+1);
                if(product2>num)
                    return false;
                else
                    left=mid+1;
            }else{
                right=mid-1;
            }
        }
        return false;
    }
}
```

##### 田忌赛马
###### [优势洗牌](https://leetcode.cn/problems/advantage-shuffle/)
给定两个长度相等的数组 `nums1` 和 `nums2`，`nums1` 相对于 `nums2` 的优势可以用满足 `nums1[i] > nums2[i]` 的索引 `i` 的数目来描述。

返回 nums1 的任意排列，使其相对于 `nums2` 的优势最大化

类似于田忌赛马，思路是首先对两个数组进行排序，不过对于num2来说，不能改变序列位置，不然就没有意义了，所以定义了一个对象，把他每个值进行排序，但又记录了每个值在原数组的位置，然后直接对num1数组进行排序，比较的时候，挨个取出队列中的值，因为是从大到小进行排序，也从大拿出num1的数组进行判断，如果num1的大，说明可以赢就直接讲num1该位置的值存储在新数组中num2的值对应的位置，如果更小，那就从num2中提取当前最小值进行添加，知道num1的值全部匹配完

```java
class Solution {
    int[] advantageCount(int[] nums1, int[] nums2) {
        PriorityQueue<int []>queue=new PriorityQueue<>(
            (int []p1,int []p2)->{
                return p2[1]-p1[1];
            }
        );
        for(int i=0;i<nums2.length;i++){
            queue.offer(new int[]{i,nums2[i]});
        }
        Arrays.sort(nums1);
        int []res=new int[nums1.length];
        int left=0;int right=nums1.length-1;
        while(!queue.isEmpty()){
            int []p=queue.poll();
            int i=p[0];int num=p[1];
            if(num<nums1[right]){
                res[i]=nums1[right];
                right--;
            }else{
                res[i]=nums1[left];
                left++;
            }
        }
        return res;
    }

}
```

##### 单调栈
###### [移掉 K 位数字](https://leetcode.cn/problems/remove-k-digits/)
给你一个以字符串表示的非负整数 `num` 和一个整数 `k` ，移除这个数中的k位数字，使得剩下的数字最小。请你以字符串形式返回这个最小的数字。

```java
class Solution {
    public String removeKdigits(String num, int k) {
        //Deque<Character> 被用来存储最终保留的数字。
        //这种数据结构支持从两端高效地添加和移除元素，非常适合这个问题，
        //因为需要频繁地在队列尾部添加元素，同时可能在尾部移除不利于最小值生成的元素
        Deque<Character> deque = new LinkedList<Character>();
        int length = num.length();
        for (int i = 0; i < length; ++i) {
            //在添加每个新字符到队列之前，如果队列非空且还需要删除更多字符 (k > 0)
            //并且队列尾部的字符大于当前字符，那么从队列尾部移除字符。
            //这一步是贪心算法的体现，确保结果字符串字典序尽可能小
            char digit = num.charAt(i);
            while (!deque.isEmpty() && k > 0 && deque.peekLast() > digit) {
                deque.pollLast();
                k--;
            }
            deque.offerLast(digit);
        }
        
        for (int i = 0; i < k; ++i) {
            //如果循环结束后仍然有剩余的 k（即没有删除足够的字符）
            //则从队列尾部继续移除字符，直到移除 k 个为止
            deque.pollLast();
        }
        
        StringBuilder ret = new StringBuilder();
        boolean leadingZero = true;
        while (!deque.isEmpty()) {
            char digit = deque.pollFirst();
            //忽略前导零：如果遇到零并且当前是序列的开始，则跳过这些零。
            //但一旦遇到非零数字，后续的零也应包含在结果中
            if (leadingZero && digit == '0') {
                continue;
            }
            leadingZero = false;
            ret.append(digit);
        }
        return ret.length() == 0 ? "0" : ret.toString();
    }
}

```

###### [拼接最大数](https://leetcode.cn/problems/create-maximum-number/)
给你两个整数数组 `nums1` 和 `nums2`，它们的长度分别为 `m` 和 `n`。数组 `nums1` 和 `nums2` 分别代表两个数各位上的数字。同时你也会得到一个整数 `k`。

请你利用这两个数组中的数字中创建一个长度为 `k <= m + n` 的最大数，在这个必须保留来自同一数组的数字的相对顺序。	

思路：通过模拟的方式求解，即每次从nums1中去i个最大子序列，从nums2中去k-i个最大子序列，然后对这两个子序列进行合并，不断更新最大值

```java
class Solution {
    public int[] maxNumber(int[] nums1, int[] nums2, int k) {
        int m = nums1.length, n = nums2.length;
        int[] maxSubsequence = new int[k];
        int start = Math.max(0, k - n), end = Math.min(k, m);
        for (int i = start; i <= end; i++) {
            int[] subsequence1 = maxSubsequence(nums1, i);
            int[] subsequence2 = maxSubsequence(nums2, k - i);
            int[] curMaxSubsequence = merge(subsequence1, subsequence2);
            if (compare(curMaxSubsequence, 0, maxSubsequence, 0) > 0) {
                System.arraycopy(curMaxSubsequence, 0, maxSubsequence, 0, k);
            }
        }
        return maxSubsequence;
    }
    //通过最大栈的方式求k个元素的最大子序列
    public int[] maxSubsequence(int[] nums, int k) {
        int length = nums.length;
        int[] stack = new int[k];
        int top = -1;
        //表示还可以跳过元素数量
        int remain = length - k;
        for (int i = 0; i < length; i++) {
            int num = nums[i];
            //对于子序列来说 321 是大于233的 所以遇到大的值就尽量放到最前面
            while (top >= 0 && stack[top] < num && remain > 0) {
                top--;
                remain--;
            }
            
            if (top < k - 1) {
                //添加元素 可跳过元素数量不变
                stack[++top] = num;
            } else {
                //不添加元素 说明可跳过元素数量减1
                remain--;
            }
        }
        return stack;
    }
    //通过双指针方式判断大小进行合并
    public int[] merge(int[] subsequence1, int[] subsequence2) {
        int x = subsequence1.length, y = subsequence2.length;
        if (x == 0) {
            return subsequence2;
        }
        if (y == 0) {
            return subsequence1;
        }
        int mergeLength = x + y;
        int[] merged = new int[mergeLength];
        int index1 = 0, index2 = 0;
        for (int i = 0; i < mergeLength; i++) {
            if (compare(subsequence1, index1, subsequence2, index2) > 0) {
                merged[i] = subsequence1[index1++];
            } else {
                merged[i] = subsequence2[index2++];
            }
        }
        return merged;
    }
    //判断大小 
    public int compare(int[] subsequence1, int index1, int[] subsequence2, int index2) {
        int x = subsequence1.length, y = subsequence2.length;
        while (index1 < x && index2 < y) {
            int difference = subsequence1[index1] - subsequence2[index2];
            if (difference != 0) {
                return difference;
            }
            index1++;
            index2++;
        }
        return (x - index1) - (y - index2);
    }
}
```

###### [找出最具竞争力的子序列](https://leetcode.cn/problems/find-the-most-competitive-subsequence/)
思路：这道题是要求数组的子序列，这个子序列长度为k，在子序列 `a` 和子序列 `b` 第一个不相同的位置上，如果 `a` 中的数字小于 `b` 中对应的数字，那么我们称子序列 `a` 比子序列 `b`（相同长度下）更具 **竞争力** 。

最开始的思路是：每一次求一个位置的最小值，比如求res的第一个位置，就对原数组的0-n-k+i的范围内进行查找最小值，下一次就是从最小值索引的下一个开始，这个方法可以做，但时间复杂度太高了

这里的思路是单调栈的思路，遍历一遍原数组，找到原数组每个元素应该在的位置，通过一个指针m进行定位，并且m满足一些要求，比如说比较元素的值的位置，不能太靠前，避免后面的值填不够通过nums.length+m-i-k>0来约束，用m<k来判断是为了最开始把数组填满，后面那些不满足条件的值就不能添加进去

```java
class Solution {
    public int[] mostCompetitive(int[] nums, int k) {
        int []res=new int[k];
        int m=0;
        for(int i=0;i<nums.length;i++){
            int n=nums[i];
            while(m>0&&res[m-1]>n&&nums.length+m-i-k>0){
                m--;
            }
            if(m<k)
                res[m++]=n;
        }
        return res;
    }

}
```



### 二叉树
###### [相同的树](https://leetcode.cn/problems/same-tree/)
判断两个树是否相同

```java
/**
 * Definition for a binary tree node.
 * public class TreeNode {
 *     int val;
 *     TreeNode left;
 *     TreeNode right;
 *     TreeNode() {}
 *     TreeNode(int val) { this.val = val; }
 *     TreeNode(int val, TreeNode left, TreeNode right) {
 *         this.val = val;
 *         this.left = left;
 *         this.right = right;
 *     }
 * }
 */
class Solution {
    public boolean isSameTree(TreeNode p, TreeNode q) {
        if(p==null&&q==null)
            return true;
        else if(p!=null&&q!=null){
            if(p.val==q.val){
                return isSameTree(p.left,q.left)&&isSameTree(p.right,q.right);
            }else
                return false;
        }else
            return false;
    }

}
```

###### [翻转二叉树](https://leetcode.cn/problems/invert-binary-tree/)
反转二叉树

```java
/**
 * Definition for a binary tree node.
 * public class TreeNode {
 *     int val;
 *     TreeNode left;
 *     TreeNode right;
 *     TreeNode() {}
 *     TreeNode(int val) { this.val = val; }
 *     TreeNode(int val, TreeNode left, TreeNode right) {
 *         this.val = val;
 *         this.left = left;
 *         this.right = right;
 *     }
 * }
 */
class Solution {
    public TreeNode invertTree(TreeNode root) {
        if(root==null)
            return null;
        TreeNode temp=root.left;
        root.left=root.right;
        root.right=temp;
        invertTree(root.left);
        invertTree(root.right);
        return root;
    }
}
```

###### [二叉树的最大深度](https://leetcode.cn/problems/maximum-depth-of-binary-tree/)
```java
class Solution {
    public int maxDepth(TreeNode root) {
        if(root==null){
            return 0;
        }
        int left=maxDepth(root.left);
        int right=maxDepth(root.right);
        int res=Math.max(left,right)+1;
        return res;
    }
}
```

###### [二叉树的最小深度](https://leetcode.cn/problems/minimum-depth-of-binary-tree/)
方法1️⃣：用dfs，遍历每个叶子结点找最短路径

```java
/**
 * Definition for a binary tree node.
 * public class TreeNode {
 *     int val;
 *     TreeNode left;
 *     TreeNode right;
 *     TreeNode() {}
 *     TreeNode(int val) { this.val = val; }
 *     TreeNode(int val, TreeNode left, TreeNode right) {
 *         this.val = val;
 *         this.left = left;
 *         this.right = right;
 *     }
 * }
 */
class Solution {
    int min_value=Integer.MAX_VALUE;
    public int minDepth(TreeNode root) {
        if(root==null)
            return 0;
        min(root,1);
        return min_value;
    }
    public void min(TreeNode root,int count){
        if(root.left==null&&root.right==null){
            if(min_value>count){
                min_value=count;
            }
        }
        if(root.left!=null)
            min(root.left,count+1);
        if(root.right!=null)
            min(root.right,count+1);
    }
}
```

###### 按之字形顺序打印二叉树
```java
public ArrayList<ArrayList<Integer> > Print(TreeNode pRoot) {
        ArrayList res=new ArrayList();
        if(pRoot==null)
            return res;
        Queue<TreeNode> q=new LinkedList<TreeNode>();
        Boolean flag=true;
        q.add(pRoot);
        while(!q.isEmpty()){
            int size=q.size();
            ArrayList a=new ArrayList();
            while(size>0){
                size--;
                TreeNode temp=q.peek();q.remove();
                if(flag==true)//用于翻转
                    a.add(temp.val);
                else
                    a.add(0,temp.val);
                if(temp.left!=null)q.add(temp.left);
                if(temp.right!=null)q.add(temp.right);
            }
            res.add(a);
            flag=!flag;
        }
        return res;
    }
```

###### [路径总和](https://leetcode.cn/problems/path-sum/)
判断该树中是否存在 根节点到叶子节点 的路径，这条路径上所有节点值相加等于目标和 `targetSum`

思路：本来加了两个if包括判断如果sum>target和相等时如果不是叶子节点就可以直接剪枝返回，但因为节点值有负数，所以两个优化都不能用

```java
/**
 * Definition for a binary tree node.
 * public class TreeNode {
 * int val;
 * TreeNode left;
 * TreeNode right;
 * TreeNode() {}
 * TreeNode(int val) { this.val = val; }
 * TreeNode(int val, TreeNode left, TreeNode right) {
 * this.val = val;
 * this.left = left;
 * this.right = right;
 * }
 * }
 */
class Solution {
    int sum = 0;

    public boolean hasPathSum(TreeNode root, int targetSum) {
        if(root==null)
            return false;
        sum+=root.val;
        if(sum==targetSum){
            if(root.left==null&&root.right==null)
                return true;
        }
        if(hasPathSum(root.left,targetSum)||hasPathSum(root.right,targetSum))
            return true;
        sum-=root.val;
        return false;
        
    }

}
```

###### [路径总和 III](https://leetcode.cn/problems/path-sum-iii/)
不需要从根节点开始，也不需要到叶子结点才算结束，统计到targetSum的路径

```java
/**
 * Definition for a binary tree node.
 * public class TreeNode {
 *     int val;
 *     TreeNode left;
 *     TreeNode right;
 *     TreeNode() {}
 *     TreeNode(int val) { this.val = val; }
 *     TreeNode(int val, TreeNode left, TreeNode right) {
 *         this.val = val;
 *         this.left = left;
 *         this.right = right;
 *     }
 * }
 */
class Solution {
    int count=0;
    public int pathSum(TreeNode root, int targetSum) {
        if(root==null)
            return 0;
        Queue<TreeNode>queue=new LinkedList<>();
        queue.add(root);
        while(!queue.isEmpty()){
            int size=queue.size();
            for(int i=0;i<size;i++){
                TreeNode node=queue.poll();
                sum(node,targetSum,0);
                if(node.left!=null)queue.add(node.left);
                if(node.right!=null)queue.add(node.right);
            }
        }
        return count;
    }
    public void sum(TreeNode node,int targetSum,long sum){
        if(node==null)
            return;
        sum+=node.val;
        if(sum==targetSum)
            count++;
        sum(node.left,targetSum,sum);
        sum(node.right,targetSum,sum);
    }
}
```

###### [求根节点到叶节点数字之和](https://leetcode.cn/problems/sum-root-to-leaf-numbers/)
计算从根节点到叶节点生成的 所有数字之和

```java
/**
 * Definition for a binary tree node.
 * public class TreeNode {
 *     int val;
 *     TreeNode left;
 *     TreeNode right;
 *     TreeNode() {}
 *     TreeNode(int val) { this.val = val; }
 *     TreeNode(int val, TreeNode left, TreeNode right) {
 *         this.val = val;
 *         this.left = left;
 *         this.right = right;
 *     }
 * }
 */
class Solution {
    int sum=0;
    int tsum=0;
    public int sumNumbers(TreeNode root) {
        sum(root);
        return sum;
    }
    public void sum(TreeNode node){
        if(node==null){
            return;
        }
        tsum=tsum*10+node.val;
        if(node.left==null&&node.right==null){
            sum+=tsum;
            tsum/=10;
            return;
        }
        sum(node.left);
        sum(node.right);
        tsum/=10;
        return;
    }
}
```

###### [二叉树中和为目标值的路径](https://leetcode.cn/problems/er-cha-shu-zhong-he-wei-mou-yi-zhi-de-lu-jing-lcof/)
找出所有从根节点到叶子节点路径总和等于给定目标和的路径

```java
/**
 * Definition for a binary tree node.
 * public class TreeNode {
 *     int val;
 *     TreeNode left;
 *     TreeNode right;
 *     TreeNode() {}
 *     TreeNode(int val) { this.val = val; }
 *     TreeNode(int val, TreeNode left, TreeNode right) {
 *         this.val = val;
 *         this.left = left;
 *         this.right = right;
 *     }
 * }
 */
class Solution {
    List<List<Integer>>res=new ArrayList<>();
    List<Integer>list=new ArrayList<>();
    public List<List<Integer>> pathTarget(TreeNode root, int target) {
        if(root==null)
            return res;
        dfs(root,target,0);
        return res;

    }
    public void dfs(TreeNode node,int target,int cur){
        cur+=node.val;
        list.add(node.val);
        if(node.left==null&&node.right==null){
            if(cur==target){
                res.add(new ArrayList(list));
            }
            list.remove(list.size()-1);
            return;
        }
        if(node.left!=null)
            dfs(node.left,target,cur);
        if(node.right!=null)
            dfs(node.right,target,cur);
        list.remove(list.size()-1);

    }

}
```

###### [二叉树的直径](https://leetcode.cn/problems/diameter-of-binary-tree/)
二叉树直径

思路：直径的长度其实是等于一个节点的左右子树深度之和，所以利用在求每个节点的最大深度时，进行不断的判断，就能找到最长直径了

```java
class Solution {
    int ans;
    public int diameterOfBinaryTree(TreeNode root) {
        ans=0;
        depth(root);
        return ans;
    }
    public int depth(TreeNode node){
        if(node==null)
            return 0;
        int left=depth(node.left);
        int right=depth(node.right);
        ans=Math.max(ans,left+right);
        return Math.max(left,right)+1;
    }
}
```

###### [ 二叉树中的最大路径和](https://leetcode.cn/problems/jC7MId/)
二叉树最大路径和

```java
/**
 * Definition for a binary tree node.
 * public class TreeNode {
 *     int val;
 *     TreeNode left;
 *     TreeNode right;
 *     TreeNode() {}
 *     TreeNode(int val) { this.val = val; }
 *     TreeNode(int val, TreeNode left, TreeNode right) {
 *         this.val = val;
 *         this.left = left;
 *         this.right = right;
 *     }
 * }
 */
class Solution {
    int maxSum=Integer.MIN_VALUE;
    public int maxPathSum(TreeNode root) {
        maxGain(root);
        return maxSum;
    }
    public int maxGain(TreeNode node){
        if(node==null)
            return 0;
        int leftGain=Math.max(maxGain(node.left),0);
        int rightGain=Math.max(maxGain(node.right),0);
        int price=node.val+leftGain+rightGain;
        maxSum=Math.max(maxSum,price);
        return node.val+Math.max(leftGain,rightGain);
    }
}
```



###### [二叉树展开为链表](https://leetcode.cn/problems/flatten-binary-tree-to-linked-list/)
二叉树转链表

```java
class Solution {
    List <TreeNode>list=new ArrayList<TreeNode>();
    public void flatten(TreeNode root) {
        pre(root);
        TreeNode temp=root;
        for(int i=1;i<list.size();i++){
            temp.right=list.get(i);
            temp.left=null;
            temp=temp.right;
        }
    }
    public void pre(TreeNode node){
        if(node==null)
            return;
        list.add(node);
        pre(node.left);
        pre(node.right);
    }
}
```

###### [填充每个节点的下一个右侧节点指针](https://leetcode.cn/problems/populating-next-right-pointers-in-each-node/)
填充它的每个 next 指针，让这个指针指向其下一个右侧节点。如果找不到下一个右侧节点，则将 next 指针设置为 `NULL`。

为每个节点增加右指针，使用bfs

```java
class Solution {
    public Node connect(Node root) {
        if(root==null){
            return null;
        }
        Queue<Node>queue=new LinkedList<>();
        queue.add(root);
        while(!queue.isEmpty()){
            int size=queue.size();
            for(int i=0;i<size;i++){
                Node node=queue.poll();
                if(i<size-1){
                    node.next=queue.peek();
                }
                if(node.left!=null)
                    queue.add(node.left);
                if(node.right!=null)
                    queue.add(node.right);
            }
            
        }
        return root;
    }
}
```

dfs

```java
class Solution {
    public Node connect(Node root) {
        if(root==null){
            return null;
        }
        add(root.left,root.right);
        return root;
    }
    public void add(Node node,Node node2){
        if(node==null)
            return;
        node.next=node2;
        add(node.left,node.right);
        add(node2.left,node2.right);
        add(node.right,node2.left);
    }
}
```

###### [填充每个节点的下一个右侧节点指针 II](https://leetcode.cn/problems/populating-next-right-pointers-in-each-node-ii/)
思路：和上一题的区别是上一题是完美二叉树，node2就不需要判断null，这道题不行，所以使用bfs来做

```java
/*
// Definition for a Node.
class Node {
    public int val;
    public Node left;
    public Node right;
    public Node next;

    public Node() {}
    
    public Node(int _val) {
        val = _val;
    }

    public Node(int _val, Node _left, Node _right, Node _next) {
        val = _val;
        left = _left;
        right = _right;
        next = _next;
    }
};
*/

class Solution {
    public Node connect(Node root) {
        if(root==null)
            return null;
        Queue<Node>queue=new LinkedList<>();
        queue.add(root);
        int sz=0;
        while(!queue.isEmpty()){
            Node node=queue.poll();
            sz=queue.size();
            if(node.left!=null)queue.add(node.left);
            if(node.right!=null)queue.add(node.right);
            for(int i=0;i<sz;i++){
                Node node1=queue.poll();
                if(node1.left!=null)queue.add(node1.left);
                if(node1.right!=null)queue.add(node1.right);
                node.next=node1;
                node=node1;
            }
        }
        return root;
    }
}
```



###### [从前序与中序遍历序列构造二叉树](https://leetcode.cn/problems/construct-binary-tree-from-preorder-and-inorder-traversal/)
前序中序构造

```java
class Solution {
    public TreeNode buildTree(int[] preorder, int[] inorder) {
        TreeNode root=build(preorder,0,preorder.length-1,inorder,0,inorder.length-1);
        return root;
    }
    public TreeNode build(int []preorder,int pl,int pr,int []inorder,int il,int ir){
        if(pl>pr)
            return null;
        int tar=preorder[pl];
        int index=0;
        for(int i=il;i<=ir;i++){
            if(inorder[i]==tar){
                index=i;
                break;
            }
        }
        int size=index-il;
        TreeNode node=new TreeNode(tar);
        node.left=build(preorder,pl+1,pl+size,inorder,il,index-1);
        node.right=build(preorder,pl+size+1,pr,inorder,index+1,ir);
        return node;
    }
}
```

###### [从中序与后序遍历序列构造二叉树](https://leetcode.cn/problems/construct-binary-tree-from-inorder-and-postorder-traversal/)
中序后序构造

```java
/**
 * Definition for a binary tree node.
 * public class TreeNode {
 *     int val;
 *     TreeNode left;
 *     TreeNode right;
 *     TreeNode() {}
 *     TreeNode(int val) { this.val = val; }
 *     TreeNode(int val, TreeNode left, TreeNode right) {
 *         this.val = val;
 *         this.left = left;
 *         this.right = right;
 *     }
 * }
 */
class Solution {
    public TreeNode buildTree(int[] inorder, int[] postorder) {
        TreeNode root=build(inorder,postorder,0,inorder.length-1,0,postorder.length-1);
        return root;
    }
    public TreeNode build(int []inorder,int []postorder,int inl,int inr,int pl,int pr){
        if(pl>pr)
            return null;
        int index=0;
        int target=postorder[pr];
        for(int i=inl;i<=inr;i++){
            if(inorder[i]==target){
                index=i-inl;
                break;
            }
        }
        TreeNode node=new TreeNode(target);
        node.left=build(inorder,postorder,inl,inl+index-1,pl,pl+index-1);
        node.right=build(inorder,postorder,inl+index+1,inr,pl+index,pr-1);
        return node;
    }
}
```

###### [二叉搜索树的最小绝对差](https://leetcode.cn/problems/minimum-absolute-difference-in-bst/)
给你一个二叉搜索树的根节点 `root` ，返回 树中任意两不同节点值之间的最小差值

```java
/**
 * Definition for a binary tree node.
 * public class TreeNode {
 *     int val;
 *     TreeNode left;
 *     TreeNode right;
 *     TreeNode() {}
 *     TreeNode(int val) { this.val = val; }
 *     TreeNode(int val, TreeNode left, TreeNode right) {
 *         this.val = val;
 *         this.left = left;
 *         this.right = right;
 *     }
 * }
 */
class Solution {
    public int getMinimumDifference(TreeNode root) {
        list=new ArrayList<>();
        inorder(root);
        int min=Integer.MAX_VALUE;
        for(int i=0;i<list.size()-1;i++){
            if(list.get(i+1)-list.get(i)<min)
                min=list.get(i+1)-list.get(i);
        }
        return min;
    }
    List<Integer>list;
    public void inorder(TreeNode node){
        if(node==null)
            return;
        inorder(node.left);
        list.add(node.val);
        inorder(node.right);
    }
}
```

###### [将有序数组转换为二叉搜索树](https://leetcode.cn/problems/convert-sorted-array-to-binary-search-tree/)
```java
/**
 * Definition for a binary tree node.
 * public class TreeNode {
 *     int val;
 *     TreeNode left;
 *     TreeNode right;
 *     TreeNode() {}
 *     TreeNode(int val) { this.val = val; }
 *     TreeNode(int val, TreeNode left, TreeNode right) {
 *         this.val = val;
 *         this.left = left;
 *         this.right = right;
 *     }
 * }
 */
class Solution {
    public TreeNode sortedArrayToBST(int[] nums) {
        return dfs(nums,0,nums.length-1);
    }

    public TreeNode dfs(int []nums,int i,int j){
        if(i>j)
            return null;
        int mid=(i+j)>>1;
        TreeNode node=new TreeNode(nums[mid]);
        node.left=dfs(nums,i,mid-1);
        node.right=dfs(nums,mid+1,j);
        return node;
    }
}
```

###### [将二叉搜索树变平衡](https://leetcode.cn/problems/balance-a-binary-search-tree/)
先中序遍历获取有序队列，然后再重新建树

```java
class Solution {
    List<Integer>list=new LinkedList<>();
    public TreeNode balanceBST(TreeNode root) {
        inorder(root);
        return balance(0,list.size()-1);
    }
    public TreeNode balance(int i,int j){
        if(i>j)
            return null;
        int mid=(i+j)>>1;
        TreeNode node=new TreeNode(list.get(mid));
        node.left=balance(i,mid-1);
        node.right=balance(mid+1,j);
        return node;
    }
    public void inorder(TreeNode root){
        if(root==null)
            return;
        inorder(root.left);
        list.add(root.val);
        inorder(root.right);
    }
}
```

###### [验证二叉搜索树的后序遍历序列](https://leetcode.cn/problems/er-cha-sou-suo-shu-de-hou-xu-bian-li-xu-lie-lcof/)
按照后序的规则进行判断，最后一个值是头节点，找到头节点后，验证前面的部分是否都是一部分连续的更小，另一部分更大

```java
class Solution {
    public boolean verifyTreeOrder(int[] postorder) {
        if(postorder.length==0||postorder==null)
            return true;
        return verify(postorder,0,postorder.length-1);
    }
    public boolean verify(int []postorder,int first,int last){
        if(first>=last)
            return true;
        int cutpoint=first;
        //传递的是数组所以需要先找头节点
        while(cutpoint<last&&postorder[cutpoint]<postorder[last])
            cutpoint++;
        for(int i=cutpoint;i<last;i++){
            if(postorder[i]<postorder[last])
                return false;
        }
        return verify(postorder,first,cutpoint-1)&&verify(postorder,cutpoint,last-1);
    }
}
```

###### [验证二叉搜索树](https://leetcode.cn/problems/validate-binary-search-tree/)
```java
/**
 * Definition for a binary tree node.
 * public class TreeNode {
 *     int val;
 *     TreeNode left;
 *     TreeNode right;
 *     TreeNode() {}
 *     TreeNode(int val) { this.val = val; }
 *     TreeNode(int val, TreeNode left, TreeNode right) {
 *         this.val = val;
 *         this.left = left;
 *         this.right = right;
 *     }
 * }
 */
class Solution {
    public boolean isValidBST(TreeNode root) {
        if(root==null)
            return false;
        return isvalid(root,Long.MIN_VALUE,Long.MAX_VALUE);
    }
    public boolean isvalid(TreeNode node,long lowwer,long upper){
        if(node==null)
            return true;
        if(node.val<=lowwer||node.val>=upper)
            return false;
        return isvalid(node.left,lowwer,node.val)&&isvalid(node.right,node.val,upper);
    }
}
```

###### [最大二叉树](https://leetcode.cn/problems/maximum-binary-tree/)
数组构建最大二叉树

```java
class Solution {
    public TreeNode constructMaximumBinaryTree(int[] nums) {
        if(nums.length==0)
            return null;
        TreeNode node=build(nums,0,nums.length-1);
        return node;
    }
    public TreeNode build(int []nums,int left,int right){
        if(left>right)
            return null;
        int max=-1;
        int index=0;
        for(int i=left;i<=right;i++){
            if(nums[i]>max){
                max=nums[i];
                index=i;
            }
        }
        TreeNode root=new TreeNode(max);
        root.left=build(nums,left,index-1);
        root.right=build(nums,index+1,right);
        return root;
    }
}
```

###### [从二叉搜索树到更大和树](https://leetcode.cn/problems/binary-search-tree-to-greater-sum-tree/)
给定一个二叉搜索树 `root` (BST)，请将它的每个节点的值替换成树中大于或者等于该节点值的所有节点值之和。

提醒一下， 二叉搜索树 满足下列约束条件：

+ 节点的左子树仅包含键 小于 节点键的节点。
+ 节点的右子树仅包含键 大于 节点键的节点。
+ 左右子树也必须是二叉搜索树。

二叉搜索树用的中序遍历

```java
class Solution {
    public TreeNode bstToGst(TreeNode root) {
        add(root);
        return root;
    }
    int res=0;
    public void add(TreeNode node){
        if(node==null)
            return;
        add(node.right);
        res+=node.val;
        node.val=res;
        add(node.left);
    }
}
```

###### [二叉搜索树中第K小的元素](https://leetcode.cn/problems/kth-smallest-element-in-a-bst/)
二叉树中第k小的元素

方法1️⃣：和上一题一样的思路，只不过是正向的

```java
class Solution {
    public int kthSmallest(TreeNode root, int k) {
        search(root,k);
        return res;
    }
    int res;
    int rank=0;
    public void search(TreeNode node,int k){
        if(node==null)
            return;
        search(node.left,k);
        rank++;
        if(k==rank){
            res=node.val;
            return;
        }
        search(node.right,k);

    }
}
```

方法2️⃣：借助set的自动排序进行求解

```java
public int KthNode (TreeNode proot, int k) {
        // write code here
        if(proot==null||k==0)
            return -1;
        Queue<TreeNode>queue=new LinkedList<TreeNode>();
        HashSet<Integer>set=new HashSet<Integer>();
        queue.add(proot);
        while(!queue.isEmpty()){
            int size=queue.size();
            while(size>0){
                size--;
                TreeNode node=queue.poll();
                set.add(node.val);
                if(node.left!=null)queue.add(node.left);
                if(node.right!=null)queue.add(node.right);
            }
        }
        if(set.size()<k)
            return -1;
        int num=0;
        for(Integer i:set){
            if(num==k-1){
                num=i;
                break;
            }
            num++;
        }
        return num;
    }
```

###### [把二叉搜索树转换为累加树](https://leetcode.cn/problems/convert-bst-to-greater-tree/)
给出二叉搜索树的根节点，该树的节点值各不相同，请你将其转换为累加树（Greater Sum Tree），使每个节点 `node` 的新值等于原树中大于或等于 `node.val` 的值之和。

```java
class Solution {
    public TreeNode convertBST(TreeNode root) {
        sum(root);
        return root;
    }
    int res;
    public void sum(TreeNode node){
        if(node ==null)
            return;
        sum(node.right);
        res+=node.val;
        node.val=res;
        sum(node.left);
    }
}
```

###### [将二叉搜索树转化为排序的双向链表](https://leetcode.cn/problems/er-cha-sou-suo-shu-yu-shuang-xiang-lian-biao-lcof/)
思路：因为要把二叉搜索树转化成双向链表，并且双向链表节点的大小是从小到大的，所以想到先利用中序遍历将二叉搜索树节点加入list数组，然后再挨个给list数组中各个节点重新设置left和right指针

```java
/*
// Definition for a Node.
class Node {
    public int val;
    public Node left;
    public Node right;

    public Node() {}

    public Node(int _val) {
        val = _val;
    }

    public Node(int _val,Node _left,Node _right) {
        val = _val;
        left = _left;
        right = _right;
    }
};
*/
class Solution {
    List<Node>res=new ArrayList<>();
    public Node treeToDoublyList(Node root) {
        if(root==null)
            return null;
        inorder(root);
        for(int i=0;i<res.size()-1;i++){
            res.get(i+1).left=res.get(i);
            res.get(i).right=res.get(i+1);
        }
        int last=res.size()-1;
        res.get(last).right=res.get(0);
        res.get(0).left=res.get(last);
        return res.get(0);
    }

    public void inorder(Node root){
        if(root==null)
            return;
        inorder(root.left);
        res.add(root);
        inorder(root.right);
    }
}
```

###### [完全二叉树的节点个数](https://leetcode.cn/problems/count-complete-tree-nodes/)
```java
class Solution {
    public int countNodes(TreeNode root) {
        if(root==null)
            return 0;
        return 1+countNodes(root.left)+countNodes(root.right);
    }
}
```

###### [子结构判断](https://leetcode.cn/problems/shu-de-zi-jie-gou-lcof/)
给定两棵二叉树`<font style="color:rgba(38, 38, 38, 0.75);">tree1</font>` 和 `<font style="color:rgba(38, 38, 38, 0.75);">tree2</font>`，判断 `<font style="color:rgba(38, 38, 38, 0.75);">tree2</font>` 是否以 `<font style="color:rgba(38, 38, 38, 0.75);">tree1</font>` 的某个节点为根的子树具有相同的结构和节点值 。  
注意，空树不会是以 `<font style="color:rgba(38, 38, 38, 0.75);">tree1</font>` 的某个节点为根的子树具有相同的结构和节点值

```java
/**
 * Definition for a binary tree node.
 * public class TreeNode {
 *     int val;
 *     TreeNode left;
 *     TreeNode right;
 *     TreeNode(int x) { val = x; }
 * }
 */
class Solution {
    public boolean isSubStructure(TreeNode A, TreeNode B) {
        if(A==null||B==null)
            return false;
        return judge(A,B)||isSubStructure(A.left,B)||isSubStructure(A.right,B); 
    }
    public boolean judge(TreeNode A,TreeNode B){
        if(A==null&&B==null)
            return true;
        if(A==null&&B!=null)
            return false;
        if(A!=null&&B==null)
            return true;
        if(A.val==B.val){
            return judge(A.left,B.left)&&judge(A.right,B.right);
        }else{
            return false;
        }
    }
}
```

###### **二叉树的镜像**
方法1️⃣：递归交换左右子树

```java
public TreeNode Mirror (TreeNode pRoot) {
        if(pRoot==null)
            return pRoot;
        TreeNode left=pRoot.left;
        TreeNode right=pRoot.right;
        pRoot.left=right;
        pRoot.right=left;
        Mirror(pRoot.left);
        Mirror(pRoot.right);
        return pRoot;
}
```

方法2️⃣：使用BFS的思路进行交换

```java
public TreeNode Mirror (TreeNode pRoot) {
            if(pRoot==null)
            return null;
        Queue<TreeNode> q=new LinkedList<TreeNode>();
        q.add(pRoot);
        while(!q.isEmpty()){
            int size=q.size();
            while(size>0){
                size--;
                TreeNode node=q.peek();q.remove();
                TreeNode temp=node.left;
                node.left=node.right;
                node.right=temp;
                if(node.left!=null)q.add(node.left);
                if(node.right!=null)q.add(node.right);
            }
        }
        return pRoot;
}
```

###### [判断是否为平衡二叉树](https://leetcode.cn/problems/ping-heng-er-cha-shu-lcof/)
输入一棵二叉树的根节点，判断该树是不是平衡二叉树。如果某二叉树中任意节点的左右子树的深度相差不超过1，那么它就是一棵平衡二叉树

```java
/**
 * Definition for a binary tree node.
 * public class TreeNode {
 *     int val;
 *     TreeNode left;
 *     TreeNode right;
 *     TreeNode() {}
 *     TreeNode(int val) { this.val = val; }
 *     TreeNode(int val, TreeNode left, TreeNode right) {
 *         this.val = val;
 *         this.left = left;
 *         this.right = right;
 *     }
 * }
 */
class Solution {
    public boolean isBalanced(TreeNode root) {
        if(root==null)
            return true;
        return Math.abs(depth(root.left)-depth(root.right))<2&&isBalanced(root.left)&&isBalanced(root.right);
    }
    public int depth(TreeNode node){
        if(node==null)
            return 0;
        int left=depth(node.left);
        int right=depth(node.right);
        return Math.max(left,right)+1;
    }
}
```

###### [对称二叉树](https://leetcode.cn/problems/symmetric-tree/)
给你一个二叉树的根节点 `root` ， 检查它是否轴对称

```java
class Solution {
    public boolean isSymmetric(TreeNode root) {
        if(root==null)
            return true;
        return judge(root.left,root.right);
    }
    public boolean judge(TreeNode r1,TreeNode r2){
        if(r1==null&&r2==null)
            return true;
        if(r1==null||r2==null)
            return false;
        if(r1.val==r2.val){
            return judge(r1.left,r2.right)&&judge(r1.right,r2.left);
        }
        return false;
    }
}
```

###### [二叉树的最近公共祖先](https://leetcode.cn/problems/lowest-common-ancestor-of-a-binary-tree/)
找公共祖先，考虑用map来记录每个节点和他的父节点，然后将一个节点及其祖先放到set中，然后查另一个节点和他的祖先看是否在set中存在，如果存在就有公共祖先

```java
/**
 * Definition for a binary tree node.
 * public class TreeNode {
 *     int val;
 *     TreeNode left;
 *     TreeNode right;
 *     TreeNode(int x) { val = x; }
 * }
 */
class Solution {
    public TreeNode lowestCommonAncestor(TreeNode root, TreeNode p, TreeNode q) {
        Queue<TreeNode>queue=new LinkedList<>();
        Map<TreeNode,TreeNode>map=new HashMap<>();
        queue.add(root);
        map.put(root,new TreeNode(-1));
        while(!queue.isEmpty()){
            TreeNode node=queue.poll();
            if(node.left!=null){
                queue.add(node.left);
                map.put(node.left,node);
            }
            if(node.right!=null){
                queue.add(node.right);
                map.put(node.right,node);
            }
        }
        HashSet<TreeNode>set=new HashSet<>();
        while(map.containsKey(p)){
            set.add(p);
            p=map.get(p);
        }
        while(!set.contains(q)){
            q=map.get(q);
        }
        return q;
    }
}
```

###### [二叉树的右视图](https://leetcode.cn/problems/binary-tree-right-side-view/)
给定一个二叉树的 根节点 `root`，想象自己站在它的右侧，按照从顶部到底部的顺序，返回从右侧所能看到的节点值

```java
/**
 * Definition for a binary tree node.
 * public class TreeNode {
 *     int val;
 *     TreeNode left;
 *     TreeNode right;
 *     TreeNode() {}
 *     TreeNode(int val) { this.val = val; }
 *     TreeNode(int val, TreeNode left, TreeNode right) {
 *         this.val = val;
 *         this.left = left;
 *         this.right = right;
 *     }
 * }
 */
class Solution {
    public List<Integer> rightSideView(TreeNode root) {
        List<Integer>list=new ArrayList<>();
        if(root==null)
            return list;
        Queue<TreeNode>queue=new LinkedList<>();
        int sz=0;
        queue.add(root);
        
        while(!queue.isEmpty()){
            sz=queue.size();
            for(int i=0;i<sz;i++){
                TreeNode node=queue.poll();
                if(i==sz-1)
                    list.add(node.val);
                if(node.left!=null)queue.add(node.left);
                if(node.right!=null)queue.add(node.right);
            }
        }
        return list;
    }
}
```

###### [二叉搜索树迭代器](https://leetcode.cn/problems/binary-search-tree-iterator/)
思路：在初始化的时候就对树进行一次中序遍历

```java
/**
 * Definition for a binary tree node.
 * public class TreeNode {
 *     int val;
 *     TreeNode left;
 *     TreeNode right;
 *     TreeNode() {}
 *     TreeNode(int val) { this.val = val; }
 *     TreeNode(int val, TreeNode left, TreeNode right) {
 *         this.val = val;
 *         this.left = left;
 *         this.right = right;
 *     }
 * }
 */
class BSTIterator {
    private List<Integer>list;
    private int idx;

    public BSTIterator(TreeNode root) {
        idx=0;
        list=new ArrayList<>();
        inorder(root);
    }
    
    public int next() {
        return list.get(idx++);
    }
    
    public boolean hasNext() {
        return idx<list.size();
    }

    private void inorder(TreeNode root){
        if(root==null)
            return;
        inorder(root.left);
        list.add(root.val);
        inorder(root.right);

    }
}

/**
 * Your BSTIterator object will be instantiated and called as such:
 * BSTIterator obj = new BSTIterator(root);
 * int param_1 = obj.next();
 * boolean param_2 = obj.hasNext();
 */
```

###### [不同的二叉搜索树](https://leetcode.cn/problems/unique-binary-search-trees/)
给你一个整数 `n` ，求恰由 `n` 个节点组成且节点值从 `1` 到 `n` 互不相同的二叉搜索树有多少种？返回满足题意的二叉搜索树的种数

使用动态规划做

```java
class Solution {
    public int numTrees(int n) {
        int[] G = new int[n + 1];
        G[0] = 1;
        G[1] = 1;
        //从i等于2开始求，一直求到等于n
        for (int i = 2; i <= n; ++i) {
            //表示以j为根结点时的个数
            for (int j = 1; j <= i; ++j) {
                G[i] += G[j - 1] * G[i - j];
            }
        }
        return G[n];
    }
}
```

###### [二叉树的完全性检验](https://leetcode.cn/problems/check-completeness-of-a-binary-tree/)
判断一个二叉树是不是完全二叉树

```java
class Solution {
    public boolean isCompleteTree(TreeNode root) {
        List<ANode> nodes = new ArrayList();
        nodes.add(new ANode(root, 1));
        int i = 0;
        while (i < nodes.size()) {
            ANode anode = nodes.get(i++);
            if (anode.node != null) {
                nodes.add(new ANode(anode.node.left, anode.code * 2));
                nodes.add(new ANode(anode.node.right, anode.code * 2 + 1));
            }
        }

        return nodes.get(i-1).code == nodes.size();
    }
}

class ANode {  // Annotated Node
    TreeNode node;
    int code;
    ANode(TreeNode node, int code) {
        this.node = node;
        this.code = code;
    }
}
```

### 图
##### dfs
###### [矩阵中的最长递增路径](https://leetcode.cn/problems/longest-increasing-path-in-a-matrix/)
给定一个 `m x n` 整数矩阵 `matrix` ，找出其中 最长递增路径 的长度。

对于每个单元格，你可以往上，下，左，右四个方向移动。 你 不能 在 对角线 方向上移动或移动到 边界外（即不允许环绕）。

```java
class Solution {
    public int[][] dirs = {{-1, 0}, {1, 0}, {0, -1}, {0, 1}};
    public int rows, columns;

    public int longestIncreasingPath(int[][] matrix) {
        if (matrix == null || matrix.length == 0 || matrix[0].length == 0) {
            return 0;
        }
        rows = matrix.length;
        columns = matrix[0].length;
        //memo用于记录该位置下最长递增路径的长度
        int[][] memo = new int[rows][columns];
        int ans = 0;
        for (int i = 0; i < rows; ++i) {
            for (int j = 0; j < columns; ++j) {
                ans = Math.max(ans, dfs(matrix, i, j, memo));
            }
        }
        return ans;
    }

    public int dfs(int[][] matrix, int row, int column, int[][] memo) {
        if (memo[row][column] != 0) {
            return memo[row][column];
        }
        ++memo[row][column];
        for (int[] dir : dirs) {
            int newRow = row + dir[0], newColumn = column + dir[1];
            if (newRow >= 0 && newRow < rows && newColumn >= 0 && newColumn < columns && matrix[newRow][newColumn] > matrix[row][column]) {
                memo[row][column] = Math.max(memo[row][column], dfs(matrix, newRow, newColumn, memo) + 1);
            }
        }
        return memo[row][column];
    }
}
```

###### [所有可能的路径](https://leetcode.cn/problems/all-paths-from-source-to-target/)
给你一个有 `<font style="color:rgba(38, 38, 38, 0.75);background-color:rgb(240, 240, 240);">n</font>` 个节点的有向无环图（DAG），请你找出所有从节点 `<font style="color:rgba(38, 38, 38, 0.75);background-color:rgb(240, 240, 240);">0</font>` 到节点 `<font style="color:rgba(38, 38, 38, 0.75);background-color:rgb(240, 240, 240);">n-1</font>` 的路径并输出（不要求按特定顺序）

`<font style="color:rgba(38, 38, 38, 0.75);background-color:rgb(240, 240, 240);">graph[i]</font>` 是一个从节点 `<font style="color:rgba(38, 38, 38, 0.75);background-color:rgb(240, 240, 240);">i</font>` 可以访问的所有节点的列表（即从节点 `<font style="color:rgba(38, 38, 38, 0.75);background-color:rgb(240, 240, 240);">i</font>` 到节点 `<font style="color:rgba(38, 38, 38, 0.75);background-color:rgb(240, 240, 240);">graph[i][j]</font>`存在一条有向边）

使用dfs的方式进行遍历图

+ 因为是有向图，不会往回走，所以不需要visit数组
+ 要所有路径，所以在外面定义一个二维数组，当条件匹配的时候就把一维数组加入二维数组，并且去掉一维数组的最后一个元素，继续进行其他的dfs遍历
+ 由于传递的是数组，是引用变量，所以每次加入res数组的时候要创建一个新的数组加进去，不然加到res里的path数组的值也会一直变化，最后都会为0

这么写，最后path为空

```java
class Solution {
    LinkedList<List<Integer>>res=new LinkedList<>();
    public List<List<Integer>> allPathsSourceTarget(int[][] graph) {
        LinkedList<Integer>path=new LinkedList<>();
        dfs(graph,path,0);
        return res;
    }
    public void dfs(int [][]graph,LinkedList<Integer>path,int k){
        path.add(k);
        if(k==graph.length-1){
            res.add(new LinkedList<>(path));
            path.removeLast();
            return;
        }
        for(Integer n:graph[k]){
            dfs(graph,path,n);
        }
        path.removeLast();
    }
}
```

对于removeLast()的位置，也可以这样写最后path是还留着第0个节点的

```java
class Solution {
    LinkedList<List<Integer>>res=new LinkedList<>();
    public List<List<Integer>> allPathsSourceTarget(int[][] graph) {
        LinkedList<Integer>path=new LinkedList<>();
        dfs(graph,path,0);
        return res;
    }
    public void dfs(int [][]graph,LinkedList<Integer>path,int k){
        path.add(k);
        if(k==graph.length-1){
            res.add(new LinkedList<>(path));
            return;
        }
        for(Integer n:graph[k]){
            dfs(graph,path,n);
            path.removeLast();
        }
    }
}
```

###### [分割回文串](https://leetcode.cn/problems/palindrome-partitioning/)
给你一个字符串 `s`，请你将 `s` 分割成一些子串，使每个子串都是 回文串。返回 `s` 所有可能的分割方案

思路：使用回溯的方式挨个遍历，找到符合条件的加入链表中

```java
class Solution {
    List<List<String>>res=new LinkedList<>();
    List<String>list=new LinkedList<>();
    public List<List<String>> partition(String s) {
        dfs(s,0);
        return res;
    }
    public void dfs(String s,int index){
        if(index==s.length()){
            res.add(new LinkedList<>(list));
            return;
        }
        for(int i=1;i+index<=s.length();i++){
            String ss=s.substring(index,index+i);
            if(ishuiwen(ss)){
                list.add(ss);
                dfs(s,i+index);
                list.remove(list.size()-1);
            }
        }

    }
    public boolean ishuiwen(String s){
        int i=0,j=s.length()-1;
        while(i<j){
            if(s.charAt(i)!=s.charAt(j))
                return false;
            i++;
            j--;
        }
        return true;
    }
}
```

###### [克隆图](https://leetcode.cn/problems/clone-graph/)
给你无向[连通](https://baike.baidu.com/item/%E8%BF%9E%E9%80%9A%E5%9B%BE/6460995?fr=aladdin)图中一个节点的引用，请你返回该图的[深拷贝](https://baike.baidu.com/item/%E6%B7%B1%E6%8B%B7%E8%B4%9D/22785317?fr=aladdin)（克隆）

思路：这道题最开始的想法是用一个hashset作为visited，dfs的类型是void，利用类作为形参传递的是引用直接进行添加。但问题这个情况下，因为是无向图，所以如果图是1-2，当dfs 1的时候把2加入了数组，但dfs 2的时候因为判断visited 这个时候1是已经visited了，所以就会直接返回，这样2中数组的1就只是一个单纯的1，这个1的数组为空

所以不能使用hashset作为visited，应该使用hashmap，所以dfs的类型也改为Node,这样当判断visited的时候，如果已经visited，就直接返回之前的那个node

```java
/*
// Definition for a Node.
class Node {
    public int val;
    public List<Node> neighbors;
    public Node() {
        val = 0;
        neighbors = new ArrayList<Node>();
    }
    public Node(int _val) {
        val = _val;
        neighbors = new ArrayList<Node>();
    }
    public Node(int _val, ArrayList<Node> _neighbors) {
        val = _val;
        neighbors = _neighbors;
    }
}
*/

class Solution {
    HashMap<Node,Node>visited=new HashMap<>();
    public Node cloneGraph(Node node) {
        if(node==null)
            return node;
        Node clonenode=dfs(node);
        return clonenode;
    }

    public Node dfs(Node node){
        if(node==null)
            return node;
        if(visited.containsKey(node))
            return visited.get(node);
        Node clonenode=new Node(node.val,new ArrayList());
        visited.put(node,clonenode);
        for(Node neighbor:node.neighbors){
            Node c=dfs(neighbor);
            clonenode.neighbors.add(c);
        }
        return clonenode;
    }
}
```

###### [除法求值](https://leetcode.cn/problems/evaluate-division/)
给你一个变量对数组 `<font style="color:rgba(38, 38, 38, 0.75);background-color:rgb(240, 240, 240);">equations</font>` 和一个实数值数组 `<font style="color:rgba(38, 38, 38, 0.75);background-color:rgb(240, 240, 240);">values</font>` 作为已知条件，其中 `<font style="color:rgba(38, 38, 38, 0.75);background-color:rgb(240, 240, 240);">equations[i] = [A</font><sub><font style="color:rgba(38, 38, 38, 0.75);background-color:rgb(240, 240, 240);">i</font></sub><font style="color:rgba(38, 38, 38, 0.75);background-color:rgb(240, 240, 240);">, B</font><sub><font style="color:rgba(38, 38, 38, 0.75);background-color:rgb(240, 240, 240);">i</font></sub><font style="color:rgba(38, 38, 38, 0.75);background-color:rgb(240, 240, 240);">]</font>` 和 `<font style="color:rgba(38, 38, 38, 0.75);background-color:rgb(240, 240, 240);">values[i]</font>` 共同表示等式 `<font style="color:rgba(38, 38, 38, 0.75);background-color:rgb(240, 240, 240);">A</font><sub><font style="color:rgba(38, 38, 38, 0.75);background-color:rgb(240, 240, 240);">i</font></sub><font style="color:rgba(38, 38, 38, 0.75);background-color:rgb(240, 240, 240);"> / B</font><sub><font style="color:rgba(38, 38, 38, 0.75);background-color:rgb(240, 240, 240);">i</font></sub><font style="color:rgba(38, 38, 38, 0.75);background-color:rgb(240, 240, 240);"> = values[i]</font>` 。每个 `<font style="color:rgba(38, 38, 38, 0.75);background-color:rgb(240, 240, 240);">A</font><sub><font style="color:rgba(38, 38, 38, 0.75);background-color:rgb(240, 240, 240);">i</font></sub>` 或 `<font style="color:rgba(38, 38, 38, 0.75);background-color:rgb(240, 240, 240);">B</font><sub><font style="color:rgba(38, 38, 38, 0.75);background-color:rgb(240, 240, 240);">i</font></sub>` 是一个表示单个变量的字符串。

另有一些以数组 `<font style="color:rgba(38, 38, 38, 0.75);background-color:rgb(240, 240, 240);">queries</font>` 表示的问题，其中 `<font style="color:rgba(38, 38, 38, 0.75);background-color:rgb(240, 240, 240);">queries[j] = [C</font><sub><font style="color:rgba(38, 38, 38, 0.75);background-color:rgb(240, 240, 240);">j</font></sub><font style="color:rgba(38, 38, 38, 0.75);background-color:rgb(240, 240, 240);">, D</font><sub><font style="color:rgba(38, 38, 38, 0.75);background-color:rgb(240, 240, 240);">j</font></sub><font style="color:rgba(38, 38, 38, 0.75);background-color:rgb(240, 240, 240);">]</font>` 表示第 `<font style="color:rgba(38, 38, 38, 0.75);background-color:rgb(240, 240, 240);">j</font>` 个问题，请你根据已知条件找出 `<font style="color:rgba(38, 38, 38, 0.75);background-color:rgb(240, 240, 240);">C</font><sub><font style="color:rgba(38, 38, 38, 0.75);background-color:rgb(240, 240, 240);">j</font></sub><font style="color:rgba(38, 38, 38, 0.75);background-color:rgb(240, 240, 240);"> / D</font><sub><font style="color:rgba(38, 38, 38, 0.75);background-color:rgb(240, 240, 240);">j</font></sub><font style="color:rgba(38, 38, 38, 0.75);background-color:rgb(240, 240, 240);"> = ?</font>` 的结果作为答案。

思路：这道题的思路比较简单，换个说法就是检查两个节点是否连通，并且在遍历的过程中要一直计算着“路程”

这道题的难点在于建图，因为节点的类型是字符串类型，就不能用List<>[]nums的形式，所以使用hashmap+edage类的形式进行建图

其中用到了一些没怎么用过的hashmap的方法，这里记录一下

+ graph.computeIfAbsent(s1,k->new ArrayList<>()).add(new edage(s2,values[i]));这个方法是建立的时候使用的，是首先会判断map中是否含有s1，如果不含有会提前创建一个value 即ArrayList，如果含有就直接add一个edage
+ graph.getOrDefault(from,new ArrayList<>())这个是获取map中from这个key所含有的value 即它的ArrayList

```java
class edage{
    String to;
    double dist;
    edage(String to,double dist){
        this.to=to;
        this.dist=dist;
    }
}


class Solution {
    Map<String, List<edage>> graph = new HashMap<>();
    public double[] calcEquation(List<List<String>> equations, double[] values, List<List<String>> queries) {
        for(int i=0;i<equations.size();i++){
            String s1=equations.get(i).get(0);
            String s2=equations.get(i).get(1);
            graph.computeIfAbsent(s1,k->new ArrayList<>()).add(new edage(s2,values[i]));
            graph.computeIfAbsent(s2,k->new ArrayList<>()).add(new edage(s1,1.0/values[i]));
        }
        double []res=new double[queries.size()];
        Arrays.fill(res,-1.0);
        for(int i=0;i<queries.size();i++){
            dfs(queries.get(i).get(0),queries.get(i).get(1),1.0,res,i,new HashSet<>());
        }
        return res;

    }
        //cur表示当前的“距离”，res存储了queris中每一轮的结果，index表示是queris的哪一轮，方便在res中赋值
    public void dfs(String from,String to,double cur,double []res,int index,HashSet<String>visited){
        if(visited.contains(from))
            return;
        visited.add(from);
      //这个containsKey是判断X/X的情况，题目中说明X/X为-1
        if(from.equals(to)&&graph.containsKey(from)){
            res[index]=cur;
            return;
        }
        for(edage e:graph.getOrDefault(from,new ArrayList<>())){
            dfs(e.to,to,cur*e.dist,res,index,visited);
        }
        
    }
}
```

##### 拓扑排序
###### [课程表](https://leetcode.cn/problems/course-schedule/)
你这个学期必须选修 `<font style="color:rgba(38, 38, 38, 0.75);background-color:rgb(240, 240, 240);">numCourses</font>` 门课程，记为 `<font style="color:rgba(38, 38, 38, 0.75);background-color:rgb(240, 240, 240);">0</font>` 到 `<font style="color:rgba(38, 38, 38, 0.75);background-color:rgb(240, 240, 240);">numCourses - 1</font>` 。

在选修某些课程之前需要一些先修课程。 先修课程按数组 `<font style="color:rgba(38, 38, 38, 0.75);background-color:rgb(240, 240, 240);">prerequisites</font>` 给出，其中 `<font style="color:rgba(38, 38, 38, 0.75);background-color:rgb(240, 240, 240);">prerequisites[i] = [a</font><sub><font style="color:rgba(38, 38, 38, 0.75);background-color:rgb(240, 240, 240);">i</font></sub><font style="color:rgba(38, 38, 38, 0.75);background-color:rgb(240, 240, 240);">, b</font><sub><font style="color:rgba(38, 38, 38, 0.75);background-color:rgb(240, 240, 240);">i</font></sub><font style="color:rgba(38, 38, 38, 0.75);background-color:rgb(240, 240, 240);">]</font>` ，表示如果要学习课程 `<font style="color:rgba(38, 38, 38, 0.75);background-color:rgb(240, 240, 240);">a</font><sub><font style="color:rgba(38, 38, 38, 0.75);background-color:rgb(240, 240, 240);">i</font></sub>` 则必须先学习课程  `<font style="color:rgba(38, 38, 38, 0.75);background-color:rgb(240, 240, 240);">b</font><sub><font style="color:rgba(38, 38, 38, 0.75);background-color:rgb(240, 240, 240);">i</font></sub>` 。

采用dfs的方式进行求解，首先是初始化各个数组list，然后进行建图，然后以每个节点作为根节点进行遍历

**对于无向的图的话，找是否有循环，首先想到的是union+parent，对于有向的图的话，可以直接用dfs+onpath进行判断**，onpath是一个数组，代表了当前的遍历路径上还在路径上的节点，因为每遍历完一个节点，要返回的时候，就会把它的onpath[x]设置为0，代表了已经剔除路径，这样的话每次进行一个新的dfs的时候只需要判断onpath，如果为1，那说明当前路径上有该节点，那说明成环了。

所以判断是否有环就可以用这种方式，遍历该点之前设置为true，遍历之后设置为false

```java
class Solution {
    int []visited;
    int []onpath;
      //定义一个成员变量，如果circle变为true，那说明有环，最后也是返回！circle
    boolean circle=false;
    public boolean canFinish(int numCourses, int[][] prerequisites) {
        LinkedList<Integer>[]graph=new LinkedList[numCourses];
        for (int i = 0; i < numCourses; i++) {
            graph[i] = new LinkedList<>();
        }
        visited=new int[numCourses];
          //建图
        for(int i=0;i<prerequisites.length;i++){
            graph[prerequisites[i][1]].add(prerequisites[i][0]);
        }
        onpath=new int[numCourses];
          //对每个节点作为根节点进行遍历
        for(int i=0;i<numCourses;i++){
            dfs(graph,i);
        }
        return !circle;

    }
    public void dfs(LinkedList<Integer>[]graph,int s){
          //在路径上，说明有环
        if(onpath[s]==1){
            circle=true;
        }
          //visited表示可能之前其他节点作为根节点的时候，已经遍历过该点了 那么就不需要再遍历了
          //circle如果有循环了也不需要遍历了
        if(visited[s]==1||circle){
            return;
        }
        onpath[s]=1;
        visited[s]=1;
        for(int t:graph[s]){
            dfs(graph,t);
        }
        onpath[s]=0;
    }
}
```

###### [课程表 II](https://leetcode.cn/problems/course-schedule-ii/)
和上一题的区别是要给一个顺序，这个顺序是在进行dfs遍历的时候的后序提取然后再反转即可，使用先序是不行的，先序根据顺序有可能会让子节点跑到前面去。

```java
class Solution {
    int []visited;
    int []onpath;
    boolean circle=false;
    LinkedList<Integer>list=new LinkedList<>();
    public int[] findOrder(int numCourses, int[][] prerequisites) {
        LinkedList<Integer>[]graph=new LinkedList[numCourses];
        for(int i=0;i<numCourses;i++){
            graph[i]=new LinkedList<>();
        }
        for(int i=0;i<prerequisites.length;i++){
            graph[prerequisites[i][1]].add(prerequisites[i][0]);
        }
        visited=new int[numCourses];
        onpath=new int[numCourses];
        for(int i=0;i<numCourses;i++){
            dfs(graph,i);
        }
        if(circle==true)
            return new int[]{};
        Collections.reverse(list);
        int[] res = new int[numCourses];
        for (int i = 0; i < numCourses; i++) {
            res[i] = list.get(i);
        }
        return res;

    }
    public void dfs(LinkedList<Integer>[]graph,int s){
        if(onpath[s]==1){
            circle=true;
        }
        if(visited[s]==1||circle){
            return;
        }
        visited[s]=1;
        onpath[s]=1;
        for(int t:graph[s]){
            dfs(graph,t);
        }
        list.add(s);
        onpath[s]=0;
    }
}
```

方法2️⃣：

使用bfs的方式，即考虑出度入度的关系，把这个想成一个图，从入度为0的点开始去除，去除之后他的子节点的入度就会减1，如果不存在循环，那么所有的点都能消掉，一共会遍历numCourses次，如果存在循环，那么在收集入度为0的点时，循环内的点都不满足条件，那么就不会加入队列，就会少遍历几次，所以最后只需要判断遍历的次数是否和numCourses相等就可以找到是否有环。对于顺序，遍历的顺序就是拓扑排序的顺序。

```java
class Solution {
    public int[] findOrder(int numCourses, int[][] prerequisites) {
        Queue<Integer>queue=new LinkedList<>();
        LinkedList<Integer>graph[]=new LinkedList[numCourses];
        int []in=new int[numCourses];
        for(int i=0;i<numCourses;i++){
            graph[i]=new LinkedList<>();
        }
        for(int []num:prerequisites){
            graph[num[1]].add(num[0]);
            in[num[0]]++;
        }
        for(int i=0;i<numCourses;i++){
            if(in[i]==0)
                queue.add(i);
        }
        int count=0;
        int []res=new int[numCourses]; 
        while(!queue.isEmpty()){
            int point=queue.poll();
            res[count]=point;
            count++;
            for(int x:graph[point]){
                in[x]--;
                if(in[x]==0)
                    queue.add(x);
            }
        }
        if(count==numCourses)
            return res;
        return new int[]{};
    }
}
```

##### 二分图
###### [ 判断二分图](https://leetcode.cn/problems/is-graph-bipartite/)
存在一个无向图 ，图中有 `n` 个节点。其中每个节点都有一个介于 `0` 到 `n - 1` 之间的唯一编号。给你一个二维数组 `graph` ，其中 `graph[u]` 是一个节点数组，由节点 `u` 的邻接节点组成。形式上，对于 `graph[u]` 中的每个 `v` ，都存在一条位于节点 `u` 和节点 `v` 之间的无向边

判断是否是二分图，用dfs来进行判断，首先创建两个数组，一个visited，一个color数组，因为不一定所有节点都连接，所以要用for循环把所有点都dfs一遍，对于dfs里面，首先进行判断是否已经不是二分图了，如果已经不是了就直接返回了，否则遍历v点的所有连接点，首先判断这个点是否是访问过的，如果访问过的就判断这两个点颜色是否相同，如果没有访问过就将这个点设置为已访问，并且颜色设置为相反颜色，然后dfs该点。

```java
class Solution {
    boolean []visited;
    boolean []color;
    boolean res=true;
    public boolean isBipartite(int[][] graph) {
        int n=graph.length;
        visited=new boolean[n];
        color=new boolean[n];
        for(int i=0;i<n;i++){
            dfs(graph,i);
        }
        return res;

    }
    public void dfs(int [][]graph,int v){
        if(!res)
            return;
        for(int u:graph[v]){
            if(visited[u]){
                if(color[u]==color[v]){
                    res=false;
                    return;
                }
            }else{
                visited[u]=true;
                color[u]=!color[v];
                dfs(graph,u);
            }
        }
    }
}
```

###### [可能的二分法](https://leetcode.cn/problems/possible-bipartition/)
给定一组 `n` 人（编号为 `1, 2, ..., n`）， 我们想把每个人分进任意大小的两组。每个人都可能不喜欢其他人，那么他们不应该属于同一组。

给定整数 `n` 和数组 `dislikes` ，其中 `dislikes[i] = [ai, bi]` ，表示不允许将编号为 `ai` 和  `bi`的人归入同一组。当可以用这种方法将所有人分进两组时，返回 `true`；否则返回 `false`。

和上题一样的思路，区别在于输入数组不同，这个需要自己建邻序表

```java
class Solution {
    boolean []visited;
    boolean []color;
    boolean res=true;
    public boolean possibleBipartition(int n, int[][] dislikes) {
        LinkedList<Integer>[]graph=new LinkedList[n+1];
        visited=new boolean[n+1];
        color=new boolean[n+1];
        for(int i=0;i<=n;i++){
            graph[i]=new LinkedList<>();
        }
        for(int []num:dislikes){
            graph[num[0]].add(num[1]);
            graph[num[1]].add(num[0]);
        }
        for(int i=1;i<=n;i++){
            dfs(graph,i);
        }
        return res;
    }
    public void dfs(LinkedList<Integer>[]graph,int v){
        if(!res)
            return;
        for(int u:graph[v]){
            if(visited[u]){
                if(color[u]==color[v]){
                    res=false;
                    return;
                }
            }else{
                visited[u]=true;
                color[u]=!color[v];
                dfs(graph,u);
            }
        }
    }
}
```

##### 并查集
###### [等式方程的可满足性](https://leetcode.cn/problems/satisfiability-of-equality-equations/)
并差集

给定一个由表示变量之间关系的字符串方程组成的数组，每个字符串方程 `equations[i]` 的长度为 `4`，并采用两种不同的形式之一：`"a==b"` 或 `"a!=b"`。在这里，a 和 b 是小写字母（不一定不同），表示单字母变量名。

这里的find使用的是递归的方式进行，这样的好处是能够把这个树的高度变为1，提高查询效率，

这道题因为不知道字母的数量，所以parent数组直接是设置的26，不需要传递类型为char类型，只需要在传递之间减一个‘a’即可

```java
class Solution {
    int []parent=new int[26] ;
    public int find(int x){
        if(parent[x]!=x)
            parent[x]=find(parent[x]);
        return parent[x];
    }
    public void union(int x,int y){
        int xp=find(x);
        int yp=find(y);
        if(xp==yp)
            return;
        parent[xp]=yp;
    }
    public boolean equationsPossible(String[] equations) {
        for(int i=0;i<26;i++)
            parent[i]=i;
        for(String s:equations){
            if(s.charAt(1)=='='){
                int x=s.charAt(0)-'a';
                int y=s.charAt(3)-'a';
                union(x,y);
            }
        }
        boolean flag=true;
        for(String s:equations){
            if(s.charAt(1)=='!'){
                int x=find(s.charAt(0)-'a');
                int y=find(s.charAt(3)-'a');
                if(x==y){
                    flag=false;
                    break;
                }
            }
        }
        return flag;
    }
}
```

##### 最小生成树
###### [连接所有点的最小费用](https://leetcode.cn/problems/min-cost-to-connect-all-points/)
曼哈顿距离

使用克鲁斯卡尔法求最小生成树

```java
class Solution {
    int []parent;
    public void union(int x,int y){
        int xp=find(x);
        int yp=find(y);
        if(xp==yp)
            return;
        parent[xp]=yp;
    }

    public int find(int x){
        if(parent[x]!=x)
            parent[x]=find(parent[x]);
        return parent[x];
    }
    
    public int dist(int []x,int []y){
        return Math.abs(x[0]-y[0])+Math.abs(x[1]-y[1]);
    }

    public int minCostConnectPoints(int[][] points) {
        int v=points.length;
        parent=new int[v];
        for(int i=0;i<v;i++)
            parent[i]=i;
        int res=0;
        int count=0;
        while(count<v-1){
            int min=Integer.MAX_VALUE;
            int from=-1;int to=-1;
            for(int i=0;i<v;i++)
                for(int j=0;j<v;j++){
                    if(dist(points[i],points[j])<min&&find(i)!=find(j)){
                        from=i;
                        to=j;
                        min=dist(points[i],points[j]);
                    }
                }
            if(from!=-1&&to!=-1){
                count++;
                res+=min;
                union(from,to);
            }
        }
        return res;
    }
}
```

##### Dijkstra
###### [概率最大的路径](https://leetcode.cn/problems/path-with-maximum-probability/)
给你一个由 `<font style="color:rgba(38, 38, 38, 0.75);background-color:rgb(240, 240, 240);">n</font>` 个节点（下标从 0 开始）组成的无向加权图，该图由一个描述边的列表组成，其中 `<font style="color:rgba(38, 38, 38, 0.75);background-color:rgb(240, 240, 240);">edges[i] = [a, b]</font>` 表示连接节点 a 和 b 的一条无向边，且该边遍历成功的概率为 `<font style="color:rgba(38, 38, 38, 0.75);background-color:rgb(240, 240, 240);">succProb[i]</font>` 。

指定两个节点分别作为起点 `<font style="color:rgba(38, 38, 38, 0.75);background-color:rgb(240, 240, 240);">start</font>` 和终点 `<font style="color:rgba(38, 38, 38, 0.75);background-color:rgb(240, 240, 240);">end</font>` ，请你找出从起点到终点成功概率最大的路径，并返回其成功概率

方法1️⃣：使用bfs的思路，逐渐更新起始点的邻接点，使用邻接列表➕队列的方式，由于不像之前的树权重都是1，不需要管权重，这个是有权重的，所以要单独定义一个类，来记录权重

其中由于是到某个点的最小距离，也可以用if进行判断，到了那个点就可以返回，不过考虑写一个模版，所以没有加

```java
import java.util.*;

class Edge {
    int to;
    double dist;
    Edge(int to,double dist){
        this.to=to;
        this.dist=dist;
    }
}

class Solution {
    LinkedList<Edge>[]graph;
    public double maxProbability(int n, int[][] edges, double[] succProb, int start_node, int end_node) {
        graph=new LinkedList[n];
        for(int i=0;i<n;i++)
            graph[i]=new LinkedList<>();
        build(edges,succProb);
        double []dist=dijsktra(n,start_node);
        if(dist[end_node]!=-1)
            return dist[end_node];
        return 0;

    }
    public double[]dijsktra(int n,int start_node){
        double[]dist=new double[n];
        Arrays.fill(dist,-1);
        dist[start_node]=1;
        Queue<Edge>queue=new LinkedList<>();
        queue.add(new Edge(start_node,1));
        while(!queue.isEmpty()){
            Edge edge=queue.poll();
            int to=edge.to;
            for(Edge edge1:graph[to]){
                int edge_to=edge1.to;
                double edge_dist=dist[to]*edge1.dist;
                if(edge_dist>dist[edge_to]){
                    dist[edge_to]=edge_dist;
                    queue.add(new Edge(edge_to,dist[edge_to]));
                }
            }
        }
        return dist;
    }
    public void build(int [][]edges,double[]succProb){
        int count=0;
        for(int []edge:edges){
            graph[edge[0]].add(new Edge(edge[1],succProb[count]));
            graph[edge[1]].add(new Edge(edge[0],succProb[count]));
            count++;
        }
    }
}
```

方法2️⃣：使用邻接矩阵的方式来存储，好处是就不需要单独定义一个类来存储权重，坏处是这样的话需要定义一个很大的邻接矩阵，会造成空间上的浪费，在leetcode上会超出内存限制

```java
class Solution {
    double [][]graph;
    public double maxProbability(int n, int[][] edges, double[] succProb, int start_node, int end_node) {
        graph=new double[n][n];
        build(edges,succProb);
        double []dist=dijsk(n,start_node);
        if(dist[end_node]!=Integer.MIN_VALUE){
            return dist[end_node];
        }
        return 0;
    }
    public double[] dijsk(int n,int start_node){
        double dist[]=new double[n];
        boolean []visited=new boolean[n];
        Arrays.fill(dist,Integer.MIN_VALUE);
        dist[start_node]=1;
        for(int i=0;i<n-1;i++){
            int index=findMax(dist,visited);
            if(index==-1)
                break;
            visited[index]=true;
            for(int j=0;j<n;j++){
                if(graph[index][j]!=0&&dist[index]>0&&!visited[j]&&dist[index]*graph[index][j]>dist[j]){
                    dist[j]=dist[index]*graph[index][j];
                }
            }
        }
        return dist;
    }

    public int findMax(double []dist,boolean []visited){
        double max=Integer.MIN_VALUE;
        int index=0;
        for(int i=0;i<dist.length;i++){
            if(!visited[i]&&dist[i]>max){
                index=i;
                max=dist[i];
            }
        }
        return index;
    }
    public void build(int [][]edges,double[]succProb){
        int count=0;
        for(int []edge:edges){
            graph[edge[0]][edge[1]]=succProb[count];
            graph[edge[1]][edge[0]]=succProb[count];
            count++;
        }
    }
}
```

###### [最小体力消耗路径](https://leetcode.cn/problems/path-with-minimum-effort/)
你准备参加一场远足活动。给你一个二维 `<font style="color:rgba(38, 38, 38, 0.75);">rows x columns</font>` 的地图 `<font style="color:rgba(38, 38, 38, 0.75);">heights</font>` ，其中 `<font style="color:rgba(38, 38, 38, 0.75);">heights[row][col]</font>` 表示格子 `<font style="color:rgba(38, 38, 38, 0.75);">(row, col)</font>` 的高度。一开始你在最左上角的格子 `<font style="color:rgba(38, 38, 38, 0.75);">(0, 0)</font>` ，且你希望去最右下角的格子 `<font style="color:rgba(38, 38, 38, 0.75);">(rows-1, columns-1)</font>` （注意下标从 0 开始编号）。你每次可以往 上，下，左，右 四个方向之一移动，你想要找到耗费体力最小的一条路径

可以像上道题一样用同样的模版，加入队列不断的更新，直到所有的更新结束之后就是到所有点的最短路径

或者因为这个是找目的地的最短路径，使用有序队列，优先找最近的节点，再用if进行判断是否在目标节点，如果到了就结束了（即要用if进行target判断就要用有序队列）

```java
import java.util.Arrays;
import java.util.LinkedList;
import java.util.PriorityQueue;
import java.util.Queue;

class location{
    int x;
    int y;
    int dist;
    location(int x,int y,int dist){
        this.x=x;
        this.y=y;
        this.dist=dist;
    }
}

class Solution {
    static int [][]position={
      {0,1},
      {0,-1},
      {1,0},
      {-1,0}
    };
    static int [][]res;

    public static int minimumEffortPath(int[][] heights) {
        int res=dijsktra(heights);
        return res;
    }

    public static int dijsktra(int [][]heights){
        int m=heights.length;
        int n=heights[0].length;
        PriorityQueue<location> queue=new PriorityQueue<>(
          (a,b)->{
            return a.dist-b.dist;
            }
        );
        res= new int[m][n];
        for (int[] row : res) {
            Arrays.fill(row, Integer.MAX_VALUE);
        }
        res[0][0]=0;
        queue.add(new location(0,0,0));
        while(!queue.isEmpty()){
            location l=queue.poll();
            int lx=l.x;
            int ly=l.y;
            int d=l.dist;
            if(lx==m-1&&ly==n-1){
                break;
            }
            for(int i=0;i<4;i++){
                if((lx+position[i][0])>=0&&(lx+position[i][0])<m&&(ly+position[i][1])>=0&&(ly+position[i][1])<n){
                    int x=Math.max(Math.abs(heights[lx+position[i][0]][ly+position[i][1]]-heights[lx][ly]),res[lx][ly]);
                    if(x<res[lx+position[i][0]][ly+position[i][1]]){
                        res[lx+position[i][0]][ly+position[i][1]]=x;
                        queue.add(new location(lx+position[i][0],ly+position[i][1],res[lx+position[i][0]][ly+position[i][1]]));
                    }
                }
            }
        }
        return res[m-1][n-1];
    }
}
```

###### [ 网络延迟时间](https://leetcode.cn/problems/network-delay-time/)
给你一个列表 `<font style="color:rgba(38, 38, 38, 0.75);">times</font>`，表示信号经过 有向 边的传递时间。 `<font style="color:rgba(38, 38, 38, 0.75);">times[i] = (u</font><sub><font style="color:rgba(38, 38, 38, 0.75);">i</font></sub><font style="color:rgba(38, 38, 38, 0.75);">, v</font><sub><font style="color:rgba(38, 38, 38, 0.75);">i</font></sub><font style="color:rgba(38, 38, 38, 0.75);">, w</font><sub><font style="color:rgba(38, 38, 38, 0.75);">i</font></sub><font style="color:rgba(38, 38, 38, 0.75);">)</font>`，其中 `<font style="color:rgba(38, 38, 38, 0.75);">u</font><sub><font style="color:rgba(38, 38, 38, 0.75);">i</font></sub>` 是源节点，`<font style="color:rgba(38, 38, 38, 0.75);">v</font><sub><font style="color:rgba(38, 38, 38, 0.75);">i</font></sub>` 是目标节点， `<font style="color:rgba(38, 38, 38, 0.75);">w</font><sub><font style="color:rgba(38, 38, 38, 0.75);">i</font></sub>` 是一个信号从源节点传递到目标节点的时间。

现在，从某个节点 `<font style="color:rgba(38, 38, 38, 0.75);">K</font>` 发出一个信号。需要多久才能使所有节点都收到信号？如果不能使所有节点收到信号，返回 `<font style="color:rgba(38, 38, 38, 0.75);">-1</font>` 。

用一样的思路

```java
import java.util.Arrays;
import java.util.LinkedList;
import java.util.PriorityQueue;

class Edge{
    int to;
    int dist;
    Edge(int to,int dist){
        this.to=to;
        this.dist=dist;
    }
}

class Solution {

    LinkedList<Edge>[]graph;
    public int networkDelayTime(int[][] times, int n, int k) {
        graph=new LinkedList[n+1];
        for(int i=0;i<=n;i++)
            graph[i]=new LinkedList<>();
        build(times);
        int []res=dijkstra(k,n);
        int max=0;
        for(int i=1;i<res.length;i++){
            if(max<res[i])
                max=res[i];
            if(res[i]==Integer.MAX_VALUE)
                return -1;
        }
        return max;
    }

    public int[] dijkstra(int k,int n){
        int []res=new int[n+1];
        Arrays.fill(res,Integer.MAX_VALUE);
        res[k]=0;
        PriorityQueue<Edge> queue=new PriorityQueue<>((a, b)->{
            return a.dist-b.dist;
        });
        queue.add(new Edge(k,0));
        while(!queue.isEmpty()){
            Edge edge=queue.poll();
            int curto=edge.to;
            int curdist=edge.dist;
            if(curdist<res[curto])
                continue;
            for(Edge edge1:graph[curto]){
                int next_to=edge1.to;
                int next_dist=edge1.dist;
                if(next_dist+res[curto]<res[next_to]){
                    res[next_to]=next_dist+res[curto];
                    queue.add(new Edge(next_to,res[next_to]));
                }
            }
        }
        return res;
    }

    public void build(int [][]times){
        for(int []time:times){
            int from=time[0];
            int to=time[1];
            int dist=time[2];
            graph[from].add(new Edge(to,dist));
        }
    }
}
```

###### [ K 站中转内最便宜的航班](https://leetcode.cn/problems/cheapest-flights-within-k-stops/)
有 `<font style="color:rgba(38, 38, 38, 0.75);background-color:rgb(240, 240, 240);">n</font>` 个城市通过一些航班连接。给你一个数组 `<font style="color:rgba(38, 38, 38, 0.75);background-color:rgb(240, 240, 240);">flights</font>` ，其中 `<font style="color:rgba(38, 38, 38, 0.75);background-color:rgb(240, 240, 240);">flights[i] = [from</font><sub><font style="color:rgba(38, 38, 38, 0.75);background-color:rgb(240, 240, 240);">i</font></sub><font style="color:rgba(38, 38, 38, 0.75);background-color:rgb(240, 240, 240);">, to</font><sub><font style="color:rgba(38, 38, 38, 0.75);background-color:rgb(240, 240, 240);">i</font></sub><font style="color:rgba(38, 38, 38, 0.75);background-color:rgb(240, 240, 240);">, price</font><sub><font style="color:rgba(38, 38, 38, 0.75);background-color:rgb(240, 240, 240);">i</font></sub><font style="color:rgba(38, 38, 38, 0.75);background-color:rgb(240, 240, 240);">]</font>` ，表示该航班都从城市 `<font style="color:rgba(38, 38, 38, 0.75);background-color:rgb(240, 240, 240);">from</font><sub><font style="color:rgba(38, 38, 38, 0.75);background-color:rgb(240, 240, 240);">i</font></sub>` 开始，以价格 `<font style="color:rgba(38, 38, 38, 0.75);background-color:rgb(240, 240, 240);">price</font><sub><font style="color:rgba(38, 38, 38, 0.75);background-color:rgb(240, 240, 240);">i</font></sub>` 抵达 `<font style="color:rgba(38, 38, 38, 0.75);background-color:rgb(240, 240, 240);">to</font><sub><font style="color:rgba(38, 38, 38, 0.75);background-color:rgb(240, 240, 240);">i</font></sub>`。

现在给定所有的城市和航班，以及出发城市 `<font style="color:rgba(38, 38, 38, 0.75);background-color:rgb(240, 240, 240);">src</font>` 和目的地 `<font style="color:rgba(38, 38, 38, 0.75);background-color:rgb(240, 240, 240);">dst</font>`，你的任务是找到出一条最多经过 `<font style="color:rgba(38, 38, 38, 0.75);background-color:rgb(240, 240, 240);">k</font>` 站中转的路线，使得从 `<font style="color:rgba(38, 38, 38, 0.75);background-color:rgb(240, 240, 240);">src</font>` 到 `<font style="color:rgba(38, 38, 38, 0.75);background-color:rgb(240, 240, 240);">dst</font>` 的 价格最便宜 ，并返回该价格。 如果不存在这样的路线，则输出 `<font style="color:rgba(38, 38, 38, 0.75);background-color:rgb(240, 240, 240);">-1</font>`。

这道题特殊点在于要考虑两个地方，一个是最短距离，一个是边的数量不能超过k+1

```java
class Solution {
    class State {
        int id;
        int costFromSrc;
        int nodeNumFromSrc;
        State(int id, int costFromSrc, int nodeNumFromSrc) {
            this.id = id;
            this.costFromSrc = costFromSrc;
            this.nodeNumFromSrc = nodeNumFromSrc;
        }
    }
    public int findCheapestPrice(int n, int[][] flights, int src, int dst, int K) {
        List<int[]>[] graph = new LinkedList[n];
        for (int i = 0; i < n; i++) {
            graph[i] = new LinkedList<>();
        }
        for (int[] edge : flights) {
            int from = edge[0];
            int to = edge[1];
            int price = edge[2];
            graph[from].add(new int[]{to, price});
        }
        K++;
        return dijkstra(graph, src, K, dst);
    }
    int dijkstra(List<int[]>[] graph, int src, int k, int dst) {
        int[] distTo = new int[graph.length];
        int[] nodeNumTo = new int[graph.length];
        Arrays.fill(distTo, Integer.MAX_VALUE);
        Arrays.fill(nodeNumTo, Integer.MAX_VALUE);
        distTo[src] = 0;
        nodeNumTo[src] = 0;
        Queue<State> pq = new PriorityQueue<>((a, b) -> {
            return a.costFromSrc - b.costFromSrc;
        });
        pq.offer(new State(src, 0, 0));
        while (!pq.isEmpty()) {
            State curState = pq.poll();
            int curNodeID = curState.id;
            int costFromSrc = curState.costFromSrc;
            int curNodeNumFromSrc = curState.nodeNumFromSrc;
            
            if (curNodeID == dst) {
                return costFromSrc;
            }
              //表示当前中转站的点肯定是不行了 看看队列中的其他节点能不能通过
            if (curNodeNumFromSrc == k) {
                continue;
            }
            for (int[] neighbor : graph[curNodeID]) {
                int nextNodeID = neighbor[0];
                int costToNextNode = costFromSrc + neighbor[1];
                int nextNodeNumFromSrc = curNodeNumFromSrc + 1;
                if (distTo[nextNodeID] > costToNextNode) {
                    distTo[nextNodeID] = costToNextNode;
                    nodeNumTo[nextNodeID] = nextNodeNumFromSrc;
                }
                if (costToNextNode > distTo[nextNodeID]
                    && nextNodeNumFromSrc > nodeNumTo[nextNodeID]) {
                    continue;
                }
                pq.offer(new State(nextNodeID, costToNextNode, nextNodeNumFromSrc));
            }
        }
        return -1;
    }

}
```

### 动态规划
重叠子问题、最优子结构、状态转移方程就是动态规划三要素

**要符合「最优子结构」，子问题间必须互相独立**。

##### 背包问题
**0-1背包问题**

```java
int knapsack(int W, int N, int[] wt, int[] val) {
    assert N == wt.length;
    // base case 已初始化
    int[][] dp = new int[N + 1][W + 1];
    for (int i = 1; i <= N; i++) {
        for (int w = 1; w <= W; w++) {
            if (w < wt[i - 1] ) {
                // 这种情况下只能选择不装入背包
                dp[i][w] = dp[i - 1][w];
            } else {
                // 装入或者不装入背包，择优
                dp[i][w] = Math.max(
                    dp[i - 1][w - wt[i-1]] + val[i-1], 
                    dp[i - 1][w]
                );
            }
        }
    }
    return dp[N][W];
}

```

###### [分割等和子集](https://leetcode.cn/problems/partition-equal-subset-sum/)
给你一个只包含正整数的非空数组 `nums` 。请你判断是否可以将这个数组分割成两个子集，使得两个子集的元素和相等

```java
class Solution {
    public boolean canPartition(int[] nums) {
        int sum=0;
        int target;
        for(int n:nums)
            sum+=n;
        if(sum%2!=0)
            return false;
        target=sum/2;
        //判断能不能达到，所以用boolean类型即可
        boolean [][]dp=new boolean[nums.length+1][sum+1];
        //初始化第0行
        dp[0][0]=true;
        //通过状态转移方程修改其余的位置
        for(int i=1;i<=nums.length;i++){
            for(int j=0;j<=sum;j++){
                if(j-nums[i-1]<0){
                    dp[i][j]=dp[i-1][j];
                }else{
                    if(dp[i-1][j]||dp[i-1][j-nums[i-1]])
                        dp[i][j]=true;
                }
            }
        }
        //判断目标位置即可
        return dp[nums.length][target];
    }
}
```

###### [ 零钱兑换](https://leetcode.cn/problems/coin-change/)
给你一个整数数组 `<font style="color:rgba(38, 38, 38, 0.75);background-color:rgb(240, 240, 240);">coins</font>` ，表示不同面额的硬币；以及一个整数 `<font style="color:rgba(38, 38, 38, 0.75);background-color:rgb(240, 240, 240);">amount</font>` ，表示总金额。

计算并返回可以凑成总金额所需的**最少的硬币个数** 。如果没有任何一种硬币组合能组成总金额，返回 `<font style="color:rgba(38, 38, 38, 0.75);background-color:rgb(240, 240, 240);">-1</font>` 。

你可以认为每种硬币的数量是无限的

自底向上的消除，设置max是amount + 1，而不是MAX_VALUE是因为怕dp[i - coins[j]] + 1越界

```java
class Solution {
    public int coinChange(int[] coins, int amount) {
        if(coins.length==1){
            return amount%coins[0]==0?amount/coins[0]:-1;
        }
        int [][]dp=new int[coins.length+1][amount+1];
        for(int []d:dp){
            Arrays.fill(d,amount+1);
        }
        dp[0][0]=0;
        for(int i=1;i<=coins.length;i++){
            for(int j=0;j<=amount;j++){
                if(j<coins[i-1]){
                    dp[i][j]=dp[i-1][j];
                }else{
                    dp[i][j]=Math.min(dp[i-1][j],dp[i][j-coins[i-1]]+1);
                }
            }
        }
        return dp[coins.length][amount]==amount+1?-1:dp[coins.length][amount];
    }
}
```

###### [零钱兑换 II](https://leetcode.cn/problems/coin-change-ii/)
给你一个整数数组 `<font style="color:rgba(38, 38, 38, 0.75);background-color:rgb(240, 240, 240);">coins</font>` 表示不同面额的硬币，另给一个整数 `<font style="color:rgba(38, 38, 38, 0.75);background-color:rgb(240, 240, 240);">amount</font>` 表示总金额。

请你计算并返回可以凑成总金额的硬币**组合数**。如果任何硬币组合都无法凑出总金额，返回 `<font style="color:rgba(38, 38, 38, 0.75);background-color:rgb(240, 240, 240);">0</font>` 。

这题属于是完全背包问题，物品的数量没有限制，所以状态方程是dp{i}{j}=dp{i-1}{j}+dp{i}{j-coins[i-1]};并且由于是求可能性，所以这个相加的，首先要将00位置设置为1，和上题一样，对于求可能性问题才这么求解，正常的背包问题第一行和第一列都是0

```java
class Solution {
    public int change(int amount, int[] coins) {
        int[][]dp=new int[coins.length+1][amount+1];
        dp[0][0]=1;
        for(int i=1;i<=coins.length;i++){
            for(int j=0;j<=amount;j++){
                if(j<coins[i-1]){
                    dp[i][j]=dp[i-1][j];
                }else{
                    dp[i][j]=dp[i-1][j]+dp[i][j-coins[i-1]];
                }
            }
        }
        return dp[coins.length][amount];
    }
}
```

###### [目标和](https://leetcode-cn.com/problems/target-sum/)
给你一个非负整数数组 `<font style="color:rgba(38, 38, 38, 0.75);background-color:rgb(240, 240, 240);">nums</font>` 和一个整数 `<font style="color:rgba(38, 38, 38, 0.75);background-color:rgb(240, 240, 240);">target</font>` 。

向数组中的每个整数前添加 `<font style="color:rgba(38, 38, 38, 0.75);background-color:rgb(240, 240, 240);">'+'</font>` 或 `<font style="color:rgba(38, 38, 38, 0.75);background-color:rgb(240, 240, 240);">'-'</font>` ，然后串联起所有整数，可以构造一个 表达式

这是一个计数问题，题意是统计到target的个数，想到用动态规划是因为每个数字就只有两种状态，一种是正，一种是负，所以可以默认首先是全为负的，那对于一个数来说，就一种不变或者变为正，两种状态

首先定义状态：dp[ i ][ j ]表示当轮到第i个数，数组和为j的个数，因为是有负的部分，所以0-sum-1表示负的部分，sum+1到2*sum表示正的部分

状态转移方程：dp(i)(j)=dp(i-1)(j)+dp(i-1)(j-2*nums[i-1])

最后返回的是和为target的，所以要找target的位置sum+target

```java
class Solution {
    public int findTargetSumWays(int[] nums, int target) {
        int sum=0;
        for(int n:nums)
            sum+=n;
        if(target>sum||target<-sum)
            return 0;
        int [][]dp=new int[nums.length+1][2*sum+1];
        dp[0][0]=1;
        for(int i=1;i<=nums.length;i++){
            for(int j=0;j<=2*sum;j++){
                if(j-2*nums[i-1]<0){
                    dp[i][j]=dp[i-1][j];
                }else{
                    dp[i][j]=dp[i-1][j]+dp[i-1][j-2*nums[i-1]];
                }
                
            }
        }
        return dp[nums.length][sum+target];

    }
}
```

##### 爬楼梯问题
都是一条完整的串，可以由几个子串拼接而成，对于这种问题，可以考虑爬楼梯的解法

每一步可以走X，Y，Z个台阶，走完n个台阶，问：

1. 有多少种走法
2. 至少需要多少步
3. 能否刚好走完

**多阶段决策模型**：

阶段个数不固定，每个阶段决策走多少个台阶

**状态定义**：

+ int dp[n+1] dp[i]表示走完i个台阶有多少个走法
+ int dp[n+1] dp[i]表示走完i个台阶有至少需要多少步
+ boolean dp[n+1] dp[n]表示是否刚好走完i个台阶

**状态转移方程**：

到达i个状态，那一步只能是走x，y，z个台阶，也就是从i-x，i-y，i-z转化过来，即由dp[i-x]，dp[i-y]，dp[i-z]转化推导出来

+ dp[i]=dp[i-x]+dp[i-y]+dp[i-z]
+ dp[i]=Math.min(Math.min(dp[i-x],dp[i-y]),dp[i-z])+1
+ dp[i]=dp[i-x] || dp[i-y] || dp[i-z]

###### [爬楼梯](https://leetcode.cn/problems/climbing-stairs/)
假设你正在爬楼梯。需要 `<font style="color:rgba(38, 38, 38, 0.75);background-color:rgb(240, 240, 240);">n</font>` 阶你才能到达楼顶。

每次你可以爬 `<font style="color:rgba(38, 38, 38, 0.75);background-color:rgb(240, 240, 240);">1</font>` 或 `<font style="color:rgba(38, 38, 38, 0.75);background-color:rgb(240, 240, 240);">2</font>` 个台阶。你有多少种不同的方法可以爬到楼顶呢？

```java
class Solution {
    public int climbStairs(int n) {
        if(n==1)
            return 1;
        int []dp=new int[n+1];
        dp[1]=1;
        dp[2]=2;
        for(int i=3;i<=n;i++){
            dp[i]=dp[i-1]+dp[i-2];
        }
        return dp[n];
    }
}
```

###### [零钱兑换](https://leetcode-cn.com/problems/coin-change/)
也可以作为爬楼梯问题，比如现在硬币是1，2，5，amount是11，可以理解为阶梯是11，现在可以走1步，2步5步，最少需要多少步，所以只需要考虑对于当前i，所有可以达到他的状态的最小值即可

```java
class Solution {
    public int coinChange(int[] coins, int amount) {
        int k=coins.length;
        int []dp=new int[amount+1];
        Arrays.fill(dp,amount+1);
        dp[0]=0;
        for(int i=1;i<=amount;i++){
            //这一重循环是有k种方式可以到当前阶段
            for(int j=0;j<k;j++){
                if(i-coins[j]>=0)
                    dp[i]=Math.min(dp[i],dp[i-coins[j]]+1);
            }
        }
        return dp[amount]==amount+1?-1:dp[amount];
    }
}
```

下面这种是以背包问题进行考虑，可以分析一下两者的区别

```java
class Solution {
    public int coinChange(int[] coins, int amount) {
        int [][]dp=new int[coins.length+1][amount+1];
        for(int []d:dp)
            Arrays.fill(d,Integer.MAX_VALUE/2);
        dp[0][0]=0;
        for(int i=1;i<=coins.length;i++){
            for(int j=0;j<=amount;j++){
                if(j<coins[i-1]){
                    dp[i][j]=dp[i-1][j];
                }else{
                    dp[i][j]=Math.min(dp[i-1][j],dp[i][j-coins[i-1]]+1);
                }
            }
        }
        return dp[coins.length][amount]==Integer.MAX_VALUE/2?-1:dp[coins.length][amount];
    }
}
```

###### [零钱兑换 II](https://leetcode.cn/problems/coin-change-ii/description/)
那对于求个数是否可以用爬楼梯问题解决呢？实际上是不行的，对于这道题，是求组合的个数，它是不考虑顺序的，1+5+5和5+1+5对于这道题来说这是一种解法，而对于爬楼梯的思路来说是两种解法，所以是不对的，所以这道题要用背包的思想，就是完全背包的计数问题，背包问题就计数的时候都是考虑哪个硬币多少个，是没有考虑顺序的

换句话来说背包问题是求解不考虑顺序的计数，爬楼梯是求讲顺序的计数

```java
class Solution {
    public int change(int amount, int[] coins) {
        int[][]dp=new int[coins.length+1][amount+1];
        dp[0][0]=1;
        for(int i=1;i<=coins.length;i++){
            for(int j=0;j<=amount;j++){
                if(j<coins[i-1]){
                    dp[i][j]=dp[i-1][j];
                }else{
                    dp[i][j]=dp[i-1][j]+dp[i][j-coins[i-1]];
                }
            }
        }
        return dp[coins.length][amount];
    }
}
```

下面这用爬楼梯思路来做，但不适用于这道题

```java
class Solution {
    public int change(int amount, int[] coins) {
        int[]dp=new int[amount+1];
        dp[0]=1;
        for(int i=1;i<=amount;i++){
            for(int j=0;j<coins.length;j++){
                if(i-coins[j]>=0)
                    dp[i]+=dp[i-coins[j]];
            }
        }
         dp[amount];
    }
}
```

###### [砍竹子 I](https://leetcode-cn.com/problems/jian-sheng-zi-lcof/)
这里用爬楼梯的思路，对于dp[i]来说，其可以由dp[i-1],dp[i-2],...dp[0]转移过来，只是区别在于之间的跨步，所以可以由状态转移方程推导dp[i] = Math.max(dp[i],  j * dp[i - j]);

因为题意是需要切割的，对于n<=3的情况，其实不切割的长度更长，属于是特殊情况，单独列出即可

```java
class Solution {
    public int cuttingBamboo(int bamboo_len) {
        if(bamboo_len==3)
            return 2;
        int[] dp = new int[bamboo_len + 1];
        dp[0]=1;
        for (int i = 1; i <= bamboo_len; i++) {
            for (int j = 1; j <=i; j++) {
                dp[i] = Math.max(dp[i],  j * dp[i - j]);
            }
        }
        return dp[bamboo_len];
    }
}
```

###### [解密数字](https://leetcode-cn.com/problems/ba-shu-zi-fan-yi-cheng-zi-fu-chuan-lcof/)
现有一串神秘的密文 ciphertext，经调查，密文的特点和规则如下：

+ 密文由非负整数组成
+ 数字 0-25 分别对应字母 a-z

请根据上述规则将密文 `<font style="color:rgba(38, 38, 38, 0.75);background-color:rgb(240, 240, 240);">ciphertext</font>` 解密为字母，并返回共有多少种解密结果。

这道题用dfs也能做

```java
class Solution {
    int count=0;
    public int crackNumber(int ciphertext) {
        dfs(Integer.toString(ciphertext),0);
        return count;
    }
    
    public void dfs(String ciphertext,int index){
        if(index==ciphertext.length()){
            count++;
            return;
        }
        for(int i=0;i<2&&index+i<ciphertext.length();i++){
            if(i==0){
                dfs(ciphertext,index+1);
            }else if(ciphertext.charAt(index)=='1'
            ||(ciphertext.charAt(index)=='2'&&ciphertext.charAt(index+1)<'6')){
                dfs(ciphertext,index+2);
            }
        }
    }
}
```

用动态规划的话，就是将上面dfs的思路转化成状态方程，如果能凑成两位数在10-25之间，就可以加上dp[i-2]，否则只能加上dp[i-1]

```java
class Solution {
    public int crackNumber(int ciphertext) {
        String text=Integer.toString(ciphertext);
        int len=text.length();
        int []dp=new int[len+1];
        dp[0]=1;dp[1]=1;
        for(int i=2;i<=len;i++){
            if(text.charAt(i-2)=='1'
            ||(text.charAt(i-2)=='2'&&text.charAt(i-1)<'6'))
                dp[i]=dp[i-1]+dp[i-2];
            else
                dp[i]=dp[i-1];
        }
        return dp[len];
    }
}
```

###### [解码方法](https://leetcode.cn/problems/decode-ways/)
和上题很像，区别在于上题是0-25，这题是1-26

```java
class Solution {
    public int numDecodings(String s) {
        int n = s.length();
        int[] f = new int[n + 1];
        f[0] = 1;
        for (int i = 1; i <= n; ++i) {
            if (s.charAt(i - 1) != '0') {
                f[i] += f[i - 1];
            }
            if (i > 1 && s.charAt(i - 2) != '0' && ((s.charAt(i - 2) - '0') * 10 + (s.charAt(i - 1) - '0') <= 26)) {
                f[i] += f[i - 2];
            }
        }
        return f[n];
    }
}

```

###### [单词拆分](https://leetcode-cn.com/problems/word-break/)
给你一个字符串 `s` 和一个字符串列表 wordDict 作为字典。如果可以利用字典中出现的一个或多个单词拼接出 `s` 则返回 `true`。

也可以使用dfs来做，代码如下

```java
class Solution {
    int []visited;
    HashSet<String>Dict;
    public boolean wordBreak(String s, List<String> wordDict) {
        int n=s.length();
        visited=new int[n];
        Arrays.fill(visited,-1);
        Dict=new HashSet<>(wordDict);
        return dp(s,0);
    }
    public boolean dp(String s,int i){
        if(i==s.length())
            return true;
        //说明之前到过这里失败了 所以就没必要继续下去了
        if(visited[i]!=-1){
            return false;
        }
        visited[i]=0;
        for(int len=1;len+i<=s.length();len++){
            String prix=s.substring(i,i+len);
            if(Dict.contains(prix)){
                boolean flag=dp(s,i+len);
                if(flag==true){
                    return true;
                }
            }
        }
        return false;
    }
}
```

和上题思路一样，不过这道题的匹配字符串长度不一样，所以需要设置一个起始位true的点，接着遍历list里的每个word，设置可达点

对于每个位置是否为true，取决于这个位置是否由其他字典拼接能够到达

在字符串匹配的情况下 dp[i]=dp[i-len1] || dp[i-len2] || dp[i-len3]

```java
class Solution {
    public boolean wordBreak(String s, List<String> wordDict) {
        boolean []dp=new boolean[s.length()+1];
        dp[0]=true;
        for(int i=0;i<=s.length();i++){
            if(dp[i]==true){
                for(String word:wordDict){
                    if(i+word.length()<=s.length()&&s.substring(i,i+word.length()).equals(word))
                        dp[i+word.length()]=true;
                }
            }
        }
        return dp[s.length()];
    }
}
```

###### [交错字符串](https://leetcode.cn/problems/interleaving-string/)
给定三个字符串 `<font style="color:rgba(38, 38, 38, 0.75);background-color:rgb(240, 240, 240);">s1</font>`、`<font style="color:rgba(38, 38, 38, 0.75);background-color:rgb(240, 240, 240);">s2</font>`、`<font style="color:rgba(38, 38, 38, 0.75);background-color:rgb(240, 240, 240);">s3</font>`，请你帮忙验证 `<font style="color:rgba(38, 38, 38, 0.75);background-color:rgb(240, 240, 240);">s3</font>` 是否是由 `<font style="color:rgba(38, 38, 38, 0.75);background-color:rgb(240, 240, 240);">s1</font>` 和 `<font style="color:rgba(38, 38, 38, 0.75);background-color:rgb(240, 240, 240);">s2</font>` 交错 组成的。

两个字符串 `<font style="color:rgba(38, 38, 38, 0.75);background-color:rgb(240, 240, 240);">s</font>` 和 `<font style="color:rgba(38, 38, 38, 0.75);background-color:rgb(240, 240, 240);">t</font>` 交错 的定义与过程如下，其中每个字符串都会被分割成若干 非空 子字符串

对于字符串s3来说，如果第i+j个位置是与s1的i相等的，那么只要dp[i-1][j]是可达的，则dp[i][j]就是可达的，同理s2也是一样的，所以这道题的状态方程就是

dp[i][j]=( dp[i-1][j]&&s1.charAt(i-1)==s3.charAt(i+j-1)  ||  dp[i][j-1]&&s2.charAt(j-1)==s3.charAt(i+j-1) );

```java
class Solution {
    public boolean isInterleave(String s1, String s2, String s3) {
        int n=s1.length();
        int m=s2.length();
        int t=s3.length();
        if(n+m!=t)
            return false;
        boolean[][]dp=new boolean[n+1][m+1];
        dp[0][0]=true;
        for(int i=1;i<=n;i++)
            dp[i][0]=dp[i-1][0]&&s1.charAt(i-1)==s3.charAt(i-1);
        for(int j=1;j<=m;j++)
            dp[0][j]=dp[0][j-1]&&s2.charAt(j-1)==s3.charAt(j-1);
        for(int i=1;i<=n;i++){
            for(int j=1;j<=m;j++){
                dp[i][j]=(dp[i-1][j]&&s1.charAt(i-1)==s3.charAt(i+j-1)
                ||dp[i][j-1]&&s2.charAt(j-1)==s3.charAt(i+j-1));
            }
        }
        return dp[n][m];
    }
}
```

###### [杨辉三角](https://leetcode.cn/problems/pascals-triangle/)
```java
class Solution {
    public List<List<Integer>> generate(int numRows) {
        List<List<Integer>>list=new LinkedList<>();
        int [][]dp=new int[numRows+1][numRows+1];
        for(int i=1;i<=numRows;i++){
            List<Integer>l=new LinkedList<>();
            for(int j=1;j<=i;j++){
                if(j==1||j==i){
                    l.add(1);
                    dp[i][j]=1;
                }else{
                    dp[i][j]=dp[i-1][j-1]+dp[i-1][j];
                    l.add(dp[i][j]);
                }
            }
            list.add(l);
        }
        return list;
    }
}
```

###### [完全平方数](https://leetcode.cn/problems/perfect-squares/)
思路：挨个遍历找之前所有的可以的平方数，求最小

```java
class Solution {
    public int numSquares(int n) {
        int []f=new int[n+1];
        for(int i=1;i<=n;i++){
            int min=Integer.MAX_VALUE;
            for(int j=1;j*j<=i;j++){
                min=Math.min(min,f[i-j*j]);
            }
            f[i]=min+1;
        }
        return f[n];
    }
}
```

###### [乘积最大子数组](https://leetcode.cn/problems/maximum-product-subarray/)
给你一个整数数组 `nums` ，请你找出数组中乘积最大的非空连续子数组（该子数组中至少包含一个数字），并返回该子数组所对应的乘积

思路：要考虑负负得正的问题，所以要维持两个数组，min数组用于存储当前最小的负数，方便当当前这个元素是负数的时候，可以负负得正变成大的值

```java
class Solution {
    public int maxProduct(int[] nums) {
        int n=nums.length;
        int []maxF=new int[n];
        int []minF=new int[n];
        System.arraycopy(nums,0,maxF,0,n);
        System.arraycopy(nums,0,minF,0,n);
        for(int i=1;i<n;i++){
            maxF[i]=Math.max(maxF[i-1]*nums[i],Math.max(minF[i-1]*nums[i],nums[i]));
            minF[i]=Math.min(maxF[i-1]*nums[i],Math.min(minF[i-1]*nums[i],nums[i]));
        }
        int ans = maxF[0];
        for (int i = 1; i < n; ++i) {
            ans = Math.max(ans, maxF[i]);
        }
        return ans;
    }
}
```

##### 路径问题
###### [礼物的最大价值](https://leetcode-cn.com/problems/li-wu-de-zui-da-jie-zhi-lcof/)
现有一个记作二维矩阵 `<font style="color:rgba(38, 38, 38, 0.75);background-color:rgb(240, 240, 240);">frame</font>` 的珠宝架，其中 `<font style="color:rgba(38, 38, 38, 0.75);background-color:rgb(240, 240, 240);">frame[i][j]</font>` 为该位置珠宝的价值。拿取珠宝的规则为：

+ 只能从架子的左上角开始拿珠宝
+ 每次可以移动到右侧或下侧的相邻位置
+ 到达珠宝架子的右下角时，停止拿取

```java
class Solution {
    public int jewelleryValue(int[][] frame) {
        int m=frame.length;
        int n=frame[0].length;
        int[][]dp=new int[m][n];
        dp[0][0]=frame[0][0];
        for(int i=1;i<m;i++)
            dp[i][0]=dp[i-1][0]+frame[i][0];
        for(int j=1;j<n;j++)
            dp[0][j]=dp[0][j-1]+frame[0][j];
        for(int i=1;i<m;i++){
            for(int j=1;j<n;j++){
                dp[i][j]=Math.max(dp[i-1][j],dp[i][j-1])+frame[i][j];
            }
        }
        return dp[m-1][n-1];
    }
}
```

###### [三角形最小路径和](https://leetcode-cn.com/problems/triangle/)
给定一个三角形 `<font style="color:rgba(38, 38, 38, 0.75);background-color:rgb(240, 240, 240);">triangle</font>` ，找出自顶向下的最小路径和。

每一步只能移动到下一行中相邻的结点上。相邻的结点在这里指的是下标与上一层结点下标相同或者等于上一层结点下标 + 1 的两个结点。也就是说，如果正位于当前行的下标 `<font style="color:rgba(38, 38, 38, 0.75);background-color:rgb(240, 240, 240);">i</font>` ，那么下一步可以移动到下一行的下标 `<font style="color:rgba(38, 38, 38, 0.75);background-color:rgb(240, 240, 240);">i</font>` 或 `<font style="color:rgba(38, 38, 38, 0.75);background-color:rgb(240, 240, 240);">i + 1</font>` 。

这道题和上道题思路一样，只不过是第0列和对角线的值是固定的，接着进行判断求最值

```java
class Solution {
    public int minimumTotal(List<List<Integer>> triangle) {
        int n=triangle.size();
        int [][]dp=new int[n][n];
        dp[0][0]=triangle.get(0).get(0);
        for(int i=1;i<n;i++){
            dp[i][0]=dp[i-1][0]+triangle.get(i).get(0);
            dp[i][i]=dp[i-1][i-1]+triangle.get(i).get(triangle.get(i).size()-1);
        }
        for(int i=2;i<n;i++){
            for(int j=1;j<i;j++){
                dp[i][j]=Math.min(dp[i-1][j-1],dp[i-1][j])+triangle.get(i).get(j);
            }
        }
        int min=Integer.MAX_VALUE;
        for(int i=0;i<n;i++){
            if(dp[n-1][i]<min)
                min=dp[n-1][i];
        }
        return min;
    }
}
```

###### [不同路径](https://leetcode.cn/problems/unique-paths/)
一个机器人位于一个 `<font style="color:rgba(38, 38, 38, 0.75);background-color:rgb(240, 240, 240);">m x n</font>` 网格的左上角 （起始点在下图中标记为 “Start” ）。

机器人每次只能向下或者向右移动一步。机器人试图达到网格的右下角（在下图中标记为 “Finish” ）。

问总共有多少条不同的路径？

```java
class Solution {
    public int uniquePaths(int m, int n) {
        int[][]dp=new int[m][n];
        for(int i=0;i<n;i++)dp[0][i]=1;
        for(int i=0;i<m;i++)dp[i][0]=1;
        for(int i=1;i<m;i++){
            for(int j=1;j<n;j++){
                dp[i][j]=dp[i-1][j]+dp[i][j-1];
            }
        }
        return dp[m-1][n-1];
    }
}
```

###### [不同路径 II](https://leetcode.cn/problems/unique-paths-ii/)
一个机器人位于一个 `<font style="color:rgba(38, 38, 38, 0.75);background-color:rgb(240, 240, 240);">m x n</font>` 网格的左上角 （起始点在下图中标记为 “Start” ）。

机器人每次只能向下或者向右移动一步。机器人试图达到网格的右下角（在下图中标记为 “Finish”）。

现在考虑网格中有障碍物。那么从左上角到右下角将会有多少条不同的路径？

和上题思路一样，只不过加了障碍，对于第0行和第0列来说，障碍之后的都为0，对于其他区域，如果是障碍就为0，如果不是障碍就等于上面和左边的和

```java
class Solution {
    public int uniquePathsWithObstacles(int[][] obstacleGrid) {
        if(obstacleGrid[0][0]==1)
            return 0;
        int m=obstacleGrid.length,n=obstacleGrid[0].length;
        int[][]dp=new int[m][n];
        for(int i=0;i<m;i++){
            if(obstacleGrid[i][0]==1)
                break;
            dp[i][0]=1;
        }
        for(int j=1;j<n;j++){
            if(obstacleGrid[0][j]==1)
                break;
            dp[0][j]=1;
        }
        for(int i=1;i<m;i++){
            for(int j=1;j<n;j++){
                if(obstacleGrid[i][j]!=1)
                    dp[i][j]=dp[i-1][j]+dp[i][j-1];
                else
                    dp[i][j]=0;
            }
        }
        return dp[m-1][n-1];
    }
}
```

###### [最小路径和](https://leetcode.cn/problems/minimum-path-sum/)
给定一个包含非负整数的 `m x n` 网格 `grid` ，请找出一条从左上角到右下角的路径，使得路径上的数字总和为最小

```java
class Solution {
    public int minPathSum(int[][] grid) {
        int[][]dp=new int[grid.length][grid[0].length];
        dp[0][0]=grid[0][0];
        for(int i=1;i<grid.length;i++){
            dp[i][0]=dp[i-1][0]+grid[i][0];
        }
        for(int j=1;j<grid[0].length;j++){
            dp[0][j]=dp[0][j-1]+grid[0][j];
        }
        for(int i=1;i<grid.length;i++){
            for(int j=1;j<grid[0].length;j++){
                dp[i][j]=Math.min(dp[i-1][j],dp[i][j-1])+grid[i][j];
            }
        }
        return dp[grid.length-1][grid[0].length-1];
    }
}
```

##### 打家劫舍问题
前后两个阶段是有牵制关系的，所以需要有一维作为状态设置，通过**上一个阶段推测不同决策的最优解**

**这类问题都是选与不选，后一个阶段求选与不选的时候，就根据前一个阶段的选与不选求最优，将所有阶段的所有状态都更新一次**

**对于这类问题，首先找到不可达状态，设置为-Inf，对于一般的题只有第-1天的持有状态是不可达的，像股票买卖Ⅲ、Ⅳ，因为多了一个限制交易次数，所以要新增一个维度，多一层for循环，交易次数不可能为负的，所以要设置为不可达，其他的都一样**

###### [打家劫舍](https://leetcode.cn/problems/house-robber/)
这道题设置的规则在于不能取连续的，这不像背包问题，背包问题是可以连续也可以不连续，所以就不需要考虑这个问题，每次只需要考虑要不要加自己即可。这类题因为需要考虑，所以需要设置一个状态数组，表示每一个数的两种状态，因为后面的dp是需要用到前面的状态的

具体步骤如下：

1、构建多阶段决策模型  
	n个房屋对应n个阶段，每个阶段决定一个房屋偷还是不偷，两种决策：偷，不偷  
2、定义状态  
	不能只记录每个阶段决策完之后，小偷可偷的最大金额，需要记录**不同决策对应的最大金额，**也就是：这个房屋偷-对应的最大金额；这个房屋不偷-对应的最大金额  
	int dp[n][2]记录每个阶段的状态  
	dp[i][0]：表示第i个物品不偷，当下的最大金额  
	dp[i][1]：表示第i个物品偷，当下的最大金额  
3、定义状态转移方程	  
	dp[i][0]=Math.max(dp[i-1][1]，dp[i-1][0])

 dp[i][1]=dp[i-1][0]+nums[i]

```java
class Solution {
    public int rob(int[] nums) {
        int n=nums.length;
        int [][]dp=new int[n+1][2];
        dp[0][1]=Integer.MIN_VALUE;
        for(int i=1;i<=n;i++){
            dp[i][0]=Math.max(dp[i-1][1],dp[i-1][0]);
            dp[i][1]=dp[i-1][0]+nums[i-1];
        }
        return Math.max(dp[n][0],dp[n][1]);
    }
}
```

###### [打家劫舍 II](https://leetcode.cn/problems/house-robber-ii/)
这道题的区别在于头和尾是不能相邻的，换个思路想那这里就有两种情况：有头就没有尾，有尾就没有头，这两种情况对应了两个范围，那么就可以求这两个范围的最大值，接着进行比较即可

```java
class Solution {
    public int rob(int[] nums) {
        if(nums.length==1)
            return nums[0];
        else if(nums.length==2)
            return Math.max(nums[0],nums[1]);
        return Math.max(
            robRange(nums,0,nums.length-2),
            robRange(nums,1,nums.length-1)
            );
    }
    public int robRange(int[]nums,int start,int end){
        int n=end-start+1;
        int [][]dp=new int[n+1][2];
        dp[0][1]=Integer.MIN_VALUE;
        for(int i=1;i<=n;i++){
            dp[i][0]=Math.max(dp[i-1][1],dp[i-1][0]);
            dp[i][1]=dp[i-1][0]+nums[i+start-1];
        }
        return Math.max(dp[n][0],dp[n][1]);
    }
}
```

###### [ 打家劫舍 III](https://leetcode.cn/problems/house-robber-iii/)
考虑到是二叉树，所以要用递归进行遍历，因为这种是多个分支的结果进行返回的，所以递归函数要带返回值，这种需要考虑这个递归函数返回值是什么，对左右子树的返回值进行处理，然后返回

这里返回值是一个数组，第一位是含有该根节点时最佳值，第二位是不含该根节点的最佳值，所以递归其子树，得到子树对应的值，然后求自己的。数组版是从左到右，这个因为不方便子节点的时候判断父节点，方便父节点的时候判断子节点，所以是反向的

具体步骤如下：  
1、构建多阶段决策模型  
	树形dp基于树这种数据结构上做状态推导，一般都是**从下往上推，子节点状态推导父节点状态**。一般都是  
基于**后序遍历**来实现。  
2、定义状态  
	每个节点有两个状态：偷，不偷  
	int money[2]表示每个节点的状态  
	money[0]：表示选择不偷此节点，当下最大金额  
	money[1]：表示选择偷此节点，当下的最大金额  
3、定义状态转移方程  
	money[0]=max(leftmoney[0]，leftmoney[1]]+max(rightmoney[0]，rightmoney[1])  
	money[1]=leftmoney[0]+rightmoney[0]+root.val

```java
class Solution {

    public int rob(TreeNode root) {
        int []res=dfs(root);
        return res[0]>res[1]?res[0]:res[1];
    }

    public int[] dfs(TreeNode node){
        if(node==null)
            return new int[]{0,0};
        int []left=dfs(node.left);
        int []right=dfs(node.right);
        //v1代表自己要偷，那么子节点就不能偷
        int v1=left[1]+right[1]+node.val;
        int v2=Math.max(left[0],left[1])+Math.max(right[0],right[1]);
        return new int[]{v1,v2};
    }
}
```

##### 最长递增子序列
###### [最长递增子序列](https://leetcode.cn/problems/longest-increasing-subsequence/)
这道题思路比较简单，状态方程就是找到当前节点之前所有比他小的节点，比较这些点的长度。dp[i]=Math.max(dp[i],dp[j]+1)

```java
class Solution {
    public int lengthOfLIS(int[] nums) {
        int n=nums.length;
        int []dp=new int[n];
        Arrays.fill(dp,1);
        for(int i=0;i<n;i++){
            for(int j=0;j<i;j++){
                if(nums[j]<nums[i]){
                    dp[i]=Math.max(dp[i],dp[j]+1);
                }
            }
        }
        int max=Integer.MIN_VALUE;
        for(int res:dp)
            if(max<res)
                max=res;
        return max;
    }
}
```

###### [俄罗斯套娃信封问题](https://leetcode.cn/problems/russian-doll-envelopes/)
思路是：首先对信封的宽度进行升序排序，如果宽度相同的信封就使用降序排序，这样只需要对信封的高度进行最长递增子序列的动态规划即可

```java
class Solution {
    public int maxEnvelopes(int[][] envelopes) {
        int n=envelopes.length;
        Arrays.sort(envelopes, new Comparator<int[]>() 
        {
            public int compare(int[] a, int[] b) {
                return a[0] == b[0] ? 
                    b[1] - a[1] : a[0] - b[0];
            }
        });
        int []heights=new int[n];
        for(int i=0;i<n;i++){
            heights[i]=envelopes[i][1];
        }
        return dp_array(heights);

    }

    public int dp_array(int[]heights){
        int n=heights.length;
        int []dp=new int[n];
        Arrays.fill(dp,1);
        for(int i=0;i<n;i++){
            for(int j=0;j<i;j++){
                if(heights[j]<heights[i]){
                    dp[i]=Math.max(dp[i],dp[j]+1);
                }
            }
        }
        int max=-1;
        for(int i=0;i<n;i++){
            if(max<dp[i])
                max=dp[i];
        }
        return max;
    }
}
```

###### [下降路径最小和](https://leetcode.cn/problems/minimum-falling-path-sum/)
给你一个 `n x n` 的 方形 整数数组 `matrix` ，请你找出并返回通过 `matrix` 的下降路径 的 最小和 。

下降路径 可以从第一行中的任何元素开始，并从每一行中选择一个元素。在下一行选择的元素和当前行所选元素最多相隔一列（即位于正下方或者沿对角线向左或者向右的第一个元素）。具体来说，位置 `(row, col)` 的下一个元素应当是 `(row + 1, col - 1)`、`(row + 1, col)` 或者 `(row + 1, col + 1)` 。

类似于背包问题，dp数组中后面的值由上一行的三个dp值的最小值和该值的和，注意判断越界

```java
class Solution {
    public int minFallingPathSum(int[][] matrix) {
        int n=matrix.length;
        int [][]dp=new int[n][n];
        for(int i=0;i<n;i++)
            dp[0][i]=matrix[0][i];
        for(int i=1;i<n;i++){
            for(int j=0;j<n;j++){
                int l_value=j==0?Integer.MAX_VALUE:dp[i-1][j-1];
                int m_value=dp[i-1][j];
                int r_value=j==n-1?Integer.MAX_VALUE:dp[i-1][j+1];
                dp[i][j]=Math.min(Math.min(l_value,m_value),r_value)+matrix[i][j];
            }
        }
        int res=Integer.MAX_VALUE;
        for(int i=0;i<n;i++){
            if(res>dp[n-1][i])
                res=dp[n-1][i];
        }
        return res;
    }
}
```

###### [最大子数组和](https://leetcode.cn/problems/maximum-subarray/)
具有最大和的连续子数组

解法1️⃣：使用动态规划，考虑使用dp数组，状态方程的话，考虑每一个dp的值表示以num[i]结尾的最大子数组的和，那么就有两种可能要么只有自己，要么和前一个相加，

```java
class Solution {
    public int maxSubArray(int[] nums) {
        int n=nums.length;
        int []dp=new int[n];
        dp[0]=nums[0];
        for(int i=1;i<n;i++){
            dp[i]=Math.max(nums[i],dp[i-1]+nums[i]);
        }
        int res=Integer.MIN_VALUE;
        for(int i=0;i<n;i++){
            if(res<dp[i])
                res=dp[i];
        }
        return res;
    }
}
```

解法2️⃣：对于这种连续的子串，就可以考虑使用滑动窗口，对于滑动窗口的方法需要考虑什么时候窗口收缩，这道题考虑是windsum小于0的时候收缩，因为窗口内的值小于0，对于新添加的元素就没必要加这一部分，只会更小，所以当窗口内的值小于0时，就不断收缩窗口

```java
class Solution {
    public int maxSubArray(int[] nums) {
        int left=0;int right=0;
        int windsum=0;
        int res=Integer.MIN_VALUE;
        while(right<nums.length){
            windsum+=nums[right];
            right++;
            if(windsum>res)
                res=windsum;
            while(windsum<0){
                windsum-=nums[left];
                left++;
            }
        }
        return res;
    }
}
```

##### 前缀问题
###### [单词拆分](https://leetcode.cn/problems/word-break/)
给你一个字符串 `s` 和一个字符串列表 `wordDict` 作为字典。如果可以利用字典中出现的一个或多个单词拼接出 `s` 则返回 `true`。

思路就是深度遍历，从s的0的位置开始，for循环，判断前len部分是不是wordDict中的，如果是就dfs遍历他的下一部分，要是没有的话循环结束就会返回false

```java
class Solution {
    int []visited;
    HashSet<String>Dict;
    public boolean wordBreak(String s, List<String> wordDict) {
        int n=s.length();
        visited=new int[n];
        Arrays.fill(visited,-1);
        Dict=new HashSet<>(wordDict);
        return dp(s,0);
    }
    public boolean dp(String s,int i){
        if(i==s.length())
            return true;
        if(visited[i]!=-1){
            return false;
        }
        visited[i]=0;
        for(int len=1;len+i<=s.length();len++){
            String prix=s.substring(i,i+len);
            if(Dict.contains(prix)){
                boolean flag=dp(s,i+len);
                if(flag==true){
                    return true;
                }
            }
        }
        return false;
    }
}
```

###### [单词拆分 II](https://leetcode.cn/problems/word-break-ii/)
给定一个字符串 `s` 和一个字符串字典 wordDict ，在字符串 `s` 中增加空格来构建一个句子，使得句子中所有的单词都在词典中。以任意顺序 返回所有这些可能的句子

和上一题一样的思路，只不过需要使用一个res数组将结果进行添加，还是把问题看作开头+子问题的结构，Listsubs=dp(s,i+len);就会返回所有的子结构的组合，只需要for循环中将前缀和子结构进行一个组合就可以得到最后的结果

```java
class Solution {
    HashSet<String>Dict;
    public List<String> wordBreak(String s, List<String> wordDict) {
        Dict=new HashSet<>(wordDict);
        return dp(s,0);
    }
    public List<String> dp(String s,int i){
        List<String>res=new LinkedList<>();
        if(i==s.length()){
            res.add("");
            return res;
        }
        for(int len=1;len+i<=s.length();len++){
            String ss=s.substring(i,i+len);
            if(Dict.contains(ss)){
                List<String>subs=dp(s,i+len);
                for(String r:subs){
                    if(r.isEmpty()){
                        res.add(ss);
                    }else{
                        res.add(ss+" "+r);
                    }
                }
            }
        }
        return res;
    }
}
```

##### 匹配问题
###### [编辑距离](https://leetcode.cn/problems/edit-distance/)
给你两个单词 `<font style="color:rgba(38, 38, 38, 0.75);background-color:rgb(240, 240, 240);">word1</font>` 和 `<font style="color:rgba(38, 38, 38, 0.75);background-color:rgb(240, 240, 240);">word2</font>`， 请返回将 `_<font style="color:rgba(38, 38, 38, 0.75);background-color:rgb(240, 240, 240);">word1</font>_` 转换成 `_<font style="color:rgba(38, 38, 38, 0.75);background-color:rgb(240, 240, 240);">word2</font>_` 所使用的最少操作数  。

你可以对一个单词进行如下三种操作：

+ 插入一个字符
+ 删除一个字符
+ 替换一个字符

解决思路，两个字符串，从最后开始同时进行遍历，如果两个字符不等，那么就有三种方式，插入删除变化，对应了i，j不同的变化，变化一次步骤就加1

最简单的思路就是使用暴力解法，就递归进行遍历，然后每次charAt不等的时候就三种方式都遍历，求操作步数最小的值

```java
class Solution {
    public int minDistance(String word1, String word2) {
        int m=word1.length();
        int n=word2.length();
        return dp(word1,m-1,word2,n-1);
    }
    public int dp(String word1,int w1,String word2,int w2){
        if(w1==-1)
            return w2+1;
        if(w2==-1)
            return w1+1;
        if(word1.charAt(w1)==word2.charAt(w2)){
            return dp(word1,w1-1,word2,w2-1);
        }else{
            return min(
                dp(word1,w1-1,word2,w2)+1,
                dp(word1,w1-1,word2,w2-1)+1,
                dp(word1,w1,word2,w2-1)+1
            );
        }
    }

    public int min(int a,int b,int c){
        return Math.min(a,Math.min(b,c));
    }
}
```

上一种方式超时了，可以考虑下面使用dp数组的方式，思路是一样的，只不过首先设置dp数组第0行和第0列的值，代表假如一个为空的，另一个在对应字符串长度时，需要多少步才能相等

![](https://cdn.jsdelivr.net/gh/ddyycc123/imageloader@main/image-20231211215441173.png)

接着就是完善dp数组的过程，和dfs遍历一样，首先判断两个对应点的时候是否相等，进行dp数组的更新，每完善一个dp的值代表了当两个字符串分别为对应位置i ，j长度时需要的最低的步数

```java
class Solution {
    public int minDistance(String word1, String word2) {
        int m=word1.length();
        int n=word2.length();
        int [][]dp=new int[m+1][n+1];
        for(int i=0;i<=n;i++)
            dp[0][i]=i;
        for(int j=0;j<=m;j++)
            dp[j][0]=j;
        for(int i=1;i<=m;i++){
            for(int j=1;j<=n;j++){
                if(word1.charAt(i-1)==word2.charAt(j-1)){
                    dp[i][j]=dp[i-1][j-1];
                }else{
                    dp[i][j]=min(
                        dp[i-1][j]+1,
                        dp[i][j-1]+1,
                        dp[i-1][j-1]+1
                    );

                }
            }
        }
        return dp[m][n]; 
    }
    public int min(int a,int b,int c){
        return Math.min(a,Math.min(b,c));
    }
}
```

###### [最长公共子序列](https://leetcode.cn/problems/longest-common-subsequence/)
思路类似于编辑距离，因为看到是两个字符串，自然就想到了编辑距离问题，将两个字符串创建一个二维的dp数组，然后状态方程也和编辑距离的类似，dpij就是两个字符串长度为i，j的时候的最值

```java
class Solution {
    public int longestCommonSubsequence(String text1, String text2) {
        int m=text1.length();
        int n=text2.length();
        int [][]dp=new int[m+1][n+1];
        for(int i=0;i<=m;i++)
            dp[i][0]=0;
        for(int i=0;i<=n;i++)
            dp[0][i]=0;
        for(int i=1;i<=m;i++){
            for(int j=1;j<=n;j++){
                if(text1.charAt(i-1)==text2.charAt(j-1)){
                    dp[i][j]=dp[i-1][j-1]+1;
                }else{
                    dp[i][j]=Math.max(dp[i-1][j],dp[i][j-1]);
                }
            }
        }
        return dp[m][n];
    }
}
```

###### [两个字符串的删除操作](https://leetcode.cn/problems/delete-operation-for-two-strings/)
给定两个单词 `<font style="color:rgba(38, 38, 38, 0.75);background-color:rgb(240, 240, 240);">word1</font>` 和 `<font style="color:rgba(38, 38, 38, 0.75);background-color:rgb(240, 240, 240);">word2</font>` ，返回使得 `<font style="color:rgba(38, 38, 38, 0.75);background-color:rgb(240, 240, 240);">word1</font>` 和  `<font style="color:rgba(38, 38, 38, 0.75);background-color:rgb(240, 240, 240);">word2</font>` 相同所需的最小步数。

每步 可以删除任意一个字符串中的一个字符。

思路一样的，对于这种两个字符串进行操作匹配的，都可以利用这种方式求解，先画一个dp表，然后根据其中的规律写状态方程，然后代码实现

```java
class Solution {
    public int minDistance(String word1, String word2) {
        int m=word1.length();
        int n=word2.length();
        int [][]dp=new int[m+1][n+1];
        for(int i=0;i<=m;i++)
            dp[i][0]=i;
        for(int j=0;j<=n;j++)
            dp[0][j]=j;
        for(int i=1;i<=m;i++){
            for(int j=1;j<=n;j++){
                if(word1.charAt(i-1)==word2.charAt(j-1))
                    dp[i][j]=dp[i-1][j-1];
                else{
                    dp[i][j]=Math.min(dp[i-1][j],dp[i][j-1])+1;
                }
            }
        }
        return dp[m][n];
    }
}
```

###### [两个字符串的最小ASCII删除和](https://leetcode.cn/problems/minimum-ascii-delete-sum-for-two-strings/)
给定两个字符串`s1` 和 `s2`，返回 使两个字符串相等所需删除字符的 ASCII 值的最小和 

这道题的区别是使用ascii码，我最开始算错了，是因为在最开始设置数组的第一行和第一列的时候，是直接设置的s1.charAt(i-1)，这应该是**叠加**的

```java
class Solution {
    public int minimumDeleteSum(String s1, String s2) {
        int m=s1.length();
        int n=s2.length();
        int [][]dp=new int[m+1][n+1];
        for(int i=1;i<=m;i++)
            dp[i][0]=s1.charAt(i-1)+dp[i-1][0];
        for(int i=1;i<=n;i++)
            dp[0][i]=s2.charAt(i-1)+dp[0][i-1];
        for(int i=1;i<=m;i++){
            for(int j=1;j<=n;j++){
                if(s1.charAt(i-1)==s2.charAt(j-1)){
                    dp[i][j]=dp[i-1][j-1];
                }else{
                    dp[i][j]=Math.min(
                        dp[i-1][j]+s1.charAt(i-1),
                        dp[i][j-1]+s2.charAt(j-1)
                    );
                }
            }
        }
        return dp[m][n];
    }
}
```

###### [最长回文子序列](https://leetcode.cn/problems/longest-palindromic-subsequence/)
给你一个字符串 `s` ，找出其中最长的回文子序列，并返回该序列的长度。

子序列定义为：不改变剩余字符顺序的情况下，删除某些字符或者不删除任何字符形成的一个序列

解法1️⃣：因为是求回文子序列，考虑将s反转成一个新的字符串，然后求这两个字符串的最长公共子序列

```java
class Solution {
    public int longestPalindromeSubseq(String s) {
        StringBuilder sb=new StringBuilder(s);
        String s2=sb.reverse().toString();
        int n=s.length();
        int [][]dp=new int[n+1][n+1];
        for(int i=1;i<=n;i++)
            for(int j=1;j<=n;j++){
                if(s.charAt(i-1)==s2.charAt(j-1)){
                    dp[i][j]=dp[i-1][j-1]+1;
                }else{
                    dp[i][j]=Math.max(
                        dp[i-1][j],
                        dp[i][j-1]
                    );
                }
            }
        return dp[n][n];
    }
}
```

解法2️⃣

对于一个回文子序列来说，去掉头尾应仍是回文子序列，按照这个为条件来建立状态方程

dp ij表示从i到j范围内字符串最长的回文子序列，只有当0≤_i_≤_j_<_n_ 才会有子序列，否则为0，所以整个dp数组就是一个矩阵的上半角

对于任何长度为1的子序列来说，都是回文子序列，所以dpii都为1，然后计算从上半三角的地步开始计算，如果i，j位置的字符相等那么长度等于去掉这两个之后的最长回文子序列➕2，如果不相等，i，j不可能同时作为同一个回文子序列的首尾，所以就求最大

```java
class Solution {
    public int longestPalindromeSubseq(String s) {
        int n=s.length();
        int [][]dp=new int[n][n];
        for(int i=n-1;i>=0;i--){
            dp[i][i]=1;
            char c1=s.charAt(i);
            for(int j=i+1;j<n;j++){
                char c2=s.charAt(j);
                if(c1==c2){
                    dp[i][j]=dp[i+1][j-1]+2;
                }else{
                    dp[i][j]=Math.max(
                        dp[i+1][j],
                        dp[i][j-1]
                    );
                }
            }
        }
        return dp[0][n-1];
    }
}
```

###### [访问数组中的位置使分数最大](https://leetcode.cn/problems/visit-array-positions-to-maximize-score/)
给你一个下标从 0 开始的整数数组 `<font style="color:rgba(38, 38, 38, 0.75);background-color:rgb(240, 240, 240);">nums</font>` 和一个正整数 `<font style="color:rgba(38, 38, 38, 0.75);background-color:rgb(240, 240, 240);">x</font>` 。

你 一开始 在数组的位置 `<font style="color:rgba(38, 38, 38, 0.75);background-color:rgb(240, 240, 240);">0</font>` 处，你可以按照下述规则访问数组中的其他位置：

+ 如果你当前在位置 `<font style="color:rgba(38, 38, 38, 0.75);background-color:rgb(240, 240, 240);">i</font>` ，那么你可以移动到满足 `<font style="color:rgba(38, 38, 38, 0.75);background-color:rgb(240, 240, 240);">i < j</font>` 的 任意 位置 `<font style="color:rgba(38, 38, 38, 0.75);background-color:rgb(240, 240, 240);">j</font>` 。
+ 对于你访问的位置 `<font style="color:rgba(38, 38, 38, 0.75);background-color:rgb(240, 240, 240);">i</font>` ，你可以获得分数 `<font style="color:rgba(38, 38, 38, 0.75);background-color:rgb(240, 240, 240);">nums[i]</font>` 。
+ 如果你从位置 `<font style="color:rgba(38, 38, 38, 0.75);background-color:rgb(240, 240, 240);">i</font>` 移动到位置 `<font style="color:rgba(38, 38, 38, 0.75);background-color:rgb(240, 240, 240);">j</font>` 且 `<font style="color:rgba(38, 38, 38, 0.75);background-color:rgb(240, 240, 240);">nums[i]</font>` 和 `<font style="color:rgba(38, 38, 38, 0.75);background-color:rgb(240, 240, 240);">nums[j]</font>` 的奇偶性不同，那么你将失去分数 `<font style="color:rgba(38, 38, 38, 0.75);background-color:rgb(240, 240, 240);">x</font>` 。

请你返回你能得到的 最大 得分之和。

思路：只需要维持奇偶两个变量即可

```java
class Solution {
    public long maxScore(int[] nums, int x) {
        long res=nums[0],cur;
        long []dp={Integer.MIN_VALUE,Integer.MIN_VALUE};
        dp[nums[0]%2]=nums[0];
        for(int i=1;i<nums.length;i++){
            int flag=nums[i]%2;
            cur=Math.max(dp[flag]+nums[i],dp[1-flag]+nums[i]-x);
            res=Math.max(cur,res);
            dp[flag]=Math.max(dp[flag],cur);
        }
        return res;
    }
}
```

思路2️⃣

```java
class Solution {
    public long maxScore(int[] nums, int x) {
        long evenMax = Integer.MIN_VALUE;
        long oddMax = Integer.MIN_VALUE;
        if (nums[0] % 2 == 0) evenMax = nums[0];
        else oddMax = nums[0];
        for (int i = 1; i < nums.length; i++) {
            if (nums[i] % 2 == 0) {
                evenMax = Math.max(evenMax + nums[i], oddMax + nums[i] - x);
            } else {
                oddMax = Math.max(oddMax + nums[i], evenMax + nums[i] - x);
            }
        }
        return Math.max(evenMax, oddMax);
    }
}
```

##### [股票买卖问题](https://labuladong.github.io/algo/di-er-zhan-a01c6/yong-dong--63ceb/yi-ge-fang-3b01b/)
股票问题，如果考虑是有k次交易（1次买入+1次卖出为一次交易）的话，那就设置为三维数组，第一维表示天数，第二维表示交易次数的限制，第三维有0-1两个值，0代表当前购入股票，1代表当前已购入股票，最后的最优结果结果最后一天没有购入股票的值

对于basecase来说，存在以下情况

表示第-1天还没有进行交易，所以对于不含股票的情况，利润为0，对于含有股票的情况，是不可能含有股票的，设置为负无穷

dp(-1)(..)(0)=0

dp(-1)(..)(1)=Integer.MIN_VALUE

表示如果交易次数限制为0的话，是不允许交易的，那么这个时候不含股票的情况利润为0，含有股票的情况是不可能的，设置为负无穷

dp(..)(0)(0)=0

dp(..)(0)(1)=Integer.MIN_VALUE

###### [买卖股票的最佳时机](https://leetcode.cn/problems/best-time-to-buy-and-sell-stock/)
给定一个数组 `<font style="color:rgba(38, 38, 38, 0.75);background-color:rgb(240, 240, 240);">prices</font>` ，它的第 `<font style="color:rgba(38, 38, 38, 0.75);background-color:rgb(240, 240, 240);">i</font>` 个元素 `<font style="color:rgba(38, 38, 38, 0.75);background-color:rgb(240, 240, 240);">prices[i]</font>` 表示一支给定股票第 `<font style="color:rgba(38, 38, 38, 0.75);background-color:rgb(240, 240, 240);">i</font>` 天的价格。

你只能选择 某一天 买入这只股票，并选择在 未来的某一个不同的日子 卖出该股票。设计一个算法来计算你所能获取的最大利润

此题中限制了只能交易一次，所以三维数组中的第二维可以省略，如果第二维减1的话，就是0，basecase中都是-prices[i]，所以可以不用添加。

其中的if是因为basecase中出现-1的情况，所以只能自己把这个时候的i给赋值了

```java
class Solution {
    public int maxProfit(int[] prices) {
        int n=prices.length;
        int [][]dp=new int[n+1][2];
        dp[0][1]=Integer.MIN_VALUE;
        for(int i=1;i<=n;i++){
            dp[i][0]=Math.max(dp[i-1][0],dp[i-1][1]+prices[i-1]);
            dp[i][1]=Math.max(dp[i-1][1],-prices[i-1]);
        }
        return Math.max(dp[n][0],dp[n][1]);
    }
}
```

###### [买卖股票的最佳时机 II](https://leetcode.cn/problems/best-time-to-buy-and-sell-stock-ii/)
对于这道题，与前一题的区别在于可以交易无限次，所以也不需要三维数组中的第二维交易的限制，直接写状态方程即可

```java
class Solution {
    public int maxProfit(int[] prices) {
        int n=prices.length;
        int [][]dp=new int[n+1][2];
        dp[0][1]=Integer.MIN_VALUE;
        for(int i=1;i<=n;i++){
            dp[i][0]=Math.max(dp[i-1][0],dp[i-1][1]+prices[i-1]);
            dp[i][1]=Math.max(dp[i-1][1],dp[i-1][0]-prices[i-1]);
        }
        return Math.max(dp[n][0],dp[n][1]);
    }
}
```

###### [买卖股票的最佳时机含冷冻期](https://leetcode.cn/problems/best-time-to-buy-and-sell-stock-with-cooldown/)
这道题相对于上道题来说，区别在于多了一个冷冻期，即前一天卖了股票，今天是不能买股票的

就相当于有三种状态：

1. 今天有股票，可能是昨天就有的，或者今天买的
2. 今天没有股票且处于冷冻期，那么只有可能是今天卖了股票
3. 今天没有股票且没有处于冷冻期，那么就有可能是前一天是冷冻期或者前一天非冷冻期但也没有股票

> 这里说的第i天处于冷冻期，指的是第i天卖了股票，第i+1天不能买股票
>

```java
class Solution {
    public int maxProfit(int[] prices) {
        int n=prices.length;
        int [][]dp=new int[n+1][3];
        dp[0][1]=Integer.MIN_VALUE;
        for(int i=1;i<=n;i++){
            //没股票
            dp[i][0]=Math.max(dp[i-1][0],dp[i-1][2]);
            //有股票
            dp[i][1]=Math.max(dp[i-1][1],dp[i-1][0]-prices[i-1]);
            //冷冻期
            dp[i][2]=dp[i-1][1]+prices[i-1];
        }
        return Math.max(Math.max(dp[n][0],dp[n][1]),dp[n][2]);
    }
}
```

###### [买卖股票的最佳时机含手续费](https://leetcode.cn/problems/best-time-to-buy-and-sell-stock-with-transaction-fee/)
这道题也是不限制交易次数，与前面的题相比，就是多了一个交易费，只需要在每次完成交易的时候添加交易费用即可

```java
class Solution {
    public int maxProfit(int[] prices, int fee) {
        int n=prices.length;
        int [][]dp=new int[n+1][2];
        dp[0][1]=Integer.MIN_VALUE/2;
        for(int i=1;i<=n;i++){
            dp[i][1]=Math.max(dp[i-1][1],dp[i-1][0]-prices[i-1]);
            dp[i][0]=Math.max(dp[i-1][0],dp[i-1][1]+prices[i-1]-fee);
        }
        return Math.max(dp[n][0],dp[n][1]);
    }
}
```

###### [买卖股票的最佳时机 III](https://leetcode.cn/problems/best-time-to-buy-and-sell-stock-iii/)
给定一个数组，它的第 `<font style="color:rgba(38, 38, 38, 0.75);background-color:rgb(240, 240, 240);">i</font>` 个元素是一支给定的股票在第 `<font style="color:rgba(38, 38, 38, 0.75);background-color:rgb(240, 240, 240);">i</font>` 天的价格。

设计一个算法来计算你所能获取的最大利润。你最多可以完成 两笔 交易。

之前是先初始化第0天的阶段，接着处理后面的天数，但这样的前提是prices不为空，所以为了避免这种情况出现，就初始化第-1天，后面每天进行判断

初始化-1天的话，有一些状态就是不可达的，主要有两个：

第一个是第-1天是不能持有股票的，所以是不可达的

第二个是购买的状态也不能是负数，也是不可达的

所以对应数组的时候，因为数组不能为负，所以所有状态后面移动一位，至于状态方程和之前的类似，区别在于要加一个for循环，来对所有状态的j进行求最佳值

```java
class Solution {
    public int maxProfit(int[] prices) {
        int[][][]dp=new int[prices.length+1][4][2];
        for(int i=0;i<=prices.length;i++)
            for(int j=0;j<2;j++)
                dp[i][0][j]=Integer.MIN_VALUE;
        for(int i=0;i<=3;i++)
            dp[0][i][1]=Integer.MIN_VALUE;
        for(int i=1;i<=prices.length;i++){
            for(int j=1;j<=3;j++){
                dp[i][j][0]=Math.max(dp[i-1][j][0],dp[i-1][j-1][1]+prices[i-1]);
                dp[i][j][1]=Math.max(dp[i-1][j][1],dp[i-1][j][0]-prices[i-1]);
            }
        }
        return dp[prices.length][3][0];
    }
}
```

###### [买卖股票的最佳时机 IV](https://leetcode.cn/problems/best-time-to-buy-and-sell-stock-iv/)
和上题一样，区别在于将2改成k

```java
class Solution {
    public int maxProfit(int k, int[] prices) {
        int[][][]dp=new int[prices.length+1][k+2][2];
        for(int i=0;i<=prices.length;i++)
            for(int j=0;j<2;j++)
                dp[i][0][j]=Integer.MIN_VALUE;
        for(int i=0;i<=k+1;i++)
            dp[0][i][1]=Integer.MIN_VALUE;
        for(int i=1;i<=prices.length;i++){
            for(int j=1;j<=k+1;j++){
                dp[i][j][0]=Math.max(dp[i-1][j][0],dp[i-1][j-1][1]+prices[i-1]);
                dp[i][j][1]=Math.max(dp[i-1][j][1],dp[i-1][j][0]-prices[i-1]);
            }
        }
        return dp[prices.length][k+1][0];
    }
}
```

将前面所有问题都结合起来，即k次限制，要交易费用，要冷冻cool_n天，可以用以下代码

```java
// 同时考虑交易次数的限制、冷冻期和手续费
int maxProfit_all_in_one(int max_k, int[] prices, int cooldown, int fee) {
    int n = prices.length;
    if (n <= 0) {
        return 0;
    }
    if (max_k > n / 2) {
        // 交易次数 k 没有限制的情况
        return maxProfit_k_inf(prices, cooldown, fee);
    }

    int[][][] dp = new int[n][max_k + 1][2];
    // k = 0 时的 base case
    for (int i = 0; i < n; i++) {
        dp[i][0][1] = Integer.MIN_VALUE;
        dp[i][0][0] = 0;
    }

    for (int i = 0; i < n; i++) 
        for (int k = max_k; k >= 1; k--) {
            if (i - 1 == -1) {
                // base case 1
                dp[i][k][0] = 0;
                dp[i][k][1] = -prices[i] - fee;
                continue;
            }

            // 包含 cooldown 的 base case
            if (i - cooldown - 1 < 0) {
                // base case 2
                dp[i][k][0] = Math.max(dp[i-1][k][0], dp[i-1][k][1] + prices[i]);
                // 别忘了减 fee
                dp[i][k][1] = Math.max(dp[i-1][k][1], -prices[i] - fee);
                continue;
            }
            dp[i][k][0] = Math.max(dp[i-1][k][0], dp[i-1][k][1] + prices[i]);
            // 同时考虑 cooldown 和 fee
            dp[i][k][1] = Math.max(dp[i-1][k][1], dp[i-cooldown-1][k-1][0] - prices[i] - fee);     
        }
    return dp[n - 1][max_k][0];
}

// k 无限制，包含手续费和冷冻期
int maxProfit_k_inf(int[] prices, int cooldown, int fee) {
    int n = prices.length;
    int[][] dp = new int[n][2];
    for (int i = 0; i < n; i++) {
        if (i - 1 == -1) {
            // base case 1
            dp[i][0] = 0;
            dp[i][1] = -prices[i] - fee;
            continue;
        }

        // 包含 cooldown 的 base case
        if (i - cooldown - 1 < 0) {
            // base case 2
            dp[i][0] = Math.max(dp[i-1][0], dp[i-1][1] + prices[i]);
            // 别忘了减 fee
            dp[i][1] = Math.max(dp[i-1][1], -prices[i] - fee);
            continue;
        }
        dp[i][0] = Math.max(dp[i - 1][0], dp[i - 1][1] + prices[i]);
        // 同时考虑 cooldown 和 fee
        dp[i][1] = Math.max(dp[i - 1][1], dp[i - cooldown - 1][0] - prices[i] - fee);
    }
    return dp[n - 1][0];
}

```

###### 解锁
一个长度为n的数组，每个位置的元素为0/1/2，且只能0---〉1---〉2---〉0，解锁的条件是数组没有连续位置相同大小的值，问最少旋转多少次，能解锁

思路：用dp数组，对于每一个dp来说，转向当前需要的转数为0，转向下一个为1，转向另一个为2，对于i来说，它到自己的num不需要转，所以当前的转数为i-1中不相同另外两个的最小值，对于num+1来说，就需要转一次，再加上i-1的另外两个的最小值，最于num+2来说，就需要转两次其他不变

```java
import java.util.Scanner;

public class Main{
    public static void main(String[] args) {
        Scanner in=new Scanner(System.in);
        int n=in.nextInt();
        String s=in.next();
        int [][]dp=new int[n][3];
        dp[0][s.charAt(0)-'0']=0;
        dp[0][(s.charAt(0)-'0'+1)%3]=1;
        dp[0][(s.charAt(0)-'0'+2)%3]=2;
        for(int i=1;i<n;i++){
            int num=s.charAt(i)-'0';
            dp[i][num]=Math.min(dp[i-1][(num+1)%3],dp[i-1][(num+2)%3]);
            dp[i][(num+1)%3]=Math.min(dp[i-1][(num+2)%3],dp[i-1][(num+3)%3])+1;
            dp[i][(num+2)%3]=Math.min(dp[i-1][(num+3)%3],dp[i-1][(num+4)%3])+2;
        }
        System.out.println(Math.min(dp[n-1][0],Math.min(dp[n-1][1],dp[n-1][2])));
    }
}
```

###### <font style="color:rgb(89, 89, 89);">异或和</font>
![](https://cdn.nlark.com/yuque/0/2024/png/42839395/1722580985268-9b0968f9-15ab-46f8-9bb7-d37217633993.png)

```java
import java.util.Scanner;

public class Solution {
    static final int mod = 1000000007;
    static final int N = 310;

    // dp[i][j][k]: 前i个数,异或和为k,最后一个数为j的方案数
    static int[][][] dp = new int[N][N][N << 1];
    // s[j][k]: dp[i-1][0~j][k]的前缀和
    static int[][] s = new int[N][N << 1];

    public static void main(String[] args) {
        Scanner scanner = new Scanner(System.in);
        int n = 200;
        int m = 200;

        // 初始化:只有一种方案,即什么都不选
        dp[0][0][0] = 1;

        for (int i = 1; i <= n; i++) {
            // 计算前缀和
            for (int j = 0; j <= m; j++) {
                for (int k = 0; k <= m + m; k++) {
                    s[j][k] = dp[i - 1][j][k];
                    if (j > 0) {
                        s[j][k] = (s[j][k] + s[j - 1][k]) % mod;
                    }
                }
            }
            // 状态转移
            for (int j = 0; j <= m; j++) {
                for (int k = 0; k <= m + m && (k ^ j) <= 2 * m; k++) {
                    dp[i][j][k] = s[j][k ^ j];
                }
            }
        }

        // 计算最终结果
        int res = 0;
        for (int j = 0; j <= m; ++j)
            res = (res + dp[n][j][m]) % mod;

        System.out.println(res);
    }
}
```

##### 其他
###### 小苯的粉丝关注
小苯是“小红书app”的忠实用户，他有n个账号，每个账号粉丝数为。

这天他又创建了一个新账号，他希望新账号的粉丝数恰好等于x。为此他可以向自己已有账号的粉丝们推荐自己的新账号，这样以来新账号就得到了之前粉丝的关注。

他想知道，他最少需要在几个旧账号发“推荐新账号”的文章，可以使得他的新账号粉丝数恰好为x，除此以外，他可以最多从中选择一个账号多次发“推荐新账号”的文章。

假设所有账号粉丝没有重叠，第i个账号可以增加i/2个粉丝，多推荐的那个可以得到全部粉丝

**输入**包含2行。 第一行两个正整数 n, x(1 ≤ n, k ≤ 100)，分别表示小苯的旧账号个数，和新账号想要的粉丝数。 第二行n个正整数 (1 ≤ ≤ 100)，表示小苯每个旧账号的粉丝数。

**思路**：对于每个账号来说有三个状态，不选，选一次，选多次，这可以用dfs对应不同的状态进行不同的递归，因为选多次只能有一个账号，所以用extra来标识

```java
public class test {

    private static int dfs(int i, int x, boolean extra, int n, int[] a, int[][][] memo) {
        if(x==0)
            return 0;
        if(i>=n)
            return n+1;
        if (memo[i][x][extra ? 1 : 0] != -1) {
            return memo[i][x][extra ? 1 : 0];
        }
        //不选
        int no=dfs(i+1,x,extra,n,a,memo);
        int one=n+1,mutil=n+1;
        //选一次
        if(x>=a[i]/2)
            one=dfs(i+1,x-a[i]/2,extra,n,a,memo)+1;
        //选多次
        if(x>a[i]&&!extra)
            mutil=dfs(i+1,x-a[i],true,n,a,memo)+1;
        //判断哪个次数最少
        memo[i][x][extra?1:0]=Math.min(Math.min(no,one),mutil);
        return memo[i][x][extra?1:0];
    }

    public static void main(String[] args) {
        Scanner scanner = new Scanner(System.in);
        int n = scanner.nextInt();
        int x = scanner.nextInt();
        int[] a = new int[n];
        for (int i = 0; i < n; i++) {
            a[i] = scanner.nextInt();
        }

        int[][][] memo = new int[n + 1][x + 1][2];
        for (int[][] arr2D : memo) {
            for (int[] arr : arr2D) {
                Arrays.fill(arr, -1);
            }
        }

        int ans = dfs(0, x, false, n, a, memo);
        if (ans == n + 1) {
            System.out.println(-1);
        } else {
            System.out.println(ans);
        }
    }
}
```

对于这种题可以抽出个模板

```java
public class DFSDynamicProgramming {

    private static int[][] memo;  // 用于记忆化的数据结构

    // 假设有一问题函数，需要基于不同的参数进行递归
    private static int dfs(int param1, int param2) {
        // 检查是否已经计算过这个状态
        if (memo[param1][param2] != -1) {
            return memo[param1][param2];
        }

        // 基本情况，决定递归的结束
        if (param1 == 0 || param2 == 0) {
            return 0; // 或其他适当的基本情况值
        }

        // 计算当前状态的结果
        int result = 0;

        // 假设递归过程依赖于几种选择
        // Option 1:
        result = Math.max(result, dfs(param1 - 1, param2) + someValue);

        // Option 2:
        result = Math.max(result, dfs(param1, param2 - 1) + someOtherValue);

        // 将结果存储在记忆化结构中
        memo[param1][param2] = result;

        return result;
    }
    public static void main(String[] args) {
        int n = 100;  // 示例参数大小
        int m = 100;  // 示例参数大小
        memo = new int[n + 1][m + 1];
        for (int[] row : memo) {
            Arrays.fill(row, -1);  // 初始化记忆化数组
        }
        // 调用dfs函数，以开始计算
        int finalResult = dfs(n, m);
        System.out.println("Final Result: " + finalResult);
    }
}
```

###### [最大正方形](https://leetcode.cn/problems/maximal-square/)
这个题是求矩阵最大正方形的面积，也就是求最大正方形的边长

所以这类题的状态dp[i][j]表示以(i，j)为右下角的正方形的**最长边长**

![](https://cdn.nlark.com/yuque/0/2024/png/42839395/1720707065549-e1bd69a3-c7b9-4166-bca7-f3d8cef8900f.png)

从上面这个图可以看出来，对于一个以(i,j)为右下角的正方形来说，他的边长由(i-1,j)(i-1,j-1)(i,j-1)这三个位置的最小值决定，这三个值的最小值才能决定(i,j)的边长

所以状态方程是：dp[i][j]=Math.min(dp[i-1][j], Math.min(dp[i-1][j-1], dp[i][j-1])) + 1;

实际上上面这个状态方程只是用于求以(i,j)为右下角的最大正方形，需要用一个maxLength来实时记录整个矩形的最大边长

这里需要考虑边界问题，对于在第0行和第0列的点来说，如果matrix[i][j]='1'，那dp[i][j]只能是1

```java
class Solution {
    public int maximalSquare(char[][] matrix) {
        int maxLength=-1;
        if(matrix==null||matrix.length==0||matrix[0].length==0)
            return 0;
        int m=matrix.length;int n=matrix[0].length;
        int [][]dp=new int[m][n];
        for(int i=0;i<m;i++){
            for(int j=0;j<n;j++){
                if(matrix[i][j]=='1'){
                    if(i==0||j==0){
                        dp[i][j]=1;
                    }else{
                        dp[i][j]=Math.min(dp[i-1][j],
                                        Math.min(dp[i-1][j-1],dp[i][j-1]))+1;
                    }
                    maxLength=Math.max(maxLength,dp[i][j]);
                }
            }
        }
        return maxLength==-1?0:maxLength*maxLength;
    }
}
```

###### [统计全为 1 的正方形子矩阵](https://leetcode.cn/problems/count-square-submatrices-with-all-ones/)
基本和上题一样，区别在于是求正方形个数，这里有一个点在于对于一个以(i,j)为右下角的正方形来说，它能增加的正方形数量取决于它的边长，dp[i][j] = x 也表示以 (i, j) 为右下角的正方形的数目为 x（即边长为 1, 2, ..., x 的正方形各一个）

所以核心思路还是求每个以(i,j)为右下角的正方形的最大边长，找到每个位置的边长，就能找到每个位置多出来的正方形数量

```java
class Solution {
    public int countSquares(int[][] matrix) {
        if(matrix==null||matrix.length==0||matrix[0].length==0)
            return 0;
        int count=0;
        int m=matrix.length;int n=matrix[0].length;
        int [][]dp=new int[m][n];
        for(int i=0;i<m;i++){
            for(int j=0;j<n;j++){
                if(matrix[i][j]==1){
                    if(i==0||j==0){
                        dp[i][j]=1;
                    }else{
                        dp[i][j]=Math.min(dp[i-1][j],
                                        Math.min(dp[i-1][j-1],dp[i][j-1]))+1;
                    }
                }
                count+=dp[i][j];
            }
        }
        return count;
    }
}
```

###### [路径总和III（树形DP)](https://leetcode.cn/problems/path-sum-iii/description/)
这道题可以用bfs+dfs来做，bfs遍历树的每个节点，接着用dfs遍历每个节点看是否sum满足Target

```java
class Solution {
    int count=0;
    public int pathSum(TreeNode root, int targetSum) {
        if(root==null)
            return 0;
        Queue<TreeNode>queue=new LinkedList<>();
        queue.add(root);
        while(!queue.isEmpty()){
            int size=queue.size();
            for(int i=0;i<size;i++){
                TreeNode node=queue.poll();
                NodeSum(node,targetSum,0);
                if(node.left!=null)queue.add(node.left);
                if(node.right!=null)queue.add(node.right);
            }
        }
        return count;
    }

    public void NodeSum(TreeNode node,int Target,long sum){
        if(node==null)
            return;
        sum+=node.val;
        if(sum==Target)
            count++;
        NodeSum(node.left,Target,sum);
        NodeSum(node.right,Target,sum);
    }
}
```

**树形dp**

思路还是和之前那道树形dp那道思路相似，还是要dfs，在dfs的过程中，遍历子节点的时候需要返回当前阶段需要的状态值，当前节点在对此进行dp，再返回

当前节点的Map<Long,Integer>代表子节点的路径和当前节点求和之后的新的sum的条数，key是新的sum，value是这个sum对应的条数

```java
class Solution {
    int count=0;
    public int pathSum(TreeNode root, int targetSum) {
        dfs(root,targetSum);
        return count;
    }
    
    public Map<Long,Integer> dfs(TreeNode node,long targetSum){
        if(node==null)return new HashMap<>();
        Map<Long,Integer>left=dfs(node.left,targetSum);
        Map<Long,Integer>right=dfs(node.right,targetSum);
        Map<Long,Integer>rootValue=new HashMap<>();
        long val=node.val;
        rootValue.put(val,1);
        for(Map.Entry<Long,Integer>entry:left.entrySet()){
            rootValue.put(val+entry.getKey(),rootValue.getOrDefault(val+entry.getKey(),0)+entry.getValue());
        }
        for(Map.Entry<Long,Integer>entry:right.entrySet()){
            rootValue.put(val+entry.getKey(),rootValue.getOrDefault(val+entry.getKey(),0)+entry.getValue());
        }
        if(rootValue.containsKey(targetSum))
            count+=rootValue.get(targetSum);
        return rootValue;
    }
}
```

###### [找出有效子序列的最大长度 I](https://leetcode.cn/problems/find-the-maximum-length-of-valid-subsequence-i/)
给你一个整数数组 `<font style="color:rgba(38, 38, 38, 0.75);background-color:rgb(240, 240, 240);">nums</font>`。

`<font style="color:rgba(38, 38, 38, 0.75);background-color:rgb(240, 240, 240);">nums</font>` 的子序列 `<font style="color:rgba(38, 38, 38, 0.75);background-color:rgb(240, 240, 240);">sub</font>` 的长度为 `<font style="color:rgba(38, 38, 38, 0.75);background-color:rgb(240, 240, 240);">x</font>` ，如果其满足以下条件，则称其为 有效子序列：

+ `<font style="color:rgba(38, 38, 38, 0.75);background-color:rgb(240, 240, 240);">(sub[0] + sub[1]) % 2 == (sub[1] + sub[2]) % 2 == ... == (sub[x - 2] + sub[x - 1]) % 2</font>`

返回 `<font style="color:rgba(38, 38, 38, 0.75);background-color:rgb(240, 240, 240);">nums</font>` 的 最长的有效子序列 的长度。

这道题其实也不算dp

```java
class Solution {
    public int maximumLength(int[] nums) {
        int ji=0,ou=0,hun=1;
        if(nums[0]%2==1)
            ji++;
        else
            ou++;
        for(int i=1;i<nums.length;i++){
            if(nums[i]%2==0)
                ou++;
            else
                ji++;
            if(nums[i-1]%2!=nums[i]%2)
                hun++;
        }
        return Math.max(ji,Math.max(ou,hun));
    }
}
```

###### [找出有效子序列的最大长度 II](https://leetcode.cn/problems/find-the-maximum-length-of-valid-subsequence-ii/)
dp[n][k]表示对于第n个数，相邻为k的最长长度，这个属于暴力dp，也属于是经典找子序列的方式，上题也可以用，不过会超时。

主要思路就是把里面这层for循环都作为子序列候选人，当前nums[i]会和其前面i个数都求和求余，然后更新对对应的长度

```java
class Solution {
    public int maximumLength(int[] nums, int k) {
        int res=-1;
        int n=nums.length;
        int [][]dp=new int[n][k];
        for(int []d:dp)
            Arrays.fill(d,1);
        for(int i=1;i<n;i++){
            for(int j=0;j<i;j++){
                dp[i][(nums[i]+nums[j])%k]=dp[j][(nums[i]+nums[j])%k]+1;
                res=Math.max(res,dp[i][(nums[i]+nums[j])%k]);
            }
        }
        return res;

    }
}
```

对于这道题，还有一个更简单的方式，首先有个前置知识，题目要求(a+b)modk=(b+c)modk，通过对这个式子的变形，可以得到(a−c)modk=0，即a和c同模即可，而c=a+1，所以翻译过来就是有效子序列的奇数项都关于模 k 同余，偶数项都关于模 k 同余

原问题等价于：寻找一个最长的子序列，满足子序列奇数项都相同，偶数项都相同

要保证上面这个条件，说明这个序列取模之后只有两个数（这两个数也可以相等），奇数项是一个数，偶数项是一个数，也就是说只要知道序列的后两位，就可以找到整个子序列。

所以定义dp[x][y]，表示在目前子序列中，x是倒数第二个模，y是最后一个模，以这两个模为子序列的长度最长，那当遍历到一个数，其模为n，此时他为序列的最后一位，那么只需要遍历模为0-k-1的y找其中dp[n][y]最长的一个即可，所以状态方程为：dp[y][n]=dp[n][y]+1，这个表示目前以n为最后一个模的子序列长度等于以y为最后一个模的子序列长度加1

```java
class Solution {
    public int maximumLength(int[] nums, int k) {
        int res=-1;
        int [][]dp=new int[k][k];
        for(int n:nums){
            n=n%k;
            for(int y=0;y<k;y++){
                dp[y][n]=dp[n][y]+1;
                res=Math.max(res,dp[y][n]);
            }
        }
        return res;
    }
}
```



### 回溯算法
回溯算法一般使用dfs遍历，写dfs的时候考虑两个点第一个是返回的条件是什么，第二个是进行下一步递归。

至于dfs里面是否会用到visited数组，取决于考虑遍历的过程中是否有可能会遇到原来的值，并且要考虑递归之后是否要把visited改回来，这个取决于后面是否还需要用到该点，比如说排列组合问题，只是在当前路径用过该点，暂时不需要，在其他路径还是有可能用到该点，所以在遍历后要把visited改回来，但像图的遍历的时候，比如像岛屿问题，遍历过该点，走过这条路，后面就不会再走了，那这个时候就不用把visited改回来

像排列组合问题，要求很多条不同路径，这个时候就要遍历前将该点加入路径，遍历之后把该点去除，这样才能保证路径的正确性，并且由于java引用的特性，每次将path路径加入res中需要创建一个新的list放进去，因为放入的是地址，不然最后res会为空

###### [最长单词](https://leetcode.cn/problems/longest-word-lcci/)
找出其中的最长单词，且该单词由这组单词中的其他单词组合而成。若有多个长度相同的结果，返回其中字典序最小的一项，若没有符合要求的单词则返回空字符串

hashset+回溯

```java
class Solution {
    public String longestWord(String[] words) {
            Set<String> allWords = new HashSet<>();
            for (String word : words) {
                allWords.add(word);
            }
            String ans = "";
            for (String word : words) {
                Set<String> tmpCollects = new HashSet<>(allWords);
                tmpCollects.remove(word);
                if (isCombinated(word, tmpCollects)) {
                    if (word.length() > ans.length()) {
                        ans = word;
                    } else if (word.length() == ans.length()) {
                        ans = ans.compareTo(word) < 0 ? ans : word;
                    }
                }
            }
            return ans;
        }

        private boolean isCombinated(String s, Set<String> words) {
            if (s.length() == 0) return true;
            for (int i = 1; i <= s.length(); i++) {
                if (words.contains(s.substring(0, i)) && isCombinated(s.substring(i), words)) {
                    return true;
                }
            }
            return false;
        }
}
```

###### [N 皇后](https://leetcode.cn/problems/n-queens/)
回溯问题一般就是暴力遍历，然后对不符合的情况进行减枝，类似于这种回溯的话，一般是要求路径的，对于要求路径的问题，方法最开始进行判断是否完成目标，若完成加入res数组，返回，如果没有则对该节点下一步的节点进行遍历，找到一个满足条件的，加入路径，进入递归，然后递归出来的时候表示已经遍历完了，就恢复（因为要求所有的结果，所有的情况），进行下一次判断

```java
class Solution {
    List<List<String>> res = new ArrayList<>();

    public List<List<String>> solveNQueens(int n) {
        List<String> board = new ArrayList<>();
        for (int i = 0; i < n; i++) {
            StringBuilder sb = new StringBuilder();
            for (int j = 0; j < n; j++) {
                sb.append('.');
            }
            board.add(sb.toString());
        }
        backtrack(board, 0);
        return res;
    }
    void backtrack(List<String> board, int row) {
        if (row == board.size()) {
            res.add(new ArrayList<>(board));
            return;
        }
        int n = board.get(row).length();
        for (int col = 0; col < n; col++) {
            if (!isValid(board, row, col)) {
                continue;
            }
            StringBuilder sb = new StringBuilder(board.get(row));
            sb.setCharAt(col, 'Q');
            board.set(row, sb.toString());
            backtrack(board, row + 1);
            sb.setCharAt(col, '.');
            board.set(row, sb.toString());
        }
    }
    boolean isValid(List<String> board, int row, int col) {
        int n = board.size();
        for (int i = 0; i < n; i++) {
            if (board.get(i).charAt(col) == 'Q') {
                return false;
            }
        }
        for (int i = row - 1, j = col + 1;
             i >= 0 && j < n; i--, j++) {
            if (board.get(i).charAt(j) == 'Q') {
                return false;
            }
        }
        for (int i = row - 1, j = col - 1;
             i >= 0 && j >= 0; i--, j--) {
            if (board.get(i).charAt(j) == 'Q') {
                return false;
            }
        }
        return true;
    }
}

```

对string进行处理，可以使用StringBuilder

###### [N 皇后 II](https://leetcode.cn/problems/n-queens-ii/)
```java
class Solution {
    int count=0;
    public int totalNQueens(int n) {
        int [][]nums=new int[n][n];
        dfs(nums,0);
        return count;

    }
    public void dfs(int [][]nums,int index){
        int n=nums.length;
        if(index==n){
            count++;
            return;
        }
        for(int i=0;i<n;i++){
            if(isVaild(nums,index,i)){
                nums[index][i]=1;
                dfs(nums,index+1);
                nums[index][i]=0;
            }
        }
    }
    public boolean isVaild(int [][]nums,int row,int col){
        int n=nums.length;
        for(int i=0;i<n;i++){
            if(nums[i][col]==1)
                return false;
        }
        for(int i=row-1,j=col-1;i>=0&&j>=0;i--,j--){
            if(nums[i][j]==1)
                return false;
        }
        for(int i=row-1,j=col+1;i>=0&&j<n;i--,j++){
            if(nums[i][j]==1)
                return false;
        }
        return true;
    }
}
```

###### [复原 IP 地址](https://leetcode.cn/problems/restore-ip-addresses/)
给定一个只包含数字的字符串 `s` ，用以表示一个 IP 地址，返回所有可能的有效 IP 地址，这些地址可以通过在 `s` 中插入 `'.'` 来形成。你 不能 重新排序或删除 `s` 中的任何数字。

思路：从string中进行遍历，找出符合条件的ip地址，思路就是首先要进行分段，一个完整的ip地址一共分为4段，所以在遍历的过程中要记录目前在第几段，和一个索引在记录目前在s中的位置

遍历的过程：如果四段都写完了，并且s也遍历完了，那么就可以加入res中，使用String.join(".",ip);进行拼接。如果没有那么就进行for循环，找新的一段再进行dfs，这找新一段考虑到最大是255，所以i<index+3，然后考虑前导0的情况，接着提取出一段转成int类型判断是否合规，如果合规就加入ip中，进行dfs，因为ip是全局变量会用于其他分支，所以dfs遍历之后要remove刚加入的字符串

```java
class Solution {
    List<String>res=new LinkedList<>();
    List<String>ip=new LinkedList<>();
    public List<String> restoreIpAddresses(String s) {
        dfs(s,0,0);
        return res;

    }
    public void dfs(String s,int index,int stage){
        if(stage==4){
            if(index==s.length()){
                String ipAdr=String.join(".",ip);
                res.add(ipAdr);
                return;
            }
        }
        for(int i=index;i<s.length()&&i<index+3;i++){
        //这句话很优雅，用于判断前导0的情况，如果index位置的是0，
        //对于用0拼接之后的情况就直接全部越过，
        //避免了单独判断是0然后dfs再写一个else判断其他的情况，这样更简单一点
            if(s.charAt(index)=='0'&&i>index)
                return;
            String ss=s.substring(index,i+1);
            int ipadr=Integer.valueOf(ss);
            if(ipadr>=0&&ipadr<=255){
                ip.add(ss);
                dfs(s,i+1,stage+1);
                ip.remove(ip.size()-1);
            }
        }
    }
}
```

##### 排列组合问题
这类题把所有组合想成是一个树，不同路径往下走

###### [全排列](https://leetcode.cn/problems/permutations/)
used遇到用过的是continue，不能用return，return的话这个分支就直接返回了，continue会找其他节点

```java
class Solution {
    List<List<Integer>>res=new LinkedList<>();
    LinkedList<Integer>path=new LinkedList<>();
    public List<List<Integer>> permute(int[] nums) { 
        boolean []used=new boolean[nums.length];
        dfs(nums,used);
        return res;

    }
    public void dfs(int []nums,boolean []used){
        if(path.size()==nums.length){
            res.add(new LinkedList(path));
            return;
        }
        for(int i=0;i<nums.length;i++){
            if(used[i]){
                continue;
            }
            used[i]=true;
            path.add(nums[i]);
            dfs(nums,used);
            path.removeLast();
            used[i]=false;
        }
    }

}
```

###### 相邻素数数组
![](https://cdn.jsdelivr.net/gh/ddyycc123/imageloader@main/image-20240417001503843.png)

思路：就是对这n个数进行全排列，然后在排的过程中不断进行判断是否满足素数的要求，如果满足就加入hashmap中，hashmap的作用是避免重复，也可以用hashset

```java
import java.util.*;

public class Main {
    public static void main(String[] args) {
        Scanner scanner = new Scanner(System.in);
        int n = scanner.nextInt();
        int[] a = new int[n];
        for (int i = 0; i < n; i++) {
            a[i] = scanner.nextInt();
        }
        //st数组就是visited数组
        boolean[] st = new boolean[n];
        Map<List<Integer>, Integer> mp = new HashMap<>();
        List<Integer> t = new ArrayList<>();
        //b数组用于记录这n个数任意两个之和是否为素数
        boolean[][] b = new boolean[n][n];
        for (int i = 0; i < n; i++) {
            for (int j = 0; j < n; j++) {
                b[i][j] = check(a[i] + a[j]);
            }
        }
        
        dfs(0, -1, a, b, st, t, mp, n);
        System.out.println(mp.size());
    }

    private static boolean check(int x) {
        if (x < 2) return false;
        for (int i = 2; i <= Math.sqrt(x); i++) {
            if (x % i == 0) return false;
        }
        return true;
    }

    private static void dfs(int u, int pre, int[] a, boolean[][] b, boolean[] st, List<Integer> t, Map<List<Integer>, Integer> mp, int n) {
        if (u == n) {
            mp.put(new ArrayList<>(t), mp.getOrDefault(t, 0) + 1);
            return;
        }
        for (int i = 0; i < n; i++) {
            if (!st[i]) {
                if (pre == -1 || b[pre][i]) {
                    st[i] = true;
                    t.add(a[i]);
                    dfs(u + 1, i, a, b, st, t, mp, n);
                    st[i] = false;
                    t.remove(t.size() - 1);
                }。
            }
        }
    }
}

```

###### [子集](https://leetcode.cn/problems/subsets/)
这道题也用的回溯的思想，区别在于找子集的时候，是按照顺序找到，即每次找的都是当前元素的后面，这样能避免重复，**所以对于这种按顺序找的就不需要used数组，而且由于每一个节点都是结果的一部分，所以也不需要if来进行判断是否结束。**

![](https://cdn.jsdelivr.net/gh/ddyycc123/imageloader@main/image-20231221103206476.png)

对于这种按顺序找的实现的话，只需要在dfs中for循环每次从start开始即可，同时dfs形参需要加入这么一个变量

```java
class Solution {
    List<List<Integer>>res=new LinkedList<>();
    List<Integer>path=new LinkedList<>();
    public List<List<Integer>> subsets(int[] nums) {
        dfs(nums,0);
        return res;
    }
    public void dfs(int[]nums,int start){
        res.add(new LinkedList(path));
        for(int i=start;i<nums.length;i++){
            path.add(nums[i]);
            dfs(nums,i+1);
            path.removeLast();
        }
    }
}
```

###### [组合](https://leetcode.cn/problems/combinations/)
组合问题比起子集问题的区别是设定了结果集的长度为k，所以只需要加个if判断，只有path长度为k的时候才加入结果集即可，并且当path长度为k，就可以直接返回，不需要再往下进行递归了

```java
class Solution {
    List<List<Integer>>res=new LinkedList<>();
    List<Integer>path=new LinkedList<>();
    public List<List<Integer>> combine(int n, int k) {
        dfs(n,k,1);
        return res;
    }

    public void dfs(int n,int k,int start){
        if(path.size()==k){
            res.add(new LinkedList(path));
            return;
        }
        for(int i=start;i<=n;i++){
            path.add(i);
            dfs(n,k,i+1);
            path.removeLast();
        }
    }
}
```

###### [子集 II](https://leetcode.cn/problems/subsets-ii/)
给你一个整数数组nums ，其中可能包含**重复元素**，请你返回该数组所有可能的子集（幂集）

这道题的区别是有集合内有重复的值，但结果集不能要重复的值，最开始的想法是在添加res数组的时候进行判断，即判断数组内是否已包含该集合，用下面代码实现

```java
class Solution {
    List<List<Integer>>res=new LinkedList<>();
    List<Integer>path=new LinkedList<>();
    public List<List<Integer>> subsetsWithDup(int[] nums) {
        dfs(nums,0);
        return res;
    }
    public void dfs(int []nums,int start){
        if(!res.contains(path)){
            res.add(new LinkedList(path));
        }
        for(int i=start;i<nums.length;i++){
            path.add(nums[i]);
            dfs(nums,i+1);
            path.removeLast();
        }
    }
}
```

但出错了，出错的原因是，如果[4,1,4]，则结果出现[4,1]和[1,4]，这是算两个结果，但这实际上是一个结果，对于这种情况考虑修正的话，可以对nums原数组进行重新排序，排序之后相同元素都放在一起了，就不会出现这种情况了，代码如下：

```java
class Solution {
    List<List<Integer>>res=new LinkedList<>();
    List<Integer>path=new LinkedList<>();
    public List<List<Integer>> subsetsWithDup(int[] nums) {
        Arrays.sort(nums);
        dfs(nums,0);
        return res;
    }
    public void dfs(int []nums,int start){
        if(!res.contains(path)){
            res.add(new LinkedList(path));
        }
        for(int i=start;i<nums.length;i++){
            path.add(nums[i]);
            dfs(nums,i+1);
            path.removeLast();
        }
    }
}
```

###### [组合总和 II](https://leetcode.cn/problems/combination-sum-ii/)
给定一个候选人编号的集合 `candidates` 和一个目标数 `target` ，找出 `candidates` 中所有可以使数字和为 `target` 的组合

解法1️⃣：这是参照子集的解法来写的，思路是没问题，但是有几个样例超过了时间限制

```java
class Solution {
    List<List<Integer>>res=new LinkedList<>();
    List<Integer>path=new LinkedList<>();
    public List<List<Integer>> combinationSum2(int[] candidates, int target) {
        Arrays.sort(candidates);
        dfs(candidates,target,0,0);
        return res;
    }
    public void dfs(int []candidates,int target,int start,int sum){
        if(sum==target&&!res.contains(path)){
            res.add(new LinkedList(path));
            return;
        }
        if(sum>target||res.contains(path)){
            return;
        }
        for(int i=start;i<candidates.length;i++){
            path.add(candidates[i]);
            sum+=candidates[i];
            dfs(candidates,target,i+1,sum);
            sum-=candidates[i];
            path.removeLast();
        }
    }
}
```

方法2️⃣：进行减枝，考虑的是当重复多个相同的值的时候，只要第一个，后面的都剪掉，剪枝的具体实施如下，除了start和他下一个可以相同以外，其他的都不可以，

```java
if(i>start&&candidates[i-1]==candidates[i])
      continue;
```

完整代码：

```java
class Solution {
    List<List<Integer>>res=new LinkedList<>();
    List<Integer>path=new LinkedList<>();
    public List<List<Integer>> combinationSum2(int[] candidates, int target) {
        Arrays.sort(candidates);
        dfs(candidates,target,0,0);
        return res;
    }
    public void dfs(int []candidates,int target,int start,int sum){
        if(sum==target){
            res.add(new LinkedList(path));
            return;
        }
        if(sum>target){
            return;
        }
        for(int i=start;i<candidates.length;i++){
            if(i>start&&candidates[i-1]==candidates[i])
                continue;
            path.add(candidates[i]);
            sum+=candidates[i];
            dfs(candidates,target,i+1,sum);
            sum-=candidates[i];
            path.removeLast();
        }
    }
}
```

###### [全排列 II](https://leetcode.cn/problems/permutations-ii/)
含重复数字

解法1️⃣：用contains来判断，其他一样

```java
class Solution {
    List<List<Integer>>res=new LinkedList<>();
    List<Integer>path=new LinkedList<>();
    boolean []used;
    public List<List<Integer>> permuteUnique(int[] nums) {
        used=new boolean[nums.length];
        Arrays.sort(nums);
        dfs(nums);
        return res;
    }
    public void dfs(int []nums){
        if(path.size()==nums.length&&!res.contains(path)){
            res.add(new LinkedList(path));
            return;
        }
        for(int i=0;i<nums.length;i++){
            if(used[i])
                continue;
            path.add(nums[i]);
            used[i]=true;
            dfs(nums);
            path.removeLast();
            used[i]=false;
        }
    }
}
```

解法2️⃣：还是使用剪枝的方式

```java
class Solution {

    List<List<Integer>> res = new LinkedList<>();
    LinkedList<Integer> track = new LinkedList<>();
    boolean[] used;
    public List<List<Integer>> permuteUnique(int[] nums) {
        Arrays.sort(nums);
        used = new boolean[nums.length];
        backtrack(nums);
        return res;
    }

    void backtrack(int[] nums) {
        if (track.size() == nums.length) {
            res.add(new LinkedList(track));
            return;
        }

        for (int i = 0; i < nums.length; i++) {
            if (used[i]) {
                continue;
            }
            // 新添加的剪枝逻辑，固定相同的元素在排列中的相对位置
            if (i > 0 && nums[i] == nums[i - 1] && !used[i - 1]) {
                continue;
            }
            track.add(nums[i]);
            used[i] = true;
            backtrack(nums);
            track.removeLast();
            used[i] = false;
        }
    }
}

```

###### [组合总和](https://leetcode.cn/problems/combination-sum/)
给你一个 无重复元素 的整数数组candidates和一个目标整数target  ，找出 candidates 中可以使数字和为目标数 target 的 所有 不同组合 ，并以列表形式返回。你可以按 任意顺序 返回这些组合。

candidates中的 同一个 数字可以 无限制重复被选取 。如果至少一个数字的被选数量不同，则两种组合是不同的。 

无重复可复用问题，最开始的想法是在for循环中都是从0开始，然后进行判断，但这样的话会导致重复的结果，考虑到之前子集的时候其去掉重复的结果是通过从start开始，逐渐往下进行递归i+1，对于这道题，也可以从start开始，往下递归的时候也是i开始，这样就做到了可复用

```java
class Solution {
    List<List<Integer>>res=new LinkedList<>();
    List<Integer>path=new LinkedList<>();
    public List<List<Integer>> combinationSum(int[] candidates, int target) {
        dfs(candidates,target,0,0);
        return res;
    }
    public void dfs(int []candidates,int target,int sum,int start){
        if(sum==target){
            res.add(new LinkedList(path));
            return;
        }
        if(sum>target){
            return;
        }
        for(int i=start;i<candidates.length;i++){
            path.add(candidates[i]);
            sum+=candidates[i];
            dfs(candidates,target,sum,i);
            sum-=candidates[i];
            path.removeLast();
        }
    }
}
```

###### [划分为k个相等的子集](https://leetcode.cn/problems/partition-to-k-equal-sum-subsets/)
给定一个整数数组  `nums` 和一个正整数 `k`，找出是否有可能把这个数组分成 `k` 个非空子集，其总和都相等。

这道题的思路也是回溯遍历，不断的进行组合，组合到一个per，就进行下一个遍历，如果都成功就返回true，如果没有就false，本质上没太大区别

```java
class Solution {
    int target;
    int []nums;
    boolean []used;
    public boolean canPartitionKSubsets(int[] nums, int k) {
        int sum=Arrays.stream(nums).sum();
        if(sum%k!=0)
            return false;
        target=sum/k;
        this.nums=nums;
        used=new boolean[nums.length];
        return dfs(0,k,0);
    }

    public boolean dfs(int start,int k,int bucket){
        if(k==0){
            return true;
        }
        if(bucket==target){
            return dfs(0,k-1,0);
        }
        for(int i=start;i<nums.length;i++){
            if(used[i])
                continue;
            if(bucket+nums[i]>target)
                continue;
            used[i]=true;
            bucket+=nums[i];
            if(dfs(i+1,k,bucket))
                return true;
            used[i]=false;
            bucket-=nums[i];
        }
        return false;
    }
}
```

###### [括号生成](https://leetcode.cn/problems/generate-parentheses/)
数字 `n` 代表生成括号的对数，请你设计一个函数，用于能够生成所有可能的并且有效的括号组合。

给定括号的数量，让生成各种组合，只需要不断遍历即可，值得注意的是这个判断的方式，不需要用栈来判断，定义一个left和right两个指针来表示左括号和右括号的数量即可

```java
class Solution {
    List<String>res=new ArrayList<>();
    StringBuilder sb=new StringBuilder();
    public List<String> generateParenthesis(int n) {
        dfs(n,n);
        return res;
    }
    public void dfs(int left,int right){
        if(left<0||right<0)
            return;
        if(left>right)
            return;
        if(left==0&&right==0){
            res.add(sb.toString());
            return;
        }
        sb.append('(');
        dfs(left-1,right);
        sb.deleteCharAt(sb.length()-1);
        sb.append(')');
        dfs(left,right-1);
        sb.deleteCharAt(sb.length()-1);
    }
}
```

###### [电话号码的字母组合](https://leetcode.cn/problems/letter-combinations-of-a-phone-number/)
思路：这个题的意思是digits个数字，每个数字从中挑一个进行组合，所以直接用dfs for循环进行append就可以了，不过dfs之后需要删除该元素

```java
class Solution {
    public List<String> letterCombinations(String digits) {
        List<String>combinations=new ArrayList<String>();
        if(digits.length()==0)
            return combinations;
        Map<Character,String>phoneMap=new HashMap<Character,String>(){
            {
                put('2',"abc");
                put('3',"def");
                put('4',"ghi");
                put('5',"jkl");
                put('6',"mno");
                put('7',"pqrs");
                put('8',"tuv");
                put('9',"wxyz");
            }
        };
        backtrack(combinations,phoneMap,digits,0,new StringBuffer());
        return combinations;
    }

    public void backtrack(List<String> combinations, Map<Character, String> phoneMap, String digits, int index, StringBuffer combination) {
        if(index==digits.length()){
            combinations.add(combination.toString());
        }else{
            char digit=digits.charAt(index);
            String letters=phoneMap.get(digit);
            int length=letters.length();
            for(int i=0;i<length;i++){
                combination.append(letters.charAt(i));
                backtrack(combinations,phoneMap,digits,index+1,combination);
                combination.deleteCharAt(index);
            }
        }
    }
}

```

###### [单词搜索](https://leetcode.cn/problems/word-search/)
给定一个 `m x n` 二维字符网格 `board` 和一个字符串单词 `word` 。如果 `word` 存在于网格中，返回 `true` ；否则，返回 `false` 。

单词必须按照字母顺序，通过相邻的单元格内的字母构成，其中“相邻”单元格是那些水平相邻或垂直相邻的单元格。同一个单元格内的字母不允许被重复使用。

思路：和地图搜索差不多，不过需要不断判断当前位置和当前位置的字符是否相等，也需要定义一个数组onpath，防止走回头路

```java
class Solution {
    int [][]onpath;
    int [][]directions={
        {1,0},
        {-1,0},
        {0,1},
        {0,-1}
    };
    public boolean exist(char[][] board, String word) {
        onpath=new int[board.length][board[0].length];
        for(int i=0;i<board.length;i++){
            for(int j=0;j<board[0].length;j++){
              
                boolean flag=dfs(board,word,i,j,0);
                if(flag==true)
                    return true;
            }
        }
        return false;
    }
    public boolean dfs(char [][]board,String word,int x,int y,int index){
        if(x<0||x>=board.length||y<0||y>=board[0].length||index>=word.length()||onpath[x][y]==1||board[x][y]!=word.charAt(index))
            return false;
        onpath[x][y]=1;
        if(index==word.length()-1)
            return true;
        for(int []direction:directions){
            int x_new=x+direction[0];
            int y_new=y+direction[1];
            boolean flag=dfs(board,word,x_new,y_new,index+1);
            if(flag==true)
                return true;
        }
        onpath[x][y]=0;
        return false;
    }
}
```

###### [单词搜索 II](https://leetcode.cn/problems/word-search-ii/)
给定一个 `m x n` 二维字符网格 `board` 和一个单词（字符串）列表 `words`， 返回所有二维网格上的单词 

单词必须按照字母顺序，通过 相邻的单元格 内的字母构成，其中“相邻”单元格是那些水平相邻或垂直相邻的单元格。同一个单元格内的字母在一个单词中不允许被重复使用

```java
class Solution {
    Set<String>set;
    List<String>ans;
    char [][]board;
    int [][]dirs={{1,0},{-1,0},{0,1},{0,-1}};
    int n,m;
    boolean[][]visit;
    public List<String> findWords(char[][] _board, String[] words) {
        set=new HashSet<>();
        ans=new LinkedList<>();
        board=_board;
        m=board.length;n=board[0].length;
        visit=new boolean[m][n];
        for(String word:words)
            set.add(word);    
        for(int i=0;i<m;i++){
            for(int j=0;j<n;j++){
                StringBuilder sb=new StringBuilder();
                visit[i][j]=true;
                sb.append(board[i][j]);
                dfs(i,j,sb);
                visit[i][j]=false;
            }
        }
        return ans;
    }
    void dfs(int i, int j, StringBuilder sb) {
        if(sb.length()>10)
            return;
        if(set.contains(sb.toString())){
            ans.add(sb.toString());
            set.remove(sb.toString());
        }
        for(int []dir:dirs){
            int x=i+dir[0],y=j+dir[1];
            if(x<0||x>=m||y<0||y>=n||visit[x][y])
                continue;
            visit[x][y]=true;
            sb.append(board[x][y]);
            dfs(x,y,sb);
            visit[x][y]=false;
            sb.deleteCharAt(sb.length()-1);
        }
    }
}

```

##### 岛屿问题
###### [飞地的数量](https://leetcode.cn/problems/number-of-enclaves/)
给你一个大小为 `m x n` 的二进制矩阵 `grid` ，其中 `0` 表示一个海洋单元格、`1` 表示一个陆地单元格。

一次 移动 是指从一个陆地单元格走到另一个相邻（上、下、左、右）的陆地单元格或跨过 `grid` 的边界。

返回网格中 无法 在任意次数的移动中离开网格边界的陆地单元格的数量。

方法1️⃣：使用经典的dfs遍历，定义一个方位数组，比较耗时，有一个用例没过

```java
class Solution {
    boolean [][]visited;
    int [][]p={
      {0,1},
      {0,-1},
      {1,0},
      {-1,0}
    };
    public int numEnclaves(int[][] grid) {
        int count=0;
        for(int i=1;i<grid.length-1;i++){
            for(int j=1;j<grid[0].length-1;j++){
                if(grid[i][j]==1){
                    visited=new boolean[grid.length][grid[0].length];
                    if(!dfs(grid,i,j)){
                        count++;
                    }
                }
            }
        }
        return count;
    }
    public boolean dfs(int [][]grid,int x,int y){
        if(x==0||x==grid.length-1||y==0||y==grid[0].length-1){
            return true;
        }
        for(int []to:p){
            int x_new=x+to[0];
            int y_new=y+to[1];
            if(grid[x_new][y_new]!=0&&!visited[x_new][y_new]){
                visited[x_new][y_new]=true;
                if(dfs(grid,x_new,y_new)){
                    return true;
                }
            }
        }
        return false;
    }
}
```

方法2️⃣：换一个角度思考，之前是找非边界的陆地是否能到边界，现在换个思路，首先遍历边界，把边界能到的陆地都走一遍，最后直接数哪些陆地没有遍历即可

```java
class Solution {
    boolean [][]visited;
    int [][]p={
      {0,1},
      {0,-1},
      {1,0},
      {-1,0}
    };
    public int numEnclaves(int[][] grid) {
        int count=0;
        visited=new boolean[grid.length][grid[0].length];
        for (int i = 0; i < grid.length; i++) {
            dfs(grid, i, 0);
            dfs(grid, i, grid[0].length-1);
        }
        for (int j = 1; j < grid[0].length-1; j++) {
            dfs(grid, 0, j);
            dfs(grid, grid.length - 1, j);
        }
        for(int i=1;i<grid.length-1;i++){
            for(int j=1;j<grid[0].length-1;j++){
                if(grid[i][j]==1&&!visited[i][j]){
                    count++;
                }
            }
        }
        return count;
    }
    public void dfs(int [][]grid,int x,int y){
        if(x<0||x>grid.length-1||y<0||y>grid[0].length-1||grid[x][y]!=1||visited[x][y]){
            return;
        }
        visited[x][y]=true;
        for(int []to:p){
            dfs(grid,x+to[0],y+to[1]);
        }
        return;
    }
}
```

###### [岛屿数量](https://leetcode.cn/problems/number-of-islands/)
给你一个由 `'1'`（陆地）和 `'0'`（水）组成的的二维网格，请你计算网格中岛屿的数量。

岛屿总是被水包围，并且每座岛屿只能由水平方向和/或竖直方向上相邻的陆地连接形成。

此外，你可以假设该网格的四条边均被水包围

这道题的思路是这样的，首先遍历遇到一个岛屿就count++，然后将这个岛屿及其所有邻接的地方全部淹没，后续就不会再遇到了，也可以用find-union来做，不过更麻烦一些

```java
class Solution {
    int [][]position={
      {1,0},
      {-1,0},
      {0,1},
      {0,-1}
    };
    public int numIslands(char[][] grid) {
        int m=grid.length;
        int n=grid[0].length;
        int count=0;
        for(int i=0;i<m;i++){
            for(int j=0;j<n;j++){
                if(grid[i][j]=='1'){
                    count++;
                    dfs(grid,i,j);
                }
            }
        }
        return count;
    }

    public void dfs(char [][]grid,int x,int y){
        if(x<0||x>grid.length-1||y<0||y>grid[0].length-1||grid[x][y]=='0'){
            return;
        }
        grid[x][y]='0';
        for(int[] to:position){
            dfs(grid,x+to[0],y+to[1]);
        }
    }
}
```

###### [统计封闭岛屿的数目](https://leetcode.cn/problems/number-of-closed-islands/)
二维矩阵 `grid` 由 `0` （土地）和 `1` （水）组成。岛是由最大的4个方向连通的 `0` 组成的群，封闭岛是一个 `完全` 由1包围（左、上、右、下）的岛。

请返回 封闭岛屿 的数目。

我的思路是：要求封闭岛屿的数量，参照上一题也是进行邻接岛屿的淹没，不过在递归的时候要进行判断，如果递归的点能到达边界，且值为0的话，那说明能和外界连通，不是封闭岛屿，将flag设置为false，如果遇到的是1，那就正常返回，通过这样的方式进行判断

```java
class Solution {
    int [][]positions={
      {0,1},
      {0,-1},
      {1,0},
      {-1,0}
    };
    boolean flag;
    public int closedIsland(int[][] grid) {
        int count=0;
        for(int i=1;i<grid.length;i++){
            for(int j=1;j<grid[0].length;j++){
                if(grid[i][j]==0){
                    flag=true;
                    dfs(grid,i,j);
                    if(flag==true)
                        count++;
                }
            }
        }
        return count;

    }
    public void dfs(int [][]grid,int x,int y){
        if(x==0||x==grid.length-1||y==0||y==grid[0].length-1){
            if(grid[x][y]==0){
                flag=false;
                return;
            }
        }
        if(grid[x][y]==1)
            return;
        grid[x][y]=1;
        for(int []position:positions){
            dfs(grid,x+position[0],y+position[1]);
        }
    }
}
```

也可以参照飞地的思路，就遍历四条边的岛屿，把边界连通的岛屿淹没掉，这样剩下的就都是封闭岛了

###### [岛屿的最大面积](https://leetcode.cn/problems/max-area-of-island/)
岛屿 是由一些相邻的 `1` (代表土地) 构成的组合，这里的「相邻」要求两个 `1` 必须在 水平或者竖直的四个方向上 相邻。你可以假设 `grid` 的四个边缘都被 `0`（代表水）包围着。

岛屿的面积是岛上值为 `1` 的单元格的数目。

计算并返回 `grid` 中最大的岛屿面积。如果没有岛屿，则返回面积为 `0` 。

这道题的思路也是淹没，区别是淹没的过程中要统计该岛的数量，最后输出最大值

```java
class Solution {
    int [][]positions={
      {0,1},
      {0,-1},
      {1,0},
      {-1,0}
    };
    int count;
    public int maxAreaOfIsland(int[][] grid) {
        int res=Integer.MIN_VALUE;
        for(int i=0;i<grid.length;i++){
            for(int j=0;j<grid[0].length;j++){
                if(grid[i][j]==1){
                    count=0;
                    dfs(grid,i,j);
                    if(count>res){
                        res=count;
                    }
                }
            }
        }
        return res==Integer.MIN_VALUE?0:res;
    }
    public void dfs(int [][]grid,int x,int y){
        if(x<0||x>grid.length-1||y<0||y>grid[0].length-1||grid[x][y]==0){
            return;
        }
        count++;
        grid[x][y]=0;
        for(int []position:positions){
            dfs(grid,x+position[0],y+position[1]);
        }
    }
}
```

###### [统计子岛屿](https://leetcode.cn/problems/count-sub-islands/)
给你两个 `m x n` 的二进制矩阵 `grid1` 和 `grid2` ，它们只包含 `0` （表示水域）和 `1` （表示陆地）。一个 岛屿 是由 四个方向 （水平或者竖直）上相邻的 `1` 组成的区域。任何矩阵以外的区域都视为水域。

如果 `grid2` 的一个岛屿，被 `grid1` 的一个岛屿 完全 包含，也就是说 `grid2` 中该岛屿的每一个格子都被 `grid1` 中同一个岛屿完全包含，那么我们称 `grid2` 中的这个岛屿为 子岛屿 。

请你返回 `grid2` 中 子岛屿 的 数目 。

一样的思路，区别在于，由于是求2中是1的岛屿数量，所以对2进行遍历，在遍历2的过程中判断2中的岛屿是否1都有，如果有的话就算子岛屿，否则就不算

```java
class Solution {
    int [][]positions={
      {1,0},
      {-1,0},
      {0,1},
      {0,-1}
    };
    boolean flag;
    public int countSubIslands(int[][] grid1, int[][] grid2) {
        int res=0;
        for(int i=0;i<grid2.length;i++){
            for(int j=0;j<grid2[0].length;j++){
                if(grid2[i][j]==1){
                    flag=true;
                    dfs(grid1,grid2,i,j);
                    if(flag==true){
                        res++;
                    }
                }
            }
        }
        return res;
    }
    public void dfs(int [][]grid1,int [][]grid2,int x,int y){
        if(x<0||x>grid2.length-1||y<0||y>grid2[0].length-1||grid2[x][y]==0){
            return;
        }
        if(grid1[x][y]!=grid2[x][y]){
            flag=false;
            return;
        }
        grid2[x][y]=0;
        for(int []position:positions){
            dfs(grid1,grid2,x+position[0],y+position[1]);
        }
    }
}
```

###### [不同岛屿数量](https://leetcode.cn/problems/number-of-distinct-islands/description/)
求地图中形状不同的岛屿数量

所以这道题的思路是将岛屿的形状记录在HashSet中，最后统计HashSet的面积即可，但如何记录形状呢，考虑序列化该岛屿，在遍历岛屿的时候，如果记录遍历整个岛屿的步骤，即每一步是向哪个方向的，递归结束后，是怎么撤回的，记录这个步骤就可以判断两个岛屿是否相同

不过这里面存在几个值得注意的地方

1. 这里面两个岛屿是否相同是两个岛屿不能有旋转之类的操作，即从同一个视图看是相同的才行
2. 遍历两个相同岛屿的时候一定是从相同的位置进行遍历的
3. 一定要加撤销的步骤，比方说「下，右，撤销右，撤销下」和「下，撤销下，右，撤销右」显然是两个不同的遍历顺序，但如果不记录撤销操作，那么它俩都是「下，右」，成了相同的遍历顺序
4. 关于记录遍历步骤，可以用StringBuilder来记录，然后再将StringBuilder toString，将入HashSet即可

```java
class Solution {
    public int numDistinctIslands(int[][] grid) {
        int m = grid.length, n = grid[0].length;
        // 记录所有岛屿的序列化结果
        HashSet<String> islands = new HashSet<>();
        for (int i = 0; i < m; i++) {
            for (int j = 0; j < n; j++) {
                if (grid[i][j] == 1) {
                    // 淹掉这个岛屿，同时存储岛屿的序列化结果
                    StringBuilder sb = new StringBuilder();
                    // 初始的方向可以随便写，不影响正确性
                    dfs(grid, i, j, sb, 666);
                    islands.add(sb.toString());
                }
            }
        }
        // 不相同的岛屿数量
        return islands.size();
    }
    void dfs(int[][] grid, int i, int j, StringBuilder sb, int dir) {
        int m = grid.length, n = grid[0].length;
        if (i < 0 || j < 0 || i >= m || j >= n 
            || grid[i][j] == 0) {
            return;
        }
        // 前序遍历位置：进入 (i, j)
        grid[i][j] = 0;
        sb.append(dir).append(',');

        dfs(grid, i - 1, j, sb, 1); // 上
        dfs(grid, i + 1, j, sb, 2); // 下
        dfs(grid, i, j - 1, sb, 3); // 左
        dfs(grid, i, j + 1, sb, 4); // 右

        // 后序遍历位置：离开 (i, j)
        sb.append(-dir).append(',');
    }
}
```

###### [被围绕的区域](https://leetcode.cn/problems/surrounded-regions/)
给你一个 `m x n` 的矩阵 `board` ，由若干字符 `'X'` 和 `'O'` 组成，捕获 所有 被围绕的区域：

+ 连接：一个单元格与水平或垂直方向上相邻的单元格连接。
+ 区域：连接所有 `'O'` 的单元格来形成一个区域。
+ 围绕：如果您可以用 `'X'` 单元格 连接这个区域，并且区域中没有任何单元格位于 `board` 边缘，则该区域被 `'X'` 单元格围绕。

通过将输入矩阵 `board` 中的所有 `'O'` 替换为 `'X'` 来 捕获被围绕的区域。

这题属于岛屿问题，思路是先用 for 循环遍历棋盘的四边，用 DFS 算法把那些与边界相连的 `O` 换成一个特殊字符，比如 A；然后再遍历整个棋盘，把剩下的 `O` 换成 `X`，把 `#` 恢复成 `O`。这样就能完成题目的要求，时间复杂度 O(MN)。

这道题有个点在于dfs的判断条件中不能用board[ x ] [ y ]=='X'来进行判断，因为考虑到一种情况，当前面进行遍历的时候，把周围的O变成了A，如果用这个判断条件的话就会对变成A的点继续遍历，这样的话就是遍历就不会结束，会一直遍历下去，所以要用board[ x ] [ y ]!='O'，或者加个visited数组

```java
class Solution {
    public void solve(char[][] board) {
        int m=board.length;
        int n=board[0].length;
        for(int i=0;i<m;i++){
            dfs(board,i,0);
            dfs(board,i,n-1);
        }
        for(int i=0;i<n;i++){
            dfs(board,0,i);
            dfs(board,m-1,i);
        }
        for(int i=0;i<m;i++){
            for(int j=0;j<n;j++){
                if(board[i][j]=='O'){
                    board[i][j]='X';
                }else if(board[i][j]=='A')
                    board[i][j]='O';
            }
        }
    }
    public void dfs(char [][]board,int x,int y){
        if(x<0||x>=board.length||y<0||y>=board[0].length||board[x][y]!='O')
            return;
        board[x][y]='A';
        dfs(board,x+1,y);
        dfs(board,x-1,y);
        dfs(board,x,y+1);
        dfs(board,x,y-1);
    }

}
```

###### [甲板上的战舰](https://leetcode.cn/problems/battleships-in-a-board/)
给你一个大小为 `m x n` 的矩阵 `board` 表示棋盘，其中，每个单元格可以是一艘战舰 `'X'` 或者是一个空位 `'.'` ，返回在棋盘 `board` 上放置的 舰队 的数量。

舰队 只能水平或者垂直放置在 `board` 上。换句话说，舰队只能按 `1 x k`（`1` 行，`k` 列）或 `k x 1`（`k` 行，`1` 列）的形状放置，其中 `k` 可以是任意大小。两个舰队之间至少有一个水平或垂直的空格分隔 （即没有相邻的舰队）。

```java
class Solution {
    int [][]directions={
        {1,0},
        {-1,0},
        {0,1},
        {0,-1}
    };
    public int countBattleships(char[][] board) {
        int count=0;
        for(int i=0;i<board.length;i++){
            for(int j=0;j<board[0].length;j++){
                if(board[i][j]=='X'){
                    count++;
                    dfs(board,i,j);
                }
            }
        }
        return count;
    }

    public void dfs(char [][]board,int i,int j){
        if(i<0||i>=board.length||j<0||j>=board[0].length||board[i][j]=='.')
            return;
        board[i][j]='.';
        for(int []direction:directions){
            int new_i =i+direction[0];
            int new_j=j+direction[1];
            dfs(board,new_i,new_j);
        }
        
    }
}
```

##### 数独问题
###### [有效的数独](https://leetcode.cn/problems/valid-sudoku/)
请你判断一个 `9 x 9` 的数独是否有效。只需要 根据以下规则 ，验证已经填入的数字是否有效即可。

1. 数字 `1-9` 在每一行只能出现一次。
2. 数字 `1-9` 在每一列只能出现一次。
3. 数字 `1-9` 在每一个以粗实线分隔的 `3x3` 宫内只能出现一次

```java
class Solution {
    public boolean isValidSudoku(char[][] board) {
        for(int i=0;i<9;i++){
            for(int j=0;j<9;j++){
                if(board[i][j]!='.'){
                    char ch=board[i][j];
                    for(int k=0;k<9;k++){
                        if((board[i][k]==ch&&k!=j)||(board[k][j]==ch&&k!=i)){
                            return false;
                        }
                        if (board[(i/3)*3 + k/3][(j/3)*3 + k%3] == ch&&!(((i/3)*3 + k/3)==i&&((j/3)*3 + k%3)==j))
                            return false;
                    }
                }
            }
        }
        return true;
    }
}
```

###### [解数独](https://leetcode.cn/problems/sudoku-solver/)
通过填充空格来解决数独问题。

数独的解法需 遵循如下规则：

1. 数字 `1-9` 在每一行只能出现一次。
2. 数字 `1-9` 在每一列只能出现一次。
3. 数字 `1-9` 在每一个以粗实线分隔的 `3x3` 宫内只能出现一次。（请参考示例图）

数独部分空格内已填入了数字，空白格用 `'.'` 表示。

这道题也是用回溯的思想，遍历每一个空的格子，找到其的值，不过和之前回溯不同的地方在于，这道题只需要求一个结果，所以可以把dfs设置为boolean，当遍历出一个正确结果，就直接返回，是下面代码体现

```java
            if(dfs(board,x,y+1)){
                return true;
            }
```

然后就是和岛屿问题不一样的是，这道题是一个组合问题，是在不断试错的过程，所以在遍历之前board(x)(y)=c，遍历之后，没有返回true，说明该方案失败，就要board(x)(y)=‘.’，岛屿问题就是一个单纯的遍历融合问题，没有这种操作，但对于迷宫问题，也有visited数组来看是否走过该路，但是遍历之后没有改回来，是因为对于迷宫问题，走不通该路径，下次再遇到还是走不通，对于这种组合问题，这次没走通，但换个组合就有可能可以走通，所以要改回来

这道题没有visited数组，是因为遍历的顺序就是从左到右，从上到下，所以也不会遇到之前走过的格子，只是如果遍历失败，就要把当前的路径抹去，换一个组合重新走

```java
class Solution {
    public void solveSudoku(char[][] board) {
        dfs(board,0,0);
    }
    public boolean dfs(char [][]board,int x,int y){
        //判断列越界
        if(y==board[0].length){
            return dfs(board,x+1,0);
        }
        //判断行越界
        if(x==board.length){
            return true;
        }
        //判断非空
        if(board[x][y]!='.'){
            return dfs(board,x,y+1);
        }
        for(char c='1';c<='9';c++){
            //判断合法
            if(!isValid(board,x,y,c))
                continue;
            board[x][y]=c;
            if(dfs(board,x,y+1)){
                return true;
            }
            board[x][y]='.';
        }
        return false;
    }
    public boolean isValid(char [][]board,int x,int y,char ch){
        for(int i=0;i<9;i++){
            //判断竖和横
            if(board[x][i]==ch||board[i][y]==ch){
                return false;
            }
            //这个判断3*3小方格内
            if (board[(x/3)*3 + i/3][(y/3)*3 + i%3] == ch)
                return false;
        }
        return true;
    }
}
```

### BFS
###### [二叉树的最小深度](https://leetcode.cn/problems/minimum-depth-of-binary-tree/)
```java
class Solution {
    public int minDepth(TreeNode root) {
        if(root==null)
            return 0;
        Queue<TreeNode>queue=new LinkedList<>();
        queue.add(root);
        int depth=1;
        while(!queue.isEmpty()){
            int size=queue.size();
            for(int i=0;i<size;i++){
                TreeNode node=queue.poll();
                if(node.left==null&&node.right==null){
                    return depth;
                }
                if(node.left!=null)
                    queue.add(node.left);
                if(node.right!=null)
                    queue.add(node.right);
            }
            depth++;
        }
        return depth;
    }
}
```

###### [打开转盘锁](https://leetcode.cn/problems/open-the-lock/)
你有一个带有四个圆形拨轮的转盘锁。每个拨轮都有10个数字： `'0', '1', '2', '3', '4', '5', '6', '7', '8', '9'` 。每个拨轮可以自由旋转：例如把 `'9'` 变为 `'0'`，`'0'` 变为 `'9'` 。每次旋转都只能旋转一个拨轮的一位数字。

锁的初始数字为 `'0000'` ，一个代表四个拨轮的数字的字符串。

列表 `deadends` 包含了一组死亡数字，一旦拨轮的数字和列表里的任何一个元素相同，这个锁将会被永久锁定，无法再被旋转。

字符串 `target` 代表可以解锁的数字，你需要给出解锁需要的最小旋转次数，如果无论如何不能解锁，返回 `-1` 。

层次遍历

这道题有意思的点在于对string的处理，一般如果是添加的话就可以转为StringBuilder，如果是就对当前的进行修改的话就可以改为toCharArray，换成char类型数组来进行处理，最后再new string

```java
class Solution {
    public int openLock(String[] deadends, String target) {
        Queue<String> queue=new LinkedList<>();
        Set<String>dead=new HashSet<>();
        int step=0;
        for(String d:deadends)
            dead.add(d);
        Set<String>visited=new HashSet<>();
        queue.add("0000");
        visited.add("0000");
        while(!queue.isEmpty()){
            int size=queue.size();
            for(int i=0;i<size;i++){
                String q=queue.poll();
                if(dead.contains(q))
                    continue;
                if(q.equals(target))
                    return step;
                for(int j=0;j<4;j++){
                    String up=plus(q,j);
                    if(!visited.contains(up)){
                        visited.add(up);
                        queue.offer(up);
                    }
                    String down=minus(q,j);
                    if(!visited.contains(down)){
                        visited.add(down);
                        queue.offer(down);
                    }
                }
            }
            step++;
        }
        return -1;
    }
    public String plus(String s,int position){
        char []array=s.toCharArray();
        if(array[position]=='9')
            array[position]='0';
        else
            array[position]+=1;
        return new String(array);
    }
    public String minus(String s,int position){
        char []array=s.toCharArray();
        if(array[position]=='0')
            array[position]='9';
        else
            array[position]-=1;
        return new String(array);
    }
}
```

###### [滑动谜题](https://leetcode.cn/problems/sliding-puzzle/)
在一个 `2 x 3` 的板上（`board`）有 5 块砖瓦，用数字 `1~5` 来表示, 以及一块空缺用 `0` 来表示。一次 移动 定义为选择 `0` 与一个相邻的数字（上下左右）进行交换.

最终当板 `board` 的结果是 `[[1,2,3],[4,5,0]]` 谜板被解开。

给出一个谜板的初始状态 `board` ，返回最少可以通过多少次移动解开谜板，如果不能解开谜板，则返回 `-1` 

这道题就是数组华容道，考虑用bfs来做，所以首先要转化为bfs问题，就是如何遍历的问题，考虑到每次都是0所在的位置和周围的位置进行交换，这就可以转化为bfs了

![](https://cdn.jsdelivr.net/gh/ddyycc123/imageloader@main/image-20231227111320830.png)

接下来就只剩一个问题，如何找到这个0所在位置是和其他哪些位置的进行交换，这里因为题目说了是2*3，所以可以考虑使用邻接表的方式，不然的话就只能使用方位数组然后进行判断越界了。

```java
class Solution {
    public int slidingPuzzle(int[][] board) {
        int m=board.length;int n=board[0].length;
        StringBuilder sb=new StringBuilder();
        for(int i=0;i<m;i++)
            for(int j=0;j<n;j++)
                sb.append(board[i][j]);
        int[][] arcmap = new int[][]{
                {1, 3},
                {0, 4, 2},
                {1, 5},
                {0, 4},
                {3, 1, 5},
                {4, 2}
        };
        Queue<String>queue=new LinkedList<>();
        Set<String>visited=new HashSet<>();
        String target="123450";
        int step=0;
        queue.add(sb.toString());
        visited.add(sb.toString());
        while(!queue.isEmpty()){
            int size=queue.size();
            for(int i=0;i<size;i++){
                String s=queue.poll();
                if(s.equals(target)){
                    return step;
                }
                int index=0;
                  //找到这个string中的0所在的位置
                for(int k=0;k<6;k++){
                    if(s.charAt(k)=='0'){
                        index=k;
                        break;
                    }
                }
                //对0所在的位置的邻接表进行遍历
                for(int p:arcmap[index]){
                    String ss=swap(s,p,index);
                    if(!visited.contains(ss)){
                        visited.add(ss);
                        queue.add(ss);
                    }
                }
            }
            step++;
        }
        return -1;
    }
    public String swap(String s,int p,int index){
        char []array=s.toCharArray();
        char temp=array[index];
        array[index]=array[p];
        array[p]=temp;
        return new String(array);
    }
}
```

###### [最小基因变化](https://leetcode.cn/problems/minimum-genetic-mutation/)
基因序列可以表示为一条由 8 个字符组成的字符串，其中每个字符都是 `'A'`、`'C'`、`'G'` 和 `'T'` 之一。

假设我们需要调查从基因序列 `start` 变为 `end` 所发生的基因变化。一次基因变化就意味着这个基因序列中的一个字符发生了变化。

思路：从start进行变化，变化的形式在基因库里面，并且要求最短变化次数，所以想到了使用bfs进行判断，思路是每一次从队列中拿出一个元素时，首先和end进行判断是否相等，如果不相等，就进行判断基因库里面有哪些是该元素能经过一次变化就得到的，如果存在这种就加入队列，一直bfs下去

用hashset将已经在队列中存在过的基因标记，遇到存在过的就不用再进行比较或者加入队列了

```java
class Solution {
    public int minMutation(String startGene, String endGene, String[] bank) {
        Queue<String>queue=new LinkedList<>();
        HashSet<String>set=new HashSet<>();
        int num=0;
        queue.add(startGene);
        while(!queue.isEmpty()){
            int size=queue.size();
            for(int i=0;i<size;i++){
                String s=queue.poll();
                set.add(s);
                if(s.equals(endGene)){
                    return num;
                }
                for(String ss:bank){
                    if(set.contains(ss))
                        continue;
                    int dif=compare(s,ss);
                    if(dif==1)
                        queue.add(ss);
                }
            }
            num++;
        }
        return -1;
    }

    public int compare(String s1,String s2){
        int num=0;
        for(int i=0;i<s1.length();i++){
            if(s1.charAt(i)!=s2.charAt(i))
                num++;
        }
        return num;
    }

}
```

###### [蛇梯棋](https://leetcode.cn/problems/snakes-and-ladders/)
给你一个大小为 `n x n` 的整数矩阵 `board` ，方格按从 `1` 到 `n2` 编号，编号遵循 [转行交替方式](https://baike.baidu.com/item/%E7%89%9B%E8%80%95%E5%BC%8F%E8%BD%AC%E8%A1%8C%E4%B9%A6%E5%86%99%E6%B3%95/17195786) ，从左下角开始 （即，从 `board[n - 1][0]` 开始）的每一行改变方向。

你一开始位于棋盘上的方格  `1`。每一回合，玩家需要从当前方格 `curr` 开始出发，按下述要求前进：

+ 选定目标方格 `next` ，目标方格的编号在范围 `[curr + 1, min(curr + 6, n2)]` 。
    - 该选择模拟了掷 六面体骰子 的情景，无论棋盘大小如何，玩家最多只能有 6 个目的地。
+ 传送玩家：如果目标方格 `next` 处存在蛇或梯子，那么玩家会传送到蛇或梯子的目的地。否则，玩家传送到目标方格 `next` 。 
+ 当玩家到达编号 `n2` 的方格时，游戏结束。

如果 `board[r][c] != -1` ，位于 `r` 行 `c` 列的棋盘格中可能存在 “蛇” 或 “梯子”。那个蛇或梯子的目的地将会是 `board[r][c]`。编号为 `1` 和 `n2` 的方格不是任何蛇或梯子的起点。

注意，玩家在每回合的前进过程中最多只能爬过蛇或梯子一次：就算目的地是另一条蛇或梯子的起点，玩家也 不能 继续移动。

```java
class Solution {
    public int snakesAndLadders(int[][] board) {
        int n = board.length;
        boolean[] vis = new boolean[n * n + 1];
        Queue<int[]> queue = new LinkedList<int[]>();
        queue.offer(new int[]{1, 0});
        while (!queue.isEmpty()) {
            int[] p = queue.poll();
            for (int i = 1; i <= 6; ++i) {
                int nxt = p[0] + i;
                if (nxt > n * n) { // 超出边界
                    break;
                }
                int[] rc = id2rc(nxt, n); // 得到下一步的行列
                if (board[rc[0]][rc[1]] > 0) { // 存在蛇或梯子
                    nxt = board[rc[0]][rc[1]];
                }
                if (nxt == n * n) { // 到达终点
                    return p[1] + 1;
                }
                if (!vis[nxt]) {
                    vis[nxt] = true;
                    queue.offer(new int[]{nxt, p[1] + 1}); // 扩展新状态
                }
            }
        }
        return -1;
    }

    public int[] id2rc(int id, int n) {
        int r = (id - 1) / n, c = (id - 1) % n;
        if (r % 2 == 1) {
            c = n - 1 - c;
        }
        return new int[]{n - 1 - r, c};
    }
}

```

###### [单词接龙](https://leetcode.cn/problems/word-ladder/)
字典 `wordList` 中从单词 `beginWord` 到 `endWord` 的 转换序列 是一个按下述规格形成的序列 `beginWord -> s1 -> s2 -> ... -> sk`：

+ 每一对相邻的单词只差一个字母。
+  对于 `1 <= i <= k` 时，每个 `si` 都在 `wordList` 中。注意， `beginWord` 不需要在 `wordList` 中。
+ `sk == endWord`

给你两个单词 `beginWord` 和 `endWord` 和一个字典 `wordList` ，返回 从 `beginWord` 到 `endWord` 的 最短转换序列 中的 单词数目 。如果不存在这样的转换序列，返回 `0` 。

思路：和最小基因变化一样的思路，只不过要把hashset.add(word);放到里面用于减少时间

```java
class Solution {
    public int ladderLength(String beginWord, String endWord, List<String> wordList) {
        Queue<String>queue=new LinkedList<>();
        HashSet<String>hashset=new HashSet<>();
        queue.add(beginWord);
        hashset.add(beginWord);
        int num=1;
        while(!queue.isEmpty()){
            int size=queue.size();
            for(int i=0;i<size;i++){
                String s=queue.poll();
                if(s.equals(endWord))
                    return num;
                for(String word:wordList){
                    if(hashset.contains(word))
                        continue;
                    int n=compare(s,word);
                    if(n==1){
                        hashset.add(word);
                        queue.add(word);
                    }     
                }
            }
            num++;
        }
        return 0;
    }

    public int compare(String s1,String s2){
        int num=0;
        for(int i=0;i<s1.length();i++){
            if(s1.charAt(i)!=s2.charAt(i))
                num++;
        }
        return num;
    }
}
```

###### [二叉树的层平均值](https://leetcode.cn/problems/average-of-levels-in-binary-tree/)
层次遍历 给定一个非空二叉树的根节点 `root` , 以数组的形式返回每一层节点的平均值。与实际答案相差 `10-5` 以内的答案可以被接受

```java
/**
 * Definition for a binary tree node.
 * public class TreeNode {
 *     int val;
 *     TreeNode left;
 *     TreeNode right;
 *     TreeNode() {}
 *     TreeNode(int val) { this.val = val; }
 *     TreeNode(int val, TreeNode left, TreeNode right) {
 *         this.val = val;
 *         this.left = left;
 *         this.right = right;
 *     }
 * }
 */
class Solution {
    public List<Double> averageOfLevels(TreeNode root) {
        List<Double>list=new LinkedList<>();
        Queue<TreeNode>queue=new LinkedList<>();
        queue.add(root);
        while(!queue.isEmpty()){
            int size=queue.size();
            Double average=0.0;
            for(int i=0;i<size;i++){
                TreeNode node=queue.poll();
                if(node.left!=null)queue.add(node.left);
                if(node.right!=null)queue.add(node.right);
                average+=node.val;
            }
            list.add(average/size);
        }
        return list;
    }
}
```

###### [二叉树的层序遍历](https://leetcode.cn/problems/binary-tree-level-order-traversal/)
层次遍历 给你二叉树的根节点 `root` ，返回其节点值的 层序遍历 

```java
/**
 * Definition for a binary tree node.
 * public class TreeNode {
 *     int val;
 *     TreeNode left;
 *     TreeNode right;
 *     TreeNode() {}
 *     TreeNode(int val) { this.val = val; }
 *     TreeNode(int val, TreeNode left, TreeNode right) {
 *         this.val = val;
 *         this.left = left;
 *         this.right = right;
 *     }
 * }
 */
class Solution {
    public List<List<Integer>> levelOrder(TreeNode root) {
        List<List<Integer>>res=new LinkedList<>();
        if(root==null)
            return res;
        Queue<TreeNode>queue=new LinkedList<>();
        queue.add(root);
        while(!queue.isEmpty()){
            int size=queue.size();
            List<Integer>list=new LinkedList<>();
            for(int i=0;i<size;i++){
                TreeNode node=queue.poll();
                list.add(node.val);
                if(node.left!=null)queue.add(node.left);
                if(node.right!=null)queue.add(node.right);
            }
            res.add(list);
        }
        return res;
    }
}
```

###### [二叉树的锯齿形层序遍历](https://leetcode.cn/problems/binary-tree-zigzag-level-order-traversal/)
给你二叉树的根节点 `root` ，返回其节点值的 锯齿形层序遍历 。（即先从左往右，再从右往左进行下一层遍历，以此类推，层与层之间交替进行）

```java
/**
 * Definition for a binary tree node.
 * public class TreeNode {
 *     int val;
 *     TreeNode left;
 *     TreeNode right;
 *     TreeNode() {}
 *     TreeNode(int val) { this.val = val; }
 *     TreeNode(int val, TreeNode left, TreeNode right) {
 *         this.val = val;
 *         this.left = left;
 *         this.right = right;
 *     }
 * }
 */
class Solution {
    public List<List<Integer>> zigzagLevelOrder(TreeNode root) {
        List<List<Integer>>res=new LinkedList<>();
        if(root==null)
            return res;
        Queue<TreeNode>queue=new LinkedList<>();
        queue.add(root);
        boolean flag=true;
        while(!queue.isEmpty()){
            int size=queue.size();
            List<Integer>list=new LinkedList<>();
            for(int i=0;i<size;i++){
                TreeNode node=queue.poll();
                if(node.left!=null)queue.add(node.left);
                if(node.right!=null)queue.add(node.right);
                if(flag)
                    list.add(node.val);
                else
                    list.add(0,node.val);
            }
            res.add(list);
            flag=!flag;
        }
        return res;
    }
}
```

### 位运算
**判断两个数是否异号**

利用的是**补码编码**的符号位。整数编码最高位是符号位，负数的符号位是 1，非负数的符号位是 0，再借助异或的特性，可以判断出两个数字是否异号。

```java
int x = -1, y = 2;
boolean f = ((x ^ y) < 0); // true

int x = 3, y = 2;
boolean f = ((x ^ y) < 0); // false
```

`n & (n-1)`** 的运用**

`n & (n-1)`** 这个操作在算法中比较常见，作用是消除数字 **`n`** 的二进制表示中的最后一个 1**。

比如说7是111，7-1就是110，这个时候用&，就是110，就会消去7的最后一个1，每次消去一个，这个也可以用来求一个数其中1的数量

###### [位1的个数](https://leetcode.cn/problems/number-of-1-bits/)
编写一个函数，获取一个正整数的二进制形式并返回其二进制表达式中设置位的个数（也被称为[汉明重量](https://baike.baidu.com/item/%E6%B1%89%E6%98%8E%E9%87%8D%E9%87%8F)）

```java
public class Solution {
    // you need to treat n as an unsigned value
    public int hammingWeight(int n) {
        int count=0;
        while(n!=0){
            n=n&(n-1);
            count++;
        }
        return count;
    }
}
```

###### [2 的幂](https://leetcode.cn/problems/power-of-two/)
给你一个整数 `n`，请你判断该整数是否是 2 的幂次方。如果是，返回 `true` ；否则，返回 `false` 。

如果存在一个整数 `x` 使得 `n == 2x` ，则认为 `n` 是 2 的幂次方。

```java
class Solution {
    public boolean isPowerOfTwo(int n) {
        if(n<0)
            return false;
        int count=0;
        while(n!=0){
            n=n&(n-1);
            count++;
        }
        return count==1?true:false;
    }
}
```

`a ^ a = 0`** 的运用**

异或即相异为1，相同为0

一个数和它本身做异或运算结果为 0，即 `a ^ a = 0`；一个数和 0 做异或运算的结果为它本身，即 `a ^ 0 = a`

###### [只出现一次的数字](https://leetcode.cn/problems/single-number/)
给你一个 非空 整数数组 `nums` ，除了某个元素只出现一次以外，其余每个元素均出现两次。找出那个只出现了一次的元素。

我们只要把所有数字进行异或，成对儿的数字就会变成 0，落单的数字和 0 做异或还是它本身，所以最后异或的结果就是只出现一次的元素，比如说[2,1,2]，就是10->11->01，就是落单的1

```java
class Solution {
    public int singleNumber(int[] nums) {
        int res=0;
        for(int n:nums){
            res^=n;
        }
        return res;
    }
}
```

###### [只出现一次的数字 II](https://leetcode.cn/problems/single-number-ii/)
给你一个整数数组 `nums` ，除某个元素仅出现 一次 外，其余每个元素都恰出现 三次 。请你找出并返回那个只出现了一次的元素。

数组中只有一个数存在一次，其他数都存在3次，如果把所有的数转成2进制，也就是说在每一位如果这一位出现的1为0或者3的倍数，就说明在这一位上目标值是0，如果不是，就说明目标值是1，那将32位的目标值每一位都找出来，那目标值就找出来了

```java
class Solution {
    public int singleNumber(int[] nums) {
        int res=0;
        for(int i=0;i<32;i++){
            int count=0;
            for(int n:nums){
                //分别找每个数在第i位是否是1
                count+=(n>>i)&1;
            }
            if(count%3!=0){
                //对res的第i位赋值
                res|=1<<i;
            }
        }
        return res;
    }
}
```

###### [丢失的数字](https://leetcode.cn/problems/missing-number/)
给定一个包含 `[0, n]` 中 `n` 个数的数组 `nums` ，找出 `[0, n]` 这个范围内没有出现在数组中的那个数。

```java
class Solution {
    public int missingNumber(int[] nums) {
        int n=nums.length;
        int res=0;
        for(int num:nums)
            res^=num;
        for(int i=0;i<=n;i++)
            res^=i;
        return res;
    }
}
```

### 分治法
###### [为运算表达式设计优先级](https://leetcode.cn/problems/different-ways-to-add-parentheses/)
给你一个由数字和运算符组成的字符串 `expression` ，按不同优先级组合数字和运算符，计算并返回所有可能组合的结果。你可以 按任意顺序 返回答案。

```java
class Solution {
    public List<Integer> diffWaysToCompute(String expression) {
        List<Integer>res=new ArrayList<>();
        for(int i=0;i<expression.length();i++){
            char ch=expression.charAt(i);
            if(ch=='+'||ch=='-'||ch=='*'){
                List<Integer>left=diffWaysToCompute(expression.substring(0,i));
                List<Integer>right=diffWaysToCompute(expression.substring(i+1));
                for(int l:left){
                    for(int r:right){
                        if(ch=='+'){
                            res.add(l+r);
                        }else if(ch=='*')
                            res.add(l*r);
                        else if(ch=='-')
                            res.add(l-r);
                    }
                }
            }
        }
        if(res.isEmpty())
            res.add(Integer.parseInt(expression));
        return res;
    }
}
```

###### [建立四叉树](https://leetcode.cn/problems/construct-quad-tree/)
```java
/*
// Definition for a QuadTree node.
class Node {
    public boolean val;
    public boolean isLeaf;
    public Node topLeft;
    public Node topRight;
    public Node bottomLeft;
    public Node bottomRight;

    
    public Node() {
        this.val = false;
        this.isLeaf = false;
        this.topLeft = null;
        this.topRight = null;
        this.bottomLeft = null;
        this.bottomRight = null;
    }
    
    public Node(boolean val, boolean isLeaf) {
        this.val = val;
        this.isLeaf = isLeaf;
        this.topLeft = null;
        this.topRight = null;
        this.bottomLeft = null;
        this.bottomRight = null;
    }
    
    public Node(boolean val, boolean isLeaf, Node topLeft, Node topRight, Node bottomLeft, Node bottomRight) {
        this.val = val;
        this.isLeaf = isLeaf;
        this.topLeft = topLeft;
        this.topRight = topRight;
        this.bottomLeft = bottomLeft;
        this.bottomRight = bottomRight;
    }
}
*/

class Solution {
    public Node construct(int[][] grid) {
        return dfs(grid, 0, 0, grid.length, grid.length);
    }

    public Node dfs(int[][] grid, int r0, int c0, int r1, int c1) {
        boolean same = true;
        for (int i = r0; i < r1; ++i) {
            for (int j = c0; j < c1; ++j) {
                if (grid[i][j] != grid[r0][c0]) {
                    same = false;
                    break;
                }
            }
            if (!same) {
                break;
            }
        }

        if (same) {
            return new Node(grid[r0][c0] == 1, true);
        }

        Node ret = new Node(
            true,
            false,
            dfs(grid, r0, c0, (r0 + r1) / 2, (c0 + c1) / 2),
            dfs(grid, r0, (c0 + c1) / 2, (r0 + r1) / 2, c1),
            dfs(grid, (r0 + r1) / 2, c0, r1, (c0 + c1) / 2),
            dfs(grid, (r0 + r1) / 2, (c0 + c1) / 2, r1, c1)
        );
        return ret;
    }
}

```

### 括号问题
###### [有效的括号](https://leetcode.cn/problems/valid-parentheses/)
给定一个只包括 `'('`，`')'`，`'{'`，`'}'`，`'['`，`']'` 的字符串 `s` ，判断字符串是否有效。

有效字符串需满足：

1. 左括号必须用相同类型的右括号闭合。
2. 左括号必须以正确的顺序闭合。
3. 每个右括号都有一个对应的相同类型的左括号。

括号匹配

最开始想的是直接用l1，l2，l3来代表三种符号的左符号，然后直接加加减减即可，但是报错"([)]"这样的情况，所以就用栈➕hashmap的方式

```java
class Solution {
    public boolean isValid(String s) {
        Stack<Character>stack=new Stack<>();
        Map<Character,Character>map=new HashMap<>();
        map.put(')','(');
        map.put('}','{');
        map.put(']','[');
        for(int i=0;i<s.length();i++){
            if(s.charAt(i)=='('||s.charAt(i)=='{'||s.charAt(i)=='['){
                stack.push(s.charAt(i));
            }
            else{
                if(!stack.isEmpty()&&map.get(s.charAt(i))==stack.peek()){
                    stack.pop();
                }else{
                    return false;
                }
            }
        }
        if(!stack.isEmpty())
            return false;
        else
            return true;
    }
}
```

###### [使括号有效的最少添加](https://leetcode.cn/problems/minimum-add-to-make-parentheses-valid/)
只有满足下面几点之一，括号字符串才是有效的：

+ 它是一个空字符串，或者
+ 它可以被写成 `AB` （`A` 与 `B` 连接）, 其中 `A` 和 `B` 都是有效字符串，或者
+ 它可以被写作 `(A)`，其中 `A` 是有效字符串。

给定一个括号字符串 `s` ，在每一次操作中，你都可以在字符串的任何位置插入一个括号

这道题就可以用left来计数，因为只有一个种类，不过要考虑一个情况就是left小于0时，属于是消不掉的右括号，就需要额外记录

```java
class Solution {
    public int minAddToMakeValid(String s) {
        int left=0;int right=0;
        for(int i=0;i<s.length();i++){
            if(s.charAt(i)=='(')
                left++;
            else{
                left--;
                if(left<0){
                    left=0;
                    right++;
                }
            }
        }
        return left+right;
    }
}
```

###### [平衡括号字符串的最少插入次数](https://leetcode.cn/problems/minimum-insertions-to-balance-a-parentheses-string/)
给你一个括号字符串 `s` ，它只包含字符 `'('` 和 `')'` 。一个括号字符串被称为平衡的当它满足：

+ 任何左括号 `'('` 必须对应两个连续的右括号 `'))'` 。
+ 左括号 `'('` 必须在对应的连续两个右括号 `'))'` 之前。

比方说 `"())"`， `"())(())))"` 和 `"(())())))"` 都是平衡的， `")()"`， `"()))"` 和 `"(()))"` 都是不平衡的。

你可以在任意位置插入字符 '(' 和 ')' 使字符串平衡

和上一题一样的思路，区别在于要多做一次右括号连续性判断，当遇到右括号的时候，要判断它的下一个是不是也右括号，如果是那么就满足条件，直接跳过，如果不是就需要right++，属于是要添加的

```java
class Solution {
    public int minInsertions(String s) {
        int left=0;
        int right=0;
        for(int i=0;i<s.length();i++){
            if(s.charAt(i)=='('){
                left++;
            }else{
                left--;
                if(left<0){
                    left=0;
                    right++;
                }
                if(i<s.length()-1&&s.charAt(i+1)==')')
                    i++;
                else
                    right++;
            }
        }
        return left*2+right;
    }
}
```

### 哈希表
###### [赎金信](https://leetcode.cn/problems/ransom-note/)
给你两个字符串：`ransomNote` 和 `magazine` ，判断 `ransomNote` 能不能由 `magazine` 里面的字符构成。

如果可以，返回 `true` ；否则返回 `false` 。

`magazine` 中的每个字符只能在 `ransomNote` 中使用一次

```java
class Solution {
    public boolean canConstruct(String ransomNote, String magazine) {
        HashMap<Character,Integer>mag=new HashMap<>();
        for(char c:magazine.toCharArray()){
            mag.put(c,mag.getOrDefault(c,0)+1);
        }
        for(char c:ransomNote.toCharArray()){
            if(!mag.containsKey(c)||mag.get(c)==0)
                return false;
            mag.put(c,mag.getOrDefault(c,0)-1);
        }
        return true;
    }
}
```

###### [同构字符串](https://leetcode.cn/problems/isomorphic-strings/)
给定两个字符串 `s` 和 `t` ，判断它们是否是同构的。

如果 `s` 中的字符可以按某种映射关系替换得到 `t` ，那么这两个字符串是同构的。

每个出现的字符都应当映射到另一个字符，同时不改变字符的顺序。不同字符不能映射到同一个字符上，相同字符只能映射到同一个字符上，字符可以映射到自己本身。

思路：用两个数组记录两个字符串的结构，遍历字符串如果之前没遇到过就记录该索引，如果map中含有，就记录第一次出现的索引，最后比较两个数组是否相同即可

```java
class Solution {
    public boolean isIsomorphic(String s, String t) {
        HashMap<Character,Integer>smap=new HashMap<>();
        HashMap<Character,Integer>tmap=new HashMap<>();
        int []sarray=new int[s.length()];
        int []tarray=new int[t.length()];
        int i=0;
        for(char ss:s.toCharArray()){
            if(!smap.containsKey(ss)){
                sarray[i]=i;
                smap.put(ss,i);
            }else{
                sarray[i]=smap.get(ss);
            }
            i++;
        }
        i=0;
        for(char tt:t.toCharArray()){
            if(!tmap.containsKey(tt)){
                tarray[i]=i;
                tmap.put(tt,i);
            }else{
                tarray[i]=tmap.get(tt);
            }
            i++;
        }
        for(i=0;i<tarray.length;i++){
            if(sarray[i]!=tarray[i])
                return false;
        }
        return true;
    }
}
```

###### [单词规律](https://leetcode.cn/problems/word-pattern/)
给定一种规律 `pattern` 和一个字符串 `s` ，判断 `s` 是否遵循相同的规律。

这里的 遵循 指完全匹配，例如， `pattern` 里的每个字母和字符串 `s` 中的每个非空单词之间存在着双向连接的对应规律。

思路：和上题一样

```java
class Solution {
    public boolean wordPattern(String pattern, String s) {
        String []sss=s.split(" ");
        if(pattern.length()!=sss.length)
            return false;
        HashMap<Character,Integer>pmap=new HashMap<>();
        HashMap<String,Integer>smap=new HashMap<>();
        int []parray=new int[pattern.length()];
        int []sarray=new int[sss.length];
        int i=0;
        for(char pp:pattern.toCharArray()){
            if(!pmap.containsKey(pp)){
                parray[i]=i;
                pmap.put(pp,i);
            }else{
                parray[i]=pmap.get(pp);
            }
            i++;
        }
        i=0;
        for(String ss:sss){
            if(!smap.containsKey(ss)){
                sarray[i]=i;
                smap.put(ss,i);
            }else{
                sarray[i]=smap.get(ss);
            }
            i++;
        }
        for(i=0;i<parray.length;i++){
            if(parray[i]!=sarray[i])
                return false;
        }
        return true;
    }
}
```

###### [有效的字母异位词](https://leetcode.cn/problems/valid-anagram/)
给定两个字符串 `s` 和 `t` ，编写一个函数来判断 `t` 是否是 `s` 的字母异位词。

字母异位词 是通过重新排列不同单词或短语的字母而形成的单词或短语，通常只使用所有原始字母一次。

```java
class Solution {
    public boolean isAnagram(String s, String t) {
        if(s.length()!=t.length())
            return false;
        HashMap<Character,Integer>smap=new HashMap<>();
        for(char ss:s.toCharArray()){
            smap.put(ss,smap.getOrDefault(ss,0)+1);
        }
        for(char tt:t.toCharArray()){
            if(!smap.containsKey(tt)){
                return false;
            }
            smap.put(tt,smap.getOrDefault(tt,0)-1);
        }
        for(Map.Entry<Character,Integer>entry:smap.entrySet()){
            if(entry.getValue()!=0)
                return false;
        }
        return true;
    }
}
```

###### [字母异位词分组](https://leetcode.cn/problems/group-anagrams/)
给你一个字符串数组，请你将 字母异位词 组合在一起。可以按任意顺序返回结果列表。

字母异位词 是由重新排列源单词的所有字母得到的一个新单词

思路：一个string的数组，要对里面同分异位的字符串进行分类，考虑的是挨个对每个字符串进行排序，然后添加在map中，这里特别的是map的value设置的是list格式，存储每个类别的list，最后再统一返回

这里获取list的方法是getOrDefault，这样的好处是不用判断是否为空，如果为空，就返回一个new ArrayList()

```java
class Solution {
    public List<List<String>> groupAnagrams(String[] strs) {
        HashMap<String,List<String>>map=new HashMap<>();
        for(String s:strs){
            char[]c=s.toCharArray();
            Arrays.sort(c);
            String key=new String(c);
            List<String>list=map.getOrDefault(key,new ArrayList<String>());
            list.add(s);
            map.put(key,list);
        }
        return new ArrayList<List<String>>(map.values());
    }
}
```

###### [最长连续序列](https://leetcode.cn/problems/longest-consecutive-sequence/)
给定一个未排序的整数数组 `nums` ，找出数字连续的最长序列（不要求序列元素在原数组中连续）的长度。

思路：最开始想的是直接对原数组进行排序，然后进行连续性判断，但如果存在相同的数字时，这个连续性就会从中间断开，所以考虑用有序的hashset来去重，对hashset进行遍历来进行连续性判断

```java
class Solution {
    public int longestConsecutive(int[] nums) {
        Set<Integer> num_set = new HashSet<Integer>();
        for (int num : nums) {
            num_set.add(num);
        }

        int longestStreak = 0;

        for (int num : num_set) {
            if (!num_set.contains(num - 1)) {
                int currentNum = num;
                int currentStreak = 1;

                while (num_set.contains(currentNum + 1)) {
                    currentNum += 1;
                    currentStreak += 1;
                }

                longestStreak = Math.max(longestStreak, currentStreak);
            }
        }

        return longestStreak;
    }
}
```

###### [两数之和](https://leetcode.cn/problems/two-sum/)
双数之和

方法1️⃣：使用hashmap，key用于数的大小，value用于记录索引

每到一个数用containsKey来进行判断

```java
class Solution {
    public int[] twoSum(int[] nums, int target) {
        HashMap<Integer,Integer>map=new HashMap<>();
        for(int i=0;i<nums.length;i++){
            if(map.containsKey(target-nums[i])){
                return new int[]{map.get(target-nums[i]),i};
            }
            map.put(nums[i],i);
        }
        return new int[]{0,0};
    }
}
```

方法2️⃣：使用双重for循环

```java
class Solution {
    public int[] twoSum(int[] nums, int target) {
        for(int i=0;i<nums.length;i++){
            for(int j=i+1;j<nums.length;j++){
                if(nums[i]+nums[j]==target)
                    return new int[]{i,j};
            }
        }
        return new int[]{0,0};
    }
}
```

###### [存在重复元素 II](https://leetcode.cn/problems/contains-duplicate-ii/)
给你一个整数数组 `nums` 和一个整数 `k` ，判断数组中是否存在两个 不同的索引 `i` 和 `j` ，满足 `nums[i] == nums[j]` 且 `abs(i - j) <= k` 。如果存在，返回 `true` ；否则，返回 `false` 。

思路：用hashmap来的key来表示值，value表示索引

```java
class Solution {
    public boolean containsNearbyDuplicate(int[] nums, int k) {
        HashMap<Integer,Integer>map=new HashMap<>();
        for(int i=0;i<nums.length;i++){
            if(map.containsKey(nums[i])){
                if(Math.abs(i-map.get(nums[i]))<=k)
                    return true;
            }
            map.put(nums[i],i);
        }
        return false;
    }
}
```

### 区间
###### [汇总区间](https://leetcode.cn/problems/summary-ranges/)
给定一个  无重复元素 的 有序 整数数组 `nums` 。

返回 恰好覆盖数组中所有数字 的 最小有序 区间范围列表 。也就是说，`nums` 的每个元素都恰好被某个区间范围所覆盖，并且不存在属于某个范围但不属于 `nums` 的数字 `x` 。

列表中的每个区间范围 `[a,b]` 应该按如下格式输出：

+ `"a->b"` ，如果 `a != b`
+ `"a"` ，如果 `a == b`

思路：利用两个循环，挨个找区间，然后放入list中

```java
class Solution {
    public List<String> summaryRanges(int[] nums) {
        List<String>res=new ArrayList<String>();
        int i=0;
        int n=nums.length;
        while(i<n){
            int low=i;
            i++;
            while(i<n&&nums[i]==nums[i-1]+1)
                i++;
            int high=i-1;
            StringBuffer temp=new StringBuffer(Integer.toString(nums[low]));
            if(low<high){
                temp.append("->");
                temp.append(Integer.toString(nums[high]));
            }
            res.add(temp.toString());
        }
        return res;
    }
}
```

###### [合并区间](https://leetcode.cn/problems/merge-intervals/)
以数组 `intervals` 表示若干个区间的集合，其中单个区间为 `intervals[i] = [starti, endi]` 。请你合并所有重叠的区间，并返回 一个不重叠的区间数组，该数组需恰好覆盖输入中的所有区间 。

方法1️⃣：自己的思路，就是首先对数组按照第一个元素进行排序，接着利用两个while循环挨个进行合并，合并到不符合范围的时候就进行下一轮

```java
import java.util.LinkedList;
import java.util.List;

class Solution {
    public int[][] merge(int[][] intervals) {
        List<List<Integer>>list=new LinkedList<>();
        int i=0;
        Arrays.sort(intervals, new Comparator<int[]>() {
            public int compare(int[] interval1, int[] interval2) {
                return interval1[0] - interval2[0];
            }
        });
        while(i<intervals.length){
            int low=intervals[i][0];
            int high=intervals[i][1];
            while(i<intervals.length-1&&high>=intervals[i+1][0]){
                i++;
                low=low<intervals[i][0]?low:intervals[i][0];
                high=high>intervals[i][1]?high:intervals[i][1];
            }
            List<Integer>list1=new LinkedList<>();
            list1.add(low);list1.add(high);
            list.add(list1);
            i++;
        }
        int [][]res=new int[list.size()][2];
        for(int j=0;j<list.size();j++){
            res[j][0]=list.get(j).get(0);
            res[j][1]=list.get(j).get(1);
        }
        return res;
    }
}
```

注意：定义数组的排序参照下面的代码

```java
        Arrays.sort(intervals, new Comparator<int[]>() {
            public int compare(int[] interval1, int[] interval2) {
                return interval1[0] - interval2[0];
            }
        });
```

方法2️⃣：官方的思路大致相同，区别在于，首先会先加入第一个区间，接着就判断后面区间，如果是重叠范围内，就比较大小，如果更大就修改list中上一个区间的范围

```java
class Solution {
    public int[][] merge(int[][] intervals) {
        if (intervals.length == 0) {
            return new int[0][2];
        }
        Arrays.sort(intervals, new Comparator<int[]>() {
            public int compare(int[] interval1, int[] interval2) {
                return interval1[0] - interval2[0];
            }
        });
        List<int[]> merged = new ArrayList<int[]>();
        for (int i = 0; i < intervals.length; ++i) {
            int L = intervals[i][0], R = intervals[i][1];
            if (merged.size() == 0 || merged.get(merged.size() - 1)[1] < L) {
                merged.add(new int[]{L, R});
            } else {
                merged.get(merged.size() - 1)[1] = Math.max(merged.get(merged.size() - 1)[1], R);
            }
        }
        return merged.toArray(new int[merged.size()][]);
    }
}
```

###### [插入区间](https://leetcode.cn/problems/insert-interval/)
给你一个 无重叠的 ，按照区间起始端点排序的区间列表 `intervals`，其中 `intervals[i] = [starti, endi]` 表示第 `i` 个区间的开始和结束，并且 `intervals` 按照 `starti` 升序排列。同样给定一个区间 `newInterval = [start, end]` 表示另一个区间的开始和结束。

在 `intervals` 中插入区间 `newInterval`，使得 `intervals` 依然按照 `starti` 升序排列，且区间之间不重叠（如果有必要的话，可以合并区间）。

返回插入之后的 `intervals`。

注意 你不需要原地修改 `intervals`。你可以创建一个新数组然后返回它。

思路：由于是插入了一个区间，所以考虑的是只有nums(0)>high或者nums(1)<low的时候是不重叠的，所以还是按照上一题的思路，for挨个遍历intervals区间，考虑到插入的区间只能和原区间融合成一个，所以定义一个flag用于判断是否已经插入了题目要求插入的区间，也是在nums(0)>high才会进行判断，当nums(1)<low直接插入原区间，重叠的时候，只需要不断找寻融合的区间的边界即可

```java
class Solution {
    public int[][] insert(int[][] intervals, int[] newInterval) {
        List<int[]>list=new LinkedList<int[]>();
        int low=newInterval[0];
        int high=newInterval[1];
        Boolean flag=true;
        for(int []interval:intervals){
            if(interval[0]>high){
                if(flag){
                    list.add(new int[]{low,high});
                    flag=false;
                }
                list.add(interval);
            }else if(interval[1]<low){
                list.add(interval);
            }else{
                low=low<interval[0]?low:interval[0];
                high=high>interval[1]?high:interval[1];
            }
        }
        if(flag)
            list.add(new int[]{low,high});
        int [][]res=new int[list.size()][2];
        for(int i=0;i<list.size();i++){
            res[i]=list.get(i);
        }
        return res;
    }
}
```

###### [用最少数量的箭引爆气球](https://leetcode.cn/problems/minimum-number-of-arrows-to-burst-balloons/)
有一些球形气球贴在一堵用 XY 平面表示的墙面上。墙面上的气球记录在整数数组 `points` ，其中`points[i] = [xstart, xend]` 表示水平直径在 `xstart` 和 `xend`之间的气球。你不知道气球的确切 y 坐标。

一支弓箭可以沿着 x 轴从不同点 完全垂直 地射出。在坐标 `x` 处射出一支箭，若有一个气球的直径的开始和结束坐标为 `x``start`，`x``end`， 且满足  `xstart ≤ x ≤ x``end`，则该气球会被 引爆 。可以射出的弓箭的数量 没有限制 。 弓箭一旦被射出之后，可以无限地前进。

给你一个数组 `points` ，返回引爆所有气球所必须射出的 最小 弓箭数

思路：和合并的区别在于合并起并集，这个求交集

这道题要注意的点越界的问题[[-2147483646,-2147483645],[2147483646,2147483647]]

```java
class Solution {
    public int findMinArrowShots(int[][] points) {
        List<int[]>list=new LinkedList<>();
        Arrays.sort(points,new Comparator<int[]>(){
            public int compare(int []point1,int []point2){
                return Integer.compare(point1[0], point2[0]);
            }
        });
        for(int []point:points){
            int left=point[0];
            int right=point[1];
            if(list.size()==0||left>list.get(list.size()-1)[1]){
                list.add(new int[]{left,right});
            }else{
                left=left>list.get(list.size()-1)[0]?left:list.get(list.size()-1)[0];
                right=right<list.get(list.size()-1)[1]?right:list.get(list.size()-1)[1];
                list.get(list.size()-1)[0]=left;
                list.get(list.size()-1)[1]=right;
            }
        }
        return list.size();
    }
}
```

###### [划分字母区间](https://leetcode.cn/problems/partition-labels/)
给你一个字符串 `s` 。我们要把这个字符串划分为尽可能多的片段，同一字母最多出现在一个片段中。

注意，划分结果需要满足：将所有划分结果按顺序连接，得到的字符串仍然是 `s` 。

返回一个表示每个字符串片段的长度的列表。

思路：用hashmap记录每个字符的最后一个位置，接着从第一个字符开始，设置index为这一段最远要到的位置，在遍历这一段的时候也在不断更新index，直到index和i相等时，说明这一段的字符到目前都结束了，就可以作为单独的一段了

```java
class Solution {
    public List<Integer> partitionLabels(String s) {
        List<Integer>list=new LinkedList<>();
        HashMap<Character,Integer>map=new HashMap<>();
        int length=0;
        for(int i=0;i<s.length();i++){
            map.put(s.charAt(i),i);
        }
        int index=0;
        for(int i=0;i<s.length();i++){
            length++;
            char ch=s.charAt(i);
            if(map.get(ch)>index){
                index=map.get(ch);
            }
            if(i==index){
                list.add(length);
                length=0;
            }
        }
        return list;
    }
}
```

###### [无需开会的工作日](https://leetcode.cn/problems/count-days-without-meetings/)
给你一个正整数 `days`，表示员工可工作的总天数（从第 1 天开始）。另给你一个二维数组 `meetings`，长度为 `n`，其中 `meetings[i] = [start_i, end_i]` 表示第 `i` 次会议的开始和结束天数（包含首尾）。

返回员工可工作且没有安排会议的天数。

思路：先对数组按第一位进行排序，接着挨个遍历，找中间的空值

```java
class Solution {
    public int countDays(int days, int[][] meetings) {
        PriorityQueue<int[]>queue=new PriorityQueue<>(
            (a,b)->{
                return a[0]-b[0];
            }
        );
        for(int []m:meetings){
            queue.add(new int[]{m[0],m[1]});
        }
        int end=0;
        int res=0;
        while(!queue.isEmpty()){
            int []n=queue.poll();
            if(n[0]>end){
                res+=n[0]-end-1;
            }
            if(n[1]>end){
                end=n[1];
            }
        }
        if(end<days){
            res+=days-end;
        }
        return res;
    }
}
```

### 其他
###### [简化路径](https://leetcode.cn/problems/simplify-path/)
给你一个字符串 `path` ，表示指向某一文件或目录的 Unix 风格 绝对路径 （以 `'/'` 开头），请你将其转化为更加简洁的规范路径。

在 Unix 风格的文件系统中，一个点（`.`）表示当前目录本身；此外，两个点 （`..`） 表示将目录切换到上一级（指向父目录）；两者都可以是复杂相对路径的组成部分。任意多个连续的斜杠（即，`'//'`）都被视为单个斜杠 `'/'` 。 对于此问题，任何其他格式的点（例如，`'...'`）均被视为文件/目录名称。

请注意，返回的 规范路径 必须遵循下述格式：

+ 始终以斜杠 `'/'` 开头。
+ 两个目录名之间必须只有一个斜杠 `'/'` 。
+ 最后一个目录名（如果存在）不能 以 `'/'` 结尾。
+ 此外，路径仅包含从根目录到目标文件或目录的路径上的目录（即，不含 `'.'` 或 `'..'`）。

返回简化后得到的 规范路径 。

思路：将path按照/进行区分，然后根据题意进行处理，index表示结果路径的位置

```java
class Solution {
    public String simplifyPath(String path) {
        String []dirs=path.split("/");
        String []res=new String[dirs.length];
        int index=0;
        for(String dir:dirs){
            if(dir.isEmpty()||dir.equals("."))
                continue;
            if(dir.equals("..")){
                index=Math.max(0,index-1);
            }else{
                res[index++]=dir;
            }
        }
        if(index==0)
            return "/";
        StringBuilder sb=new StringBuilder();
        for(int i=0;i<index;i++){
            sb.append("/"+res[i]);
        }
        return sb.toString();

    }
}
```

###### [计数质数](https://leetcode.cn/problems/count-primes/)
给定整数 `n` ，返回 所有小于非负整数 `n` 的质数的数量 

素数

方法1️⃣：最直接的判别法     超时

```java
class Solution {
    public int countPrimes(int n) {
        int count=0;
        for(int i=2;i<n;i++){
            if(isPrimes(i))
                count++;
        }
        return count;
    }
    public boolean isPrimes(int n){
        for(int i=2;i<=Math.sqrt(n);i++){
            if(n%i==0)
                return false;
        }
        return true;
    }
}
```

方法2️⃣：素数筛选法

这道题的思路是当某一个数是素数时，它的倍数都不是素数，所以就将它的倍数都设置为非素数即可

```java
class Solution {
    public int countPrimes(int n) {
        boolean []isPrimes=new boolean[n];
        Arrays.fill(isPrimes,true);
        for(int i=2;i*i < n;i++){
            if(isPrimes[i]){
                  //设置该素数的2倍，3倍，4倍...都为非素数
                for(int j=i*2;j<n;j+=i)
                    isPrimes[j]=false;
            }
        }
        int count=0;
        for(int i=2;i<n;i++)
            if(isPrimes[i])
                count++;
        return count;
    }
}
```

###### [分割数组为连续子序列](https://leetcode.cn/problems/split-array-into-consecutive-subsequences/)
给你一个按 非递减顺序 排列的整数数组 `nums` 。

请你判断是否能在将 `nums` 分割成 一个或多个子序列 的同时满足下述 两个 条件：

+ 每个子序列都是一个 连续递增序列（即，每个整数 恰好 比前一个整数大 1 ）。
+ 所有子序列的长度 至少 为 `3` 。

如果可以分割 `nums` 并满足上述条件，则返回 `true` ；否则，返回 `false` 。

这道题的思路是挨个判断每个数组，考虑到对于数组中的每个数字，只有两种选择，第一种是将该数字加到一个序列的后面，一个是将该数字作为一个序列的开头，所以整体思路就是对每个数字判断是否属于这两种情况，如果不属于就返回false

具体实现的话定义两个hashmap，freq记录了每个数字出现的次数，`need`** 记录哪些元素可以被接到其他子序列后面**

```java
class Solution {
    public boolean isPossible(int[] nums) {
        HashMap<Integer,Integer>freq=new HashMap<>();
        HashMap<Integer,Integer>need=new HashMap<>();
        //统计每个元素的数量
        for(int n:nums)
            freq.put(n,freq.getOrDefault(n,0)+1);
        for(int n:nums){
            if(freq.get(n)==0)
                continue;
            //加到一个序列中
            if(need.containsKey(n)&&need.get(n)>0){
                freq.put(n,freq.getOrDefault(n,0)-1);
                need.put(n,need.getOrDefault(n,0)-1);
                need.put(n+1,need.getOrDefault(n+1,0)+1);
            //变成一个新序列的头
            }else if(freq.containsKey(n)&&freq.get(n)>0&&freq.containsKey(n+1)&&freq.get(n+1)>0&&freq.containsKey(n+2)&&freq.get(n+2)>0){
                freq.put(n, freq.get(n) - 1);
                freq.put(n + 1, freq.get(n + 1) - 1);2
                freq.put(n + 2, freq.get(n + 2) - 1);
                need.put(n + 3, need.getOrDefault(n + 3, 0) + 1);
            }else{
                return false;
            }
        }
        return true;
    }
}
```

###### [煎饼排序](https://leetcode.cn/problems/pancake-sorting/)
给你一个整数数组 `arr` ，请使用 煎饼翻转 完成对数组的排序。

一次煎饼翻转的执行过程如下：

+ 选择一个整数 `k` ，`1 <= k <= arr.length`
+ 反转子数组 `arr[0...k-1]`（下标从 0 开始）

本质上就是一个选择排序，每次挑出当前集合中的最大值，反转两次，一次反转到数组头，一次反转到数组尾，注意是反转不是交换

```java
class Solution {
    List<Integer>res=new ArrayList<>();
    public List<Integer> pancakeSort(int[] arr) {
        sort(arr,arr.length);
        return res;
    }

    public void sort(int []arr,int n){
        if(n==1)
            return;
        int max=Integer.MIN_VALUE;
        int index=-1;
        for(int i=0;i<n;i++){
            if(arr[i]>max){
                max=arr[i];
                index=i;
            }
        }
        reverse(arr,0,index);
        res.add(index+1);
        reverse(arr,0,n-1);
        res.add(n);
        sort(arr,n-1);
    }

    public void reverse(int []arr,int i,int j){
        while(i<j){
            int temp=arr[i];
            arr[i]=arr[j];
            arr[j]=temp;
            i++;
            j--;
        }
    }
}
```

###### [字符串相乘](https://leetcode.cn/problems/multiply-strings/)
给定两个以字符串形式表示的非负整数 `num1` 和 `num2`，返回 `num1` 和 `num2` 的乘积，它们的乘积也表示为字符串形式。

这道题因为考虑num1，num2的过长的情况，且不能使用任何内置的 BigInteger 库或直接将输入转换为整数。所以考虑模拟乘法计算式来计算

整体的思路就是按照下面的乘法式来进行计算

![](https://cdn.jsdelivr.net/gh/ddyycc123/imageloader@main/image-20231229163756219.png)

使用一个res的数组来记录结果，这里需要注意的是，每次i，j计算的时候所涉及到的res中的索引位置就是i+j和i+j+1

```java
class Solution {
    public String multiply(String num1, String num2) {
        int m=num1.length();int n=num2.length();
        int []res=new int[m+n];
        for(int i=m-1;i>=0;i--){
            for(int j=n-1;j>=0;j--){
                int mul=(num1.charAt(i)-'0')*(num2.charAt(j)-'0');
                int p=i+j;
                int q=i+j+1;
                  //考虑到进位的问题，所以先让mul和res[q]相加，得到完整结果再来在res中赋值
                int sum=res[q]+mul;
                res[q]=sum%10;
                res[p]+=sum/10;
            }
        }
        int i=0;
        //这里是考虑到数组前面有空位的情况，不一定把数组中所有位子都用完了
        while(i<res.length&&res[i]==0)
            i++;
        StringBuilder sb=new StringBuilder();
        for(;i<res.length;i++)
            sb.append(res[i]);
        String s=sb.toString();
        return s.length()==0?"0":s;
    }
}
```

###### [基本计算器](https://leetcode.cn/problems/basic-calculator/)
实现计算器要考虑几个点

+ 字符串怎么转数字-----》for循环转 如果不是运算符就一直num = 10 * num + (c-'0');
+ 先考虑加减法，遇到加减法直接把该num加入栈
+ 考虑乘除法，需要从栈中拿出一个数计算后放入栈
+ 考虑括号，用递归来计算，每一个括号内部就是一个eva
+ 考虑到用递归，所以用队列来存储字符串，这样每次传入就只有当前的了
+ 遇到）的时候，对这种情况的判断要放到最后，因为这个属于是符号，所以要对前面所有的括号内的结果进行清算，不能提前break
+ 本解法考虑的是遇到后面的运算符，就把前面的数字进行清算，所以用一个sign来记录前面的数字的符号 比如说6-2+2 最开始sign就是+，遇到c为-的时候就把+6传入栈，把sign改为-，遇到c为+，就把-2传入，sign改为+

```java
class Solution {
    public int calculate(String s) {
        Queue<Character> queue = new LinkedList<Character>();
        for(char c:s.toCharArray()){
            if(c!=' ')
                queue.add(c);
        }
        int res=eva(queue);
        return res;
    }

    public int eva(Queue<Character>queue){
        Stack<Integer>stack=new Stack<>();
        char sign='+';
        int num=0;
        while(!queue.isEmpty()){
            char c=queue.poll();
            if (Character.isDigit(c)) {
                num = 10 * num + (c-'0');
            }
            if(c=='(')
                num=eva(queue);
            if (!Character.isDigit(c)|| queue.isEmpty()) {
                if (sign == '+') {
                    stack.push(num);
                } else if (sign == '-') {
                    stack.push(-num);
                } else if (sign == '*') {
                    stack.push(stack.pop() * num);
                } else if (sign == '/') {
                    stack.push(stack.pop() / num);       
                }
                num = 0;
                sign = c;
            }
            if(c==')')
                break;
        }
        int res = 0;
        for (int i : stack) {
            res += i;
        }
        return res;

    }
}
```

###### [盛最多水的容器](https://leetcode.cn/problems/container-with-most-water/)
给定一个长度为 `n` 的整数数组 `height` 。有 `n` 条垂线，第 `i` 条线的两个端点是 `(i, 0)` 和 `(i, height[i])` 。

找出其中的两条线，使得它们与 `x` 轴共同构成的容器可以容纳最多的水。

返回容器可以储存的最大水量。

和前面的柱状图求最大矩形是一样的

解法1️⃣：直接暴力搜索所有情况 超时

```java
class Solution {
    public int maxArea(int[] height) {
        int max=Integer.MIN_VALUE;
        for(int i=0;i<height.length-1;i++){
            for(int j=i+1;j<height.length;j++){
                int h=height[i]<height[j]?height[i]:height[j];
                int size=h*(j-i);
                if(size>max)
                    max=size;
            }
        }
        return max;
    }
}
```

解法2️⃣：

双指针，每次求一个size后，再让小的那个指针移动

```java
class Solution {
    public int maxArea(int[] height) {
        int left=0;int right=height.length-1;
        int res=0;
        while(left<right){
            int size=Math.min(height[left],height[right])*(right-left);
            res=Math.max(res,size);
            if(height[left]<height[right])
                left++;
            else
                right--;
        }
        return res;
    }
}
```

###### [接雨水](https://leetcode.cn/problems/trapping-rain-water/)
![](https://cdn.jsdelivr.net/gh/ddyycc123/imageloader@main/image-20231229203109952.png)

接雨水问题，可以考虑其实对于每一个可以接雨水的格子，其能接的雨水的量有两边最高点中的最低点决定，所以可以进行暴力求解

解法1️⃣：计算每个接水点的两边最高点求其接水的量，有一个用例超时

```java
class Solution {
    int trap(int[] height) {
        int n = height.length;
        int res = 0;
        for (int i = 1; i < n - 1; i++) {
            int l_max = 0, r_max = 0;
              //求该位置右边最高点
            for (int j = i; j < n; j++)
                r_max = Math.max(r_max, height[j]);
            //求左边最高点
            for (int j = i; j >= 0; j--)
                l_max = Math.max(l_max, height[j]);
            //两个最高点的最小值就是高度
            res += Math.min(l_max, r_max) - height[i];
        }
        return res;
    }
}
```

解法2️⃣：用备忘录记录最高点，就不需要每次都再找一遍

```java
class Solution {
    int trap(int[] height) {
        int []l_max=new int[height.length];
        int []r_max=new int[height.length];
        l_max[0]=height[0];
        r_max[height.length-1]=height[height.length-1];
        for(int i=1;i<height.length;i++){
            l_max[i]=Math.max(l_max[i-1],height[i]);
        }
        for(int i=height.length-2;i>=0;i--){
            r_max[i]=Math.max(r_max[i+1],height[i]);
        }
        int res=0;
        for(int i=1;i<height.length-1;i++){
            res+=Math.min(l_max[i],r_max[i])-height[i];
        }
        return res;
    }
}
```

解法3️⃣：双指针

思路也是找两边的最大值，不过是边加边找

考虑的情况是：最开始有两个指针在数组的两端，所以它们分别有一端的最值是知道的，左指针的左边最大值是知道的，是自己，右指针的右边最大值也是自己，将这两个最大值进行比较

比如说l_max更小，而此时的r_max都不一定是left位置右边的最大值，所以l_max一定是两边的最小值，所以就计算left的雨水，反之就计算right的雨水，再逐个往中间推进

这里值得注意的是while判断条件是left<right，因为按照这个逻辑走，最后left和right相聚的点一定是整个数组的最大点，所以不会存在雨水

```java
class Solution {
    int trap(int[] height) {
        int left=0;int right=height.length-1;
        int l_max=0;int r_max=0;
        int res=0;
        while(left<right){
            l_max=Math.max(l_max,height[left]);
            r_max=Math.max(r_max,height[right]);
            if(l_max<r_max){
                res+=l_max-height[left];
                left++;
            }
            else{
                res+=r_max-height[right];
                right--;
            }

        }
        return res;
    }
}
```

###### [完美矩形](https://leetcode.cn/problems/perfect-rectangle/)
给你一个数组 `rectangles` ，其中 `rectangles[i] = [xi, yi, ai, bi]` 表示一个坐标轴平行的矩形。这个矩形的左下顶点是 `(xi, yi)` ，右上顶点是 `(ai, bi)` 。

如果所有矩形一起精确覆盖了某个矩形区域，则返回 `true` ；否则，返回 `false` 。

这道题判断完美矩形，要考虑以下两个问题：

1. 判断矩形组合之后时候会有重叠和空缺
2. 判断各个小矩形的点是否都连接

对于上面第一个问题通过判断面积是否相等来解决，找到最终矩形的顶点求的最终矩形的面积，然后与各个小矩形面积之和进行判断，就能看是否有重叠等问题

对于上面第二个问题，判断小矩形的点是否都连接，考虑如果两个矩形连接，那么他们连接的顶点就会融合消失，最后就只剩四个顶点，所以最后可以通过判断顶点数量来判断是否连接

![](https://cdn.jsdelivr.net/gh/ddyycc123/imageloader@main/image-20240105175103593.png)

从上图可以看出，情况二和情况四的红色顶点会消失，情况一和情况三的红色顶点是存在的，即如果该位置的顶点数量是奇数那该顶点就存在，否则该位置的顶点就消失，可以使用一个HashSet数组进行判断，如果在HashSet中存在就remove，如果不存在就加入，最后看HashSet中的顶点数量是否为4个。

不过考虑下面这个情况，即有两个矩阵融合，按照HashSet的思路，最后也只剩4个顶点，并且面积也是相同的，所以还要多一次判断，即判断最后的矩阵的四个顶点是不是HashSet中剩下的四个顶点，如果是HashSet中剩下的四个顶点，才能算成功

![](https://cdn.jsdelivr.net/gh/ddyycc123/imageloader@main/image-20240105175421096.png)

```java
class Solution {
    public boolean isRectangleCover(int[][] rectangles) {
        int X1=Integer.MAX_VALUE;int Y1=Integer.MAX_VALUE;
        int X2=Integer.MIN_VALUE;int Y2=Integer.MIN_VALUE;
        int actual_size=0;int expected_size=0;
        HashSet<String>points=new HashSet<>();
        for(int []rectangle:rectangles){
            int x1=rectangle[0];int y1=rectangle[1];
            int x2=rectangle[2];int y2=rectangle[3];
            //求完美矩阵的顶点
            X1=Math.min(x1,X1);
            Y1=Math.min(y1,Y1);
            X2=Math.max(x2,X2);
            Y2=Math.max(y2,Y2);
            //计算小矩阵的面积之和
            actual_size+=(x2-x1)*(y2-y1);
            //把四个顶点都加入HashSet，进行顶点判断
            String p1 = x1 + "," + y1;
            String p2 = x1 + "," + y2;
            String p3 = x2 + "," + y1;
            String p4 = x2 + "," + y2;
            String []ps={p1,p2,p3,p4};
            for(String p:ps){
                if(points.contains(p))
                    points.remove(p);
                else
                    points.add(p);
            }
        }
        //首先判断面积
        expected_size=(X2-X1)*(Y2-Y1);
        if(actual_size!=expected_size)
            return false;
        //判断顶点数量
        if(points.size()!=4)
            return false;
        //判断points中的四个顶点是否就是完美矩阵的四个顶点
        String p1 = X1 + "," + Y1;
        String p2 = X1 + "," + Y2;
        String p3 = X2 + "," + Y1;
        String p4 = X2 + "," + Y2;
        String []ps={p1,p2,p3,p4};
        for(String p:ps){
            if(!points.contains(p))
                return false;
        }
        return true;
    }
}
```

###### [把字符串转换成整数 (atoi)](https://leetcode.cn/problems/ba-zi-fu-chuan-zhuan-huan-cheng-zheng-shu-lcof/)
字符串转整数

```java
class Solution {
    public int myAtoi(String str) {
        int i=0;
        int sign=1;
        int val=Integer.MAX_VALUE/10;
        int res=0;
        if(str.length()==0)
            return 0;
        while(str.charAt(i)==' '){
            i++;
            if(i==str.length())
                return 0;
        }
        if(str.charAt(i)=='-'){
            sign=-1;
            i++;
        }else if(str.charAt(i)=='+'){
            //加这个判断是因为字符串里不一定会有符号
            i++;
        }
        for(int j=i;j<str.length();j++){
            if(str.charAt(j)<'0'||str.charAt(j)>'9')
                break;
            //判断越界
            if(res>val||res==val&&str.charAt(j)>Integer.MAX_VALUE%10+'0')
                return sign==1?Integer.MAX_VALUE:Integer.MIN_VALUE;
            res=res*10+(str.charAt(j)-'0');
        }
        return sign*res;
    }
}
```

###### [表示数值的字符串](https://www.nowcoder.com/practice/e69148f8528c4039ad89bb2546fd4ff8?tpId=13&tqId=11206&ru=/exam/oj)
判断字符串str是否表示数值（包括科学计数法的数字，小数和整数）

科学计数法的数字(按顺序）可以分成以下几个部分:

1.若干空格

2.一个整数或者小数

3.（可选）一个 'e' 或 'E' ，后面跟着一个整数(可正可负)

4.若干空格

判断字符串str是否表示数值

```java
public boolean isNumeric (String str) {
    // write code here
    boolean isnum=false,isdot=false,ise=false;
    char[] s=str.trim().toCharArray();
    for(int i=0;i<s.length;i++){
        if(s[i]>='0'&&s[i]<='9')
            isnum=true;
        else if(s[i]=='.'){
            if(isdot||ise)
                return false;
            isdot=true;
        }
        else if(s[i]=='e'||s[i]=='E'){
            if(ise||!isnum)
                return false;
            ise=true;
            isnum=false;
        }
        else if(s[i]=='+'||s[i]=='-'){
            if(i!=0&&s[i-1]!='e'&&s[i-1]!='E')
                return false;
        }
        else
            return false;
    }
    return isnum;
}
```

###### [连续的子数组和](https://leetcode.cn/problems/continuous-subarray-sum/)
给你一个整数数组 `nums` 和一个整数 `k` ，如果 `nums` 有一个 好的子数组 返回 `true` ，否则返回 `false`：

一个 好的子数组 是：

+ 长度 至少为 2 ，且
+ 子数组元素总和为 `k` 的倍数。

连续子数组和

方法1️⃣：直接双重for循环，超时

```java
class Solution {
    public boolean checkSubarraySum(int[] nums, int k) {
        for(int i=0;i<nums.length;i++){
            int sum=nums[i];
            for(int j=i+1;j<nums.length;j++){
                sum+=nums[j];
                if(sum%k==0||sum==0)
                    return true;
            }
        }
        return false;
    }
}
```

方法2️⃣：利用前缀和+hashmap进行求解

求长度至少为2，和为k的倍数的子数组和

思路是考虑是找k的倍数，首先求数组中每个数字对应的前缀和，然后对k进行求余，如果有两个数字求得的余数是一样的，并且这两个数字时间间隔大于等于2，说明这两个数字之间的数字之和为k的n倍

```java
class Solution {
    public boolean checkSubarraySum(int[] nums, int k) {
        Map<Integer,Integer>map=new HashMap<>();
        int prefix=0;
        int sum=0;
        map.put(0,-1);
        for(int i=0;i<nums.length;i++){
            sum+=nums[i];
            prefix=sum%k;
            if(map.containsKey(prefix)){
                if(i-map.get(prefix)>=2)
                    return true;
            }else{
                map.put(prefix,i);
            }
        }
        return false;
    }
}
```

###### [最小栈](https://leetcode.cn/problems/min-stack/)
设计一个支持 `push` ，`pop` ，`top` 操作，并能在常数时间内检索到最小元素的栈。

```java
class MinStack {
    Deque<Integer> xStack;
    Deque<Integer> minStack;

    public MinStack() {
        xStack = new LinkedList<Integer>();
        minStack = new LinkedList<Integer>();
        minStack.push(Integer.MAX_VALUE);
    }
    
    public void push(int x) {
        xStack.push(x);
        minStack.push(Math.min(minStack.peek(), x));
    }
    
    public void pop() {
        xStack.pop();
        minStack.pop();
    }
    
    public int top() {
        return xStack.peek();
    }
    
    public int getMin() {
        return minStack.peek();
    }
}

```

###### [Pow(x, n)](https://leetcode.cn/problems/powx-n/)
快速幂

```java
class Solution {
    public double myPow(double x, int n) {
        long N = n;
        return N >= 0 ? quickMul(x, N) : 1.0 / quickMul(x, -N);
    }

    public double quickMul(double x, long N) {
        if (N == 0) {
            return 1.0;
        }
        double y = quickMul(x, N / 2);
        return N % 2 == 0 ? y * y : y * y * x;
    }
}

```

###### [加一](https://leetcode.cn/problems/plus-one/)
给定一个由 整数 组成的 非空 数组所表示的非负整数，在该数的基础上加一。

最高位数字存放在数组的首位， 数组中每个元素只存储单个数字。

你可以假设除了整数 0 之外，这个整数不会以零开头

```java
class Solution {
    public int[] plusOne(int[] digits) {
        int index=digits.length-1;
        while(index>=0){
            if(digits[index]!=9){
                digits[index]+=1;
                break;
            }else{
                digits[index]=0;
                index--;
            }
        }
        if(index==-1){
            int []res=new int[digits.length+1];
            res[0]=1;
            System.arraycopy(digits, 0, res, 1, digits.length);
            return res;
        }
        return digits;
    }
}
```

###### [阶乘后的零](https://leetcode.cn/problems/factorial-trailing-zeroes/)
求阶乘后零的个数，给定一个整数 `n` ，返回 `n!` 结果中尾随零的数量

```java
class Solution {
    public int trailingZeroes(int n) {
        if(n==0)
            return 0;
        int index=5;
        int count=0;
        while(index<=n){
            for (int i =index; i % 5 == 0; i /= 5) {
                ++count;
            }
            index+=5;
        }
        return count;

    }
}
```

###### [二进制求和](https://leetcode.cn/problems/add-binary/)
给你两个二进制字符串 `a` 和 `b` ，以二进制字符串的形式返回它们的和

设置了carry作为进位，加法主要要考虑的就是进位，carry需要和两个数字当前位求和，然后除余求出新的当前位，再除以进制求出新的进位

```java
class Solution {
    public String addBinary(String a, String b) {
        StringBuffer ans=new StringBuffer();
        int n=Math.max(a.length(),b.length()),carry=0;
        for(int i=0;i<n;i++){
            carry+=i<a.length()?(a.charAt(a.length()-i-1)-'0'):0;
            carry+=i<b.length()?(b.charAt(b.length()-i-1)-'0'):0;
            ans.append((char)(carry%2+'0'));
            carry/=2;
        }
        if(carry>0)
            ans.append('1');
        ans.reverse();
        return ans.toString();
    }
}
```

###### <font style="color:rgb(89, 89, 89);">进制表示</font>
![](https://cdn.nlark.com/yuque/0/2024/png/42839395/1722569478372-fbf07dee-ef09-47b3-9c32-f614b67d2589.png)

思路：求进制，先取余，再除，直到为0

```java
import java.util.*;

public class Solution {
    public static void main(String[] args) {
        Scanner sc=new Scanner(System.in);
        int n=sc.nextInt();
        int res=0;
        for(int i=2;i<=36;i++){
            int t=n;
            int count=0;
            while(t>0){
                int z=t%i;
                count+=String.valueOf(z).chars().filter(ch->ch=='1').count();
                t/=i;
            }
            res=Math.max(res,count);
        }
        System.out.println(res);
    }
}
```

###### [颠倒二进制位](https://leetcode.cn/problems/reverse-bits/)
颠倒给定的 32 位无符号整数的二进制位

每一次取n的最后一位对res进行拼接

```java
public class Solution {
    // you need treat n as an unsigned value
    public int reverseBits(int n) {
        int i=32;
        int res=0;
        while(i-->0){
            res<<=1;
            res+=n&1;
            n>>=1;
        }
        return res;
    }

}
```

不用位运算

```java
public class Solution {
    // you need treat n as an unsigned value
    public int reverseBits(int n) {
        StringBuilder sb=new StringBuilder();
        String binaryString = Integer.toBinaryString(n);
       
        sb.append(binaryString);
        for (int i = 0; i < 32 - binaryString.length(); i++) {
            sb.insert(0, "0");
        }

        String s=sb.reverse().toString();
      
        int num=s.length();
        int res=0;
        for(int i=1;i<=num;i++){
            if(sb.charAt(i-1)!='0'){
                res+=pow(2,num-i);
            }
        }
        return res;
    }

    public int pow(int n,int k){
        if(k==0)
            return 1;
        int y=pow(n,k/2);
        return k%2==0?y*y:y*y*n;
    }

}
```

###### [数字范围按位与](https://leetcode.cn/problems/bitwise-and-of-numbers-range/)
给你两个整数 `left` 和 `right` ，表示区间 `[left, right]` ，返回此区间内所有数字 按位与 的结果（包含 `left` 、`right` 端点）。

求连续区间内所有元素与的结果，最直接的方式就是全部相与，但这会超时，可以考虑对于这些数字来说，只有全为1的位置，结果这个位置才为1，否则就为0，并且由于是连续的数，所以可以找两个数的最长前缀，就是这个区间元素与的结果

那怎么求最长前缀呢，就将两个数依次右移，直到两者相等，就说明找到了相同前缀，再将这个前缀左移刚刚右移的次数即可

```java
class Solution {
    public int rangeBitwiseAnd(int left, int right) {
        int count=0;
        while(left<right){
            left>>=1;
            right>>=1;
            count++;
        }
        return left<<count;
    }
}
```

###### KMP
```java
public class KMPAlgorithm {

    // 构建部分匹配表（前缀函数）
    private int[] buildPartialMatchTable(String pattern) {
        int[] lps = new int[pattern.length()]; // lps: longest proper prefix which is also suffix
        int length = 0;
        lps[0] = 0; // 第一个字符的部分匹配值为0

        int i = 1;
        while (i < pattern.length()) {
            if (pattern.charAt(i) == pattern.charAt(length)) {
                length++;
                lps[i] = length;
                i++;
            } else {
                if (length != 0) {
                    // 这里不递增 i，继续检查 lps[length - 1]
                    length = lps[length - 1];
                } else {
                    lps[i] = 0;
                    i++;
                }
            }
        }
        return lps;
    }

    // KMP 字符串匹配算法
    public int kmpSearch(String text, String pattern) {
        int[] lps = buildPartialMatchTable(pattern);
        int i = 0; // text 的指针
        int j = 0; // pattern 的指针

        while (i < text.length()) {
            if (pattern.charAt(j) == text.charAt(i)) {
                i++;
                j++;
            }

            if (j == pattern.length()) {
                // 找到匹配，返回匹配的起始位置
                return i - j;
            } else if (i < text.length() && pattern.charAt(j) != text.charAt(i)) {
                if (j != 0) {
                    j = lps[j - 1];
                } else {
                    i++;
                }
            }
        }
        // 如果没有匹配，返回 -1
        return -1;
    }

    public static void main(String[] args) {
        KMPAlgorithm kmp = new KMPAlgorithm();
        String text = "ABABDABACDABABCABAB";
        String pattern = "ABABCABAB";
        int result = kmp.kmpSearch(text, pattern);
        if (result == -1) {
            System.out.println("Pattern not found in text.");
        } else {
            System.out.println("Pattern found at index: " + result);
        }
    }
}
```

### 真题
###### 小于N的最大数
给定一个数组，求将数组中元素进行组合，得到小于N的最大数

```java
import java.util.Arrays;
import java.util.List;

public class MaxNumberFinder {
    
    /**
     * 返回第index位后可组成的最大的数字，包括第index位
     */
    private static int dfs(int[] arr, int[] nums, boolean preEq, int index) {
        if (index == nums.length) {
            return 0; // 如果考虑完了最后一位，那么能组成的最大值为0
        }
        
        if (preEq) {
            for (int i = arr.length - 1; i >= 0; i--) {
                if (arr[i] <= nums[index]) {
                    int temp = dfs(arr, nums, arr[i] == nums[index], index + 1);
                    if (temp != -1) {
                        return arr[i] * (int) Math.pow(10, nums.length - index - 1) + temp;
                    }
                }
            }
            if (index == 0) {
                return dfs(arr, nums, false, index + 1); // 从第二位开始尝试，将preEq置为False
            }
            return -1;
        } else {
            return arr[arr.length - 1] * (int) Math.pow(10, nums.length - index - 1) + dfs(arr, nums, false, index + 1);
        }
    }
    
    public static int find(int N, int[] arr) {
        Arrays.sort(arr);
        int[] nums = Integer.toString(N).chars().map(c -> c - '0').toArray(); // 将数字N转换成每一位的数组
        return dfs(arr, nums, true, 0);
    }
    
    public static void main(String[] args) {
        int[] arrInputList = {2, 3, 4, 5};
        int[] NList = {1234, 2234, 2231, 2134};
        int[] ansList = {555, 2233, 2225, 555};
        
        for (int i = 0; i < NList.length; i++) {
            int ans = find(NList[i], arrInputList);
            System.out.println(NList[i] + " " + Arrays.toString(arrInputList) + " " + ans);
        }
    }
}
```

###### [最大数](https://leetcode.cn/problems/largest-number/)
给定一组非负整数 `nums`，重新排列每个数的顺序（每个数不可拆分）使之组成一个最大的整数

思路：首先将每个数字变成string类型，方便后面排序时好compare，最后需要去除前缀零

```java
class Solution {
    public String largestNumber(int[] nums) {
        int n = nums.length;
        String[] ss = new String[n];
        for (int i = 0; i < n; i++) ss[i] = "" + nums[i];
        Arrays.sort(ss, (a, b) -> {
            String sa = a + b, sb = b + a ;
            return sb.compareTo(sa);
        });
        
        StringBuilder sb = new StringBuilder();
        for (String s : ss) 
            sb.append(s);
        int len = sb.length();
        int k = 0;
        while (k < len - 1 && sb.charAt(k) == '0') 
            k++;
        return sb.substring(k);
    }
}

```

###### [字符串相加](https://leetcode.cn/problems/add-strings/)
给定两个字符串形式的非负整数 `num1` 和`num2` ，计算它们的和并同样以字符串形式返回。

```java
class Solution {
    public String addStrings(String num1, String num2) {
        int i = num1.length() - 1, j = num2.length() - 1, add = 0;
        StringBuffer ans = new StringBuffer();
        while (i >= 0 || j >= 0 || add != 0) {
            int x = i >= 0 ? num1.charAt(i) - '0' : 0;
            int y = j >= 0 ? num2.charAt(j) - '0' : 0;
            int result = x + y + add;
            ans.append(result % 10);
            add = result / 10;
            i--;
            j--;
        }
        // 计算完以后的答案需要翻转过来
        ans.reverse();
        return ans.toString();
    }
}

```

###### [字典序排数](https://leetcode.cn/problems/lexicographical-numbers/)
给你一个整数 `n` ，按字典序返回范围 `[1, n]` 内所有整数。

你必须设计一个时间复杂度为 `O(n)` 且使用 `O(1)` 额外空间的算法。

> 输入：n = 13
>
> 输出：[1,10,11,12,13,2,3,4,5,6,7,8,9]
>

```java
class Solution {
    public List<Integer> lexicalOrder(int n) {
        List<Integer> ret = new ArrayList<Integer>();
        int number = 1;
        for (int i = 0; i < n; i++) {
            ret.add(number);
            if (number * 10 <= n) {
                number *= 10;
            } else {
                //当达到下面条件的时候就可以进行退位 向上退一位
                while (number % 10 == 9 || number + 1 > n) {
                    number /= 10;
                }
                number++;
            }
        }
        return ret;
    }
}
```

###### [字符串解码](https://leetcode.cn/problems/decode-string/)
```java
import java.util.Collections;
import java.util.LinkedList;
import java.util.Stack;

class Solution {
    public String decodeString(String s) {
        Stack<Character> stack = new Stack<>();
        for(char c : s.toCharArray()) {
            if(c != ']')
                stack.push(c); // 把所有的字母push进去，除了]
            else {
                //step 1: 取出[] 内的字符串
                StringBuilder sb = new StringBuilder();
                while(!stack.isEmpty() && Character.isLetter(stack.peek())) 
                    sb.insert(0, stack.pop());
                String sub = sb.toString(); //[ ]内的字符串
                stack.pop(); // 去除[
                //step 2: 获取倍数数字
                sb = new StringBuilder();
                while(!stack.isEmpty() && Character.isDigit(stack.peek()))
                    sb.insert(0, stack.pop());
                int count = Integer.valueOf(sb.toString()); //倍数
                //step 3: 根据倍数把字母再push回去
                while(count > 0) {
                    for(char ch : sub.toCharArray())
                        stack.push(ch);
                    count--;
                }
            }
        }
        //把栈里面所有的字母取出来
        StringBuilder retv = new StringBuilder();
        while(!stack.isEmpty())
            retv.insert(0, stack.pop());
        return retv.toString();
    }
}
```

###### 括号匹配
给定一个字符串，编写一段代码测试该段字符串的括号是否完全 闭合。

[()]{}{()()}-true.       [(()]- false

```java
import java.util.*;

class Solution {
    public static boolean areBracketsClosed(String ss) {
        Stack<Character>stack=new Stack<>();
        for(char c:ss.toCharArray()){
            switch (c){
                case '[':
                case '(':
                case '{':
                    stack.push(c);
                    break;
                case ')':
                    if(stack.isEmpty()||stack.pop()!='(')
                        return false;
                    break;
                case ']':
                    if(stack.isEmpty()||stack.pop()!='[')
                        return false;
                    break;
                case '}':
                    if(stack.isEmpty()||stack.pop()!='{')
                        return false;
                    break;
                default:
                    break;
            }
        }
        return stack.isEmpty();
    }

    public static void main(String[] args) {
        String str1 = "[()]{}{()()}";
        String str2 = "[(()]";

        System.out.println(areBracketsClosed(str1));  // true
        System.out.println(areBracketsClosed(str2));  // false
    }
}
```

###### 找重复数字
数组a[N]，存放了数字1至N-1，其中某个数字重复一次。写一个函数，找出被重复的数字。时间复杂度必须为O(N)，空间复杂度不能是O[N]。 函数原型:int find(int all, int N)

方法1️⃣：使用hash      不过没有利用到行参

```java
    public static int find(int []all, int N) {
        HashSet<Integer>set=new HashSet<>();
        for(int n:all){
            if(!set.contains(n))
                set.add(n);
            else
                return n;
        }
        return -1;
    }
```

方法2️⃣：对数组进行求和 然后相减

```java
import java.util.*;

class Solution {
    public static int find(int []all, int N) {
        int sum=(1+N-1)*(N-1)/2;
        int res=0;
        for(int n:all){
            res+=n;
        }
        return res-sum;
    }

    public static void main(String[] args) {
        int[] a = {1, 2, 3, 4, 5, 6, 7, 8, 9, 2};
        System.out.println(find(a,10));
    }
}
```

###### 二叉树路径和
```java
import java.util.*;

class TreeNode{
    TreeNode left;
    TreeNode right;
    int val;
    TreeNode(int val){
        this.val=val;
    }
}

class Solution {
    public static boolean hashpath(TreeNode root,int value) {
        if(root==null)
            return false;
        boolean res=dfs(root,value,0);
        return res;
    }

    public static boolean dfs(TreeNode node,int value,int cur){
        cur+=node.val;
        if(node.left==null&&node.right==null){
            if(cur==value)
                return true;
            return false;
        }
        boolean l=false,r=false;
        if(node.left!=null)
            l=dfs(node.left,value,cur);
        if(node.right!=null)
            r=dfs(node.right,value,cur);
        return l||r;
    }
}
```

###### 求无序数组最长子序列
给定一个无序的整数数组，找到其中最长的上升子序列。  
示例：  
输入：［10,9,2,5,3,7,101,18］  
输出：［2,3,7,101］或 ［2,3,7,18］说明：  
可能会有多种最长上升子序列的组合，你只需要输出其中一个即可

用动态规划求解，记录路径可以用stringbuffer记录，也可以用hashmap来记录

```java
import java.util.*;

public class Solution {
    public static void lengthOfLIS(int[] nums) {
        int []dp=new int[nums.length];
        List<StringBuffer>path=new LinkedList<>();
        for(int i=0;i<nums.length;i++){
            path.add(new StringBuffer(String.valueOf(i)));
        }
        Arrays.fill(dp,1);
        int max=0;
        int index=-1;
        for(int i=0;i<nums.length;i++){
            for(int j=0;j<i;j++){
                if(nums[j]<nums[i]){
                    if(dp[i]<dp[j]+1){
                        dp[i]=dp[j]+1;
                        path.get(i).setLength(0);
                        path.get(i).append(path.get(j)).append("->").append(nums[i]);
                    }
                }
            }
            if(dp[i]>max){
                max=dp[i];
                index=i;
            }
        }
        String r=path.get(index).toString();
        String []re=r.split("->");
        int []res=new int[re.length];
        for(int i=0;i<re.length;i++)
            res[i]=Integer.valueOf(re[i]);
        System.out.println(Arrays.toString(res));
    }

    public static void main(String[] args) {
        int []nums={10,9,2,5,3,7,101,18};
        lengthOfLIS(nums);
    }


}
```

###### LRU
```java
class LRUCache {
    class DLinkedNode{
        int key;
        int value;
        DLinkedNode prev;
        DLinkedNode next;
        DLinkedNode(){
        }
        public DLinkedNode(int key,int value){
            this.key=key;
            this.value=value;
        }
    }
    private Map<Integer,DLinkedNode>cache=new HashMap<Integer,DLinkedNode>();
    private int size;
    private int capacity;
    private DLinkedNode head,tail;

    public LRUCache(int capacity) {
        this.size=0;
        this.capacity=capacity;
        head=new DLinkedNode();
        tail=new DLinkedNode();
        head.next=tail;
        tail.prev=head;
    }
    
    public int get(int key) {
        DLinkedNode node=cache.get(key);
        if(node==null){
            return -1;
        }
        moveToHead(node);
        return node.value;
    }
    
    public void put(int key, int value) {
        DLinkedNode node=cache.get(key);
        if(node==null){
            DLinkedNode newNode=new DLinkedNode(key,value);
            //添加进hash表中
            cache.put(key,newNode);
            //添加至双向链表头部
            addToHead(newNode);
            size++;
            if(size>capacity){
                DLinkedNode tail=removeTail();
                cache.remove(tail.key);
                size--;
            }
        }else{
            node.value=value;
            moveToHead(node);
        }
    }
    private void addToHead(DLinkedNode node){
        node.prev=head;
        node.next=head.next;
        head.next.prev=node;
        head.next=node;
    }
    private void removeNode(DLinkedNode node){
        node.prev.next=node.next;
        node.next.prev=node.prev;
    }
    private void moveToHead(DLinkedNode node){
        removeNode(node);
        addToHead(node);
    }
    private DLinkedNode removeTail(){
        DLinkedNode res=tail.prev;
        removeNode(res);
        return res;
    }
}

/**
 * Your LRUCache object will be instantiated and called as such:
 * LRUCache obj = new LRUCache(capacity);
 * int param_1 = obj.get(key);
 * obj.put(key,value);
 */
```

###### 英文句子反转
```java
public class SentenceReverser {
    public static String reverseSentence(String sentence) {
        // 将句子按逗号分割
        String[] parts = sentence.split(",");
        StringBuilder result = new StringBuilder();

        // 遍历每个部分并反转单词的顺序
        for (String part : parts) {
            String[] words = part.trim().split(" ");
            for (int i = words.length - 1; i >= 0; i--) {
                result.append(words[i]);
                if (i > 0) {
                    result.append(" ");
                }
            }
            result.append(",");
        }

        // 移除最后一个多余的逗号
        result.deleteCharAt(result.length() - 1);
        return result.toString();
    }

    public static void main(String[] args) {
        String sentence = "hello world,give me money";
        String reversed = reverseSentence(sentence);
        System.out.println(reversed);
    }
}

```

###### 多多做作业
![](https://cdn.nlark.com/yuque/0/2024/png/42839395/1723604654898-4e7aaf70-f6ef-4413-9dc4-be2e202b2d76.png)

题意：<font style="color:rgb(51, 51, 51);">给定n个作业，作业有开始时间和需要的时间，求最小总作业消耗时间，单个作业消耗时间是作业完成时间减去作业开始时间，可以考虑使用优先队列进行贪心， 每次获取作业的时候，对于前面那个时间片，我们优先让消耗时间小的作业被计算</font>

```java
import java.util.*;

public class Main {
    public static void main(String[] args) {
        Scanner scanner = new Scanner(System.in);
        long n = scanner.nextLong();
        List<Pair> vp = new ArrayList<>();
        for (long i = 0; i < n; i++) {
            long first = scanner.nextLong();
            long second = scanner.nextLong();
            vp.add(new Pair(first, second));
        }

        // 使用有序队列，小的数字优先出队
        PriorityQueue<Long> pq = new PriorityQueue<>();
        long t = 1, ans = 0;
        pq.add(vp.get(0).second);
        //i表示目前要加入第几个作业
        for (long i = 1; i < n; i++) {
            //d表示前一个作业距离这一个作业还有几天，在这几天内就要对队列中的目前正在做的作业求消耗时间
            long d = vp.get((int) i).first - vp.get((int) (i - 1)).first;
            while (!pq.isEmpty() && d > 0) {
                //取出目前正在做的作业中耗时最小的
                long x = pq.poll();
                //dis用于计算要跳过的时间，看是当前要做的作业先做完，还是先加入新作业
                long dis = Math.min(x, d);
                //计算总耗时，队列中的每份作业都在耗时
                ans += dis * (pq.size() + 1);
                //跳过dis时间后，作业消耗时间和相隔天数都要减少
                x -= dis;
                d -= dis;
                if (x > 0) {
                    pq.add(x);
                }
            }
            pq.add(vp.get((int) i).second);
        }

        while (!pq.isEmpty()) {
            long x = pq.poll();
            ans += x * (pq.size() + 1);
        }

        System.out.println(ans);
    }

    // 辅助类用于存储数对
    static class Pair {
        long first;
        long second;

        public Pair(long first, long second) {
            this.first = first;
            this.second = second;
        }
    }
}
```

###### <font style="color:rgb(0, 0, 0);">积木</font>
![](https://cdn.nlark.com/yuque/0/2024/png/42839395/1723888109945-51c84d11-01ef-4c71-a442-c8c177b5c2a4.png)

```java
import java.util.*;

// 注意类名必须为 Main, 不要有任何 package xxx 信息
public class Main {
    public static void main(String[] args) {
        Scanner sc=new Scanner(System.in);
        int a=sc.nextInt();
        int b=sc.nextInt();
        String s1=sc.next();
        String s2=sc.next();
        int ans=a+b;
        for(int i=0;i<s1.length();i++){
            if(isValid(s1.substring(i),s2)){
                int len=i+Math.max(s1.length()-i,s2.length());
                ans=Math.min(ans,len);
            }
        }
        for(int i=0;i<s2.length();i++){
            if(isValid(s2.substring(i),s1)){
                int len=i+Math.max(s2.length()-i,s1.length());
                ans=Math.min(ans,len);
            }
        }
        System.out.println(ans);

    }
    public static boolean isValid(String s1,String s2){
        int num=Math.min(s1.length(),s2.length());
        for(int i=0;i<num;i++){
            if(s1.charAt(i)=='2'&&s2.charAt(i)=='2'){
                return false;
            }
        }
        return true;
    }
}
```

###### <font style="color:rgb(0, 0, 0);">过年</font>
![](https://cdn.nlark.com/yuque/0/2024/png/42839395/1723890409449-ddef2e4c-3765-43a2-a05f-daf67614b925.png)

```java
import java.util.*;

// 注意类名必须为 Main, 不要有任何 package xxx 信息
public class Main {
    public static void main(String[] args) {
        Scanner scanner=new Scanner(System.in);
        int numVertices = scanner.nextInt(), numEdges = scanner.nextInt(), maxWeight = scanner.nextInt();
        List<int []>edges=new ArrayList<>();
        for(int i=0;i<numEdges;i++){
            edges.add(new int[]{scanner.nextInt(),scanner.nextInt(),scanner.nextInt()});
        }
        Collections.sort(edges,(a,b)->{return a[2]-b[2];});
        //dp表示起始点到numVertices点花费为maxWeight的路径条数
        int [][]dp=new int[numVertices+1][maxWeight+1];
        dp[1][0]=1;
        // 用于标记是否存在多余模数的情况
        boolean[] overflowFlag = new boolean[numVertices+1];
        final int MODULO = 20220201;
        for(int weight=0;weight<maxWeight;weight++){
            for(int []edge:edges){
                if(edge[2]+weight>maxWeight)break;
                //统计的是路径数量，全部都加是因为如果之前没有那也是加0 没有影响
                dp[edge[1]][weight + edge[2]] += dp[edge[0]][weight];
                if(dp[edge[1]][weight + edge[2]]>MODULO){
                    dp[edge[1]][weight + edge[2]]-=MODULO;
                    //表示溢出
                    overflowFlag[edge[1]]=true;
                }
            }
        }
        if(overflowFlag[numVertices]){
            System.out.println("所有的路通向家！");
        }
        System.out.println(dp[numVertices][maxWeight]);

    }
}
```

###### 线程池
线程池

```java
public class ThreadPoolTrader implements Executor {

    private final AtomicInteger ctl = new AtomicInteger(0);

    private volatile int corePoolSize;
    private volatile int maximumPoolSize;

    private final BlockingQueue<Runnable> workQueue;

    public ThreadPoolTrader(int corePoolSize, int maximumPoolSize, BlockingQueue<Runnable> workQueue) {
        this.corePoolSize = corePoolSize;
        this.maximumPoolSize = maximumPoolSize;
        this.workQueue = workQueue;
    }
    //ctl是当前线程数 
    //1.判断当前工作线程是否达到核心线程数
    //1.1 如果没有就调用addWorker方法
    //1.2 如果达到就添加任务队列
    //    如果任务队列满了，就再次尝试再次添加 失败的话就拒绝
    @Override
    public void execute(Runnable command) {
        int c = ctl.get();
        if (c < corePoolSize) {
            if (!addWorker(command)) {
                reject();
            }
            return;
        }
        if (!workQueue.offer(command)) {
            if (!addWorker(command)) {
                reject();
            }
        }
    }

    //源码
    public void execute(Runnable command) {
        if (command == null)
            throw new NullPointerException();
        int c = ctl.get();
        if (workerCountOf(c) < corePoolSize) {
            if (addWorker(command, true))
                return;
            c = ctl.get();
        }
        if (isRunning(c) && workQueue.offer(command)) {
            int recheck = ctl.get();
            if (! isRunning(recheck) && remove(command))
                reject(command);
            else if (workerCountOf(recheck) == 0)
                addWorker(null, false);
        }
        else if (!addWorker(command, false))
            reject(command);
    }

    //addWorker方法用于创建并启动一个新的工作线程
    //首先检查当前的线程数是否已经达到 maximumPoolSize
    //如果达到最大值，返回 false 表示不能再添加新线程
    //如果可以添加线程，则创建一个新的 Worker 对象，并启动它。
    //启动后，线程数（ctl）增加
    private boolean addWorker(Runnable firstTask) {
        if (ctl.get() >= maximumPoolSize) 
            return false;
        Worker worker = new Worker(firstTask);
        worker.thread.start();
        ctl.incrementAndGet();
        return true;
    }

    //Worker 类是线程池中用于执行任务的工作线程
    //每个 Worker 都是一个 Runnable
    private final class Worker implements Runnable {

        final Thread thread;
        Runnable firstTask;

        public Worker(Runnable firstTask) {
            this.thread = new Thread(this);
            this.firstTask = firstTask;
        }

        //Worker 的 run 方法是线程的执行逻辑
        //直到任务队列为空或者线程数超出最大线程数。
        @Override
        public void run() {
            //它首先执行传入的 firstTask
            Runnable task = firstTask;
            try {
                //然后不断从任务队列中获取新任务并执行
                while (task != null || (task = getTask()) != null) {
                    task.run();
                    //当线程数大于最大线程数就可以退出
                    if (ctl.get() > maximumPoolSize) {
                        break;
                    }
                    task = null;
                }
            } finally {
                ctl.decrementAndGet();
            }
        }
        //从 workQueue 中获取任务
        //如果队列为空，会阻塞等待新任务
        private Runnable getTask() {
            while(true){
                try {
                    System.out.println("workQueue.size：" + workQueue.size());
                    return workQueue.take();
                } catch (InterruptedException e) {
                    e.printStackTrace();
                }
            }
        }
    }

    //拒接策略
    private void reject() {
        throw new RuntimeException("Error！ctl.count：" + ctl.get() + " workQueue.size：" + workQueue.size());
    }

    public static void main(String[] args) {
        ThreadPoolTrader threadPoolTrader = new ThreadPoolTrader(2, 2, new ArrayBlockingQueue<Runnable>(10));

        for (int i = 0; i < 10; i++) {
            int finalI = i;
            threadPoolTrader.execute(() -> {
                try {
                    Thread.sleep(1500);
                } catch (InterruptedException e) {
                    e.printStackTrace();
                }
                System.out.println("任务编号：" + finalI);
            });
        }
    }

}
```

###### 死锁
死锁

```java
public class DeadlockExample {

    public static void main(String[] args) {
        // 定义两个锁对象
        Object lock1 = new Object();
        Object lock2 = new Object();
        Thread thread1 = new Thread(() -> {
            synchronized (lock1) {
                System.out.println("Thread 1: Holding lock 1...");

                try { Thread.sleep(50); } catch (InterruptedException e) {}

                System.out.println("Thread 1: Waiting for lock 2...");
                synchronized (lock2) {
                    System.out.println("Thread 1: Holding lock 1 & 2...");
                }
            }
        });

        Thread thread2 = new Thread(() -> {
            synchronized (lock2) {
                System.out.println("Thread 2: Holding lock 2...");

                try { Thread.sleep(50); } catch (InterruptedException e) {}

                System.out.println("Thread 2: Waiting for lock 1...");
                synchronized (lock1) {
                    System.out.println("Thread 2: Holding lock 1 & 2...");
                }
            }
        });

        thread1.start();
        thread2.start();
    }
}
```

