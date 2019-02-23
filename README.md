# MUE CPU Specification v1.0-alpha

## Introduction

This document is a complete specification of MUE central processing unit (MCPU).
It describes the system, the principle of operation, interrupt processing, the list of instructions, the list of registers, etc.
MCPU is a 16 bit RISK processor with the ability to connect I/O devices, I/O processing, interrupt queue processing and execution of the program.
65535 devices can be connected to the MCPU at the same time.
One of the connected devices is the MCPU itself (with index 0).
MCPU frequency is set separately.

## Definitions

**MCPU** - MUE central processing unit.

**RAM** - Random-access memory of MCPU.

**Word** - 2 byte memory cell. The machine word of MCPU.

**Instruction** - Elementary atomic MCPU instruction defined in the instruction list. The instruction has a certain execution cost. Each instruction has its own code and name (mnemonic).

**Operand** - The part of a operation which specifies what data is to be operated on. This can be a register, a value in RAM or a constant. The operand has a certain execution cost.

**Operation** - General concept for the instruction and several operands. May have zero, one or two operands. The cost of the operation is the cost of the instruction and the cost of all its operands.

**Frequency** - Possible number of MCPU cycles per second.

**Tick** - A single clock cycle.

**Cost** - How many ticks does the operation take.

**Context** - Values of all registers (And, perhaps internal variables implementation).

## Architecture

**16-bit** machine word.

**Little-endian** endianness.

**131072 bytes** (128 kb) of RAM.

**256 interrupts** of interrupt queue.

**10** registers. (6 - general purpose, 4 - special)

**44** MCPU instructions.

Possibly **65535** ports for devices.

The ability to save and restore **context**.

## Numbers representation

The method of signed number representation is **Two's complement**.

An example of the representation of numbers in MCPU:
| Decimal (unsigned) | Decimal (signed) | Binary (word)       |
|:------------------:|:----------------:|:-------------------:|
| 0                  | 0                | 0000_0000_0000_0000 |
| 1                  | 1                | 0000_0000_0000_0001 |
| 2                  | 2                | 0000_0000_0000_0010 |
| 32767              | 32767            | 0111_1111_1111_1111 |
| 65535              | -1               | 1111_1111_1111_1111 |
| 65534              | -2               | 1111_1111_1111_1110 |
| 32768              | -32768           | 1000_0000_0000_0000 |

## Registers

MCPU has 10 registers (A, B, C, D, E, F, X, S, P, I) 2 bytes long (16 bit).
6 of them are general purpose (A, B, C, D, E, F) and 4 special (X, S, P, I).
These registers are usually denoted by capital letters.

Each register has a name and its purpose:
| Register | Name        | Purpose                                                  |
|:--------:|:------------|:---------------------------------------------------------|
| A        | Accumulator | General. Usually used to accumulate value.               |
| B        | Base        | General.                                                 |
| C        | Counter     | General. Usually used as a counter.                      |
| D        | Data        | General.                                                 |
| E        | Error       | General. Usually used to return an error code.           |
| F        | Function    | General. Usually used to return a value from a function. |
| X        | eXtension   | The storage of high digits of a long number.             |
| S        | Stack       | Stack pointer.                                           |
| P        | Port        | Register to work with connected devices.                 |
| I        | Instruction | Instruction pointer.                                     |

## Memory

In addition to the ten registers, MCPU has RAM of 65536 words.
Each memory cell has a length of one word and its own number.
Numbering cells from 0 to 65535.

Also MCPU has a memory to save context.
This is denoted in the same way as common registers, only with apostrophe (A', B', C', D', E', F', X', S', P', I').

Other auxiliary memory for MCPU is not specified.

## Operands

Each operation has zero, one or two operands.
Each operand has an execution cost and its own number.
An operand can be a register, a constant or a value in RAM.
To read the value of a constant, you need to read next word after the instruction (or two words, if both operands are constants).

