SMP1: Simple Shell
==================

INSTRUCTIONS
============

In this MP, you will explore and extend a simple Unix shell interpreter.
In doing so, you will learn the basics of system calls for creating and
managing processes.


STEP 1:  Compile the shell
==========================

    chmod +x b.sh
    make
    make test   # Use in Step 5 to test your changes to the MP
    ./shell


STEP 2:  Try using the shell
============================

  Note: You need to specify the absolute paths of commands.

  Some commands to try:
    /bin/ls
    /bin/ls ..
    cd /
    /bin/pwd
    /bin/bash
    exit
    ./shell     (Note: You need to be in the smp1 directory.)
    ./shell&    (Note: You need to be in the smp1 directory.)
    ./b.sh      (Note: You need to be in the smp1 directory.)
    /bin/kill -s KILL nnnn      (Where nnnn is a process ID.)

  "./" means the current directory


STEP 3:  Study the implementation of the shell
==============================================

  In preparation for the questions in Step 4, please explore the source code
  for the shell contained in 'shell.c'.  You needn't understand every detail
  of the implementation, but try to familiarize yourself with the structure
  of the code, what it's doing, and the various library functions involved.
  Please use the 'man' command to browse the Unix manual pages describing
  functions with which you are unfamiliar.


STEP 4:  Questions
==================

  1. Why is it necessary to implement a change directory 'cd' command in
     the shell?  Could it be implemented by an external program instead?
         Change directory is a command used to maneuver between your directories in your shell.
         Other programs have a built in version of cd, but cd itself can't be an seperate program
         because when the system call, fork(), was created it caused the only working cd program to 
         no longer function properly.

  2. Explain how our sample shell implements the change directory command.
         This shell checks and compares the  two arguments/strings to see if they are the same/different.
         One of the if statements compares the first string/argument with the string 'cd'.
         If successful it will change directories.


  3. What would happen if this program did not use the fork function, but
     just used execv directly?  (Try it!)

     Try temporarily changing the code 'pid_from_fork = fork();'
     to 'pid_from_fork = 0;'
         It looks like if fork wasn't used it would swap the current program in the process
         with a new program.
  4. Explain what the return value of fork() means and how this program
     uses it.
         the system call, fork(), returns 3 values, a negative value, positive value, or 0.
         If it's a negative value then creating the child process failed. if it's a positive
         value then a child process was created and returned the PID of the new child process
         to the parent process.
         If it returns a 0 then it is instead returned to the child process.

  5. What would happen if fork() were called prior to chdir(), and chdir()
     invoked within the forked child process?  (Try it!)

     Try temporarily changing the code for 'cd' to use fork:

     if (fork() == 0) {
         if (chdir(exec_argv[1]))
             /* Error: change directory failed */
             fprintf(stderr, "cd: failed to chdir %s\n", exec_argv[1]);
         exit(EXIT_SUCCESS);
     }
         If called prior to chdir() then a new child process will be created and then the
         new child process will call chdir(), if it fails then it will provide the error,
         if it is successful then only the child process will change the directory.

  6. Can you run multiple versions of ./b.sh in the background?
     What happens to their output?
         You can run multiple versions and the output should display as it finishes.

  7. Can you execute a second instance of our shell from within our shell
     program (use './shell')?  Which shell receives your input?
         You can create a shell within a shell(shellception). the child shell will receive
         the input and any changes and modifications does in the second instance of our 
         shell will not effect anything in the parent shell.

  8. What happens if you type CTRL-C while the countdown script ./b.sh is
     running?  What if ./b.sh is running in the background?
         CTRL-C will stop the process that is currently running. if ./b.sh is running then 
         a new process should continue after the prior process has stopped.

  9. Can a shell kill itself?  Can a shell within a shell kill the parent
     shell?

     ./shell
     ./shell
     /bin/kill -s KILL NNN      (Where NNN is the the parent's PID.)

         Yes the child shell can kill the parent shell if parent PID is provided as the PID
         is unique to each shell.

  10. What happens to background processes when you exit from the shell?
      Do they continue to run?  Can you see them with the 'ps' command?

      ./shell
      ./b.sh&
      exit
      ps

      Background processes should still continue even after exiting the shell. Using any
      version of the ps command will show the existing background processes.


STEP 5:  Modify the MP
======================

  Please make the following modifications to the given file shell.c.  As in
  SMP0, we have included some built-in test cases, which are described along
  with the feature requests below.

  In addition to running the tests as listed individually, you can run
  "make test" to attempt all tests on your modified code.


  1. Modify this MP so that you can use 'ls' instead of '/bin/ls'
     (i.e. the shell searches the path for the command to execute.)

     Test: ./shell -test path

  2. Modify this MP so that the command prompt includes a counter that
     increments for each command executed (starting with 1).  Your
     program should use the following prompt format:
       "Shell(pid=%1)%2> "  %1=process pid %2=counter
     (You will need to change this into a correct printf format)
     Do not increment the counter if no command is supplied to execute.

     Test: ./shell -test counter

  3. Modify this MP so that '!NN' re-executes the n'th command entered.
     You can assume that NN will only be tested with values 1 through 9,
     no more than 9 values will be entered.

     Shell(...)1> ls
     Shell(...)2> !1     # re-executes ls
     Shell(...)3> !2     # re-executes ls
     Shell(...)4> !4     # prints "Not valid" to stderr

     Test: ./shell -test rerun

  4. Modify the MP so that it uses waitpid instead of wait.

  5. Create a new builtin command 'sub' that forks the program to create
     a new subshell.  The parent shell should run the imtheparent()
     function just as if we were running an external command (like 'ls').

     ./shell
     Shell(.n1..)1> sub
     Shell(.n2..)1> exit  # Exits sub shell
     Shell(.n1..)1> exit  # Exits back to 'real' shell

  6. Create a new global variable to prevent a subshell from invoking
     a subshell invoking a subshell (i.e., more than 3 levels deep):

     ./shell
     Shell(.n1..)1> sub
     Shell(.n2..)1> sub
     Shell(.n3..)1> sub   # prints "Too deep!" to stderr

     Test: ./shell -test sub
