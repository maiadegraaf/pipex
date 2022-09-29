# pipex

This project is somewhat of a precurser to [minishell](https://github.com/maiadegraaf), a project where we recreate bash.  Although, in pipex, the goal is just to replicate the working of pipes (`|`), and further serve as an introduction to using the `pipe()`, `fork()`, `dup()` and `execve()` functions.  It was also the first time I encountered multiple processes.

## How Real Pipes Work in Bash

The program needs to specifically replicate the following command:
```sh
$> < file1 cmd1 | cmd2 > file2
```

Where:<br>
`<`&emsp;&emsp;&emsp;donates the following word replaces `std:in`<br>
`file1`&emsp;represents the file to read from.<br>
`cmd1`&emsp;&ensp;is the command that will be used on the input from `file1`<br>
`|`&emsp;&emsp;&emsp;sends the output from `cmd1` to `cmd2`<br>
`cmd2`&emsp;&ensp;is the command that will be used on the output from `cmd1`<br>
`>`&emsp;&emsp;&emsp;donates the following word replaces `std:out`<br>

## Input
```sh
./pipex infile cmd1 cmd2 outfile
```
