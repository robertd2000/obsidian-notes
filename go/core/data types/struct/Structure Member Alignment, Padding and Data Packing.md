https://www.geeksforgeeks.org/c/structure-member-alignment-padding-and-data-packing/
In C, the structures are used as data packs. They don't provide any data encapsulation or data hiding features.

In this article, we will discuss the property of structure padding in C along with data alignment and structure packing.

## Data Alignment in Memory

Each data type has a natural alignment requirement, determined by the CPU architecture.

![32 bit data bus memory layout](https://media.geeksforgeeks.org/wp-content/uploads/20230404113848/32-bit-data-bus-layout.png)

Historically memory is byte addressable and arranged sequentially. If the memory is arranged as a single bank of one-byte width, the processor needs to issue 4 memory read cycles to fetch an integer. It is more economical to read all 4 bytes of an integer in one memory cycle. To take such advantage, the memory will be arranged as a group of 4 banks as shown in the above figure.

The memory addressing still be sequential. If bank 0 occupies an address X, bank 1, bank 2 and bank 3 will be at (X + 1), (X + 2), and (X + 3) addresses. If an integer of 4 bytes is allocated on X address (X is a multiple of 4), the processor needs only one memory cycle to read the entire integer. Whereas, if the integer is allocated at an address other than a multiple of 4, it spans across two rows of the banks as shown in the below figure. Such an integer requires two memory read cycles to fetch the data.

![misalign data in memory](https://media.geeksforgeeks.org/wp-content/uploads/20230404113922/misalign-data-in-memory.png)

A variable's _****data alignment****_ deals with the way the data is stored in these banks. For example, the natural alignment of _****int****_ on a 32-bit machine is 4 bytes. When a data type is naturally aligned, the CPU fetches it in minimum read cycles.

Similarly, the natural alignment of a _****short int****_ is 2 bytes. It means a _****short int****_ can be stored in bank 0 - bank 1 pair or bank 2 - bank 3 pair. A _****double****_ requires 8 bytes and occupies two rows in the memory banks. Any misalignment of _****double****_ will force more than two read cycles to fetch _****double****_ data.

Note that a ****double**** variable will be allocated on an 8-byte boundary on a 32-bit machine and requires two memory read cycles. On a 64-bit machine, based on a number of banks, a ****double**** variable will be allocated on the 8-byte boundary and requires only one memory read cycle.

## Structure Padding in C

Structure padding is the addition of some empty bytes of memory in the structure to naturally align the data members in the memory. It is done to minimize the CPU read cycles to retrieve different data members in the structure.

Try to calculate the size of the following structures:

```c
// structure A
typedef struct structa_tag {
    char c;
    short int s;
} structa_t;

// structure B
typedef struct structb_tag {
    short int s;
    char c;
    int i;
} structb_t;

// structure C
typedef struct structc_tag {
    char c;
    double d;
    int s;
} structc_t;

// structure D
typedef struct structd_tag {
    double d;
    int s;
    char c;
} structd_t;
```

  
Calculating the size of each structure by directly adding the size of all the members, we get:

- ****Size of Structure A**** = Size of (char + short int) = 1 + 2 = ****3.****
- ****Size of Structure B**** = Size of (short int + char + int) = 2 + 1 + 4 = ****7.****
- ****Size of Structure C**** = Size of (char + double + int) = 1 + 8 + 4 = ****13.****
- ****Size of Structure A**** = Size of (double + int + char) = 8 + 4 + 1= ****13.****

Now let's confirm the size of these structures using the given C Program:

// C Program to demonstrate the structure padding property

```c
// C Program to demonstrate the structure padding property
#include <stdio.h>

// Alignment requirements
// (typical 32 bit machine)

// char         1 byte
// short int    2 bytes
// int          4 bytes
// double       8 bytes

// structure A
typedef struct structa_tag {
    char c;
    short int s;
} structa_t;

// structure B
typedef struct structb_tag {
    short int s;
    char c;
    int i;
} structb_t;

// structure C
typedef struct structc_tag {
    char c;
    double d;
    int s;
} structc_t;

// structure D
typedef struct structd_tag {
    double d;
    int s;
    char c;
} structd_t;

int main()
{
    printf("sizeof(structa_t) = %lu\n", sizeof(structa_t));
    printf("sizeof(structb_t) = %lu\n", sizeof(structb_t));
    printf("sizeof(structc_t) = %lu\n", sizeof(structc_t));
    printf("sizeof(structd_t) = %lu\n", sizeof(structd_t));

    return 0;
}
```

  
**Output**

```
sizeof(structa_t) = 4
sizeof(structb_t) = 8
sizeof(structc_t) = 24
sizeof(structd_t) = 16
```


As we can see, the size of the structures is different from those we calculated.

> ****This is because of the alignment requirements of various data types, every member of the structure should be naturally aligned. The members of the structure are allocated sequentially in increasing order.****

Let us analyze each struct declared in the above program. For the sake of convenience, assume every structure type variable is allocated on a 4-byte boundary (say 0x0000), i.e. the base address of the structure is multiple of 4 (need not necessarily always, see an explanation of structc_t).

#### Structure A

The __structa_t__ first element is __char__ which is one byte aligned, followed by __short int__. short int is 2 bytes aligned. If the short int element is immediately allocated after the char element, it will start at an odd address boundary. The compiler will insert a padding byte after the char to ensure short int will have an address multiple of 2 (i.e. 2 byte aligned). The total size of structa_t will be,

****sizeof(char) + 1 (padding) + sizeof(short), 1 + 1 + 2 = 4 bytes.****

#### Structure B

The first member of __structb_t__ is short int followed by char. Since char can be on any byte boundary no padding is required between short int and char, in total, they occupy 3 bytes. The next member is int. If the int is allocated immediately, it will start at an odd byte boundary. We need 1-byte padding after the char member to make the address of the next int member 4-byte aligned. On total,

the __structb_t__ requires , ****2 + 1 + 1 (padding) + 4 = 8 bytes.****

#### Structure C - Every structure will also have alignment requirements

Applying same analysis, __structc_t__ needs sizeof(char) + 7-byte padding + sizeof(double) + sizeof(int) = 1 + 7 + 8 + 4 = 20 bytes. However, the sizeof(structc_t) is 24 bytes. It is because, along with structure members, structure type variables will also have natural alignment. Let us understand it by an example. Say, we declared an array of structc_t as shown below 

structc_t structc_array[3];

Assume, the base address of __structc_array__ is 0x0000 for easy calculations. If the structc_t occupies 20 (0x14) bytes as we calculated, the second structc_t array element (indexed at 1) will be at 0x0000 + 0x0014 = 0x0014. It is the start address of the index 1 element of the array. The double member of this structc_t will be allocated on 0x0014 + 0x1 + 0x7 = 0x001C (decimal 28) which is not multiple of 8 and conflicts with the alignment requirements of double. As we mentioned at the top, the alignment requirement of double is 8 bytes.

In order to avoid such misalignment, ****the compiler introduces alignment requirements to every structure.**** It will be as that of the largest member of the structure. In our case alignment of structa_t is 2, structb_t is 4 and structc_t is 8. If we need nested structures, the size of the largest inner structure will be the alignment of an immediate larger structure.

In structc_t of the above program, there will be a padding of 4 bytes after the int member to make the structure size multiple of its alignment. Thus the size of (structc_t) is 24 bytes. It guarantees correct alignment even in arrays.

#### Structure D

In a similar way, the size of the structure D is : 

sizeof(double) + sizeof(int) + sizeof(char) + padding(3) = 8 + 4 + 1 + 3 = 16 bytes

### How to Reduce Structure Padding?

By now, it may be clear that padding is unavoidable. There is a way to minimize padding. The programmer should declare the structure members in their increasing/decreasing order of size. An example is structd_t given in our code, whose size is 16 bytes in lieu of 24 bytes of structc_t.

## What is Structure Packing?

Sometimes it is mandatory to avoid padded bytes among the members of the structure. For example, reading contents of ELF file header or BMP or JPEG file header. We need to define a structure similar to that of the header layout and map it. However, care should be exercised in accessing such members. Typically reading byte by byte is an option to avoid misaligned exceptions but there will be a hit on performance.

Most of the compilers provide nonstandard extensions to switch off the default padding like pragmas or command line switches. Consult the documentation of the respective compiler for more details.

In GCC, we can use the following code for structure packing:

```c
#pragma pack(1)

```

or

```c
struct name {  
    ...  
}__attribute__((packed));
```


### Example of Structure Packing

```c
// C Program to demonstrate the structure packing
#include <stdio.h>
#pragma pack(1)

// structure A
typedef struct structa_tag {
    char c;
    short int s;
} structa_t;

// structure B
typedef struct structb_tag {
    short int s;
    char c;
    int i;
} structb_t;

// structure C
typedef struct structc_tag {
    char c;
    double d;
    int s;
} structc_t;

// structure D
typedef struct structd_tag {
    double d;
    int s;
    char c;
} structd_t;

int main()
{
    printf("sizeof(structa_t) = %lu\n", sizeof(structa_t));
    printf("sizeof(structb_t) = %lu\n", sizeof(structb_t));
    printf("sizeof(structc_t) = %lu\n", sizeof(structc_t));
    printf("sizeof(structd_t) = %lu\n", sizeof(structd_t));

    return 0;
}
```
  
**Output**

```
sizeof(structa_t) = 3
sizeof(structb_t) = 7
sizeof(structc_t) = 13
sizeof(structd_t) = 13
```
