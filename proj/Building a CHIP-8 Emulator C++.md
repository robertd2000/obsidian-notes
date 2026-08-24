https://austinmorlan.com/posts/chip8_emulator/#how-does-a-cpu-work

I’ve always loved emulators because they let me play old games that I enjoyed as a kid, so I thought it might be fun to learn how they work and how to build one. My real goal is to build an NES emulator, but after doing some research, I decided to take the advice of the internet and start by building an emulator for the much less complex [CHIP-8](https://en.wikipedia.org/wiki/CHIP-8) instead. It’s a good stepping stone to the NES.

Technically the CHIP-8 was never a real machine at all but instead a virtual machine. People would write programs for this virtual machine, and then an interpreter would decode the instructions. Because it was virtual, the same programs could run on different machines as long as they had an interpreter. What we're going to create is actually one of those interpreters, but we'll just consider it an emulator.

There are a lot of CHIP-8 emulators out there, and a lot of websites about building them, but I noticed a divide between the two. There might be code without a lot of explanation, or explanations without a lot of code. Those are fine for an experienced programmer who can fill in the gaps, but I want to provide a holistic reference for the inexperienced. Not only do I hope to show how to build a simple emulator, but I also hope to teach some low-level computer fundamentals.

The only requirement is a basic understanding of C++.

- [How Does a CPU Work?](https://austinmorlan.com/posts/chip8_emulator/#how-does-a-cpu-work)
- [What is an Emulator?](https://austinmorlan.com/posts/chip8_emulator/#what-is-an-emulator)
- [CHIP-8 Description](https://austinmorlan.com/posts/chip8_emulator/#chip-8-description)
    - [16 8-bit Registers](https://austinmorlan.com/posts/chip8_emulator/#16-8-bit-registers)
    - [4K Bytes of Memory](https://austinmorlan.com/posts/chip8_emulator/#4k-bytes-of-memory)
    - [16-bit Index Register](https://austinmorlan.com/posts/chip8_emulator/#16-bit-index-register)
    - [16-bit Program Counter](https://austinmorlan.com/posts/chip8_emulator/#16-bit-program-counter)
    - [16-level Stack](https://austinmorlan.com/posts/chip8_emulator/#16-level-stack)
    - [8-bit Stack Pointer](https://austinmorlan.com/posts/chip8_emulator/#8-bit-stack-pointer)
    - [8-bit Delay Timer](https://austinmorlan.com/posts/chip8_emulator/#8-bit-delay-timer)
    - [8-bit Sound Timer](https://austinmorlan.com/posts/chip8_emulator/#8-bit-sound-timer)
    - [16 Input Keys](https://austinmorlan.com/posts/chip8_emulator/#16-input-keys)
    - [64x32 Monochrome Display Memory](https://austinmorlan.com/posts/chip8_emulator/#64x32-monochrome-display-memory)
    - [Class Members](https://austinmorlan.com/posts/chip8_emulator/#class-members)
- [Loading a ROM](https://austinmorlan.com/posts/chip8_emulator/#loading-a-rom)
- [Loading the Fonts](https://austinmorlan.com/posts/chip8_emulator/#loading-the-fonts)
- [Random Number Generator](https://austinmorlan.com/posts/chip8_emulator/#random-number-generator)
- [The Instructions](https://austinmorlan.com/posts/chip8_emulator/#the-instructions)
- [Function Pointer Table](https://austinmorlan.com/posts/chip8_emulator/#function-pointer-table)
- [Fetch, Decode, Execute](https://austinmorlan.com/posts/chip8_emulator/#fetch-decode-execute)
- [The Platform Layer](https://austinmorlan.com/posts/chip8_emulator/#the-platform-layer)
- [The Main Loop](https://austinmorlan.com/posts/chip8_emulator/#the-main-loop)
- [Results](https://austinmorlan.com/posts/chip8_emulator/#results)
- [Conclusion](https://austinmorlan.com/posts/chip8_emulator/#conclusion)
- [Appendix: Bits, Bytes, Etc.](https://austinmorlan.com/posts/chip8_emulator/#appendix-bits-bytes-etc)
    - [Binary](https://austinmorlan.com/posts/chip8_emulator/#binary)
    - [Hexadecimal](https://austinmorlan.com/posts/chip8_emulator/#hexadecimal)
    - [Bits and Bytes](https://austinmorlan.com/posts/chip8_emulator/#bits-and-bytes)
    - [AND, OR, NOT, XOR](https://austinmorlan.com/posts/chip8_emulator/#and-or-not-xor)
    - [Bitmasking](https://austinmorlan.com/posts/chip8_emulator/#bitmasking)
    - [Bit Shifting](https://austinmorlan.com/posts/chip8_emulator/#bit-shifting)
- [Source Code](https://austinmorlan.com/posts/chip8_emulator/#source-code)

## How Does a CPU Work?

---

Before we can talk about emulators, we first need a basic understanding of how a CPU works.

A CPU reads instructions from somewhere in memory that tell it what to do. A CPU may be really fast but it’s stupid. You have to be very explicit and logical to get it to do what you want and you do that using a discrete set of instructions.

For example, consider this instruction for the [CPU inside of the original Game Boy](https://gbdev.io/gb-opcodes/optables/): **$C622**. It encodes an operation and relevant data into a number that a machine can read.

Different instructions are encoded in different ways. In this case, the first byte (**$C6**) says **ADD to Register A**, and the second byte (**$22**) is the value that will be added to Register A. So this instruction says **Add $22 to Register A**.

As explained above, the CHIP-8 was not a real physical CPU but instead a virtual machine with its own instruction set, but the same principles apply. For example, consider the CHIP-8 instruction **$7522**. The first byte (**$75**) says **ADD to Register 5** and the second byte (**$22**) is the value to be added to Register 5. So this instruction says **ADD $22 to Register 5**.

**$** and **0x** mean that a number is represented in hexadecimal (hex). That is the primary radix used when dealing with computers at a low-level because it's a much more concise way of presenting large values (like memory addresses).  
  
See the Appendix for more info.

Way back in the day, programmers would write programs in an **assembly language** rather than the high-level languages (like C++) that we use today. Assembly is as low as you can go while still being human readable. An **assembler** would translate their human-readable assembly into the 1s and 0s that the computer could understand.

Keeping with the earlier example, the assembly program would have **ADD V5, $22**, and the assembler would translate that to **$7522**, which the CHIP-8 interpreter can read. The same thing happens today, only we have another layer above assembly in the form of high-level languages.

## What is an Emulator?

---

An emulator reads the original machine code instructions that were assembled for the target machine, interprets them, and then replicates the functionality of the target machine on the host machine. The ROM files contain the instructions, the emulator reads those instructions, and then does work to mimic the original machine.

When an emulator reads the instruction **$7522**, it would emulate the behavior of the CHIP-8 by doing something like this:

|   |   |
|---|---|
|```<br>1<br>```|```cpp<br>registers[5] += 0x22;<br>```|

That’s really all there is to it. When emulating more advanced machines you also have to emulate other components like the graphics processor and the sound chip. The CHIP-8 is a nice starting project because the CPU only has 34 instructions, the graphics are simple monochrome pixels, and the sounds are just a single buzzer tone.

## CHIP-8 Description

---

We’d like to mimic the components of a CHIP-8 in our emulator, so let’s describe them.

### 16 8-bit Registers

A register is a dedicated location on a CPU for storage. Any operations that a CPU does must be done within its registers. A CPU typically only has a few registers, so long-term data is held in memory instead. Operations often involve loading data from memory into registers, operating on those registers, and then storing the result back into memory.

The CHIP-8 has sixteen 8-bit registers, labeled **V0** to **VF**. Each register is able to hold any value from **0x00** to **0xFF**.

Register VF is a bit special. It’s used as a flag to hold information about the result of operations.

### 4K Bytes of Memory

There is relatively little register-space (because it’s expensive), so a computer needs a large chunk of general memory dedicated to holding program instructions, long-term data, and short-term data. It references different locations in that memory using an **address**.

The CHIP-8 has 4096 bytes of memory, meaning the address space is from **0x000** to **0xFFF**.

The address space is segmented into three sections:

- **0x000-0x1FF:** Originally reserved for the CHIP-8 interpreter, but in our modern emulator we will just never write to or read from that area. Except for…
    
- **0x050-0x0A0:** Storage space for the 16 built-in characters (**0** through **F**), which we will need to manually put into our memory because ROMs will be looking for those characters.
    
- **0x200-0xFFF:** Instructions from the ROM will be stored starting at 0x200, and anything left after the ROM’s space is free to use.
    

### 16-bit Index Register

The Index Register is a special register used to store memory addresses for use in operations. It’s a 16-bit register because the maximum memory address (0xFFF) is too big for an 8-bit register.

### 16-bit Program Counter

As mentioned earlier, the actual program instructions are stored in memory starting at address **0x200**. The CPU needs a way of keeping track of which instruction to execute next.

The Program Counter (PC) is a special register that holds the address of the next instruction to execute. Again, it’s 16 bits because it has to be able to hold the maximum memory address (0xFFF).

An instruction is two bytes but memory is addressed as a single byte, so when we fetch an instruction from memory we need to fetch a byte from PC and a byte from PC+1 and connect them into a single value. We then increment the PC by 2 because We have to increment the PC before we execute any instructions because some instructions will manipulate the PC to control program flow. Some will add to the PC, some will subtract from it, and some will change it completely.

### 16-level Stack

A stack is a way for a CPU to keep track of the order of execution when it calls into functions. There is an instruction (**CALL**) that will cause the CPU to begin executing instructions in a different region of the program. When the program reaches another instruction (**RET**), it must be able to go back to where it was when it hit the CALL instruction. The stack holds the PC value when the CALL instruction was executed, and the RETURN statement pulls that address from the stack and puts it back into the PC so the CPU will execute it on the next cycle.

The CHIP-8 has 16 levels of stack, meaning it can hold 16 different PCs. Multiple levels allow for one function to call another function and so on, until they all return to the original caller site.

Putting a PC onto the stack is called **pushing** and pulling a PC off of the stack is called **popping**.

### 8-bit Stack Pointer

Similar to how the PC is used to keep track of where in memory the CPU is executing, we need a Stack Pointer (SP) to tell us where in the 16-levels of stack our most recent value was placed (i.e, the top).

We only need 8 bits for our stack pointer because the stack will be represented as an array, so our stack pointer can just be an index into that array. We only need sixteen indices then, which a single byte can manage.

When we pop a value off the stack, we won’t actually delete it from the array but instead just copy the value and decrement the SP so it “points” to the previous value.

Let’s go through an example of how the stack works. Here is the example program we’re running, with the address of the instruction on the left and the actual instruction on the right. Don’t worry about the **LD** instructions, just focus on the __CALL__s and the __RET__s. A **JMP** is like a CALL, but it doesn’t put anything onto the stack.

|   |   |
|---|---|
|```<br>1<br>2<br>3<br>4<br>5<br>6<br>7<br>8<br>```|```text<br>$200: CALL $208<br>$202: JMP $20E<br>$204: LD V1, $1<br>$206: RET<br>$208: LD V3, $3<br>$20A: CALL $204<br>$20C: RET<br>$20E: LD V4, $4<br>```|

Initially the stack pointer starts at 0, indicating the top of the stack, and the stack itself is just an array filled with zeroes. As we execute CALLs and RETs, we’ll see what the stack does.

|   |   |
|---|---|
|```<br> 1<br> 2<br> 3<br> 4<br> 5<br> 6<br> 7<br> 8<br> 9<br>10<br>11<br>12<br>13<br>14<br>15<br>16<br>17<br>18<br>19<br>20<br>21<br>22<br>23<br>24<br>25<br>26<br>27<br>28<br>29<br>30<br>31<br>32<br>33<br>34<br>35<br>```|```text<br>                   PC: $202          PC: $20A          PC: $20C          PC: $206           PC: $208          PC: $20E         PC: $204          PC: $210<br>                $200: CALL $208   $208: LD V3, $3   $20A: CALL $204   $204: LD V1, $1      $206: RET         $20C: RET        $202: JMP $20E    $20E: JMP<br> --------          --------          --------          --------          --------          --------          --------          --------          --------<br>\|00\|0x000\| <- SP  \|00\|0x000\| <- SP  \|00\|0x202\|        \|00\|0x202\|        \|00\|0x202\|        \|00\|0x202\|        \|00\|0x202\|        \|00\|0x202\| <- SP  \|00\|0x202\| <- SP<br>\|--------\|        \|--------\|        \|--------\|        \|--------\|        \|--------\|        \|--------\|        \|--------\|        \|--------\|        \|--------\|<br>\|01\|0x000\|        \|01\|0x000\|        \|01\|0x000\| <- SP  \|01\|0x000\| <- SP  \|01\|0x20C\|        \|01\|0x20C\|        \|01\|0x20C\| <- SP  \|01\|0x20C\|        \|01\|0x20C\|<br>\|--------\|        \|--------\|        \|--------\|        \|--------\|        \|--------\|        \|--------\|        \|--------\|        \|--------\|        \|--------\|<br>\|02\|0x000\|        \|02\|0x000\|        \|02\|0x000\|        \|02\|0x000\|        \|02\|0x000\| <- SP  \|02\|0x000\| <- SP  \|02\|0x000\|        \|02\|0x000\|        \|02\|0x000\|<br>\|--------\|        \|--------\|        \|--------\|        \|--------\|        \|--------\|        \|--------\|        \|--------\|        \|--------\|        \|--------\|<br>\|03\|0x000\|        \|03\|0x000\|        \|03\|0x000\|        \|03\|0x000\|        \|03\|0x000\|        \|03\|0x000\|        \|03\|0x000\|        \|03\|0x000\|        \|03\|0x000\|<br>\|--------\|        \|--------\|        \|--------\|        \|--------\|        \|--------\|        \|--------\|        \|--------\|        \|--------\|        \|--------\|<br>\|04\|0x000\|        \|04\|0x000\|        \|04\|0x000\|        \|04\|0x000\|        \|04\|0x000\|        \|04\|0x000\|        \|04\|0x000\|        \|04\|0x000\|        \|04\|0x000\|<br>\|--------\|        \|--------\|        \|--------\|        \|--------\|        \|--------\|        \|--------\|        \|--------\|        \|--------\|        \|--------\|<br>\|05\|0x000\|        \|05\|0x000\|        \|05\|0x000\|        \|05\|0x000\|        \|05\|0x000\|        \|05\|0x000\|        \|05\|0x000\|        \|05\|0x000\|        \|05\|0x000\|<br>\|--------\|        \|--------\|        \|--------\|        \|--------\|        \|--------\|        \|--------\|        \|--------\|        \|--------\|        \|--------\|<br>\|06\|0x000\|        \|06\|0x000\|        \|06\|0x000\|        \|06\|0x000\|        \|06\|0x000\|        \|06\|0x000\|        \|06\|0x000\|        \|06\|0x000\|        \|06\|0x000\|<br>\|--------\|        \|--------\|        \|--------\|        \|--------\|        \|--------\|        \|--------\|        \|--------\|        \|--------\|        \|--------\|<br>\|07\|0x000\|        \|07\|0x000\|        \|07\|0x000\|        \|07\|0x000\|        \|07\|0x000\|        \|07\|0x000\|        \|07\|0x000\|        \|07\|0x000\|        \|07\|0x000\|<br>\|--------\|        \|--------\|        \|--------\|        \|--------\|        \|--------\|        \|--------\|        \|--------\|        \|--------\|        \|--------\|<br>\|08\|0x000\|        \|08\|0x000\|        \|08\|0x000\|        \|08\|0x000\|        \|08\|0x000\|        \|08\|0x000\|        \|08\|0x000\|        \|08\|0x000\|        \|08\|0x000\|<br>\|--------\|        \|--------\|        \|--------\|        \|--------\|        \|--------\|        \|--------\|        \|--------\|        \|--------\|        \|--------\|<br>\|09\|0x000\|        \|09\|0x000\|        \|09\|0x000\|        \|09\|0x000\|        \|09\|0x000\|        \|09\|0x000\|        \|09\|0x000\|        \|09\|0x000\|        \|09\|0x000\|<br>\|--------\|        \|--------\|        \|--------\|        \|--------\|        \|--------\|        \|--------\|        \|--------\|        \|--------\|        \|--------\|<br>\|10\|0x000\|        \|10\|0x000\|        \|10\|0x000\|        \|10\|0x000\|        \|10\|0x000\|        \|10\|0x000\|        \|10\|0x000\|        \|10\|0x000\|        \|10\|0x000\|<br>\|--------\|        \|--------\|        \|--------\|        \|--------\|        \|--------\|        \|--------\|        \|--------\|        \|--------\|        \|--------\|<br>\|11\|0x000\|        \|11\|0x000\|        \|11\|0x000\|        \|11\|0x000\|        \|11\|0x000\|        \|11\|0x000\|        \|11\|0x000\|        \|11\|0x000\|        \|11\|0x000\|<br>\|--------\|        \|--------\|        \|--------\|        \|--------\|        \|--------\|        \|--------\|        \|--------\|        \|--------\|        \|--------\|<br>\|12\|0x000\|        \|12\|0x000\|        \|12\|0x000\|        \|12\|0x000\|        \|12\|0x000\|        \|12\|0x000\|        \|12\|0x000\|        \|12\|0x000\|        \|12\|0x000\|<br>\|--------\|        \|--------\|        \|--------\|        \|--------\|        \|--------\|        \|--------\|        \|--------\|        \|--------\|        \|--------\|<br>\|13\|0x000\|        \|13\|0x000\|        \|13\|0x000\|        \|13\|0x000\|        \|13\|0x000\|        \|13\|0x000\|        \|13\|0x000\|        \|13\|0x000\|        \|13\|0x000\|<br>\|--------\|        \|--------\|        \|--------\|        \|--------\|        \|--------\|        \|--------\|        \|--------\|        \|--------\|        \|--------\|<br>\|14\|0x000\|        \|14\|0x000\|        \|14\|0x000\|        \|14\|0x000\|        \|14\|0x000\|        \|14\|0x000\|        \|14\|0x000\|        \|14\|0x000\|        \|14\|0x000\|<br>\|--------\|        \|--------\|        \|--------\|        \|--------\|        \|--------\|        \|--------\|        \|--------\|        \|--------\|        \|--------\|<br>\|15\|0x000\|        \|15\|0x000\|        \|15\|0x000\|        \|15\|0x000\|        \|15\|0x000\|        \|15\|0x000\|        \|15\|0x000\|        \|15\|0x000\|        \|15\|0x000\|<br> --------          --------          --------          --------          --------          --------          --------          --------          --------<br>```|

With each CALL, the current PC (which was previously incremented to point to the _next_ instruction) is placed where the SP was pointing, and the SP is incremented.

With each RET, the stack pointer is decremented by one and the address that it’s pointing to is put into the PC for execution.

The actual order of execution looks like this:

|   |   |
|---|---|
|```<br>1<br>2<br>3<br>4<br>5<br>6<br>7<br>8<br>```|```text<br>$200: CALL $208 -> PC += 2 = $202 \| SP = 0 \| Put $202 in stack[0] and ++SP = 1    \| PC = $208 \| Next cycle at PC = $208<br>$208: LD V3, $3 -> PC += 2 = $20A \| SP = 1 \| Do not modify stack or SP            \| PC = $20A \| Next cycle at PC = $20A<br>$20A: CALL $204 -> PC += 2 = $20C \| SP = 1 \| Put $20C on stack[1] and ++SP = 2    \| PC = $204 \| Next cycle at PC = $204<br>$204: LD V1, $1 -> PC += 2 = $206 \| SP = 2 \| Do not modify stack or SP            \| PC = $206 \| Next cycle at PC = $206<br>$206: RET       -> PC += 2 = $208 \| SP = 2 \| --SP = 1 and Pull $20C from stack[1] \| PC = $20C \| Next cycle at PC = $20C<br>$20C: RET       -> PC += 2 = $20E \| SP = 0 \| --SP = 0 and Pull $202 from stack[0] \| PC = $202 \| Next cycle at PC = $202<br>$202: JMP $20E  -> PC += 2 = $204 \| SP = 0 \| Do not modify stack or SP            \| PC = $204 \| Next cycle at PC = $204<br>$20E: LD V4, $4 -> PC += 2 = $210 \| SP = 0 \| Do not modify stack or SP            \| PC = $210 \| Next cycle at PC = $210<br>```|

### 8-bit Delay Timer

The CHIP-8 has a simple timer used for timing. If the timer value is zero, it stays zero. If it is loaded with a value, it will decrement at a rate of 60Hz.

Rather than making sure that the delay timer actually decrements at a rate of 60Hz, I just decrement it at whatever rate we have the cycle clock set to (discussed later) which has worked fine for all the games I’ve tested.

### 8-bit Sound Timer

The CHIP-8 also has another simple timer used for sound. Its behavior is the same (decrementing at 60Hz if non-zero), but a single tone will buzz when it’s non-zero. Programmers used this for simple sound emission.

While I do have a sound timer in my implementation, I opted to not bother with making the application actually emit any sound. See [here](https://stackoverflow.com/a/45002609) for a way to generate a tone with SDL.

### 16 Input Keys

The CHIP-8 has 16 input keys that match the first 16 hex values: **0** through **F**. Each key is either pressed or not pressed.

I used the input mapping recommended [here](http://www.multigesture.net/articles/how-to-write-an-emulator-chip-8-interpreter/).

|   |   |
|---|---|
|```<br> 1<br> 2<br> 3<br> 4<br> 5<br> 6<br> 7<br> 8<br> 9<br>10<br>```|```text<br>Keypad       Keyboard<br>+-+-+-+-+    +-+-+-+-+<br>\|1\|2\|3\|C\|    \|1\|2\|3\|4\|<br>+-+-+-+-+    +-+-+-+-+<br>\|4\|5\|6\|D\|    \|Q\|W\|E\|R\|<br>+-+-+-+-+ => +-+-+-+-+<br>\|7\|8\|9\|E\|    \|A\|S\|D\|F\|<br>+-+-+-+-+    +-+-+-+-+<br>\|A\|0\|B\|F\|    \|Z\|X\|C\|V\|<br>+-+-+-+-+    +-+-+-+-+<br>```|

### 64x32 Monochrome Display Memory

The CHIP-8 has an additional memory buffer used for storing the graphics to display. It is 64 pixels wide and 32 pixels high. Each pixel is either on or off, so only two colors can be represented.

Understanding and then emulating its operation is probably the most challenging part of the entire project (but still very easy compared to the NES).

I’ll cover the exact implementation of the draw instruction down below, but first let’s cover how the drawing works. As mentioned, a pixel can be either on or off. In our case, we’ll use a **uint32** for each pixel to make it easy to use with SDL (discussed later), so on is **0xFFFFFFFF** and off is **0x00000000**.

The draw instruction iterates over each pixel in a sprite and XORs the sprite pixel with the display pixel.

- Sprite Pixel Off XOR Display Pixel Off = Display Pixel Off
- Sprite Pixel Off XOR Display Pixel On = Display Pixel On
- Sprite Pixel On XOR Display Pixel Off = Display Pixel On
- Sprite Pixel On XOR Display Pixel On = Display Pixel Off

In other words, a display pixel can be set or unset with a sprite. This is often done to only update a specific part of the screen. If you knew you had drawn a sprite at (X,Y) and you now want to draw it at (X+1,Y+1), you could first issue a draw command again at (X,Y) which would erase the sprite, and then you could issue another draw command at (X+1,Y+1) to draw it in the new location. This is why moving objects in CHIP-8 games flicker.

As an example, let’s pretend we have a blank screen that is 16x10.

[![](https://austinmorlan.com/posts/chip8_emulator/media/draw_01.png)](https://austinmorlan.com/posts/chip8_emulator/media/draw_01.png)

We draw a 10x4 sprite at (1,1), so it extends from (1,1) to (10,4).

[![](https://austinmorlan.com/posts/chip8_emulator/media/draw_02.png)](https://austinmorlan.com/posts/chip8_emulator/media/draw_02.png)

We then draw an 8x2 sprite at (6,6), so it extends from (6,6) to (13,7).

[![](https://austinmorlan.com/posts/chip8_emulator/media/draw_03.png)](https://austinmorlan.com/posts/chip8_emulator/media/draw_03.png)

If we then draw a 3x4 sprite at (7,3), it would cut a piece out of each of them and draw a line in the gap. The overlapping pixels would turn off (on XOR on = off), and the off pixels would turn on (off XOR on = on).

[![](https://austinmorlan.com/posts/chip8_emulator/media/draw_04.png)](https://austinmorlan.com/posts/chip8_emulator/media/draw_04.png)

Some references I’ve read say that a sprite should wrap around to the other side of the screen if attempting to draw off-screen, so that’s what we’ll do.

### Class Members

Given the aforementioned components, our class data could look like this:

|   |   |
|---|---|
|```<br> 1<br> 2<br> 3<br> 4<br> 5<br> 6<br> 7<br> 8<br> 9<br>10<br>11<br>12<br>13<br>14<br>15<br>16<br>17<br>```|```cpp<br>#include <cstdint><br><br>class Chip8<br>{<br>public:<br>	uint8_t registers[16]{};<br>	uint8_t memory[4096]{};<br>	uint16_t index{};<br>	uint16_t pc{};<br>	uint16_t stack[16]{};<br>	uint8_t sp{};<br>	uint8_t delayTimer{};<br>	uint8_t soundTimer{};<br>	uint8_t keypad[16]{};<br>	uint32_t video[64 * 32]{};<br>	uint16_t opcode;<br>};<br>```|

## Loading a ROM

---

Before we can execute instructions, we need to have those instructions in memory, so we’ll need a function that loads the contents of a ROM file.

|   |   |
|---|---|
|```<br> 1<br> 2<br> 3<br> 4<br> 5<br> 6<br> 7<br> 8<br> 9<br>10<br>11<br>12<br>13<br>14<br>15<br>16<br>17<br>18<br>19<br>20<br>21<br>22<br>23<br>24<br>25<br>26<br>27<br>28<br>29<br>30<br>```|```cpp<br>#include <fstream><br><br>const unsigned int START_ADDRESS = 0x200;<br><br>void Chip8::LoadROM(char const* filename)<br>{<br>	// Open the file as a stream of binary and move the file pointer to the end<br>	std::ifstream file(filename, std::ios::binary \| std::ios::ate);<br><br>	if (file.is_open())<br>	{<br>		// Get size of file and allocate a buffer to hold the contents<br>		std::streampos size = file.tellg();<br>		char* buffer = new char[size];<br><br>		// Go back to the beginning of the file and fill the buffer<br>		file.seekg(0, std::ios::beg);<br>		file.read(buffer, size);<br>		file.close();<br><br>		// Load the ROM contents into the Chip8's memory, starting at 0x200<br>		for (long i = 0; i < size; ++i)<br>		{<br>			memory[START_ADDRESS + i] = buffer[i];<br>		}<br><br>		// Free the buffer<br>		delete[] buffer;<br>	}<br>}<br>```|

As mentioned earlier, the Chip8’s memory from 0x000 to 0x1FF is reserved, so the ROM instructions must start at 0x200.

We also want to initially set the PC to 0x200 in the constructor because that will be the first instruction executed.

|   |   |
|---|---|
|```<br>1<br>2<br>3<br>4<br>5<br>```|```cpp<br>Chip8::Chip8()<br>{<br>	// Initialize PC<br>	pc = START_ADDRESS;<br>}<br>```|

## Loading the Fonts

---

There are sixteen characters that ROMs expected at a certain location so they can write characters to the screen, so we need to put those characters into memory.

The characters are examples of **sprites**, which we’ll see more of later. Each character sprite is five bytes.

The character F, for example, is **0xF0, 0x80, 0xF0, 0x80, 0x80**. Take a look at the binary representation:

|   |   |
|---|---|
|```<br>1<br>2<br>3<br>4<br>5<br>```|```text<br>11110000<br>10000000<br>11110000<br>10000000<br>10000000<br>```|

Can you see it? Each bit represents a pixel, where a 1 means the pixel is on and a 0 means the pixel is off.

We need an array of these bytes to load into memory. There are 16 characters at 5 bytes each, so we need an array of 80 bytes.

|   |   |
|---|---|
|```<br> 1<br> 2<br> 3<br> 4<br> 5<br> 6<br> 7<br> 8<br> 9<br>10<br>11<br>12<br>13<br>14<br>15<br>16<br>17<br>18<br>19<br>20<br>21<br>```|```cpp<br>const unsigned int FONTSET_SIZE = 80;<br><br>uint8_t fontset[FONTSET_SIZE] =<br>{<br>	0xF0, 0x90, 0x90, 0x90, 0xF0, // 0<br>	0x20, 0x60, 0x20, 0x20, 0x70, // 1<br>	0xF0, 0x10, 0xF0, 0x80, 0xF0, // 2<br>	0xF0, 0x10, 0xF0, 0x10, 0xF0, // 3<br>	0x90, 0x90, 0xF0, 0x10, 0x10, // 4<br>	0xF0, 0x80, 0xF0, 0x10, 0xF0, // 5<br>	0xF0, 0x80, 0xF0, 0x90, 0xF0, // 6<br>	0xF0, 0x10, 0x20, 0x40, 0x40, // 7<br>	0xF0, 0x90, 0xF0, 0x90, 0xF0, // 8<br>	0xF0, 0x90, 0xF0, 0x10, 0xF0, // 9<br>	0xF0, 0x90, 0xF0, 0x90, 0x90, // A<br>	0xE0, 0x90, 0xE0, 0x90, 0xE0, // B<br>	0xF0, 0x80, 0x80, 0x80, 0xF0, // C<br>	0xE0, 0x90, 0x90, 0x90, 0xE0, // D<br>	0xF0, 0x80, 0xF0, 0x80, 0xF0, // E<br>	0xF0, 0x80, 0xF0, 0x80, 0x80  // F<br>};<br>```|

We can then load them into memory starting at **0x50** in our constructor.

|   |   |
|---|---|
|```<br> 1<br> 2<br> 3<br> 4<br> 5<br> 6<br> 7<br> 8<br> 9<br>10<br>11<br>12<br>```|```cpp<br>const unsigned int FONTSET_START_ADDRESS = 0x50;<br><br>Chip8::Chip8()<br>{<br>	[...]<br><br>	// Load fonts into memory<br>	for (unsigned int i = 0; i < FONTSET_SIZE; ++i)<br>	{<br>		memory[FONTSET_START_ADDRESS + i] = fontset[i];<br>	}<br>}<br>```|

## Random Number Generator

---

There is an instruction which places a random number into a register. With physical hardware this could be achieved by, reading the value from a noisy disconnected pin or using a dedicated RNG chip, but we’ll just use C++’s built in random facilities.

We’ll need to add two member variables and seed the RNG in the constructor. A simple seed is the system clock.

|   |   |
|---|---|
|```<br> 1<br> 2<br> 3<br> 4<br> 5<br> 6<br> 7<br> 8<br> 9<br>10<br>11<br>12<br>13<br>14<br>15<br>16<br>17<br>18<br>19<br>20<br>```|```cpp<br>#include <chrono><br>#include <random><br><br>[...]<br><br>class Chip8<br>{<br>	Chip8()<br>		: randGen(std::chrono::system_clock::now().time_since_epoch().count())<br>	{<br>		[...]<br><br>		// Initialize RNG<br>		randByte = std::uniform_int_distribution<uint8_t>(0, 255U);<br>	}<br><br>	[...]<br><br>	std::default_random_engine randGen;<br>	std::uniform_int_distribution<uint8_t> randByte;<br>```|

We can then get a random number between 0 and 255 by using **randByte**.

## The Instructions

---

The CHIP-8 has 34 instructions that we need to emulate.

I’ll show the opcode and its description, add any needed helpful info, and show an implementation in code. The opcode list and descriptions are taken from [here](http://devernay.free.fr/hacks/chip8/C8TECH10.HTM).

There are differing opinions about how certain instructions should behave. The source I used was the link just referenced (Cowgod), but there is a second reference [here](http://mattmik.com/files/chip8/mastering/chip8.html) with notable differences in 8xy6, 8xye, Fx55, and Fx65. I implemented both and the games I played worked the same in both cases, but the test ROMs (discussed later) failed.  
  
It's likely that the test ROMs were built referencing the same possibly erroneous reference, which is why they fail when using the actual correct implementation. I'm not going to worry about it, but feel free to use the second reference instead if you like. Just be aware that the test ROMs used at the end will fail for you.  
  
See [here](https://old.reddit.com/r/programming/comments/3ca4ry/writing_a_chip8_interpreteremulator_in_c14_10/csu7w8k/) for more info.

**00E0: CLS**

_Clear the display._

We can simply set the entire video buffer to zeroes.

|   |   |
|---|---|
|```<br>1<br>2<br>3<br>4<br>```|```cpp<br>void Chip8::OP_00E0()<br>{<br>	memset(video, 0, sizeof(video));<br>}<br>```|

**00EE: RET**

_Return from a subroutine._

The top of the stack has the address of one instruction past the one that called the subroutine, so we can put that back into the PC. Note that this overwrites our preemptive **pc += 2** earlier.

|   |   |
|---|---|
|```<br>1<br>2<br>3<br>4<br>5<br>```|```cpp<br>void Chip8::OP_00EE()<br>{<br>	--sp;<br>	pc = stack[sp];<br>}<br>```|

**1nnn: JP addr**

_Jump to location nnn._

_The interpreter sets the program counter to nnn._

A jump doesn’t remember its origin, so no stack interaction required.

|   |   |
|---|---|
|```<br>1<br>2<br>3<br>4<br>5<br>6<br>```|```cpp<br>void Chip8::OP_1nnn()<br>{<br>	uint16_t address = opcode & 0x0FFFu;<br><br>	pc = address;<br>}<br>```|

**2nnn - CALL addr**

_Call subroutine at nnn._

When we call a subroutine, we want to return eventually, so we put the current PC onto the top of the stack. Remember that we did **pc += 2** in **Cycle()**, so the current PC holds the next instruction after this CALL, which is correct. We don’t want to return to the CALL instruction because it would be an infinite loop of CALLs and RETs.

|   |   |
|---|---|
|```<br>1<br>2<br>3<br>4<br>5<br>6<br>7<br>8<br>```|```cpp<br>void Chip8::OP_2nnn()<br>{<br>	uint16_t address = opcode & 0x0FFFu;<br><br>	stack[sp] = pc;<br>	++sp;<br>	pc = address;<br>}<br>```|

**3xkk - SE Vx, byte**

_Skip next instruction if Vx = kk._

Since our PC has already been incremented by 2 in **Cycle()**, we can just increment by 2 again to skip the next instruction.

|   |   |
|---|---|
|```<br> 1<br> 2<br> 3<br> 4<br> 5<br> 6<br> 7<br> 8<br> 9<br>10<br>```|```cpp<br>void Chip8::OP_3xkk()<br>{<br>	uint8_t Vx = (opcode & 0x0F00u) >> 8u;<br>	uint8_t byte = opcode & 0x00FFu;<br><br>	if (registers[Vx] == byte)<br>	{<br>		pc += 2;<br>	}<br>}<br>```|

**4xkk - SNE Vx, byte**

_Skip next instruction if Vx != kk._

Since our PC has already been incremented by 2 in **Cycle()**, we can just increment by 2 again to skip the next instruction.

|   |   |
|---|---|
|```<br> 1<br> 2<br> 3<br> 4<br> 5<br> 6<br> 7<br> 8<br> 9<br>10<br>```|```cpp<br>void Chip8::OP_4xkk()<br>{<br>	uint8_t Vx = (opcode & 0x0F00u) >> 8u;<br>	uint8_t byte = opcode & 0x00FFu;<br><br>	if (registers[Vx] != byte)<br>	{<br>		pc += 2;<br>	}<br>}<br>```|

**5xy0 - SE Vx, Vy**

_Skip next instruction if Vx = Vy._

Since our PC has already been incremented by 2 in **Cycle()**, we can just increment by 2 again to skip the next instruction.

|   |   |
|---|---|
|```<br> 1<br> 2<br> 3<br> 4<br> 5<br> 6<br> 7<br> 8<br> 9<br>10<br>```|```cpp<br>void Chip8::OP_5xy0()<br>{<br>	uint8_t Vx = (opcode & 0x0F00u) >> 8u;<br>	uint8_t Vy = (opcode & 0x00F0u) >> 4u;<br><br>	if (registers[Vx] == registers[Vy])<br>	{<br>		pc += 2;<br>	}<br>}<br>```|

**6xkk - LD Vx, byte**

_Set Vx = kk._

|   |   |
|---|---|
|```<br>1<br>2<br>3<br>4<br>5<br>6<br>7<br>```|```cpp<br>void Chip8::OP_6xkk()<br>{<br>	uint8_t Vx = (opcode & 0x0F00u) >> 8u;<br>	uint8_t byte = opcode & 0x00FFu;<br><br>	registers[Vx] = byte;<br>}<br>```|

**7xkk - ADD Vx, byte**

_Set Vx = Vx + kk._

|   |   |
|---|---|
|```<br>1<br>2<br>3<br>4<br>5<br>6<br>7<br>```|```cpp<br>void Chip8::OP_7xkk()<br>{<br>	uint8_t Vx = (opcode & 0x0F00u) >> 8u;<br>	uint8_t byte = opcode & 0x00FFu;<br><br>	registers[Vx] += byte;<br>}<br>```|

**8xy0 - LD Vx, Vy**

_Set Vx = Vy._

|   |   |
|---|---|
|```<br>1<br>2<br>3<br>4<br>5<br>6<br>7<br>```|```cpp<br>void Chip8::OP_8xy0()<br>{<br>	uint8_t Vx = (opcode & 0x0F00u) >> 8u;<br>	uint8_t Vy = (opcode & 0x00F0u) >> 4u;<br><br>	registers[Vx] = registers[Vy];<br>}<br>```|

**8xy1 - OR Vx, Vy**

_Set Vx = Vx OR Vy._

|   |   |
|---|---|
|```<br>1<br>2<br>3<br>4<br>5<br>6<br>7<br>```|```cpp<br>void Chip8::OP_8xy1()<br>{<br>	uint8_t Vx = (opcode & 0x0F00u) >> 8u;<br>	uint8_t Vy = (opcode & 0x00F0u) >> 4u;<br><br>	registers[Vx] \|= registers[Vy];<br>}<br>```|

**8xy2 - AND Vx, Vy**

_Set Vx = Vx AND Vy._

|   |   |
|---|---|
|```<br>1<br>2<br>3<br>4<br>5<br>6<br>7<br>```|```cpp<br>void Chip8::OP_8xy2()<br>{<br>	uint8_t Vx = (opcode & 0x0F00u) >> 8u;<br>	uint8_t Vy = (opcode & 0x00F0u) >> 4u;<br><br>	registers[Vx] &= registers[Vy];<br>}<br>```|

**8xy3 - XOR Vx, Vy**

_Set Vx = Vx XOR Vy._

|   |   |
|---|---|
|```<br>1<br>2<br>3<br>4<br>5<br>6<br>7<br>```|```cpp<br>void Chip8::OP_8xy3()<br>{<br>	uint8_t Vx = (opcode & 0x0F00u) >> 8u;<br>	uint8_t Vy = (opcode & 0x00F0u) >> 4u;<br><br>	registers[Vx] ^= registers[Vy];<br>}<br>```|

**8xy4 - ADD Vx, Vy**

_Set Vx = Vx + Vy, set VF = carry._

_The values of Vx and Vy are added together. If the result is greater than 8 bits (i.e., > 255,) VF is set to 1, otherwise 0. Only the lowest 8 bits of the result are kept, and stored in Vx._

This is an ADD with an overflow flag. If the sum is greater than what can fit into a byte (255), register VF will be set to 1 as a flag.

|   |   |
|---|---|
|```<br> 1<br> 2<br> 3<br> 4<br> 5<br> 6<br> 7<br> 8<br> 9<br>10<br>11<br>12<br>13<br>14<br>15<br>16<br>17<br>18<br>```|```cpp<br>void Chip8::OP_8xy4()<br>{<br>	uint8_t Vx = (opcode & 0x0F00u) >> 8u;<br>	uint8_t Vy = (opcode & 0x00F0u) >> 4u;<br><br>	uint16_t sum = registers[Vx] + registers[Vy];<br><br>	if (sum > 255U)<br>	{<br>		registers[0xF] = 1;<br>	}<br>	else<br>	{<br>		registers[0xF] = 0;<br>	}<br><br>	registers[Vx] = sum & 0xFFu;<br>}<br>```|

**8xy5 - SUB Vx, Vy**

_Set Vx = Vx - Vy, set VF = NOT borrow._

_If Vx > Vy, then VF is set to 1, otherwise 0. Then Vy is subtracted from Vx, and the results stored in Vx._

|   |   |
|---|---|
|```<br> 1<br> 2<br> 3<br> 4<br> 5<br> 6<br> 7<br> 8<br> 9<br>10<br>11<br>12<br>13<br>14<br>15<br>16<br>```|```cpp<br>void Chip8::OP_8xy5()<br>{<br>	uint8_t Vx = (opcode & 0x0F00u) >> 8u;<br>	uint8_t Vy = (opcode & 0x00F0u) >> 4u;<br><br>	if (registers[Vx] > registers[Vy])<br>	{<br>		registers[0xF] = 1;<br>	}<br>	else<br>	{<br>		registers[0xF] = 0;<br>	}<br><br>	registers[Vx] -= registers[Vy];<br>}<br>```|

**8xy6 - SHR Vx**

_Set Vx = Vx SHR 1._

_If the least-significant bit of Vx is 1, then VF is set to 1, otherwise 0. Then Vx is divided by 2._

A right shift is performed (division by 2), and the least significant bit is saved in Register VF.

|   |   |
|---|---|
|```<br>1<br>2<br>3<br>4<br>5<br>6<br>7<br>8<br>9<br>```|```cpp<br>void Chip8::OP_8xy6()<br>{<br>	uint8_t Vx = (opcode & 0x0F00u) >> 8u;<br><br>	// Save LSB in VF<br>	registers[0xF] = (registers[Vx] & 0x1u);<br><br>	registers[Vx] >>= 1;<br>}<br>```|

**8xy7 - SUBN Vx, Vy**

_Set Vx = Vy - Vx, set VF = NOT borrow._

_If Vy > Vx, then VF is set to 1, otherwise 0. Then Vx is subtracted from Vy, and the results stored in Vx._

|   |   |
|---|---|
|```<br> 1<br> 2<br> 3<br> 4<br> 5<br> 6<br> 7<br> 8<br> 9<br>10<br>11<br>12<br>13<br>14<br>15<br>16<br>```|```cpp<br>void Chip8::OP_8xy7()<br>{<br>	uint8_t Vx = (opcode & 0x0F00u) >> 8u;<br>	uint8_t Vy = (opcode & 0x00F0u) >> 4u;<br><br>	if (registers[Vy] > registers[Vx])<br>	{<br>		registers[0xF] = 1;<br>	}<br>	else<br>	{<br>		registers[0xF] = 0;<br>	}<br><br>	registers[Vx] = registers[Vy] - registers[Vx];<br>}<br>```|

**8xyE - SHL Vx {, Vy}**

_Set Vx = Vx SHL 1._

_If the most-significant bit of Vx is 1, then VF is set to 1, otherwise to 0. Then Vx is multiplied by 2._

A left shift is performed (multiplication by 2), and the most significant bit is saved in Register VF.

|   |   |
|---|---|
|```<br>1<br>2<br>3<br>4<br>5<br>6<br>7<br>8<br>9<br>```|```cpp<br>void Chip8::OP_8xyE()<br>{<br>	uint8_t Vx = (opcode & 0x0F00u) >> 8u;<br><br>	// Save MSB in VF<br>	registers[0xF] = (registers[Vx] & 0x80u) >> 7u;<br><br>	registers[Vx] <<= 1;<br>}<br>```|

**9xy0 - SNE Vx, Vy**

_Skip next instruction if Vx != Vy._

Since our PC has already been incremented by 2 in **Cycle()**, we can just increment by 2 again to skip the next instruction.

|   |   |
|---|---|
|```<br> 1<br> 2<br> 3<br> 4<br> 5<br> 6<br> 7<br> 8<br> 9<br>10<br>```|```cpp<br>void Chip8::OP_9xy0()<br>{<br>	uint8_t Vx = (opcode & 0x0F00u) >> 8u;<br>	uint8_t Vy = (opcode & 0x00F0u) >> 4u;<br><br>	if (registers[Vx] != registers[Vy])<br>	{<br>		pc += 2;<br>	}<br>}<br>```|

**Annn - LD I, addr**

_Set I = nnn._

|   |   |
|---|---|
|```<br>1<br>2<br>3<br>4<br>5<br>6<br>```|```cpp<br>void Chip8::OP_Annn()<br>{<br>	uint16_t address = opcode & 0x0FFFu;<br><br>	index = address;<br>}<br>```|

**Bnnn - JP V0, addr**

_Jump to location nnn + V0._

|   |   |
|---|---|
|```<br>1<br>2<br>3<br>4<br>5<br>6<br>```|```cpp<br>void Chip8::OP_Bnnn()<br>{<br>	uint16_t address = opcode & 0x0FFFu;<br><br>	pc = registers[0] + address;<br>}<br>```|

**Cxkk - RND Vx, byte**

_Set Vx = random byte AND kk._

|   |   |
|---|---|
|```<br>1<br>2<br>3<br>4<br>5<br>6<br>7<br>```|```cpp<br>void Chip8::OP_Cxkk()<br>{<br>	uint8_t Vx = (opcode & 0x0F00u) >> 8u;<br>	uint8_t byte = opcode & 0x00FFu;<br><br>	registers[Vx] = randByte(randGen) & byte;<br>}<br>```|

**Dxyn - DRW Vx, Vy, nibble**

_Display n-byte sprite starting at memory location I at (Vx, Vy), set VF = collision._

We iterate over the sprite, row by row and column by column. We know there are eight columns because a sprite is guaranteed to be eight pixels wide.

If a sprite pixel is on then there may be a collision with what’s already being displayed, so we check if our screen pixel in the same location is set. If so we must set the VF register to express collision.

Then we can just XOR the screen pixel with 0xFFFFFFFF to essentially XOR it with the sprite pixel (which we now know is on). We can’t XOR directly because the sprite pixel is either 1 or 0 while our video pixel is either 0x00000000 or 0xFFFFFFFF.

|   |   |
|---|---|
|```<br> 1<br> 2<br> 3<br> 4<br> 5<br> 6<br> 7<br> 8<br> 9<br>10<br>11<br>12<br>13<br>14<br>15<br>16<br>17<br>18<br>19<br>20<br>21<br>22<br>23<br>24<br>25<br>26<br>27<br>28<br>29<br>30<br>31<br>32<br>33<br>34<br>35<br>36<br>```|```cpp<br>void Chip8::OP_Dxyn()<br>{<br>	uint8_t Vx = (opcode & 0x0F00u) >> 8u;<br>	uint8_t Vy = (opcode & 0x00F0u) >> 4u;<br>	uint8_t height = opcode & 0x000Fu;<br><br>	// Wrap if going beyond screen boundaries<br>	uint8_t xPos = registers[Vx] % VIDEO_WIDTH;<br>	uint8_t yPos = registers[Vy] % VIDEO_HEIGHT;<br><br>	registers[0xF] = 0;<br><br>	for (unsigned int row = 0; row < height; ++row)<br>	{<br>		uint8_t spriteByte = memory[index + row];<br><br>		for (unsigned int col = 0; col < 8; ++col)<br>		{<br>			uint8_t spritePixel = spriteByte & (0x80u >> col);<br>			uint32_t* screenPixel = &video[(yPos + row) * VIDEO_WIDTH + (xPos + col)];<br><br>			// Sprite pixel is on<br>			if (spritePixel)<br>			{<br>				// Screen pixel also on - collision<br>				if (*screenPixel == 0xFFFFFFFF)<br>				{<br>					registers[0xF] = 1;<br>				}<br><br>				// Effectively XOR with the sprite pixel<br>				*screenPixel ^= 0xFFFFFFFF;<br>			}<br>		}<br>	}<br>}<br>```|

**Ex9E - SKP Vx**

_Skip next instruction if key with the value of Vx is pressed._

Since our PC has already been incremented by 2 in **Cycle()**, we can just increment by 2 again to skip the next instruction.

|   |   |
|---|---|
|```<br> 1<br> 2<br> 3<br> 4<br> 5<br> 6<br> 7<br> 8<br> 9<br>10<br>11<br>```|```cpp<br>void Chip8::OP_Ex9E()<br>{<br>	uint8_t Vx = (opcode & 0x0F00u) >> 8u;<br><br>	uint8_t key = registers[Vx];<br><br>	if (keypad[key])<br>	{<br>		pc += 2;<br>	}<br>}<br>```|

**ExA1 - SKNP Vx**

_Skip next instruction if key with the value of Vx is not pressed._

Since our PC has already been incremented by 2 in **Cycle()**, we can just increment by 2 again to skip the next instruction.

|   |   |
|---|---|
|```<br> 1<br> 2<br> 3<br> 4<br> 5<br> 6<br> 7<br> 8<br> 9<br>10<br>11<br>```|```cpp<br>void Chip8::OP_ExA1()<br>{<br>	uint8_t Vx = (opcode & 0x0F00u) >> 8u;<br><br>	uint8_t key = registers[Vx];<br><br>	if (!keypad[key])<br>	{<br>		pc += 2;<br>	}<br>}<br>```|

**Fx07 - LD Vx, DT**

_Set Vx = delay timer value._

|   |   |
|---|---|
|```<br>1<br>2<br>3<br>4<br>5<br>6<br>```|```cpp<br>void Chip8::OP_Fx07()<br>{<br>	uint8_t Vx = (opcode & 0x0F00u) >> 8u;<br><br>	registers[Vx] = delayTimer;<br>}<br>```|

**Fx0A - LD Vx, K**

_Wait for a key press, store the value of the key in Vx._

The easiest way to “wait” is to decrement the PC by 2 whenever a keypad value is not detected. This has the effect of running the same instruction repeatedly.

|   |   |
|---|---|
|```<br> 1<br> 2<br> 3<br> 4<br> 5<br> 6<br> 7<br> 8<br> 9<br>10<br>11<br>12<br>13<br>14<br>15<br>16<br>17<br>18<br>19<br>20<br>21<br>22<br>23<br>24<br>25<br>26<br>27<br>28<br>29<br>30<br>31<br>32<br>33<br>34<br>35<br>36<br>37<br>38<br>39<br>40<br>41<br>42<br>43<br>44<br>45<br>46<br>47<br>48<br>49<br>50<br>51<br>52<br>53<br>54<br>55<br>56<br>57<br>58<br>59<br>60<br>61<br>62<br>63<br>64<br>65<br>66<br>67<br>68<br>69<br>70<br>71<br>72<br>73<br>```|```cpp<br>void Chip8::OP_Fx0A()<br>{<br>	uint8_t Vx = (opcode & 0x0F00u) >> 8u;<br><br>	if (keypad[0])<br>	{<br>		registers[Vx] = 0;<br>	}<br>	else if (keypad[1])<br>	{<br>		registers[Vx] = 1;<br>	}<br>	else if (keypad[2])<br>	{<br>		registers[Vx] = 2;<br>	}<br>	else if (keypad[3])<br>	{<br>		registers[Vx] = 3;<br>	}<br>	else if (keypad[4])<br>	{<br>		registers[Vx] = 4;<br>	}<br>	else if (keypad[5])<br>	{<br>		registers[Vx] = 5;<br>	}<br>	else if (keypad[6])<br>	{<br>		registers[Vx] = 6;<br>	}<br>	else if (keypad[7])<br>	{<br>		registers[Vx] = 7;<br>	}<br>	else if (keypad[8])<br>	{<br>		registers[Vx] = 8;<br>	}<br>	else if (keypad[9])<br>	{<br>		registers[Vx] = 9;<br>	}<br>	else if (keypad[10])<br>	{<br>		registers[Vx] = 10;<br>	}<br>	else if (keypad[11])<br>	{<br>		registers[Vx] = 11;<br>	}<br>	else if (keypad[12])<br>	{<br>		registers[Vx] = 12;<br>	}<br>	else if (keypad[13])<br>	{<br>		registers[Vx] = 13;<br>	}<br>	else if (keypad[14])<br>	{<br>		registers[Vx] = 14;<br>	}<br>	else if (keypad[15])<br>	{<br>		registers[Vx] = 15;<br>	}<br>	else<br>	{<br>		pc -= 2;<br>	}<br>}<br>```|

**Fx15 - LD DT, Vx**

_Set delay timer = Vx._

|   |   |
|---|---|
|```<br>1<br>2<br>3<br>4<br>5<br>6<br>```|```cpp<br>void Chip8::OP_Fx15()<br>{<br>	uint8_t Vx = (opcode & 0x0F00u) >> 8u;<br><br>	delayTimer = registers[Vx];<br>}<br>```|

**Fx18 - LD ST, Vx**

_Set sound timer = Vx._

|   |   |
|---|---|
|```<br>1<br>2<br>3<br>4<br>5<br>6<br>```|```cpp<br>void Chip8::OP_Fx18()<br>{<br>	uint8_t Vx = (opcode & 0x0F00u) >> 8u;<br><br>	soundTimer = registers[Vx];<br>}<br>```|

**Fx1E - ADD I, Vx**

_Set I = I + Vx._

|   |   |
|---|---|
|```<br>1<br>2<br>3<br>4<br>5<br>6<br>```|```cpp<br>void Chip8::OP_Fx1E()<br>{<br>	uint8_t Vx = (opcode & 0x0F00u) >> 8u;<br><br>	index += registers[Vx];<br>}<br>```|

**Fx29 - LD F, Vx**

_Set I = location of sprite for digit Vx._

We know the font characters are located at 0x50, and we know they’re five bytes each, so we can get the address of the first byte of any character by taking an offset from the start address.

|   |   |
|---|---|
|```<br>1<br>2<br>3<br>4<br>5<br>6<br>7<br>```|```cpp<br>void Chip8::OP_Fx29()<br>{<br>	uint8_t Vx = (opcode & 0x0F00u) >> 8u;<br>	uint8_t digit = registers[Vx];<br><br>	index = FONTSET_START_ADDRESS + (5 * digit);<br>}<br>```|

**Fx33 - LD B, Vx**

_Store BCD representation of Vx in memory locations I, I+1, and I+2._

_The interpreter takes the decimal value of Vx, and places the hundreds digit in memory at location in I, the tens digit at location I+1, and the ones digit at location I+2._

We can use the modulus operator to get the right-most digit of a number, and then do a division to remove that digit. A division by ten will either completely remove the digit (340 / 10 = 34), or result in a float which will be truncated (345 / 10 = 34.5 = 34).

|   |   |
|---|---|
|```<br> 1<br> 2<br> 3<br> 4<br> 5<br> 6<br> 7<br> 8<br> 9<br>10<br>11<br>12<br>13<br>14<br>15<br>16<br>```|```cpp<br>void Chip8::OP_Fx33()<br>{<br>	uint8_t Vx = (opcode & 0x0F00u) >> 8u;<br>	uint8_t value = registers[Vx];<br><br>	// Ones-place<br>	memory[index + 2] = value % 10;<br>	value /= 10;<br><br>	// Tens-place<br>	memory[index + 1] = value % 10;<br>	value /= 10;<br><br>	// Hundreds-place<br>	memory[index] = value % 10;<br>}<br>```|

**Fx55 - LD [I], Vx**

_Store registers V0 through Vx in memory starting at location I._

|   |   |
|---|---|
|```<br>1<br>2<br>3<br>4<br>5<br>6<br>7<br>8<br>9<br>```|```cpp<br>void Chip8::OP_Fx55()<br>{<br>	uint8_t Vx = (opcode & 0x0F00u) >> 8u;<br><br>	for (uint8_t i = 0; i <= Vx; ++i)<br>	{<br>		memory[index + i] = registers[i];<br>	}<br>}<br>```|

**Fx65 - LD Vx, [I]**

_Read registers V0 through Vx from memory starting at location I._

|   |   |
|---|---|
|```<br>1<br>2<br>3<br>4<br>5<br>6<br>7<br>8<br>9<br>```|```cpp<br>void Chip8::OP_Fx65()<br>{<br>	uint8_t Vx = (opcode & 0x0F00u) >> 8u;<br><br>	for (uint8_t i = 0; i <= Vx; ++i)<br>	{<br>		registers[i] = memory[index + i];<br>	}<br>}<br>```|

## Function Pointer Table

---

The easiest way to decode an opcode is with a switch statement, but it gets messy when you have a lot of instructions. The CHIP-8 isn’t so bad, but we’ll use a different technique that is more scalable and good to know when making more advanced emulators.

Instead we’ll implement an array of function pointers where the opcode is an index into an array of function pointers. The downside to a function pointer table is that we must have an array big enough to account for every opcode because the opcode is an index into the array. Dereferencing a pointer for every instruction may also have problems, but when your emulator is complex it’s probably worth it.

If you look at the list of opcodes, you’ll notice that there are four types:

- The entire opcode is unique:
    - $1nnn
    - $2nnn
    - $3xkk
    - $4xkk
    - $5xy0
    - $6xkk
    - $7xkk
    - $9xy0
    - $Annn
    - $Bnnn
    - $Cxkk
    - $Dxyn
- The first digit repeats but the last digit is unique:
    - $8xy0
    - $8xy1
    - $8xy2
    - $8xy3
    - $8xy4
    - $8xy5
    - $8xy6
    - $8xy7
    - $8xyE
- The first three digits are $00E but the fourth digit is unique:
    - $00E0
    - $00EE
- The first digit repeats but the last two digits are unique:
    - $ExA1
    - $Ex9E
    - $Fx07
    - $Fx0A
    - $Fx15
    - $Fx18
    - $Fx1E
    - $Fx29
    - $Fx33
    - $Fx55
    - $Fx65

The first digits go from $0 to $F so we’ll need an array of function pointers that can be indexed up to $F, which requires $F + 1 elements.

For the opcodes with first digits that repeat ($0, $8, $E, $F), we’ll need secondary tables that can accommodate each of those.

- $0 needs an array that can index up to $E+1
- $8 needs an array that can index up to $E+1
- $E needs an array that can index up to $E+1
- $F needs an array that can index up to $65+1

In the master **table**, we set up a function pointer to a function that then indexes correctly based on the relevant parts of the opcode.

Just in case one of the invalid opcodes is called, we can create a dummy **OP_NULL** function that does nothing, but will be the default function called if a proper function pointer is not set.

C++ member function pointer syntax is horrendous.

|   |   |
|---|---|
|```<br> 1<br> 2<br> 3<br> 4<br> 5<br> 6<br> 7<br> 8<br> 9<br>10<br>11<br>12<br>13<br>14<br>15<br>16<br>17<br>18<br>19<br>20<br>21<br>22<br>23<br>24<br>25<br>26<br>27<br>28<br>29<br>30<br>31<br>32<br>33<br>34<br>35<br>36<br>37<br>38<br>39<br>40<br>41<br>42<br>43<br>44<br>45<br>46<br>47<br>48<br>49<br>50<br>51<br>52<br>53<br>54<br>55<br>56<br>57<br>58<br>59<br>60<br>61<br>62<br>63<br>64<br>65<br>66<br>67<br>68<br>69<br>70<br>71<br>72<br>73<br>74<br>75<br>76<br>77<br>78<br>79<br>80<br>81<br>82<br>83<br>84<br>85<br>86<br>87<br>88<br>89<br>90<br>91<br>92<br>93<br>94<br>95<br>96<br>97<br>```|```cpp<br>class Chip8<br>{<br>	Chip8()<br>		: randGen(std::chrono::system_clock::now().time_since_epoch().count())<br>	{<br>		[...]<br><br>		// Set up function pointer table<br>		table[0x0] = &Chip8::Table0;<br>		table[0x1] = &Chip8::OP_1nnn;<br>		table[0x2] = &Chip8::OP_2nnn;<br>		table[0x3] = &Chip8::OP_3xkk;<br>		table[0x4] = &Chip8::OP_4xkk;<br>		table[0x5] = &Chip8::OP_5xy0;<br>		table[0x6] = &Chip8::OP_6xkk;<br>		table[0x7] = &Chip8::OP_7xkk;<br>		table[0x8] = &Chip8::Table8;<br>		table[0x9] = &Chip8::OP_9xy0;<br>		table[0xA] = &Chip8::OP_Annn;<br>		table[0xB] = &Chip8::OP_Bnnn;<br>		table[0xC] = &Chip8::OP_Cxkk;<br>		table[0xD] = &Chip8::OP_Dxyn;<br>		table[0xE] = &Chip8::TableE;<br>		table[0xF] = &Chip8::TableF;<br><br>		for (size_t i = 0; i <= 0xE; i++)<br>		{<br>			table0[i] = &Chip8::OP_NULL;<br>			table8[i] = &Chip8::OP_NULL;<br>			tableE[i] = &Chip8::OP_NULL;<br>		}<br><br>		table0[0x0] = &Chip8::OP_00E0;<br>		table0[0xE] = &Chip8::OP_00EE;<br><br>		table8[0x0] = &Chip8::OP_8xy0;<br>		table8[0x1] = &Chip8::OP_8xy1;<br>		table8[0x2] = &Chip8::OP_8xy2;<br>		table8[0x3] = &Chip8::OP_8xy3;<br>		table8[0x4] = &Chip8::OP_8xy4;<br>		table8[0x5] = &Chip8::OP_8xy5;<br>		table8[0x6] = &Chip8::OP_8xy6;<br>		table8[0x7] = &Chip8::OP_8xy7;<br>		table8[0xE] = &Chip8::OP_8xyE;<br><br>		tableE[0x1] = &Chip8::OP_ExA1;<br>		tableE[0xE] = &Chip8::OP_Ex9E;<br><br>		for (size_t i = 0; i <= 0x65; i++)<br>		{<br>			tableF[i] = &Chip8::OP_NULL;<br>		}<br><br>		tableF[0x07] = &Chip8::OP_Fx07;<br>		tableF[0x0A] = &Chip8::OP_Fx0A;<br>		tableF[0x15] = &Chip8::OP_Fx15;<br>		tableF[0x18] = &Chip8::OP_Fx18;<br>		tableF[0x1E] = &Chip8::OP_Fx1E;<br>		tableF[0x29] = &Chip8::OP_Fx29;<br>		tableF[0x33] = &Chip8::OP_Fx33;<br>		tableF[0x55] = &Chip8::OP_Fx55;<br>		tableF[0x65] = &Chip8::OP_Fx65;<br>	}<br><br>	void Table0()<br>	{<br>		((*this).*(table0[opcode & 0x000Fu]))();<br>	}<br><br>	void Table8()<br>	{<br>		((*this).*(table8[opcode & 0x000Fu]))();<br>	}<br><br>	void TableE()<br>	{<br>		((*this).*(tableE[opcode & 0x000Fu]))();<br>	}<br><br>	void TableF()<br>	{<br>		((*this).*(tableF[opcode & 0x00FFu]))();<br>	}<br><br>	void OP_NULL()<br>	{}<br><br><br>	[...]<br><br>	typedef void (Chip8::*Chip8Func)();<br>	Chip8Func table[0xF + 1];<br>	Chip8Func table0[0xE + 1];<br>	Chip8Func table8[0xE + 1];<br>	Chip8Func tableE[0xE + 1];<br>	Chip8Func tableF[0x65 + 1];<br>}<br>```|

## Fetch, Decode, Execute

---

When we talk about one cycle of this primitive CPU that we’re emulating, we’re talking about it doing three things:

- Fetch the next instruction in the form of an opcode
- Decode the instruction to determine what operation needs to occur
- Execute the instruction

The decoding and executing are done with the function pointers we just implemented. We get the first digit of the opcode with a bitmask, shift it over so that it becomes a single digit from $0 to $F, and use that as index into the function pointer array. It’s then further decoded in the **Table()** method.

|   |   |
|---|---|
|```<br> 1<br> 2<br> 3<br> 4<br> 5<br> 6<br> 7<br> 8<br> 9<br>10<br>11<br>12<br>13<br>14<br>15<br>16<br>17<br>18<br>19<br>20<br>21<br>22<br>23<br>```|```cpp<br>void Chip8::Cycle()<br>{<br>	// Fetch<br>	opcode = (memory[pc] << 8u) \| memory[pc + 1];<br><br>	// Increment the PC before we execute anything<br>	pc += 2;<br><br>	// Decode and Execute<br>	((*this).*(table[(opcode & 0xF000u) >> 12u]))();<br><br>	// Decrement the delay timer if it's been set<br>	if (delayTimer > 0)<br>	{<br>		--delayTimer;<br>	}<br><br>	// Decrement the sound timer if it's been set<br>	if (soundTimer > 0)<br>	{<br>		--soundTimer;<br>	}<br>}<br>```|

## The Platform Layer

---

We’ll use SDL to render and get input in a multi-platform way. Using an **SDL_Renderer** gives us 2D GPU acceleration, and an **SDL_Texture** is an easy way to render a 2D image. Consult the [SDL documentation](https://wiki.libsdl.org/APIByCategory) for more specific information.

|   |   |
|---|---|
|```<br>  1<br>  2<br>  3<br>  4<br>  5<br>  6<br>  7<br>  8<br>  9<br> 10<br> 11<br> 12<br> 13<br> 14<br> 15<br> 16<br> 17<br> 18<br> 19<br> 20<br> 21<br> 22<br> 23<br> 24<br> 25<br> 26<br> 27<br> 28<br> 29<br> 30<br> 31<br> 32<br> 33<br> 34<br> 35<br> 36<br> 37<br> 38<br> 39<br> 40<br> 41<br> 42<br> 43<br> 44<br> 45<br> 46<br> 47<br> 48<br> 49<br> 50<br> 51<br> 52<br> 53<br> 54<br> 55<br> 56<br> 57<br> 58<br> 59<br> 60<br> 61<br> 62<br> 63<br> 64<br> 65<br> 66<br> 67<br> 68<br> 69<br> 70<br> 71<br> 72<br> 73<br> 74<br> 75<br> 76<br> 77<br> 78<br> 79<br> 80<br> 81<br> 82<br> 83<br> 84<br> 85<br> 86<br> 87<br> 88<br> 89<br> 90<br> 91<br> 92<br> 93<br> 94<br> 95<br> 96<br> 97<br> 98<br> 99<br>100<br>101<br>102<br>103<br>104<br>105<br>106<br>107<br>108<br>109<br>110<br>111<br>112<br>113<br>114<br>115<br>116<br>117<br>118<br>119<br>120<br>121<br>122<br>123<br>124<br>125<br>126<br>127<br>128<br>129<br>130<br>131<br>132<br>133<br>134<br>135<br>136<br>137<br>138<br>139<br>140<br>141<br>142<br>143<br>144<br>145<br>146<br>147<br>148<br>149<br>150<br>151<br>152<br>153<br>154<br>155<br>156<br>157<br>158<br>159<br>160<br>161<br>162<br>163<br>164<br>165<br>166<br>167<br>168<br>169<br>170<br>171<br>172<br>173<br>174<br>175<br>176<br>177<br>178<br>179<br>180<br>181<br>182<br>183<br>184<br>185<br>186<br>187<br>188<br>189<br>190<br>191<br>192<br>193<br>194<br>195<br>196<br>197<br>198<br>199<br>200<br>201<br>202<br>203<br>204<br>205<br>206<br>207<br>208<br>209<br>210<br>211<br>212<br>213<br>214<br>215<br>216<br>217<br>218<br>219<br>220<br>221<br>222<br>223<br>224<br>225<br>226<br>227<br>228<br>229<br>230<br>231<br>232<br>233<br>```|```cpp<br>class Platform<br>{<br>public:<br>	Platform(char const* title, int windowWidth, int windowHeight, int textureWidth, int textureHeight)<br>	{<br>		SDL_Init(SDL_INIT_VIDEO);<br><br>		window = SDL_CreateWindow(title, 0, 0, windowWidth, windowHeight, SDL_WINDOW_SHOWN);<br><br>		renderer = SDL_CreateRenderer(window, -1, SDL_RENDERER_ACCELERATED);<br><br>		texture = SDL_CreateTexture(<br>			renderer, SDL_PIXELFORMAT_RGBA8888, SDL_TEXTUREACCESS_STREAMING, textureWidth, textureHeight);<br>	}<br><br>	~Platform()<br>	{<br>		SDL_DestroyTexture(texture);<br>		SDL_DestroyRenderer(renderer);<br>		SDL_DestroyWindow(window);<br>		SDL_Quit();<br>	}<br><br>	void Update(void const* buffer, int pitch)<br>	{<br>		SDL_UpdateTexture(texture, nullptr, buffer, pitch);<br>		SDL_RenderClear(renderer);<br>		SDL_RenderCopy(renderer, texture, nullptr, nullptr);<br>		SDL_RenderPresent(renderer);<br>	}<br><br>	bool ProcessInput(uint8_t* keys)<br>	{<br>		bool quit = false;<br><br>		SDL_Event event;<br><br>		while (SDL_PollEvent(&event))<br>		{<br>			switch (event.type)<br>			{<br>				case SDL_QUIT:<br>				{<br>					quit = true;<br>				} break;<br><br>				case SDL_KEYDOWN:<br>				{<br>					switch (event.key.keysym.sym)<br>					{<br>						case SDLK_ESCAPE:<br>						{<br>							quit = true;<br>						} break;<br><br>						case SDLK_x:<br>						{<br>							keys[0] = 1;<br>						} break;<br><br>						case SDLK_1:<br>						{<br>							keys[1] = 1;<br>						} break;<br><br>						case SDLK_2:<br>						{<br>							keys[2] = 1;<br>						} break;<br><br>						case SDLK_3:<br>						{<br>							keys[3] = 1;<br>						} break;<br><br>						case SDLK_q:<br>						{<br>							keys[4] = 1;<br>						} break;<br><br>						case SDLK_w:<br>						{<br>							keys[5] = 1;<br>						} break;<br><br>						case SDLK_e:<br>						{<br>							keys[6] = 1;<br>						} break;<br><br>						case SDLK_a:<br>						{<br>							keys[7] = 1;<br>						} break;<br><br>						case SDLK_s:<br>						{<br>							keys[8] = 1;<br>						} break;<br><br>						case SDLK_d:<br>						{<br>							keys[9] = 1;<br>						} break;<br><br>						case SDLK_z:<br>						{<br>							keys[0xA] = 1;<br>						} break;<br><br>						case SDLK_c:<br>						{<br>							keys[0xB] = 1;<br>						} break;<br><br>						case SDLK_4:<br>						{<br>							keys[0xC] = 1;<br>						} break;<br><br>						case SDLK_r:<br>						{<br>							keys[0xD] = 1;<br>						} break;<br><br>						case SDLK_f:<br>						{<br>							keys[0xE] = 1;<br>						} break;<br><br>						case SDLK_v:<br>						{<br>							keys[0xF] = 1;<br>						} break;<br>					}<br>				} break;<br><br>				case SDL_KEYUP:<br>				{<br>					switch (event.key.keysym.sym)<br>					{<br>						case SDLK_x:<br>						{<br>							keys[0] = 0;<br>						} break;<br><br>						case SDLK_1:<br>						{<br>							keys[1] = 0;<br>						} break;<br><br>						case SDLK_2:<br>						{<br>							keys[2] = 0;<br>						} break;<br><br>						case SDLK_3:<br>						{<br>							keys[3] = 0;<br>						} break;<br><br>						case SDLK_q:<br>						{<br>							keys[4] = 0;<br>						} break;<br><br>						case SDLK_w:<br>						{<br>							keys[5] = 0;<br>						} break;<br><br>						case SDLK_e:<br>						{<br>							keys[6] = 0;<br>						} break;<br><br>						case SDLK_a:<br>						{<br>							keys[7] = 0;<br>						} break;<br><br>						case SDLK_s:<br>						{<br>							keys[8] = 0;<br>						} break;<br><br>						case SDLK_d:<br>						{<br>							keys[9] = 0;<br>						} break;<br><br>						case SDLK_z:<br>						{<br>							keys[0xA] = 0;<br>						} break;<br><br>						case SDLK_c:<br>						{<br>							keys[0xB] = 0;<br>						} break;<br><br>						case SDLK_4:<br>						{<br>							keys[0xC] = 0;<br>						} break;<br><br>						case SDLK_r:<br>						{<br>							keys[0xD] = 0;<br>						} break;<br><br>						case SDLK_f:<br>						{<br>							keys[0xE] = 0;<br>						} break;<br><br>						case SDLK_v:<br>						{<br>							keys[0xF] = 0;<br>						} break;<br>					}<br>				} break;<br>			}<br>		}<br><br>		return quit;<br>	}<br><br>private:<br>	SDL_Window* window{};<br>	SDL_Renderer* renderer{};<br>	SDL_Texture* texture{};<br>};<br>```|

## The Main Loop

---

Finally we need our main loop that will call our **Chip8::Cycle()** function continuously until exit, handle input, and render with SDL.

We’ll use three command-line arguments:

- **Scale:** The CHIP-8 video buffer is only 64x32, so we’ll need an integer scale factor to be able to play on our big modern monitors.
- **Delay:** The CHIP-8 had no specified clock speed, so we’ll use a delay to determine the time in milliseconds between cycles. Different games run best at different speeds, so we can control it here.
- **ROM:** The ROM file to load.

With each iteration of the loop: input from the keyboard is parsed, a delay is checked to see if enough time has passed between cycles and a cycle is run if so, and the screen is updated.

Due to the way SDL works, we can simply pass in the **video** parameter to SDL and it will scale it automatically for us to the size of our window texture.

|   |   |
|---|---|
|```<br> 1<br> 2<br> 3<br> 4<br> 5<br> 6<br> 7<br> 8<br> 9<br>10<br>11<br>12<br>13<br>14<br>15<br>16<br>17<br>18<br>19<br>20<br>21<br>22<br>23<br>24<br>25<br>26<br>27<br>28<br>29<br>30<br>31<br>32<br>33<br>34<br>35<br>36<br>37<br>38<br>39<br>40<br>41<br>42<br>43<br>44<br>```|```cpp<br>#include <chrono><br>#include <iostream><br><br>int main(int argc, char__ argv)<br>{<br>	if (argc != 4)<br>	{<br>		std::cerr << "Usage: " << argv[0] << " <Scale> <Delay> <ROM>\n";<br>		std::exit(EXIT_FAILURE);<br>	}<br><br>	int videoScale = std::stoi(argv[1]);<br>	int cycleDelay = std::stoi(argv[2]);<br>	char const* romFilename = argv[3];<br><br>	Platform platform("CHIP-8 Emulator", VIDEO_WIDTH * videoScale, VIDEO_HEIGHT * videoScale, VIDEO_WIDTH, VIDEO_HEIGHT);<br><br>	Chip8 chip8;<br>	chip8.LoadROM(romFilename);<br><br>	int videoPitch = sizeof(chip8.video[0]) * VIDEO_WIDTH;<br><br>	auto lastCycleTime = std::chrono::high_resolution_clock::now();<br>	bool quit = false;<br><br>	while (!quit)<br>	{<br>		quit = platform.ProcessInput(chip8.keypad);<br><br>		auto currentTime = std::chrono::high_resolution_clock::now();<br>		float dt = std::chrono::duration<float, std::chrono::milliseconds::period>(currentTime - lastCycleTime).count();<br><br>		if (dt > cycleDelay)<br>		{<br>			lastCycleTime = currentTime;<br><br>			chip8.Cycle();<br><br>			platform.Update(chip8.video, videoPitch);<br>		}<br>	}<br><br>	return 0;<br>}<br>```|

## Results

---

We can use [this test ROM](https://github.com/corax89/chip8-test-rom) to ensure that our CPU is operating like we expect.

|   |   |
|---|---|
|```<br>1<br>```|```text<br>./chip8 10 1 test_opcode.ch8<br>```|

[![](https://austinmorlan.com/posts/chip8_emulator/media/test_opcode.png)](https://austinmorlan.com/posts/chip8_emulator/media/test_opcode.png)

Let’s also test our input by playing a game: [Tetris](https://github.com/dmatlack/chip8/tree/master/roms/games). I’ve found a delay of 3 or 4 is pretty good.

|   |   |
|---|---|
|```<br>1<br>```|```text<br>./chip8 10 3 Tetris.ch8<br>```|

## Conclusion

---

Congratulations! We’ve successfully built a very simple emulator. Hopefully you’ve learned something about emulators, bit-twiddling, logic, CPU operations, and more. Take that knowledge and put it to use in other programming endeavors, or move onto building an NES emulator. That’s what I’ll be doing.

## Appendix: Bits, Bytes, Etc.

---

Low-level bit manipulation is a very important part of this project, so if you need a primer or refresher, here it is.

### Binary

Computers operate on numbers in the form of **binary**, which is **base-2**. Any number can be presented in binary. There is a pattern to binary that helps with understanding it. Here is the sequence from 0 to 15:

|   |   |
|---|---|
|```<br> 1<br> 2<br> 3<br> 4<br> 5<br> 6<br> 7<br> 8<br> 9<br>10<br>11<br>12<br>13<br>14<br>15<br>16<br>17<br>18<br>```|```text<br>\|3\|2\|1\|0\|<br>\|-\|-\|-\|-\|<br>\|0\|0\|0\|0\|<br>\|0\|0\|0\|1\|<br>\|0\|0\|1\|0\|<br>\|0\|0\|1\|1\|<br>\|0\|1\|0\|0\|<br>\|0\|1\|0\|1\|<br>\|0\|1\|1\|0\|<br>\|0\|1\|1\|1\|<br>\|1\|0\|0\|0\|<br>\|1\|0\|0\|1\|<br>\|1\|0\|1\|0\|<br>\|1\|0\|1\|1\|<br>\|1\|1\|0\|0\|<br>\|1\|1\|0\|1\|<br>\|1\|1\|1\|0\|<br>\|1\|1\|1\|1\|<br>```|

If we label each bit starting from **Bit 3** on the left to **Bit 0** on the right, we can see a pattern. Bit 0 alternates off and on with every number. Bit 1 alternates at 1/2 the rate of Bit 1. Bit 2 alternates at 1/4 the rate of Bit 1. And Bit 3 alternates at 1/8 the rate of Bit 1.

With our **base-10** system, numbers count up from 0 to 9 and then shift over to the left when we grow larger than what 9 can represent. 9 becomes 10. 19 becomes 20. 29 becomes 30. And so on.

Binary is no different, only it’s base-2, so we only have 0 and 1. We go from 0b0001 to 0b0010. 0b0011 to 0b0100. 0b0111 to 0b1000.

In base-10, we can get any digit with a sum. Remember talking about the “ones place”, the “tens place”, the “hundreds place”, etc? The base-10 number 216 has a 2 in the hundreds place, a 1 in the tens place, and a 6 in the hundreds place. 206 can then be created by multiplying each digit by its place and summing them all: (2⋅102)+(1⋅101)+(6⋅100)=216(2⋅10​2​​)+(1⋅10​1​​)+(6⋅10​0​​)=216

Again, binary is no different, only you have a ones place, a twos place, a fours place, etc. The binary number 0b101 has a 1 in the fours place, a 0 in the twos place, and a 1 in the ones place: (1⋅22)+(0⋅21)+(1⋅20)=5(1⋅2​2​​)+(0⋅2​1​​)+(1⋅2​0​​)=5

Bit 3 is called the **Most Significant Bit (MSB)** because it contributes the most to the final value of the number (because its the eights place). Bit 0 is called the **Least Significant Bit (LSB)** because it contributes the least to the final value (because it’s the ones place).

### Hexadecimal

Reading binary becomes difficult for a human after about four bits though, so we often use **hexadecimal (hex)** which is **base-16**. While base-2 has 0 and 1, and base-10 has 0 through 9, base-16 has 0 through 9 and A through F. The principles of counting it are the same as base-2 and base-10, there are just now letters involved.

- A = 10
- B = 11
- C = 12
- D = 13
- E = 14
- F = 15.

The hex number 0xBEEF is (B⋅163)+(E⋅162)+(E⋅161)+(F⋅160)=48879(B⋅16​3​​)+(E⋅16​2​​)+(E⋅16​1​​)+(F⋅16​0​​)=48879

In binary that would be 0b1011111011101111, which is much less pleasant to the human experience.

### Bits and Bytes

A single binary digit is called a **bit**. Because it’s binary, it can either be **ON (1)** or **OFF (0)**.

Eight bits make a **byte**, so a byte can go as high as 28=2552​8​​=255. In hex that is 0xFF.

The common multiples you will see are 1 byte (0xFF), 2 bytes (0xFFFF), 4 bytes (0xFFFFFFFF), and 8 bytes (0xFFFFFFFFFFFFFFFF).

### AND, OR, NOT, XOR

Beyond understanding how to read binary and hex, you need to know how to manipulate them with some common operations: **AND**, **OR**, and **XOR**.

They are explained most concisely in table form:

|   |   |
|---|---|
|```<br>1<br>2<br>3<br>4<br>5<br>6<br>```|```text<br>\| AND \|    \|  OR \|    \| NOT \|    \| XOR \|<br>-------    -------    -------    -------<br>\|0\|0\|0\|    \|0\|0\|0\|    \|0\|1\|      \|0\|0\|0\|<br>\|0\|1\|0\|    \|0\|1\|1\|    \|1\|0\|      \|0\|1\|1\|<br>\|1\|0\|0\|    \|1\|0\|1\|               \|1\|0\|1\|<br>\|1\|1\|1\|    \|1\|1\|1\|               \|1\|1\|0\|<br>```|

AND is true if both bits are true. OR is true if either bit is true. NOT toggles the bit. XOR is true if only a single bit is true.

In C++, AND is using the symbol **&**, OR uses the symbol **|**, NOT uses the symbol **~**, and XOR uses the symbol **^**.

### Bitmasking

AND and OR are especially useful for **bitmasking**, where you can manipulate a binary number with another binary number.

AND is good for **clearing bits**.

Let’s say we have the binary number 0b0110 and we want to clear Bit 1. We can AND it with 0b1101. The 0 will clear the bit if it’s set (1 AND 0 = 0), but any set bits will stay set (1 AND 1 = 1).

|   |   |
|---|---|
|```<br>1<br>2<br>3<br>4<br>```|```text<br>   0b0110<br>AND<br>   0b1101---------<br>   0b0100<br>```|

OR is good for **setting bits**.

Let’s say we have the binary number 0b0110 and we want to set Bit 3. We can OR it with 0b1000. The 1 will set the bit if it’s not set (1 OR 0 = 1), but any unset bits will stay unset (0 OR 1 = 0).

|   |   |
|---|---|
|```<br>1<br>2<br>3<br>4<br>5<br>```|```text<br>   0b0110<br>OR<br>   0b1000<br>---------<br>   0b1110<br>```|

NOT is good for **toggling bits**.

Let’s say we have the binary number 0b0110 and we want to toggle the off bits on and the on bits off. We can use the NOT operation.

|   |   |
|---|---|
|```<br>1<br>2<br>3<br>4<br>```|```text<br>NOT<br>   0b0110<br>---------<br>   0b1001<br>```|

XOR is good for many things, but one common application in CPUs is to clear a register by XORing it with itself.

Let’s say we have the binary number 0b0110 and we want to clear it all to zeros without using any other numbers. We can use XOR.

|   |   |
|---|---|
|```<br>1<br>2<br>3<br>4<br>5<br>```|```text<br>   0b0110<br>XOR<br>   0b0110<br>---------<br>   0b0000<br>```|

### Bit Shifting

**Bit shifting** is the act of moving the bits of a number left or right. A left shift is equivalent to multiplying by 2 and a right shift is equivalent to dividing by 2.

|   |   |
|---|---|
|```<br>1<br>2<br>3<br>```|```text<br>0b100 (4) -> 0b010 (2) -> 0b001 (1)<br><br>0b001 (1) <- 0b010 (2) <- 0b100 (4)<br>```|

## Source Code

---

You can find all of the code [here](https://code.austinmorlan.com/austin/2019-chip8-emulator).

  
  
Last Edited: Jun 19, 2024