| Number | Operand | Cost | Read/Write |
|:-------|:-------:|:----:|:----------:|
| 0      |  A      | 0    | rw         |
| 1      |  B      | 0    | rw         |
| 2      |  C      | 0    | rw         |
| 3      |  D      | 0    | rw         |
| 4      |  E      | 0    | rw         |
| 5      |  F      | 0    | rw         |
| 6      |  X      | 0    | rw         |
| 7      |  S      | 0    | rw         |
| 8      |  P      | 0    | rw         |
| 9      |  I      | 0    | rw         |
| 10     | [A]     | 0    | rw         |
| 11     | [B]     | 0    | rw         | 
| 12     | [C]     | 0    | rw         |
| 13     | [D]     | 0    | rw         |
| 14     | [E]     | 0    | rw         |
| 15     | [F]     | 0    | rw         |
| 16     | [X]     | 0    | rw         |
| 17     | [S]     | 0    | rw         |
| 18     | [P]     | 0    | rw         |
| 19     | [I]     | 0    | rw         |
| 20     | [A + c] | 1    | rw         |
| 21     | [B + c] | 1    | rw         |
| 22     | [C + c] | 1    | rw         |
| 23     | [D + c] | 1    | rw         |
| 24     | [E + c] | 1    | rw         |
| 25     | [F + c] | 1    | rw         |
| 26     | [X + c] | 1    | rw         |
| 27     | [S + c] | 1    | rw         |
| 28     | [P + c] | 1    | rw         |
| 29     | [I + c] | 1    | rw         |
| 30     |  c      | 1    | r          |
| 31     | [c]     | 1    | rw         |

Where: *c* - constant. *[ x ]* - value in RAM at address *x*.

Any operand except a constant can read and write value. You can only read a value from a constant.

## Instructions

This is a table of all MCPU instructions.
Each instruction has its cost in ticks.

| Number | Mnemonic | Definition                                                  | Cost |
|:-------|:---------|:------------------------------------------------------------|:----:|
| 0      | pass     | Doing nothing                                               | 1    |
| 1      | stop a   | Stop executing with code a                                  | 1    |
| 2      | set  a b | a <- b                                                      | 1    |
| 3      | add  a b | a += b                                                      | 2    |
| 4      | sub  a b | a -= b                                                      | 2    |
| 5      | addx a b | X:a <- a + b                                                | 3    |
| 6      | subx a b | X:a <- a - b                                                | 3    |
| 7      | mul  a b | X:a <- a * b                                                | 3    |
| 8      | muli a b | X:a <- a * b (sign)                                         | 3    |
| 9      | div  a b | a /= b                                                      | 3    |
| 10     | divi a b | a /= b (sign)                                               | 3    |
| 11     | mod  a b | a %= b                                                      | 3    |
| 12     | inc  a   | a += 1                                                      | 1    |
| 13     | dec  a   | a -= 1                                                      | 1    |
| 14     | not  a   | a <- ~a                                                     | 1    |
| 15     | and  a b | a &= b                                                      | 1    |
| 16     | or   a b | a \|= b                                                     | 1    |
| 17     | xor  a b | a ^= b                                                      | 1    |
| 18     | shl  a b | a <<= b                                                     | 1    |
| 19     | shr  a b | a >>= b                                                     | 1    |
| 20     | shra a b | a >>= b (arithmetic)                                        | 1    |
| 21     | push a   | S -= 1; [S] <- a                                            | 1    |
| 22     | pop  a   | a <- [S]; S += 1                                            | 1    |
| 23     | go   a   | I <- a                                                      | 1    |
| 24     | call a   | S += 1; [S] <- N(I); I <- a                                 | 2    |
| 25     | ret      | I <- [S]; S -= 1                                            | 1    |
| 26     | devs a b | Set device value by number b to a                           | 1    |
| 27     | devg a b | Get device value by number b in a                           | 1    |
| 28     | in   a   | Read value from device to a                                 | 1    |
| 29     | out  a   | Write value to device from a                                | 1    |
| 30     | int  a   | Add interrupt by address a to interrupt queue               | 3    |
| 31     | iret     | End of an interrupt procedure and restore context           | 3    |
| 32     | ift  a   | Execute next instruction only if a <> 0 else skip           | 2    |
| 33     | iff  a   | Execute next instruction only if a = 0 else skip            | 2    |
| 34     | ife  a b | Execute next instruction only if a = b else skip            | 2    |
| 35     | ifne a b | Execute next instruction only if a <> b else skip           | 2    |
| 36     | ifg  a b | Execute next instruction only if a > b else skip            | 2    |
| 37     | ifa  a b | Execute next instruction only if a > b else skip (sign)     | 2    |
| 38     | ifl  a b | Execute next instruction only if a < b else skip            | 2    |
| 39     | ifb  a b | Execute next instruction only if a < b else skip (sign)     | 2    |
| 40     | ifng a b | Execute next instruction only if not a > b else skip        | 2    |
| 41     | ifna a b | Execute next instruction only if not a > b else skip (sign) | 2    |
| 42     | ifnl a b | Execute next instruction only if not a < b else skip        | 2    |
| 43     | ifnb a b | Execute next instruction only if not a < b else skip (sign) | 2    |

