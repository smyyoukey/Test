### 绪论
这篇文章是为初级读者准备的，文章中介绍栈和队列这两种数据结构。

#### 方法一 （两个队列，压入 -*O(1)*， 弹出 -*O(n)*）

**思路**

栈是一种 **后进先出**（last in - first out， LIFO）的数据结构，栈内元素从顶端压入（push），从顶端弹出（pop）。一般我们用数组或者链表来实现栈，但是这篇文章会来介绍如何用队列来实现栈。队列是一种与栈相反的 **先进先出**（first in - first out， FIFO）的数据结构，队列中元素只能从 `后端`（rear）入队（push），然后从 `前端`（front）端出队（pop）。为了满足栈的特性，我们需要维护两个队列 `q1` 和 `q2`。同时，我们用一个额外的变量来保存栈顶元素。

**算法**

**压入（push）**

新元素永远从 `q1` 的后端入队，同时 `q1` 的后端也是栈的 `栈顶`（top）元素。

 [Push an element in stack](https://pic.leetcode-cn.com/73b3988402ba76f30372520cd8a3dd77afd4f2bf54020966f4b8975708e84dc9-file_1561370741978)


*Figure 1. Push an element in stack*


```Java []
private Queue<Integer> q1 = new LinkedList<>();
private Queue<Integer> q2 = new LinkedList<>();
private int top;

// Push element x onto stack.
public void push(int x) {
    q1.add(x);
    top = x;
}
```

**复杂度分析**

* 时间复杂度：*O(1)*
队列是通过链表来实现的，`入队`（add）操作的时间复杂度为 *O(1)*。

* 空间复杂度：*O(1)*

**弹出（pop）**

我们需要把栈顶元素弹出，就是 `q1` 中最后入队的元素。

考虑到队列是一种 **FIFO** 的数据结构，最后入队的元素应该在最后被出队。因此我们需要维护另外一个队列 `q2`，这个队列用作临时存储 `q1` 中出队的元素。`q2` 中最后入队的元素将作为新的栈顶元素。接着将 `q1` 中最后剩下的元素出队。我们通过把 `q1` 和 `q2` 互相交换的方式来避免把 `q2` 中的元素往 `q1` 中拷贝。

 [Pop an element from stack](https://pic.leetcode-cn.com/558b9e9258a8ba35c6456ea714d05f55d35da3c3306bef8fa47099093a3ab5b7-file_1561370741978)


*Figure 2. Pop an element from stack*


```Java []
// Removes the element on top of the stack.
public void pop() {
    while (q1.size() > 1) {
        top = q1.remove();
        q2.add(top);
    }
    q1.remove();
    Queue<Integer> temp = q1;
    q1 = q2;
    q2 = temp;
}
```

**复杂度分析**

* 时间复杂度：*O(n)*
算法让 `q1` 中的 *n* 个元素出队，让 *n - 1* 个元素从 q2 入队，在这里 *n* 是栈的大小。这个过程总共产生了 *2n - 1* 次操作，时间复杂度为 *O(n)*。

#### 方法二 （两个队列， 压入 - *O(n)*， 弹出 - *O(1)*）

**算法**

**压入（push)**

接下来介绍的算法让每一个新元素从 `q2` 入队，同时把这个元素作为栈顶元素保存。当 `q1` 非空（也就是栈非空），我们让 `q1` 中所有的元素全部出队，再将出队的元素从 `q2` 入队。通过这样的方式，新元素（栈中的栈顶元素）将会在 `q2` 的前端。我们通过将 `q1`， `q2` 互相交换的方式来避免把 `q2` 中的元素往 `q1` 中拷贝。

 [Push an element in stack](https://pic.leetcode-cn.com/1acd10c255534e86719cf83b07f294c76967687c52db3ec44367d0cb7c45483e-file_1561370741978)


*Figure 3. Push an element in stack*


```Java []
public void push(int x) {
    q2.add(x);
    top = x;
    while (!q1.isEmpty()) {                
        q2.add(q1.remove());
    }
    Queue<Integer> temp = q1;
    q1 = q2;
    q2 = temp;
}
```

**复杂度分析**

* 时间复杂度：*O(n)*
算法会让 `q1` 出队 *n* 个元素，同时入队 *n + 1* 个元素到 `q2`。这个过程会产生 *2n + 1* 步操作，同时链表中 `插入` 操作和 `移除` 操作的时间复杂度为 *O(1)*，因此时间复杂度为 *O(n)*。

* 空间复杂度：*O(1)*

**弹出（pop）**

直接让 `q1` 中元素出队，同时将出队后的 `q1` 中的队首元素作为栈顶元素保存。

 [Pop an element from stack](https://pic.leetcode-cn.com/fc27d76b78bbe094f6912a0aa56dee5f8e618a4f04834ab043eb39ecb2e0cc93-file_1561370741978)


*Figure 4. Pop an element from stack*


```Java []
// Removes the element on top of the stack.
public int pop() {
    q1.remove();
    int res = top;
    if (!q1.isEmpty()) {
        top = q1.peek();
    }
    return res;
}
```

**复杂度分析**

* 时间复杂度：*O(1)*
* 空间复杂度：*O(1)*

`判断空`（empty）和 `取栈顶元素`（top）是同样的实现方式。

**判断空（empty）**

`q1` 里包含了栈中所有的元素，所以只需要检查 `q1` 是否为空就可以了。

```Java []
// Returns whether the stack is empty.
public boolean empty() {
    return q1.isEmpty();
}
```

时间复杂度：*O(1)*
空间复杂度：*O(1)*

**取栈顶元素（top）**

栈顶元素被保存在 `top` 变量里，每次我们 `压入` 或者 `弹出` 一个元素的时候都会随之更新这个变量。

```Java []
// Get the top element.
public int top() {
    return top;
}
```

时间复杂度：*O(1)*
栈顶元素每次都是被提前计算出来的，同时只有 `top` 操作可以得到它的值。

空间复杂度：*O(1)*

#### 方法三 （一个队列， 压入 - *O(n)*， 弹出 - *O(1)*）

上面介绍的两个方法都有一个缺点，它们都用到了两个队列。下面介绍的方法只需要使用一个队列。

**算法**

**压入（push）**

当我们将一个元素从队列入队的时候，根据队列的性质这个元素会存在队列的后端。

但当我们实现一个栈的时候，最后入队的元素应该在前端，而不是在后端。为了实现这个目的，每当入队一个新元素的时候，我们可以把队列的顺序反转过来。

 [Push an element in stack](https://pic.leetcode-cn.com/69e61df7e86fc04ffcf6a37a5a502e40f4a651d69542e7829282b4c2013164b4-image.png)


*Figure 5. Push an element in stack*


```Java []
private LinkedList<Integer> q1 = new LinkedList<>();

// Push element x onto stack.
public void push(int x) {
    q1.add(x);
    int sz = q1.size();
    while (sz > 1) {
        q1.add(q1.remove());
        sz--;
    }
}
```

**复杂度分析**

* 时间复杂度：*O(n)*
这个算法需要从 `q1` 中出队 *n* 个元素，同时还需要入队 *n* 个元素到 `q1`，其中 *n* 是栈的大小。这个过程总共产生了 *2n + 1* 步操作。链表中 `插入` 操作和 `移除` 操作的时间复杂度为 *O(1)*，因此时间复杂度为 *O(n)*。

* 空间复杂度：*O(1)*

**弹出（pop）**

最后一个压入的元素永远在 `q1` 的前端，这样的话我们就能在常数时间内让它 `出队`。

```Java []
// Removes the element on top of the stack.
public void pop() {
    q1.remove();
}
```

**复杂度分析**

* 时间复杂度：*O(1)*
* 空间复杂度：*O(1)*

**判断空（empty）**

`q1` 中包含了栈中所有的元素，所以只需要检查 `q1` 是否为空就可以了。

```Java []
// Return whether the stack is empty.
public boolean empty() {
    return q1.isEmpty();
}
```

时间复杂度：*O(1)*
空间复杂度：*O(1)*

**取栈顶（top）**

栈顶元素永远在 `q1` 的前端，直接返回就可以了。

```Java []
// Get the top element.
public int top() {
    return q1.peek();
}
```

时间复杂度：*O(1)*
空间复杂度：*O(1)*