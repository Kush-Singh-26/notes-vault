---
title: "The Complete Bash Scripting Reference for Beginners"
description: "Beginners guide for bash scripting."
---

## 1. What is Bash?

Bash stands for "Bourne Again SHell." It is the default command-line interpreter on most Linux distributions and macOS. A bash script is simply a plain text file containing a sequence of commands that the shell executes top to bottom, exactly as if you typed them yourself at the terminal.

Bash scripting lets you automate repetitive tasks, manage files and processes, build deployment pipelines, write system administration tools, and glue together programs that do not natively speak to each other.

---

## 2. Your First Script

Every bash script begins with a shebang line. This tells the operating system which interpreter to use.

```bash
#!/usr/bin/env bash

echo "Hello, World!"
```

The shebang `#!/usr/bin/env bash` is preferred over `#!/bin/bash` because it finds bash wherever it lives in the system's PATH, making scripts more portable.

Save the file as `hello.sh`.

---

## 3. Running Scripts

**Make the script executable:**
```bash
chmod +x hello.sh
```

**Run it:**
```bash
./hello.sh
```

**Run without making it executable (explicit interpreter):**
```bash
bash hello.sh
```

**Run in the current shell (no subshell — variables persist):**
```bash
source hello.sh
# or equivalently:
. hello.sh
```

The difference matters: running `./hello.sh` spawns a child process. Any variables or `cd` commands inside that script do not affect your current terminal session. Using `source` runs the script in your current session.

---

## 4. Variables

### Assigning Variables

```bash
name="Kush"
age=21
pi=3.14159
```

Critical rules:
- No spaces around the `=` sign. `name = "Kush"` is a syntax error.
- Variable names are case-sensitive. `Name` and `name` are different variables.
- By convention, local script variables use lowercase; environment variables use UPPERCASE.

### Using Variables

Prefix with `$` to expand a variable:

```bash
echo $name
echo "Hello, $name"
echo "Hello, ${name}"   # braces are safer in complex expressions
```

Always prefer `${variable}` with braces when the variable is followed immediately by other characters:

```bash
file="report"
echo "${file}_backup.txt"   # correct: report_backup.txt
echo "$file_backup.txt"     # wrong: bash looks for $file_backup
```

### Variable Types

**String (default):**
```bash
greeting="Good morning"
```

**Integer (declare explicitly for arithmetic):**
```bash
declare -i count=0
```

**Read-only (constant):**
```bash
readonly MAX_RETRIES=5
declare -r PI=3.14159
```

**Exported (available to child processes):**
```bash
export DATABASE_URL="postgres://localhost/mydb"
```

### Special Variables

These are built in and set automatically by bash:

| Variable | Meaning |
|---|---|
| `$0` | Name of the script itself |
| `$1`, `$2`, ... `$9` | Positional arguments passed to the script |
| `$@` | All positional arguments as separate words |
| `$*` | All positional arguments as a single word |
| `$#` | Number of positional arguments |
| `$?` | Exit code of the last command |
| `$$` | PID of the current shell |
| `$!` | PID of the last background process |
| `$_` | Last argument of the previous command |
| `$LINENO` | Current line number in the script |
| `$BASH_VERSION` | Version of bash running |
| `$HOME` | Current user's home directory |
| `$PWD` | Current working directory |
| `$USER` | Current logged-in username |
| `$HOSTNAME` | Machine hostname |
| `$RANDOM` | A random integer 0–32767 each time accessed |
| `$SECONDS` | Number of seconds since the script started |
| `$IFS` | Internal Field Separator (default: space, tab, newline) |

**Example using positional parameters:**
```bash
#!/usr/bin/env bash
# Usage: ./greet.sh Kush 21
echo "Name: $1"
echo "Age:  $2"
echo "Total arguments: $#"
echo "All arguments: $@"
```

### Quoting Rules

This is one of the most important concepts in bash and the source of most beginner bugs.

**Double quotes** `"..."`: Expand variables and command substitutions. Protect spaces in values.
```bash
name="Kush Singh"
echo "$name"        # prints: Kush Singh
echo $name          # also prints: Kush Singh (but dangerous with spaces in filenames)
```

**Single quotes** `'...'`: Treat everything literally. No expansion at all.
```bash
echo '$name'        # prints: $name (literal)
echo '$((2+2))'     # prints: $((2+2)) (literal)
```

**No quotes**: Word splitting and glob expansion happen. Dangerous with filenames containing spaces.
```bash
file="my report.txt"
cat $file           # tries to cat "my" and "report.txt" separately — bug!
cat "$file"         # correct
```

**Rule of thumb: always double-quote variable expansions unless you specifically need word splitting.**

---

## 5. User Input

### Reading from stdin

```bash
echo -n "Enter your name: "
read name
echo "Hello, $name"
```

**Read with a prompt on the same line:**
```bash
read -p "Enter your name: " name
```

**Read silently (for passwords):**
```bash
read -sp "Enter password: " password
echo    # newline after silent input
```

**Read with a timeout:**
```bash
read -t 10 -p "You have 10 seconds: " answer
```

**Read into an array:**
```bash
read -a colors -p "Enter colors separated by spaces: "
echo "First color: ${colors[0]}"
```

**Read a specific number of characters:**
```bash
read -n 1 -p "Press any key to continue..."
```

### Command-Line Arguments

```bash
#!/usr/bin/env bash
# ./deploy.sh staging 8080

environment=$1
port=${2:-3000}   # default to 3000 if not provided

echo "Deploying to $environment on port $port"
```

### Parsing Options with getopts

`getopts` is the standard POSIX way to parse flags like `-v`, `-f filename`.

```bash
#!/usr/bin/env bash

verbose=false
output_file=""

while getopts "vf:" opt; do
    case $opt in
        v) verbose=true ;;
        f) output_file="$OPTARG" ;;
        ?) echo "Usage: $0 [-v] [-f filename]"; exit 1 ;;
    esac
done

# Shift past the parsed options
shift $((OPTIND - 1))

if $verbose; then
    echo "Verbose mode on"
fi
echo "Output file: $output_file"
echo "Remaining arguments: $@"
```

