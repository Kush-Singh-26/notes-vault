---
title: "BASH SCRIPTING: THE COMPLETE REFERENCE"
description: "For Intermediate-Advanced Practitioners."
---

## 1. SHELL FUNDAMENTALS & EXECUTION MODEL

### Shebang & Interpreter Selection

```bash
#!/usr/bin/env bash          # Portable — finds bash in PATH
#!/bin/bash                  # Absolute — faster but less portable
#!/usr/bin/env bash -euo pipefail  # Inline options (not universally supported)
```

Always use `env bash` for portability across macOS, Linux, BSD. Never write `#!/bin/sh` when you need bash-specific features — `/bin/sh` may be dash, ash, or BusyBox.

### Shell Options (set -*)

```bash
set -e          # Exit immediately on error (errexit)
set -u          # Treat unset variables as errors (nounset)
set -o pipefail # Pipeline fails if any command fails
set -x          # Print each command before execution (xtrace)
set -v          # Print shell input lines as read (verbose)
set -n          # Read commands but don't execute (syntax check)
set -f          # Disable globbing
set -C          # Prevent > from overwriting existing files
set -T          # Inherit DEBUG/RETURN traps in functions
set -E          # Inherit ERR trap in subshells/functions

# The canonical safe scripting header:
set -euo pipefail

# Turn options off with +:
set +x          # Stop tracing
set +e          # Disable exit-on-error temporarily
```

### shopt — Shell Options (Bash-Specific)

```bash
shopt -s option   # Set
shopt -u option   # Unset
shopt -p          # Print all settings

# Key options:
shopt -s nullglob       # Unmatched globs expand to nothing (not literal)
shopt -s failglob       # Unmatched globs cause an error
shopt -s globstar       # Enable ** recursive globbing
shopt -s extglob        # Enable extended glob patterns
shopt -s dotglob        # Include dotfiles in glob expansion
shopt -s nocaseglob     # Case-insensitive globbing
shopt -s nocasematch    # Case-insensitive [[ ]] and case matching
shopt -s autocd         # Type a directory name to cd into it
shopt -s cdspell        # Correct minor typos in cd
shopt -s checkwinsize   # Update LINES/COLUMNS after each command
shopt -s histappend     # Append to history file, don't overwrite
shopt -s cmdhist        # Save multiline commands as single history entry
shopt -s lithist        # Preserve newlines in multiline commands
shopt -s expand_aliases # Allow aliases in scripts (off by default)
shopt -s lastpipe       # Last command of pipeline runs in current shell
shopt -s inherit_errexit # Subshells inherit -e
```

### How Bash Executes a Script

1. Fork a child process
2. Read the shebang line
3. exec() the interpreter with the script as argument
4. Bash tokenizes input: words, operators, metacharacters
5. Alias expansion (interactive only, unless expand_aliases)
6. Brace expansion
7. Tilde expansion
8. Parameter/variable expansion
9. Command substitution
10. Arithmetic expansion
11. Word splitting (on IFS)
12. Pathname expansion (globbing)
13. Quote removal
14. Execute

Understanding this order is critical — it explains why certain quoting strategies work the way they do.

### Special Characters & Quoting

```bash
# Metacharacters: | & ; ( ) < > space tab newline
# Must be quoted or escaped to be treated literally.

"double quotes"    # Allows $, ``, \, ! (in interactive), but prevents word split/glob
'single quotes'    # Literal everything — no exceptions
$'...'             # ANSI-C quoting: interprets \n, \t, \x41, \u0041, etc.
$"..."             # Locale translation (rarely used)
\                  # Escape next character (outside quotes)

# Examples:
echo $'hello\nworld'     # Outputs two lines
echo $'\x41'             # Outputs: A
echo $'\u2603'           # Outputs: ☃ (Unicode snowman)
```

---

## 2. VARIABLES, SCOPE & DATA TYPES

### Variable Assignment

```bash
var=value           # No spaces around =
var="hello world"
var=$(command)      # Command substitution
var=$((1 + 2))      # Arithmetic expansion
var=${other_var}    # Variable expansion (braces optional but clear)

# Declare / typeset (synonymous):
declare var         # Declare variable
declare -i num      # Integer attribute: arithmetic on assignment
declare -r const    # Readonly
declare -x exported # Export to environment
declare -l lower    # Convert to lowercase on assignment
declare -u upper    # Convert to uppercase on assignment
declare -a arr      # Indexed array
declare -A map      # Associative array
declare -n ref      # Nameref (reference to another variable, Bash 4.3+)
declare -p var      # Print variable declaration (useful for debugging)
declare -f          # Print all function definitions
declare -F          # Print function names only
```

### Variable Scope

```bash
# All variables are global by default.
# 'local' limits scope to the function and its callees.

myfunc() {
    local x=10          # local to this function
    local -i counter=0  # local integer
    global_var=100      # modifies or creates global
}

# Nameref (Bash 4.3+): reference a variable by name
declare -n ref=varname
ref="value"   # Sets varname="value"

# Passing variable names to functions:
set_var() {
    declare -n _target=$1
    _target="$2"
}
set_var myvar "hello"   # myvar is now "hello"
```

### Special Variables

```bash
$0          # Script name (path as invoked)
$1..$9      # Positional parameters
${10}+      # Positional parameters >= 10 need braces
$#          # Number of positional parameters
$@          # All positional parameters as separate words (use in loops)
$*          # All positional parameters as single word (respect IFS)
"$@"        # Each arg as a separate quoted string — PREFERRED
"$*"        # All args joined by first char of IFS
$?          # Exit status of last command
$!          # PID of last background job
$$          # PID of current shell
$-          # Current shell options (like -eux)
$_          # Last argument of last command
$LINENO     # Current line number
$FUNCNAME   # Array of function call stack (index 0 = current)
$BASH_SOURCE # Array of source files (parallel to FUNCNAME)
$BASH_LINENO # Array of line numbers (parallel to FUNCNAME)
$PPID       # PID of parent process
$BASHPID    # PID of current bash process (differs from $$ in subshells)
$BASH_SUBSHELL  # Subshell depth (0 = main shell)
$SHLVL      # Shell nesting level
$BASH_VERSION / $BASH_VERSINFO  # Version info
$SECONDS    # Seconds since shell started
$RANDOM     # Random integer 0-32767 (re-evaluated each access)
$LINENO     # Line number in script
$HISTCMD    # Current command number in history
$REPLY      # Default variable for read
$IFS        # Internal Field Separator (default: space, tab, newline)
$PATH, $HOME, $USER, $SHELL, $TERM  # Standard env vars
$OLDPWD     # Previous working directory (used by cd -)
$DIRSTACK   # Array of dirs from pushd/popd
$PIPESTATUS # Array of exit codes from last pipeline
$BASH_COMMAND # Command currently executing (useful in traps)
$COMP_*     # Completion variables
$READLINE_* # Readline variables
```

---

## 3. PARAMETER EXPANSION (COMPLETE)

This is one of the most powerful and underused features of bash.

### Basic Expansion

```bash
${var}              # Basic expansion (braces avoid ambiguity)
${var:-default}     # Use default if var unset or empty
${var:=default}     # Set and use default if var unset or empty
${var:?message}     # Error and exit if var unset or empty
${var:+alternate}   # Use alternate if var is set and non-empty

# Without colon: only tests for unset (not empty)
${var-default}      # Use default only if var is unset
${var=default}      # Set default only if var is unset
${var?message}      # Error only if var is unset (empty is OK)
${var+alternate}    # Use alternate if var is set (even if empty)
```

### String Length & Substring

```bash
${#var}             # Length of string value
${#array[@]}        # Number of elements in array
${#array[i]}        # Length of element i

${var:offset}       # Substring from offset to end
${var:offset:length}  # Substring of length chars starting at offset
${var: -5}          # Last 5 chars (space before - is REQUIRED)
${var: -5:3}        # 3 chars starting 5 from end

# Positional parameters:
${@:2}              # All args starting from $2
${@:2:3}            # 3 args starting from $2
```

### Pattern Matching in Expansion

```bash
${var#pattern}      # Remove shortest match from beginning
${var##pattern}     # Remove longest match from beginning (greedy)
${var%pattern}      # Remove shortest match from end
${var%%pattern}     # Remove longest match from end (greedy)

# Examples:
file="path/to/file.tar.gz"
${file#*/}          # "to/file.tar.gz"
${file##*/}         # "file.tar.gz"  (basename equivalent)
${file%/*}          # "path/to"      (dirname equivalent)
${file%%.*}         # "path/to/file"
${file#*.}          # "tar.gz"
${file##*.}         # "gz"
```

