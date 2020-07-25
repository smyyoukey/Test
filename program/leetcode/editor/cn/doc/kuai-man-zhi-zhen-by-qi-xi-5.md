### 解题思路
一个速度为1，一个速度为2 ，只要存在环，在一定时间内一定给会相遇

### 代码

```java
/**
 * Definition for singly-linked list.
 * class ListNode {
 *     int val;
 *     ListNode next;
 *     ListNode(int x) {
 *         val = x;
 *         next = null;
 *     }
 * }
 */
public class Solution {
    public boolean hasCycle(ListNode head) {
        if(head == null || head.next == null)
            return false;
        ListNode next = head.next.next;
        head = head.next;
        while(next != null && next.next != null){
            if(head.val == next.val)
                return true;
            next = next.next.next;
            head = head.next;
        }
        return false;
    }
}
```