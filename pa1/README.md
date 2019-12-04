## Project #1: Oh my pretty shell

### *** Due on 11:59:59pm, October 9 (Wednesday)***

> Mars will come to fear my botany powers.
> -- <cite>Mark Watney, The Martian, 2015.</cite>

### Goal

With the system calls learned in the class and a few additional ones, you are ready to manipulate processes in the system. Let the operating system come to fear your system-call powers by implementing your own command shell.


### Problem Specification
- The shell program `mpsh` waits for your command line input after printing out a prompt `$`. When you enter a line of command, it will be parsed into tokens as of project assignment 0, and the framework will call `run_command()` function with the tokens. You need to implement following features in that function.


#### Execute commands (50 pts)
- The primary role of shell is to execute the command specified in the command until user enters `exit`. Each command is comprised of one exeutable followed by zero or more arguments. For example;

- ```shell
  $ ./toy arg1 arg2 arg3 arg4
  pid  = xxxxxx
  argc = 5
  argv[0] = ./toy
  argv[1] = arg1
  argv[2] = arg2
  argv[3] = arg3
  argv[4] = arg4
  done!
  $ ./toy sleep 10
  pid  = xxxxxx
  argc = 3
  argv[0] = ./toy
  argv[1] = sleep
  argv[2] = 10
  ( ... toy will sleep for 10 seconds ... )
  $
  $ /bin/ls
  Makefile  pa1.c  parser.c  parser.o   toy    toy.o
  mpsh	  pa1.o  parser.h  README.md  toy.c  types.h
  $ ls
  Makefile  pa1.c  parser.c  parser.o   toy    toy.o
  mpsh	    pa1.o  parser.h  README.md  toy.c  types.h
  $ pwd
  /home/sanghoon/pa/os/pa1
  $ cp mpsh mpsh.copied
  $ ls
  Makefile  mpsh.copied  pa1.o	   parser.h  README.md  toy.c  types.h
  mpsh	    pa1.c        parser.c  parser.o  toy	      toy.o
  $ exit
  ```
  
- Note that the command can be either *built-in* or *external*; Built-in commands are the ones that are supported natively by the shell program itself. In other words, built-in commands run without any other executable. External commands are, on the other hand, the ones that are not the built-in commands; We need an executable in one of the directories specified in `$PATH` environment variable to process external commands. The  shell should be able to handle `$PATH` environment variable so that you don't need to specify the full path for each external command. This sounds scarly. But no worries, it is obvious if you know the p-variants of `exec()` command family.

- Use `toy` application for your development and testing. It will sleep for `@N` seconds if you run it with `sleep @N` as its arguments, as shown above.

- Hint: `fork(), exec(), wait(), waitpid(), getenv()`


#### Terminate timed-out program (100 pts)

- Let the shell detect non-finishing programs and terminiate them. Let `timeout` built-in command set the desired time out specified in seconds (2 by default). When the executing program does not return for the specified period, `mpsh` should terminate the program by sending a `SIGKILL` signal to it. When the timeout is set to 0, disable the timeout feature.

- ```shell
  $ timeout
  Current timeout is 0 second
  $ timeout 10
  Timeout is set to 10 seconds
  $ sleep 5
  ( ... after 5 seconds ... )
  $ ./toy sleep 20
  ( ... after 10 seconds ...)
  ./toy is timed out
  $ timeout 0
  Timeout is disabled
  $ sleep 100
  ( ... after 100 seconds ...)
  $
  ```

- Use `__timeout` variable to read the current timeout value. Use `set_timeout()` function to set the value. DO NOT MODIFY `__timeout` DIRECTLY.

- On time out, print out the name of the application followed by " is timed out\n" to `stderr` using `fprintf`. Use the exact format to pass testcase checking.

- ```C
  /* Assume that char *name points to the name of the application */
  fprintf(stderr, "%s is timed out\n", name);
  ```

- Hint: `sigaction(), alarm(), kill()`.

- Make sure `alarm` is disabled once the program exits otherwise the shell itself might be be terminated. Use `sigaction` over `signal` for code portability. Also, have a look at `man 7 signal` to get an extensible explanation about signal handling.


#### Change working directory (30 pts)

- Each process has so-called *working directory* on which the process is working. This value can be checked by executing `/bin/pwd` or calling `getcwd()` system call. Implement `cd` command which changes the current working directory.

- Each user has one's *home directory* which is denoted by `~`. The actual path is defined in `$HOME` environment variable. Make `cd` command to understand it

- ```shell
  $ pwd
  /home/directory/of/your/account  # Assume this is the home directory of the user
  $ cd /somewhere/i/dont/know
  $ pwd
  /somewhere/i/dont/know
  $ cd ~
  $ pwd
  /home/directory/of/your/account
  ```

- Hints: `chdir(), getcwd(), getenv("HOME")`


#### Repeat the command n times (50 pts)
- Implement `for <N> <command ...>` built-in command to run the commands N-times. `for` can be nested.

- ```shell
  $ for 5 pwd
  /home/pasubmit/some/directory/in/depth
  /home/pasubmit/some/directory/in/depth
  /home/pasubmit/some/directory/in/depth
  /home/pasubmit/some/directory/in/depth
  /home/pasubmit/some/directory/in/depth
  $ for 4 cd ..
  $ pwd
  /home/pasubmit
  $ for 2 for 3 echo hello world
  hello world
  hello world
  hello world
  hello world
  hello world
  hello world
  $ for 20 for 10 echo for 2 echo nested for
  nested for
  (... 398 more "nested for" ...)
  nested for
  $
  ```


#### Change prompt (20 pts)
- Change the prompt with `prompt <new prompt>` command as follow:

- ```shell
  $ prompt #
  # prompt sce213
  sce213 prompt $
  $
  ```

- Note that the framework makes the prompt using the contents in `char __prompt[]`. You may set the value directly unlike `__timeout`.


### Restriction

- DO NOT USE `system()` system call. You will get 0 pts if it is used.
- For sure, you should implement the features by calling relevent system calls; just printing out expected results will make your score -250 pts.
- TEST YOUR CODE ON YOUR COMPUTER FIRST. You might be be penalized if you blow up the pasubmit system multiple times with obvious mistakes. In this sense, it would be best to test your implementation on your machine using `make test-run`, `make test-timeout`, `make test-cd`, `make test-for`, and `make test-prompt`. `make test-all` runs all the tests one after the other. Have a look at `Makefile` to learn making test automatic.
- For your coding practice, the compiler is set to halt on some (important) warnings. Write your code to comply the C99 standard.


### Submission / Grading

- Use [PAsubmit](https://sslab.ajou.ac.kr/pasubmit) for submission
	- 280 pts in total

- Source: pa1.c (250 pts in total)
  - Points will be prorated by testcase results.

- Git repository URL at git.ajou.ac.kr (10 pts)
  - Refer to https://www.youtube.com/channel/UC-Un-7jmeP-1OaHkS7awO0Q for using gitlab at Ajou University.
  - Will be announced shortly

- Document: *One PDF* document (20 pts). It should include followings;
	- Outline how programs are launched and how arguments are passed
  - How the time-out feature is implemented
  - How the `for` built-in command is implemented
  - Lesson learned (if you have any)
  - NO MORE THAN ***THREE*** PAGES
  - DO NOT INCLUDE COVER PAGE
    - No need to specify your name nor student ID on the document
  - DO NOT COMPRESS THE FILE
  - OTHERWISE YOU WILL GET 0 pts for the report

- WILL NOT ANSWER THE QUESTIONS ABOUT THOSE ALREADY SPECIFIED ON THE HANDOUT.