### Search & Replace

```bash
${var/pattern/replacement}   # Replace first match
${var//pattern/replacement}  # Replace all matches
${var/#pattern/replacement}  # Replace if match at beginning
${var/%pattern/replacement}  # Replace if match at end

# Delete: omit replacement
${var/pattern}               # Delete first match
${var//pattern}              # Delete all matches

# Examples:
str="hello world world"
${str/world/bash}    # "hello bash world"
${str//world/bash}   # "hello bash bash"
${str/#hello/hi}     # "hi world world"
${str/%world/earth}  # "hello world earth"
```

### Case Modification (Bash 4.0+)

```bash
${var^}     # Uppercase first character
${var^^}    # Uppercase all characters
${var,}     # Lowercase first character
${var,,}    # Lowercase all characters
${var~}     # Toggle case of first character
${var~~}    # Toggle case of all characters

# With pattern:
${var^[aeiou]}   # Uppercase first vowel
${var^^[aeiou]}  # Uppercase all vowels
```

### Indirect Expansion

```bash
varname="PATH"
${!varname}      # Expands to value of $PATH — indirect reference

# List variable names matching prefix:
${!prefix*}      # Names matching prefix (as single word)
${!prefix@}      # Names matching prefix (as separate words, preferred)

# Array keys:
${!array[@]}     # All indices/keys of array
```

---

## 4. ARRAYS & ASSOCIATIVE ARRAYS

### Indexed Arrays

```bash
# Declaration and initialization:
arr=()                          # Empty array
arr=(a b c d)                   # Literal
arr=([0]=a [2]=c [5]=e)         # Sparse — indices can be non-contiguous
declare -a arr
arr+=(more elements)            # Append

# Access:
${arr[0]}               # First element
${arr[-1]}              # Last element (Bash 4.3+)
${arr[@]}               # All elements as separate words
${arr[*]}               # All elements as one word (respects IFS)
${#arr[@]}              # Number of elements
${!arr[@]}              # All indices
${arr[@]:2:3}           # Slice: 3 elements starting at index 2

# Modification:
arr[3]="value"          # Set element
unset arr[2]            # Delete element (leaves sparse array!)
arr=("${arr[@]}")       # Reindex to compact (remove gaps)

# Iteration:
for elem in "${arr[@]}"; do
    echo "$elem"
done

# Iterate with indices:
for i in "${!arr[@]}"; do
    echo "$i: ${arr[$i]}"
done

# Append single element:
arr+=("new element")

# Append another array:
arr+=("${other[@]}")

# Copy array:
copy=("${arr[@]}")

# Passing arrays to functions (use nameref or serialization):
print_array() {
    declare -n _arr=$1
    for elem in "${_arr[@]}"; do
        echo "$elem"
    done
}
print_array arr
```

### Associative Arrays (Bash 4.0+)

```bash
declare -A map

# Initialize:
declare -A map=([key1]=val1 [key2]=val2)
map[key3]="val3"

# Access:
${map[key1]}
${!map[@]}          # All keys
${map[@]}           # All values
${#map[@]}          # Number of entries

# Check if key exists:
if [[ -v map[key] ]]; then echo "exists"; fi

# Iterate:
for key in "${!map[@]}"; do
    echo "$key => ${map[$key]}"
done

# Delete entry:
unset map[key1]

# Serialize / deserialize:
declare -p map       # Prints the declare statement — copy-pasteable
```

### Mapfile / Readarray

```bash
# Read lines into array:
mapfile -t lines < file.txt       # Read file into array, strip newlines
mapfile -t lines < <(command)     # From command output
readarray -t lines < file.txt     # Synonym

# Options:
mapfile -t -s 5 -n 10 lines < file.txt  # Skip 5 lines, read 10
mapfile -t -d '' lines < <(find . -print0)  # Null-delimited
mapfile -t -C callback -c 3 lines < file    # Call callback every 3 lines
```

---

## 5. STRING OPERATIONS

### Without External Tools

```bash
str="Hello, World!"

# Length:
${#str}                    # 13

# Substring:
${str:7:5}                 # "World"
${str: -6}                 # "orld!"  — wait, 6 from end = "orld!" is 5...
${str: -6}                 # "orld!" (last 6 chars)

# Replace:
${str/World/Bash}          # "Hello, Bash!"

# Case:
${str,,}                   # "hello, world!"
${str^^}                   # "HELLO, WORLD!"

# Strip prefix/suffix patterns:
path="  hello  "
# Trim whitespace (no external tools):
trimmed="${path#"${path%%[![:space:]]*}"}"
trimmed="${trimmed%"${trimmed##*[![:space:]]}"}"

# Check if contains substring:
if [[ "$str" == *"World"* ]]; then echo "found"; fi

# Check prefix/suffix:
if [[ "$str" == Hello* ]]; then echo "starts with Hello"; fi
if [[ "$str" == *"!" ]];   then echo "ends with !"; fi

# String repetition:
printf '%0.s-' {1..40}    # Print 40 dashes

# Pad string:
printf '%-20s|\n' "left"   # Left-padded to 20
printf '%20s|\n'  "right"  # Right-padded to 20
printf '%020d\n'  42       # Zero-padded integer
```

### With External Tools (When Performance Allows)

```bash
# sed — stream editor:
echo "hello" | sed 's/hello/world/'          # Replace
echo "hello" | sed 's/[aeiou]/_/g'          # Replace all vowels
sed -n '5,10p' file                          # Print lines 5-10
sed '/pattern/d' file                        # Delete matching lines
sed -i 's/old/new/g' file                    # In-place edit
sed -i.bak 's/old/new/g' file               # In-place with backup

# awk — pattern scanning and processing:
awk '{print $2}' file                        # Print second field
awk -F: '{print $1}' /etc/passwd             # Custom delimiter
awk 'NR==5' file                             # Print line 5
awk '/pattern/' file                         # Print matching lines
awk '{sum+=$1} END{print sum}' file          # Sum first column
awk 'NF>3' file                              # Lines with more than 3 fields
awk 'BEGIN{FS=":"; OFS="\t"} {print $1,$3}' /etc/passwd

# grep:
grep -E 'regex' file     # Extended regex
grep -P 'pcre' file      # Perl-compatible regex (if available)
grep -o 'pattern' file   # Print only matching part
grep -c 'pattern' file   # Count matching lines
grep -v 'pattern' file   # Invert match
grep -n 'pattern' file   # Show line numbers
grep -r 'pattern' dir/   # Recursive
grep -l 'pattern' dir/*  # List files with matches only
grep -q 'pattern' file   # Quiet — for use in conditionals

# tr:
tr 'a-z' 'A-Z'           # Translate
tr -d '\n'               # Delete newlines
tr -s ' '                # Squeeze repeated spaces
tr -dc 'a-zA-Z0-9'       # Delete non-alphanumeric

# cut:
cut -d: -f1,3 file       # Fields 1 and 3, colon-delimited
cut -c1-10 file          # Characters 1-10

# paste:
paste file1 file2        # Merge files side by side
paste -d, file1 file2    # Custom delimiter
paste -s file            # Merge lines of file serially
```

---

## 6. ARITHMETIC

### Arithmetic Expansion & Contexts

```bash
# $(( )) — arithmetic expansion (returns result as string):
result=$((2 + 3))
result=$((a * b + c))
result=$(( (a + b) * c ))

# (( )) — arithmetic command (returns 0/1 exit status):
((count++))
((a = b + c))
if ((x > 5)); then echo "big"; fi
while ((i < 10)); do ((i++)); done

# let — older syntax, avoid in modern scripts:
let result=2+3
let "result = 2 + 3"

# declare -i:
declare -i n
n="3 + 4"     # n is now 7 (arithmetic on assignment)
```

### Operators

