# Purpose
The provided code is a C header file that defines data structures and operations for two types of binary trees: splay trees and red-black trees. These trees are used for efficient data storage and retrieval, with each type offering distinct characteristics and performance benefits. The file includes macros and inline functions to facilitate the creation, manipulation, and traversal of these trees, making it a versatile tool for developers who need to implement self-balancing tree structures in their applications.

Splay trees are self-organizing structures that move accessed nodes to the root, optimizing for access patterns with locality. The code provides macros for defining splay tree nodes and operations such as insertion, removal, and finding nodes. Red-black trees, on the other hand, are balanced binary search trees that maintain balance through node coloring and rotations, ensuring operations are performed in logarithmic time. The header file includes macros for defining red-black tree nodes and functions for insertion, removal, and traversal. This file is intended to be included in other C programs, providing a robust and reusable implementation of these tree structures without exposing the underlying complexity to the end user.
# Global Variables

---
### spe_right
- **Type**: `struct type *`
- **Description**: The `spe_right` variable is a pointer to a structure of type `type`, representing the right child or element in a splay tree node. It is part of the `SPLAY_ENTRY` macro, which defines the structure of a node in a splay tree, including pointers to both left and right children.
- **Use**: This variable is used to navigate and manipulate the right subtree of a node within a splay tree data structure.


---
### rbe_right
- **Type**: `struct type *`
- **Description**: The `rbe_right` variable is a pointer to a structure of type `type`, representing the right child node in a red-black tree node structure. Red-black trees are a type of self-balancing binary search tree, and each node in the tree contains pointers to its left and right children, as well as its parent.
- **Use**: This variable is used to navigate and manipulate the right subtree of a node within a red-black tree.


---
### rbe_parent
- **Type**: `struct type*`
- **Description**: The `rbe_parent` variable is a pointer to a structure of type `type`, representing the parent element in a red-black tree node. It is part of the `RB_ENTRY` macro, which defines the structure of a node in a red-black tree, including pointers to the left and right children, the parent, and the color of the node.
- **Use**: This variable is used to maintain the hierarchical relationship between nodes in a red-black tree, allowing traversal and manipulation of the tree structure.


---
### rbe_color
- **Type**: `int`
- **Description**: The `rbe_color` variable is an integer that represents the color of a node in a red-black tree. Red-black trees are a type of self-balancing binary search tree where each node has an additional color attribute, which can be either red or black. This color attribute helps maintain the balance of the tree during insertions and deletions.
- **Use**: The `rbe_color` variable is used to store and manage the color of nodes in a red-black tree, ensuring the tree remains balanced according to red-black tree properties.


---
### struct
- **Type**: `macro`
- **Description**: The `struct` macro in this context is part of a macro definition for red-black tree operations. It is used to define a function that finds the minimum or maximum node in a red-black tree, depending on the value of the `val` parameter. The macro expands to a function prototype that takes a pointer to a red-black tree head and an integer, and returns a pointer to the node of the specified type.
- **Use**: This macro is used to generate a function that retrieves the minimum or maximum node from a red-black tree based on the provided integer value.


# Functions

---
### _SPLAY_FIND<!-- {{#callable:_SPLAY_FIND}} -->
The `_SPLAY_FIND` function searches for a node in a splay tree and moves it to the root if found.
- **Inputs**:
    - `head`: A pointer to the splay tree structure, which contains the root of the tree.
    - `elm`: A pointer to the node being searched for in the splay tree.
- **Control Flow**:
    - Check if the tree is empty using `SPLAY_EMPTY`; if true, return NULL.
    - Call the `name##_SPLAY` function to splay the tree with `elm` as the target, moving it close to the root.
    - Compare `elm` with the root of the tree using the `cmp` function.
    - If the comparison returns 0, indicating a match, return the root node.
    - If no match is found, return NULL.
- **Output**:
    - Returns a pointer to the root node if `elm` is found and matches the root after splaying, otherwise returns NULL.


---
### _SPLAY_NEXT<!-- {{#callable:_SPLAY_NEXT}} -->
The `_SPLAY_NEXT` function finds the next node in a splay tree that is greater than the given node.
- **Inputs**:
    - `head`: A pointer to the root of the splay tree structure.
    - `elm`: A pointer to the current node in the splay tree for which the next node is to be found.
