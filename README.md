## Binary Tree Traversal
These are the primary types of traversal methods for a binary tree:
- Preorder
  - Traverses the middle node first, then the left node, then the right node
- Inorder
 - Traverses the left node, then the middle node, then the right node
- Postorder
 - Traverses the left node, then the right node, then the middle node

More concretely, we can define the following pseudocode processes:
```
function preorder(node, visited)
  if node is NULL, terminate

  visited.insert(node);
  preorder(LEFT_CHILD(node), visited)
  preorder(RIGHT_CHILD(node), visited)
```
```
function inorder(node, visited)
  if node is NULL, terminate

  inorder(LEFT_CHILD(node), visited)
  visited.insert(node)
  inorder(RIGHT_CHILD(node), visited)
```
```
function postorder(node, visited)
  if node is NULL, terminate

  postorder(LEFT_CHILD(node), visited)
  postorder(RIGHT_CHILD(node), visited)
  visited.insert(node)
```
Following these pseudocode processes and understanding when nodes are added to the `visited` list will give you a good idea of how to interpret the `visited` list if you are given just the list itself.

#### Time Complexity
Each of these operations take O(n) time where n is the total number of nodes in the binary tree. Since every node is added to the visited array exactly once, we are able to deduce this time complexity.

## Trie Recap
Tries store strings in a tree structure by storing each letter as a distinct node. Each "level" or "depth" in the trie correspond to an index in strings. Tries can be created as prefix or suffix tries (you have most likely seen the former in CS 61B). You can create a suffix trie by iterating through the input strings in reverse and adding nodes like that. Using one over the other is highly dependent on the given problem.

Each node in the trie has a "stop" value which is true if the node marks the end of an inserted word and false otherwise. This is useful for checking the existence of a string. If we didn't have this stop value, then we would not be able to distinguish between the strings "aab" and "aa". In this example, "aa" might not be a word, but if we did not keep stop values, then the trie will have no way of telling one way or another.

Tries do not necessarily need to represent a dictionary of words. You can even keep a binary trie for the binary strings of integers in an array, for example. When doing bitwise operations, this technique might come in handy.

#### Time Complexity
Inserting a node takes O(s) where s is the length of the string being inserted. Retrieving a string takes O(s) worst case as well. This is not bad considering searching through a list of words takes O(s*n) where n is the length of the list of words since we have to iterate over every word and naively comparing two strings takes O(s) time (i.e. character-wise comparison).

## AVL Trees
AVL Tree is a type of self-balancing binary tree, just like left leaning red black trees (refer to CS 61B course material if you are unfamiliar with this topic!). AVL trees determine whether a rotation should occur on a node based on the **balance factor**. This number is calculated by taking the maximum distance from the right node to a leaf and subtracting that from the maximum distance from the left node to a leaf. More succinctly,
```
balance = max_dist(LEFT_CHILD(node)) - max_dist(RIGHT_CHILD(node))
```
If the balance factor for a node is within +1 and -1, the node is considered balanced. However, if the balance factor is < -1, this means that the right child is "heavier" so we do a left rotation. If the balance factor is > 1, we do a right rotation by symmetry. The concept of rotations here is exactly the same as the concept of rotations with LLRBs.

When inserting a node, we insert the node as a leaf and then move up the tree to make rotations wherever necessary.

See the following demo of inserting into an AVL tree (source: Wikipedia):

