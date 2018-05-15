---
title: "Place Volatile Accurately"
categories:
  - Notes
tags:
  - C
  - volatile
---

A volatile object is one whose value might change spontaneously. That is, when you declare an object to be volatile, you’re telling the compiler that the object might change state even though no statements in the program appear to change it.

Compilers can optimize accesses to nonvolatile objects by reading an object’s value into a CPU register, working with that register for a while, and eventually writing the value in the register back to the object. Compilers aren’t permitted to do this sort of optimization with volatile objects. Every time the source program says to read from or write to a volatile object, the compiled code must do so.

In my last column, I advised you to use the volatile qualifier, but use it judiciously.[1](http://www.embedded.com/electronics-blogs/programming-pointers/4025609/Place-volatile-accurately#endnotes) This month, I’ll  present some specific situations to show why and how you should think carefully about exactly where to place the **volatile** keyword in declarations.

**Modeling memory-mapped device registers** Many processors use memory-mapped I/O, which maps device registers to fixed addresses in the conventional memory space. To a C or C++ programmer, a memory-mapped device register looks very much like an ordinary data object.

For the past year or so, I’ve been presenting examples of memory-mapped I/O using the ARM Evaluator-7T single-board computer. The board’s documentation refers to the device registers as *special registers*. The memory is byte-addressable, but each register is a four-byte word aligned to an address that’s a multiple of four. You can manipulate each special register as if it were an **unsigned int**, or if you prefer, a **uint32_t**. (Fixed-size integer types such as **int16_t** and **uint32_t** are defined in the C99 header **<stdint.h>**.)[2](http://www.embedded.com/electronics-blogs/programming-pointers/4025609/Place-volatile-accurately#endnotes)

I generally prefer to use a symbolic type whose name conveys the meaning of the type rather than its physical extent, such as:

**typedef uint32_t special_register;**

This style works well on the Evaluator-7T. All of its special registers are of the same type, so you can get away with using only one typedef. On a machine with device registers of several different sizes, you might find yourself defining several such typedefs. In that case, many programmers prefer to stick with types such as **uint8_t**, **uint16_t**, and **uint32_t**.

Many devices interact through a small collection of device registers, rather than just one. For example, the Evaluator-7T has two UARTs, numbered 0 and 1. Each UART is controlled by six special registers. You can represent these registers as members of a struct defined as:

```c
typedef struct UART UART; 
struct UART 
	{ 
	special_register ULCON; 
	special_register UCON; 
	special_register USTAT; 
	special_register UTXBUF; 
	special_register URXBUF; 
	special_register UBRDIV; 
	};
```

The typedef before the struct definition elevates the name **UART** from a mere tag to a full-fledged type name.[3 ](http://www.embedded.com/electronics-blogs/programming-pointers/4025609/Place-volatile-accurately#endnotes)In C++, I’d define this struct as a class with appropriate member functions. Whether **UART** is a C struct or a C++ class doesn’t affect the following discussion.

The special registers for UART 0 reside at address 0x03FFD000. A program can access these registers via a “pointer to **UART**” whose value is that address. As I explained in an earlier column, you can define that pointer as a macro:[4](http://www.embedded.com/electronics-blogs/programming-pointers/4025609/Place-volatile-accurately#endnotes)

```c
#define UART0 ((UART *)0x03FFD000)
```

or as a constant object:

```c
UART *const UART0  = (UART *) 0x03FFD000;
```

In C++, you can use a reference instead of a pointer.[5](http://www.embedded.com/electronics-blogs/programming-pointers/4025609/Place-volatile-accurately#endnotes) Whether you use a pointer or reference doesn’t affect the following discussion, so I’ll just use a pointer.

**Hey! What about volatile?** Thus far, I haven’t used the keyword **volatile** in any of these declarations. The special registers that control a UART in the Evaluator-7T, like nearly all device registers everywhere, are volatile. As I explained in my previous column, if you don’t use volatile where needed, the compiler may optimize your source code too aggressively into object code that doesn’t work properly.

One way to ensure that the compiler treats UART 0 as a volatile object is to place the keyword **volatile** in the pointer declaration, as either:

```c
#define UART0 ((UART volatile *) 0x03FFD000)
```

or as:

```C
UART volatile *const UART0  = (UART *) 0x03FFD000;
```

If you use the latter declaration (the constant pointer object), you could also write **volatile** in the cast, as in:

```C
UART volatile *const UART0  = (UART volatile *)0x03FFD000
```

but it’s not necessary. For any type T, C and C++ provide a standard (built-in) conversion from “pointer to **T**” to “pointer to **volatile T**“, as well as a conversion from “pointer to **T**” to “pointer to **const T**“.

Declaring an entire object to be volatile (and/or const) effectively declares each member of that object as volatile (and/or const).

Adding the keyword **volatile** to the declaration of **UART0** will probably force you to add **volatile** to other declarations in the program. For example, suppose that:

```c
void put(char const *s, UART *u)
```

is a function that transmits characters one at a time from the null-terminated character sequence starting at **s** to the UART at **u**. If **UART0** is a “pointer to **volatile UART**“, the call:

```c
put("hello, world\n", UART0);   // error
```

won’t compile. The compiler will not convert a “pointer to **volatile UART**” into “pointer to **UART**“, unless you use a cast, as in:

```c
put("hello, world\n", (UART *)UART0);    // ouch!
```

The cast will allow the code to compile, but it won’t run properly because the put function will treat a volatile UART as if it were nonvolatile.

Forget the cast. What you should do is add **volatile** to the declaration of put’s second parameter, as in:

```c
void put(char const *s, UART volatile *u);    // yes!
```

Of course, adding **volatile** here may force you to add **volatile** elsewhere. Your compiler will be glad to point out where.**Modeling registers accurately** A declaration such as:

```c
UART volatile *const UART0 = ...;
```

has a subtle, but important, implication: that UART objects are not inherently volatile. That is, the declaration suggests that, while **UART0** points to a volatile UART, some UARTs elsewhere in the system might not be volatile. This is good programming style only if it’s an accurate model of the hardware. 

On the other hand, if all UARTs are indeed volatile, as is the case on the Evaluator-7T, then the model is inaccurate, and you would do better to build the volatility into the UART type.An easy way to build in volatility into each UART is to write the typedef as:

```c
typedef struct UART volatile UART;
```

C and C++ let you write **typedef**, **struct UART**, and **volatile** in any order. I suspect many programmers would prefer:

```c
typedef volatile struct UART UART;
```

I recommend placing **const** and **volatile** to the right of the types they modify.

Building volatility into the UART cleans up your code a bit. You no longer need to use **volatile** in the declaration of the pointer to the memory-mapped UART. That is, the declaration for UART0 can revert to either:

```c
#define UART0 ((UART *)0x03FFD000)
```

or:

```c
UART *const UART0  = (UART *) 0x03FFD000;
```

and the declaration of the put function can revert to:

```c
void put(char const *s, UART *u);
```

This is good.

On the other hand, the declaration:

```c
typedef struct UART volatile UART;
```

leaves me feeling a bit queasy. It actually defines two different types: **UART** as a volatile type and **struct UART** as a nonvolatile type. I want to ensure that all UART objects are volatile, but this declaration makes it possible to declare something like:

```c
struct UART *const UART0  = (struct UART *) 0x03FFD000;
```

and then access a UART as a nonvolatile object. I would think that’s a bug, not a feature.

In fact, the typedef:

```c
typedef struct UART volatile UART;
```

compiles only in C, not in C++. C++ compilers complain (or should complain) that the typedef is an invalid redefinition of type UART. Yet another reason to prefer C++ over C.

A better way to define UART as an inherently volatile type is to fold the entire struct definition into the typedef and eliminate the structure tag, as in:

```c
typedef struct /* no tag */ 
{ 
    special_register ULCON; 
    special_register UCON; 
    // etc. 
} volatile UART;
```

Alternatively, you can declare every member of the struct to be volatile, as in: typedef struct UART UART;

```c
struct UART 
{ 
    special_register volatile ULCON; 
    special_register volatile UCON; 
    // etc. 
};
```

Here we are again. 

If the **special_register** type is not inherently volatile, that implies that some special registers might not be volatile. In the Evaluator-7T, as in every other machine I’ve seen, all the memory-mapped device registers should be declared volatile. The most obvious way to do that is define **special_register** as a volatile type, as in:

```c
typedef uint32_t volatile special_register;
```

This is the style I’ve used in previous articles, and it’s the style I recommend.

**Me and my shadow** Under what circumstances would you leave volatility out of your device register type(s)? Some machines have some device registers that don’t like to be read. If you need to keep track of the value you last wrote to such a register, you must maintain a “shadow” copy of the register’s value stored in RAM. For clarity, the type of the shadow should be the same as the type of its corresponding device register. However, the shadow need not be declared **volatile**.

When dealing with a shadow register, you have a few stylistics choices. The simplest approach is to use the exact same type–a volatile type–for both the shadow and the device registers. The only downside to this approach is that the compiler may generate less than optimal code for accessing the shadow register. I suspect the impact will be negligible in most cases.

I recommend defining a nonvolatile type for the shadow register and a volatile version of the shadow register type for the corresponding device register, as in:

```c
typedef uint32_t shadow_register; 
typedef shadow_register volatile special_register;
```

Works for me.

Thanks to Andrew Sloss at ARM for help with this article.

**\*Dan Saks** is president of Saks & Associates, a C/C++ training and consulting company. For more information about Dan Saks, visit his website at www.dansaks.com. Dan also welcomes your feedback: e-mail him at dsaks@wittenberg.edu.*