Where: *N(I)* is an address of next instruction. *a*, *b* is any operands.

### Detailed specification

**pass** - Empty instruction. Does nothing.

**stop a** - Stop execution. Stops the MCPU and sets the termination code *a*. *a* can be read only.

**set a b** - Assignment. Sets *a* to *b*. *b* can be read only. *a* must be writable.

**add a b** - Addition. Adds *a* and *b* and writes the result to *a*. *b* can be read only. *a* must be writable. If the result exceeds the word, the high-order bits are discarded.

**sub a b** - Subtraction. Subtracts *a* from *b* and writes the result to *a*. *b* can be read only. *a* must be writable. If the result exceeds the word, the high-order bits are discarded.

**addx a b** - Extended addition. Adds *a* and *b* and writes the lower 16 bits of result to *a*, and the high bits to register *X*. *b* can be read only. *a* must be writable. If *a* is *X* then the error will trigger.

**subx a b** - Extended subtraction. Subtracts *a* from *b* and writes the lower 16 bits of result to *a*, and the high bits to register *X*. *b* can be read only. *a* must be writable. If *a* is *X* then the error will trigger.

**mul a b** - Multiplication. Multiplies *a* by *b* and writes the lower 16 bits of result to *a*, and the high bits to register *X*. *b* can be read only. *a* must be writable. If *a* is *X* then the error will trigger.

**muli a b** - Sign multiplication. Multiplies *a* by *b* and writes the lower 16 bits of result to *a*, and the high bits to register *X*. The values of *a*, *b* and result are interpreted as signed numbers. *b* can be read only. *a* must be writable. If *a* is *X* then the error will triggered.

**div a b** - Division. Divides *a* by *b* and writes the result to *a*. *b* can be read only. *a* must be writable.

**divi a b** - Sign division. Divides *a* by *b* and writes the result to *a*. The values of *a*, *b* and result are interpreted as signed numbers. *b* can be read only. *a* must be writable.

**mod a b** - Modulo. Calculates the modulo *a* and *b* and writes the result to *a*. *b* can be read only. *a* must be writable.

**inc a** - Increment. Increases *a* by 1. *a* must be writable.

**dec a** - Decrement. Decreases *a* by 1. *a* must be writable.

**not a** - Bitwise NOT. Sets *a* to bitwise NOT of *a*. *a* must be writable.

**and a b** - Bitwise AND. Calculates bitwise *a* AND *b*, then sets *a* to the result. *b* can be read only. *a* must be writable.

**or a b** - Bitwise OR. Calculates bitwise *a* OR *b*, then sets *a* to the result. *b* can be read only. *a* must be writable.

**xor a b** - Bitwise XOR. Calculates bitwise *a* XOR *b*, then sets *a* to the result. *b* can be read only. *a* must be writable.

**shl a b** - Left circular shift. Bitwise shifts to the left *a* by *b* positions. *b* can be read only. *a* must be writable.

**shr a b** - Right circular shift. Bitwise shifts to the right *a* by *b* positions. *b* can be read only. *a* must be writable.

**shra a b** - Arithmetic right shift. Bitwise shifts to the right *a* by *b* positions without changing the sign of the value *a*. *b* can be read only. *a* must be writable.

**push a** - Push to stack. Decreases *S* by 1, then writes *a* to stack. *a* can be read only.

**pop a** - Extract from stack. Writes value from stack to *a*, then increases *S* by 1. *a* must be writable.

**go a** - Go to. Sets *I* to *a*. *a* can be read only.

**call a** - Function (procedure) call. Increases *S* by 1, then writes address of next operation to stack, then sets *I* to *a*. *a* can be read only.

**ret** - Return. Sets *I* to value from stack, then decreases *S* by 1.