```bash
# Standard:
+ - * /         # Add, subtract, multiply, divide (integer)
%               # Modulo
**              # Exponentiation (Bash 4.0+)
-n              # Unary negation

# Bitwise:
&  |  ^  ~      # AND, OR, XOR, NOT
<< >>           # Left shift, right shift

# Assignment:
= += -= *= /= %= **= &= |= ^= <<= >>=

# Comparison (inside (( )) ):
== != < > <= >=

# Logical (inside (( )) ):
&&  ||  !

# Increment/decrement:
i++ i-- ++i --i

# Ternary:
result=$(( a > b ? a : b ))

# Comma (evaluate multiple, return last):
$(( a++, b++, a + b ))

# Base conversion:
echo $((16#FF))      # Hex to decimal: 255
echo $((8#77))       # Octal to decimal: 63
echo $((2#1010))     # Binary to decimal: 10
printf '%x\n' 255    # Decimal to hex: ff
printf '%o\n' 255    # Decimal to octal: 377
```

### Floating Point (Requires bc or awk)

```bash
# bc:
echo "scale=4; 22/7" | bc              # 3.1428
echo "sqrt(2)" | bc -l                 # 1.4142135623...
result=$(echo "scale=2; $a + $b" | bc)

# awk:
awk "BEGIN {printf \"%.4f\n\", 22/7}"
result=$(awk "BEGIN {print $a + $b}")

# python3 (when available):
python3 -c "print(f'{22/7:.4f}')"
```

---

## 7. CONTROL FLOW

### if / elif / else

```bash
if condition; then
    ...
elif condition; then
    ...
else
    ...
fi

# Condition types:
if [ expression ]        # POSIX test (requires spaces around [])
if [[ expression ]]      # Bash extended test (preferred)
if command               # True if command exits 0
if (( arithmetic ))      # True if arithmetic != 0
if ( subshell )          # True if subshell exits 0
```

### Test Expressions: [ ] vs [[ ]]

```bash
# File tests (work in both):
-e file     # Exists
-f file     # Regular file
-d file     # Directory
-l file     # Symlink
-r file     # Readable
-w file     # Writable
-x file     # Executable
-s file     # Non-empty (size > 0)
-O file     # Owned by current user
-G file     # Owned by current group
-b file     # Block special
-c file     # Character special
-p file     # Named pipe (FIFO)
-S file     # Socket
-N file     # Modified since last read
-t fd       # File descriptor is open on terminal
f1 -nt f2   # f1 newer than f2 (modification time)
f1 -ot f2   # f1 older than f2
f1 -ef f2   # Same file (hard links)

# String tests:
-z str      # Empty string
-n str      # Non-empty string
str1 = str2   # Equal (in [ ])
str1 == str2  # Equal (in [[ ]])
str1 != str2  # Not equal
str1 < str2   # Lexicographic less (in [ ] must escape: \<)
str1 > str2   # Lexicographic greater
[[ str == pattern ]]  # Glob pattern match (not regex)
[[ str =~ regex ]]    # Regex match (ERE) — ONLY in [[ ]]

# Numeric tests (in [ ] and [[ ]]):
n1 -eq n2   # Equal
n1 -ne n2   # Not equal
n1 -lt n2   # Less than
n1 -le n2   # Less than or equal
n1 -gt n2   # Greater than
n1 -ge n2   # Greater than or equal

# Logical:
# In [ ]:  -a (AND), -o (OR), ! (NOT) — avoid -a/-o, use [[ ]] instead
# In [[ ]]: && || !
# Combined:
[[ -f file && -r file ]]
[[ str == a* || str == b* ]]
[[ ! -d dir ]]
```

### Regex Matching with [[ =~ ]]

```bash
if [[ "$str" =~ ^[0-9]+$ ]]; then
    echo "pure integer"
fi

# Capture groups stored in BASH_REMATCH:
if [[ "2024-01-15" =~ ^([0-9]{4})-([0-9]{2})-([0-9]{2})$ ]]; then
    year="${BASH_REMATCH[1]}"
    month="${BASH_REMATCH[2]}"
    day="${BASH_REMATCH[3]}"
fi

# IMPORTANT: Don't quote the regex pattern — it breaks matching:
re='^[0-9]+$'
[[ "$str" =~ $re ]]    # Correct
[[ "$str" =~ "$re" ]]  # WRONG — treats as literal string
```

### case Statement

```bash
case "$var" in
    pattern1)
        ;;
    pattern2 | pattern3)    # OR
        ;;
    prefix*)                # Glob
        ;;
    [0-9]*)                 # Bracket expression
        ;;
    ?(a|b))                 # Extended glob (if extglob set)
        ;;
    *)                      # Default
        ;;
esac

# Bash 4.0+: fall-through with ;&
case "$var" in
    a)
        echo "a"
        ;&              # Fall through to next pattern
    b)
        echo "b"        # Executed for both a and b
        ;;
    c)
        echo "c"
        ;;&             # Continue checking patterns
    [abc])
        echo "letter"   # Also executed for a, b, or c
        ;;
esac
```

### Loops

```bash
# for (C-style):
for ((i=0; i<10; i++)); do
    echo $i
done

# for-in (word list):
for item in a b c d; do echo $item; done
for file in *.txt; do process "$file"; done
for dir in */; do echo "$dir"; done

# for-in (array):
for elem in "${arr[@]}"; do echo "$elem"; done
for key in "${!map[@]}"; do echo "$key=${map[$key]}"; done

# while:
while condition; do
    ...
done

while read -r line; do
    echo "$line"
done < file.txt

while IFS=: read -r user pass uid gid info home shell; do
    echo "User: $user, Home: $home"
done < /etc/passwd

# until:
until condition; do
    ...
done

# Loop control:
break           # Exit innermost loop
break 2         # Exit 2 levels of loops
continue        # Next iteration of innermost loop
continue 2      # Next iteration of outer loop

# Infinite loop:
while true; do ...; done
for (( ; ; )); do ...; done
```

### select — Interactive Menus

```bash
PS3="Choose an option: "
options=("Option 1" "Option 2" "Quit")
select opt in "${options[@]}"; do
    case $opt in
        "Option 1") echo "You chose 1" ;;
        "Option 2") echo "You chose 2" ;;
        "Quit") break ;;
        *) echo "Invalid choice $REPLY" ;;
    esac
done
```

---

## 8. FUNCTIONS

### Definition Syntax

```bash
# Both syntaxes are equivalent:
function myfunc {
    echo "hello"
}

myfunc() {
    echo "hello"
}

# Prefer the second (POSIX-compatible style).
```

### Arguments, Return Values & Exit Status

```bash
myfunc() {
    # Arguments via $1, $2, ..., $@, $#
    local arg1="$1"
    local arg2="${2:-default}"  # Default value

    # Return value via stdout:
    echo "result"

    # Exit status via return:
    return 0    # Success
    return 1    # Failure (any 1-255)
}

# Capture output:
result=$(myfunc arg1 arg2)

# Check exit status:
if myfunc; then echo "success"; fi

# Functions see and modify global variables unless local:
counter=0
increment() {
    ((counter++))
}
```

### Returning Complex Data

```bash
# Via nameref (Bash 4.3+):
get_array() {
    declare -n _result=$1
    _result=(a b c d)
}
get_array myarr
echo "${myarr[@]}"   # a b c d

# Via global variable:
_RESULT=""
get_value() {
    _RESULT="computed_value"
}
get_value
echo "$_RESULT"

# Via stdout + eval (avoid if possible):
get_pair() {
    echo "key=value"
}
eval "$(get_pair)"
echo "$key"   # value
```

### Recursion

```bash
factorial() {
    local n=$1
    if ((n <= 1)); then
        echo 1
        return
    fi
    local sub
    sub=$(factorial $((n - 1)))
    echo $((n * sub))
}

# WARNING: each recursive call is a subshell if using $().
# For deep recursion, use a global/nameref approach:
_fact_result=0
factorial_fast() {
    local n=$1
    if ((n <= 1)); then
        _fact_result=1
        return
    fi
    factorial_fast $((n - 1))
    ((_fact_result *= n))
}
```

### Function Libraries & Sourcing

```bash
# Source a library:
source ./lib/utils.sh
. ./lib/utils.sh        # Dot command — POSIX compatible

# Guard against double-sourcing:
[[ -n "${_MY_LIB_LOADED:-}" ]] && return
_MY_LIB_LOADED=1

# Find script directory (robust):
SCRIPT_DIR="$(cd "$(dirname "${BASH_SOURCE[0]}")" && pwd)"
source "$SCRIPT_DIR/lib/utils.sh"
```

---

## 9. I/O, REDIRECTION & FILE DESCRIPTORS

### Standard File Descriptors

```
0 = stdin
1 = stdout
2 = stderr
3+ = user-defined
```

