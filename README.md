# Registers

Name | Meaning
---|---
**EAX** | Accumulator
**EBX** | Base
**ECX** | Counter
**EDX** | Data

```
	        |      AX      | 16 bit
                |  AH  ||  AL  | 8 bit
================================
|              EAX             | 32 bit
```



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



# Code Examples

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
		pop eax 	; Eemove param
		pop ecx 	; Eestore ecx
		loop loop1 	; Loop based on ecx
}
return 0;
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
