# CS246 - Lecture 15 - Oct 30, 2018



![UML Diagram3](Images/cs246_Oct25_UML_Modeling_Diagram3.jpg "UML \"is a\" relationship")

Now consider the method `isHeavy` - When is book heavy?
- for ordinary books > 200 pages
- for Texts > 500 pages
- for Comics > 30 pages

```C++
class Book {
  ...
  public:
    bool isHeavy () const {return length > 200;}
};

class Text: public Book { // comic: similiar
    ...
  public:
    bool isHeavy() const {return length > 500;}
    ...
};

Book b {_______,______, 50};
Comic c {______,______, 40, _______};

cout << b.isHeavy(); // false
cout << c.isHeavy(); //true

```

```C++
//Since, inheritance means "is a" we can do this.

Book b = Comic {______, ______, 40, ______}; 
```

**Q:** Is `b`heavy? i.e. `b` `isHeavy - true or false 

**A:** No - `b` is not heavy Book::isHeavy runs

**Why?**
insert image
```C++
b=c;
```
- `Book b = Comic {...} tries to fit a Comic obj, where there is only space for a Book obj.  
What happens? - Comic is **sliced**
- hero field chopped off - comic coerced into a Book

So `Book b = Comic {...};` creates a Book and `Book::isHeavy` runs.

When accessing objs - through ptrs, slicing is unnecessary and does not happen.

```C++
Comic c = {____,____, 40, _______};
Book *pb = &c;
Comic *pc = &c;
pc->isHeavy(); //true
pb->isHeavy(); //false

```

... and still, `Book::isHeavy` runs when we access `pb->isHeavy()`
Computer uses the type of ptr (or reference) to decide which `isHeavy` to run 
- doesn't consider the actual type of the object.

Same object behaves differently, depening on what type of ptr accesses it.

So a Comic is only a Comic when a Comic ptr (or ref) ptrs to it???

Can we make Comic act like a Comic, even when pointed to by the Book ptr?

Declare the method **virtual**:

```C++
class Book {
  ...
  protected:
    int length;

  public:
    virtual bool isHeavy() const {return length > 200;}

};

class Comic : public Book {
  ...
  public:
    bool isHeavy() const override {return length > 30;}
};
```
```C++
Comic c = {____,____, 40, _______};
Book  *pb = &c;
Book  &pb = c;
Comic *pc = &c;
pc->isHeavy(); //true
pb->isHeavy(); //true
rb.isHeavy(); //true  // Comic::isHeavy in each case.

```

Virtual methods - choose which class method to run, based on the actual type of the object at runtime.

**Eg.** my book collection.

```C++
Book *myBooks [20];

...

for (int i = 0; i < 20; ++i) {
  cout << myBooks[i]->isHeavy() << endl;
}
```
Notes: 
- Uses `Book::isHeavy` for Books.
- Uses `Text::isHeavy` for Texts
- Uses `Comic::isHeavy` for Comics

## Polymorphism

Accommodating multiple types under one abstraction: **polymorphism**  ("many forms")

**Danger:** Consider

```C++
class A {
  int x,y;
  ...
};

class B:public A {
  int z;
  ...
};

void f (A *a) {
  a[0] = A{6,7};
  a[1] = A{8,9};
}

B myArray[2] = {{1, 2, 3}, {4, 5, 6}};
f(myArray);
```

insert image

Data is misaligned.

**Never** use arrays of objs. polymorphically. Use arrays of pointers.

## Destructor Revisited 

```C++
class x {
  int *a;
  X(int n): a {new int [n]}{}
  ~X() {delete [] a;} 
}

class Y:public X {
  int *b;
  public:
    Y(int n, (n + m) : X{n}, b {new int [m];}, {}
    ~Y() {delete [] b;} 
}
```
X `*myx = new Y{10, 20}
delete myX; // leaked! WHY?`
- So only a, but not b, was freed.

How can we ensure that deletion through superclass ptr will call the subclass dtor? Make the dtor virtual:

```C++
class X {
  ...
  public:
    virtual ~X() {delete [] a;}
};
```

**Always:**
 Make the dtor `virtual` in classes that are meant to have subclasses.
 - even if the dtor doesn't do anything



