# Practice Problems: Advanced Data Structures and Algorithms

## Problem 1: Reverse a Linked List

Write a function that takes the head of a singly linked list and reverses it. Return the new head of the reversed list.

### Solution

```cpp
#include <iostream>
#include <memory>

template <typename T>
struct Node {
    T data;
    std::unique_ptr<Node<T>> next;
    Node(T d) : data(d), next(nullptr) {}
};

// This implementation is tricky with unique_ptr due to ownership rules.
// A more common interview-style implementation would use raw pointers.
// Let's show the raw pointer version as it's more typical for this problem.

struct RawNode {
    int data;
    RawNode* next;
    RawNode(int d) : data(d), next(nullptr) {}
};

RawNode* reverseList(RawNode* head) {
    RawNode* prev = nullptr;
    RawNode* current = head;
    RawNode* next = nullptr;

    while (current != nullptr) {
        next = current->next;
        current->next = prev;
        prev = current;
        current = next;
    }
    return prev;
}

void printList(RawNode* head) {
    while (head) {
        std::cout << head->data << " ";
        head = head->next;
    }
    std::cout << std::endl;
}

int main() {
    RawNode* head = new RawNode(1);
    head->next = new RawNode(2);
    head->next->next = new RawNode(3);

    printList(head);
    head = reverseList(head);
    printList(head);

    // Clean up memory
    while (head) {
        RawNode* temp = head;
        head = head->next;
        delete temp;
    }
    return 0;
}
```

## Problem 2: Binary Search Tree Validation

Write a function that determines if a given binary tree is a valid Binary Search Tree (BST).

### Solution

```cpp
#include <iostream>
#include <climits>

struct TreeNode {
    int val;
    TreeNode *left;
    TreeNode *right;
    TreeNode(int x) : val(x), left(nullptr), right(nullptr) {}
};

bool isValidBSTHelper(TreeNode* node, long minVal, long maxVal) {
    if (!node) return true;
    if (node->val <= minVal || node->val >= maxVal) return false;
    
    return isValidBSTHelper(node->left, minVal, node->val) && 
           isValidBSTHelper(node->right, node->val, maxVal);
}

bool isValidBST(TreeNode* root) {
    return isValidBSTHelper(root, LONG_MIN, LONG_MAX);
}

int main() {
    // Valid BST
    TreeNode* root1 = new TreeNode(2);
    root1->left = new TreeNode(1);
    root1->right = new TreeNode(3);
    std::cout << "Tree 1 is valid BST: " << std::boolalpha << isValidBST(root1) << std::endl;

    // Invalid BST
    TreeNode* root2 = new TreeNode(5);
    root2->left = new TreeNode(1);
    root2->right = new TreeNode(4);
    root2->right->left = new TreeNode(3);
    root2->right->right = new TreeNode(6);
    std::cout << "Tree 2 is valid BST: " << std::boolalpha << isValidBST(root2) << std::endl;

    // ... cleanup memory (omitted for brevity)
    return 0;
}
```

## Problem 3: 0/1 Knapsack Problem (Dynamic Programming)

Given weights and values of n items, put these items in a knapsack of capacity W to get the maximum total value in the knapsack.

### Solution

```cpp
#include <iostream>
#include <vector>
#include <algorithm>

int knapsack(int W, const std::vector<int>& weights, const std::vector<int>& values) {
    int n = weights.size();
    std::vector<std::vector<int>> dp(n + 1, std::vector<int>(W + 1, 0));

    for (int i = 1; i <= n; ++i) {
        for (int w = 1; w <= W; ++w) {
            if (weights[i - 1] <= w) {
                dp[i][w] = std::max(values[i - 1] + dp[i - 1][w - weights[i - 1]],
                                    dp[i - 1][w]);
            } else {
                dp[i][w] = dp[i - 1][w];
            }
        }
    }
    return dp[n][W];
}

int main() {
    std::vector<int> values = {60, 100, 120};
    std::vector<int> weights = {10, 20, 30};
    int W = 50;

    int max_value = knapsack(W, weights, values);
    std::cout << "Maximum value in knapsack: " << max_value << std::endl; // Expected: 220

    return 0;
}
```
This DP solution builds a table `dp[i][w]` which stores the maximum value that can be obtained using the first `i` items with a knapsack capacity of `w`.
