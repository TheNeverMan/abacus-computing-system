What is Abacus Computing System?
---------------------------------
![Abacus Picture](https://upload.wikimedia.org/wikipedia/commons/thumb/a/a0/Kugleramme.jpg/800px-Kugleramme.jpg)
An Abacus  
Abacus is calculating tool that is used since ancient times. Although many times it is called grandfather of computers, in reality it is manual arithmometer - a device that can be used to do basic math operations on numbers.  
In this project however I changed abacus into fullblown mechanical computer that mimics behaviour of real computers. You can program your abacus to do any task - it is Turing Complete. Also it has many pros over real electronic computers, such as: 
- small electricity consumption (0 Watts!)
- small price (<2 $)
- actually lightweight and portable (say that to PC users)
- works in any conditions (no restrictions on temperature or humidity)
- easy to repair (no knowledge required)
- can be used by anyone (no skill required).  
It has no downsides too. Now lets move on to the general system architecture.

How does it work? (General Idea)
-----------------

Abacus Computing System architecture is very similar to computer architecture. Abacus must have instructions stored in memory (program), a human that operates it (or anything else that has enough mindpower), and an output.
1. Human reads next instruction from memory
2. Executes it by modifying abacus state and/or memory
3. Go to step 1         

Program execution can stop after special commands or errors. If you haven't guessed yet, both "memory" and "output" are actually pieces of paper with commands written with pen or anything else, no electric-hdd-sdd-flash bullshit here! Only real memory with real data.

How does it work? (Detailed)
---------------------------

ACS is designed for abacuses that have 10 wired, each having 10 beads. Its registers are maxium 2 wires long, which means that it operates on 2 digit numbers (0-99), higher wire represents tenths digit (5 in 54), lower singulars (4 in 54). Number in ACS has similar meaning to byte, digit is bit (this will be important in command descriptions).The philosophy of this system is that abacus itself holds all data required for program to continue, so theoretically even if human-operator changed after each command it shouldn't affect program outcome at all.

Memory Format and Instructions
------------------------------

Data and instructions are stored in rows on memory pages (which are real paper pages, now memory page has a new whole meaning...). Abacus can adress up to 10 pages from 0 to 9, each having 100 rows (from 0-99). On first page in the "disk" (notebook or similar) there should be adress - to which abacus should jump when "booting". ACS adress looks like this:    
(page, row)   
Every row can hold single instruction or single two digit number.

Registers
---------

Wires are divided in to registers, similarily to CPUs. 
Going from top to bottom:  
- First two wires are A register.
- Third and fourth is B register.
- Fifth and sixth is J register.
- Seventh is E register.
- Eight is P register.
- Ninth and tenth is PC register.     

 A and B are general purpose registers that hold 2 digit number.   
 J can serve as general purpose register, but is also used in subroutine jumps.  
 E is Error Register, it holds data about how last command execution went: overflows, underflows, errors... You can not directly access this register.  
 P and PC are program counters - they store address of currently executed command. P is page and PC is row. PC is incremented after command execution. You can not directly access these registers.  
 
 | Register | Direct Read | Direct Write |
 |---------:|-------------|--------------|
 |         A|          yes|           yes|
 |         B|          yes|           yes|
 |         J|          yes|           yes|
 |         E| special commands (conditional jumps)|            no| 
 |    P & PC|           no| special commands (jumps)|
 
 Command Execution & Error Register
 -------------------
 
 Abacus executes command that P and PC point to. After execution PC is incremented (if it overflows back to 0 from 99, P is incremented to, if both P and PC overflow under incrementation they are resetted to 0) and E is set to outcome of command:
 - 0 (R) Command was executed without any errors (success)
 - 1 (O) Arthmetic Overflow (attempt to increment value past 99)
 - 2 (U) Arthmetic Underflow (attempt to decrement value below 0)
 - 3 (RE) Arthmetic Error (Division by 0)
 - 4 (Z) Zero (Zero as result of operation)
 - 5 (N) Arthmetic Rounding (Rounding on division operation)
 - 6 (AE) Adress Error (Invalid Adress or Register)
 - 7 (IE) I/O Error
 - 8 (ME) Memory Error (Value or Command is unreadable)
 - 9 (CE) Command Error (Invalid Command)    

When command triggers CE, execution halts. If command triggered more than one state, one with highest value will be stored in E. When Overflow happens register value is set to 99 (on Underflow it is 0).
 
Detailed Start-Up Procedure
--------------------

1. Reset value of all registers to zero.
2. Prepare one blank piece of paper - it will be the "output".
3. Prepare your program.
4. Find first page of program, find boot address, set P and PC to it.
5. Execute command at (P, PC).
6. Increment PC.
7. Set E to outcome of command.
8. Halt if CE is outcome.
9. Jump to step 5.   

Commands List
------------

ACS is programmed with its own custom pseudoassembly language. Programs look very similar to assembly mnemonic listings. There must be one command per row, and every row should start with its number (row address).
Command is three letter long, after it there should be space separated list of arguments. If you want to add comments, you should separate them from command with `//`.
Example:   
```
(row number) (command) (arguments)  (optional comments)
34 mul J A B //Multiply A * B and store result in J   
35 joo (4,35) //If Overflow happened jump to Row 35 on Page 4
```  

Arguments Notation
-------------

Commands can take 4 types of arguments: address, register, decimal value, character.   

| Name | Notation | Example | Exceptions | Short Name |
|------|----------|---------|------------|------------|
|Register|Register name (A or B or J), eventually specifying the digit (H or L)| B | If is invalid (eg. trying to write to register PC) results in AE, if is unreadable, results in ME| reg |
|Decimal Value| Decimal value from 0 to 99, prefixed with lowercase d| d54 | If out of range (0-99), results with ME. If unreadable, also ME| val |
|Character| Single character in double quotes. | "x" | If unreadable, results in ME | (same as decimal value) |
|Address| Pair of two values separated by comma in brackets. First value is page number, second row. These values can be either: register (value of register is used), decimal value, * (value of P (if considered as page address) or PC (if considered as row address)| (4,56) - Page 4 Row 56, (J, B) - Page Number is value of J, Row Number is value of B, (\*, JH) - Page number is equal to current command page, Row Number is tenths digit of J register| AE when pointing to invalid address, invalid register in address. ME is row in unreadable| addr |

H & L digits respectively mean tenths digit of register or singulars digit of register:   
```
J = 45
JH = 4
JL = 5
```
However during operations H digit is actually treated like tenths digit not singulars!
```
J = 45
A = B + JH => A = B + 40
(3,JH) => (3, 40)
(3,JL) => (3, 5)
Watch out!
```

List of Commands
----------------

`set out(reg, addr) in(val, reg, addr)` 
Sets in to out. When in is one digit of register (eg. JL, AH), the other digit of out is not modified! You can not set incompatibile digits to each other (eg. set JL AH) - this results in ME.    

`jmp addr(addr)`   
Sets P and PC to addr or simply - jumps to addr.    

`inc reg(reg)`   
Increments reg by one.  

`dec reg(reg)`   
Decrements reg by one.   

```
add out(reg) in1(reg) in2(reg)
sub out(reg) in1(reg) in2(reg)
mul out(reg) in1(reg) in2(reg)
div out(reg) in1(reg) in2(reg)
mod out(reg) in1(reg) in2(reg)
```
Performs given arthmetic operation:    
```
add: out = in1 + in2
sub: out = in1 - in2
mul: out = in1 * in2
div: out = in1 / in2
mod: out = in1 % in2
```
All arguments can be same register. This could give following results:   
- O - out is higher than 99
- U - out is lower than 0
- RE - Division by 0
- Z - out is equal 0
- N - out is rounded to whole number after division operation   
 
 Remember about H & L digit behaviour:   
 ```
 J=34
 B=25
 JH + BH => 30 + 20 = 70
 JH + JL => 30 + 4 = 34
 ```
 
 `jsb jump(addr, reg, val)`   
 Jumps to given row address (if addr is provided as argument then value under it is taken) and saves current row number in J register.  
 
 `ret`  
 Sets PC to J,(returns from jsb).  
 
 ```
 joe addr(addr)
 joz addr(addr)
 joo addr(addr)
 jou addr(addr)
 jor addr(addr)
 jon addr(addr)
 ```
 Performs conditional jump based on value in E register. If condition is not met this command is skiped. Important: THIS COMMAND DOES NOT OVERWRITE OR MODIFY E REGISTER AFTER EXECUTION!   
 ```
 joe - if last command result was error (RE, ME, IE)
 joz - if last command result was zero (Z)
 joo - if last command result was overflow
 jou - if last command result was underflow
 jor - if last commands result was success
 jon - if last command result was rounding
 ```
 
  ```
 jse jump(addr, reg, val)
 jsz jump(addr, reg, val)
 jso jump(addr, reg, val)
 jsu jump(addr, reg, val)
 jsr jump(addr, reg, val)
 jsn jump(addr, reg, val)
 ```
 Performs conditional jump to provied row value (like `jsb`), and sets J register to current value of PC, based on value in E register. If condition is not met this command is skiped. Important: THIS COMMAND DOES NOT OVERWRITE OR MODIFY E REGISTER AFTER EXECUTION!   
 ```
 jse - if last command result was error (RE, ME, IE)
 jsz - if last command result was zero (Z)
 jso - if last command result was overflow
 jsu - if last command result was underflow
 jsr - if last commands result was success
 jsn - if last command result was rounding
 ```   
 
 ```
 joq addr(addr) v1(addr,val,reg) v2(addr,val,reg)
 jsq jump(addr,val,reg) v1(addr,val,reg) v2(addr,val,reg)
 ```
 Performs conditional jump (`joq` - like `jmp` and `jsq` - like `jsb`) when v1 = v2  
 
 `inp reg(reg)`  
 Sets reg to value provided by user.  
 
 `out reg(reg)`
 Prints value of reg on output piece of paper.   
 
 `neg reg(reg)`   
 Negates reg, sets its value to its complementary number (eg. 12 => 98. 55 => 44).  
 
 `rot reg(reg)`   
 Rotates reg, reverses its digits (eg. 12 => 21, 55 => 55).  
 
 `mor reg(reg)`
 Equivalent of moving one bead from lower wire to higher (eg. 62 => 71, 55 => 64), causes RE if singulars digit is equal 0 (eg. 10)   
 
 `smr reg(reg)`  
 Similar to above, but if singular digit is equal to 0 adds nine to it (eg. 10 => 29).  
 
 `ror reg(reg)`
 Rotates reg upwards (tenths becomes singulars and singulars becomes 0).   
 
 `rol reg(reg)` 
 Rotates reg downwards (reversed version of above command).   
 
 `sfd out(reg) in(addr,val,reg)`   
 Sets L digit of out to L digit of in.   
 
 `ssd out(reg) in(addr,val,reg)`  
 Sets H digit of out to L digit of in. (Not like normal H & L behaviour).  
 
 `end`   
 Halts program.
 
 `res`
 Restarts abacus.   
 
 `nop`
 Does nothing.  
 
 ```
 adi out(reg) in(reg)
 sui out(reg) in(reg)
 mui out(reg) in(reg)
 dii out(reg) in(reg)
 moi out(reg) in(reg)
 ```
 Performs operation on both digits of in and stores result in out. This command does not behave like normal H & L operations:   
 ```
 J=56
 add J JH JL => 56 = 50 + 6
 adi J J => 11 = 5 + 6
 ```
