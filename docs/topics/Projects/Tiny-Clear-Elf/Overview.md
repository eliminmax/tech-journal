<!--
SPDX-FileCopyrightText: 2022 - 2025 Eli Array Minkoff

SPDX-License-Identifier: CC-BY-SA-4.0
-->

# Tiny Clear Elf

```
 ___________________________________________
/    _|_o._      _                   _  __  \
|     |_|| |\/__| |___ __ _ _ _  ___| |/ _| |
|   /\      // _| / -_) _` | '_|/ -_) |  _| |
| <(oo)>     \__|_\___\__,_|_|  \___|_|_|   |
\___________________________________________/
```

Minimal binaries to clear the terminal, along with its scrollback buffer - [available here](https://github.com/eliminmax/tiny-clear-elf).

## About this Project

I created a series of 9 unique executables that clear the terminal and scrollback buffer. The scope of this project was originally defined in the initial commit to the Tiny Clear Elf GitHub repository (commit [8b1336d](https://github.com/eliminmax/tiny-clear-elf/tree/8b1336d8d66523ed227150bbb5f7e1a1461b040d), on the 22nd of August, 2022). The full content of the README.md file, the only file in that commit, was as follows:

```markdown
# Tiny Clear
The goal of this project is to create the smallest possible ELF binary for all architectures that are officially supported by Debian GNU/Linux Bullseye. Why? I feel like it.

The way they work is simple - they print out the following hexadecimal data to stdout: `1b 5b 48 1b 5b 4a 1b 5b 33 4a`.

Breaking it down further, what that does is print 3 ANSI escape sequences:
1. `ESC[H` - move the cursor to position 0,0
2. `ESC[J` - clear the screen
3. `ESC[3J` - clear any scrollback lines

Note that these utilities are hand-written in a hex editor, so source code distribution is not really an applicable concept. As such, I am not releasing them under a formal license. If you want to use them, you can do so however you want, but I'd appreciate it if you'd let me know - that's not a requirement, just something I'm curious about.

