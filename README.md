# Sprint Challenge: Processes and Scheduling

## Multiple Choice and Short Answer Questions

Add your answers inline, below, with your pull request.

1. List all of the main states a process may be in at any point in time on a
   standard Unix system. Briefly explain what each of these states mean.

   * Created: Temporary state when a process is created. 
   * Ready: Process is waiting to be assigned to a CPU core/processor by the OS.
   * Running: Once the process has been assigned to a processor by the OS scheduler, the process state is set to running and the processor executes its instructions.
   * Waiting: Process moves into the waiting state if it needs to wait for a resource, such as waiting for user input, or waiting for a file to become available.
   * Terminated: Once the process finishes its execution, or it is terminated by the operating system, it is moved to the terminated state where it waits to be removed from main memory.
   

2. What is a Zombie Process? How does it get created? How does it get destroyed?
A Zombie Process is a process that has executed completely, but still has an entry in the 
process table. This happens when a parent process does not use the wait system call after a 
child dies to read its exit status. When a child process terminates before the parent has 
called wait, the kernel retains some information about the process, such as its exit status, 
to enable its parent to use the wait system call later. Because the child is still consuming 
system resources, but not executing, it is known as a zombie process. 

My question: At this point, I believe I understand that the parent reaps the status of the child and then it is removed from the process table, but what does it really mean to reap the status of the child since it is terminating? Does this just mean that the parent takes on the child process ID as is the case when a child process is dying?


3. Describe the job of the Scheduler in the OS in general.
The purpose of the Scheduler is to decide when to run any given process at any time.

4. Describe the benefits of the MLFQ over a plain Round-Robin scheduler.
The benefit of using MLFQ over a RR scheduler is that it assigns priority to processes based
on the behaviour of the observed process. So, it is able to evaluate what process is an
urgent request and allot the appropriate resources to ensure it is being served first. 
With a RR scheduler, urgent requests do not have priority as all requests are seen as equal 
and thus distributed all at once. With a RR scheduler, each process is given the same short 
period of time to make progress on their execution and when that time is up, the next process begins its work toward its execution. If they don't complete in that time section, it stops and waits until the RR comes back around to them and it's given the same amount of time to make progress on its execution. This continues until all processes are complete. This can be burdensome for performance.  

## Programming Exercise: The Lambda School Shell (`lssh`)

This program implements a new shell that you can use to run commands from in
Unix, similar to bash!

At the end of the day, you should be able to run your shell, the run commands within it like so:

```
[bash]$ ./lssh 
lambda-shell$ ls -l
total 32
-rwxr-xr-x  1 beej  staff  9108 Mar 15 13:28 lssh
-rw-r--r--  1 beej  staff  2772 Mar 15 13:27 lssh.c
lambda-shell$ head lssh.c
#include <stdio.h>
#include <stdlib.h>
#include <unistd.h>
#include <string.h>

#define PROMPT "lambda-shell$ "

#define MAX_TOKENS 100
#define COMMANDLINE_BUFSIZE 1024
#define DEBUG 0  // Set to 1 to turn on some debugging output
lambda-shell$ exit
[bash]$ 
```

General attack is to:

* Loop until the user exits.
  * Print a prompt.
  * Read a line of input from the keyboard.
  * Parse that line down into individual space-separated parts of the command.
  * Fork a child process to run the new command.
  * Exec the command in the child process.
  * Parent process waits for child to complete.

Some of the shell is already written, but you have to write the guts of it.

Examine the `lssh.c` file and:

* Determine which parts of the program do what.
* Read the header comments.
* Find the parts you need to implement.
* Come up with an individual plan of attack for each of those parts.
* Determine how to exit the shell.
* What happens if you build it?
* What happens if you run it?

Resist the urge to start coding until you've done all that and read this
complete challenge.

Hint: you probably want one of the `exec` variants that has the `p` suffix on it
so that the `PATH` will be searched for the file you're trying to run. Choose
your `exec` variant carefully. Some will be hard-to-impossible to use. Some will
be easy.

When you get this working, _you will have your own shell_. It's not as
full-featured as bash, by any means, but it is a legitimate shell and the core
is the same as any other shell.

If you finish early, look at extra credit to start implementing more features.