### Redirection Operators

```bash
cmd > file          # Redirect stdout to file (overwrite)
cmd >> file         # Redirect stdout to file (append)
cmd < file          # Redirect stdin from file
cmd 2> file         # Redirect stderr to file
cmd 2>> file        # Redirect stderr to file (append)
cmd &> file         # Redirect both stdout+stderr (Bash syntax)
cmd > file 2>&1     # Same, POSIX-compatible (order matters!)
cmd 2>&1 > file     # WRONG: stderr goes to old stdout, stdout to file
cmd > file 2>&1     # CORRECT: both go to file

cmd 1>/dev/null 2>&1   # Suppress all output
cmd &>/dev/null         # Same, bash syntax

cmd < /dev/null         # Prevent reading from terminal
cmd > /dev/null 2>&1 < /dev/null  # Fully detached I/O

# Redirect to stderr explicitly:
echo "error message" >&2

# Append both to file:
cmd >> file 2>&1
```

### Custom File Descriptors

```bash
# Open FD for reading:
exec 3< file.txt
read -r line <&3
exec 3<&-           # Close FD 3

# Open FD for writing:
exec 4> output.txt
echo "data" >&4
exec 4>&-           # Close FD 4

# Open FD for reading and writing:
exec 5<> file.txt

# Duplicate FDs:
exec 6>&1           # Save stdout
exec 1> log.txt     # Redirect stdout to log
echo "this goes to log"
exec 1>&6           # Restore stdout
exec 6>&-           # Close saved FD

# Useful pattern: log to file and console simultaneously:
exec > >(tee -a logfile.txt)
exec 2>&1
```

### read Builtin

```bash
read var                        # Read line into var
read -r var                     # Raw: don't interpret backslashes (almost always use -r)
read -p "Prompt: " var          # With prompt
read -s var                     # Silent (for passwords)
read -t 5 var                   # Timeout in seconds
read -n 3 var                   # Read exactly 3 characters
read -N 3 var                   # Read exactly 3 chars (no newline shortcut)
read -d '' var                  # Read until null byte
read -d ':' var                 # Read until delimiter
read -a arr                     # Read words into array
read -u 3 var                   # Read from FD 3

# Read multiple fields:
IFS=: read -r user pass uid gid info home shell <<< "$line"
IFS=' ' read -r -a words <<< "$sentence"

# Read with default:
read -r -p "Name [$default]: " name
name="${name:-$default}"

# Read password securely:
read -s -r -p "Password: " password
echo                            # Newline after silent input
```

### printf

```bash
# Preferred over echo for portability and control:
printf 'format' args

printf '%s\n' "string"          # String
printf '%d\n' 42                # Decimal integer
printf '%05d\n' 42              # Zero-padded: 00042
printf '%-10s|\n' "left"        # Left-aligned in 10 chars
printf '%10s|\n' "right"        # Right-aligned in 10 chars
printf '%f\n' 3.14              # Float
printf '%.2f\n' 3.14159         # 2 decimal places
printf '%e\n' 12345.678         # Scientific notation
printf '%x\n' 255               # Hex (lowercase)
printf '%X\n' 255               # Hex (uppercase)
printf '%o\n' 255               # Octal
printf '%b\n' "hello\nworld"    # Interpret backslash escapes
printf '%q\n' "hello world"     # Shell-quoted for re-use

# Multiple args cycle through format:
printf '%s\n' a b c d           # Prints each on own line

# Redirect to variable (printf -v):
printf -v var '%05d' 42         # var="00042" (no subshell)
printf -v timestamp '%(%Y-%m-%d)T' -1   # Current date (Bash 4.2+)
```

---

## 10. PIPELINES & PROCESS SUBSTITUTION

### Pipelines

```bash
cmd1 | cmd2 | cmd3      # Pipe stdout of each to stdin of next

# Exit status of pipeline = exit status of last command
# With pipefail:
set -o pipefail
# Exit status = first failing command (or 0 if all succeed)

# Check individual pipe statuses:
cmd1 | cmd2 | cmd3
echo "${PIPESTATUS[@]}"     # Array of exit codes: e.g. "0 1 0"
echo "${PIPESTATUS[1]}"     # Exit code of cmd2

# Avoid subshell for last command with lastpipe:
shopt -s lastpipe
var=0
some_command | while read line; do ((var++)); done
echo $var   # Works! Without lastpipe, var would be 0 (subshell)
```

### Process Substitution

```bash
# <(cmd) — substitute command output as if it were a file:
diff <(sort file1) <(sort file2)
comm <(sort list1) <(sort list2)
while read line; do ...; done < <(command)

# >(cmd) — substitute command input as if it were a file:
tee >(gzip > output.gz) >(wc -l) > /dev/null

# These create named pipes (FIFOs) under /dev/fd/
# The substitution expression evaluates to a file path

# Critical advantage over pipes: no subshell for the outer process.
# Variables modified inside while+process-sub are visible outside:
count=0
while read line; do
    ((count++))
done < <(generate_lines)
echo $count    # Correct value
```

### Command Substitution

```bash
var=$(command)          # Preferred syntax
var=`command`           # Old syntax (avoid — hard to nest, confusing)

# Nesting:
result=$(outer $(inner arg))

# Preserve newlines — quote the substitution:
lines=$(cat file)       # Trailing newlines stripped by bash
echo "$lines"           # Quoted: preserves internal newlines

# Subshell: variables set inside don't affect parent
var=0
result=$(var=99; echo $var)    # result=99, but outer var still 0
```

---

## 11. JOB CONTROL & SIGNALS

### Job Control

```bash
cmd &               # Run in background
jobs                # List background jobs
jobs -l             # With PIDs
fg %1               # Bring job 1 to foreground
bg %1               # Send stopped job 1 to background
kill %1             # Kill job 1
wait                # Wait for all background jobs
wait $!             # Wait for last background job
wait -n             # Wait for any job to finish (Bash 5.1+)
wait $pid           # Wait for specific PID

# Disown — remove job from job table (survives shell exit):
cmd &
disown %1           # or: disown $!

# nohup — immune to hangup:
nohup cmd &
```

### Signals

```bash
# Common signals:
SIGHUP  (1)    # Terminal hangup
SIGINT  (2)    # Interrupt (Ctrl+C)
SIGQUIT (3)    # Quit (Ctrl+\)
SIGKILL (9)    # Kill (cannot be caught or ignored)
SIGTERM (15)   # Terminate (can be caught)
SIGSTOP (17)   # Stop (cannot be caught or ignored)
SIGTSTP (18)   # Terminal stop (Ctrl+Z)
SIGUSR1 (10)   # User-defined
SIGUSR2 (12)   # User-defined
SIGCHLD (17)   # Child status changed
SIGPIPE (13)   # Broken pipe
SIGALRM (14)   # Alarm clock
SIGWINCH(28)   # Window resize

kill -SIGNAL pid    # Send signal to PID
kill -l             # List all signals
```

### trap — Signal Handling

```bash
# Syntax: trap 'commands' SIGNALS
trap 'echo "Interrupted"' INT
trap 'cleanup_function' EXIT INT TERM
trap '' SIGPIPE              # Ignore SIGPIPE
trap - INT                   # Reset INT to default

# Pseudo-signals:
trap 'command' EXIT          # Runs on shell exit (any cause)
trap 'command' ERR           # Runs when command returns non-zero
trap 'command' DEBUG         # Runs before every command
trap 'command' RETURN        # Runs when function returns

# Robust cleanup pattern:
cleanup() {
    local exit_code=$?
    rm -f "$tmpfile"
    # ... other cleanup
    exit $exit_code
}
trap cleanup EXIT

# Temporarily disable trap:
trap - EXIT
critical_operation
trap cleanup EXIT

# Get current trap value:
current_trap=$(trap -p EXIT)

# Save and restore traps:
old_trap=$(trap -p INT)
trap 'custom' INT
# ... do work ...
eval "$old_trap"

# ERR trap with function name and line:
error_handler() {
    echo "Error in ${BASH_SOURCE[0]} line ${BASH_LINENO[0]}: ${BASH_COMMAND}"
}
trap error_handler ERR
```

---

## 12. PATTERN MATCHING & REGULAR EXPRESSIONS

### Glob Patterns

```bash
*       # Match any string (including empty)
?       # Match any single character
[abc]   # Match a, b, or c
[a-z]   # Match any lowercase letter
[!abc]  # Match any char except a, b, c
[^abc]  # Same as [!abc]

# With globstar enabled:
**      # Match any path including subdirectories
```