The colon after `f:` means `-f` expects an argument. `$OPTARG` holds that argument.

---

## 6. Arithmetic

Bash only natively handles integer arithmetic.

### $(( )) — Arithmetic Expansion

```bash
a=10
b=3

echo $((a + b))    # 13
echo $((a - b))    # 7
echo $((a * b))    # 30
echo $((a / b))    # 3  (integer division — truncates)
echo $((a % b))    # 1  (modulo / remainder)
echo $((a ** b))   # 1000 (exponentiation)
```

**Increment and decrement:**
```bash
count=0
((count++))        # post-increment
((count--))        # post-decrement
((count += 5))     # add 5
((count *= 2))     # multiply by 2
```

**Assigning arithmetic results:**
```bash
result=$((10 * 5 + 3))
echo $result    # 53
```

### let

```bash
let "result = 10 * 5"
let result++
```

### expr (older, avoid in modern scripts)

```bash
result=$(expr 10 + 5)
```

### bc — Floating Point Arithmetic

Bash does not do floating point. Use `bc`:

```bash
result=$(echo "scale=4; 22/7" | bc)
echo $result    # 3.1428

# Square root:
sqrt=$(echo "scale=6; sqrt(2)" | bc -l)
echo $sqrt      # 1.414213
```

The `scale` variable in `bc` sets the number of decimal places. The `-l` flag loads the math library (required for `sqrt`, `sin`, `cos`, etc.).

### Comparison in Arithmetic Context

Inside `(( ))`, you can use C-style comparisons that return 0 (true) or 1 (false):

```bash
if (( a > b )); then
    echo "a is greater"
fi

if (( a == 10 )); then
    echo "a is ten"
fi
```

---

## 7. Strings

### String Length

```bash
name="Kush"
echo ${#name}     # 4
```

### Substring Extraction

Syntax: `${variable:offset:length}`

```bash
str="Hello, World!"
echo ${str:7:5}     # World
echo ${str:7}       # World!  (from offset 7 to end)
echo ${str: -6}     # orld!  (from 6 chars before end — note the space)
```

### String Replacement

**Replace first occurrence:**
```bash
sentence="I like cats and cats"
echo ${sentence/cats/dogs}    # I like dogs and cats
```

**Replace all occurrences:**
```bash
echo ${sentence//cats/dogs}   # I like dogs and dogs
```

**Replace only at the beginning:**
```bash
echo ${sentence/#I/We}        # We like cats and cats
```

**Replace only at the end:**
```bash
echo ${sentence/%cats/dogs}   # I like cats and dogs
```

### Case Conversion (Bash 4+)

```bash
str="Hello World"
echo ${str,,}     # hello world  (all lowercase)
echo ${str^^}     # HELLO WORLD  (all uppercase)
echo ${str^}      # Hello World  (capitalize first char)
echo ${str,}      # hELLO WORLD  (lowercase first char)
```

### Default Values

```bash
# Use default if variable is unset or empty
echo ${name:-"Anonymous"}

# Assign default if variable is unset or empty
echo ${name:="Anonymous"}     # also assigns "Anonymous" to $name

# Error if unset
echo ${name:?"name is required"}

# Use alternate value if variable IS set
echo ${name:+"name is set"}
```

### Trimming (Parameter Expansion)

```bash
filename="report.tar.gz"

# Remove shortest match from the end
echo ${filename%.*}     # report.tar

# Remove longest match from the end
echo ${filename%%.*}    # report

# Remove shortest match from the beginning
echo ${filename#*.}     # tar.gz

# Remove longest match from the beginning
echo ${filename##*.}    # gz
```

This is extremely useful for extracting file extensions, directory paths, etc.

**Practical examples:**
```bash
path="/home/kush/scripts/deploy.sh"

echo ${path##*/}      # deploy.sh   (basename)
echo ${path%/*}       # /home/kush/scripts  (dirname)
```

### Checking if a String Contains a Substring

```bash
str="Hello, World"
if [[ "$str" == *"World"* ]]; then
    echo "Found"
fi
```

### String Concatenation

Bash concatenates by placing variables adjacent to each other:

```bash
first="Hello"
second=" World"
combined="${first}${second}"
echo $combined     # Hello World
```

---

## 8. Arrays

Bash supports indexed arrays (zero-based) and associative arrays (key-value maps).

### Indexed Arrays

**Declaration and assignment:**
```bash
# Method 1: declare all at once
fruits=("apple" "banana" "cherry")

# Method 2: declare first, assign by index
declare -a colors
colors[0]="red"
colors[1]="green"
colors[2]="blue"

# Method 3: append
fruits+=("date")
```

**Accessing elements:**
```bash
echo ${fruits[0]}      # apple
echo ${fruits[2]}      # cherry
echo ${fruits[-1]}     # cherry (last element, bash 4.3+)
```

**All elements:**
```bash
echo ${fruits[@]}      # apple banana cherry date
echo ${fruits[*]}      # same but splits differently when quoted
```

`"${fruits[@]}"` expands each element as a separate word. Always use `[@]` with double quotes when iterating.

**Number of elements:**
```bash
echo ${#fruits[@]}     # 4
```

**All indices:**
```bash
echo ${!fruits[@]}     # 0 1 2 3
```

**Slicing:**
```bash
echo ${fruits[@]:1:2}  # banana cherry (starting at index 1, take 2)
```

**Deleting an element:**
```bash
unset fruits[1]
echo ${fruits[@]}      # apple cherry date (index 1 is now empty, gap remains)
```

**Iterating:**
```bash
for fruit in "${fruits[@]}"; do
    echo "$fruit"
done
```

**Iterating with index:**
```bash
for i in "${!fruits[@]}"; do
    echo "[$i] = ${fruits[$i]}"
done
```

**Deleting the entire array:**
```bash
unset fruits
```

