---
layout: post
comments: false
title:  "Pointers for GPGPU"
excerpt: "Refresher on Pointers."
date:   2022-02-26 21:00:00

---

## Summary

This post contains a detailed discussion on the following points:

- CPUs are latency oriented while GPUs are throughput oriented devices.
- Linear layout of bytes in RAM.
- There's a unique address associated with each byte in RAM.
- Pointer stores the address of an object.
- Pointer is necessary to declare a dynamic array.

## Introduction

The graphics processing unit, or GPU, has become one of the most important types of computing technology. Although they’re best known for their capabilities in gaming or mining crypto-currencies, GPUs are becoming more popular for use in Creative Production, Artificial Intelligence (AI) and Scientific Computing. Broadly speaking, GPUs are known to have large number of processing units but there are a few other things that distinguish a GPU from a CPU. Understanding the strengths and weaknesses of GPUs is crucial in leveraging their power and *an efficient high performance computer program must make use of both CPU and GPU*.

Here are a few points of difference between a CPU and a GPU:

- CPUs have powerful ALUs to reduce latency, while GPUs have efficient but large number of ALUs to maximize throughput.
- CPUs use large caches to store data close to the ALU, while GPUs don't use caches to store data but rather a staging area for large threads.
- CPUs have sophisticated control, while GPUs have simple control.

<div class="imgcap">
<img src="https://raw.githubusercontent.com/tgautam03/tgautam03.github.io/master/assets/GPGPU_SERIES/00_Pointers/Figure_1.gif" alt="this slowpoke moves"  width="800"/>
<div class="thecap">Figure 1: Differences between CPU and GPU. </div>
</div>

<!--
<div class="imgcap">
<img src="/assets/GPGPU_SERIES/00_Pointers/Figure_5.png">
<div class="thecap">Figure 1: Differences between CPU and GPU. </div>
</div> -->

