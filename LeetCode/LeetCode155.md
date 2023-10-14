# 最小栈

## 题目描述

设计一个支持 `push` ，`pop` ，`top` 操作，并能在常数时间内检索到最小元素的栈。

实现 `MinStack` 类:

- `MinStack()` 初始化堆栈对象。
- `void push(int val)` 将元素val推入堆栈。
- `void pop()` 删除堆栈顶部的元素。
- `int top()` 获取堆栈顶部的元素。
- `int getMin()` 获取堆栈中的最小元素。

**示例 1:**

```
输入：
["MinStack","push","push","push","getMin","pop","top","getMin"]
[[],[-2],[0],[-3],[],[],[],[]]

输出：
[null,null,null,null,-3,null,0,-2]

解释：
MinStack minStack = new MinStack();
minStack.push(-2);
minStack.push(0);
minStack.push(-3);
minStack.getMin();   --> 返回 -3.
minStack.pop();
minStack.top();      --> 返回 0.
minStack.getMin();   --> 返回 -2.
```

**提示：**

- `-231 <= val <= 231 - 1`
- `pop`、`top` 和 `getMin` 操作总是在 **非空栈** 上调用
- `push`, `pop`, `top`, and `getMin`最多被调用 `3 * 104` 次

## 代码实现

```cpp
class MinStack {
public:
    MinStack() {
        // 构造函数，初始化MinStack对象。
    }
    
    void push(int val) {
        _elem.push(val); // 将元素值压入元素栈。

        // 如果最小栈为空或者val小于等于最小栈的栈顶元素，则将val也压入最小栈。
        if(_min.empty() || val <= _min.top()) {
            _min.push(val);
        }
    }
    
    void pop() {
        // 如果元素栈的栈顶元素等于最小栈的栈顶元素，表示要弹出的元素是最小值，需要同时从最小栈和元素栈中弹出。
        if(_elem.top() == _min.top()) {
            _min.pop();
        }
        _elem.pop(); // 从元素栈中弹出元素。
    }
    
    int top() {
        return _elem.top(); // 返回元素栈的栈顶元素。
    }
    
    int getMin() {
        return _min.top(); // 返回最小栈的栈顶元素，即最小值。
    }
private:
    std::stack<int> _elem; // 用于存储元素的栈。
    std::stack<int> _min; // 用于存储最小值的栈。
};
```

## 解题思路

- 构造函数 `MinStack()`：这是类的构造函数，用于初始化 `MinStack` 对象。

- `push(int val)` 方法：这个方法用于将元素值 `val` 压入元素栈 `_elem` 中。如果最小栈 `_min` 为空，或者 `val` 小于等于最小栈的栈顶元素，那么也将 `val` 压入最小栈 `_min` 中，以确保 `_min` 始终包含当前的最小元素。

- `pop()` 方法：这个方法用于出栈操作。如果元素栈 `_elem` 的栈顶元素等于最小栈 `_min` 的栈顶元素，表示要弹出的元素是最小值，因此需要同时从最小栈和元素栈中弹出该元素。然后，只需要从元素栈 `_elem` 中弹出元素。

- `top()` 方法：这个方法用于返回元素栈 `_elem` 的栈顶元素，即最近入栈的元素。

- `getMin()` 方法：这个方法用于返回最小栈 `_min` 的栈顶元素，即最小值。由于在 `push` 操作中保持了最小栈 `_min` 始终包含当前的最小元素，因此可以轻松地获取最小值。

> 这个类实现了一个特殊的栈，可以在 `O(1)` 时间内获取栈中的最小值。这对于需要频繁查找最小值的问题非常有用，因为不需要每次都遍历整个栈来找到最小元素。