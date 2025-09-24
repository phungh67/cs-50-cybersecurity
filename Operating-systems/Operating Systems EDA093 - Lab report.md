# Operating systems ED093 - Lab 1: implement a simple shell

## Group 51:

  

# 1. Lab's requirements

- The shell meets all the features required by the TAs. It passed all the automation test provided in the lab's repository and the manual test by group's members.

- This shell can handle the EOF signal (caused by pressing Crtl + D)

```C++

  for (;;) {

    char *line;

    line = readline("> ");

  

    // Check for EOF (Ctrl+D)

    if (line == NULL) {

      printf("\n");

  

      break;

    }

```

- With the basic commands like `ls`, `date` and `who`, this shell invokes these commands by looking in the PATH with the `execvp(p->pgmlist[0], p->pgmlist)`. In this line of code, the `execvp` will execute the program provided by `p->pgmlist[0]`, followed by arguments `p->pgmlist`. As the manual of `exec` family said, the `execvp` will recognize the `PATH`.

- Background execution is also supported in this shell. If a command passed with `cmd.background` option, this shell will create a new isolated process to run it. This method protects the background processes from `Ctrl + C` signal the shell received. We also used a tracking method to ensure every background processes (if they are still running) will be terminated on exit.

```C++

else if (cmd.background) {

            // The process is running in the background

            LOG("Process running in background with PID %d\n", pid);

  

            // Create new process group to isolate from terminal signals

            // This prevents Ctrl+C from affecting background processes

            setpgid(pid, 0);

  

            // Register the process as a background process

            add_background_process(pid)

}

```

- The piping is the most challenge part on this lab. But we made it through some experiment. This shell fully supports pipe, a typicall "pipe" command likes `ls | grep out | wc -w` has the same output as executed with `Bash` or `sh` or `zsh`. One of the most crucial discovery is "It is necessary to close the pipe on the shell side too". The child processes created from the shell need pipe for data flow, that is correct, but in order to these process to understand "that's all the data" and to prompt the appropriate output, the shell must close the pipe too.

```C++

    // init the pipe creation at the start of child handler

    int pipefd[2];

    ...

    // duplicate the pipe for redirecting output if needed

    if (fd_out != STDOUT_FILENO) {

        dup2(fd_out, STDOUT_FILENO);

        close(fd_out);

    }

    ...

    // handle the pipe-communication between processes

    if (p->next != NULL) {

        close(pipefd[1]); // Close unused write end

        dup2(pipefd[0], STDIN_FILENO);

        close(pipefd[0]);

    }

    // in the parent side

         // Close the write end of the previous pipe if it exists

      if (fd_out != STDOUT_FILENO) {

        close(fd_out);

    }

  

    // If there is a next command, set up fd_out for the next iteration

      if (p->next != NULL) {

        close(pipefd[0]); // Close unused read end

        fd_out = pipefd[1];  // Save write end for next command

    }

```

- With the successfully implmentation of the shell, the redirection for input and output is quite simple. These two are only applied if the command that the program is processing is the last or the first command in the "pipe". The first command should accept input from `STDIN` or the redirection one, and the last command should write the output to the destination provided by user instead of `STDOUT`.

```C++

    // Handle input redirection from file (only for the first command)

    if (cmd->rstdin && p->next == NULL) {

        int fd = open(cmd->rstdin, O_RDONLY);

        if (fd == -1) {

          perror("open input file");

          exit(1);

        }

        dup2(fd, STDIN_FILENO);

        close(fd);

    }

  

    // Handle output redirection to file (only for the last command)

    if (cmd->rstdout && cmd_count == 0) {

        int fd = open(cmd->rstdout, O_WRONLY | O_CREAT | O_TRUNC, 0644);

        if (fd == -1) {

          perror("open output file");

          exit(1);

        }

        dup2(fd, STDOUT_FILENO);

        close(fd);

    }

```

- As required, this shell has a couple of built-in commands: `cd` for change directory, `exit` to quit the shell. There are two extra commands are implemented: `jobs` to list all the background jobs (and check if these jobs are terminated appropriately in the job record table), `fg` to bring a background job to the foreground. For better structure, these built-in commands are handled separately on `builtin.c` and included as `builtin.h`. These commands are executed directly within the shell and do not generate a child process.

```C++

// sample of cd built-in command

int handle_cd(Command *cmd) {

  if (cmd->pgm->pgmlist[1] != NULL) {

    if (chdir(cmd->pgm->pgmlist[1]) == -1) {

      perror("chdir");

      return -1;

    }

  } else {

    fprintf(stderr, "cd: missing argument\n");

    return -1;

  }

  return 0;

}

```