### Associative Arrays (Bash 4+)

Associative arrays use string keys.

```bash
declare -A person
person["name"]="Kush"
person["age"]="21"
person["university"]="KIIT"

# Or initialize all at once:
declare -A config=(
    ["host"]="localhost"
    ["port"]="5432"
    ["db"]="myapp"
)
```

**Accessing:**
```bash
echo ${person["name"]}    # Kush
echo ${config["port"]}    # 5432
```

**All keys:**
```bash
echo ${!person[@]}
```

**All values:**
```bash
echo ${person[@]}
```

**Iterating:**
```bash
for key in "${!config[@]}"; do
    echo "$key => ${config[$key]}"
done
```

**Checking if a key exists:**
```bash
if [[ -v person["name"] ]]; then
    echo "Key exists"
fi
```

---

## 9. Conditional Logic

### test, [ ], and [[ ]]

There are three ways to test conditions in bash:

- `test expression` — POSIX, most compatible
- `[ expression ]` — equivalent to `test`, requires spaces inside brackets
- `[[ expression ]]` — bash-specific, more powerful, safer; prefer this

**Important:** `[ ]` and `[[ ]]` must have spaces inside: `[ -f file ]` not `[-f file]`.

### File Tests

```bash
[[ -e "$path" ]]    # exists (any type)
[[ -f "$path" ]]    # is a regular file
[[ -d "$path" ]]    # is a directory
[[ -l "$path" ]]    # is a symbolic link
[[ -r "$path" ]]    # is readable
[[ -w "$path" ]]    # is writable
[[ -x "$path" ]]    # is executable
[[ -s "$path" ]]    # exists and is non-empty (size > 0)
[[ -z "$path" ]]    # string is empty
[[ -n "$path" ]]    # string is non-empty
```

### String Comparisons

```bash
[[ "$a" == "$b" ]]    # equal
[[ "$a" != "$b" ]]    # not equal
[[ "$a" < "$b" ]]     # lexicographically less (inside [[ ]])
[[ "$a" > "$b" ]]     # lexicographically greater
[[ -z "$a" ]]         # is empty
[[ -n "$a" ]]         # is non-empty
[[ "$a" =~ ^[0-9]+$ ]] # matches regex (bash 3.2+)
```

### Integer Comparisons

Inside `[ ]` and `[[ ]]`, use these flags for numbers (not `<` or `>`):

```bash
[[ $a -eq $b ]]    # equal
[[ $a -ne $b ]]    # not equal
[[ $a -lt $b ]]    # less than
[[ $a -le $b ]]    # less than or equal
[[ $a -gt $b ]]    # greater than
[[ $a -ge $b ]]    # greater than or equal
```

Or use arithmetic context which is cleaner:
```bash
(( a > b ))
(( a == b ))
```

### Logical Operators

```bash
# Inside [[ ]]
[[ condition1 && condition2 ]]    # AND
[[ condition1 || condition2 ]]    # OR
[[ ! condition ]]                 # NOT

# Chaining commands (outside brackets)
command1 && command2    # run command2 only if command1 succeeds
command1 || command2    # run command2 only if command1 fails
```

### if / elif / else

```bash
if [[ condition ]]; then
    # commands
elif [[ other_condition ]]; then
    # commands
else
    # commands
fi
```

**Real example:**
```bash
#!/usr/bin/env bash

score=$1

if [[ -z "$score" ]]; then
    echo "Usage: $0 <score>"
    exit 1
elif (( score >= 90 )); then
    echo "Grade: A"
elif (( score >= 80 )); then
    echo "Grade: B"
elif (( score >= 70 )); then
    echo "Grade: C"
else
    echo "Grade: F"
fi
```

### case Statement

`case` is cleaner than chained `if/elif` when matching one variable against multiple patterns.

```bash
case "$variable" in
    pattern1)
        commands
        ;;
    pattern2 | pattern3)    # multiple patterns with |
        commands
        ;;
    *)                       # default (like else)
        commands
        ;;
esac
```

**Real example:**
```bash
day=$(date +%A)

case "$day" in
    Monday | Tuesday | Wednesday | Thursday | Friday)
        echo "Weekday"
        ;;
    Saturday | Sunday)
        echo "Weekend"
        ;;
    *)
        echo "Unknown day"
        ;;
esac
```

**With glob patterns:**
```bash
filename="photo.jpg"

case "$filename" in
    *.jpg | *.jpeg | *.png | *.gif)
        echo "Image file"
        ;;
    *.mp3 | *.wav | *.flac)
        echo "Audio file"
        ;;
    *.sh)
        echo "Shell script"
        ;;
    *)
        echo "Unknown type"
        ;;
esac
```

---

## 10. Loops

### for Loop — Iterating a List

```bash
for item in item1 item2 item3; do
    echo "$item"
done
```

**Iterating over an array:**
```bash
fruits=("apple" "banana" "cherry")
for fruit in "${fruits[@]}"; do
    echo "$fruit"
done
```

**Iterating over files:**
```bash
for file in /var/log/*.log; do
    echo "Processing: $file"
done
```

**Iterating over command output:**
```bash
for user in $(cat /etc/passwd | cut -d: -f1); do
    echo "$user"
done
```

**Note:** Using `$(command)` in a for loop breaks on whitespace in filenames. For files, prefer `find` with `-exec` or a `while read` loop instead.

### C-Style for Loop

```bash
for (( i=0; i<10; i++ )); do
    echo "i = $i"
done

# Count down:
for (( i=10; i>0; i-- )); do
    echo "$i"
done

# Step by 2:
for (( i=0; i<=20; i+=2 )); do
    echo "$i"
done
```

### Brace Expansion in Loops

```bash
# Numbers 1 to 10:
for i in {1..10}; do
    echo "$i"
done

# With step (bash 4+):
for i in {0..20..2}; do
    echo "$i"
done

# Letters:
for letter in {a..z}; do
    echo "$letter"
done
```

### while Loop