### Extended Globs (shopt -s extglob)

```bash
?(pattern)   # Zero or one occurrence
*(pattern)   # Zero or more occurrences
+(pattern)   # One or more occurrences
@(pattern)   # Exactly one occurrence (like grouping)
!(pattern)   # Anything that doesn't match pattern

# Examples:
ls *.@(jpg|png|gif)          # Files ending in jpg, png, or gif
ls !(*.txt)                  # All files except .txt
rm !(important.txt)          # Delete all but important.txt
[[ "$file" == *.+(tar.)@(gz|bz2|xz) ]]  # Match archives

# Extended globs work in:
# - case patterns
# - [[ == ]] comparisons
# - filename expansion
```

### Regular Expressions (ERE in [[ =~ ]])

```bash
# Anchors:
^       # Start of string
$       # End of string
\b      # Word boundary (limited support)

# Character classes:
.       # Any character
[abc]   # Character class
[^abc]  # Negated class
[a-z]   # Range
[:alpha:], [:digit:], [:space:], [:upper:], [:lower:], [:alnum:]

# Quantifiers:
*       # Zero or more
+       # One or more
?       # Zero or one
{n}     # Exactly n
{n,}    # n or more
{n,m}   # Between n and m

# Grouping:
(...)   # Capture group → stored in BASH_REMATCH
(?:...) # Non-capturing group (NOT supported in bash ERE)

# Alternation:
a|b     # a or b

# Examples:
[[ "$email" =~ ^[a-zA-Z0-9._%+\-]+@[a-zA-Z0-9.\-]+\.[a-zA-Z]{2,}$ ]]
[[ "$ip" =~ ^([0-9]{1,3}\.){3}[0-9]{1,3}$ ]]
[[ "$date" =~ ^[0-9]{4}-[0-9]{2}-[0-9]{2}$ ]]
[[ "$phone" =~ ^\+?[0-9]{10,15}$ ]]

# Access capture groups:
if [[ "foo123bar" =~ ([a-z]+)([0-9]+)([a-z]+) ]]; then
    echo "${BASH_REMATCH[0]}"  # Full match: foo123bar
    echo "${BASH_REMATCH[1]}"  # Group 1: foo
    echo "${BASH_REMATCH[2]}"  # Group 2: 123
    echo "${BASH_REMATCH[3]}"  # Group 3: bar
fi
```

---

## 13. HERE DOCUMENTS & HERE STRINGS

### Here Documents

```bash
# Basic heredoc:
cat << 'EOF'
Literal text — no expansion
$var is not expanded
EOF

# Expanding heredoc (no quotes):
cat << EOF
Hello $USER, today is $(date)
EOF

# Indented heredoc (Bash — strip leading tabs with <<-):
if true; then
    cat <<- EOF
        Indented content
        Tab-stripped (only leading tabs, not spaces)
    EOF
fi

# Heredoc to variable:
read -r -d '' content << 'EOF'
Multi-line
content here
EOF

# Heredoc to file:
cat > /etc/config << 'EOF'
key=value
other=data
EOF

# Heredoc into command:
mysql -u user -p database << 'EOF'
SELECT * FROM table;
UPDATE table SET col=val;
EOF

# Heredoc with pipe:
grep "pattern" << 'EOF' | sort
line c
line a
line b
EOF

# Heredoc to FD:
exec 3<< 'EOF'
some data
EOF
read -r line <&3
```

### Here Strings

```bash
# <<< — single string as stdin (no subprocess):
read -r var <<< "hello world"
grep "pattern" <<< "$var"
awk '{print $2}' <<< "hello world"    # Prints: world
base64 <<< "encode this"

# More efficient than echo | cmd:
# echo "text" | grep "pattern"   — creates subshell
# grep "pattern" <<< "text"      — no subshell
```

---

## 14. SUBSHELLS & COMMAND GROUPING

### Subshells

```bash
# ( ) — runs in subshell (child process):
(cd /tmp && do_something)   # cd doesn't affect parent
(set +e; risky_cmd)         # set doesn't affect parent

# Variables set in subshell don't affect parent:
x=1
(x=2; echo "inner: $x")     # inner: 2
echo "outer: $x"             # outer: 1

# Subshell exit status:
(exit 42)
echo $?    # 42

# Subshell with return value:
result=$(  
    do_stuff
    echo "$output"
)
```

### Command Grouping

```bash
# { } — runs in current shell (no subshell):
{
    echo "first"
    echo "second"
} > output.txt              # Redirect group output

# Semicolons/newlines required inside { }:
{ cmd1; cmd2; }
{
    cmd1
    cmd2
}

# Use case: capture multiple commands as one unit:
{ produce_data; produce_more; } | consumer_command

# vs subshell:
# ( ) → new process, cleaner isolation but slower
# { } → same process, faster, variable changes persist
```

### Coprocess (Bash 4.0+)

```bash
# Bidirectional pipe to a background process:
coproc NAME { command; }

# FDs automatically created:
# NAME[0] — read end (stdout of coproc)
# NAME[1] — write end (stdin of coproc)

coproc BC { bc -l; }
echo "scale=4; 22/7" >&"${BC[1]}"
read result <&"${BC[0]}"
echo $result    # 3.1428...
```

---

## 15. PROCESS MANAGEMENT

### Fork & Exec

```bash
# Every external command forks a child process.
# Builtins run in the current process:
# Builtins: echo, read, printf, test, [, [[, let, local, declare,
#           export, source, ., cd, pwd, exit, return, set, shopt,
#           trap, wait, jobs, fg, bg, kill, type, command, builtin,
#           eval, exec, getopts, mapfile, readarray, umask, ulimit...

# exec — replace current process (no fork):
exec /path/to/program       # Replaces this shell entirely
exec command args           # No return possible
exec > logfile              # Redirect this shell's stdout permanently
exec 2>&1                   # Redirect this shell's stderr permanently
exec 3< file                # Open FD 3 for reading
```

### Parallelism Patterns

```bash
# Simple parallel execution:
for item in "${items[@]}"; do
    process "$item" &
done
wait    # Wait for all

# With exit code tracking:
pids=()
for item in "${items[@]}"; do
    process "$item" &
    pids+=($!)
done
for pid in "${pids[@]}"; do
    wait "$pid" || echo "Job $pid failed"
done

# Limit parallelism (worker pool):
max_jobs=4
job_count=0
for item in "${items[@]}"; do
    process "$item" &
    ((++job_count))
    if ((job_count >= max_jobs)); then
        wait -n 2>/dev/null || wait    # Wait for any job (Bash 5.1+)
        ((--job_count))
    fi
done
wait

# Semaphore pattern with FIFOs:
mkfifo /tmp/semaphore
exec 9<>/tmp/semaphore
for _ in $(seq $max_jobs); do echo >&9; done

for item in "${items[@]}"; do
    read -u 9   # Acquire slot
    { process "$item"; echo >&9; } &   # Release on completion
done
wait
exec 9>&-
rm /tmp/semaphore
```

### Temporary Files & Directories

```bash
# mktemp:
tmpfile=$(mktemp)                     # /tmp/tmp.XXXXXXXXXX
tmpfile=$(mktemp /tmp/myscript.XXXXX) # Custom prefix/location
tmpdir=$(mktemp -d)                   # Temporary directory
tmpfile=$(mktemp --suffix=.log)       # With suffix

# Always clean up:
cleanup() {
    rm -f "$tmpfile"
    rm -rf "$tmpdir"
}
trap cleanup EXIT
```

---

## 16. ERROR HANDLING & DEFENSIVE PROGRAMMING

### Exit Codes

```bash
# Convention:
# 0    = success
# 1    = general error
# 2    = misuse of shell builtins
# 126  = command not executable
# 127  = command not found
# 128  = invalid exit argument
# 128+n = fatal signal n (e.g. 130 = Ctrl+C = 128+2)
# 255  = out of range

exit 0      # Success
exit 1      # General failure

# Check last command:
if ! command; then handle_error; fi
command || { echo "failed" >&2; exit 1; }
command || die "command failed"

# Die function pattern:
die() {
    echo "${BASH_SOURCE[1]}: line ${BASH_LINENO[0]}: ERROR: $*" >&2
    exit 1
}
```

### Comprehensive Error Handling

