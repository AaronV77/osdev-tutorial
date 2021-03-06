# Roll your own toy UNIX-clone OS

This book is based on the tutorials of James Molloy's "Roll your own toy UNIX-clone OS" which was recently taken down. In the chapters to come you will learn how to build a simple kernel, and for those who've read James' tutorials before, you will see a lot of familiar code and words. I do not claim intelectual ownership of his work, but I will be working to improve the, already, old tutorial and maintain the code as much as I can. As a result of his tutorials I was able to roll my own OS called [Dart](https://github.com/conmarap/dart), which I had extended to read ELF binaries and have its own shell, but ended up rewriting it from scratch; a lot of Dart's code will be shown in these tutorials.

## The Beginning

This set of tutorials aims to take you through programming a simple UNIX-clone operating system for the x86 architecture. The tutorial uses C as the language of choice, with liberally mixed in bits of assembler. The aim is to talk you through the design and implementation decisions in making an operating system. The OS we make is monolithic in design (drivers are loaded through kernel-mode modules as opposed to user-mode programs), as this is simpler.

This set of tutorials is very practical in nature. The theory is given in every section, but the majority of the tutorial deals with getting dirty and implementing the abstract ideas and mechanisms discussed everywhere. It is important to note that the kernel implemented is a teaching kernel. I know that the algorithms used are not the most space efficient or optimal. They normally are chosen for their simplicity and ease of understanding.The aim of this is to get you into the correct mindset, and to give you a grounding upon which you can work. The kernel given is extensible, and good algorithms can easily be plugged in.

If you have problems with the theory, there are plenty of sites that would be delighted to help you (most questions on OSDev forums are concerned with implementation - *"My gets function doesn't work! help!"* - A theory question is a breath of fresh air to many ;) ). Links can be found at the bottom of the page.

******

# Table of contents
#### 1. Environment setup
#### 2. Genesis
#### 3. The Screen
#### 4. The GDT and IDT
#### 5. IRQs and the PIT
#### 6. Paging
#### 7. The Heap
#### 8. The VFS and the initrd
#### 9. Multitasking
#### 10. User Mode

******

# Prerequisites

To compile and run the sample code provided, this series requires GCC, ld, NASM and GNU Make. NASM is an open-source x86 assembler, and is the assembler-of-choice for many x86 OS-devs. Furthermore, QEMU will be used as the main virtualization platform.

There is no point, however, in just compiling and running without comprehension. You must understand what is being coded, and as such you should have a very strong knowledge of C, especially regarding pointers. You should also know a little bit of assembly (Intel syntax is used in these tutorials), including what the EBP register is used for.

# Resources
There are plenty of resources out there if you know where to look. In particular, you should find these useful:

 * [RTFM!](https://en.wikibooks.org/wiki/Operating_System_Design) The intel manuals are a godsend
 * [The osdev.org](http://www.osdev.org/wiki) wiki and [forums](http://www.osdev.org/forum).
 * [Osdever.net](http://www.osdever.net/) has many good tutorials and papers, and in particular [Bran's kernel development tutorials](http://www.osdever.net/bkerndev/index.php), which some of the earlier code from this tutorial is based off. (I myself used these tutorials to get started, and the code is so good I haven't had to change it over the years
 * [alt.os.development](http://groups.google.co.uk/group/alt.os.development/topics) can answer many of your non-n00b questions. N00b questions are better asked on the osdever.net forums.