Runs as long as the condition is true.

```bash
count=1
while [[ $count -le 5 ]]; do
    echo "Count: $count"
    ((count++))
done
```

**Reading a file line by line (the correct way):**
```bash
while IFS= read -r line; do
    echo "$line"
done < input.txt
```

`IFS=` prevents stripping leading/trailing whitespace. `-r` prevents backslash interpretation. This is the canonical safe way to read files in bash.

**Reading output of a command line by line:**
```bash
while IFS= read -r line; do
    echo "User: $line"
done < <(cut -d: -f1 /etc/passwd)
```

The `< <(command)` is a process substitution. It feeds the output of `command` as if it were a file.

**Infinite loop with break:**
```bash
while true; do
    read -p "Enter 'quit' to exit: " input
    if [[ "$input" == "quit" ]]; then
        break
    fi
    echo "You said: $input"
done
```

### until Loop

Runs until the condition becomes true (opposite of while).

```bash
count=1
until [[ $count -gt 5 ]]; do
    echo "$count"
    ((count++))
done
```

### Loop Control

```bash
break        # exit the loop immediately
continue     # skip to the next iteration
break 2      # break out of 2 nested loops
continue 2   # continue the outer of 2 nested loops
```

**Example with continue:**
```bash
for i in {1..10}; do
    if (( i % 2 == 0 )); then
        continue    # skip even numbers
    fi
    echo "$i"
done
```

### select Loop — Interactive Menus

```bash
options=("Start server" "Stop server" "Restart server" "Quit")

select choice in "${options[@]}"; do
    case $choice in
        "Start server")  echo "Starting..."; break ;;
        "Stop server")   echo "Stopping..."; break ;;
        "Restart server") echo "Restarting..."; break ;;
        "Quit")          break ;;
        *) echo "Invalid option $REPLY" ;;
    esac
done
```

`select` automatically prints a numbered menu and reads the user's choice into `$REPLY`.

---

## 11. Functions

### Defining and Calling Functions

```bash
# Definition (two equivalent syntaxes):
greet() {
    echo "Hello, $1!"
}

function greet {
    echo "Hello, $1!"
}

# Call:
greet "Kush"    # Hello, Kush!
```

Functions must be defined before they are called.

### Parameters

Functions receive arguments exactly like scripts: `$1`, `$2`, `$@`, `$#`, etc.

```bash
add() {
    local a=$1
    local b=$2
    echo $((a + b))
}

result=$(add 10 20)
echo $result    # 30
```

### local Variables

Variables inside functions are global by default, which is a major source of bugs. Always use `local` for function-internal variables:

```bash
x=100

modify() {
    local x=999    # local to this function
    echo "Inside: $x"
}

modify
echo "Outside: $x"    # still 100
```

Without `local`, the function would overwrite `x=100` in the global scope.

### Return Values

Bash functions can only `return` an integer exit code (0–255). To return a string or other data, use one of these patterns:

**Pattern 1: Echo and capture with command substitution**
```bash
to_uppercase() {
    echo "${1^^}"
}

result=$(to_uppercase "hello")
echo $result    # HELLO
```

**Pattern 2: Assign to a global/nameref variable**
```bash
get_hostname() {
    hostname_result=$(hostname)
}

get_hostname
echo $hostname_result
```

**Pattern 3: Return an exit code and check $?**
```bash
is_even() {
    (( $1 % 2 == 0 ))    # returns 0 (true) if even, 1 (false) if odd
}

if is_even 4; then
    echo "Even"
fi
```

### Recursive Functions

```bash
factorial() {
    local n=$1
    if (( n <= 1 )); then
        echo 1
    else
        local prev=$(factorial $((n - 1)))
        echo $((n * prev))
    fi
}

echo $(factorial 5)    # 120
```

### Function Libraries

You can source a file of helper functions into your script:

```bash
# helpers.sh
log_info() { echo "[INFO]  $*"; }
log_error() { echo "[ERROR] $*" >&2; }
log_warn() { echo "[WARN]  $*"; }
```

```bash
# main.sh
source ./helpers.sh

log_info "Starting deployment"
log_error "Connection refused"
```

---

## 12. Input/Output & Redirection

### File Descriptors

Every process has three standard file descriptors:

| FD | Name | Default |
|---|---|---|
| 0 | stdin | keyboard |
| 1 | stdout | terminal screen |
| 2 | stderr | terminal screen |

### Output Redirection

```bash
echo "Hello" > file.txt        # write stdout to file (overwrites)
echo "Hello" >> file.txt       # append stdout to file
echo "Error" 2> error.log      # write stderr to file
echo "Both" > out.txt 2>&1     # redirect both stdout and stderr to file
echo "Both" &> out.txt         # bash shorthand for above
echo "Both" >> out.txt 2>&1    # append both to file
```

**Discard output (send to the null device):**
```bash
command > /dev/null            # discard stdout
command 2> /dev/null           # discard stderr
command &> /dev/null           # discard all output
```

### Input Redirection

```bash
command < file.txt             # feed file as stdin
wc -l < /etc/passwd            # count lines in passwd
```

### Here Document (heredoc)

Feed multiline text as stdin to a command:

```bash
cat << EOF
Line 1
Line 2
$variable_is_expanded_here
EOF
```

The delimiter (`EOF` is conventional but can be any word). If you quote the delimiter, no variable expansion occurs:

```bash
cat << 'EOF'
$variable is NOT expanded here
EOF
```

**Indented heredoc (bash 4.4+):**
```bash
cat <<- EOF
    This leading tab is stripped.
    Useful inside indented code.
EOF
```

### Here String

Feed a single string as stdin:

```bash
wc -w <<< "Hello World"    # 2
read a b <<< "hello world"
echo $a    # hello
```

### tee — Splitting Output

`tee` writes to both stdout and a file simultaneously:

```bash
make 2>&1 | tee build.log    # see output on screen AND save it
```

### Process Substitution

Treat the output of a command as a file:

```bash
diff <(sort file1.txt) <(sort file2.txt)    # compare sorted versions
```

---

## 13. Pipes

A pipe `|` connects the stdout of one command to the stdin of the next. Pipes are one of the most powerful Unix concepts.

```bash
command1 | command2 | command3
```

**Examples:**

```bash
# Count files in a directory:
ls | wc -l

# Find the 5 largest files:
du -sh * | sort -rh | head -5

# Find all processes named "python":
ps aux | grep python

# Count unique words in a file:
cat file.txt | tr ' ' '\n' | sort | uniq -c | sort -rn

# Monitor a log file and filter for errors:
tail -f app.log | grep --line-buffered "ERROR"
```

### Pipeline Exit Codes

By default, a pipeline's exit code is the exit code of the last command. To make a pipeline fail if any command fails:

```bash
set -o pipefail
```

This is important in scripts where you want `ls /nonexistent | wc -l` to fail, not silently return 0.

### Named Pipes (FIFOs)

```bash
mkfifo mypipe
command1 > mypipe &
command2 < mypipe
```

Useful for inter-process communication.

---

## 14. Exit Codes & Error Handling

### Exit Codes

Every command returns an exit code when it finishes. By convention: 0 = success, non-zero = failure.

```bash
ls /tmp
echo $?    # 0 (success)

ls /nonexistent
echo $?    # 2 (error)
```

Always check `$?` immediately after the command you care about — it is overwritten by every subsequent command.

### Exit a Script

```bash
exit 0      # success
exit 1      # general error
exit 2      # misuse of shell builtins
```

### set -e — Exit on Error

```bash
set -e      # exit the script if any command returns non-zero
```

Extremely important for production scripts. Without it, a failed command is silently ignored and execution continues.

### set -u — Treat Unset Variables as Errors

```bash
set -u      # exit if you reference an unset variable
```

Without it, `echo $typo` just prints nothing. With `set -u`, it crashes with an error.

### set -x — Debug Mode

```bash
set -x      # print each command before executing it
```

Invaluable for debugging. Turn off with `set +x`.

### The "Safe Script" Header

Most production bash scripts start with:

```bash
#!/usr/bin/env bash
set -euo pipefail
```

This means: exit on any error (`-e`), treat unset variables as errors (`-u`), and propagate pipe failures (`-o pipefail`).

### Handling Errors Explicitly

```bash
if ! cp source.txt dest.txt; then
    echo "Copy failed" >&2
    exit 1
fi
```

Or using `||`:

```bash
cp source.txt dest.txt || { echo "Copy failed" >&2; exit 1; }
```

The `{ }` groups multiple commands. The semicolon before `}` is required.

### trap — Catch Signals and Errors

`trap` runs a command when the script receives a signal or exits.

```bash
# Clean up on exit (even if the script is killed):
cleanup() {
    rm -f /tmp/myapp_lock
    echo "Cleaned up"
}
trap cleanup EXIT

# Handle Ctrl+C:
trap "echo 'Interrupted!'; exit 1" INT

# Handle errors:
trap "echo 'Error on line $LINENO'" ERR
```

**Signals you can trap:**

| Signal | When triggered |
|---|---|
| EXIT | Script exits for any reason |
| ERR | Any command returns non-zero |
| INT | User presses Ctrl+C |
| TERM | Script receives kill signal |
| DEBUG | Before every command (verbose) |
| HUP | Terminal hangup |

**Practical cleanup pattern:**
```bash
#!/usr/bin/env bash
set -euo pipefail

TMPDIR=$(mktemp -d)
trap "rm -rf $TMPDIR" EXIT

# Do work in $TMPDIR — it will always be cleaned up
cp important_file.txt "$TMPDIR/"
process "$TMPDIR/important_file.txt"
```

---

## 15. File & Directory Operations

### Creating and Removing

```bash
touch file.txt                  # create empty file or update timestamp
mkdir mydir                     # create directory
mkdir -p path/to/nested/dir     # create all intermediate dirs
rm file.txt                     # remove file
rm -f file.txt                  # force remove (no error if not found)
rm -r mydir                     # remove directory recursively
rm -rf mydir                    # force remove directory (dangerous!)
rmdir mydir                     # remove empty directory only
```

### Copying and Moving

```bash
cp source dest                  # copy file
cp -r sourcedir destdir         # copy directory recursively
cp -p file dest                 # preserve timestamps and permissions
mv source dest                  # move or rename
```

### Reading Files

```bash
cat file.txt                    # print entire file
head -n 20 file.txt             # first 20 lines
tail -n 20 file.txt             # last 20 lines
tail -f file.txt                # follow file (live updates)
wc -l file.txt                  # count lines
wc -w file.txt                  # count words
wc -c file.txt                  # count bytes
```

### Finding Files

```bash
find /path -name "*.sh"                     # find by name
find /path -type f -name "*.log"            # only regular files
find /path -type d -name "backup"           # only directories
find /path -mtime -7                        # modified in last 7 days
find /path -size +100M                      # files larger than 100MB
find /path -name "*.tmp" -delete            # find and delete
find /path -name "*.sh" -exec chmod +x {} \;  # find and run command
```

### Checking if Files Exist

```bash
if [[ -f "$file" ]]; then
    echo "$file exists and is a regular file"
fi

if [[ -d "$dir" ]]; then
    echo "$dir is a directory"
fi

if [[ ! -e "$path" ]]; then
    echo "$path does not exist"
fi
```

### Temporary Files

```bash
tmpfile=$(mktemp)              # creates /tmp/tmp.XXXXXX
tmpdir=$(mktemp -d)            # creates a temp directory
trap "rm -f $tmpfile" EXIT     # always clean up
```

### File Permissions

```bash
chmod +x script.sh             # add execute
chmod 755 script.sh            # rwxr-xr-x
chmod 644 file.txt             # rw-r--r--
chown user:group file.txt      # change owner
ls -la                         # list with permissions
```

---

## 16. String Manipulation

Beyond what is covered in section 7, here are more string tools:

