# CS246 - Lecture 12 - Oct 18, 2018

### Recall

```C++
int x, y;
...
Vec operator + (const Vec &other) {
    return {x + otherx, y + other.y};
}

Vec operator * (const int k) {
    return {x * k, y * k}; //Note: implements v * k
}
```
`k * v`: must be non-member operator:

```C++
Vec operator * (const int k, const Vec &v) { return v * k;}
```

## I/O operators:
```C++
struct vec {
    ...
    ostream &operator << (ostream &out) {
        return out << x << ' ' << y;
    }
};
```
## What's wrong with this? Makes the Vec the first operand, not the second

-> Use as  `v << cout;` - Confusing

So define operator `<<`, `>>` as non-members

Certain operator **must** be members: 
- operator =
- operator []
- operator ->
- operator ()
- operator T (where T is a type)

## Arrays of Objects:

```C++
struct Vec {
    int x, y;
    Vec (int x, int y): x{x}, y{y} {}
};

Vec *vp = new Vec[10]; //Error - These want to call the default constructor on each item
Vec moveVecs [10]; //If no default constructor - can't initialize the items - error
```

### Options:
1. Provide a default ctor.
2. Stack arrays:
```C++
Vec moreVecs [3] = {{0, 0}, {1, 1}, {2, 4}};
// Heap arrays of know size,
Vec * vp = new Vec [3] {{0, 0}, {1, 1}, {2, 4}};
```
3. Heap arrays of dynamic size: create an array of ptrs.
```C++
Vec **vp = new Vec* [5];
vp [0] = new Vec {0, 0};
vp [1] = new Vec {0, 0};
//etc.

for (int i = 0; i < 5; ++i) delete vp[i];
delete [] vp;
```

## Const Objects

```C++
int f (const Node &n) {...}
```
> Const objects arise often, especially as params.

What is a const object? - can't change the fields.

Can we call methods on a const obj?

**Issue:** the method might modify fields, violate `const`

**Answer:** Yes, we can call methods that promise not to modify fields

**Eg.**
```C++
struct Student {
    int assns, mt, final;
    float grade() const; // doesn't modify fields
};

Compilers checks that const methods don't modify fields

Only `const` methods can be called on const objects.
```

### Now consider:
Want to collect usage stats on `Student` objects:

```C++
struct Student {
    ...
    int numCall = 0;
    float grade () const { //now can't call grade on const Students
        ++numCalls; // But mutating numCalls affects only the physical constness of Student objects, not the logical constness
        return ------;
    }
};
```

Want to be able to update `numCalls`, even if the object is const
declare the field **mutable**.

```C++
struct Student {
    ...
    mutable int numCalls = 0; //can be changed, even if the object
    float grade () const {
        ++numCalls;
        return ------;
    }
};
```
## Static Fields + Methods

`numCalls` tracked # times a method was called on a **particular** object.

What if we want to track # times a method is called over **all** students?

Or track how many students are created?

**Static members** - associated with the class itself, not with any particular object.

```C++
struct Student {
    ...
    static int numStudents;
    student (-----); ----- {
        ++numStudents;
    }
};

int Student::numStudents = 0; // in .cc file
```
Static fields must be defined external to the class.

### Static member functions:
- don't depend on a specific instance (no **this** param)
- can only call access static fields + call other static member functions

```C++
struct Student {
    ...
    static int numStudents;
    ...
    static void howMany () {
        cout << numsStudents;
    }

};

Student billy{60, 70, 80}, jane{70, 80, 90};
Student::howMany(); // 2
```

## Invariants and Encapsulation

### Consider
```C++
struct Node {
    int data;
    Node *next;
    ~Node () {delete next;}
};

Node n1 {1, new Node {2, nullptr}};
Node n2 {3, nullptr};
Node n3 {4, &n2};
```

### What happens when these go out of scope?
- `n1` - dtor runs, entire list is deleted. Ok.
- `n3` - dtor tries to delete `&n2`, but `n2` is on the stack, not the heap! Undefined behavior!

Node relies on an assumption for its proper operation - that next is either `nullptr`, or was allocated by new.

- This is an **invarient** - statement that must hold true - that Node relies on

But we can't guarentee this invarient - can't trust the user to use Node properly

**Eg.** stack - invarient - last item popped is the first item popped.
- But not if the client can rearrange the underlying data.
Hard to reason about programs if you can't rely on invarients.

## To enforce invarients - encapsulation
- treat objects as black boxes - capsules
    - seal away implementation           - abstraction
    - only interact via provided methods - abstraction
- regain control

**Eg.**
```C++
struct Vec {
    Vec (int x, int y); // also public - default visibility = public.
    private:
        int x, y; // can't be accessed outside struct Vec
    
    public:
        Vec operator + (const Vec &other); // anyone can see
        ...
};
```
**In general**: want private fields; only methods should be public.

Better to have default visibility = private.

Switch from `struct` to `class`.

```C++
class Vec {
    int x, y;

    public:
        Vec (int x, int y);
        Vec operator + (const vec &other);

};
```

Difference is **default visibility**
- public in `struct`, private in class

Let's fix our linked lists.

```C++
// list.h 
class List {
    struct Node; // private nested class. - only accessible inside class List
    Node * theList = nullptr;

    public:

}
//will continue next class
```







