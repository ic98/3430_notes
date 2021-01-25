# Interlude: Process API

## fork() system call
- used to create a new process.
- Strange routine.
    - The process created is an almost exact copy of the the calling program.
    - Child process doesn't start at main, but rather as if it had called fork() itself.
        - Parent received pid of child from fork()
        - child receives 0 from fork()
        - Scheduler decides which runs first, so might get different outputs.


## wait() system call
- If parent calls wait(), the system call usually won't return until the child has run and exited.
    - In some cases, it might. 

## exec() system call
- This system call is useful when you want to run a program that is different from the calling program.
- Also strange like fork()
    - loads code of executable, overwrites the current code segment (and static data) with it, and reinitializes the heap, stack, and other parts of the memory space of program.
    - Then OS passes in any arguments as argv. 
    - So NO new process created, rather current process is transformed. 
    - a successful exec() call never returns. 

## Why?
- separation of fork and exec is essential to the build a UNIX shell. It lets the shell run code after the call to fork() but before the call to exec(). This code can alter the environment of the about-to-br-run program, allowing interesting features to be built.
    - Allows pipes, input/output redirection and other cool features without changing the programs being run.
- shell is just a user program. Shows a prompt and then waits for you to type. When a command is typed, shell find executable and any arguments, calls fork() to create a child process, then some variant of exec() to run command then waits for command to complete by calling wait().
- When complete, shell returns from wait and and print out a promt and is ready for another command.
- `$prompt> wc p3.c > newfile.txt`
    - the way shell does this is simply by closing standard output and opening newfile.txt when the child is created but before calling exec(). 
        - Due to this, any output is sent to the file rather than printed on screen.
- Can also be done in code, by closing STDOUT_FILENO and opening the desired output file.
    - because UNIX systems start looking for free file descriptors at 0
- pipes are implemented in a similar way but with the pipe() system call.
    - in this cas, the output of one process is connected to that same pipe, thus the output of one process seamlessly is used as the input to the next process. 

## process control and users
- there are a lot of other interfaces for interacting with processes other than fork, exec, and wait. 
    - kill() is used to send signals to a process, including directives to pause, die, and others. 
    - UNIX keystroke combos are mapped to some these calls
        - ctrl+c: SIGINT interrupt, usually terminates
        - ctrl+z: SIGSTP stop, stops process mid execution
- signals subsystem provides a rich infrastructure to deliver external events to processes, including ways to recevie and process signals withing individual processes, and ways to send signals to individual processes and process groups. 
    - To use this, process should use the signal() system call to catch various signals
    - allows proper behaviour by program when a signal is received.
- concept of user is important. A user can create and control its own processes, the OS takes the job of managing resources for various users to meet overall system goals.
    - The superuser can kill other processes and has complete control over the system, so those priveleges should be used sparingly. 

## useful tools
- ps command allows to see which processes are running
- top displays the processes and how much CPU anmd other resources are eaten up
- kill can kill processes, killall does what you'd imagine.
- MenuMeters is a CPU meter that shows how much CPU is being used.