### printf — Formatted Output

`printf` is more precise than `echo` and does not add a newline by default:

```bash
printf "Name: %s\n" "Kush"
printf "Score: %d / %d\n" 92 100
printf "Pi: %.4f\n" 3.14159265
printf "%-20s %5d\n" "item" 42    # left-align string, right-align number
```

Format specifiers: `%s` string, `%d` integer, `%f` float, `%x` hex, `%o` octal.

### tr — Transliterate Characters

```bash
echo "hello world" | tr 'a-z' 'A-Z'    # HELLO WORLD
echo "hello" | tr -d 'l'                # heo (delete 'l')
echo "aabbcc" | tr -s 'a-z'            # abc (squeeze repeated chars)
echo "hello world" | tr ' ' '\n'       # one word per line
```

### cut — Extract Fields

```bash
echo "kush:21:KIIT" | cut -d: -f1      # kush (field 1, colon delimiter)
echo "kush:21:KIIT" | cut -d: -f2-3    # 21:KIIT (fields 2 through 3)
cut -c1-5 file.txt                      # characters 1 through 5 of each line
```

### sort — Sort Lines

```bash
sort file.txt                   # alphabetical
sort -n numbers.txt             # numeric sort
sort -rn numbers.txt            # numeric, reversed
sort -u file.txt                # sort and remove duplicates
sort -t: -k3 -n /etc/passwd     # sort by field 3, colon delimiter
```

### uniq — Remove Duplicate Lines

```bash
sort file.txt | uniq             # remove consecutive duplicates (sort first!)
sort file.txt | uniq -c          # prefix each line with count
sort file.txt | uniq -d          # only show duplicate lines
sort file.txt | uniq -u          # only show unique lines
```

### paste — Merge Lines

```bash
paste file1.txt file2.txt        # side by side with tab separator
paste -d, file1.txt file2.txt    # comma separated
```

---

## 17. Regular Expressions, grep, sed, and awk

These three tools are the backbone of text processing in bash. Each deserves a full book, but here is what every beginner needs.

### grep — Search for Patterns

```bash
grep "pattern" file.txt              # lines containing pattern
grep -i "pattern" file.txt           # case-insensitive
grep -v "pattern" file.txt           # lines NOT containing pattern
grep -n "pattern" file.txt           # show line numbers
grep -c "pattern" file.txt           # count matching lines
grep -r "pattern" /path/             # recursive search
grep -l "pattern" *.txt              # only show filenames
grep -w "word" file.txt              # whole word match only
grep -A 3 "pattern" file.txt         # 3 lines after match
grep -B 3 "pattern" file.txt         # 3 lines before match
grep -E "pattern1|pattern2" file.txt # extended regex (alternation)
grep -P "\d{3}-\d{4}" file.txt       # Perl-compatible regex
```

**Useful regex patterns:**
```
^          start of line
$          end of line
.          any single character
*          zero or more of preceding
+          one or more (extended regex)
?          zero or one (extended regex)
[abc]      character class: a, b, or c
[^abc]     not a, b, or c
[a-z]      range
\d         digit (Perl regex)
\w         word character (Perl regex)
\s         whitespace (Perl regex)
(a|b)      group: a or b
{3}        exactly 3 of preceding
{2,5}      between 2 and 5
```

### sed — Stream Editor

`sed` reads a file line by line and applies transformations.

**Basic substitution:**
```bash
sed 's/old/new/' file.txt           # replace first occurrence per line
sed 's/old/new/g' file.txt          # replace all occurrences (global)
sed 's/old/new/gi' file.txt         # case-insensitive replace
sed -i 's/old/new/g' file.txt       # in-place edit (modifies the file)
sed -i.bak 's/old/new/g' file.txt   # in-place with backup
```

**Delete lines:**
```bash
sed '/pattern/d' file.txt           # delete lines matching pattern
sed '5d' file.txt                   # delete line 5
sed '2,5d' file.txt                 # delete lines 2 through 5
sed '/^$/d' file.txt                # delete blank lines
sed '/^#/d' file.txt                # delete comment lines
```

**Print specific lines:**
```bash
sed -n '5p' file.txt                # print only line 5
sed -n '2,5p' file.txt              # print lines 2–5
sed -n '/pattern/p' file.txt        # print matching lines
```

**Insert and append:**
```bash
sed '3i\New line before line 3' file.txt
sed '3a\New line after line 3' file.txt
```

**Multiple commands:**
```bash
sed -e 's/foo/bar/g' -e '/^#/d' file.txt
```

### awk — Pattern Scanning and Processing

awk processes text line by line, splitting each line into fields. It is a full programming language.

**Basic syntax:**
```
awk 'pattern { action }' file
```

**Built-in variables:**

| Variable | Meaning |
|---|---|
| `$0` | Entire current line |
| `$1`, `$2`, ... | Fields (space-separated by default) |
| `NF` | Number of fields in current line |
| `NR` | Current line number |
| `FS` | Input field separator (default: whitespace) |
| `OFS` | Output field separator |
| `RS` | Input record separator (default: newline) |
| `ORS` | Output record separator |

**Examples:**

```bash
# Print first field of each line:
awk '{print $1}' file.txt

# Print with custom delimiter:
awk -F: '{print $1}' /etc/passwd       # first field, colon-separated

# Print lines where field 3 > 100:
awk '$3 > 100' file.txt

# Print specific fields with formatting:
awk -F: '{printf "%-20s %s\n", $1, $7}' /etc/passwd

# Sum a column:
awk '{sum += $1} END {print sum}' numbers.txt

# Count lines matching a pattern:
awk '/error/{count++} END {print count}' app.log

# Print lines 5 through 10:
awk 'NR>=5 && NR<=10' file.txt

# Print last field of each line:
awk '{print $NF}' file.txt

# BEGIN and END blocks:
awk 'BEGIN {print "Start"} {print $0} END {print "End"}' file.txt

# Filter and transform CSV:
awk -F, 'NR>1 && $3 > 50 {print $1, $2}' data.csv
```