```bash
#!/usr/bin/env bash
set -euo pipefail
IFS=$'\n\t'        # Safer IFS (split on newlines and tabs, not spaces)

# Error handler:
error_handler() {
    local exit_code=$?
    local line_number=${BASH_LINENO[0]}
    local command="${BASH_COMMAND}"
    echo "ERROR: Command '$command' failed with exit code $exit_code at line $line_number" >&2
    exit "$exit_code"
}
trap error_handler ERR

# Or a full stack trace:
error_with_trace() {
    local i=0
    echo "ERROR: ${BASH_COMMAND} (exit $?)" >&2
    echo "Stack trace:" >&2
    while caller $i; do
        ((i++))
    done | awk '{print "  Line " $1 " in " $3 ":" $2}' >&2
    exit 1
}
trap error_with_trace ERR

# Ignore errors for specific commands:
set +e
possibly_failing_command
local_exit=$?
set -e

# Or:
possibly_failing_command || true   # Absorb failure
possibly_failing_command || :      # Same (:  is always true)

# Propagate errors through pipelines:
set -o pipefail
```

### Input Validation

```bash
# Check argument count:
[[ $# -lt 2 ]] && { echo "Usage: $0 <arg1> <arg2>" >&2; exit 1; }

# Validate integer:
is_integer() {
    [[ "$1" =~ ^-?[0-9]+$ ]]
}

# Validate non-empty:
[[ -z "${1:-}" ]] && die "Argument required"

# Validate file exists:
[[ -f "$file" ]] || die "File not found: $file"

# Validate command exists:
command -v required_tool &>/dev/null || die "required_tool not found"

# Validate env var set:
: "${REQUIRED_VAR:?REQUIRED_VAR must be set}"
```

### Defensive Patterns

```bash
# Avoid word splitting and globbing issues — ALWAYS QUOTE:
process "$file"             # Not: process $file
for f in "${arr[@]}"; do    # Not: for f in ${arr[@]}
rm -f "$tmpfile"            # Not: rm -f $tmpfile

# Avoid ambiguous options (use --):
rm -- "$file"               # Handles files starting with -
grep -- "$pattern" file

# Use read -r to avoid backslash interpretation:
while IFS= read -r line; do    # IFS= preserves leading whitespace
    process "$line"
done < file

# Avoid ls in scripts — use glob instead:
for file in /path/*.txt; do    # Not: for file in $(ls /path/*.txt)
    [[ -f "$file" ]] || continue   # Handle empty glob
    process "$file"
done

# Use nullglob to handle empty globs:
shopt -s nullglob
files=(*.txt)
[[ ${#files[@]} -eq 0 ]] && echo "No files found"

# Prevent accidental variable expansion in printf:
printf '%s\n' "$user_input"    # Not: printf "$user_input\n"

# Check for numeric context:
declare -i n
n="${user_input}" 2>/dev/null || die "Not an integer"
```

---

## 17. DEBUGGING TECHNIQUES

### Tracing & Logging

```bash
# Enable tracing:
set -x                          # Trace to stderr
set -xv                         # Trace + show input lines

# Custom trace output (PS4):
export PS4='+(${BASH_SOURCE}:${LINENO}): ${FUNCNAME[0]:+${FUNCNAME[0]}(): }'
set -x

# Redirect trace to file (not stderr):
exec {BASH_XTRACEFD}>trace.log
set -x    # Now trace goes to FD instead of stderr

# Selective tracing:
{
    set -x
    # code to trace
    set +x
} 2> trace.log

# Debug logging function:
debug() {
    [[ "${DEBUG:-}" == "1" ]] || return 0
    echo "[DEBUG ${BASH_SOURCE[1]}:${BASH_LINENO[0]}] $*" >&2
}
```

### Inspection Tools

```bash
# Print variable declaration:
declare -p varname

# Print all function definitions:
declare -f

# Print specific function:
declare -f function_name

# Check type of command:
type -a command_name    # Shows all definitions (alias, function, builtin, file)
type -t command_name    # Returns: alias, function, builtin, file, or keyword
which command_name      # Full path of executable

# Check exit codes in a pipeline:
cmd1 | cmd2 | cmd3
echo "${PIPESTATUS[@]}"

# Syntax check without execution:
bash -n script.sh

# Run with debugging:
bash -x script.sh
bash -xv script.sh

# Shellcheck (external tool — use always):
shellcheck script.sh
```

### Caller & Stack

```bash
# Print call stack in a function:
print_stack() {
    local i=0
    while caller $i; do
        ((i++))
    done
}
# Output: line_num subroutine_name file_name (per frame)

# Get caller info:
calling_function="${FUNCNAME[1]}"
calling_line="${BASH_LINENO[0]}"
calling_file="${BASH_SOURCE[1]}"
```

---

## 18. PERFORMANCE & OPTIMIZATION

### Profiling

```bash
# Measure execution time:
time command
time { block of commands; }

# Finer granularity:
start=$SECONDS
# or:
start=$(date +%s%N)    # Nanoseconds (if supported)
# ... work ...
end=$(date +%s%N)
elapsed=$(( (end - start) / 1000000 ))   # Milliseconds

# Profile every line:
PS4='+ $(date "+%s.%N")\011 '
exec 3>&2 2>/tmp/bash.profile
set -x
# ... script ...
set +x
exec 2>&3 3>&-
```

### Key Performance Principles

```bash
# 1. Avoid subshells — they fork a process:
# Slow:
for i in $(seq 1 1000); do ...  # 1000 subshells
# Fast:
for ((i=1; i<=1000; i++)); do ... # No subshell

# 2. Avoid external commands for string operations:
# Slow:
len=$(echo -n "$str" | wc -c)
# Fast:
len=${#str}

# 3. Use printf -v instead of $():
# Slow:
padded=$(printf '%05d' $n)
# Fast:
printf -v padded '%05d' $n

# 4. Read files efficiently:
# Slow: cat file | while read ...
# Fast: while read ... done < file

# 5. Avoid unnecessary pipes:
# Slow: cat file | grep pattern | awk '{print $1}'
# Fast: awk '/pattern/{print $1}' file

# 6. Use [[ ]] over [ ] (no fork):
# [ is an external command on some systems; [[ is a keyword

# 7. Batch external tool calls:
# Slow: call sed/awk once per item in a loop
# Fast: call sed/awk once on all items together

# 8. Prefer arithmetic builtins:
((i++))         # Faster than: i=$(( i + 1 ))
let i++         # Also fast

# 9. Use local to prevent pollution and improve lookup:
local var=value  # Reduces variable lookup scope

# 10. Avoid unnecessary forks in loops:
names=()
while IFS= read -r line; do
    names+=("$line")    # No fork
done < <(find . -name "*.txt")

# 11. Use mapfile for bulk reads:
mapfile -t lines < file    # Much faster than loop+read for large files
```

---

## 19. SECURITY HARDENING

### Critical Practices

```bash
# 1. ALWAYS quote variables — prevents word splitting and glob injection:
rm -rf "$user_input"       # If unquoted: rm -rf * could happen

# 2. Validate all external input:
sanitize() {
    # Remove dangerous characters
    local str="$1"
    str="${str//[^a-zA-Z0-9_.-]/}"
    echo "$str"
}

# 3. Avoid eval — it executes arbitrary code:
# Never: eval "command $user_input"
# Use arrays instead:
args=("$@")
command "${args[@]}"

# 4. Use -- to separate options from arguments:
rm -- "$file"
grep -- "$pattern" "$file"

# 5. Don't use backticks with user input (use $() carefully):
# Still avoid eval regardless of syntax

# 6. Secure temporary files:
tmpfile=$(mktemp) || exit 1    # mktemp is atomic and secure
# Never: tmpfile="/tmp/myapp.$PID"  (predictable, race condition)

# 7. Set restrictive umask for sensitive files:
umask 077
create_config_file    # Created with 600 permissions

# 8. Don't store secrets in variables that appear in /proc:
# ps aux can show process env — use files or stdin instead
# Pass secrets via FD:
read -rs -u 3 secret 3< <(get_secret)

# 9. Restrict PATH:
export PATH="/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin"

# 10. Check script is not world-writable:
[[ "$(stat -c %a "$0")" =~ ^[0-7][0-6][0-4]$ ]] || die "Script is too permissive"

# 11. Avoid globbing in critical contexts:
set -f    # Disable globbing
critical_command $potentially_wild_input
set +f

# 12. Readonly critical variables:
declare -r CONFIG_DIR="/etc/myapp"
declare -r SCRIPT_VERSION="1.2.3"
```