**devs a b** - Set device value. Set device value by number *b* to *a*. The device is defined by value in register *P*. *a* and *b* can be read only.

**devg a b** - Get device value. Get device value by number *b* in *a*. The device is defined by value in register *P*. *a* must be writable. *b* can be read only.

**in a** - Read value from device and set to *a*. The device is defined by value in register *P*. *a* must be writable.

**out a** - Write value *a* to device. The device is defined by value in register *P*. *a* can be read only.

**int a** - Add interrupt by address *a* to interrupt queue. Then MCPU will save context and will call interrupt procedure at address *a*. *a* can be read only.

**iret** - Return from interrupt procedure. End of an interrupt procedure and restore context.

**ift** - If true. Execute next instruction only if *a* is not 0, else skip. *a* can be read only.

**iff** - If false. Execute next instruction only if *a* is 0, else skip. *a* can be read only.

**ife** - If equal. Execute next instruction only if *a* is *b*, else skip. *a* and *b* can be read only.

**ifne** - If not equal. Execute next instruction only if *a* is not *b*, else skip. *a* and *b* can be read only.

**ifg** - If greater. Execute next instruction only if *a* > *b*, else skip. The values of *a* and *b* are interpreted as unsigned numbers. *a* and *b* can be read only.

**ifa** - If above. Execute next instruction only if *a* > *b*, else skip. The values of *a* and *b* are interpreted as signed numbers. *a* and *b* can be read only.

**ifl** - If less. Execute next instruction only if *a* < *b*, else skip. The values of *a* and *b* are interpreted as unsigned numbers. *a* and *b* can be read only.

**ifb** - If below. Execute next instruction only if *a* < *b*, else skip. The values of *a* and *b* are interpreted as signed numbers. *a* and *b* can be read only.

**ifng** - If not greater. Execute next instruction only if *a* <= *b*, else skip. The values of *a* and *b* are interpreted as unsigned numbers. *a* and *b* can be read only.

**ifna** - If not above. Execute next instruction only if *a* <= *b*, else skip. The values of *a* and *b* are interpreted as signed numbers. *a* and *b* can be read only.

**ifnl** - If not less. Execute next instruction only if *a* >= *b*, else skip. The values of *a* and *b* are interpreted as unsigned numbers. *a* and *b* can be read only.

**ifnb** - If not below. Execute next instruction only if *a* >= *b*, else skip. The values of *a* and *b* are interpreted as signed numbers. *a* and *b* can be read only.

Instructions of *if* type (ift, iff, ife, ifne, ifg, ifa, ifl, ifb, ifng, ifna, ifnl, ifnb) check condition.
If it is true, the next instruction is executed.
If it is false, the next instruction is skipped.
Instructions of *if* type can be chained.
If the next instruction is also a condition, the conditions are checked sequentially, each after each.
The last instruction of chaining (not *if*) is executed if all previous conditions are true.
MCPU spends ticks on all conditions, even if the general condition is false, and on last instruction, even if it's not executed.

Example:
```
set A 42
set B 42
ife A B
    stop 0
```
This program will stop with code 0, because the condition *ife A B* is true (A = B).

Another example:
```
set A 1
ift A
    ife A 4
        stop 0
```
In this chain of conditions, *ift A* (true) is checked first, then *ife A 4* (false).
The *stop 0* instruction will not be executed because one of the conditions is false, but both conditions and the last instruction will cost MCPU ticks.

## Operations

An operation is specified by the instruction number and operand numbers.
An operation is located in RAM as follows: the 6 low bits of the word are the instruction number, the next 5 high bits are the first operand and the next 5 bits is the second operand.

It looks that way:
```
[ bbbbbaaaaaiiiiii ] - binary instruction.

i - bits of instruction.
a - bits of the first operand.
b - bits of the second operand.
```

If an instruction has one or zero operands, the free space is filled with 0:
```
[ 00000aaaaaiiiiii ] - unary instruction.
[ 0000000000iiiiii ] - nullary instruction.
```

If an operation contains constants, they are located in the next instruction word.
If an operation contains two constants, then first the constant of the first operand is located, then the second one.

Example:
```
0001: [ bbbbbaaaaaiiiiii ] - instruction.
0002: [ xxxxxxxxxxxxxxxx ] - first operand constant.
0003: [ yyyyyyyyyyyyyyyy ] - second operand constant.
```