**General-purpose computing on graphics processing units** (**GPGPU**) is the use of a [graphics processing unit](https://en.wikipedia.org/wiki/Graphics_processing_unit) (GPU), which typically handles computation only for [computer graphics](https://en.wikipedia.org/wiki/Computer_graphics), to perform computation in applications traditionally handled by the [central processing unit](https://en.wikipedia.org/wiki/Central_processing_unit) (CPU).

Before studying GPUs, it's important to know how CPU perform computations.

## Memory (RAM) in modern computers

Arrays are the most frequently used data structures when it comes to High Performance Computing. Whenever an array is declared, it's elements are stored in the system RAM in a specific manner. Before diving into arrays, it's important to understand how memory systems work in a modern computer.

RAM can be viewed as an array of bytes with a unique address for each location as shown in *Figure 2*.

<div class="imgcap">
<img src="https://raw.githubusercontent.com/tgautam03/tgautam03.github.io/master/assets/GPGPU_SERIES/00_Pointers/Figure_2.gif" alt="this slowpoke moves"  width="800"/>
<div class="thecap">Figure 2: Memory layout in modern computer. </div>
</div>

## Introduction to Pointers

Different data types take up different amount of space in the memory, for example an *integer* takes up 4 bytes and a double takes up 8 bytes. So when we declare `int a = 4`  in a program, it's stored in the memory where it takes up 4 *consecutive* bytes (shown by highlighted section in *Figure 3*).

<div class="imgcap">
<img src="https://raw.githubusercontent.com/tgautam03/tgautam03.github.io/master/assets/GPGPU_SERIES/00_Pointers/Figure_3.gif" alt="this slowpoke moves"  width="800"/>
<div class="thecap">Figure 3: 4 bytes occupied by variable in RAM with starting location at 102. </div>
</div>

**A pointer is an object that contains a memory address.** Very often this address is the location of another object such as variable. The general form of a pointer variable declaration is `type *var_name`. The base type of a pointer determines what type of data the pointer will be pointing to.

### Pointer Operators

Continuing the example used above, if we want `ptr` to points to integer `a`, i.e. store the address of integer variable `a`, we first have to define it with type `int` (line 2 of code in *Figure 4*). Now that we have defined a pointer, there are two special operators that are used with pointers to perform different tasks.

- `&`: Returns the memory address of it's operand. In line 3 of *Figure 4*, `&` retrieves the address at which variable `a` is stored in memory and puts that in the pointer `ptr`.

  *We know that variable `a` is stored in 4 locations, so which address value is retrieved by `&` operator?*

  The answer to this is that it returns the address at which the variable starts (i.e. in *Figure 4* it's location number 102).

- `*`: It's compliment of `&` and returns the value of the variable located at the address specified by its operand. Line 4 in *Figure 4* puts 4 in the variable `val`.

  *At this point, `ptr` only contains the location 102, but to form the integer 4, data at locations 103, 104 and 105 is also required. So how does a computer figure this out?*

  From the type of a pointer (`int` in this case), a compiler knows how many *consecutive* locations it has to access in order to provide the correct value.

  This operator can also be used to assign values to a variable.

  ```c++
  int x;
  int *ptr;

  ptr = &x; // ptr assigned address of x

  *ptr = 123; // x assigned value 123

  (*ptr)++; // Increments x by 1;
  cout << x << endl; // Prints 124

  (*ptr)--; // Decrements x by 1;
  cout << x << endl; // Prints 123

  return 0;
  ```

  <div class="imgcap">
  <img src="https://raw.githubusercontent.com/tgautam03/tgautam03.github.io/master/assets/GPGPU_SERIES/00_Pointers/Figure_4.gif" alt="this slowpoke moves"  width="800"/>
  <div class="thecap">Figure 4: Declaring and using a pointer.</div>
  </div>

**The base type of a pointer is important**. Pointer with base type `int` can only hold address of `int` variable. We can *cast* the type to get rid of the error but that will lead to different problems.

```c++
int *p;
double f;

p = &f; // ERROR!
```

## Pointers and Arrays

In C++. there's a close relationship between arrays and pointers. When an array `arr` is declared (`int arr[10]`), and the name is used without index, it returns a pointer that points to the 1st element of the array. Let's look at an example where we define an array with two elements. A pointer can then be used to access or manipulate the defined array as follows

```c++
int arr[2] = {3, 4}; // Defining array
int *ptr= 0; // Initialising pointer
ptr = arr; // Pointer now points to 1st element of array
```

We can now perform arithmetic operations on elements of array using pointer. Note that now we can use pointer and array interchangeably, hence `*(ptr+i)` will skip 3 locations and point to the next element of the array.

```c++
for (int i = 0; i < 2; i++)
    *(ptr+i) += 1; // Adding 1 to each element

// Printing array
for (int i = 0; i < 2; i++)
    std::cout << arr[i] << "\n";
```

> `*(ptr+i)` is equivalent to `ptr[i]`.

The above code gives the following output.

```terminal
4
5
```

> C++ allows two ways to access arrays, and using pointers is usually faster as compiler generates different code and mostly indexing operator has an overhead.

## Multidimensional Arrays

As the memory layout in a computer is linear, all multidimensional arrays are flattened to 1D and then stored. Let's consider an example of a 2D array with 2 rows and 2 columns. C++ uses row major layout, i.e. different rows are stacked one after the other to make a long 1D array as shown in *Figure 5*.

<div class="imgcap">
<img src="https://raw.githubusercontent.com/tgautam03/tgautam03.github.io/master/assets/GPGPU_SERIES/00_Pointers/Figure_5.gif" alt="this slowpoke moves"  width="800"/>
<div class="thecap">Figure 5: Linear layout of 2D array in memory. </div>
</div>

We can use similar technique to define a flattened 2D array (dynamic) in C++. Dynamic array is an array that can adjust it's size during runtime. It's memory space is allocated during runtime and is located in heap. To create a dynamic array we need a pointer variable and then use `new` to allocate space in the heap.

```c++
int *arr;
arr = new int[4]; // Allocate space for 4 integer variables
arr[0] = 2; // arr[0][0]
arr[1] = 3; // arr[0][1]
arr[2] = 4; // arr[1][0]
arr[3] = 5; // arr[1][1]
```

After we're done using this dynamic array it's **crucial** to delete it and free the space as follows.

```c++
delete [] arr;
```

> There is a way to create 2D dynamic arrays via *pointer of pointer*, but that doesn't work well for GPGPU programming.

## Conclusion

- CPUs and GPUs have very different architecture.

- Pointers are really powerful when it comes to manipulating data stored in the memory.

- In C/C++, Pointers and Arrays go hand in hand.

- Pointers provide the way to create dynamic arrays.

- All modern programs must use the combination of CPUs and GPUs.

## References

- [Link to intel website](https://www.intel.com/content/www/us/en/products/docs/processors/what-is-a-gpu.html)
- [Link to Wikipedia](https://en.wikipedia.org/wiki/General-purpose_computing_on_graphics_processing_units)
