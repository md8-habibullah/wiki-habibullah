---
title: C++ Ultimate Survival Guide
date:
description: A massive reference for C++20, STL, Memory Management, OOP, and Interview Patterns.
tags:
  - cpp
  - algorithms
  - dsa
  - interview
  - oop
  - LLM
---

> [!abstract] Overview
> C++ is the language of high-performance computing, game engines, and trading systems. Unlike Python or JS, it gives you direct control over hardware resources.
> **Philosophy:** "You only pay for what you use."

## 1. The Environment (Fedora Setup)
Since you are on **Fedora**, you have the best tooling available natively.

```bash
# Install GCC/G++ and Debuggers
sudo dnf install gcc-c++ gdb cmake valgrind

# Verify
g++ --version
````

-----

## 2\. The Standard Template Library (STL) 📦

The STL is your weapon for coding interviews. Never write your own linked list unless asked. Use the optimized containers.

### A. Vectors (Dynamic Arrays)

The default container. Contiguous memory, $O(1)$ access.

```cpp
#include <vector>
#include <algorithm> // for sort

void vectorMagic() {
    std::vector<int> nums = {5, 2, 9, 1, 5, 6};

    nums.push_back(10); // O(1) amortized
    
    // Sorting (IntroSort: Hybrid of QuickSort, HeapSort, InsertionSort)
    std::sort(nums.begin(), nums.end()); // O(N log N)

    // Binary Search (Only works on sorted arrays)
    bool exists = std::binary_search(nums.begin(), nums.end(), 9); 

    // Iteration (Modern C++11 range-based loop)
    for (int x : nums) {
        // do something
    }
}
```

### B. Maps & Sets (Hash Tables vs Trees)

Knowing the difference is critical for System Design.

| Container | Implementation | Search Time | Use Case |
| :--- | :--- | :--- | :--- |
| `std::map` | Red-Black Tree (BST) | $O(\log N)$ | Keys must be sorted/ordered. |
| `std::unordered_map` | Hash Table | $O(1)$ Avg | Speed is priority. LeetCode default. |
| `std::set` | Red-Black Tree | $O(\log N)$ | Unique elements, sorted. |

```cpp
#include <unordered_map>
#include <string>

void mapMagic() {
    std::unordered_map<std::string, int> cache;
    
    // Insert
    cache["user_1"] = 500;
    
    // Check existence (The safe way)
    if (cache.find("user_1") != cache.end()) {
        // Key exists
    }
}
```

-----

## 3\. Memory Management (The Interview Killer) 🧠

This is what separates a **Coder** from an **Engineer**.

### Stack vs Heap

  * **Stack:** Fast, automatic, small size. Variables declared inside functions.
  * **Heap:** Slow, manual, huge size. Created with `new` or `malloc`.

### The Old Way (Raw Pointers) - *Dangerous*

```cpp
int* ptr = new int(10); // Allocates on Heap
delete ptr;             // MUST delete, or Memory Leak happens
ptr = nullptr;          // Good practice to prevent dangling pointer
```

### The Modern Way (Smart Pointers) - *Safe*

Always use these in production code. (RAII Principle: Resource Acquisition Is Initialization).

```cpp
#include <memory>

void smartPointers() {
    // 1. unique_ptr (Exclusive Ownership)
    // Automatically deletes memory when it goes out of scope.
    std::unique_ptr<int> ptr1 = std::make_unique<int>(10);
    
    // 2. shared_ptr (Shared Ownership)
    // Uses Reference Counting. Deletes only when count reaches 0.
    std::shared_ptr<int> ptr2 = std::make_shared<int>(20);
    std::shared_ptr<int> ptr3 = ptr2; // Count = 2
} // Count = 0, memory freed automatically
```

-----

## 4\. Object Oriented Programming (OOP) 🏗️

### The Big 4 Concepts

1.  **Encapsulation:** Hiding data (private/public).
2.  **Abstraction:** Hiding complexity.
3.  **Inheritance:** `class Dog : public Animal`.
4.  **Polymorphism:** The ability to treat different objects as a common type.

### Virtual Functions & V-Tables

This is the most asked C++ interview question.

  * **Problem:** How does C++ know which function to call at runtime?
  * **Solution:** The V-Table (Virtual Table). A hidden pointer table created for classes with `virtual` functions.

<!-- end list -->

```cpp
class Shape {
public:
    // "virtual" enables Runtime Polymorphism
    virtual void draw() { 
        std::cout << "Drawing Shape"; 
    }
    // Pure Virtual Function (Makes this an Abstract Class/Interface)
    virtual void area() = 0; 
    
