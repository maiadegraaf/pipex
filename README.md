# pipex

This project is somewhat of a precurser to [minishell](https://github.com/maiadegraaf), a project where we recreate bash.  Although, in pipex, the goal is just to replicate the working of pipes (`|`), it further serves as an introduction to using the `pipe()`, `fork()`, `dup()` and `execve()` functions.  It was also the first time I encountered multiple processes.

## Table of Contents
- [How Real Pipes Work in Bash](#how-real-pipes-work-in-bash)
- [Input](#input)
- [An Overview of Useful Functions](#an-overview-of-useful-functions)
- [Order of Operations](#order-of-operations)
	- [Handle the Input](#handle-the-input)
	- [Creating Pipes and Forks](#creating-pipes-and-forks)
	- [Child Processes](#child-processes)
	- [Wait.....](#wait)
- [Installation](#installation)
	- [Clone the repository](#clone-the-repository)
	- [Expected Input](#expected-input)
	- [Some commands to try](#some-commands-to-try)

## How Real Pipes Work in Bash

The program needs to specifically replicate the following command:
```sh
$> < infile cmd1 | cmd2 > outfile
```

|    Key     |   Meaning                                                                       |
| :------------: | ------------------------------------------------------------------------------ |
|   `< infile`   | Changes STDIN to infile                                                        |
|  `> outfile`   | Changes STDOUT to outfile                                                      |
| `cmd[1 or 2] ` | A bash command                                                                 |
| `        \| `  | Connects the STDOUT from the first process, to the STDIN of the second process |

Bash reads from the `infile`, then executes `cmd1` using the `infile` as input, the output of which is sent to `cmd2` which then writes to the `outile` after executing.  It is specifically this function that pipex replicates. 

## Input

The input pipex takes is slightly different than bash.  As shown below.
```sh
./pipex infile cmd1 cmd2 outfile
```

This means we don't have to parse any tokens (`<`, `>`, or `|`), and instead focus on the 4 arguments.

## An Overview of Useful Functions

| Function | Explanation |
| :---:|---|
|`int pipe(int end[2])`| Takes an array of two ints and links them together, assigning an *fd* to each end. This allows for communication between them. |
| `pid_t fork(void)` | Splits the process into two child processes.  It returns 0 for the child process, a the process ID of the child process to the parent process, and -1 incase of an error. |
| `int dup2(int fd1, int fd2)`| Closes fd2 and duplicates the value of fd2 to fd1|
| `int execve(const char *path, char *const argv[], char *const envp[]);` | Executes the command passed as a 2D array |


## Order of Operations
#### Handle the Input
The first thing the program does is make sure the infile already exists and is a readable file, if it doesn't, it throws an error.  It also makes the outfile, with read/write permissions.  Both of these actions are done using `open()`, and their file descriptor's are stored in a main struct, as well as `envp` and the two commands, which are split into a 2D array. This program also uses the `envp` in order to access the `PATHS` to the various bash commands stored in the computer.  The `PATHS` are stored in the main struct through a special parser that extracts them from `envp`.

#### Creating Pipes and Forks
The main struct is passed to a function `pipex()` which creates a pipe, using `pipe()`, and two child processes using `fork()`, aptly named child 1 and 2.

#### Child Processes
The first child uses `dup2()` to replace STDIN with the infile file descriptor, and the STDOUT with one end of the pipe. In order to recieve the output from the first child, the second child replaces STDOUT with the outfile file descriptor, and the STDIN with the other end of the pipe, also using `dup2`.  Both processes then loop through all the available paths until they have found the correct one to the command's binary location, which is then past to `execve()`.  As `execve()` transforms the calling process into a new process, once it has executed the command it deletes all ongoing processes, thereby ending the child process.  If the command doesn't exist the child throws an error and exists.

#### Wait.....
While the child processes are ongoing, using `waitpid()`, `pipex()` waits for the processes to end. Once they are done, the program ends.

## Installation
### Clone the repository
``` 
git clone https://github.com/maiadegraaf/pipex.git
cd pipex
make
```

### Expected Input
```sh
$> < infile cmd1 | cmd2 > outfile
```

You have to create an infile for the program to read from.  The program creates its own outfile.

### Some commands to try
These also create infiles to read from, and displays the outfile.
<br>
<br>
```sh
ls -la > infile
./pipex infile 'grep pipe' 'tr a-z A-Z' outfile
cat outfile
```
*Greps all lines with 'pipe' and capitalizes them.*
<br>
<br>
```sh
cat README.md > infile
./pipex infile "sed s/pipex/some_other_name/g" "wc -l" outfile
cat outfile
```
*Changes all occurrences of 'pipex' with 'some_other_name', then outputs the amount of changed lines to the outfile.*
<br>
<br>
```sh
echo $PATH > infile
./pipex infile "grep -o :" "wc -w" outfile
cat outfile
```
*Counts the occurrences of `:` in PATH, an environment variable.*
