# Linux Shell Basics

*Linux* normally refers to a operating system kernel. *Shell* is a **command interpreter** that user can use to interact with operating system. It also maintain the environment variables and alias. 

There are several shells such as `bash`, `csh`,`sh`, while `bash` is the default interactive shell for most Linux distributions. `Terminal` or `Console` are programs that launch a shell for user. 

## Terminal

In terminal, use `CTRL+INS` and `SHIFT+INS` for copy and paste. 

Use `SHIFT+CTRL+C` and `SHIFT+CTRL+V` for copy and paste. 

## Virtual Consoles

When system boots in multi-user mode (runlevel 2,3,5), six virtual console `tty1` to `tty6` are created with text-based logins. If X system is running, it usually runs in `tty7`. 

When in X, use `CTRL+ALT+F1`, ..., `CTRL+ALT+F6` to switch `tty`. When in text-based terminal, use `ALT+F1`, ..., `ALT+F7` to switch.

Each `tty` allows you to log in with different accounts. Processes running on different `tty` will not be affected after switch. 

## Shell Configuration

*Login shell* is the first shell after a successful login, while *non-login shell* is started by other programs. 

- To check if current shell is a login shell, use `# echo $0`. If the output shell name **prepended by a dash**, it is a login shell. 
- Login shell will read system-wide startup file in `/etc/profile` and user-specific startup file in `~/.bash_profile` and `~/.profile`.
- Both shell will read system-wide initialization file in `/etc/(bash.)?bashrc` and user specific initialization file `~/.bashrc`.
- The configuration files may execute scripts in `/etc/profile.d`. 
- When exiting a shell, it will read  `~/.bash_logout`. 
- Command history is stored in `~/.bash_history`. 

Check out some of useful `.bashrc` files [here](https://serverfault.com/questions/3743/what-useful-things-can-one-add-to-ones-bashrc) and [here](https://ubuntuforums.org/showthread.php?t=679762). 

### Alias

Use `alias` to list and set aliases.

```shell
$ alias
alias grep='grep --color=auto'
alias l='ls -CF'
alias la='ls -A'
$ alias ll='ls -lahF'
# Add the line to ~/.bashrc to be persistent
```

Use `unalias <alias>` to remove aliases for current shell session.

### History

Use `history` to list bash history. 

```shell
$ history # list all
$ history 5 # list last 5 commands
```

Use `UP ARROW` and `DOWN ARROW` to move among the history commands.

```shell
$ !! # Run the previous command
$ !997 # Run command number 997 from history
```

Use `CTRL+r` to reverse search for a string in history. 

```shell
$ <CTRL+r>
(reverse-i-search)`ss': sudo /usr/bin/less /var/log/messages
<CTRL+r> # repeatedly search backward
```

### Environment Variables

*Local variable* is available only to current shell, while *environment variable* is inherited by all programs run from current shell. **A local variable must be exported to become environment variable.** 

Use `set | less` to list all local, environment variables, and functions.

Use `env` to list environment variables.

Use `VAR=VALUE` to set local variables.

```shell
$ ABC=123
$ echo $ABC
123
```

Use `export` to set environment variable.

```shell
$ export ABC=123
# Concatenate to existing variable using : as delimiter. 
$ export PATH=$PATH:/home/chris/bin
```

- add `export PATH=$PATH:/dir` command as a new line in `$HOME/.bashrc` so that all shells will run this command after starting. 
- add `PATH=$PATH:/dir` as a new line in `/etc/environment` to make it globally seen to all processes. **Note `export` is removed. Restart is required.** 

## Redirection and Pipes

### Standard I/O

Default `stdin` is keyboard, default `stdout` and `stderr` is display. File descriptor for `stdin`, `stdout`, and `stderr` is `0,1,2` respectively. 

### Redirection

We can redirect `stdin`, `stdout` and `stderr` to a file or device. 

- ``<`` read from specified file instead of keyboard. It is only useful when the command does not take filename parameter, or the behavior is different when taking filename versus `stdin`. 
- `>` **writes** output to specified file instead of display. 
- `>>` **appends** output to specified file instead of display. 
- `2>` **writes error message** to specified file instead of display. Note 2 is the file descriptor for `stderr`. This is useful if you want to keep a log of automatic cron job. 
- `&>` writes both output and error message to same file. The full form is `ls /fake > myout 2>&1` where `&1` refers to current `stdout`. Note there is **no space between `2>` and `&1`**. And the order of `>` and `2>` matters here. 
- `/dev/null` is a special bit bucket file. Anything goes to it is discarded. 
- `cat < file1 > file2` passes the content of `file1` to `cat`, then save the output to `file2`. 

```shell
# use passwd file as input to mail
$ mail -s "hello" hacker < /etc/passwd 	
# append stdout to file
$ ls /tmp >> output.txt
# stdout to file, stderr to display
$ ls /tmp /tmmp > output.txt
# stdout to display, stderr to file
$ ls /tmp /tmmp 2> errors.txt
# stdout and stderr to different files
$ ls /tmp /tmmp 2> errors.txt > output.txt
# stdout and stderr to same file
$ ls /tmp /tmmp > everything.txt 2>&1
```

### Pipe, Backticks, and `xargs`

Pipe is used to redirect information between processes. 

- `|` redirects the output of first command as the input to the second command. 

- `tee` will duplicate the piped content and write to both the `stdout` and files. 

  ![tee](./res/tee.png)

Backticks is used to **execute the command in the backticks first**, then **replace the section with its output** to the rest of the command line, usually as argument. 

  ```shell
# run `which bash` first to find the full path to bash command
# then list the path
$ ls -l `which bash`
  ```

`xargs` is used to passing multiple outputs individually to a command. 

```shell
# executes ls /bin/b* to obtain a list of paths
# then passes one path at a time to command dpkg-query -S as its argument.
$ ls /bin/b* | xargs dpkg-query -S
```
Note **`xargs` will run as many times** as needed to process all entries of output of piped command. On the other hand, **backticks** push entire output to the command and **runs it only once**. If the output is too large using backticks may fail. 

## Auto Refresh

When output of command or a file is changing, it can be automatic refreshed.

Use `watch -d` and backticks to refresh the output of a command

```shell
$ watch -d `cat /proc/loadavg`
$ watch -d `ls -l mydownload.iso`
```

Use `tail` with `-f` follow option to refresh content of a file. 

```shell
$ tail -f /var/log/messages
```