### Injection Prevention

```bash
# SQL injection in bash (if calling mysql):
# BAD:
mysql -e "SELECT * FROM users WHERE name='$user'"
# GOOD:
mysql --execute="SELECT * FROM users WHERE name='$(printf '%s' "$user" | sed "s/'/\\\\'/g")'"
# BETTER: use prepared statements or parameterized queries from a proper language

# Command injection prevention:
# BAD:
eval "echo $user_controlled"
bash -c "$user_controlled"
# GOOD: never run user input as code

# Filename injection:
# BAD: for f in $(ls $dir)
# GOOD: 
for f in "$dir"/; do [[ -d "$f" ]] && ...; done

# Path traversal:
user_file="${user_input##*/}"    # Strip directory components
# Or validate:
[[ "$user_file" =~ ^[a-zA-Z0-9_.-]+$ ]] || die "Invalid filename"
```

---

## 20. ADVANCED PATTERNS & IDIOMS

### Option Parsing with getopts

```bash
usage() {
    cat << EOF
Usage: $0 [-v] [-o output] [-n number] <input>
  -v          Verbose mode
  -o output   Output file
  -n number   Number of iterations
EOF
    exit 1
}

verbose=0
output=""
iterations=1

while getopts ":vo:n:h" opt; do
    case $opt in
        v) verbose=1 ;;
        o) output="$OPTARG" ;;
        n) iterations="$OPTARG"
           [[ "$iterations" =~ ^[0-9]+$ ]] || { echo "Invalid number"; exit 1; }
           ;;
        h) usage ;;
        :) echo "Option -$OPTARG requires an argument" >&2; exit 1 ;;
        \?) echo "Unknown option: -$OPTARG" >&2; exit 1 ;;
    esac
done
shift $((OPTIND - 1))   # Remove parsed options, leave positional args
# Now $1, $2, etc. are the remaining arguments
```

### Long Options (getopt — external tool)

```bash
OPTS=$(getopt -o vo:n:h --long verbose,output:,number:,help -n "$0" -- "$@")
[[ $? -eq 0 ]] || { usage; exit 1; }
eval set -- "$OPTS"

while true; do
    case "$1" in
        -v|--verbose) verbose=1; shift ;;
        -o|--output) output="$2"; shift 2 ;;
        -n|--number) iterations="$2"; shift 2 ;;
        -h|--help) usage ;;
        --) shift; break ;;
        *) break ;;
    esac
done
```

### Logging Framework

```bash
# Log levels:
readonly LOG_LEVEL_DEBUG=0
readonly LOG_LEVEL_INFO=1
readonly LOG_LEVEL_WARN=2
readonly LOG_LEVEL_ERROR=3
readonly LOG_LEVEL_FATAL=4

LOG_LEVEL=${LOG_LEVEL:-$LOG_LEVEL_INFO}
LOG_FILE=${LOG_FILE:-""}

log() {
    local level=$1; shift
    local level_name=$1; shift
    local message="$*"
    local timestamp
    printf -v timestamp '%(%Y-%m-%d %H:%M:%S)T' -1    # Bash 4.2+
    local caller_info="${BASH_SOURCE[2]##*/}:${BASH_LINENO[1]}"
    local line="[$timestamp] [$level_name] [$caller_info] $message"
    
    [[ $level -ge $LOG_LEVEL ]] || return 0
    
    if [[ $level -ge $LOG_LEVEL_ERROR ]]; then
        echo "$line" >&2
    else
        echo "$line"
    fi
    
    [[ -n "$LOG_FILE" ]] && echo "$line" >> "$LOG_FILE"
}

debug() { log $LOG_LEVEL_DEBUG "DEBUG" "$@"; }
info()  { log $LOG_LEVEL_INFO  "INFO " "$@"; }
warn()  { log $LOG_LEVEL_WARN  "WARN " "$@"; }
error() { log $LOG_LEVEL_ERROR "ERROR" "$@"; }
fatal() { log $LOG_LEVEL_FATAL "FATAL" "$@"; exit 1; }
```

### Config File Parsing

```bash
# INI-style config:
parse_config() {
    local config_file="$1"
    local section=""
    
    while IFS= read -r line || [[ -n "$line" ]]; do
        line="${line%%#*}"          # Strip comments
        line="${line%"${line##*[! ]}"}"  # Strip trailing whitespace
        
        [[ -z "$line" ]] && continue
        
        if [[ "$line" =~ ^\[(.+)\]$ ]]; then
            section="${BASH_REMATCH[1]}"
        elif [[ "$line" =~ ^([^=]+)=(.*)$ ]]; then
            local key="${BASH_REMATCH[1]}"
            local value="${BASH_REMATCH[2]}"
            key="${key%"${key##*[! ]}"}"    # Trim key
            value="${value#"${value%%[! ]*}"}"  # Trim value
            
            if [[ -n "$section" ]]; then
                declare -g "${section}_${key}=${value}"
            else
                declare -g "${key}=${value}"
            fi
        fi
    done < "$config_file"
}
```

### Lock Files (Mutual Exclusion)

```bash
LOCKFILE="/var/run/myscript.lock"

acquire_lock() {
    # Atomic test-and-set using noclobber:
    if (set -C; echo "$$" > "$LOCKFILE") 2>/dev/null; then
        trap 'rm -f "$LOCKFILE"' EXIT
        return 0
    fi
    
    # Check if the holding process still exists:
    local pid
    pid=$(cat "$LOCKFILE" 2>/dev/null)
    if [[ -n "$pid" ]] && ! kill -0 "$pid" 2>/dev/null; then
        rm -f "$LOCKFILE"
        acquire_lock   # Retry
    else
        return 1
    fi
}

acquire_lock || die "Another instance is running (PID: $(cat $LOCKFILE))"
```

### fd-Based Mutex (More Robust)

```bash
exec {LOCK_FD}>/var/run/myscript.lock
flock -n "$LOCK_FD" || die "Already running"
trap 'flock -u "$LOCK_FD"' EXIT
```

### Progress Indicators

```bash
# Spinner:
spinner() {
    local pid=$1
    local delay=0.1
    local spinstr='|/-\\'
    while kill -0 "$pid" 2>/dev/null; do
        local temp=${spinstr#?}
        printf ' [%c]  ' "$spinstr"
        spinstr=$temp${spinstr%"$temp"}
        sleep $delay
        printf '\b\b\b\b\b\b'
    done
    printf '    \b\b\b\b'
}

long_command &
spinner $!
wait $!

# Progress bar:
progress_bar() {
    local current=$1 total=$2 width=${3:-50}
    local pct=$(( current * 100 / total ))
    local filled=$(( current * width / total ))
    local empty=$(( width - filled ))
    printf '\r[%s%s] %3d%%' \
        "$(printf '#%.0s' $(seq 1 $filled))" \
        "$(printf ' %.0s' $(seq 1 $empty))" \
        "$pct"
}
```

### Retry Logic

```bash
retry() {
    local max_attempts=$1
    local delay=$2
    shift 2
    local attempt=1
    
    while true; do
        if "$@"; then
            return 0
        fi
        
        if ((attempt >= max_attempts)); then
            echo "Failed after $max_attempts attempts" >&2
            return 1
        fi
        
        echo "Attempt $attempt failed, retrying in ${delay}s..." >&2
        sleep "$delay"
        ((attempt++))
        delay=$((delay * 2))    # Exponential backoff
    done
}

retry 5 2 curl -f "https://example.com/api"
```

### Version Comparison

```bash
version_gt() {
    test "$(printf '%s\n' "$1" "$2" | sort -V | head -1)" != "$1"
}

version_ge() {
    test "$(printf '%s\n' "$1" "$2" | sort -V | head -1)" = "$2"
}

if version_gt "2.1.0" "1.9.5"; then echo "newer"; fi
if version_ge "$(bash --version | head -1 | grep -oP '\d+\.\d+')" "4.4"; then
    echo "Bash >= 4.4 features available"
fi
```

### Templating

