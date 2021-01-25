# Introduction

## OS
- body of software responsible for making it easy to run programs (even allowing you to seemingly run many at the same time), allowing programs to share memory, enabling programs to interact with devices, and other fun stuff like that. 
- it is in charge of making sure the system operates correctly and efficiently in an easy-to-use manner
- primary way it does it is through **virtualization**
    - virtualization -  OS takes a physical resource and transforms it into a more general, powerful, easy-to-use virtual form. Sometimes OS is called Virtual machine.
    - provides interfaces (APIs) to allow user to give commands.
        - System calls to access memory, run proggrams, access other devices, and other related actiions.
            - known as standard library
    - virtualization allows programs to run concurrently, as OS manages concurrent access to instruuctions and data, and other devices like disks. OS is sometimes called resource manager.
    - CPU, memory, Disk are all resources.
    - OS must manage them fairly or with other goals.

## Virtualization is first piece
### virtulizing CPU
- OS creates an illusion of sorts, allowing many programs to run on one CPU core. The illusion is that the system has a lot of virtual CPUs, turning a few CPUs into seemingly infinite CPUs thus allowing multiple programs to run at once. 
- needs policy to decide which programs run, which run first, and so one. 
    - Policy - decides
    - Mechanism - does what policy decides

### virtualizing memory
- Modern machines present memory as a simple array of bytes. To read we need to give an address. To write, we need to also specify the data to be written along with the address. 
- Memory is accessed all the time when program runs. 
- OS virtualizes it
    - Each process accesses it's own virtual address space (that the OS maps to physical memory).
    - So memory reference within a program doesn't affect other programs.
    - Running program sees memory all for itself even though reality is that physical memory is shared

## Concurrency
- When many threads run at same time or different processes run at the same time. 
- happens in OS, but also multithreaded programs.
- Because these instructions do not executeatomically(all at once), strange things can happen. It is this problem of concurrency that we will address in great detail in the second part of this book.

## Persistence
- Memory is volatile, loses all data when power is lost (shudown, restart etc.)
- Need a more persistent storage for hardware and software to be able to store data.
- Comes in the form of I/O devices
    - Hard drives are used as common repositories of long-lived information, but SSDs are gaining more market share.
- Software that manages the disk is called file system.
    - it is thus responsible for storing any files the user creates in a reliable and efficient manner on the disks of the system.
- Unlike abstractions for CPU and Memory, OS doesn't create private virtualized disk. Rather it's assumed that users will want to share information in files
    - eg: edit file in emacs and run with gcc to get executable and then run executable.
    - need shared access to do this. emacs creates file that gcc also accesses, gcc creates execuatable that OS also accesses to run it.
- Programs access system calls which are routed to the file system which handles requests and returns some error code. 
- FS does complex work to write
    - Figures out where on the disk data will reside
    - keeps track of various structures it maintains
    - Requires I/O requests to storage device
        - A device driver is some code in the operating system that knows how to deal with a specific device. We will talk more about devices and device drivers later.
        - Non trivial
    - OS provides abstactions for this
- For performance, writes are often delayed and batched into larger groups
    - Most file systems incorporate intricate write protocols like journaling or copy-on-write to handle problems of system crashes. Writes are ordered carefully to ensure that system can restore to a reasonable state if failures occur during write sequence. 
    - FS employ many Data structures from simple lists to complex b-trees. 

## Design Goals
- There are always tradeoffs when trying to achieve many different goals.
    - Important to make the right trade-offs in building a system.
- Most basic goal is abstraction
- Another is high performance or minimizing overhead
    - Virtualization and ease-of-use, and other OS features have a cost. Goal is to provide these without excessive overheads
    - overheads can be time, space on disk and memory. Seek to minimize both, but often not possible so must make tradeoff in that case.
- Another goal is protection, often through isolation. OS allows many programs to run at once, so it must protect against programs with bad or malicious behaviour. 
- Another is high degree of reliability since everything depends on OS.
- Some other possible goals include energy efficiency, security, mobility.

## Some history
- Early OS - just libraries
    - for commonly used functions like I/O handling.
    - Ran one program at a time, controlled by human operator
    - known as batch processing. Programs set up byhh operator and run in a batch. 
    - Not interactive

- Beyond libraries: Protection
    - OS took a more central role in managing machines. Realization that OS is special since it accesses everything. Can't allow that for all programs or else privacy goes out the window.
    - System call, pioneered by Atlas computing system. Instead of OS routines as library, idea was to add a special pair of hardware instructions and hardware state to make the transition into OS more formal and controlled.
        - key difference is that system call transfers control into OS whie raising hardware privilege level
        - User apps run in user mode, meaning hardware restricts what apps can do.
            - typically no I/O, no physical memory access, no sending network packets etc.
        - When system call initiated, hardware transfers control to trap handler set by OS and simultaneously privelege is raised to kernel mode where OS has access to stuff like stuff like I/O requests, network packet sending.
        - WHen done, OS called a special return-from-trap instruction to pass control back to application.

- minicomputer and multiprogramming
    -  Instead of just running one job at a time, the OS would load a number of jobs into memory and switch rapidly between them, thus improving CPU utilization. This switching was particularly important because I/O devices were slow; having a program wait on the CPU while its I/O was being serviced was a waste ofC PU time.
    - Memory protection became important. Processes shouldn't just access any memory.

- Unix - based on multics
    - Unifying priciple of building small programs that can be connected for bigger workflows.
    - Open source early example
    - came with C programming language
    - BSD (Berkeley Systems Distribution) was created from this
    - Spread slowed and companies asserted ownership due to profit reasons. 

- Linux
    - Borrowed heavily from Unix principles and ideas, but not the codebase. Avoiding issues of legality.
    - Most modern companies like google, facebook, amazon run linux.
    - Android is based on Linux too


- Modern era (PCs)
    - one machine per desktop instead od minicomputer per workgroup.
    - early OSs like DOS were a step back, abandoning many lessons learned from the era of minicomputers, like memory protection.
    - Mac OS (v9 and before) took cooperative approach to scheduling, thus one thread in infinite loop could take over the whole system forcing a reboot.
    - But modern OSs improved and incorporated the lessons learned from OS development.
