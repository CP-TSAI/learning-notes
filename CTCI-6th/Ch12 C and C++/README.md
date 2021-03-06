### Ch12 C/C++

- keyword: inheritance, polymorphism, (pure) virtual function, vptr/vtable, based/derived class, 

---

- Class and Inheritance

```
include <iostream>
using namespace std;

#define NAMESIZE 50//Defines a macro

class Person {
	int id; // all members are private by default 
	char name[NAME_SIZE];

public:
	void aboutMe() { cout « "I am a person."; }
};

class Student : public Person { 
public:
	void aboutMe() {cout << "I am a student. "}
};

int main() {
	Student * p = new Student();
	p->aboutMe(); // prints "I am a student."
	delete p; // Important! Make sure to delete allocated memory.
	return 0;
}
```

---

- Virtual Functions

Q: what if we change the above code to the following, what's the result?
```
Person* p = new Student();
p->aboutMe();
```

A: The result is **I am a person.**  
(1) Because the function is resolved in *compile-time*, which is also called *static binding*.  
(2) If you want to call student's **aboutMe()**, you need to use **virtual function**.  
```
class Person {
	...
public:
	virtual void aboutMe() { cout « "I am a person."; }
};

```

**注：將function定義為virtual, 就是將定義留給繼承他的class決定**

Q: what if we don't want to implement the class in parent class?  
A: Make the parent class as a **abstract class**, and define the function as a **pure virtual function**.  
```
class Person {
	...
public:
	virtual void aboutMe() { cout « "I am a person."; }
	virtual bool addCourse(string s) = 0;
};

class Student : public Person { 
public:
	void aboutMe() {cout << "I am a student. "}
	bool addCourse(string s) {
		cout << "addCourse " << s << endl;
		return true;
	}

};

int main() {
	Person * p = new Student();
	p->aboutMe(); // prints "I am a student."
	p->addCourse("history");
	delete p;
}
```
**注：將function定義為pure virtual function, 就是在後面加"= 0", 並使之成為 **abstract class** 之後就無法利用該parent class 來建立object了**

---

- Virtual Destructor

Q: Let's take a look at the naive solution, what's the problem?

```
class Person {
public:
	~Person() { cout << "Deleting a person" << endl; }
}

class Student : public Person {
public:
	~Student() {cout << "Deleting a student" << endl;}
}

int main() {
	Person* p = new Student();
	delete p; // print "Deleting a person"
}
```

A:  
(1) Since p is a **Person**, the destructor of **Person** is called.  But the memory for **Student** is not cleaned up.  
(2) To fix it, the destructor of **Person** needs to be **virtual**.

```
class Person {
public:
	virtual ~Person() { cout << "Deleting a person" << endl; }
}

class Student : public Person {
public:
	~Student() {cout << "Deleting a student" << endl;}
}

int main() {
	Person* p = new Student();
	delete p;
}

the result would be:  
Deleting a student  
Deleting a person
```

---

- Pointers and References

Q: What's the difference between them?  
A:  
(1) **Pointer**: holds the address of a variable, and can be used to perform any operation that can be directly done on the variable, ex: access and modify it. The size of pointer is 32bits/64bits (depending on the machine).  
(2) **Reference**: an alias for a pre-existing object, and it does not have memory of its own.  
(3) Pointer can be null, reference can't.  
(4) Pointer can be re-assigned, reference can't.  

---

- Questions


(2) Implement a function to reverse a string  
```
#include <iostream>
using namespace std;

string reverse_str(string str){
    int head = 0;
    int end = str.size() - 1;
    while(head < end){
        char tmp = str[end];
        str[end] = str[head];
        str[head] = tmp;
        head++;
        end--;
    }
    return str;
}


int main(){
    string input = "abcd";
    string rev_input = reverse_str(input);
    cout << "rev_input = " << rev_input << endl;     
    return 0;
}
```


(3) Compare **hash-table** and **STL map**. If the data is small, which data structure can be used instead of hash-table.  
 - **Hash-table**:  
 - key-> idx = hash-function(key) -> linked-list[idx] = value  
 - insert and look up is O(1)  
 - Need to handle **collision** by **chaining** (creating a linked-list of all values whose keys map to a particular index).  
 - In short, the element in the linked-list need to hold: {the original key, value}. A good hash-function is needed to ensure the keys are well-distributed. The collision slows the look-up process.  
 - **STL map**:  
 - the {key, value} pair is inserted to a BST based on the keys.  
 - No need to handle collision  
 - Balanced tree, so the insert and lookup is O(logN).  
 - **Data is small**: if it's the case, then STL map (time: O(logN)) is a good choice, since N is small.  


(4) **How do virtual function works in C++?**  
 - vtable (used to resolve the address of the function when virtual function is called.)
 - it stores the address of the virtual functions of the class.  
 - a **vptr** (added by compiler) points to the vtable if the class. 
 - If a virtual function is not overriden in the derived class, the vtable of the derived class stores the address of the function in its parent class.  
 - Dynamic binding in C++ is performed through the vtable mechanism.  
 - When the derived class object is assigned to the base class pointer (ex: **Person* p = new Student();**), the vptr variable points to the vtable of the derived class.  
 - for example:
```
class Shape {
public:
	int edge_length;
	virtual int circumference {
		cout << "circumference of base class" << endl;
		return 0;
	}
};

class Triangle: public Shape {
public:
	int circumference() {
		cout << "Curcumference of Triangle Class" << endl;
		return 3 * edge_length;
	}
};

void main() {
	Shape* x = new Shape();
	x->circumference(); // "circumference of base class"
	Shape* y = new Triangle();
	y->circumference(); // "Curcumference of Triangle Class"
}
```
 - C++ non-virtual function calls are resolved at **compile time** with **static binding**, while virtual function calls are resolved at **run time** with **dynamic binding**.  


(5) What's the difference between **shallow copy** and **deep copy**?  
 - shallow copy: copy all member values from one object to another.  
 - deep copy: in addition to copy all member values, it also copies any **pointer objects**.  
 - shallow copy may cause runtime errors, especially with the **creation and deletion of objects**.  
 - We should rarely use shallow copy. 
```
struct Test {
	char* ptr;
};

void shallow_copy(Test& src, Test& dest) {
	dest.ptr = src.ptr;
}

void deep_copy(Test& src, Test& dest) {
	dest.ptr = (char*)malloc(strlen(stc.ptr) + 1);
	strcpy(dest.ptr, src.ptr);
}

```

(7) Why does a **destructor in base class** need to be declared **virtual**?  
 - See the tutorial above. 
 - In short, we want to make sure that the destructor for the most derived class is called. 