### Extra Credit: Change Directories with `cd`

In bash, you can change directories with the built-in `cd` command.

Each process keep track of which directory it is running in, and the shell is no
exception. Each process can change its current directory, as well.

> Why is `cd` built into the shell? Why can't it run as an external command?

Because it is built in, you should check to see if the user entered `cd` in
`args[0]` _before_ running the command. And if they did, you should
short-circuit the rest of the main loop with a `continue` statement.

> Look at the implementation of the built-in `exit` command for inspiration.

You can use the program `pwd` to see what directory you are in.

Example run:

```
lambda-shell$ pwd
/Users/example
lambda-shell$ cd src
lambda-shell$ pwd
/Users/example/src
lambda-shell$ cd ..
lambda-shell$ pwd
/Users/example
lambda-shell$ cd foobar
chdir: No such file or directory
lambda-shell$ 
```

If the user entered `cd` as the first argument:

1. Check to make sure they've entered 2 total arguments
2. Run the system call `chdir()` on the second argument to change directories
3. Error check the result of `chdir()`. If it returns `-1`, meaning an error
   occurred, you can print out an error message with:
   ```
   perror("chdir"); // #include <errno.h> to use this
   ```
4. Execute a `continue` statement to short-circuit the rest of the main loop.

Note that `.` and `..` are actual directories. You don't need to write any
special case code to handle them.

### Extra Credit: Background Tasks

In bash, you can run a program in the background by adding an `&` after the
command.

```
ls -la &
```

Add similar functionality to `lssh` by checking to see if the last argument is
an `&`.

If it is:

1. Strip the `&` off the `args` (by setting that pointer to `NULL`).
2. Run the command in the child as usual.
3. Prevent the parent from `wait()`ing for the child to complete. Just give a
   new prompt immediately. The child will continue to run in the background.

In every instance of the main loop after the user hits `RETURN`, you should wait
in a loop to reap any background zombies that have died in the meantime. You can
use a loop do this with without blocking like so:

```c
// Wait for any other processes that have ended in the
// meantime. A more correct solution would be to listen for the
// SIGCHLD signal and wait for zombies at that point.
while (waitpid(-1, NULL, WNOHANG) > 0)
    ;
```

The above (empty) loop will just call `waitpid()` repeatedly until there are no
more zombies to reap. Then the program continues on.

What happens if you don't do this?

Mini stretch challenge: wait for the `SIGCHLD` signal and put the above `while`
loop in there instead.

Note that you might get weird output when doing this, like the prompt might
appear before the program completes, or not at all if the program's output
overwrites it. If it looks like it hangs at the end, just hit `RETURN` to get
another prompt back.


### Extra Credit: File Redirection

In bash, you can redirect the output of a program into a file with `>`. This
creates a new file and puts the output of the command in there instead of
writing it to the screen.

```
ls -l > foo.txt
```

Check the `args` array for a `>`. If it's there:

1. Get the output file name from the next element in `args`.
2. Strip everything out of `args` from the `>` on. (Set the `args` element with
   the `>` in it to `NULL`).
3. In the child process:
    1. `open()` the file for output. Store the resultant file descriptor in a
       variable `fd`.
    2. Use the `dup2()` system call to make `stdout` (file descriptor `1`) refer
       to the newly-opened file instead:

	    ```c
		int fd = open(...
		dup2(fd, 1);  // Now stdout goes to the file instead
		```
	3. `exec` the program like normal.

Note that depending on your [`umask`](https://en.wikipedia.org/wiki/Umask), the
newly-created file might not be actually readable by you. If you can't read it,
run `chmod 600 filename` on the new file and then you'll have permission.

### Extra Credit: Pipes

In bash, you can pipe the output of one command into the input of another with
the `|` symbol.

For example, this takes the output of `ls -l` and uses it as the input for `wc
-l` (which counts the number of lines in the input):

```
ls -l | wc -l
```

This uses the `pipe()` system call.

Use a process similar to the above extra credit challenges, along with [this
description of how to implement pipes in
C](https://github.com/LambdaSchool/CS-Wiki/wiki/How-Unix-Pipes-are-Implemented)
to get pipes implemented in `lssh`.