I can't test all of these on real hardware, so I plan on using QEMU to test them. I also should note that am writing this README having never worked with pure assembly of any kind in any capacity, so I might be biting off more than I can chew with this project. For now, this repo is a statement of intent rather than a finished project.
```

While I think most of what I said there was accurate about my plan, there was a bit more to the motivation than "why not". If it was just about the smallest possible binaries, I could implement a program that does nothing, and call it a day. I could say it's about the smallest possible *useful* binary, but that's not why I chose to implement `clear`.

## Why `clear`, specifically?

The first seed of inspiration is that the `clear` command does not work the way I expected in my terminal emulator of choice.

For a few reasons, I use the kitty terminal, and, despite what the man page for `clear` states, it does not clear the scrollback buffer. This is by design - the `clear` command tries to clear the scrollback by looking at the terminfo for an escape sequence labeled "`E3`", and writing it out. Kitty's `terminfo.py` file, which is used to generate it's terminfo, deliberately does not define the `E3` sequence, as Kovid Goyal, the main developer of kitty, [believes that it should not be the default behavior](https://github.com/kovidgoyal/kitty/issues/6255#issuecomment-1539308742):

> This is by design. clear's default behavior is wrong. It should clear only the screen by default and have an option to clear scrollback. Destroying thousands of lines of scrollback accidentally is simply not good behavior.

While I see the point is coming from, and agree with the argument about default behavior, when I run `clear`, I am used to having the scrollback cleared, and do want that. At the time I started the tiny-clear-elf project, I was not aware of Kovid Goyal's reasoning, but I did find [this github issue](https://github.com/kovidgoyal/kitty/issues/268).

I have been interested in using ANSI escape sequences to colorize simple shell and/or python scripts of mine, as I find large amounts of text info to be difficult to parse without colors to help me keep track of where I am and see what's what, so I like to color-code my shell script output. Reading the issue I linked above, I recognized the suggested ways to clear the scrollback buffer as ANSI escape sequences, and, through testing different sequences on different terminal emulators, landed on the sequence I described in the README, based on [this ANSI Escape Codes Gist](https://gist.github.com/fnky/458719343aabd01cfb17a3a4f7296797), which documents all three of the sequences used in a `tiny-clear-elf` executable, among many others.

I had defined a shell alias for clear that included those or some similar set of escape sequences - I do not recall the exact sequence used. For some reason, though, I did not feel satisfied with that solution. I was curious about what the smallest possible program that wrote those escape sequences to the standard output, and on a whim, I decided to create what would become the tiny-clear-elf project to find out. When the time came to figure out my college Capstone project, it seemed like a good way to do this in the context of my studies.

## Timeline

Based on cross-referencing my recollection of the project with a `git log` of the tiny-clear-elf repository, I have reconstructed the following timeline of the project:

### August 2022: Planning

In August 2022, I began working on clarifying the scope of the project, and figuring out what tools to use. Because I was doing this project as part of my senior year Capstone project, it couldn't be too trivial. Partly because of that, and partly due to a desire to take on a challenge, I decided to do 2 things to make the project harder.

First, I decided that rather than writing the program in assembly, which is already quite low level, I would use a hex editor, ultimately choosing [`hed`](https://github.com/fr0zn/hed), a minimalist hex editor inspired by vim.

Second, I decided that I would support every architecture officially supported by Debian 11 "Bullseye", the stable release at the time, to ensure that there was more of a challenge.

### September to October 2022: x86 Implementations

As part of Champlain's capstone process, at least for my major, was an exploration of a few potential topics, with the goal of ensuring that a capstone project was a good choice. For my exploration of what at the time I called "Tiny Clear", I decided to attempt to implement the `x86` versions - `amd64` and `i386`, to use the architecture names used by Debian.

I started with `amd64`, as it seemed to be the logical choice, as it was the architecture of the computer I was running on. I didn't realize how poor a choice it was until much later.

Armed with [A Whirlwind Tutorial on Creating Really Teensy ELF Executables for Linux](https://www.muppetlabs.com/~breadbox/software/tiny/teensy.html), a great exploration of ELF files and how the Linux kernel handles them, `hed`, and a few tools from [the Radare2 reverse engineering toolkit](https://github.com/radareorg/radare2/releases), I jumped right in.

I used `rasm2` to translate between assembly instructions and hex, `rax2` to translate between hex, binary, and decimal, and `r2` to debug the programs as I was working on them.

The first step was to understand the ELF format, or at least enough about it to get things working. This was difficult, as I'd had never touched anything this low level before. To that end, on September 4th, I downloaded the Debian 11 build of BusyBox, and looked at the first 64 bytes of the `busybox` executable - the ELF header. By cross-referencing the `elf.h` header file, the "whirlwind tutorial" mentioned above, and the `ELF(5)` man page, I was able to eventually document the purpose of every single one of those 64 bytes.

As I did that, I worked on creating an ELF header in `hed`, and got to the point where the `file` command, which reports on file types, recognized it as an x86-64 ELF file using the SYSV Application Binary Interface. While I was at it, I renamed it from "Tiny Clear" to "Tiny Clear ELF".

By the end of October, I had a working `amd64` tiny-clear-elf implementation, which I began to use on my systems, and had adapted it to target 32-bit systems, making it an `i386` system.

### October - December 2022: Fixes and early ARM

During this period, I worked on documentation, fixed an issue where the `i386` was exiting with the wrong exit code, saved some bytes in the 2 x86 implementations, and worked on the `armel`/`armhf` implementation - both are variants of the 32-bit ARM architecture, but `armhf` includes newer instructions and assumes the presence of hardware floating-point units, while `armel` doesn't use hardware floating point instructions, and is limited to an older version of ARM, to support older devices.

The initial implementation for both used instructions that I did not realize were not supported on some older ARM versions still supported by Debian's `armel` architecture, meaning that it was technically invalid. I did not catch that until over a year later.

### January - February 2023: Remaining architectures

In January through early February I completed support for the remaining Debian Bullseye architectures, starting with `arm64`, then going to `ppc64el`, then `s390x`, and finally `mipsel` and `mips64el`. At some point in this period, I switched from generating the encodings for the instructions I needed to use with `rasm2`, then hand-typing them in a hex editor, to looking into how the instructions were encoded, and figuring it out by hand, using `rasm2` to check my work. The MIPS implementations were the easiest, not just because I did them last, but because MIPS is a much easier architecture to understand. With the final target architecture added, I believed that my work on the binaries themselves was done, and I tagged the git repository as `v1.0.0`.

### April - May 2023: Presentation

In preparation for presenting my capstone, I wrote a script to print info about the different implementations in. I also wrote a simple C program that works the same way as a tiny clear ELF implementation - if statically linked without the default startup files, it will make the same 2 system calls, but it is a much larger binary.

### July 2023: Cleanup and Scope Update

I'd done it. I'd graduated, but now I was stuck. I had some health issues keeping me from looking for work right away, and I decided, while sick and bored at home, to do some work on the README.md file, and change the scope of the project from targeting architectures supported by Debian 11 "Bullseye" to Debian 12 "Bookworm". This was not a substantial change, as the list of supported architectures did not change, so it required no actual changes to the binaries.

### August 2023: Debian Project Changes

On July 24th, 2023, [it was announced that "riscv64 is now an official architecture" for Debian](https://lists.debian.org/debian-riscv/2023/07/msg00053.html). If memory serves, I found out about this in August, and decided to try to support any new architectures in new Debian versions, preferably before they reach stable status, at least for as long as I remain interested. I had a working riscv64 implementation within a few days of that decision.

### February - March 2024: Version 2.0

I decided to revisit the existing implementations, and try to save a few bytes here and there. I quickly realized that some of the instructions used in the `armel` version were invalid in some versions of ARM that were supported by Debian as part of `armel`, thus making it an invalid implementation by my own rules.

I read up on 32-bit ARM's so-called Thumb instructions, which can encode ARM instructions in either 16 or 32 bits, rather than just 32 bits. Using that, not all instructions can be shortened, but some can. By switching over to Thumb instructions, I was able to make a valid `armhf` version that was 16 bytes shorter than the previous one.

I also learned that RISC V has an optional architecture extension that adds similarly shortened instructions - the "C" extension, for "Compressed". Because Debian's `riscv64` minimum hardware requirements include support for the "C" extension, it is allowable to make use of that. Unlike ARM's thumb instructions, "C" extension instructions can be freely mixed with standard full-length instructions, but not all RISC V instructions have compressed forms. By encoding supported instructions in the "C" extension's compressed forms, I was able to save 8 bytes.

After that, I looked into `pc`-relative addressing. There's a special CPU register on most architectures called `pc`, which holds the address of the current instruction, but typically can't be interacted with using normal instructions. In the `armhf` version, I'd been using a half-length and full-length instruction together to load a memory address into a register, but it turns out that the `ADR` instruction can be used to add an offset value to the contents of `pc`, saving the result to a register. By switching to the Thumb encoding of that instruction, I save a further 4 bytes. As a bonus, it no longer used instructions which were invalid for Debian's `armel` architecture. While 64-bit ARM does not have Thumb instructions, it does have the `ADR` instruction, and I was similarly able to save 4 bytes by switching to it. I looked at the other architectures, but either they didn't support setting a register to a `pc`-relative value at all, or did not support it in a way that would save me bytes.

While looking into `pc`-relative addressing, I was also looking into various tricks to save on bytes for x86 systems. Because x86 uses variable-length instructions, it was harder to reason about, but it turns out that there are tricks to save bytes while using multiple instructions to replace one instruction. It was complex and beyond the scope of this overview, but I was able to save 9 bytes on the `amd64` implementation, and 8 bytes on the `i386` implementation.

Lastly, I added some explicitly dependency checking to the `presentation.sh` script.

I then tagged the repository as `v2.0.0`, and decided that my work on the project was done for the time being.

### March 2024 - April 2024: Minor tweaks to presentation script

While I considered work on the actual binaries to be more-or-less complete, I continued to tweak the `presentation.sh` script, making it check an environment variable to get it to explicitly use `qemu-user` programs instead of relying on `binfmt_misc` to execute foregin binaries, and use the standard `\033` instead of `\e` when running printf commands, to try to make it work more portably - mainly because I wanted to run it in the Termux command-line environment for Android. It turns out that not all of the needed `qemu-user` binaries are available anyway, but at least I tried.

### February 2025

I discovered that the sequence `1b 63` (the escape character, followed by the character `c`), can replace the first 2 escape sequences. By switching to that sequence, I was able to cut all of the binaries down by 4 bytes.

I then reworked the architecture-specific READMEs to reflect that, and fixed a few small issues with the READMEs I noticed in the process.

I tagged the repository as `v3.0.0`. Because I thought that the binaries were complete in February 2023, but I went back to improve them, then I again thought that they were complete in March of 2024, I decided not to declare the project complete.
