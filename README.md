# Comp 124 Assembly Notes

These are my notes concerning x86 Assembly which was taught in the Comp 124 module in the University of Liverpool

# Registers

Name | Meaning
---|---
**EAX** | Accumulator
**EBX** | Base
**ECX** | Counter
**EDX** | Data
**ESP** | Stack Pointer
**EIP** | Instruction Pointer (not really a register)


```
	        |      AX      | 16 bit
                |  AH  ||  AL  | 8 bit
================================
|              EAX             | 32 bit
```
Every register is split like this (example with **EBX** - the parts are called BX, BH, and BL)



# Flags

Name | Meaning
---|---
**S** | Sign
**Z** | Zero
**C** | Carry
**O** | Overflow


# Instructions

Name | Meaning
---|---
**MOV** x, y | Assign y to x
**INC** x    | Increment x by one
**DEC** x    | Decrease x by one
**ADD** x, y | Add y to x
**SUB** x, y | Subtract y from x
**JMP** x    | Jump to x
**CMP** x, y | Compare x to y
**LEA** x, y | Get adress of y and store it in x (Lead Effective Adress) (Often used with EBX as x)
**LOOP** x   | Decrement ECX and Jump to x if not Zero
**CALL** x   | Call function or procedure x
**RET**      | Retrieve stored return address and put it back into the EIP


# Jumps

### Flag Jumps

Name | Meaning
---|---
**JS** | S = 1
**JNS** | S = 0
**JZ** | Z = 1
**JNZ** | Z = 0
**JC** | C = 1
**JNC** | C = 0
**JO** | O = 1
**JNO** | O = 0

### Conditional Jumps
(assuming execution just after **CMP**)

Name | Meaning
---|---
**JE** | Equal
**JNE** | Not Equal
**JG/JNLE** | Greater (Not Less or Equal)
**JL/JNGE** | Less (Not Greater or Equal)
**JGE/JNL** | Greater or Equal (Not Less)
**JLE/JNG** | Less or Equal (Not Greater)

# Procedures

### Declaration of a procedure

```
label PROC
	.
	.
	.
	RET
label ENDP
```

Then the procedure can be called with `CALL label`


### The CALL instruction

**CALL** does the following:
1. Records the current value of **EIP** (Instruction Pointer) as the **return address**
2. Puts the required subroutine address into **EIP**, so the next instruction to be executed is the first instruction of the subroutine

We can also call C functions with `CALL`

### The RET instruction

**RET** retrieves the stored **return address** and puts it back into the **EIP**

# The Stack

* A Stack is a data structure 
* The order of storing and retrieving values can be described as **LIFO** (Last In, First Out)
* Every stack is equipped with two operations, **PUSH** and **POP**
* **PUSH** and **POP** make use of the **ESP** (Stack Pointer Register)
* In the x86 architecture the ctack grows **down** in memory
* The stack is adressed using the **ESP** (Stack Pointer)
* When adding to the **ESP** you move it upwards, freeing the memory that was previously adressed by it
* You can move **ESP** yourself, but **PUSH** and **POP** can do it for you

### The PUSH instruction

1. Decrements the address in **ESP** so that it points to a free space on the stack
2. Writes an item to the memory location pointed to by the **ESP**

### The POP instruction

1. Fetches the item addressed by the **ESP**
2. Increments the **ESP** by the correct amount to remove the item from the stack

It is the programmer's responsibility to ensure that items are not left on the stack when no longer needed

# Code Examples

### Full program that takes a number, adds 10 to it and prints it

To run the program you need to use Visual Studio 2017

If you haven't properly configured VS you may get errors, please refer to the COMP124 Parctical Session 1 for proper configuration

Common mistakes include:

* Not having headers removed (the first line in this example)
* Creating a file, and not a cpp project

