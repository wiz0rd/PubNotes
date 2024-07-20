# Linux Background Processes: nohup, bg, and I/O Redirection

## Table of Contents
1. [[#Introduction]]
2. [[#nohup Command]]
3. [[#bg Command]]
4. [[#I/O Redirection (2>&1 &)]]
5. [[#Practical Examples]]
6. [[#Best Practices]]
7. [[#Troubleshooting]]
8. [[#Further Reading]]

## Introduction

In Linux, running processes in the background is a powerful feature that allows users to execute long-running tasks without tying up their terminal. This guide covers the usage of `nohup`, `bg`, and I/O redirection to effectively manage background processes.

## nohup Command

`nohup` stands for "no hang up". It's used to run a command immune to hang ups, with output to a non-tty.

### Syntax
```bash
nohup command [arguments] [> output_file] [2>&1] &
```

### Key Points
- Prevents the command from being terminated when the terminal session ends.
- By default, redirects output to `nohup.out` in the current directory.
- Commonly used with `&` to immediately background the process.

### Example
```bash
nohup long_running_script.sh > output.log 2>&1 &
```

[Picture: nohup command execution and output]

## bg Command

`bg` is used to send suspended jobs to the background.

### Syntax
```bash
bg [job_spec]
```

### Key Points
- Resumes a suspended job and runs it in the background.
- If no `job_spec` is given, the current job is used.
- Jobs can be suspended with Ctrl+Z and listed with the `jobs` command.

### Example
```bash
./long_script.sh
# Press Ctrl+Z to suspend
bg
```

[Picture: bg command usage and job control]

## I/O Redirection (2>&1 &)

This syntax is used for redirecting both stdout and stderr to a file and running the process in the background.

### Breakdown
- `2>&1`: Redirects stderr (2) to stdout (1)
- `&` at the end: Runs the command in the background

### Key Points
- `>` redirects stdout
- `2>` redirects stderr
- `&1` refers to stdout

### Example
```bash
command > output.log 2>&1 &
```

[Picture: I/O redirection diagram]

## Practical Examples

1. Running a web scraper in the background:
   ```bash
   nohup python3 web_scraper.py > scraper_log.txt 2>&1 &
   ```

2. Compiling a large project:
   ```bash
   nohup make -j4 > compile_log.txt 2>&1 &
   ```

3. Long-running data analysis:
   ```bash
   nohup Rscript analysis.R > analysis_output.log 2>&1 &
   ```

## Best Practices

1. Always redirect output to a log file for later inspection.
2. Use meaningful names for output files.
3. Check the status of background jobs with `jobs` command.
4. Use `disown` to detach a job from the current shell session.

## Troubleshooting

1. Process died after logout:
   - Ensure `nohup` was used correctly.
   - Check if the process requires terminal input.

2. Cannot find output:
   - Check the current directory for `nohup.out`.
   - Verify the path used for custom output files.

3. Process hogging resources:
   - Use `nice` command to adjust process priority.
   - Monitor with `top` or `htop`.

## Further Reading

- [[Linux Process Management]]
- [[Bash Job Control]]
- [[Advanced I/O Redirection]]

---

Links to official documentation:
- [nohup man page](https://linux.die.net/man/1/nohup)
- [bg man page](https://linux.die.net/man/1/bg)
- [Bash Manual: Redirections](https://www.gnu.org/software/bash/manual/html_node/Redirections.html)
