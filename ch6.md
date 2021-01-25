# Mechanism:  Limited Direct Execution
- OS time shares (run one process for little bit, then run another process for little bit and so on) the CPU to virtualize it.
- challenges include
    - performance: how to run many process without excessive overhead.
    - control: how to run processes efficiently while retaining control over CPU
        - control is really important for OS, since it is in charge of resources.
        - without control, and process can just run forever and take over the machine or access sensitive information.
- high performance while keeping control is one of the central challenges of building an OS.

## Basic Technique - Limited direct execution
- Direct execution: OS does all the setup (memory, arguments, I/O) as previously described and executes call main(). Program runs directly on CPU. OS frees resources and removes process from list.

### problem 1: restricted operations
- direct execution is obviously fast, but what if the process tries a restricted operation like I/O or other memory. 
    - processor modes are a solution
        - in user mode, instructions are restricted.
        - in Kernel mode, code can do what it likes such as I/O, restricted instructions, etc.
        - To allow user processes to perform priveleged operations, the concept of system call was invented.
            - hardware assists OS by providing different modes of execution: special instructions to trap into the kernel and return-from-trap back to user-mode programs are provided.
            - Also provided are instructions that OS can use to tell the hardware whern the trap table resided in memory.
        - To execute a system call, program must execute a special trap instruction. 
            - Jumps into kernel and raises privelege to kernel mode. OS can do the required work and call return-from-trap instruction, which returns control to user program and reduces privelege to user mode.
        - Hardware needs to be careful when executing trap in that it must make sure to save enough of the caller's registers to be able to return correctly OS issues return-from-trap.
            - on x86, processor will push the Program counter, flags, and few other registers onto a per-process kernel stack. Basic concepts are similar on other platforms.
        - Using trap table, OS can specify what code (trap handler) to execute for different traps. Cannot give the user this ability because that is a bad idea.
            - additionally, OS must check for mailicious or erroneous inputs and not allow access to restricted areas.
        - To specify the exact system call, a system-call number is usually assigned to each system call. 
            - OS checks and verifies it corresponds to a valid number and then executes the corresponding system call.
            - this indirection serves as protection since user cannot specify exact address. 
        - The trap table operation is really important, and hence privileged. OS won't allow to do this in user mode.
- 2 steps for Limited direct execution protocol
    - at boot, kernel specifies trap tables by privileged instruction, and hardware remembers it.
    - When process wants a system call, it traps back into OS which handles the request and returns control to the program via return-from-trap call. 

### problem 2: switching between processes
- When process runs, OS is by definition not running. How does OS regain control?

- cooperative approach - wait for system calls or illegal operations like divide by 0. Used by Mac OS in the past, but one process can keep control infinitely if it doesn't do the things required for OS to gain control.
- non cooperative: OS takes control
    - Without more help from harware, OS can't do much at all when a process refuses to make system calls.
    - cooperative approach had the only solution of rebooting the machine.
    - The solution is a timer interrupt. A timer can be programmed to raise an interrupt every so many milliseconds
        - When interrupt is raised, the process running is halted and a pre-configured interrupt handler in the OS runs. At this point OS has regained control of CPU. 
        - So OS must inform the hardware of which code to run when this interrput happens and OS must also start the timer during boot sequence.
        - this timer can be turned off.
        - hardware has responsibility to save enough state that a return-from-trap is later possible.
- saving and restoring context
    - after regaining control, OS must decide whether that program should continue to run or whether another program should run. The part of OS called scheduler decides this.
    - If the decision to switch is made, OS executes the low level mechanism known as context switch.
        -  context switch is conceptually simple: all the OS has to do is save a few register values for the currently-executing process (onto its kernel stack, for example) and restore a few for the soon-to-be-executing process (from its kernel stack). By doing so, the OS thus ensures that when the return-from-trap instruction is finally executed, instead of returning to the process that was running, the system resumes execution of another process.
            - To save the context of the currently-running process, the OS will execute some low-level assembly code to save the general purpose registers, PC, and the kernel stack pointer of the currently-running process
            - It then restores those registers and pointers for the next process to execute.
            - By switching stacks, the kernel enters the call to the switch code in the context of one process (the one that was interrupted) and returns in the context of another (the soon-to-be-executing one). 
                - When OS decides to run another process, it has to explicitly save kernel registers so that when it switches from A to B, the kernel stack for B is restored even though originally A was running. 

### Concurrency
- What if another interrupt happend when handling one interrupt.
    - OS might disable interrupts when processing an interrupt. So CPU won't get other interrupts. This can be bad if too many interrupts are lost.
    - OS can have sophisticated locking schemes to protect concurrent access to internal data structures. This enables multiple activities to be on-going withing the kernel at the same time, particularly useful on multiprocessors.
        - locking can be complicated and lead to a variety of interesting and hard-to-find bugs.
    


        
        