Then, the operation *add D F* (at memory index 1) will be as follows:
```
0001: [ 00101 00011 000100 ]
```

Since the operand *D* has the number 5 (101), *F* has the number 3 (11) and the instruction *add* has the number 4 (100).

Another example. The operation *set [10] 6* (at memory index 2) will be as follows:
```
0002: [ 1111011111000011 ]
0003: [ 0000000000001010 ]
0004: [ 0000000000000110 ]
```

Since the operand of *[c]* (where *c* is constant) has the number 31 (11111), *c* has the number 30 (11110) and the instruction *set* has the number 3 (11).
The value of the first constant *10* (1010) is located in the word following the instruction.
The value of the second constant *6* (110) is located in the next word after the first constant.

## Interrupts

MCPU has an interrupt queue. Queue size is 256 interrupts.
Each interrupt is an interrupt address and its size is one word.
Any connected device or MCPU itself can add an interrupt to the queue.
After processing of instruction, if there are interrupts in the queue, then MCPU saves current context and sets *I* to the interrupt address from the queue.
Thus, it is necessary to process all interrupts in the queue.
Each interrupt procedure must end with *iret* instruction.
*iret* completes the interrupt processing.
After processing all interrupts in the queue MCPU restores the context.

Handling the case of interrupt queue overflowing is not specific.

If address of the interrupt is 0, its processing must be ignored.

You can call an interrupt from the program of MCPU using the instructions *int a*, where *a* is the interrupt address.
In this case, the interrupt will add to queue and will call before next instruction is executed.

Example:
```
int 40 # call the interrupt by address 40
stop 0 # this instruction will be executed after the interrupt is processed
```

Another example:
```
set A 1
int int_code
stop A

int_code:
        set A 2
        iret
```
This program will call an interrupt at the label address *int_code*.
The procedure sets *A* to 2 and ends.
Then program will end with code 1, since after processing the interrupt, the execution context is restored.
To save the data processed in interrupt procedure, you need write it in the RAM.
```
int int_code
stop [code_value]

int_code:
        set [code_value] 2
        iret

code_value: 0
```
This program will end with code 2.

MCPU allows you to add interrupts to a queue inside another interrupt.
In this case, MCPU will wait for the current interrupt to complete and will process a new interrupt in turn.

Example:
```
int int_1
stop [x]

int_1:
        int int_2
        iret

int_2:
        set [x] 5
        iret

x: 0
```
First, the *int_1* interrupt will be triggered, then after it is completed, the *int_2* interrupt will be triggered.

## Devices

Up to 65535 devices can be connected to the MCPU.
A device can be anything.
For example: keyboard, display, data storage, network interface, timer, random number generator, etc.
The device number 0 is always MCPU itself.
*P* register is used to work and exchange data with devices.
To work with a device, you need to set device number in *P*.

Each device must support the input/output interface (if necessary) and have the its own state, changeable from the MCPU (if necessary). It may also require interrupts triggering.
For input/output MCPU uses instructions *in* and *out*.
To set device state MCPU uses *devs* and *devg* to get device state.

Example. Let the display be connected to MCPU in port 1. It is assumed that the display has a character display mode: value 1, mode 1.
Then, to display *login* message, you need to execute following program:
```
set  P 1 # set port to display
devs 1 1 # set write mode
out 'l'
out 'o'
out 'g'
out 'i'
out 'n'
```

Any device can trigger interrupts.

Example. There is a keyboard device (at port 2) that triggers an interrupt by pressing a key.
Let, initial keyboard interrupt address 0. Then, set required address via *devs* by value number 1.
When a key is pressed, keyboard must trigger the interrupt and transmit value of pressed key through input interface. MCPU can read input using the instruction *in*.
Following code will handle keystrokes:
```
set P 2 # set port to keyboard
devs key_int 1 # set interrupt address

# ...

key_int:
        in A # read input to A
        set [pressed_key] A
        iret

pressed_key: 0
```
Each time a key is pressed, the key value will be written to *pressed_key*.

## Stack

**Stack** is the hardware stack structure located in RAM, supported *push* and *pop* operations.
The stack pointer is *S* register. The address of first value on stack is last address of RAM (65535 in decimal or 1111_1111_1111_1111 in binary).

