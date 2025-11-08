---
layout: post
title: Flare-On 11 Write-Up Part 3
date: 2024-12-07 15:07 +0100
categories: [Hacking, Software]
tags: [hacking, software, capture the flag, write-up, reverse engineering]
media_subpath: /assets/img/flareon11_3/
---

This is the third post in my **Flare-On 11** series. If you haven’t already, I recommend checking out the [previous writeup](/posts/flare-on-11-write-up-part-2/) to get up to speed. 

As with the first post, you can find supplementary resources, including all the scripts I used during the CTF, in [this repository](https://github.com/vik0t0r/flare-on-11).

## Challenge 5 - sshd

> Our server in the FLARE Intergalactic HQ has crashed! Now criminals are trying to sell me my own data!!! Do your part, random internet hacker, to help FLARE out and tell us what data they stole! We used the best forensic preservation technique of just copying all the files on the system for you.

From my perspective, this was the hardest challenge to solve—and my favorite so far. It references the infamous **xz utils backdoor** from February. In this challenge, we are given the files from a Linux server to analyze.

#### Step 1: Exploring the filesystem

We begin by extracting the archive and listing its contents:

```bash
ls
bin  boot  dev  etc  fmnt  home  lib  lib64  media  mnt  opt  proc  root  run  sbin  srv  sys  tmp  usr  var
```

After exploring the filesystem, we find an interesting file in the root directory:

```bash
ls -lh root/
total 4.0K
-rw-r--r-- 1 root root    0 Aug  3 21:21 anonymized.phony
-rw-r--r-- 1 root root 2.3K Sep 11 22:55 flag.txt
cat root/flag.txt
⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⣧⠀⠀⠀⠀⠀⣿⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀
⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⢀⣿⣧⠀⠀⠀⢰⡿⡇⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀
⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⢸⣿⡟⡆⠀⠀⣿⡇⢻⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀
⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⢸⣿⠀⣿⠀⢰⣿⡇⢸⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀
⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⢸⣿⡄⢸⠀⢸⣿⡇⢸⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀
⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠘⣿⡇⢸⡄⠸⣿⡇⣿⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀
⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⢿⣿⢸⡅⠀⣿⢠⡏⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀
⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠈⣿⣿⣥⣾⣿⣿⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀
⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⣿⣿⣿⣿⣿⣿⣿⣆⠀⠀⠀⠀⠀⠀⠀⠀⠀
⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⢸⣿⣿⣿⡿⡿⣿⣿⡿⡅⠀⠀⠀⠀⠀⠀⠀⠀
⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⢸⠉⠀⠉⡙⢔⠛⣟⢋⠦⢵⠀⠀⠀⠀⠀⠀⠀
⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⣾⣄⠀⠀⠁⣿⣯⡥⠃⠀⢳⠀⠀⠀⠀⠀⠀⠀
⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⢀⣴⣿⡇⠀⠀⠀⠐⠠⠊⢀⠀⢸⠀⠀⠀⠀⠀⠀⠀
⠀⠀⠀⠀⠀⠀⠀⢀⣴⣿⣿⣿⡿⠀⠀⠀⠀⠀⠈⠁⠀⠀⠘⣿⣄⠀⠀⠀⠀⠀
⠀⠀⠀⠀⠀⣠⣿⣿⣿⣿⣿⡟⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠈⣿⣷⡀⠀⠀⠀
⠀⠀⠀⠀⣾⣿⣿⣿⣿⣿⠋⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠈⣿⣿⣧⠀⠀
⠀⠀⠀⡜⣭⠤⢍⣿⡟⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⢸⢛⢭⣗⠀
⠀⠀⠀⠁⠈⠀⠀⣀⠝⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠄⠠⠀⠀⠰⡅
⠀⠀⠀⢀⠀⠀⡀⠡⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠁⠔⠠⡕⠀
⠀⠀⠀⠀⣿⣷⣶⠒⠁⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⢰⠀⠀⠀⠀
⠀⠀⠀⠀⠘⣿⣿⡇⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠰⠀⠀⠀⠀⠀
⠀⠀⠀⠀⠀⠈⢿⣿⣦⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⢠⠊⠉⢆⠀⠀⠀⠀
⠀⢀⠤⠀⠀⢤⣤⣽⣿⣿⣦⣀⢀⡠⢤⡤⠄⠀⠒⠀⠁⠀⠀⠀⢘⠔⠀⠀⠀⠀
⠀⠀⠀⡐⠈⠁⠈⠛⣛⠿⠟⠑⠈⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀
⠀⠀⠉⠑⠒⠀⠁⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀
if only it were that easy......
```

Clearly, this is a decoy. Digging further, we discover a core dump from the sshd process. The challenge name starts to make sense:
```bash
tree
...
    │   ├── private
    │   ├── shells.state
    │   ├── systemd
    │   │   ├── catalog
    │   │   │   └── database
    │   │   ├── coredump
    │   │   │   └── sshd.core.93794.0.0.11.1725917676
    │   │   ├── deb-systemd-helper-enabled
    │   │   │   ├── apt-daily.timer.dsh-also
    │   │   │   ├── apt-daily-upgrade.timer.dsh-also
...
```
#### Step 2: Opening the core dump

We open the core dump with gdb for analysis. For better functionality, I used gef (GDB Enhanced Features).

```bash
gdb usr/sbin/sshd var/lib/systemd/coredump/sshd.core.93794.0.0.11.1725917676
```

![img-description](backtrace_empty.png){: width="1000" }


Apparently sshd crashed on a call to liblzma. At this point I understood the reference to the **xz utils backdoor** and I started to do some research. [This video](https://www.youtube.com/watch?v=Q6ovtLdSbEA) and [this github repository](https://github.com/amlweems/xzbot) helped me a lot by explaining the inner workings of the original backdoor.

#### Step 3: Donwloading debug symbols:

To improve the backtrace (replacing `??` with function names), I installed the same Debian version in a Docker container and downloaded the appropriate debug symbols.

**Setting Up the Environment**

We create the following `docker-compose.yaml`, We can determine the system's Debian version using the file `/etc/debian_version`:
```yaml
version: '3.8'

services:
  debian-container:
    image: jrei/systemd-debian:12
    container_name: debian_shell
    stdin_open: true  # Keep stdin open to allow interactive shell
    tty: true         # Allocate a pseudo-TTY for the container
    volumes:
      - ./ssh_container:/mnt
    command: /bin/bash

volumes:
  ssh_container:
```

Then we run and attach to the docker getting a shell:

```bash
docker-compose up
docker attach debian_shell
```

Then we update the docker package repositories to enable debug sysmbols and source code repositories by writting to `/etc/apt/sources.list`:

```bash
deb http://deb.debian.org/debian-debug/ bookworm-debug main
deb-src http://deb.debian.org/debian bookworm main
```

And then I installed  the necessary dependencies:

```bash
apt update

apt install nano
apt install dpkg-dev
apt install gdb
# GEF dependencies
apt install curl
apt install python3
apt install file

# install GEF:
bash -c "$(wget https://gef.blah.cat/sh -O -)"

# fix for `UnicodeEncodeError` on GEF:
export LC_ALL=C.UTF-8
```

Now I was ready to download the debug symbols and the source code for sshd:
```bash
# download dbgsymbols
apt install openssh-server-dbgsy

# download source code on openssh-9.2p1/
apt source openssh-server
```

Next, we run gdb, load the core dump using the binary from the Docker instance, and include the source files in gdb for debugging.

```bash
gdb /usr/sbin/sshd /mnt/var/lib/systemd/coredump/sshd.core.93794.0.0.11.1725917676

gef> directory openssh-9.2p1/
Source directories searched: /openssh-9.2p1:$cdir:$cwd
```

With debug symbols loaded, the backtrace becomes more meaningful:

![img-description](backtrace.png){: width="1000" }


Now we can analyze each frame and print the context, frame 10 is the last one for which we have symbols:

![img-description](frame10.png){: width="1000" }

As we can see the last function called is `RSA_public_decrypt`.

#### Step 4: Finding the backdoor function

So if we did paid attention at how the xz backdoor did its thing, we will know that it used linking magic to replace the call to `RSA_public_decrypt`, with one of the functions. At this moment is when I though about static analyzing the liblzma library provided on the filesystem. 

We need to find wich function is executed when we call `RSA_public_decrypt`. For this I'll be running sshd with the malicious liblzma and attaching to it with gdb.

```bash
rm /usr/lib/x86_64-linux-gnu/liblzma.so.5.4.1
cp /mnt/usr/lib/x86_64-linux-gnu/liblzma.so.5.4.1 /usr/lib/x86_64-linux-gnu/liblzma.so.5.4.1

# we need to run this without systemd
/sbin/sshd
gdb -p PID
```
Now we can see that liblzma is loaded with:

```bash
gef➤  vmmap liblzma
[ Legend:  Code | Heap | Stack ]
Start              End                Offset             Perm Path
0x00007f37e2fcd000 0x00007f37e2fd1000 0x0000000000000000 r-- /usr/lib/x86_64-linux-gnu/liblzma.so.5.4.1
0x00007f37e2fd1000 0x00007f37e2ff0000 0x0000000000004000 r-x /usr/lib/x86_64-linux-gnu/liblzma.so.5.4.1
0x00007f37e2ff0000 0x00007f37e2ffe000 0x0000000000023000 r-- /usr/lib/x86_64-linux-gnu/liblzma.so.5.4.1
0x00007f37e2ffe000 0x00007f37e2fff000 0x0000000000030000 r-- /usr/lib/x86_64-linux-gnu/liblzma.so.5.4.1
0x00007f37e2fff000 0x00007f37e3000000 0x0000000000031000 rw- /usr/lib/x86_64-linux-gnu/liblzma.so.5.4.1
```
First we search for the function definitions, only one is defined inside /usr/sbin/sshd scope:

```bash
gef➤  i functions RSA_public_decrypt
All functions matching regular expression "RSA_public_decrypt":

Non-debugging symbols:
0x000055e181c0b2b0  RSA_public_decrypt@plt
0x00007f37e34495a0  RSA_public_decrypt@plt
0x00007f37e35f0c90  RSA_public_decrypt
gef➤  vmmap 0x000055e181c0b2b0
[ Legend:  Code | Heap | Stack ]
Start              End                Offset             Perm Path
0x000055e181c0b000 0x000055e181cd9000 0x000000000000d000 r-x /usr/sbin/sshd
gef➤  vmmap 0x00007f37e34495a0
[ Legend:  Code | Heap | Stack ]
Start              End                Offset             Perm Path
0x00007f37e343f000 0x00007f37e36bb000 0x00000000000c5000 r-x /usr/lib/x86_64-linux-gnu/libcrypto.so.3
gef➤  vmmap 0x00007f37e35f0c90
[ Legend:  Code | Heap | Stack ]
Start              End                Offset             Perm Path
0x00007f37e343f000 0x00007f37e36bb000 0x00000000000c5000 r-x /usr/lib/x86_64-linux-gnu/libcrypto.so.3
```

And now we can print the instructions at the PLT:
```bash
gef➤  x/5i 0x000055e181c0b2b0
   0x55e181c0b2b0 <RSA_public_decrypt@plt>:	jmp    QWORD PTR [rip+0x125f62]        # 0x55e181d31218 <RSA_public_decrypt@got.plt>
   0x55e181c0b2b6 <RSA_public_decrypt@plt+6>:	push   0x28
   0x55e181c0b2bb <RSA_public_decrypt@plt+11>:	jmp    0x55e181c0b020
   0x55e181c0b2c0 <fork@plt>:	jmp    QWORD PTR [rip+0x125f5a]        # 0x55e181d31220 <fork@got.plt>
   0x55e181c0b2c6 <fork@plt+6>:	push   0x29
```
As we can see, our target function address is stored on address `0x55e181d31218`, with the following commands we can print it. See how the address for `RSA_public_decrypt` is on liblzma.so.5.4.1 which shouldn't be the case.

```bash
gef➤  x/a 0x55e181d31218
0x55e181d31218 <RSA_public_decrypt@got.plt>:	0x7f37e2fd6820
gef➤  vmmap 0x7f37e2fd6820
[ Legend:  Code | Heap | Stack ]
Start              End                Offset             Perm Path
0x00007f37e2fd1000 0x00007f37e2ff0000 0x0000000000004000 r-x /usr/lib/x86_64-linux-gnu/liblzma.so.5.4.1
```

From now on, I'll be calling this functionn `RSA_public_decrypt_hook`.

#### Step 5: Analyzing the hook function

With this info, we can now open liblzma.so.4.1 with ghidra and search for this function, we simply need to use the memory map to change the base address to the one on hour runnning environment (in this case `0x7f37e2fcd000` which we got from the previous vmmap), jump to the address of the function `0x7f37e2fd6820`, and proceed to analyze it:

![img-description](RSA_public_decrypt_hook.png){: width="1000" }

We can properly decode system calls by running the script `ResolveX86orX64LinuxSyscallsScript.java`.

As we can see, this function does the following:
1. Checks that the uid of the process is 0 (if not the proper RSA_public_decrypt will be called)
2. Allocates some memory with mmap()
3. Copies some data to that memory with memcpy()
4. Jumps into this memory with `(*ptrTarget)()` or `call r8`

For getting the source code of `ptrTarget` the easiest path is to run the backdoor, and break just before the `call` instruction is executed.

For this purpose I will be calling `RSA_public_decrypt_hook` from gdb, i have written the following c code:

```c
#include <stdio.h>
#include <stdlib.h>
#include <dlfcn.h>
#include <stdint.h>

typedef void (*func_ptr)();  // Define a function pointer type


char payload[] = {0x48,0x7a,0x40,0xc5,0x94,0x3d,0xf6,0x38,0xa8,0x18,0x13,0xe2,0xde,0x63,0x18,0xa5,0x07,0xf9,0xa0,0xba,0x2d,0xbb,0x8a,0x7b,0xa6,0x36,0x66,0xd0,0x8d,0x11,0xa6,0x5e,0xc9,0x14,0xd6,0x6f,0xf2,0x36,0x83,0x9f,0x4d,0xcd,0x71,0x1a,0x52,0x86,0x29,0x55,0x58,0x58,0xd1,0xb7,0xf9,0xa7,0xc2,0x0d,0x36,0xde,0x0e,0x19,0xea,0xa3,0x05,0x96,0xda,0x59,0xb9,0xb9,0x0d,0x17,0x8f,0x41,0x42,0x3d,0x7e,0xeb,0x15,0x07,0xb5,0xdc,0x03,0x9c,0xb8,0x49,0xa8,0x59,0x98,0xcc,0x61,0x1f,0x37,0x9b,0x4d,0x0a,0xf2,0x50,0xbd,0xab,0x37,0x2d,0x0c,0x37,0x16,0xe2,0xa3,0x40,0x4b,0x11,0x51,0xad,0x49,0xa9,0x4a,0x1a,0x95,0x8e,0x26,0x6b,0x98,0x91,0x6a,0xb0,0xa7,0x08,0xee,0xcb,0xd0,0xf3,0xd2,0x01,0x47,0xd0,0x5f,0x9e,0x67,0xa9,0xf1,0x2d,0x6c,0x15,0x8d,0x6f,0xa5,0xbd,0x32,0xd5,0x8a,0x17,0x6d,0x7e,0x61,0x85,0x16,0xe6,0x6c,0x31,0x48,0x14,0x03,0x8a,0x9a,0x4f,0x80,0xac,0xea,0x50,0x68,0x5c,0x2f,0x74,0x0d,0x9f,0x00,0x0a,0xb6,0x8a,0xda,0x79,0x42,0x3a,0x70,0x2a,0x99,0x17,0xfb,0xbd,0x95,0x53,0x51,0x63,0xbc,0x83,0x94,0xff,0x7b,0x81,0x70,0xb7,0x82,0x64,0x1e,0x3c,0x1f,0xa0,0xad,0x4f,0x7a,0xe0,0xe3,0x81,0xed,0xbd,0x39,0x5e,0xab,0x40,0x10,0xe3,0x45,0x2d,0xb2,0xdc,0x0f,0xbc,0x79,0x01,0x15,0x03,0x71,0x02,0xe8,0x2e,0x9f,0xa9,0x15,0x43,0x06,0x15,0xa3,0xc3,0x94,0x78,0x21,0x96,0x69,0x73,0x9e,0xaa,0x72,0x1b,0x7c,0x52,0x32,0x6b,0x23,0xad,0x14,0xef,0x31,0xfd,0x2a,0xcf,0xa3,0xae,0xf9,0xde,0xca,0x0c,0xb6,0x57,0x41,0xa7,0x73,0xce,0x9f,0xb6,0x60,0x96,0x3d,0xe2,0xac,0x4b,0x68,0x20,0x78,0x63,0x8e,0xe1,0x71,0xc1,0x20,0xe6,0xc1,0x8b,0x87,0x12,0xa6,0x45,0xe0,0x48,0x64,0x1c,0xc9,0xb2,0x81,0xeb,0x3e,0x3d,0x3e,0x48,0x54,0x64,0x83,0xa9,0x98,0x22,0xb1,0x86,0xa2,0x2e,0x84,0x17,0x25,0xc8,0xcb,0xc9,0xba,0x05,0xd2,0xca,0x8f,0x1e,0xc0,0x69,0x1f,0x9d,0x42,0x75,0x6f,0x7b,0x42,0x3e,0xa1,0x9c,0xe6,0x85,0xdf,0x1f,0x35,0x15,0x7b,0x10,0x11,0xac,0x66,0x5d,0xc3,0xf0,0x14,0x7c,0xc6,0xc5,0x6c,0xa5,0x6b,0x90,0xc4,0x57,0x06,0x83,0xf8,0xd8,0x69,0x2d,0xea,0x68,0x1f,0xfb,0x14,0x77,0x33,0xed,0x77,0x53,0x77,0x36,0x27,0xae,0xc3,0xdc,0x25,0x57,0xe7,0xd9,0x5b,0xd5,0x98,0x3b,0xa7,0x84,0xaf,0x12,0xfb,0xfe,0x06,0x08,0x70,0x27,0x2d,0xe4,0xb9,0x01,0xa6,0x7c,0x5b,0x1e,0x8c,0xf7,0x49,0x74,0x3a,0xad,0xf0,0x03,0x71,0xc4,0x96,0x5f,0xed,0x5d,0x53,0x3b,0x39,0x04,0x77,0x85,0xf1,0x84,0x3a,0xfc,0xda,0xbe,0x7c,0x10,0x69,0xbd,0xe7,0x91,0xd4,0x3a,0xc6,0xbd,0xe9,0xcb,0x0a,0x7b,0xdd,0xc7,0x75,0xb4,0x00,0x6a,0x53,0x63,0x43,0x31,0x7c,0x3f,0x32,0xdf,0xae,0x7d,0xd6,0xda,0x84,0xb5,0x3e,0xc7,0x17,0x38,0x2b,0x84,0x67,0x0e,0xfe,0xec,0xa5,0x90,0x22,0x8d,0xfa,0x00,0x76,0xa5,0x97};

int vik0t0rCallPayload(uintptr_t base_address) {
    uintptr_t offset = 0x0;                   // Offset to the function
    func_ptr func = (func_ptr)(base_address + offset);

    // Call the function at the offset
    printf("Calling function...\n");
    func(NULL, &payload, NULL, NULL, NULL);  // Invoke the function

    return 0;
}
```

The payload array is the original signature which called `RSA_public_decrypt` and caused the coredump. I was sending this to the function because I though that it would contain the commands to be executed, as the original xz utils backdoor, but later I found that the parameters are ignored.


Then we can compile with: `gcc -shared -o call.so -fPIC -g call.c`

And now we are ready to attach with gdb and call our target function:

We need to find the address of `RSA_public_decrypt_hook`. We can use some of the other symbols defined at `liblzma.so`. For example `lzma_vli_size` is `0x450` bytes before. 

First we get the address to lzma_vli_size
```bash
gef➤  info functions lzma_vli_size
All functions matching regular expression "lzma_vli_size":

Non-debugging symbols:
0x00007f37e2fd1c30  lzma_vli_size@plt
0x00007f37e2fd63d0  lzma_vli_size
```
Then we calculate the address to `RSA_public_decrypt_hook`

```bash
addr = 0x7f37e2fd63d0 + 0x450 = 0x7f37e2fd6820
```

And now we can load our shared library and call the function:

```bash
call dlopen("/root/call.so",1)
break *0x7f37e2fd6820 
call vik0t0rCallPayload(0x7f37e2fd6820)
```
Now we just need to break on the instruction `call r8`:

```bash
# first print the assembly
x/100i 0x7f37e2fd6820
# break on call r8
break 0x7f37e2fd693c
```

We see that r8 point to a RWX region, supicious at least...

```bash
gef➤  vmmap $r8
[ Legend:  Code | Heap | Stack ]
Start              End                Offset             Perm Path
0x00007f37e2d94000 0x00007f37e2d95000 0x0000000000000000 rwx 
```

Now we can dump that memory region to a file:

```bash
dump memory payload.bin 0x00007f37e2d94000 0x00007f37e2d95000
```

#### Step 6: Analyzing the shellcode

Here there is the anotated ghidra decompilation of the shellcode:

![img-description](payload1.png){: width="1000" }

Looking through the shellcode and analyzing which syscalls we arrive to the conclusion that this is a backdoor which:

1. opens a socket and connects to 10.0.2.15:1337 (we can see this ip by running the shellcode and sniffing the traffic)
2. reads 32 bytes (a chacha20 key) from the socket
3. reads 12 bytes (chacha20 nonce) from the socket
4. reads 4 bytes = an integer N (length of the path to read)
5. reads N bytes from the socket (path of the file to read)
6. opens the file 
7. reads up to 128 bytes from the file
8. strlen's the file contents
9. chacha20 encrypt the file
10. send the file to the socket

The coredump happens after the execution of this payload, so the ciphered file contents must exist somewhere on the coredump.

#### Step 7: Interacting with the shellcode

For dynamically analyzing the shellcode I added the target `sudo ip addr add 10.0.2.15/24 dev enp0s3` and wrote the following python script (with the help of chatgpt of course), the key and nonce are extracted from the core dump, it is explained on the next section:

```python
import socket
import struct

def recv_all(sock, length):
    """Receive exactly `length` bytes from the socket."""
    data = b''
    while len(data) < length:
        packet = sock.recv(length - len(data))  # Get what's remaining
        if not packet:
            # Connection closed or error
            raise ConnectionError("Connection closed before receiving all data")
        data += packet
    return data

# Function to generate a fixed ChaCha20 key and nonce
def get_fixed_chacha20_key_nonce():
    #key = b'\x22' * 32  # 32 bytes key of all 2s
#    key = bytes.fromhex("94 3d f6 38 a8 18 13 e2 de 63 18 a5 07 f9 a0 ba 2d bb 8a 7b a6 36 66 d0 8d 11 a6 5e c9 14 d6 6f".strip(" "))
#    nonce = bytes.fromhex("f2 36 83 9f 4d cd 71 1a 52 86 29 55".strip(" "))
    key = bytes.fromhex("8d ec 91 12 eb 76 0e da 7c 7d 87 a4 43 27 1c 35 d9 e0 cb 87 89 93 b4 d9 04 ae f9 34 fa 21 66 d7")
    nonce = bytes.fromhex("11 11 11 11 11 11 11 11 11 11 11 11")

    #nonce = b'\x11' * 12  # 12 bytes nonce of all 0s
    return key, nonce

def main():
    # Create a socket
    server_socket = socket.socket(socket.AF_INET, socket.SOCK_STREAM)
    server_socket.setsockopt(socket.SOL_SOCKET, socket.SO_REUSEADDR,1)
    server_socket.bind(('10.0.2.15', 1337))
    server_socket.listen(1)
    print('Server listening on 10.0.2.15:1337')

    while True:
        client_socket, addr = server_socket.accept()
        print(f"Connection accepted from {addr}")

        # Get the fixed ChaCha20 key and nonce
        key, nonce = get_fixed_chacha20_key_nonce()

        # Send the 32-byte key and 12-byte nonce
        client_socket.sendall(key)
        client_socket.sendall(nonce)

        # Prepare the string to send
        string_to_send = b"/a.txt"
        string_length = len(string_to_send)

        # Send a 4-byte uint32 (length of the string)
        client_socket.sendall(struct.pack('!I', string_length))

        # Send the actual string
        client_socket.sendall(string_to_send)

        # Wait to receive a 4-byte uint32 (length of buffer to receive)
        buffer_length_data = recv_all(client_socket, 4)
        print("Received: ", buffer_length_data)
        buffer_length = int.from_bytes(buffer_length_data, byteorder="little")
        print("Receiving n bytes: ", buffer_length)
        # Receive the encrypted buffer
        encrypted_message = recv_all(client_socket, buffer_length)

        # Decrypt the message using ChaCha20
        print(f"Received bytes: {encrypted_message}")

        # Close the client connection
        client_socket.close()

if __name__ == "__main__":
    main()

```

I also tried to decipher the received buffer unsuccesfully. They must be using a modified version of the chacha20 cipher!

#### Step 8: Recovering secrest from the stack

To find the location for the key, we will analyze the stack locations and try to find them on the coredump.

In the following image see how key, nonce, PATH and readedFileContents are one after another on the stack:

![img-description](stack.png){: width="1000" }

Now we print the stack from the core dump, see how `/root/certificate_authority_signing_key.txt` seems like the exfiltrated file, so it must be the offset for PATH:

```bash
gdb /usr/sbin/sshd /mnt/var/lib/systemd/coredump/sshd.core.93794.0.0.11.1725917676

gef➤  hexdump byte $rsp-0x1500 --size 0x400

x00007ffcc6600998     04 5f 78 19 4a 7f 00 00 d0 0a 7b 19 00 00 00 80    ._x.J.....{.....
0x00007ffcc66009a8     02 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00    ................
0x00007ffcc66009b8     00 04 00 00 00 00 00 00 00 20 01 19 4a 7f 00 00    ......... ..J...
0x00007ffcc66009c8     d0 0a 7b 19 4a 7f 00 00 00 00 00 00 00 00 00 00    ..{.J...........
0x00007ffcc66009d8     75 60 78 19 4a 7f 00 00 02 00 00 00 00 00 00 00    u`x.J...........
0x00007ffcc66009e8     02 00 00 90 00 00 00 00 60 0a 60 c6 fc 7f 00 00    ........`.`.....
0x00007ffcc66009f8     00 00 00 00 00 00 00 00 00 00 00 00 02 00 00 90    ................
0x00007ffcc6600a08     02 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00    ................
0x00007ffcc6600a18     70 0a 60 c6 fc 7f 00 00 00 00 00 00 00 00 00 00    p.`.............
0x00007ffcc6600a28     00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00    ................
0x00007ffcc6600a38     00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00    ................
0x00007ffcc6600a48     00 00 00 10 00 00 00 00 00 00 00 00 00 00 00 00    ................
0x00007ffcc6600a58     30 90 58 6d b4 55 00 00 b0 2e 60 c6 fc 7f 00 00    0.Xm.U....`.....
0x00007ffcc6600a68     a4 6c 78 19 4a 7f 00 00 b0 18 7b 19 4a 7f 00 00    .lx.J.....{.J...
0x00007ffcc6600a78     0d 8e 1e 82 00 00 00 00 a0 f1 77 c6 fc 7f 00 00    ..........w.....
0x00007ffcc6600a88     b8 f1 77 c6 fc 7f 00 00 44 0b 60 c6 fc 7f 00 00    ..w.....D.`.....
0x00007ffcc6600a98     38 42 eb 18 4a 7f 00 00 02 00 00 00 00 00 00 00    8B..J...........
0x00007ffcc6600aa8     38 42 eb 18 4a 7f 00 00 00 00 00 00 00 00 00 00    8B..J...........
0x00007ffcc6600ab8     40 10 60 c6 fc 7f 00 00 25 00 00 00 00 00 00 00    @.`.....%.......
0x00007ffcc6600ac8     c0 11 60 c6 fc 7f 00 00 57 71 7c 6c b4 55 00 00    ..`.....Wq|l.U..
0x00007ffcc6600ad8     55 71 7c 6c b4 55 00 00 02 00 00 00 00 00 00 00    Uq|l.U..........
0x00007ffcc6600ae8     dd d9 e8 18 4a 7f 00 00 4d 71 7c 6c b4 55 00 00    ....J...Mq|l.U..
0x00007ffcc6600af8     00 00 00 00 4a 7f 00 00 68 0d 00 00 00 00 00 00    ....J...h.......
0x00007ffcc6600b08     00 00 00 00 4a 7f 00 00 00 00 00 00 00 00 00 00    ....J...........
0x00007ffcc6600b18     00 00 00 00 fc 7f 00 00 00 00 00 00 20 00 00 00    ............ ...
0x00007ffcc6600b28     00 00 00 00 02 00 00 00 4d 71 7c 6c b4 55 00 00    ........Mq|l.U..
0x00007ffcc6600b38     00 00 00 00 03 00 00 00 00 00 00 00 00 00 00 00    ................
0x00007ffcc6600b48     00 00 00 00 02 00 00 00 ff ff ff ff ff ff ff ff    ................
0x00007ffcc6600b58     00 00 00 00 00 00 00 00 ff ff ff ff ff ff ff ff    ................
0x00007ffcc6600b68     00 00 00 00 00 00 00 00 57 71 7c 6c b4 55 00 00    ........Wq|l.U..
0x00007ffcc6600b78     0a 00 00 00 00 00 00 00 34 ca 7b 6c b4 55 00 00    ........4.{l.U..
0x00007ffcc6600b88     38 42 eb 18 4a 7f 00 00 54 71 7c 6c b4 55 00 00    8B..J...Tq|l.U..
0x00007ffcc6600b98     00 00 00 00 04 00 00 00 00 00 00 00 00 00 00 00    ................
0x00007ffcc6600ba8     30 11 60 c6 fc 7f 00 00 e0 02 00 19 4a 7f 00 00    0.`.........J...
0x00007ffcc6600bb8     40 10 60 c6 fc 7f 00 00 25 00 00 00 00 00 00 00    @.`.....%.......
0x00007ffcc6600bc8     02 00 00 00 00 00 00 00 30 11 60 c6 fc 7f 00 00    ........0.`.....
0x00007ffcc6600bd8     28 00 00 00 30 00 00 00 a0 12 60 c6 fc 7f 00 00    (...0.....`.....
0x00007ffcc6600be8     8d ec 91 12 eb 76 0e da 7c 7d 87 a4 43 27 1c 35    .....v..|}..C'.5
0x00007ffcc6600bf8     d9 e0 cb 87 89 93 b4 d9 04 ae f9 34 fa 21 66 d7    ...........4.!f.
0x00007ffcc6600c08     11 11 11 11 11 11 11 11 11 11 11 11 20 00 00 00    ............ ...
0x00007ffcc6600c18     2f 72 6f 6f 74 2f 63 65 72 74 69 66 69 63 61 74    /root/certificat
0x00007ffcc6600c28     65 5f 61 75 74 68 6f 72 69 74 79 5f 73 69 67 6e    e_authority_sign
0x00007ffcc6600c38     69 6e 67 5f 6b 65 79 2e 74 78 74 00 ff ff ff ff    ing_key.txt.....
0x00007ffcc6600c48     00 00 00 00 00 00 00 00 f0 0d 60 c6 fc 7f 00 00    ..........`.....
0x00007ffcc6600c58     01 00 00 00 00 00 00 00 05 17 60 c6 fc 7f 00 00    ..........`.....
0x00007ffcc6600c68     ff ff ff ff 00 00 00 00 c0 de e3 18 4a 7f 00 00    ............J...
0x00007ffcc6600c78     00 20 01 19 4a 7f 00 00 00 00 00 00 00 00 00 00    . ..J...........
0x00007ffcc6600c88     00 00 00 00 00 00 00 00 1e 00 00 00 00 00 00 00    ................
0x00007ffcc6600c98     b1 49 78 19 4a 7f 00 00 00 00 00 00 00 00 00 00    .Ix.J...........
0x00007ffcc6600ca8     30 11 60 c6 fc 7f 00 00 a8 0c 00 00 00 00 00 00    0.`.............
0x00007ffcc6600cb8     a8 0c 00 00 00 00 00 00 00 10 00 00 00 00 00 00    ................
0x00007ffcc6600cc8     30 00 00 00 30 00 00 00 98 21 60 c6 fc 7f 00 00    0...0....!`.....
0x00007ffcc6600cd8     d0 20 60 c6 fc 7f 00 00 e0 0d 60 c6 fc 7f 00 00    . `.......`.....
0x00007ffcc6600ce8     ab e5 78 19 4a 7f 00 00 fc ec e0 18 4a 7f 00 00    ..x.J.......J...
0x00007ffcc6600cf8     10 00 00 00 00 00 00 00 e0 2a 01 19 4a 7f 00 00    .........*..J...
0x00007ffcc6600d08     00 00 00 00 00 00 00 00 e0 0d 60 c6 fc 7f 00 00    ..........`.....
0x00007ffcc6600d18     a9 f6 34 08 42 2a 9e 1c 0c 03 a8 08 94 70 bb 8d    ..4.B*.......p..
0x00007ffcc6600d28     aa dc 6d 7b 24 ff 7f 24 7c da 83 9e 92 f7 07 1d    ..m{$..$|.......
0x00007ffcc6600d38     02 63 90 2e c1 58 00 00 d0 b4 58 6d b4 55 00 00    .c...X....Xm.U..
0x00007ffcc6600d48     20 ea 78 19 4a 7f 00 00 d0 b4 58 6d b4 55 00 00     .x.J.....Xm.U..
0x00007ffcc6600d58     30 d1 77 19 4a 7f 00 00 f0 cb 77 19 4a 7f 00 00    0.w.J.....w.J...
0x00007ffcc6600d68     e0 2a 01 19 4a 7f 00 00 00 20 01 19 4a 7f 00 00    .*..J.... ..J...
0x00007ffcc6600d78     d0 0a 7b 19 4a 7f 00 00 ac f8 18 43 c6 70 80 96    ..{.J......C.p..
0x00007ffcc6600d88     ac f8 4c a6 e9 cd ed 97 00 00 00 00 4a 7f 00 00    ..L.........J...
```
Once we have got the offset for PATH we can calculate the relative offset for the other variables and get:

* key = `8d ec 91 12 eb 76 0e da 7c 7d 87 a4 43 27 1c 35 d9 e0 cb 87 89 93 b4 d9 04 ae f9 34 fa 21 66 d7`
* nonce =  `11 11 11 11 11 11 11 11 11 11 11 11`
* readedFileContents = `a9 f6 34 08 42 2a 9e 1c 0c 03 a8 08 94 70 bb 8d aa dc 6d 7b 24 ff 7f 24 7c da 83 9e 92 f7 07 1d`

#### Step 9: Deciphering the flag

Finally I just wrote a python script to decode the flag. The script needs to calculate the cryptostream to XOR with our readedFileContents. 

To get the cryptostream I called the previous python script given the key and nonce used and reading a file full of `A` characters. That way when we XOR the returned ciphered data with all `A` we get the cryptostream used for ciphering. 

```python
ciphertext_core = bytes.fromhex("a9 f6 34 08 42 2a 9e 1c 0c 03 a8 08 94 70 bb 8d aa dc 6d 7b 24 ff 7f 24 7c da 83 9e 92 f7 07 1d")


ciphertext = b"\x9b\xc2\x0592\x12\x80>%#\xd8'\x8aB\x8f\xa2\x8f\xa9Uz\x03\xd2_\x17X\xb6\xad\xb1\xfd\xd5)1I\xa4\x89\x024Lt\x064\t\xe2\xf3\x8f\xac;\xeb\xc1j4\xf1\xfb?h\xf2\x05\xcf3_\xf6\x83\xc2\x17\x9b\xc2\x0592\x12\x80>%#\xd8'\x8aB\x8f\xa2\x8f\xa9Uz\x03\xd2_\x17X\xb6\xad\xb1\xfd\xd5)1I\xa4\x89\x024Lt\x064\t\xe2\xf3\x8f\xac;\xeb\xc1j4\xf1\xfb?h\xf2\x05\xcf3_\xf6\x83\xc2\x17"
cleartext = b"A"*128

print(len(ciphertext))
print(len(cleartext))

def xor_bytes(bytes1, bytes2):
    # Use zip to pair corresponding bytes, apply XOR, and return a new bytes object
    return bytes(b1 ^ b2 for b1, b2 in zip(bytes1, bytes2))

cryptostream = xor_bytes(ciphertext, cleartext)
print("Cryptostream: ", cryptostream.hex())
print(xor_bytes(cryptostream, ciphertext_core))
```

And we get our precious flag: `supp1y_cha1n_sund4y@flare-on.com`

## Continuation

The last post will be coming soon