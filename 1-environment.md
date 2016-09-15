# Roll your own toy UNIX-clone OS

This set of tutorials aims to take you through programming a simple UNIX-clone operating system for the x86 architecture. The tutorial uses C as the language of choice, with liberally mixed in bits of assembler. The aim is to talk you through the design and implementation decisions in making an operating system. The OS we make is monolithic in design (drivers are loaded through kernel-mode modules as opposed to user-mode programs), as this is simpler.

# 1. Environment setup

We need a base from which to design and make our kernel. Here I will be assuming that you are using a *nix system, with the GNU toolchain. If you want to use a windows system, you must either use cygwin (which is a *nix emulation environment) or DJGPP. Either way, the makefiles and commands in this tutorial may not work.

## 1.1. Directory structure

My directory is laid out as follows:

```
osdev
 |
 +-- src
 |
 +-- scripts
 |
 +-- toolchain
 |
 +-- docs
```
 
All your source files will go in src, and all your documentation (you do write documentation? ;) ) should go in docs.

## 1.2. Compiling

The examples in this tutorial should compile successfully with the GNU toolchain (gcc, ld, gas, etc). The assembly examples are written in intel syntax, which is (my personal opinion) a much more human-readable syntax than the AT&T syntax that GNU AS uses. To assemble these, you will need the [Netwide Assembler](http://web.archive.org/web/20140118193432/http://nasm.sourceforge.net/) (or NASM).

This tutorial is not a bootloader tutorial. We will be using [GRUB](http://web.archive.org/web/20140118193432/http://www.gnu.org/software/grub) to load our kernel. To do this, we need a floppy disk image with GRUB preloaded onto it. There are [tutorials](http://web.archive.org/web/20140118193432/http://www.jamesmolloy.co.uk/tutorial_html/tutorials) to do this, but, happily, I have made a standard image, which can be found [here](http://web.archive.org/web/20140118193432/http://www.jamesmolloy.co.uk/downloads/floppy.img). This goes in your 'tutorial' (or whatever you named it) directory.

## 1.3. Running

There is no alternative for bare hardware as a testbed system. Unfortunately, bare hardware is pretty pants at telling you where things went wrong (but of course, you're going to write completely bug-free code first time, aren't you?). Enter Bochs. Bochs is an open-source x86-64 emulator. When things go completely awry, bochs will tell you, and store the processor state in a logfile, which is extremely useful. Also it can be run and rebooted much faster than a real machine. My examples will be made to run well on bochs.

## 1.4. QEMU

As stated in the Introduction, we will be using QEMU as our testbed. It is as easy as calling one command and supplying our kernel binary. We will be creating and supplying it with an ISO file later on, however.

Depending on the distribution of Linux that you are running, you can install QEMU through your package manager. In my case it is Ubuntu, so I can install it like this:

``` bash
sudo apt-get install qemu qemu-system qemu-system-x86
```

As you may have noticed, I installed 3 packages. These packages will give us a full environment.

## 1.5. Useful scripts

We are going to be doing several things very often - making (compiling and linking) our project, and transferring the resulting kernel binary to our floppy disk image.

### 1.5.1. Makefile

``` makefile
# Makefile for JamesM's kernel tutorials.
# The C and C++ rules are already setup by default.
# The only one that needs changing is the assembler 
# rule, as we use nasm instead of GNU as.

SOURCES=boot.o

CFLAGS=
LDFLAGS=-Tlink.ld
ASFLAGS=-felf

all: $(SOURCES) link 

clean:
	-rm *.o kernel

link:
	ld $(LDFLAGS) -o kernel $(SOURCES)

.s.o:
	nasm $(ASFLAGS) $<

```
 
This Makefile will compile every file in SOURCES, then link them together into one ELF binary, 'kernel'. It uses a linker script, 'link.ld' to do this:

### 1.5.2. Link.ld

``` linker
/* Link.ld -- Linker script for the kernel - ensure everything goes in the */
/*            Correct place.  */
/*            Original file taken from Bran's Kernel Development */
/*            tutorials: http://www.osdever.net/bkerndev/index.php. */

ENTRY(start)
SECTIONS
{
  .text 0x100000 :
  {
    code = .; _code = .; __code = .;
    *(.text)
    . = ALIGN(4096);
  }

  .data :
  {
     data = .; _data = .; __data = .;
     *(.data)
     *(.rodata)
     . = ALIGN(4096);
  }

  .bss :
  {
    bss = .; _bss = .; __bss = .;
    *(.bss)
    . = ALIGN(4096);
  }

  end = .; _end = .; __end = .;
```

This script tells LD how to set up our kernel image. Firstly it tells LD that the start location of our binary should be the symbol 'start'. It then tells LD that the .text section (that's where all your code goes) should be first, and should start at 0x100000 (1MB). The .data (initialised static data) and the .bss (uninitialised static data) should be next, and each should be page-aligned (ALIGN(4096)). Linux GCC also adds in an extra data section: .rodata. This is for read-only initialised data, such as constants. For simplicity we simply bundle this in with the .data section.

### 1.5.3. `update_image.sh`

#### `TODO`: Obsolete; remove.

A nice little script that will poke your new kernel binary into the floppy image file (This assumes you have made a directory /mnt). Note: you will need /sbin in your \$PATH to use losetup.

``` bash
#!/bin/bash

sudo losetup /dev/loop0 floppy.img
sudo mount /dev/loop0 /mnt
sudo cp src/kernel /mnt/kernel
sudo umount /dev/loop0
sudo losetup -d /dev/loop0
```

### 1.5.5. `build_toolchain.sh` `WIP`

##### Skip to`vagrant` if you want to avoid building a toolchain

Since we are targetting, initially at least, the `i386` (32-bit) architecture, we will need a toolchain capable of building 32-bit binaries. If you are like me, you are running a 64-bit version of your operating system, thus lacking the ability to build such binaries.

Hence, we need a script that will download the sources and, build them and install them in a convenient spot for use during the build process.

The sources that we need are the following:

* [binutils-2.27](http://ftp.gnu.org/gnu/binutils/binutils-2.27.tar.bz2)
* [gcc-6.2.0](http://ftp.gnu.org/gnu/gcc/gcc-6.2.0/gcc-6.2.0.tar.bz2)
* [nasm-2.12.02](http://www.nasm.us/pub/nasm/releasebuilds/2.12.02/nasm-2.12.02.tar.xz)
* [mpfr-3.1.4](http://www.mpfr.org/mpfr-3.1.4/mpfr-3.1.4.tar.xz)
* [mpc-1.0.3](http://www.multiprecision.org/mpc/download/mpc-1.0.3.tar.gz)
* [gmp-6.1.1](http://ftp.gnu.org/gnu/gmp/gmp-6.1.1.tar.xz)

The last three are required by GCC. The process that we will follow is similar to that described in the [Linux From Scratch](http://linuxfromscratch.org/lfs/view/development/index.html) book.

The complete script can be seen [here](https://github.com/conmarap/dart/blob/master/scripts/toolchain.sh).

#### `TODO` Complete!

### 1.5.5. Vagrant

You may also download and maintain a Vagrant virtual machine instead of building your own toolchain, although it is still a good practice to do so.

[Vagrant](https://www.vagrantup.com/) is a virtual machine that aims on lightweightness and portable environments. It is based on Oracle's [VirtualBox](https://www.virtualbox.org/wiki/Downloads). The main advantage of Vagrant is that it is easy to directly access the VM through your terminal and that no UI is needed to access it; just `SSH`!

``` bash
vagrant init ubuntu/trusty32
vagrant up --provider virtualbox
```

Once set up, you can SSH into it and install the build tools:

``` bash
sudo apt-get install build-essentials
```

1. Box [Ubuntu Trusty 32](https://atlas.hashicorp.com/ubuntu/boxes/trusty32)

****

This set of tutorials is very practical in nature. The theory is given in every section, but the majority of the tutorial deals with getting dirty and implementing the abstract ideas and mechanisms discussed everywhere. It is important to note that the kernel implemented is a teaching kernel. I know that the algorithms used are not the most space efficient or optimal. They normally are chosen for their simplicity and ease of understanding.The aim of this is to get you into the correct mindset, and to give you a grounding upon which you can work. The kernel given is extensible, and good algorithms can easily be plugged in.

If you have problems with the theory, there are plenty of sites that would be delighted to help you (most questions on OSDev forums are concerned with implementation - *"My gets function doesn't work! help!"* - A theory question is a breath of fresh air to many ;) ). Links can be found at the bottom of the page.
