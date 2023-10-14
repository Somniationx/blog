# 栈实现队列

## 题目描述

请你仅使用两个栈实现先入先出队列。队列应当支持一般队列支持的所有操作（`push`、`pop`、`peek`、`empty`）：

实现 `MyQueue` 类：

- `void push(int x)` 将元素 x 推到队列的末尾
- `int pop()` 从队列的开头移除并返回元素
- `int peek()` 返回队列开头的元素
- `boolean empty()` 如果队列为空，返回 `true` ；否则，返回 `false`

**说明：**

- 你 **只能** 使用标准的栈操作 —— 也就是只有 `push to top`, `peek/pop from top`, `size`, 和 `is empty` 操作是合法的。
- 你所使用的语言也许不支持栈。你可以使用 `list` 或者 `deque`（双端队列）来模拟一个栈，只要是标准的栈操作即可。

 

**示例 1：**

```html
输入：
["MyQueue", "push", "push", "peek", "pop", "empty"]
[[], [1], [2], [], [], []]
输出：
[null, null, null, 1, 1, false]

解释：
MyQueue myQueue = new MyQueue();
myQueue.push(1); // queue is: [1]
myQueue.push(2); // queue is: [1, 2] (leftmost is front of the queue)
myQueue.peek(); // return 1
myQueue.pop(); // return 1, queue is [2]
myQueue.empty(); // return false
```

**提示：**

- `1 <= x <= 9`
- 最多调用 `100` 次 `push`、`pop`、`peek` 和 `empty`
- 假设所有操作都是有效的 （例如，一个空的队列不会调用 `pop` 或者 `peek` 操作）

**进阶：**

- 你能否实现每个操作均摊时间复杂度为 `O(1)` 的队列？换句话说，执行 `n` 个操作的总时间复杂度为 `O(n)` ，即使其中一个操作可能花费较长时间。

## 代码实现

```cpp
class MyQueue {
public:
    MyQueue() {}

    void push(int x) {
        // 将元素推入 stack1，模拟入队操作
        stack1.push(x);
    }

    int pop() {
        if (stack2.empty()) {
            // 如果 stack2 为空，将 stack1 中的元素逐个弹出并压入 stack2，以实现出队操作
            while (!stack1.empty()) {
                stack2.push(stack1.top());
                stack1.pop();
            }
        }
        // 弹出并返回 stack2 的栈顶元素
        int front = stack2.top();
        stack2.pop();
        return front;
    }

    int peek() {
        if (stack2.empty()) {
            // 如果 stack2 为空，将 stack1 中的元素逐个弹出并压入 stack2，以实现获取队首元素操作
            while (!stack1.empty()) {
                stack2.push(stack1.top());
                stack1.pop();
            }
        }
        // 返回 stack2 的栈顶元素，不弹出
        return stack2.top();
    }

    bool empty() {
        // 如果两个栈都为空，队列为空
        return stack1.empty() && stack2.empty();
    }

private:
    std::stack<int> stack1; // 用于入队操作
    std::stack<int> stack2; // 用于出队操作
};
```

## 解题思路

1. 队列的基本特性是先进先出 (FIFO)，但栈的特性是后进先出 (LIFO)。为了模拟队列操作，我们使用两个栈，`stack1` 和 `stack2`。其中 `stack1` 用于入队操作，而 `stack2` 用于出队和获取队首元素操作。

2. `push(int x)` 方法用于入队操作。它将元素 `x` 推入 `stack1` 中，因为队列的元素是按照入队的顺序排列的。

3. `pop()` 方法用于出队操作。如果 `stack2` 为空，这意味着当前没有元素准备好出队。为了执行出队操作，需要将 `stack1` 中的元素逐个弹出并压入 `stack2`，以确保最早入队的元素位于 `stack2` 的栈底，最晚入队的元素位于栈顶。然后，我们从 `stack2` 中弹出并返回栈顶元素，模拟出队操作。

4. `peek()` 方法用于获取队首元素，但不执行出队操作。它的实现方式与 `pop()` 方法类似，如果 `stack2` 为空，需要将 `stack1` 中的元素逐个弹出并压入 `stack2`，然后返回 `stack2` 的栈顶元素，但不将其从 `stack2` 中弹出。

5. `empty()` 方法用于检查队列是否为空。当两个栈都为空时，队列为空。这是因为如果两个栈都为空，意味着没有元素在队列中。

这种方法确保了队列的先进先出特性，同时使用两个栈来模拟队列操作。这是一种非常有效的方法，可以执行队列的基本操作，如入队、出队、获取队首元素和检查队列是否为空。