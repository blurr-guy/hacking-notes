# Bash Scripting 
Beginner to Advance 
---

## Basic Commands 
- echo



## Basic file manipulation :
### `rm` — Remove Files and Directories

`rm` deletes files and directories. There is **no trash bin** — by default, deletion is permanent. Use it carefully, especially with wildcards or recursive flags.

#### Basic Syntax

```bash
rm [OPTIONS] FILE...
```

#### Basic Usage

```bash
rm file.txt              # delete a single file
rm file1.txt file2.txt   # delete multiple files
rm *.log                 # delete all .log files in current directory
```

If the file is write-protected, `rm` (without `-f`) will prompt for confirmation before deleting.

---

#### Flags

##### `-f`, `--force`
Force deletion without prompting, and ignore nonexistent files (no error if the file isn't there).

```bash
rm -f file.txt
```
Use with caution — this silently succeeds even if nothing was deleted, which can hide mistakes in scripts.

---

##### `-i`
Prompt before every removal.

```bash
rm -i file.txt
### rm: remove regular file 'file.txt'? y
```
Good habit when deleting things manually, especially with wildcards.

---

### `-I`
Prompt once before removing more than three files, or when removing recursively. Less intrusive than `-i` but still gives a safety check for bulk deletes.

```bash
rm -I *.txt
```

---

### `-r`, `-R`, `--recursive`
Remove directories and their contents recursively. Required to delete a non-empty directory.

```bash
rm -r some_folder/
```

---

### `-d`, `--dir`
Remove **empty** directories (without needing `-r`). Fails if the directory has contents.

```bash
rm -d empty_folder/
```

---

### `-v`, `--verbose`
Explain what is being done — prints each file as it's deleted.

```bash
rm -v file1.txt file2.txt
# removed 'file1.txt'
# removed 'file2.txt'
```

---

### `--interactive[=WHEN]`
More granular version of `-i`/`-I`. `WHEN` can be:
- `never` — no prompts
- `once` — same as `-I`
- `always` — same as `-i`

```bash
rm --interactive=once *.log
```

---

### `--one-file-system`
When removing recursively, skip any directory that is on a different filesystem than the one the command started on. Useful safety measure when deleting across mount points.

```bash
rm -r --one-file-system /mnt/data/
```

---

### `--no-preserve-root`
Allows `rm` to operate on `/` (root) if combined with `-r`. **Disabled by default** as a safety measure. Never use this unless you fully understand the consequences — it will wipe the filesystem.

```bash
# EXTREMELY DANGEROUS — do not run
rm -rf --no-preserve-root /
```

---

### `--preserve-root[=all]`
Default behavior — refuses to operate recursively on `/`. With `=all`, also refuses on any argument that resolves to a root-like path across all mounted filesystems.

---

### `--help`
Display help text and exit.

### `--version`
Display version information and exit.

---

## Common Flag Combinations

| Command | Meaning |
|---|---|
| `rm -rf folder/` | Force-delete a folder and everything inside it, no prompts |
| `rm -ri folder/` | Recursively delete but confirm each file |
| `rm -rv folder/` | Recursively delete with verbose output (good for logging what was removed) |
| `rm -I *.tmp` | Delete multiple files with one safety prompt |

---

## Safety Notes

- **`rm` does not support undo.** Unlike GUI file managers, there's no recycle bin — deleted files are gone unless recovered via specialized tools (and even then, not guaranteed).
- **Be extremely careful with `rm -rf`**, especially with variables in scripts:
  ```bash
  rm -rf "$dir"/*    # if $dir is empty/unset, this can expand to rm -rf /*
  ```
  Always quote variables and validate they're non-empty before using them in a `rm -rf` context.
- **Wildcards expand before `rm` sees them.** `rm *` in the wrong directory deletes everything in it. Double-check with `ls` first if unsure what a glob will match.
- Consider aliasing `rm` to `rm -i` in your `.bashrc` while learning, to build a habit of confirmation prompts:
  ```bash
  alias rm='rm -i'
  ```
Here alias create a command rm -i and always use it when we use rm 
## Related Commands

- `rmdir` — removes only empty directories (safer, more limited alternative to `rm -d`)
- `shred` — securely deletes a file by overwriting its contents before removal (relevant when data must be unrecoverable)
- `trash-cli` — an installable utility (`apt install trash-cli`) that gives you a recoverable trash bin instead of permanent deletion




### Hidden files 
`touch .filename` --> this will create a hidden file ( not visible in GUI and cli by default).
Use `ls -a` to show the hidden files too.



## `grep` — Search Text Using Patterns
 
`grep` searches input (files, or piped output) for lines matching a pattern and prints those lines. Short for **G**lobal **R**egular **E**xpression **P**rint. One of the most-used commands in recon/log analysis workflows.
 
### Basic Syntax
 
```bash
grep [OPTIONS] PATTERN [FILE...]
```
 
### Basic Usage
 
```bash
grep "error" logfile.txt         # print lines containing "error"
grep "error" file1.txt file2.txt # search multiple files
cat file.txt | grep "error"      # search piped input
```
 
---
 
### Flags
 
#### `-i`, `--ignore-case`
Case-insensitive matching.
 
```bash
grep -i "error" logfile.txt
```
 
---
 
#### `-v`, `--invert-match`
Print lines that do **NOT** match the pattern.
 
```bash
grep -v "debug" logfile.txt
```
 
---
 
#### `-r`, `-R`, `--recursive`
Search recursively through directories. `-R` also follows symbolic links, `-r` does not.
 
```bash
grep -r "TODO" ./src/
```
 
---
 
#### `-n`, `--line-number`
Prefix each match with its line number in the file.
 
```bash
grep -n "error" logfile.txt
# 42:connection error occurred
```
 
---
 
#### `-c`, `--count`
Print only a count of matching lines, not the lines themselves.
 
```bash
grep -c "error" logfile.txt
```
 
---
 
#### `-l`, `--files-with-matches`
Print only the names of files that contain at least one match (not the matching lines). Useful for scanning many files quickly.
 
```bash
grep -rl "api_key" ./project/
```
 
---
 
#### `-L`, `--files-without-match`
Print only the names of files that do **NOT** contain a match.
 
```bash
grep -rL "TODO" ./src/
```
 
---
 
#### `-w`, `--word-regexp`
Match only whole words, not substrings.
 
```bash
grep -w "cat" file.txt   # matches "cat" but not "category"
```
 
---
 
#### `-x`, `--line-regexp`
Match only whole lines — the entire line must equal the pattern.
 
```bash
grep -x "admin" users.txt
```
 
---
 
#### `-o`, `--only-matching`
Print only the matched portion of each line, not the whole line. Very useful for extracting data (IPs, URLs, tokens) out of larger output.
 
```bash
grep -oE "[0-9]{1,3}(\.[0-9]{1,3}){3}" logfile.txt   # extract IPv4 addresses
```
 
---
 
#### `-E`, `--extended-regexp`
Use Extended Regular Expressions (ERE) — lets you use `+`, `?`, `|`, `()` without escaping them. Equivalent to using `egrep`.
 
```bash
grep -E "error|warning" logfile.txt
```
 
---
 
#### `-F`, `--fixed-strings`
Treat the pattern as a literal string, not a regex. Faster and avoids needing to escape special characters. Equivalent to `fgrep`.
 
```bash
grep -F "192.168.1.1" logfile.txt
```
 
---
 
#### `-A NUM`, `--after-context=NUM`
Show `NUM` lines of trailing context after each match.
 
```bash
grep -A 3 "Exception" app.log
```
 
---
 
#### `-B NUM`, `--before-context=NUM`
Show `NUM` lines of leading context before each match.
 
```bash
grep -B 3 "Exception" app.log
```
 
---
 
#### `-C NUM`, `--context=NUM`
Show `NUM` lines of context both before and after each match.
 
```bash
grep -C 2 "Exception" app.log
```
 
---
 
#### `-e PATTERN`, `--regexp=PATTERN`
Specify a pattern explicitly — lets you combine multiple `-e` patterns (matches lines with any of them), or use a pattern that starts with a dash.
 
```bash
grep -e "error" -e "fail" logfile.txt
```
 
---
 
#### `-f FILE`, `--file=FILE`
Read patterns from a file, one pattern per line. Useful for matching against a wordlist.
 
```bash
grep -f patterns.txt logfile.txt
```
 
---
 
#### `--color=auto`
Highlight the matching text in color in the terminal output.
 
```bash
grep --color=auto "error" logfile.txt
```
 
---
 
#### `-q`, `--quiet`, `--silent`
Suppress all normal output; only the exit status indicates whether a match was found. Useful inside scripts/conditionals.
 
```bash
if grep -q "error" logfile.txt; then
    echo "Errors found"
fi
```
 
---
 
#### `-m NUM`, `--max-count=NUM`
Stop reading a file after `NUM` matching lines.
 
```bash
grep -m 5 "error" logfile.txt
```
 
---
 
#### `-s`, `--no-messages`
Suppress error messages about nonexistent or unreadable files.
 
```bash
grep -s "pattern" *.log
```
 
---
 
#### `-z`, `--null-data`
Treat input as a set of lines terminated by a null character instead of a newline. Useful for filenames with newlines or binary-safe processing.
 
---
 
### Common Flag Combinations
 
| Command | Meaning |
|---|---|
| `grep -rn "TODO" ./src/` | Recursively find "TODO" with line numbers |
| `grep -riE "error\|fail" logfile.txt` | Case-insensitive, extended regex, multiple patterns |
| `grep -c "error" *.log` | Count matches per file |
| `grep -oE "[0-9]{1,3}(\.[0-9]{1,3}){3}"` | Extract IPs from piped output |
| `grep -q "pattern" file.txt && echo found` | Silent check inside a conditional |
 
---
 
### Security/Recon Relevance
 
- `grep -oE` is commonly used to pull structured data (emails, IPs, URLs, tokens, API keys) out of large recon dumps or scraped pages.
- `grep -rl "password" .` — quickly find files likely containing sensitive strings during a source code review.
- Combine with `curl` to filter HTTP responses live: `curl -s https://target.com | grep -oE 'href="[^"]+"'`.
### Related Commands
 
- `egrep` — equivalent to `grep -E`
- `fgrep` — equivalent to `grep -F`
- `zgrep` — grep for gzip-compressed files
- `ripgrep` (`rg`) — much faster modern alternative, respects `.gitignore` by default
 