```cpp
#include "pch.h" // This may have to be removed depending on your configuration
#include <iostream>

int main() {
	char fmt[] = "%d";

	int num;

	_asm {
		// Reading user input
		lea eax, num	// Save the adress of num in eax
		push eax	// Push eax on the stack
		lea eax, fmt	// Save the adress of the format string in eax
		push eax	// Push eax on the stack
		call scanf	// Call scanf, a C function
		add esp, 8      // Reset the ESP (Stack Pointer)
		// We need to add 8 to ESP, because both adresses we put on the stack take
		// up 4 bytes. Now the stack is empty, and we have the user input stored in num

		// Adding 5 to num
		mov eax, num	// Save the value of num in eax
		add eax, 10	// Add 10 to eax
		mov num, eax	// Save the value of 10 to eax
		// We can't add to num directly

		// Printing num
		push num	// Push the value of num on the stack
		lea eax, fmt	// Save the adress of fmt in eax
		push eax	// Push eax on the stack
		call printf	// Call printf, a C function
		add esp, 8	// Reset the ESP
		// We need to add 8 to ESP because the value of nu, which is an int, takes up
		// 4 bytes, and the pointer to fmt also takes 4 bytes
	}
	return 0;
}
```

We could use **POP** instead of ``add esp, 8`` in this example and it would also work.

### Print "Hello World"
```cpp
char msg[] = "Hello World\n"; // declare variables in C

_asm {
		lea eax, msg	; Put address of string into eax
		push eax 	; Stack the parameter
		call printf	; Use library function
		pop eax 	; Take parameter off stack
}
return 0;
```

### Print "Hello World" 10 times
```cpp
char msg[] = "Hello World\n"; // declare variables in C

_asm {
		mov ecx, 10 ; Set up loop counter
	loop1: 	
		push ecx 	; Save ecx on stack
		lea eax,msg 	; Save "Hello World" in EAX
		push eax 	; Stack the parameter
		call printf 	; Use function
		pop eax 	; Remove param
		pop ecx 	; Restore ecx
		loop loop1 	; Loop based on ecx
}
return 0;
```

### Print a number

Equivalent of `printf("Number is %d\n", n);`

```cpp
char msg[] = "Number is %d\n";
int n = 157;
_asm { 
	push n 		; Push the int first
	lea eax, msg
	push eax 	; Now stack the string
	call printf
	add esp,8 	; Clean 8 bytes from stack
}
return 0;
```

### Reading input

Reading values can be achieved with calls to `scanf("%d", &num);`

```cpp
char fmt[] = "%d"; 
int num;
_asm {
	lea eax, num 	; we need to push the address of num
	push eax
	lea eax, fmt ; now the format string
	push eax
	call scanf
	add esp, 8 	; clean stack
}
```

### Fibonacci up to 1000

```cpp
while1:
	mov eax,fib2
	cmp eax,1000
	jge end_while
	mov eax,fib1
	mov fib0,eax
	mov eax,fib2
	mov fib1,eax
	add eax, fib0
	mov fib2,eax
	jmp while1
end_while:
```

### Sum the elements of an array

```cpp
int myarray[5]; // declaration of an array of integers

myarray[0] = 1;
myarray[1] = 3;
myarray[2] = 5;
myarray[3] = 7;
myarray[4] = 9;

_asm {
	lea ebx,myarray ; save adress of array (0th element) in ebx
	mov ecx,5 	; size of the array in ecx
	mov eax,0 	; initialise sum to 0
loop1:  add eax,[ebx] 	; get element pointed to by ebx
	add ebx,4 	; to point to next integer element
	loop loop1 	; go round again
}
```

Alternatively:

```cpp
_asm {
	mov ebx, 0 		; start with array offset of zero
	mov ecx,5 		; size of the array in ecx
	mov eax,0 		; initialise sum to 0
loop1: 	add eax, myarray[ebx] 	; use ebx to index array
	add ebx,4 		; update offset
	loop loop1 		; go round again
}
```

### Determine bigger of two numbers

```cpp
/* Call procedure to determine bigger of 2 numbders
* The numbers are passed in eax and ebx.
* Result returned in eax
*/

	mov eax, first
	mov ebx, second ; CALLING
	call bigger ; SEQUENCE
	mov max, eax
	...
bigger: proc
	cmp eax,ebx
	jl second_big
	ret
second_big:
	mov eax,ebx
	ret
bigger: endp
```
