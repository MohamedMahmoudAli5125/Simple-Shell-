# SimpleShell

A Unix shell implementation in C with multi-processing support, built as part of an OS systems programming course. Supports foreground/background process execution, signal handling, built-in commands, and environment variable management.

---

## Features

- **Foreground execution** — runs commands and waits for them to complete
- **Background execution** — run commands with `&` to keep the shell responsive
- **Built-in commands** — `cd`, `echo`, `export`, `exit`
- **Variable expansion** — define and use variables with `export` and `$VAR` syntax
- **SIGCHLD handling** — zombie processes are reaped automatically via `waitpid(-1, NULL, WNOHANG)`
- **Process logging** — every child termination is recorded to `shell_log.txt`
- **Hash map storage** — environment variables stored using [uthash](https://troydhanson.github.io/uthash/)

---

## Build
```bash
gcc -o simpleshell shell.c
```

---

## Run
```bash
./simpleshell
```

The shell starts silently (no prompt). Type commands and press Enter.

---

## Supported Commands

### External Commands
Any executable on your system's PATH:
```
ls
ls -l
mkdir test
firefox
```

### Background Execution
Append `&` to run a command without blocking the shell:
```
firefox &
```
The shell prints the background PID and continues accepting input immediately.

### Built-in: `cd`
```bash
cd              # go to $HOME
cd ~            # go to $HOME
cd ..           # go up one directory
cd /some/path   # absolute path
cd relative/dir # relative path
```

### Built-in: `echo`
Prints text with variable substitution (input must be in double quotes):
```bash
echo "Hello world"
export name=Alice
echo "Hello $name"    # → Hello Alice
```

### Built-in: `export`
Set environment variables for use within the shell:
```bash
export x=-l
ls $x              # runs: ls -l

export flags="-a -l -h"
ls $flags          # runs: ls -a -l -h

export greeting="Hello world"
echo "$greeting"   # → Hello world
```

### Exit
```bash
exit
```
Terminates the shell and frees all allocated memory.

---

## How It Works

1. `main()` calls `parent_main()`, which registers a `SIGCHLD` handler, calls `setup_environment()`, then starts the shell loop.
2. `setup_environment()` saves the launch directory for the log file, then `chdir`s to `/`.
3. On each iteration, the shell reads a line via `fgets()`, strips the newline, and calls `parse_input()` to tokenize it with `strtok()`.
4. `evaluate_expression()` checks if the command is a built-in (`cd`, `echo`, `export`) or an external executable.
5. External commands are run via `fork()` + `execvp()`. Foreground commands block with `waitpid()`. Background commands (`&`) return immediately and increment `bg_children`.
6. When any child exits, `SIGCHLD` fires `on_child_exit()`, which calls `reap_child_zombie()` to collect all finished children and appends a line to the log file.

---

## Logging

When the shell launches, it saves its working directory and creates a log file at:
```
<launch_directory>/shell_log.txt
```

Every time a background child process terminates, the following line is appended:
```
Child process was terminated
```

---

## Example Session
```bash
ls
mkdir test
ls
ls -a -l -h
export x="-a -l -h"
ls $x
firefox           # shell blocks until firefox closes
firefox &         # shell stays responsive; firefox runs in background
heyy              # prints: Command execution failed: No such file or directory
exit
```

---

## Dependencies

- **C standard library** — `stdio.h`, `stdlib.h`, `string.h`, `unistd.h`
- **POSIX** — `sys/wait.h`, `signal.h`
- **[uthash](https://troydhanson.github.io/uthash/)** — header-only hash map, bundled directly in `shell.c`

---

## Limitations

- No pipe (`|`) or I/O redirection (`>`, `<`) support
- No tab completion or command history
- Assumes no spaces in file paths for `cd`
- `echo` input must be enclosed in double quotes for variable expansion to work

---

## License

uthash is used under its BSD-style license (see source header). All other code is for academic use.
<img src="https://t.bkit.co/w_6a0a395425dc7.gif" />