When MCPU starts, the value of *S* is 0. That is, stack pointer points to cell number 0.

When MCPU executes *push* instruction, it decreases *S* by 1 and writes the value to stack top.
Thus, the first *push* execution will write the value to cell 65535, since decrementing of 0 (0000_0000_0000_0000) gives the value 65535 (1111_1111_1111_1111).

When MCPU executes *pop* instruction, it reads the value from stack top and increments *S* by 1.

Initial state of RAM:
```
65534: [ 0000 0000 0000 0000 ]
65535: [ 0000 0000 0000 0000 ]
00000: [ 0000 0000 0000 0000 ] <- S
```
*S* points to cell 0.
After execution *push 5* instruction the state of RAM will be:
```
65534: [ 0000 0000 0000 0000 ]
65535: [ 0000 0000 0000 0101 ] <- S
00000: [ 0000 0000 0000 0000 ]
```
*S* points to cell 65535.
After execution *pop A* instruction, *A* will set to 5 (value of stack top), but the value will still be in RAM and *S* not be point to it.
The state of RAM will be:
```
65534: [ 0000 0000 0000 0000 ]
65535: [ 0000 0000 0000 0101 ]
00000: [ 0000 0000 0000 0000 ] <- S
```

Stack boundaries are not specific and can be limited architecturally or programmatically.

## Description of implementation

Before MCPU starts the program is loaded into the RAM starting at address 0.
When MCPU starts value of all its registers is 0.
Value of all memory cells by default is 0.
MCPU interrupt queue is empty.

Each operation is sequentially executed one after another (if there was no jump).
After execution of the operation register I sets on address of the next operation.
The address of the next operation is the address of the previous operation + size (words) of the previous operation.
MCPU has a set frequency - how many ticks are spent per second.
Each operation spends time in ticks.
Cost of operation is cost of its instruction and cost of its operands.

## Simple assembly language

File extension for MUE assembly language is *.mue*

Single-line comments begin with *#* and multi-line with *##* and end with *##*:
```
# This line a comment and will not be converted to MCPU code

##
        This is a
        multi-line comment.
##
```

Any numeric literals are written to the result code according to rules for representing binary numbers:
```
0 12 -1
```
It will be compiled as (binary):
```
[ 0000 0000 0000 0000 ] (0)
[ 0000 0000 0000 1100 ] (12)
[ 1111 1111 1111 1111 ] (-1 as signed)
```

Decimal literals can end with *d*.
Binary literals must end with *b*.
Octal literals must end with *o*.
Hexadecimal literals can contain numbers *A*, *B*, *C*, *D*, *E*, *F* must end with *x*.
Any numeric literals can contain *_* not at the beginning.

Example:
```
-15d
0010_0000_1010_1111b
107o
A9FFx
```

String literal is encoded to utf-8 and supports escaping characters.
String literal begins with character *'* and ends with character *'*.

Example:
```
'hello'
```
It will be compiled:
```
[ 0000 0000 0110 1000 ] (68x)
[ 0000 0000 0110 0101 ] (65x)
[ 0000 0000 0110 1100 ] (6Cx)
[ 0000 0000 0110 1100 ] (6Cx)
[ 0000 0000 0110 1111 ] (6Fx)
```

Anywhere in code of program, you can create a label with a name that may contain letters or numbers not at the beginning. To create a label, you need to write its name and *:* character.

Example:
```
some_label:
```

Label is a pointer to the place of the program. To use a label you need to write its name.

Example:
```
x: 0 # create label 'x'

set [x] 5 # set 5 to cell of RAM which label 'x' points
```

Processor operations are compiled by instruction number and operand number. If necessary, recorded constant values.

Example:
```
set A [5]
```
It will be compiled:
```
[ 1111 1010 1000 0010 ]
[ 0000 0000 0000 0101 ]
```

## Sample program

Let's write a program for displaying message *Hello Abzal!* on display connected to port 1.
Suppose that display output mode is value 1 to mode 1.
Then the program will be:
```
go main

message: 'Hello Abzal!\n' 0

main:
        set P 1
        devs 1 1
        set C 0

loop:
        set A [message + C]
        iff A
                go end
        
        out A
        inc C
        go loop

end:
        stop 0

```

## Copyright

Copyright © 2019 CRECAT Inc. All Rights Reserved