---

## 18. Process Management

### Running Processes

```bash
command &           # run in background
command1 && command2   # run command2 only if command1 succeeds
command1 || command2   # run command2 only if command1 fails
command1; command2     # run command2 regardless
```

### Jobs

```bash
jobs                # list background jobs
fg                  # bring last background job to foreground
fg %2               # bring job 2 to foreground
bg                  # resume last suspended job in background
Ctrl+Z              # suspend current foreground job
Ctrl+C              # terminate current foreground job
```

### Process Info

```bash
ps aux              # all running processes
ps aux | grep name  # find a process
pgrep name          # get PID of process by name
pidof name          # same
top                 # interactive process viewer
htop                # better interactive viewer (install separately)
```

### Killing Processes

```bash
kill PID            # send SIGTERM (graceful stop)
kill -9 PID         # send SIGKILL (force stop)
kill -HUP PID       # send SIGHUP (reload config)
pkill name          # kill by name
killall name        # kill all processes with this name
```

### wait — Synchronize Parallel Jobs

```bash
job1 &
job2 &
job3 &
wait    # wait for all background jobs to finish

# Wait for a specific PID:
some_command &
pid=$!
wait $pid
echo "Exit code: $?"
```

**Running tasks in parallel with a limit:**
```bash
max_jobs=4
current_jobs=0

for item in "${items[@]}"; do
    process_item "$item" &
    ((current_jobs++))
    if (( current_jobs >= max_jobs )); then
        wait
        current_jobs=0
    fi
done
wait    # final wait for remaining jobs
```

### Command Substitution

Run a command and capture its output:

```bash
today=$(date +%Y-%m-%d)
lines=$(wc -l < file.txt)
hostname=$(hostname)
```

The old backtick syntax `` `command` `` works the same but is harder to nest. Always prefer `$()`.

---

## 19. Scheduling with Cron

### Crontab Syntax

```
* * * * * command
│ │ │ │ │
│ │ │ │ └─── day of week (0–7, 0 and 7 = Sunday)
│ │ │ └───── month (1–12)
│ │ └─────── day of month (1–31)
│ └───────── hour (0–23)
└─────────── minute (0–59)
```

**Examples:**

```
# Run every minute:
* * * * * /path/to/script.sh

# Run every day at 2:30 AM:
30 2 * * * /path/to/backup.sh

# Run every Monday at 8 AM:
0 8 * * 1 /path/to/report.sh

# Run every 15 minutes:
*/15 * * * * /path/to/check.sh

# Run on 1st of every month at midnight:
0 0 1 * * /path/to/monthly.sh

# Run at 9 AM on weekdays:
0 9 * * 1-5 /path/to/job.sh
```

### Managing Crontabs

```bash
crontab -e          # edit your crontab
crontab -l          # list your crontab
crontab -r          # remove your crontab
crontab -u user -l  # list another user's crontab (requires root)
```

### Cron Best Practices

Always use absolute paths in cron jobs. Cron runs with a minimal environment and will not find commands in your `$PATH`. Redirect output to a log file:

```
0 2 * * * /usr/bin/bash /home/kush/scripts/backup.sh >> /var/log/backup.log 2>&1
```

---

## 20. Debugging

### set -x — Trace Execution

```bash
set -x      # turn on tracing
# ... your commands ...
set +x      # turn off tracing
```

Or enable for a specific block:
```bash
{
    set -x
    suspicious_command
    set +x
} 2> trace.log
```

### Running with bash -x

```bash
bash -x script.sh
```

This prints every command with a `+` prefix before executing it.

### bash -n — Check Syntax Without Running

```bash
bash -n script.sh
```

Parses the script for syntax errors without executing anything.

### Verbose Mode: bash -v

```bash
bash -v script.sh
```

Prints each line of the script as it is read (before expansion).

### Using echo and printf for Debugging

Add debug output and guard it with a flag:

```bash
DEBUG=true

debug() {
    if [[ "$DEBUG" == "true" ]]; then
        echo "[DEBUG] $*" >&2
    fi
}

debug "About to process file: $filename"
```

Sending debug output to stderr (`>&2`) keeps it separate from the program's real output.

### Checking Exit Codes

```bash
cp source.txt dest.txt
echo "cp exit code: $?"
```

### Common Bugs and How to Find Them

**Bug: Unquoted variable with spaces**
```bash
file="my file.txt"
rm $file        # tries to remove "my" and "file.txt" separately
rm "$file"      # correct
```

**Bug: Forgetting to initialize a counter**
```bash
# Wrong: $count might be empty, causing (( )) to silently do nothing
((count++))

# Right:
count=0
((count++))
```

**Bug: Using = instead of == in [ ]**
```bash
if [ $var = "hello" ]    # works but is POSIX style
if [[ $var == "hello" ]] # bash style, safer
```

**Bug: Comparing strings with integer operators**
```bash
if [[ "10" -gt "9" ]]     # correct (numeric comparison)
if [[ "10" > "9" ]]       # wrong! "10" < "9" lexicographically
```

**Bug: cat file | while read creates a subshell**
```bash
# Variables modified inside the while won't be visible outside
count=0
cat file.txt | while read line; do
    ((count++))    # modifies a copy in the subshell
done
echo $count    # still 0!

# Fix: use input redirection instead
while IFS= read -r line; do
    ((count++))
done < file.txt
echo $count    # correct
```

### ShellCheck

`shellcheck` is a static analysis tool for bash scripts. It catches the majority of common bugs automatically.

```bash
shellcheck script.sh
```

Install it with your package manager (`apt install shellcheck`, `brew install shellcheck`). Use it. It will save you hours.

---

## 21. Best Practices

**1. Always start with the safe header.**
```bash
#!/usr/bin/env bash
set -euo pipefail
```

**2. Always quote your variables.**
```bash
echo "$variable"    # not echo $variable
```

