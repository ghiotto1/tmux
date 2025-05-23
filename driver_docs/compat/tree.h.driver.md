# Purpose
The provided code is a C header file that defines data structures and operations for two types of binary trees: splay trees and red-black trees. These trees are used for efficient data storage and retrieval, with each type offering distinct advantages. Splay trees are self-adjusting binary search trees that bring recently accessed elements to the root, optimizing for access patterns with locality of reference. Red-black trees, on the other hand, are balanced binary search trees that maintain balance through node coloring and specific properties, ensuring operations such as insertion, deletion, and lookup are performed in logarithmic time.

The file includes macros for defining tree structures, initializing them, and performing various operations such as insertion, removal, and searching. For splay trees, it provides macros to define tree heads and entries, and functions to perform splay operations, which adjust the tree structure to bring a node closer to the root. For red-black trees, it defines macros for tree nodes with color attributes and functions to maintain tree balance during insertions and deletions. The file is intended to be included in other C programs, providing a reusable and efficient implementation of these tree data structures. It does not define a public API in the traditional sense but offers a set of macros and inline functions that can be used to manipulate these tree structures in a consistent manner.
# Global Variables

---
### spe_right
- **Type**: `struct type*`
- **Description**: The `spe_right` variable is a pointer to a structure of type `type`, representing the right child or element in a splay tree node. It is part of the `SPLAY_ENTRY` macro, which defines the structure of a node in a splay tree.
- **Use**: This variable is used to access or modify the right child of a node in a splay tree.


---
### rbe_right
- **Type**: `struct type *`
- **Description**: The `rbe_right` variable is a pointer to a structure of type `type`, representing the right child node in a red-black tree node structure. It is part of the `RB_ENTRY` macro, which defines the elements of a red-black tree node, including pointers to the left and right children, the parent, and the node color.
- **Use**: This variable is used to navigate and manipulate the right subtree of a node in a red-black tree.


---
### rbe_parent
- **Type**: `struct type*`
- **Description**: The `rbe_parent` variable is a pointer to a `struct type`, representing the parent element in a red-black tree node. It is part of the `RB_ENTRY` macro, which defines the structure of a node in a red-black tree.
- **Use**: This variable is used to maintain the hierarchical relationship between nodes in a red-black tree, allowing traversal and manipulation of the tree structure.


---
### rbe_color
- **Type**: `int`
- **Description**: The `rbe_color` variable is an integer that represents the color of a node in a red-black tree. In red-black trees, nodes are typically colored either red or black to maintain balance during insertions and deletions.
- **Use**: This variable is used to store the color attribute of a node, which is crucial for maintaining the properties of a red-black tree.


---
### struct
- **Type**: `struct`
- **Description**: The `struct` in this context is used as a part of a macro definition for red-black trees, which are a type of self-balancing binary search tree. The macro `name##_RB_MINMAX` is designed to find the minimum or maximum node in a red-black tree, depending on the value of the `val` parameter.
- **Use**: This `struct` is used to define the structure of nodes within a red-black tree, allowing for operations such as finding the minimum or maximum node.


# Functions

---
### _SPLAY_FIND<!-- {{#callable:_SPLAY_FIND}} -->
The `_SPLAY_FIND` function searches for a node in a splay tree and moves it to the root if found.
- **Inputs**:
    - `head`: A pointer to the splay tree structure, which contains the root of the tree.
    - `elm`: A pointer to the node being searched for in the splay tree.
- **Control Flow**:
    - Check if the splay tree is empty using `SPLAY_EMPTY`; if true, return NULL.
    - Call the `name##_SPLAY` function to splay the tree with the given element `elm`, moving it closer to the root.
    - Compare the element `elm` with the current root of the tree using the `cmp` function.
    - If the comparison indicates equality (i.e., `cmp` returns 0), return the root of the tree.
    - If the element is not found, return NULL.
- **Output**:
    - Returns a pointer to the root node if the element is found, otherwise returns NULL.


---
### _SPLAY_NEXT<!-- {{#callable:_SPLAY_NEXT}} -->
The `_SPLAY_NEXT` function finds the next node in a splay tree that is greater than the given node.
- **Inputs**:
    - `head`: A pointer to the splay tree's root structure.
    - `elm`: A pointer to the current node in the splay tree for which the next node is to be found.
- **Control Flow**:
    - The function first calls `name##_SPLAY` to splay the tree with the given node `elm` to bring it to the root or close to it.
    - It checks if the right child of `elm` is not NULL.
    - If the right child exists, it moves to the right child and then iteratively moves to the leftmost child of this subtree to find the next node.
    - If the right child does not exist, it sets `elm` to NULL, indicating there is no next node.
- **Output**:
    - The function returns a pointer to the next node in the splay tree that is greater than the given node, or NULL if no such node exists.


---
### _SPLAY_MIN_MAX<!-- {{#callable:_SPLAY_MIN_MAX}} -->
The `_SPLAY_MIN_MAX` function performs a splay operation to bring either the minimum or maximum element to the root of a splay tree and returns the root of the tree.
- **Inputs**:
    - `head`: A pointer to the splay tree structure, which contains the root of the tree.
    - `val`: An integer value indicating whether to splay with the minimum (-1) or maximum (1) element.
- **Control Flow**:
    - Call the `name##_SPLAY_MINMAX` function with `head` and `val` to perform the splay operation, which rearranges the tree to bring the desired element (minimum or maximum) to the root.
    - Return the root of the tree using the `SPLAY_ROOT` macro.
- **Output**:
    - The function returns a pointer to the root of the splay tree after the splay operation.


---
### _SPLAY_MINMAX<!-- {{#callable:_SPLAY_MINMAX}} -->
The `_SPLAY_MINMAX` function performs a splay operation on a splay tree to bring either the minimum or maximum element to the root based on the comparison value provided.
- **Inputs**:
    - `head`: A pointer to the root of the splay tree structure, which is of type `struct name`.
    - `__comp`: An integer that determines whether to splay the minimum or maximum element to the root; a negative value indicates minimum, and a positive value indicates maximum.
- **Control Flow**:
    - Initialize a temporary node `__node` with both left and right pointers set to NULL.
    - Set `__left` and `__right` pointers to point to `__node`.
    - Enter an infinite loop to traverse the tree based on the comparison value `__comp`.
    - If `__comp` is negative, attempt to move left in the tree; if the left child is NULL, break the loop.
    - If `__comp` is negative and the left child exists, perform a right rotation and check again if the left child is NULL, breaking if so.
    - Link the current root to the right subtree using `SPLAY_LINKLEFT`.
    - If `__comp` is positive, attempt to move right in the tree; if the right child is NULL, break the loop.
    - If `__comp` is positive and the right child exists, perform a left rotation and check again if the right child is NULL, breaking if so.
    - Link the current root to the left subtree using `SPLAY_LINKRIGHT`.
    - After exiting the loop, assemble the tree using `SPLAY_ASSEMBLE` to finalize the splay operation.
- **Output**:
    - The function does not return a value; it modifies the tree in place, bringing either the minimum or maximum element to the root based on the comparison value.


