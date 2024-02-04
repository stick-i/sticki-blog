# LeetCode 剑指 Offer 35. 复杂链表的复制

## 题目描述

请实现 copyRandomList 函数，复制一个复杂链表。在复杂链表中，每个节点除了有一个 next 指针指向下一个节点，还有一个 random 指针指向链表中的任意节点或者 null。

**示例 1：**

![img](https://assets.leetcode-cn.com/aliyun-lc-upload/uploads/2020/01/09/e1.png)

```
输入：head = [[7,null],[13,0],[11,4],[10,2],[1,0]]
输出：[[7,null],[13,0],[11,4],[10,2],[1,0]]
```

**示例 2：**

![img](https://assets.leetcode-cn.com/aliyun-lc-upload/uploads/2020/01/09/e2.png)

```
输入：head = [[1,1],[2,1]]
输出：[[1,1],[2,1]]
```

**示例 3：**

```
输入：head = []
输出：[]
解释：给定的链表为空（空指针），因此返回 null。
```

**提示：**

- -10000 <= Node.val <= 10000
- Node.random 为空（null）或指向链表中的节点。
- 节点数目不超过 1000 。

**题目链接**
[https://leetcode.cn/problems/fu-za-lian-biao-de-fu-zhi-lcof](https://leetcode.cn/problems/fu-za-lian-biao-de-fu-zhi-lcof)

## 解题思路
这题我刚开始也没看懂题，直接`return head;` 然后就提交了🤣🤣

然后发现不对，它题目的意思是，所有的节点你都必须重新new一个才行，只要你的链表节点中有对它原来的链表的引用那就不对。

于是我想到了用map映射的方法写出了这道题，后来看到评论区大哥们说另一种思路，于是我用那种思路也实现了一遍，都贴在下面了。

### Map映射法
时间复杂度O(n) ，空间复杂度O(1)。

大致思路：
- 将新节点先串成普通链表，并将新节点和老节点对应的put到map中
- 将新链表的每一个random进行更新，通过map中的映射关系。

```java
class Solution {
    public Node copyRandomList(Node head) {
        HashMap<Node, Node> map = new HashMap<>(2000);
        Node first = new Node(0);
        Node cur = first;
        // 将新节点先串成普通链表，并将新节点和老节点对应的put到map中
        while ( head != null) {
            cur.next = new Node(head.val);
            cur.next.random = head.random;
            map.put(head,cur.next);
            cur = cur.next;
            head = head.next;
        }
        // 将新链表的每一个random进行更新，通过map中的映射关系。
        cur = first.next;
        while ( cur != null ) {
            if (cur.random != null) {
                cur.random = map.get(cur.random);
            }
            cur = cur.next;
        }
        return first.next;
    }
}
```
运行截图

![image-20221017224854036](image/image-20221017224854036.png)


### 原地更新法

时间复杂度O(n) ，空间复杂度O(1)。

这个方法个人感觉比HashMap难想一点，而且它步骤也多一点，我分为了三步：

1. 先在每个节点后面都复制一个新的节点，例如对于链表 A → B → C，我们可以将其拆分为 A → A' → B → B' → C → C'。
2. 然后把新结点的random节点进行调整，修改为目标节点的下一个节点。
3. 把两个链表进行分离，返回新链表。

需要注意的是，要把原来的链表也进行还原，不能只把自己的链表拼接就不管旧链表了，否则系统会判错。

==这个方法建议在草稿纸上先画一下图解，判断一下指针改如何改变，再解题。==

```java
class Solution {
    public Node copyRandomList(Node head) {
        if (head == null) {
            return head;
        }
        Node h = head;
        // 第一步先在每个节点后面都复制一个新的节点
        while (head != null) {
            Node temp = new Node(head.val);
            temp.next = head.next;
            head.next = temp;
            head = head.next.next;
        }
        // 第二步把新结点的random节点进行调整
        head = h;
        while (head != null) {
            Node cur = head.next;
            if (head.random != null) {
                cur.random = head.random.next;
            }
            head = head.next.next;
        }
        // 第三步把两个链表进行分离
        head = h;
        Node myHead = head.next;
        while (head != null && head.next.next != null) {
            Node cur = head.next;
            head.next = head.next.next;
            cur.next = cur.next.next;
            head = head.next;
        }
        // 这里要把最后节点的next指向null
        head.next = null;
        return myHead;
    }

}
```
运行截图

![image-20221017224915927](image/image-20221017224915927.png)