**3. Use `[[ ]]` instead of `[ ]`.**
`[[ ]]` is safer, supports `&&`/`||` directly, and handles empty strings without errors.

**4. Use `local` in functions.**
Every function-internal variable should be declared `local` unless you intentionally need to set a global.

**5. Use meaningful variable names.**
```bash
input_file="$1"    # not f or x
max_retries=5      # not m or n
```

**6. Write to stderr for errors and diagnostics.**
```bash
echo "Error: file not found" >&2
```

**7. Use `mktemp` for temporary files.**
Never hardcode `/tmp/myfile.txt`. Use `mktemp` and clean up with `trap`.

**8. Validate inputs.**
```bash
if [[ $# -ne 2 ]]; then
    echo "Usage: $0 <source> <dest>" >&2
    exit 1
fi
```

**9. Prefer `$(...)` over backticks.**
Backticks are harder to read and nest poorly.

**10. Use `printf` over `echo` for formatted or portable output.**
`echo` behavior varies across systems (`-e`, `-n` flags are not portable). `printf` is consistent.

**11. Check that external commands exist before using them.**
```bash
if ! command -v jq &> /dev/null; then
    echo "jq is required but not installed" >&2
    exit 1
fi
```

**12. Use `readonly` for constants.**
```bash
readonly MAX_SIZE=1000
readonly CONFIG_FILE="/etc/myapp/config.cfg"
```

**13. Use arrays instead of space-separated strings.**
```bash
# Wrong:
files="a.txt b.txt c.txt"
for f in $files; do ...    # breaks on filenames with spaces

# Right:
files=("a.txt" "b with spaces.txt" "c.txt")
for f in "${files[@]}"; do ...
```

**14. Comment your code.**
```bash
# Retry logic: attempt the command up to MAX_RETRIES times
# with exponential backoff between attempts
for attempt in $(seq 1 $MAX_RETRIES); do
    ...
done
```

**15. Run ShellCheck before committing.**
It catches 90% of common bash mistakes automatically.

---

## 22. Quick Reference Cheat Sheet

### Variables
```bash
var="value"           # assign
echo "$var"           # use (always quote!)
echo "${var}"         # use with braces
unset var             # delete
readonly var="const"  # constant
export var            # export to child processes
local var             # scope to function
```

### String Operations
```bash
${#str}               # length
${str:2:5}            # substring offset 2 length 5
${str/old/new}        # replace first
${str//old/new}       # replace all
${str,,}              # lowercase
${str^^}              # uppercase
${str#prefix}         # strip shortest prefix
${str##prefix}        # strip longest prefix
${str%suffix}         # strip shortest suffix
${str%%suffix}        # strip longest suffix
${var:-default}       # use default if unset
${var:=default}       # assign default if unset
```

### Conditionals
```bash
[[ -z "$s" ]]         # string empty
[[ -n "$s" ]]         # string non-empty
[[ "$a" == "$b" ]]    # string equal
[[ "$a" != "$b" ]]    # string not equal
[[ "$a" =~ regex ]]   # regex match
(( a > b ))           # numeric greater than
(( a == b ))          # numeric equal
[[ -f path ]]         # is regular file
[[ -d path ]]         # is directory
[[ -e path ]]         # exists
[[ -r path ]]         # readable
[[ -x path ]]         # executable
```

### Loops
```bash
for item in list; do ... done
for (( i=0; i<n; i++ )); do ... done
for i in {1..10}; do ... done
while [[ condition ]]; do ... done
until [[ condition ]]; do ... done
while IFS= read -r line; do ... done < file
break / continue
```

### Arrays
```bash
arr=(a b c)           # indexed array
arr[3]="d"            # assign by index
${arr[0]}             # element 0
${arr[@]}             # all elements
${#arr[@]}            # length
${!arr[@]}            # all indices
arr+=("e")            # append
unset arr[2]          # delete element
declare -A map        # associative array
map["key"]="val"      # assign
${map["key"]}         # access
${!map[@]}            # all keys
```

### Functions
```bash
myfunc() {
    local var="$1"
    echo "$var"
}
result=$(myfunc arg)
```

### I/O
```bash
> file                # stdout to file (overwrite)
>> file               # stdout to file (append)
2> file               # stderr to file
&> file               # stdout+stderr to file
< file                # file to stdin
cmd1 | cmd2           # pipe stdout to stdin
<<< "string"          # here-string
<< EOF ... EOF        # here-document
$(command)            # command substitution
<(command)            # process substitution
```

### Arithmetic
```bash
$(( a + b ))          # addition
$(( a - b ))          # subtraction
$(( a * b ))          # multiplication
$(( a / b ))          # integer division
$(( a % b ))          # modulo
$(( a ** b ))         # exponentiation
(( a++ ))             # increment
(( a += 5 ))          # add-assign
echo "scale=2; 5/3" | bc  # float math
```

### Exit Codes & Errors
```bash
exit 0                # success
exit 1                # error
$?                    # last exit code
set -e                # exit on error
set -u                # error on unset variable
set -o pipefail       # fail on pipe error
trap cleanup EXIT     # run on exit
trap handler ERR      # run on error
command || { echo "failed"; exit 1; }
```

### Useful One-Liners
```bash
# Read file line by line:
while IFS= read -r line; do echo "$line"; done < file.txt

# Count lines matching pattern:
grep -c "pattern" file.txt

# Replace in file in-place:
sed -i 's/old/new/g' file.txt

# Sum a column:
awk '{sum+=$1} END{print sum}' file.txt

# Find and delete files older than 30 days:
find /tmp -mtime +30 -type f -delete

# Get filename without extension:
name="${file%.*}"

# Get file extension:
ext="${file##*.}"

# Retry a command 3 times:
for i in 1 2 3; do command && break || sleep 5; done

# Check if running as root:
if (( EUID != 0 )); then echo "Run as root"; exit 1; fi

# Generate a random 16-char hex string:
openssl rand -hex 16

# Watch a command every 2 seconds:
watch -n 2 'df -h'
```