```bash
# Simple variable substitution:
render_template() {
    local template="$1"
    eval "echo \"$(cat "$template")\""    # Caution: eval; only safe with trusted templates
}

# Safer: sed-based:
render_template_safe() {
    local template="$1"
    local output="$2"
    sed \
        -e "s|{{NAME}}|$name|g" \
        -e "s|{{VERSION}}|$version|g" \
        -e "s|{{DATE}}|$(date +%Y-%m-%d)|g" \
        "$template" > "$output"
}

# envsubst (from gettext):
export NAME VERSION DATE
envsubst < template.txt > output.txt
envsubst '${NAME} ${VERSION}' < template.txt > output.txt  # Only substitute specified
```

### Parallel Processing with Results

```bash
# Process files in parallel, collect results:
declare -A results

process_file() {
    local file="$1"
    local result_var="$2"
    # ... processing ...
    declare -g "${result_var}=output"
}

for file in *.txt; do
    var="result_${file//[^a-zA-Z0-9]/_}"
    process_file "$file" "$var" &
done
wait

for file in *.txt; do
    var="result_${file//[^a-zA-Z0-9]/_}"
    echo "$file: ${!var}"
done
```

### Named Pipes (FIFOs)

```bash
# Create and use FIFO:
fifo=$(mktemp -u)
mkfifo "$fifo"
trap 'rm -f "$fifo"' EXIT

producer | tee "$fifo" | consumer1 &
consumer2 < "$fifo"
wait

# Bidirectional communication:
mkfifo in_pipe out_pipe
trap 'rm -f in_pipe out_pipe' EXIT

service < in_pipe > out_pipe &
SERVICE_PID=$!

exec 3> in_pipe 4< out_pipe
echo "request" >&3
read -r response <&4
```

---

## 21. SCRIPTING BEST PRACTICES

### Script Template

```bash
#!/usr/bin/env bash
# =============================================================================
# script_name.sh — Brief description
# Author: Name <email>
# Version: 1.0.0
# Usage: script_name.sh [OPTIONS] <args>
# =============================================================================

set -euo pipefail
IFS=$'\n\t'

# ── Constants ─────────────────────────────────────────────────────────────────
readonly SCRIPT_NAME="${0##*/}"
readonly SCRIPT_DIR="$(cd "$(dirname "${BASH_SOURCE[0]}")" && pwd)"
readonly SCRIPT_VERSION="1.0.0"

# ── Defaults ──────────────────────────────────────────────────────────────────
verbose=0
dry_run=0

# ── Functions ─────────────────────────────────────────────────────────────────
usage() {
    cat << EOF
Usage: $SCRIPT_NAME [OPTIONS] <input>

Options:
  -v, --verbose     Enable verbose output
  -n, --dry-run     Don't make changes
  -h, --help        Show this help

Examples:
  $SCRIPT_NAME -v input.txt
EOF
}

log()   { echo "[$SCRIPT_NAME] $*"; }
warn()  { echo "[$SCRIPT_NAME] WARN: $*" >&2; }
error() { echo "[$SCRIPT_NAME] ERROR: $*" >&2; }
die()   { error "$*"; exit 1; }

cleanup() {
    # Remove temp files etc.
    :
}
trap cleanup EXIT

# ── Argument Parsing ──────────────────────────────────────────────────────────
while [[ $# -gt 0 ]]; do
    case $1 in
        -v|--verbose) verbose=1; shift ;;
        -n|--dry-run) dry_run=1; shift ;;
        -h|--help) usage; exit 0 ;;
        --) shift; break ;;
        -*) die "Unknown option: $1" ;;
        *) break ;;
    esac
done

[[ $# -lt 1 ]] && { usage; exit 1; }
input="$1"

# ── Validation ────────────────────────────────────────────────────────────────
[[ -f "$input" ]] || die "File not found: $input"

# ── Main ──────────────────────────────────────────────────────────────────────
main() {
    log "Processing $input..."
    # ... script body ...
}

main "$@"
```

### The 10 Commandments of Bash Scripts

1. **Quote everything.** `"$var"`, `"${arr[@]}"`, always. Unquoted variables are silent bugs waiting to detonate.

2. **Use `set -euo pipefail`.** Catch errors at the source, not three functions later when the damage is done.

3. **Prefer `[[ ]]` over `[ ]`.** It's faster, doesn't need quoting tricks for comparisons, supports `=~` and `&&`/`||`, and has no surprises with special characters.

4. **Use `local` in functions.** Global state shared across functions is the source of the most insidious bugs.

5. **Never parse `ls`.** Use globs. Never use `$(cat file)` when you can redirect. Avoid unnecessary forks.

6. **Validate all inputs.** Every argument, every environment variable, every file path. Scripts fail in production in ways they never fail during development.

7. **Use `mktemp` for temp files.** Never hardcode `/tmp/something.$$`. Use `trap cleanup EXIT` to remove them.

8. **Write to stderr for errors.** `echo "error" >&2`. Users may redirect stdout; don't swallow error messages.

9. **Run `shellcheck`.** It catches 80% of common mistakes automatically. Make it part of your CI/CD.

10. **Design for re-entrancy and idempotency.** A script that can safely run twice is a script that can safely run once after partial failure.

---

### Portability Reference

| Feature | bash 3.x | bash 4.0 | bash 4.3 | bash 5.0 | bash 5.1 |
|---|---|---|---|---|---|
| Associative arrays | no | YES | yes | yes | yes |
| `declare -n` nameref | no | no | YES | yes | yes |
| `**` globstar | no | YES | yes | yes | yes |
| `${var^^}` case mod | no | YES | yes | yes | yes |
| `printf '%()T'` | no | no | YES | yes | yes |
| Coprocess | no | YES | yes | yes | yes |
| `wait -n` | no | no | no | no | YES |
| `${arr[-1]}` | no | no | YES | yes | yes |
| `local -` scope opts | no | no | no | YES | yes |
| BASH_ARGV0 | no | no | no | YES | yes |

---

### Quick Reference: Most-Used Patterns

```bash
# Dirname/basename without subprocess:
dir="${path%/*}"
base="${path##*/}"
name="${base%.*}"
ext="${base##*.}"

# Check bash version:
(( BASH_VERSINFO[0] >= 4 )) || die "Bash 4+ required"

# Absolute path of script:
SCRIPT_DIR="$(cd "$(dirname "${BASH_SOURCE[0]}")" && pwd)"

# Check if running as root:
(( EUID == 0 )) || die "Must run as root"

# Check if interactive:
[[ -t 0 ]] && echo "stdin is a terminal"
[[ -t 1 ]] && echo "stdout is a terminal"

# Read entire file into variable:
content=$(<file)           # Faster than $(cat file)

# Write variable to file:
printf '%s\n' "$content" > file

# Count lines without wc:
mapfile -t lines < file
echo "${#lines[@]}"

# Join array with delimiter:
IFS=, ; joined="${arr[*]}"; unset IFS
# Or:
printf -v joined '%s,' "${arr[@]}"; joined="${joined%,}"

# Split string into array:
IFS=',' read -ra arr <<< "$csv_string"

# Check if element in array:
is_in_array() {
    local needle="$1"; shift
    local elem
    for elem; do [[ "$elem" == "$needle" ]] && return 0; done
    return 1
}
is_in_array "target" "${arr[@]}"

# Unique elements:
mapfile -t unique < <(printf '%s\n' "${arr[@]}" | sort -u)

# Sort array:
mapfile -t sorted < <(printf '%s\n' "${arr[@]}" | sort)

# Reverse array:
reversed=()
for ((i=${#arr[@]}-1; i>=0; i--)); do
    reversed+=("${arr[$i]}")
done

# Timestamp:
ts=$(date +%Y%m%d_%H%M%S)
printf -v ts '%(%Y%m%d_%H%M%S)T' -1   # Bash 4.2+, no fork

# URL encode:
urlencode() {
    local LC_ALL=C
    local char
    for ((i=0; i<${#1}; i++)); do
        char="${1:$i:1}"
        if [[ "$char" =~ [a-zA-Z0-9.~_-] ]]; then
            printf '%s' "$char"
        else
            printf '%%%02X' "'$char"
        fi
    done
}

# Confirm prompt:
confirm() {
    read -r -p "${1:-Are you sure?} [y/N] " response
    [[ "${response,,}" == "y" ]]
}
confirm "Delete files?" || exit 0

# Run only if not sourced:
if [[ "${BASH_SOURCE[0]}" == "${0}" ]]; then
    main "$@"
fi
```

---

*This reference covers bash through version 5.2. Features marked with version requirements are not available in older shells. Always test on your target environment and run `shellcheck` on every script.*