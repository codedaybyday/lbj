## 二叉树

### 前序遍历
1. 递归实现, 需要实现一个辅助函数，作用是递归修改结果

```javascript
function preOrderTraversal(root) {
    const res = [];
    if (!root) {
        return res;
    }

    _traversal(root, res);

    return res;
}

function _traversal(root, res) {
    // 终止条件
    if (!root) {
        return res;
    }
    // 先访问根节点
    res.push(root.val);
    _traversal(root.left, res);
    _traversal(root.right, res);
}
```
2. 非递归实现,两个重点
- 使用栈
- 左节点后入栈保证下次遍历的时候能最先访问到
```javascript
function preOrderTraversal(root) {
    const stack = [];
    const res = [];

    if (!root) {
        return res;
    }

    stack.push(root);

    while (stack.length) {
        const node = stack.pop();
        res.push(node);

        if (node.right) {
            stack.push(node.right);
        }

        if (node.left) {
            stack.push(node.left);
        }
    }
}
```

### 中序遍历
1. 递归
```javascript
function inorderTraversal(root) {
    const res = [];
    _traversal(root, res);

    return root;
}

function _traversal(root, res) {
    // 终止条件
    if (!root) {
        return res;
    }

    _traversal(root.left, res);
    res.push(root.val);
    _traversal(root.right, res);
}
```
2. 非递归
```javascript
function inorderTraversal(root) {
    const stack = [];
    const result = [];
    while(root || stack.length) {
        // 一直遍历，到没有左节点
        while(root) {
            stack.push(root);
            root = root.left; // 指针移动
        }
        // 出栈
        root = stack[stack.length - 1];
        result.push(root.val); // left
        const node = stack.pop();
        root = node.right; // 开始遍历右边
    }

    return result;
}
```
为什么前序遍历中while只要判断stack的长度即可，中序遍历既要判断当前节点还要判断stack的长度?

在前序遍历中，我们使用了一个辅助栈来存储待处理的节点。在每次循环中，我们都将当前节点的值加入结果数组，并依次处理其左子节点和右子节点。由于前序遍历的顺序是根节点 -> 左子节点 -> 右子节点，所以我们只需要判断栈的长度是否大于0，即可确定是否还有待处理的节点。

而在中序遍历中，我们需要先处理左子树，然后处理根节点，最后处理右子树。所以在每次循环中，我们首先将当前节点及其左子树的所有左节点依次入栈，直到当前节点为空。然后，从栈中弹出一个节点，将其值加入结果数组中，并将指针 curr 指向该节点的右子树。此时，我们需要判断当前节点是否为空，以及栈的长度是否大于0，来确定是否还有待处理的节点。

因此，在中序遍历中，我们需要同时判断当前节点是否为空和栈的长度是否大于0，以确保处理完所有的节点。而在前序遍历中，由于根节点总是先被处理，所以只需要判断栈的长度是否大于0即可。