    // ALWAYS make destructor virtual in base classes!
    virtual ~Shape() {}
};

class Circle : public Shape {
public:
    void draw() override { // "override" keyword ensures safety
        std::cout << "Drawing Circle"; 
    }
    void area() override {
        // calculate area
    }
};

void render(Shape* s) {
    s->draw(); // Calls Circle::draw() if s points to a Circle
}
```

-----

## 5\. Modern C++ Features (C++11/14/17/20) 🚀

### Lambda Expressions

Anonymous functions. Great for custom sorting.

```cpp
auto sortDesc = [](int a, int b) { return a > b; };
std::sort(nums.begin(), nums.end(), sortDesc);
```

### Move Semantics (`std::move`)

Optimization to transfer resources instead of copying them.

```cpp
std::string a = "Heavy String...";
std::string b = std::move(a); 
// 'a' is now empty. 'b' owns the data. 0 Copies made. Very fast.
```

### Auto & Const

```cpp
const int MAX_USERS = 100; // Prefer const over #define
auto it = myMap.begin();   // Let compiler deduce types (Cleaner code)
```

-----

## 6\. Interview Algorithms Cheat Sheet 📝

### Pattern 1: Two Pointers (Arrays/Strings)

*Problem: Reverse string, Palindrome, Two Sum Sorted.*

```cpp
bool isPalindrome(std::string s) {
    int left = 0, right = s.length() - 1;
    while (left < right) {
        if (s[left] != s[right]) return false;
        left++;
        right--;
    }
    return true;
}
```

### Pattern 2: Sliding Window

*Problem: Longest Substring Without Repeating Characters.*

```cpp
int lengthOfLongestSubstring(std::string s) {
    std::vector<int> map(128, 0);
    int left = 0, right = 0, counter = 0, d = 0;
    
    while (right < s.length()) {
        if (map[s[right++]]++ > 0) counter++; 
        while (counter > 0) {
            if (map[s[left++]]-- > 1) counter--;
        }
        d = std::max(d, right - left);
    }
    return d;
}
```

### Pattern 3: Fast & Slow Pointers (Floyd's Cycle)

*Problem: Detect cycle in Linked List.*

```cpp
bool hasCycle(ListNode *head) {
    if (!head) return false;
    ListNode *slow = head, *fast = head;
    while (fast && fast->next) {
        slow = slow->next;
        fast = fast->next->next;
        if (slow == fast) return true;
    }
    return false;
}
```

### Pattern 4: Depth First Search (DFS) - Recursion

*Problem: Max Depth of Binary Tree.*

```cpp
int maxDepth(TreeNode* root) {
    if (!root) return 0;
    int left = maxDepth(root->left);
    int right = maxDepth(root->right);
    return std::max(left, right) + 1;
}
```

-----

## 7\. The "Gotchas" (Undefined Behavior) ⚠️

Things that will fail you in an interview.

1.  **Dangling Reference:** Returning a reference to a local variable.
    ```cpp
    int& bad() {
        int x = 10;
        return x; // ERROR: x is destroyed after function ends
    }
    ```
2.  **Iterator Invalidation:** Modifying a vector while looping over it with an iterator.
3.  **Memory Leaks:** Forgetting `delete` or circular references in `shared_ptr` (Fix: use `weak_ptr`).

-----

## Linked Notes

  * [[Memory-Management-Deep-Dive]]
  * [[Competitive-Programming-Setup]]
  * [[System-Design-Basics]]

<!-- end list -->
***

### How to use this note?
This note is dense.
1.  **When Reviewing:** Scan the "Gotchas" and "Smart Pointers" section before an interview.
2.  **When Coding:** Copy the "Two Pointers" or "DFS" snippet directly into your IDE to save time.
3.  **The Graph:** Link this to your `University` section (CS courses) and your `DevOps` section (System Programming).