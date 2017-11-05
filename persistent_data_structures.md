# Persistent Data Structures

https://www.youtube.com/watch?v=T0yzrZL1py0

## Models of computation

These matter, why? What are they?

## Pointer Machine

Nodes with fields (constant number, so O(1) fields), each field can be:
* a pointer to another node
* a null pointer
* some data

------    -----
|    | -> |   |
------    -----
| 5  |
------
| 7  |
------

Operations:

* Create nodes (x = new node)
* Look at fields (x = y.field)
* Set fields (x.field = y)
* Compute on the contained data (x = y.field + z.field)
* Destroy nodes if you feel like it

There is going to be one node in this structure, the 'root node'. It
has a constant number of fields like any other node. In our operations
above, x & y are fields on the root node. You're always working
relative to the root node.


Lots of data structures fall into this category, e.g.
* Balanced Binary Search Tree
* Linked List


## Temporal Data Structure Models

* Persistence - branching universe model of time travel, where if you
  make a change in the past you get a new universe. You never destroy
  old universes.

* Retroactivity - model where you can go back, make a change, return
  to the future and see the result of that change.

## Persistence

The general goal is to remember everything (the versions of the DS).

All DS operations are relative to a specified version, when you
perform an operation you must specify a version.

### 4 Levels of persistence

1. Partial Persistence
* Only allowed to update the latest version
* So the versions are *linearly ordered*
* Easiest to achieve

|-|-|-|-|-|-|-|-|-|->

This allows us to query old versions, but we can't change them.


2. Full Persistence
* You can update any version
* The versions form a *tree*
* Versions don't know about other versions
* No way to merge versions together :-(

  -|-|->
 /
|-|-|->
     \
      -|->


3. Confluent Persistence
* Like full persistence, but you *can* combine two versions to create
  a new version
* Versions for a DAG (Directed Acyclic Graph)
* This is hard, and mostly an open problem

  -|-|-|-|-|->
 /           \
|-|-|->       -|-|->
     \        /
      -|-|-|->


4. Functional Data Structures
* You can't modify nodes, *ever*, all you can do is make new nodes
* You get all previous 3 levels for free because of these constraints


### Partial Persistence

Any pointer machine data structure with a constant number of pointers
into any node ( <= O(1) / constant indegree). In a pointer machine you
always have a constant number of pointers out of a node, but for
partial persistence we have this additoinal constraint.

Any data structure in this world ^ can be made partially persistent
with:
  O(1) amortized factor overhead
+ O(1) space / change

#### Proof

P is the bound on the indegree of a node.

1. Store back pointers (for latest version of DS)
This is why we need the constant indegree constraint. So whenever we
store a pointer to a node, we want that node to have a back pointer so
that we know where all the pointers came from. We need P back pointers
(so still constant).

2. Store modifications to fields of DS
Mods are (version, field, value). So we don't actually change the
fields and their values, we leave them and store mods on the node.
We need 2P mods stored on a node (constant).

So does this give us constant amortized overhead?

Reading a field:
Given some version v, what is the value at a given field?
* Constant time to look through mods with version <= v, did this field
change? Look at the latest one. So constant time.

Changing a field:
To set node.field = x:
* If node is not full (space left in mods space), add mod (now, field, x)

* If node is full, we need to make a new node, node' with all mods
  including our new mod *applied* (and back pointers intact). The mods
  will be empty on node'. What about the outdated pointers and back
  pointers. Conveniently, the back pointers are easy:
   * Update back pointers to node to node'
   * Update pointers recursively


Analysing this algorithm using the potential method:
This is using the CLRS notion of amortized cost which is actual cost +
change in potential.

c * sum #mods in latest version
=> amortized cost <= c (time) + c (+1 mod) + [-2cp + p * recursion]?
= O(1)


### Full Persistence

Versions are no longer numbers, they're nodes in a tree. This is
annoying because trees are harder than lines. So, we linearize the
tree of versions with tree traversal. When doing so, we want to
maintain the begin & end of each subtree of versions.

UPTO: 44:37





# Notes from Purely Functional Data Structures

## Why?
Why is writing performant data structures different in a functional
language as opposed to an imperative one?

1. We don't allow ourselves to perform destructive updates,
   e.g. assignments.

2. We actually expect more flexibility from functional data
   structures, in particular we expect a 'history of versions'.

Because FP demands immutability, all data structure are inherently
persistent.

## Strict vs. Lazy
By which we mean: whether the arguments to a function are evaluated
before the body of the function is evaluated, or on-demand.

One advantage of strict evaluation is that it's typically easier to
reason about asymptotic complexity, as it's often syntactically
obvious what the evaluation order will be.

Design & Analysis: Strict languages can describe worst-case scenarios
but not amortized, while lazy languages can describe amortized but not
worst-case.

So we need a language that supports both!


## Persistence

A distinctive property of functional data structures is that they're
always persistent.


### Lists

The core functions supported by lists are essentially those of the
stack abstraction: https://www.safaribooksonline.com/library/view/purely-functional-data/9780521631242/images/P008.jpg

```
type Stack a

val empty   : Stack a
val isEmpty : Stack a -> Bool
val cons    : a -> Stack a -> Stack a
val head    : Stack a -> a
val tail    : Stack a -> Stack a
```

Another common function we might consider adding to this signature is
`++` which catenates (i.e., appends) two lists. In an imperative
setting, this function can be easily supported in O(1) time by
maintaining pointers to both the first & last cell in each list. `++`
then simply modifies the last cell of the first list to point to the
first cell of the second list.


# Resources

https://www.safaribooksonline.com/library/view/purely-functional-data/9780521631242/xhtml/07_ch02.html#ch02

https://www.infoq.com/presentations/Value-Identity-State-Rich-Hickey

https://ocw.mit.edu/courses/electrical-engineering-and-computer-science/6-854j-advanced-algorithms-fall-2005/lecture-notes/persistent.pdf

http://www.cs.cmu.edu/~sleator/papers/Persistence.htm


Definitions:

Persistent: supports multiple versions

Ephemeral: supports only a single version at a time


