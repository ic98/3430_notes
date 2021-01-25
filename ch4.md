# The abstraction: The process

- a process, informally, is a running program.
    - How to provide the illusion of many CPUs with only few physical CPUs?
    - Virtualization of the CPU does this. 
        - OS runs one process, stops it and runs another and so on.
        - OS can promote the illusion of many CPUs this way when in fact there is one or a few physical CPUs.
        - Known as time sharing of the CPU - allows user to run as many concurrent processes as they want.
            - Potential cost is performance as each process will run more slowly if the CPU(s) must be shared.
    - To implement virtualization, OS needs low level machinery (mechanisms) and high level intelligence.
        - mechanisms are low level methods or protocols that implement a needed piece of functionality. Eg. context switch, which gives OS ability to stop one program and start another on a given CPU.
        - On top of mechanisms is some of the intelligence, in form of Policies.
            - policies are algorithms for making decisions within OS. 
            - Eg. Scheduling policy might decide which program out of many to run first using historical information(which program has run in last minute?), workload knowledge(types of programs run), and performance metrics(is system optimized for interactive performance, or throughput?).
        
## Process
- a process is simply a running program; at any instant in time, we can summarize a process by taking an inventory of the different pieces of the system it accesses or affects during the course of its execution.
- machine state of a process: what a program can read or update when it is running.At any given time, what parts of the machine are important to theexecu-tion of this program
    - Memory is an obvious component of machine state
        - instructions are in memory, data read and to be written is also in memory
        - The memory a process can address (address space) is part of the process. 
    - Registers are also part of machine state of a process
        - Program counter (PC) (or Instruction Pointer (IP)) is a special register that points to next intruction to be executed. 
        - Stack Pointer and associate frame pointer are used to manage the stack for function params, local variables, and return addresses.
        - Programs access persistent storage too, so I/O information might include a list of the files the process has open.

## Process API
- following must be included in the interface of an OS
    - Create: OS must have some method to create new processes (used when click an icon or type a command in shell).
    - Destroy: Similarly, OS must have a method to destroy processes forcefully. Many processes run and exit normally, but user may want to kill a process.
    - Wait: Sometimes useful to wait for a process to stop running on its own.
    - Miscellaneous control: OSes also provide other options than kill or wait. Eg. a method to suspend and resume.
    - Status: Usually, there are interfaces to get status information about processes like how long it's been running and what is its current state.

## process creation
- first OS must load code and any static data like initialized variables into memory, into the address space of the process. 
    - Programs reside on disk in some kind of executable format.
    - Earlier OSes did this eagerly, all at once before running
    - Modern OSed do it lazily, loading code or data only when needed during execution.
        - Need to understand paging and swapping first
    - Some memory must be allocated for runtime stack of the program. 
    - OS may also allocate some memory for the program's heap. ]
        - OS can get involved when heap needs to be resized.
        - Usually some minimal memory is allocated for heap, even when program doesn't need it.
        - Everything is done through system calls, even though we call the wrappers like malloc() which actually leads to OS calling sbrk().
    - OS initialized stack with arguemtns, like argv and argc for main().
    - OS will also do other initialization tasks, like I/O. 
        - Unix has has 3 open descriptors, for standard input, output, and error. 
    - OS can then start the program execution from main() and transferring control of the CPU to the process. 

## process states
- States are
    - Running: iin a running state, the process is running on processor, ie. executing instruction.
    - Ready: Process is ready to run, but OS has chosen not to run it.
    - Blocked: Process has performed some operation that makes it not ready to run until some other event happens. Like I/O request to disk.

- Ready -> running  when program is scheduled
    - Running -> ready when descheduled
    - Running -> I/O request -> blocked
    - blocked -> I/O done -> ready
 
## Data structures
- Process list for all processes that are ready and some additional information to track which process is currently running. OS also tracks blocked process.
    - When IO event completes the OS should make sure to wake the correct process and ready it to run. 
- Register context will hold, for stopped process, the contents of registers. When process is stopped, registers will be saved to this memory location. OS can resume running the process by restoring these registers. This is context switch.
- System might also have inital state the process is in. A process could be placed in final state too, where it has exited but not cleaned up. Known as zombie state in UNIX.
    - final stat is useful since it allows other processes, usually parent process, to examine the return code and see if it exited successfully.
    - When finished, parent will make one final call to wait for completion of child, and also indicate to the OS that it can clean up and relevant Data structures that referred to the now-extinct process.

## SUMMMARY
- The processis the major OS abstraction of a running program. At any point in time, the process can be described by its state: the contents of memory in its address space, the contents of CPU registers(including theprogram counter and stack pointer, among others),and information about I/O (such as open files which can be read or written).
- The process API consists of calls programs can make related to processes. Typically, this includes creation, destruction, and other useful calls.
- Processes exist in one of many different process states, including running, ready to run, and blocked. Different events (e.g., getting scheduled or descheduled, or waiting for an I/O to complete) transition a process from one of these states to the other.
- A process list contains information about all processes in the system. Each entry is found in what is sometimes called a process control block(PCB), which is really just a structure that contains information about a specific process.

