# CS246 - Lecture 13 - Oct 23, 2018

### Continued from last class (node invarient/encapsulation)

> list.h
```C++
#ifndef _LIST_H_
#define _LIST_H_

class List {
  struct Node; // private nested class/struct only accessible within class List
  Node *theList = nullptr;

 public:
  void addToFront(int n);
  int &ith(int i); // is a reference so client can mutate item
                   // ex. List l; l.ith(5) = 4;
  ~List();
};

#endif
```
> list.cc
```C++
#include "list.h"

struct List::Node { // Nested class
  int data;
  Node *next;

  Node (int data, Node *next): data{data}, next{next} {}
  ~Node() { delete next; }
};

List::~List() { delete theList; }

void List::addToFront(int n) { theList = new Node(n, theList); }

int &List::ith(int i) {
  Node *cur = theList;
  for (int j = 0; j < i && cur; ++j, cur = cur -> next);
  return cur->data;
}
```
Only List can create/manipulate node objects \
Therefore, we **can** guarentee the invarient that next is always nullptr or allocated by new

- But now we can't traverse the list from node to node as we would a linked list since ith() is O(n), we can't traverse the List in O(n) because we are repeatebly calling ith over and over which gives O(n^2) time.
-  We can expose the nodes but then we lose encapsulation

## SE topic: Design Patterns
- certain programming problems arise often
- keep track of good solutions to these problems - reuse + adapt them

### Soln: Iterator Pattern
- Create a class that manages access to nodes
    - Abstraction of a pointer
    - Can use to walk the list without exposing the external pointers
  
### Aside:
```C++
for (int *p = curr; p != arr + size; p++) {
    printf("%d\n, *p); // traversing an array using pointer arithmetic
}
```

```C++
Class List {
    struct node;
    Node *theList = nullptr;

    public:
        class Iterator {
            Node *p;

            public:
                explicit Iterator (Node *p) : p{p} {}
                int &operator *() {return p->data;}
                Iterator & operator ++ () {p = p->next; return *this;}

        }
}
```