- For the `Ctrl + C` signal, it only causes the foreground process to be terminated, not the background ones. This is done by a handler and a global variable for foreground process. In the case if shell does not have any running foreground, this signal only generates a new line.

```C++

void sigint_handler(int sig) {

  (void)sig; // Unused parameter, this paramenter can be discarded, but when compling, the compiler maybe complain about it

  

  if (!has_foreground_process) {

    // There is no foreground process, just print a new prompt

    printf("\n");

    rl_on_new_line();       // Inform readline that we have moved to a new line

    rl_replace_line("", 0); // Clear the current input line

    rl_redisplay();         // Redisplay the prompt

  }

  // If there is a foreground process, do nothing here and let the signal be

  // delivered to the foreground process (default behavior)

}

...

// the process handler in the main loop

else {

            // The process is running in the foreground

            LOG("Process running in foreground with PID %d\n", pid);

            has_foreground_process = true;

  

            // Parent process waits for the child to finish if not background

            waitpid(pid, NULL, 0);

}

```

- With the clean up after each call (and parent always waits for children to send signal, to deterime whether they are successfully executed, encountered errors,...) this shell leaves no zombies nor orphans. It was tested with `top` command as well as additional tool provided by the TA, the `rptree`. Simon also wrote an extra layer of debbuging by actually looking inside the `\proc` directory of the Operating System, tracking them with the actual `pid` number as in the function named `list_background_processes()` in `job.c` file.

  

# 2. The process of making this lab.

At first, we met after the "Lab submission" and decided to make an agreement on how we will work in the future. With this lab, we decided to code independently till the last week comes. We often share the process through a communication chanel. Each member tried to make the shell work alone, then when the deadline is near, a shared GitHub repository will be created and we will combine each one's best in to a working solution.

  

For the order of this lab, each member has a distiguish way to achive.

  

With Simon, since he already has experience in coding with `Rust` and actually has some real-work, he chose to do everything at once. The first version of this shell which he shared with team was not as clean as this one. Every functions were implemented right in the `lsh.c` file and it took me (the one who is bad at coding) a while to actually understand his idea.

  

As Nick has another way to complete this lab, he said he took the top-down approaching.

  

For me, not very good at coding, I tried to complete these items one by one, the easier first and the hard later. With each items solved, I noted in my private journal-repo to read again if needed. The most challenging thing is the pipe. It is quite complex theory and took me 2 days to crawl for information, demo, sample code and even books to at least know how it works.

  

Then, when all group members had tried to code and achieved somethings, we met, discussed and exchanged idea to solve things that was not resolved. Simon is the member with the furthest progress, he shared his method and guided the rest with "pipe" section - the most troublesome feature of this shell.

  

We also decied that we should take at least one and a half day from that meeting to thoroughly implement and at least, program something individually. After that, a draft of report will be share among group members, tweak if needed and submit it along the best "code".

  

# 3. Challenges

The most challenge thing is "Pipe" section. This can considered to be the most difficult for this lab. With redirection or background process and the signal handling, it quite easy to understand the concepts, hence, the implementing can be achieved shortly. But not with the pipeling, many version of coding can accept the pipe command, allows new process to be forked from parent with the "[Pipe] log" in the tool `rptree`.

  

As Simon, he also encountered the "Pipe" problem and it took him 1-2 days to completely solve the problem. In fact, he is the one with the best "Pipe" code in this group.

  

For Nick, because most of his experience with C if from embedded system, not for the Operating system, his final code before group meeting only passed 3/5 total (as he said).

  

For me, the most challenging is not the "Pipe" although it took me several days to understand. Since I misunderstod the requirements, I tried to implement my own `ls`, `cd`, `who` and `wc` command, even with arguments stuff. I even instructed my program to look in a `user-define` PATH instead of `PATH` of the OS, so it took me a lot of working, researching and of course, time.

  

# 4. Feedback

We found that the automation test is really helpful. These test cases are a well defined set and can catch almost all the common errors when coding this shell. But we are agree that, the test maybe missing: What if a command in the pipe is missing from this shell?

  

We tried to pasre a simple pipe: `ls | nonexistentcommand | wc -w`, the shell (should) prints "Command not found" and with the last result, wc must prompt 0. Scanning the test file, this case is not in there. We think it is good to include this case because, there is a chance that in a pipeline, a command will go wrong and the shell must be able to handle it.