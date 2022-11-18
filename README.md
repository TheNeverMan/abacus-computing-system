
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
ACS is designed for abacuses that have 10 wired, each having 10 beads. Its registers are maxium 2 wires long, which means that it operates on 2 digit numbers (0-99). Number in ACS has similar meaning to byte, digit is bit (this will be important in command descriptions).
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
The philosophy of this system is that abacus itself holds all data required for program to continue, so theoretically even if human-operator changed after each command it shouldn't affect program outcome at all.