![](https://upload.wikimedia.org/wikipedia/commons/f/fd/AVL_Tree_Example.gif)

You can play around with inserting/deleting into an AVL tree [here](https://www.cs.usfca.edu/~galles/visualization/AVLtree.html).

**PRACTICE PROBLEM:** See if you can code an AVL tree which can handle insertions. You might find it useful to dig through your LLRB implementation from CS 61B.

#### Time Complexity
Inserting and deleting from an AVL tree takes approximately O(log n). Since we move along paths in the tree from the given node to the root a constant number of times, we achieve this time complexity. Finding is also O(log n). The fact that AVL trees are self balancing prevent the worst case instances which happen with regular binary trees (where you could possibly get something that looks like a linked list that forces operations to take O(n) time).

## Segment Trees
Segment tree is a type of full binary tree. These trees are particularly useful for range queries in arrays. For example, lets say you want the ability to find the sum in a range of elements in an array. If you were to answer this question using knowledge you learned last week, you would likely suggest computing the prefix sum array and then using that to find range sums. Let's say we modify this problem slightly by allowing modification of the array (i.e. some values in the array can be changed). Whenever we update an element in the array, we will have to recompute the prefix sums array which takes O(n). This is oftentimes too slow, so can we do better?

The answer is yes, we can implement segment trees. These kinds of trees contain information about smaller sub-ranges within the array. For example, if we have an array like the following:
```
[..., a, b, ...]
```
we would have a node in the segment tree representing some information about the range containing `a` and `b`.

More concretely, we assign all the values in the array to be the leaves of the tree. For every adjacent non-overlapping interval of size 2 in the array, we create a new parent node for the two leaves that it represents, and this parent node contains some information about the range which contains its children. We continue doing this with the new level of parents that we created (now treating them as the leaves for the next level) until we finish with the root node. Visually, you can think of this as building a pyramid from the ground up where the base level is the leaves containing information about the individual array elements and the top most node contains the range information of the entire array.

The following image is illustrative of a segment tree for sum within a range (source: Geeks for Geeks):
![](https://media.geeksforgeeks.org/wp-content/cdn-uploads/segment-tree1.png)

Updating a segment tree is done by updating the leaf value which denotes the modified index in the array and then moving up the tree by following the parents of the leaf node all the way to the root.

Querying a segment tree is done by starting at the root and then moving down to the parents. If we encounter a node which represents a range that is within the range we are querying, we can return the value of the node from our query operation. This is efficient since, by starting at the root and going down the tree, we are attempting to choose the largest range possible. If a range of a node is not within the range we are querying, we simply go down one level and then aggregate the results of those two levels. In some senses, this is like an inductive argument, but the query function can be reformulated to be iterative (which is demonstrated in an implementation of segment trees below).

Below is the implementation of segment trees for sum in a range. Note that we are storing the tree in an array where each index `i` represents a node and its children are at indices `2*i` (for the left child) and `2*i + 1` (for the right child).
```
#include <bits/stdc++.h>

using namespace std;

class SegmentTree {
    public:
    SegmentTree(vector<int> array) {
        N = array.size();

        // if there are N leaves, then there are 2*N total nodes
        tree.resize(2*N);

        // fill in the leaves to be the array values
        for (int i = N; i < 2*N; ++i) {
            tree[i] = array[i - N];
        }

        // fill in the other non-leaf nodes based on info from the leaves
        for (int i = N - 1; i >= 1; --i) {
            tree[i] = tree[2*i] + tree[2*i + 1];
        }
    }

    int query(int a, int b) {
        // add N to both values since the tree has offset the original
        // array values by N
        a += N;
        b += N;
        int sum = 0;

        // iterate until the range is no longer valid
        while (a <= b) {

            // if the lower bound is a right leaf, then add the value to the sum since
            // its parent node will not be part of the sum
            if (a % 2 == 1) {
                sum += tree[a];
                a++;
            }

            // if the lower bound is a left leaf, then add the value to the sum since
            // its parent node will not be part of the sum
            if (b % 2 == 0) {
                sum += tree[b];
                b--;
            }

            // move one level up in the tree
            a /= 2;
            b /= 2;
        }
        return sum;
    }

    void update(int i, int x) {
        // add N since the tree has offset the original array values by N
        i += N;

        // update the value
        tree[i] = x;

        // move one level up
        i /= 2;

        // traverse up to the root and update the values
        for (i; i >= 1; i /= 2) {
            tree[i] = tree[2*i] + tree[2*i + 1];
        }
    }

    private:
    vector<int> tree;
    int N;
};


int main() {
    vector<int> test {1, 2, 3, 4};
    SegmentTree st(test);
    cout << st.query(1, 2) << endl;
    return 0;
}
```

**PRACTICE PROBLEM:** The segment tree code above represents a *range sum query*. See if you can implement a *range* ***min*** *query* using segment trees.

#### Time Complexity
Updating a segment tree takes O(log n) (where n is the number of elements in the array) since we move up a path to the root which has length O(log 2n) = O(log n) (remember that there are a total of approximately 2n nodes if there are n leaves in a tree). Creating the segment tree takes O(n), and querying the segment tree takes O(log n). The proof of the second claim requires some involved casework which we will omit for the sake of brevity.

## Practice Problems
#### Easy
- See the practice problem from the AVL section
- [Lowest Common Ancestor of a Binary Search Tree](https://leetcode.com/problems/lowest-common-ancestor-of-a-binary-search-tree/)

#### Medium
- See the practice problem from the Segment Trees section
- [Validate Binary Search Tree](https://leetcode.com/problems/validate-binary-search-tree/)
- [Construct Binary Tree from Preorder and Inorder Traversal](https://leetcode.com/problems/construct-binary-tree-from-preorder-and-inorder-traversal/)
- [Implement Trie Prefix Tree](https://leetcode.com/problems/implement-trie-prefix-tree/)
- [Knight Tournament](https://codeforces.com/contest/356/problem/A)
 - The first non-LeetCode problem! This problem is from Codeforces and requires that you create an account before you submit. Note that for Codeforces, time limit is strictly enforced so using a slower language like Python is not advisable. I would recommend that you use either C++ or Java.

#### Hard
- [Word Search II](https://leetcode.com/problems/word-search-ii/)
  - Note that this problem requires some knowledge of graph search algorithms whicih were covered in CS 61B. It might be beneficial to review that material if you are a little shaky.
- [Sereja and Brackets](https://codeforces.com/contest/380/problem/C)
  - Another Codeforces problem! This is likely the hardest question I have given so far, but I am confident in your abilities to attempt this. Please ask on Slack if you have any questions or need a hint.
