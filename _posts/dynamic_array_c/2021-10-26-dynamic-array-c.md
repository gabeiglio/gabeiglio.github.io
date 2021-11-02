---
title: Dynamic Arrays in C
date: 2021-10-26 00:00:00
tags: [software, c, data structures]
description: Implement an array like data structre that allocates memory as needed automatically in the c programming language
image:
---

In this blog post, I will explain how you can implement an array like data structure that lets you allocate as many
items as needed without worrying about the size of the array at compile time.

##  The Problem

C has many advantages than other programming languages, for me specially is its simplicity, but that comes with a cost.
The language lacks of many basic data structures used in day to day basis like **arrays** (at least dynamic ones).

With C you can only create arrays with constant sizes know at compile time.
```c
// array of integers of size 5
int array_int[5];

// initializing the items would be something like
for (int i = 0; i < 5; i++)
    array_int[i] = i;

// And accessing the items at an index would look like this
for (int i = 0; i < 5; i++)
    printf("%i\n", array_int[i]);

// output
0
1
2
3
4
```
But what happens if we want to keep adding numbers? ... well we can't, remember that we initialized the array with a
constant size of 5. But we could initialize the array with a significantly bigger number and not worry about going out
of bounds right? The answer to that question is yes, but that would waste a lot of memory that could be used by other
allocations.

## The Solution

The solution surprisingly is quite simple, we need to create an object that mimics some of the properties that a dynamic
array has, we need to keep track of the **capacity** (how many items can it hold until it fills up), the **count**
(currently in use slots in the array) and lastly the actual **array** which will be a pointer to the first item of the
array.

It would look like this.
```c
typedef struct {
    size_t capacity; // how much can currently hold
    size_t count;    // how much is currently holding
    int* array;      // pointer to the first item in the sequence
} Array;
```
Now its need a method to initialize the array.
```c
void init_array(Array* a) {
    a->capacity = 8; // we initialize the array to 8 slots, but it can be whatever number you like
    a->count = 0;   //  initially we dont occupy any slots 
    a->array = malloc(sizeof(int) * a->capacity); //allocate sufficient memory to hold 8 (capacity) integers
}
```
Let's talk about that weird function **malloc** (memory allocation), it allocates bytes of unitialized memory, and
returns a pointer if succesfull, but we will talk about this later in the post.

Great! now our array needs a function in order to add items to it.
```c
void add_array(Array* a, int item) {
    if (a->capacity < a->count + 1) {
        a->capacity *= 2;
        a-array = realloc(a->array, sizeof(int) * a->capacity);
    }

    a->array[a->count] = item;
    a->count++;
}
```
Let's explain what this functions does. In order to add an item to the array, first we have to check if the array is
full, basically if the count plus the one item that we are trying to add is less than the capacity, then no
problem we just assign it to the current index and increment the count to keep track of how many slots we are currently
taking.

But, if the count plus the new item is more than the capacity, we do two things first, we double the capacity, and
reallocate the array to a new memory space with the size of recently augmented capacity.

**Note:** realloc not always just allocates new memory and just point to it, [here](https://en.cppreference.com/w/c/memory/realloc) is an article in which explains how realloc works, but just know that realloc can fail, so we will handle that later.

And lastly, we need a function to free the array when we are done using it, since we allocated memory on the heap, we
are responsible of freeing it.
```c
void free_array(Array* a) {
    a->capacity = 0;
    a->count = 0;
    free(a->array);
}
```

## End Result

With these couple of methods we can use our Array like this.
```c
int main(int argc, char* argv[]) {
    
    // create and initialize array
    Array a;
    init_array(&a);

    //Add items to it
    add_array(&a, 1);
    add_array(&a, 2);
    add_array(&a, 3);
    add_array(&a, 4);
    add_array(&a, 5);
    add_array(&a, 6);
    add_array(&a, 7);
    add_array(&a, 8);
    add_array(&a, 9); // here the array will be reallocated automatically

    // you can access the values at a given index like this
    printf("%i", a.array[0]); // output: 1
    printf("%i", a.array[8]); // output: 8
 
    // free the array when we are done using it
    free_array(&a);
}
```

## Let's fix some potential bugs!

We have two functions malloc and realloc, that returns a pointer to a memory address, and we faithfully assign it to our
array pointer trusting that we are getting the actual pointer instead of a null pointer since the program could not find
sufficient space to fill our needs.

In order to handle this error we assign the malloc and realloc result to a temporary pointer variable, in which then we
check if the result is null, if it is null, we handle that error (we just print "Error: Could not allocate memory" and
exit the program), if the pointer is not null, then we assign the temporary pointer to our array pointer.

Our init_array function would look like:
```c
void init_array(Array* a) {
    a->capacity = 8;
    a-> count = 0;

    int* tmp = malloc(sizeof(int) * a->capacity);

    if (!tmp) {
        fprintf(stderr, "[ERROR] Coud not allocate memory");
        exit(1);
    }

    a->array = tmp;
}
```

And add_array would look like:
```c
void add_array(Array* a, int item) {
    if (a->capacity < a->count + 1) {
        a->capacity *= 2;

        int* tmp = realloc(a->array, sizeof(int) * a->capacity);

        if (!tmp) {
            fprintf(stderr, "[ERROR] Could not allocate memory");
            exit(1);
        }

        a->array = tmp;
    }

    a->array[a->count] = item;
    a->count++;
}
```

## Conclusion

So with all of this now you are ready to allocate as many items as you want in the array at runtime without worrying
about the size of our array and concentrate more on what we are actually trying to code.

### Full Code Listing
```c
#include <stdio.h>
#include <stdlib.h>

typedef struct {
    size_t capacity; // how much can currently hold
    size_t count;    // how much is currently holding
    int* array;      // pointer to the first item in the sequence
} Array;

void init_array(Array* a) {
    a->capacity = 8;
    a-> count = 0;

    int* tmp = malloc(sizeof(int) * a->capacity);

    if (!tmp) {
        fprintf(stderr, "[ERROR] Coud not allocate memory");
        exit(1);
    }

    a->array = tmp;
}

void add_array(Array* a, int item) {
    if (a->capacity < a->count + 1) {
        a->capacity *= 2;

        int* tmp = realloc(a->array, sizeof(int) * a->capacity);

        if (!tmp) {
            fprintf(stderr, "[ERROR] Could not allocate memory");
            exit(1);
        }

        a->array = tmp;
    }

    a->array[a->count] = item;
    a->count++;
}

void free_array(Array* a) {
    a->capacity = 0;
    a->count = 0;
    free(a->array);
}

int main(int argc, char* argv[]) {

    // create and initialize array
    Array a;
    init_array(&a);

    //Add items to it
    add_array(&a, 1);
    add_array(&a, 2);
    add_array(&a, 3);
    add_array(&a, 4);
    add_array(&a, 5);
    add_array(&a, 6);
    add_array(&a, 7);
    add_array(&a, 8);
    add_array(&a, 9); // here the array will be reallocated automatically

    // you can access the values at a given index like this
    printf("%i", a.array[0]); // output: 1
    printf("%i", a.array[8]); // output: 9

    // free the array when we are done using it
    free_array(&a);
}
```