- **Control Flow**:
    - The function first calls `name##_SPLAY(head, elm)` to splay the tree with `elm` as the root, bringing it to the top of the tree.
    - It checks if the right child of `elm` is not NULL, indicating there is a node greater than `elm`.
    - If the right child exists, it moves to the right child and then traverses left down the tree to find the smallest node greater than `elm`.
    - If the right child does not exist, it sets `elm` to NULL, indicating there is no next node.
- **Output**:
    - The function returns a pointer to the next node in the splay tree that is greater than the given node, or NULL if no such node exists.


---
### _SPLAY_MIN_MAX<!-- {{#callable:_SPLAY_MIN_MAX}} -->
The `_SPLAY_MIN_MAX` function performs a splay operation to bring either the minimum or maximum element of a splay tree to the root and returns the root of the tree.
- **Inputs**:
    - `head`: A pointer to the splay tree structure, which contains the root of the tree.
    - `val`: An integer value indicating whether to splay the minimum or maximum element to the root; typically, a negative value for minimum and a positive value for maximum.
- **Control Flow**:
    - The function calls `name##_SPLAY_MINMAX(head, val)` to perform the splay operation, which rearranges the tree so that the minimum or maximum element is at the root, depending on the value of `val`.
    - After the splay operation, the function returns the root of the tree using `SPLAY_ROOT(head)`.
- **Output**:
    - The function returns a pointer to the root of the splay tree after the splay operation, which is either the minimum or maximum element, depending on the input `val`.


---
### _SPLAY_MINMAX<!-- {{#callable:_SPLAY_MINMAX}} -->
The `_SPLAY_MINMAX` function performs a splay operation on a splay tree to bring either the minimum or maximum element to the root, based on the comparison value provided.
- **Inputs**:
    - `head`: A pointer to the root of the splay tree structure, which is of type `struct name`.
    - `__comp`: An integer that determines whether to splay the minimum or maximum element to the root; a negative value indicates minimum, and a positive value indicates maximum.
- **Control Flow**:
    - Initialize a temporary node `__node` with both left and right pointers set to NULL.
    - Set `__left` and `__right` pointers to point to `__node`.
    - Enter an infinite loop to traverse the tree.
    - If `__comp` is negative, attempt to splay the left subtree:
    - - Retrieve the left child of the root and assign it to `__tmp`.
    - - If `__tmp` is NULL, break the loop.
    - - If `__comp` is still negative, perform a right rotation on the tree and check if the left child of the root is NULL; if so, break the loop.
    - - Link the current root to the right of `__right`.
    - If `__comp` is positive, attempt to splay the right subtree:
    - - Retrieve the right child of the root and assign it to `__tmp`.
    - - If `__tmp` is NULL, break the loop.
    - - If `__comp` is still positive, perform a left rotation on the tree and check if the right child of the root is NULL; if so, break the loop.
    - - Link the current root to the left of `__left`.
    - After exiting the loop, assemble the tree by linking the left and right subtrees to the root.
- **Output**:
    - The function does not return a value; it modifies the tree in place, bringing either the minimum or maximum element to the root.


# Function Declarations (Public API)

---
### _SPLAY_MINMAX<!-- {{#callable_declaration:_SPLAY_MINMAX}} -->
Reorganizes a splay tree to bring the minimum or maximum element to the root.
- **Description**: Use this function to adjust the structure of a splay tree such that either the minimum or maximum element is moved to the root, depending on the comparison value provided. This operation is useful for efficiently accessing the smallest or largest element in the tree. The function should be called on a non-empty tree, and the tree must be properly initialized before use. The function does not handle invalid input values for the comparison parameter, so ensure the comparison value is either negative or positive to indicate the desired operation.
- **Inputs**:
    - `head`: A pointer to the splay tree structure. Must not be null and should point to a valid, initialized splay tree.
    - `__comp`: An integer indicating the operation: a negative value to bring the minimum element to the root, or a positive value to bring the maximum element to the root. Invalid values are not handled.
- **Output**: None
- **See also**: [`_SPLAY_MINMAX`](#_SPLAY_MINMAX)  (Implementation)


