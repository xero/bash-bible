---
permalink: /
layout: bible
---

<!-- CHAPTER START -->

# Foreword

Shell scripting can sometimes feel esoteric, cryptic, unintuitive, and error-prone. The goal of the document is to both offer a primer and quick reference of solutions to common issues and pitfalls for the bourne again shell. Examples will be in function format showcasing how to incorporate them into your code. Much of this guide will be presented as a collection of pure `bash` alternatives to external processes and programs. Calling an external process in `bash` is expensive and excessive use will cause a noticeable slowdown. Scripts and programs written using built-in methods _can_ be faster, and fewer dependencies makes them more portable.

<!-- CHAPTER END -->

<!-- CHAPTER START -->

# Subshells

<p></p>

> **_NOTE:_** _"everything"_ in `bash` is a subshell

## What is a subshell?

In UNIX `{,like}` systems processes are a tree. Upon script evaluation, `bash` creates a parent process called a **shell**. Which in-turn can spawn it's own child process called a **subshell**. It's possible for one parent to spawn multiple subshells.

A subshell, also known as a child shell, is a separate instance of the shell that is spawned from the current shell process. It inherits the environment and variables from its parent shell but operates independently, allowing for isolated execution of it's commands. When a subshell is created, it runs in a separate process, distinct from the parent shell.

> **_NOTE:_** Any changes made to the environment within the subshell, such as modifying variables or defining functions, are isolated and do not persist in the parent shell after the subshell terminates.

## Subshell environment

Every subshell has it's own context memory, called an **environment**. Parents in the tree are responsible for their children, and provide them with a _copy_ of their entire environment which they base their own upon. This means, only variables that are part of the currently executing shell's environment are available in the child process.

When you execute a program from the interactive `bash` prompt, e.g. `ls`

```sh
$ ls
````

`Bash` performs two steps:
* Makes a copy of itself (a subshell)
* The copy replaces itself with the `ls` program

The copy of `bash` will inherit the environment from the "main bash" process, with all environment variables copied to the new subshell process.

> **_NOTE:_** The replacement process is refereed to as **forking**

For a short moment, you have a process tree similar to:

```
.
└── bash
    └── bash (copy)
```

the "bash (copy)" subshell replaces itself with the `ls` program, then executes it:

```
.
└── bash
    └── ls
```

These two steps result in one program being run. The copy of the environment from the first step (forking) becomes the environment for the final running program (in this case, `ls`).

In this example, the `ls` program runs inside its own environment, it can't affect the environment of its parent process (in this case, `bash`). The state of the shell environment is copied as `ls` executes. Nothing is _"copied back"_ to the parent environment when `ls` terminates.

## Parentheses and curly braces

The commands enclosed within parentheses are executed in a subshell. This is one of the most common and straightforward ways to create a subshell in Bash.

```sh
# Create a subshell
$ (pwd; ls; whoami)
# Using curly braces {...} around a set of commands can also create a subshell
$ { sleep 3; printf '%s\n' "Hello from subshell"; }
# Subshell created
$ echo "Back in parent shell"
```

## Command substitution

Command substitution creates a subshell and captures its output, which can be assigned to a variable or used in another command.

```sh
# Assign the output of a subshell to a variable
localfunc(){ echo hi;}
$output=$(localfunc;whoami)
```

## Explicit subshell invocation

The bash built-in command can be used to start a subshell and execute commands within it explicitly. The -c option allows you to specify the commands to be executed.

```sh
# Execute a subshell
$ bash -c "ls; whoami"
```

<!-- CHAPTER END -->

<!-- CHAPTER START -->

# Variables

## Scoping

The parent shell and its subshells have a hierarchical relationship. As I have mentioned, the subshell inherits the environment variables, functions, and other settings from the parent shell, but any modifications made to the environment within the subshell are isolated and do not affect the parent shell.

```sh
declare a b c
a=foo
b=bar
c=baz

foo() {
    local a=1 b=2 c=3

    echo "function scope"
    declare -p a
    declare -p b
    declare -p c
}

foo

echo "global scope"
declare -p a
declare -p b
declare -p c
```

Output:
```
function scope
declare -- a="1"
declare -- b="2"
declare -- c="3"
global scope
declare -- a="foo"
declare -- b="bar"
declare -- c="baz"
```

## Multiple assignment

```sh
$ var1=var2=var3="same value"
```

## Assign and access a variable using a variable

```sh
$ hello_world="value"

# Create the variable name.
$ var="world"
$ ref="hello_$var"

# Print the value of the variable name stored in 'hello_$var'.
$ printf '%s\n' "${!ref}"
value
```

Alternatively, on `bash` 4.3+:

```sh
$ hello_world="value"
$ var="world"

# Declare a nameref.
$ declare -n ref=hello_$var

$ printf '%s\n' "$ref"
value
```

## Name a variable based on another variable

```sh
$ var="world"
$ declare "hello_$var=value"
$ printf '%s\n' "$hello_world"
value
```

<!-- CHAPTER END -->

<!-- CHAPTER START -->

# Special Characters

Some characters are evaluated by Bash to have a non-literal meaning. Instead, these characters carry out a special instruction, or have an alternate meaning; they are called "special characters", or "meta-characters".

| Char| Description |
| --- | ----------- |
| `" "` | **Whitespace** this is a tab, newline, vertical tab, form feed, carriage return, or space. Bash uses whitespace to determine where words begin and end. The first word is the command name and additional words become arguments to that command.
| `$` | **Expansion** introduces various types of expansion: parameter expansion (e.g. $var or ${var}), command substitution (e.g. $(command)), or arithmetic expansion (e.g. $((expression))). More on expansions later.
| `''` | **Single Quotes** protect the text inside them so that it has a literal meaning. With them, generally any kind of interpretation by Bash is ignored: special characters are passed over and multiple words are prevented from being split.
| `""` | **Double Quotes** protect the text inside them from being split into multiple words or arguments, yet allow substitutions to occur; the meaning of most other special characters is usually prevented.
| `\` | **Escape** (backslash) prevents the next character from being interpreted as a special character. This works outside of quoting, inside double quotes, and generally ignored in single quotes.
| `#` | **Comment** the # character begins a commentary that extends to the end of the line. Comments are notes of explanation and are not processed by the shell.
| `=` | **Assignment** assign a value to a variable (e.g. logdir=/var/log/myprog). Whitespace is not allowed on either side of the = character.
| `[[ ]]` | **Test** an evaluation of a conditional expression to determine whether it is "true" or "false". Tests are used in Bash to compare strings, check the existence of a file, etc. More of this will be covered later.
| `!` | **Negate** used to negate or reverse a test or exit status. For example: ! grep text file; exit $?.
| `>, >>, <` | **Redirection** redirect a command's output or input to a file. Redirections will be covered later.
| `|` | **Pipe** send the output from one command to the input of another command. This is a method of chaining commands together. Example: echo **Hello beautiful" \| grep -o beautiful**
| `;` | **Command Separator** used to separate multiple commands that are on the same line.
| `{ }` | **Inline Group** commands inside the curly braces are treated as if they were one command. It is convenient to use these when Bash syntax requires only one command and a function doesn't feel warranted.
| `( )` | **Subshell Group** similar to the above but where commands within are executed in a subshell (a new process). Used much like a sandbox, if a command causes side effects (like changing variables), it will have no effect on the current shell.
| `(( ))` | **Arithmetic Expression** with an arithmetic expression, characters such as +, -, *, and / are mathematical operators used for calculations. They can be used for variable assignments like (( a = 1 + 4 )) as well as tests like if (( a < b )). More on this later.
| `$(( ))` | **Arithmetic Expansion** Comparable to the above, but the expression is replaced with the result of its arithmetic evaluation. Example: echo "The average is $(( (a+b)/2 ))".
| `*, ?` | **Globs** "wildcard" characters which match parts of filenames (e.g. ls *.txt).
| `~` | **Home** directory the tilde is a representation of a home directory. When alone or followed by a /, it means the current user's home directory; otherwise, a username must be specified (e.g. ls ~/Documents; cp ~john/.bashrc .).
| `&` | **Background** when used at the end of a command, run the command in the background (do not wait for it to complete).

Examples:

```sh
$ LOGNAME="LOG"
$ echo "I am "$LOGNAME""
I am LOG
$ echo 'I am $LOGNAME'
I am $LOGNAME
$ # ignore me
$ echo An open\ \ \ space
An open   space
$ echo "My computer is $(hostname)"
My computer is localhost
$ echo "$STUFF" > file
$ echo "append" >> file
$ echo $(( 5 + 5 ))
10
$ (( 5 > 0 )) && echo "Five is greater than zero."
Five is greater than zero.
```

<!-- CHAPTER END -->

<!-- CHAPTER START -->

# Strings

## Trim leading and trailing white-space from string

This is an alternative to `sed`, `awk`, `perl` and other tools. The
function below works by finding all leading and trailing white-space and
removing it from the start and end of the string. The `:` built-in is used in place of a temporary variable.

**Example Function:**

```sh
trim_string() {
    # Usage: trim_string "   example   string    "
    : "${1#"${1%%[![:space:]]*}"}"
    : "${_%"${_##*[![:space:]]}"}"
    printf '%s\n' "$_"
}
```

**Example Usage:**

```sh
$ trim_string "    Hello,  World    "
Hello,  World

$ name="   John Black  "
$ trim_string "$name"
John Black
```

## Trim all white-space from string and truncate spaces

This is an alternative to `sed`, `awk`, `perl` and other tools. The
function below works by abusing word splitting to create a new string
without leading/trailing white-space and with truncated spaces.

**Example Function:**

```sh
# shellcheck disable=SC2086,SC2048
trim_all() {
    # Usage: trim_all "   example   string    "
    set -f
    set -- $*
    printf '%s\n' "$*"
    set +f
}
```

**Example Usage:**

```sh
$ trim_all "    Hello,    World    "
Hello, World

$ name="   John   Black  is     my    name.    "
$ trim_all "$name"
John Black is my name.
```

## Use regex on a string

The result of `bash`'s regex matching can be used to replace `sed` for a
large number of use-cases.

> **_CAVEAT:_** This is one of the few platform dependent `bash` features.
`bash` will use whatever regex engine is installed on the user's system.
Stick to POSIX regex features if aiming for compatibility.
{: .caveat }

> **_CAVEAT:_** This example only prints the first matching group. When using
multiple capture groups some modification is needed.
{: .caveat }

**Example Function:**

```sh
regex() {
    # Usage: regex "string" "regex"
    [[ $1 =~ $2 ]] && printf '%s\n' "${BASH_REMATCH[1]}"
}
```

**Example Usage:**

```sh
$ # Trim leading white-space.
$ regex '    hello' '^\s*(.*)'
hello

$ # Validate a hex color.
$ regex "#FFFFFF" '^(#?([a-fA-F0-9]{6}|[a-fA-F0-9]{3}))$'
#FFFFFF

$ # Validate a hex color (invalid).
$ regex "red" '^(#?([a-fA-F0-9]{6}|[a-fA-F0-9]{3}))$'
# no output (invalid)
```

**Example Usage in script:**

```sh
is_hex_color() {
    if [[ $1 =~ ^(#?([a-fA-F0-9]{6}|[a-fA-F0-9]{3}))$ ]]; then
        printf '%s\n' "${BASH_REMATCH[1]}"
    else
        printf '%s\n' "error: $1 is an invalid color."
        return 1
    fi
}

read -r color
is_hex_color "$color" || color="#FFFFFF"

# Do stuff.
```

## Split a string on a delimiter

> **_NOTE:_** Requires `bash` 4+

This is an alternative to `cut`, `awk` and other tools.

**Example Function:**

```sh
split() {
   # Usage: split "string" "delimiter"
   IFS=$'\n' read -d "" -ra arr <<< "${1//$2/$'\n'}"
   printf '%s\n' "${arr[@]}"
}
```

**Example Usage:**

```sh
$ split "apples,oranges,pears,grapes" ","
apples
oranges
pears
grapes

$ split "1, 2, 3, 4, 5" ", "
1
2
3
4
5

# Multi char delimiters work too!
$ split "hello---world---my---name---is---john" "---"
hello
world
my
name
is
john
```

## Change a string to lowercase

> **_NOTE:_** Requires `bash` 4+

**Example Function:**

```sh
lower() {
    # Usage: lower "string"
    printf '%s\n' "${1,,}"
}
```

**Example Usage:**

```sh
$ lower "HELLO"
hello

$ lower "HeLlO"
hello

$ lower "hello"
hello
```

## Change a string to uppercase

> **_NOTE:_** Requires `bash` 4+

**Example Function:**

```sh
upper() {
    # Usage: upper "string"
    printf '%s\n' "${1^^}"
}
```

**Example Usage:**

```sh
$ upper "hello"
HELLO

$ upper "HeLlO"
HELLO

$ upper "HELLO"
HELLO
```

## Reverse a string case

> **_NOTE:_** Requires `bash` 4+

**Example Function:**

```sh
reverse_case() {
    # Usage: reverse_case "string"
    printf '%s\n' "${1~~}"
}
```

**Example Usage:**

```sh
$ reverse_case "hello"
HELLO

$ reverse_case "HeLlO"
hElLo

$ reverse_case "HELLO"
hello
```

## Trim quotes from a string

**Example Function:**

```sh
trim_quotes() {
    # Usage: trim_quotes "string"
    : "${1//\'}"
    printf '%s\n' "${_//\"}"
}
```

**Example Usage:**

```sh
$ var="'Hello', \"World\""
$ trim_quotes "$var"
Hello, World
```

## Strip all instances of pattern from string

**Example Function:**

```sh
strip_all() {
    # Usage: strip_all "string" "pattern"
    printf '%s\n' "${1//$2}"
}
```

**Example Usage:**

```sh
$ strip_all "The Quick Brown Fox" "[aeiou]"
Th Qck Brwn Fx

$ strip_all "The Quick Brown Fox" "[[:space:]]"
TheQuickBrownFox

$ strip_all "The Quick Brown Fox" "Quick "
The Brown Fox
```

## Strip first occurrence of pattern from string

**Example Function:**

```sh
strip() {
    # Usage: strip "string" "pattern"
    printf '%s\n' "${1/$2}"
}
```

**Example Usage:**

```sh
$ strip "The Quick Brown Fox" "[aeiou]"
Th Quick Brown Fox

$ strip "The Quick Brown Fox" "[[:space:]]"
TheQuick Brown Fox
```

## Strip pattern from start of string

**Example Function:**

```sh
lstrip() {
    # Usage: lstrip "string" "pattern"
    printf '%s\n' "${1##$2}"
}
```

**Example Usage:**

```sh
$ lstrip "The Quick Brown Fox" "The "
Quick Brown Fox
```

## Strip pattern from end of string

**Example Function:**

```sh
rstrip() {
    # Usage: rstrip "string" "pattern"
    printf '%s\n' "${1%%$2}"
}
```

**Example Usage:**

```sh
$ rstrip "The Quick Brown Fox" " Fox"
The Quick Brown
```

## Percent-encode a string

**Example Function:**

```sh
urlencode() {
    # Usage: urlencode "string"
    local LC_ALL=C
    for (( i = 0; i < ${#1}; i++ )); do
        : "${1:i:1}"
        case "$_" in
            [a-zA-Z0-9.~_-])
                printf '%s' "$_"
            ;;

            *)
                printf '%%%02X' "'$_"
            ;;
        esac
    done
    printf '\n'
}
```

**Example Usage:**

```sh
$ urlencode "https://github.com/dylanaraps/pure-bash-bible"
https%3A%2F%2Fgithub.com%2Fdylanaraps%2Fpure-bash-bible
```

## Decode a percent-encoded string

**Example Function:**

```sh
urldecode() {
    # Usage: urldecode "string"
    : "${1//+/ }"
    printf '%b\n' "${_//%/\\x}"
}
```

**Example Usage:**

```sh
$ urldecode "https%3A%2F%2Fgithub.com%2Fdylanaraps%2Fpure-bash-bible"
https://github.com/dylanaraps/pure-bash-bible
```

## Check if string contains a sub-string

**Using a test:**

```sh
if [[ $var == *sub_string* ]]; then
    printf '%s\n' "sub_string is in var."
fi

# Inverse (substring not in string).
if [[ $var != *sub_string* ]]; then
    printf '%s\n' "sub_string is not in var."
fi

# This works for arrays too!
if [[ ${arr[*]} == *sub_string* ]]; then
    printf '%s\n' "sub_string is in array."
fi
```

**Using a case statement:**

```sh
case "$var" in
    *sub_string*)
        # Do stuff
    ;;

    *sub_string2*)
        # Do more stuff
    ;;

    *)
        # Else
    ;;
esac
```

## Check if string starts with sub-string

```sh
if [[ $var == sub_string* ]]; then
    printf '%s\n' "var starts with sub_string."
fi

# Inverse (var does not start with sub_string).
if [[ $var != sub_string* ]]; then
    printf '%s\n' "var does not start with sub_string."
fi
```

## Check if string ends with sub-string

```sh
if [[ $var == *sub_string ]]; then
    printf '%s\n' "var ends with sub_string."
fi

# Inverse (var does not end with sub_string).
if [[ $var != *sub_string ]]; then
    printf '%s\n' "var does not end with sub_string."
fi
```

<!-- CHAPTER END -->

<!-- CHAPTER START -->

# Arrays

## Reverse an array

Enabling `extdebug` allows access to the `BASH_ARGV` array which stores
the current function’s arguments in reverse.

> **_NOTE:_** Requires `shopt -s compat44` in `bash` 5.0+.

**Example Function:**

```sh
reverse_array() {
    # Usage: reverse_array "array"
    shopt -s extdebug
    f()(printf '%s\n' "${BASH_ARGV[@]}"); f "$@"
    shopt -u extdebug
}
```

**Example Usage:**

```sh
$ reverse_array 1 2 3 4 5
5
4
3
2
1

$ arr=(red blue green)
$ reverse_array "${arr[@]}"
green
blue
red
```

## Remove duplicate array elements

Create a temporary associative array. When setting associative array
values and a duplicate assignment occurs, bash overwrites the key. This
allows us to effectively remove array duplicates.

> **_NOTE:_** Requires `bash` 4+

> **_CAVEAT:_** List order may not stay the same.
{: .caveat }

**Example Function:**

```sh
remove_array_dups() {
    # Usage: remove_array_dups "array"
    declare -A tmp_array

    for i in "$@"; do
        [[ $i ]] && IFS=" " tmp_array["${i:- }"]=1
    done

    printf '%s\n' "${!tmp_array[@]}"
}
```

**Example Usage:**

```sh
$ remove_array_dups 1 1 2 2 3 3 3 3 3 4 4 4 4 4 5 5 5 5 5 5
1
2
3
4
5

$ arr=(red red green blue blue)
$ remove_array_dups "${arr[@]}"
red
green
blue
```

## Random array element

**Example Function:**

```sh
random_array_element() {
    # Usage: random_array_element "array"
    local arr=("$@")
    printf '%s\n' "${arr[RANDOM % $#]}"
}
```

**Example Usage:**

```sh
$ array=(red green blue yellow brown)
$ random_array_element "${array[@]}"
yellow

# Multiple arguments can also be passed.
$ random_array_element 1 2 3 4 5 6 7
3
```

## Cycle through an array

Each time the `printf` is called, the next array element is printed. When
the print hits the last array element it starts from the first element
again.

```sh
arr=(a b c d)

cycle() {
    printf '%s ' "${arr[${i:=0}]}"
    ((i=i>=${#arr[@]}-1?0:++i))
}
```

## Toggle between two values

This works the same as above, this is just a different use case.

```sh
arr=(true false)

cycle() {
    printf '%s ' "${arr[${i:=0}]}"
    ((i=i>=${#arr[@]}-1?0:++i))
}
```

<!-- CHAPTER END -->

<!-- CHAPTER START -->

# Loops

## Loop over a range of numbers

Alternative to `seq`.

```sh
# Loop from 0-100 (no variable support).
for i in {0..100}; do
    printf '%s\n' "$i"
done
```

## Loop over a variable range of numbers

Alternative to `seq`.

```sh
# Loop from 0-VAR.
VAR=50
for ((i=0;i<=VAR;i++)); do
    printf '%s\n' "$i"
done
```

## Loop over an array

```sh
arr=(apples oranges tomatoes)

# Just elements.
for element in "${arr[@]}"; do
    printf '%s\n' "$element"
done
```

## Loop over an array with an index

```sh
arr=(apples oranges tomatoes)

# Elements and index.
for i in "${!arr[@]}"; do
    printf '%s\n' "${arr[i]}"
done

# Alternative method.
for ((i=0;i<${#arr[@]};i++)); do
    printf '%s\n' "${arr[i]}"
done
```

## Loop over the contents of a file

```sh
while read -r line; do
    printf '%s\n' "$line"
done < "file"
```

## Loop over files and directories

Don’t use `ls`.

```sh
# Greedy example.
for file in *; do
    printf '%s\n' "$file"
done

# PNG files in dir.
for file in ~/Pictures/*.png; do
    printf '%s\n' "$file"
done

# Iterate over directories.
for dir in ~/Downloads/*/; do
    printf '%s\n' "$dir"
done

# Brace Expansion.
for file in /path/to/parentdir/{file1,file2,subdir/file3}; do
    printf '%s\n' "$file"
done

# Iterate recursively.
shopt -s globstar
for file in ~/Pictures/**/*; do
    printf '%s\n' "$file"
done
shopt -u globstar
```

<!-- CHAPTER END -->

<!-- CHAPTER START -->

# File Handling

> **_CAVEAT:_** `bash` does not handle binary data properly in versions `< 4.4`.
{: .caveat }

## Read a file to a string

Alternative to the `cat` command.

```sh
file_data="$(<"file")"
```

## Read a file to an array (*by line*)

Alternative to the `cat` command.

```sh
# Bash <4 (discarding empty lines).
IFS=$'\n' read -d "" -ra file_data < "file"

# Bash <4 (preserving empty lines).
while read -r line; do
    file_data+=("$line")
done < "file"

# Bash 4+
mapfile -t file_data < "file"
```

## Get the first N lines of a file

Alternative to the `head` command.

**NOTE:** Requires `bash` 4+

**Example Function:**

```sh
head() {
    # Usage: head "n" "file"
    mapfile -tn "$1" line < "$2"
    printf '%s\n' "${line[@]}"
}
```

**Example Usage:**

```sh
$ head 2 ~/.bashrc
# Prompt
PS1='➜ '

$ head 1 ~/.bashrc
# Prompt
```

## Get the last N lines of a file

Alternative to the `tail` command.

**NOTE:** Requires `bash` 4+

**Example Function:**

```sh
tail() {
    # Usage: tail "n" "file"
    mapfile -tn 0 line < "$2"
    printf '%s\n' "${line[@]: -$1}"
}
```

**Example Usage:**

```sh
$ tail 2 ~/.bashrc
# Enable tmux.
# [[ -z "$TMUX"  ]] && exec tmux

$ tail 1 ~/.bashrc
# [[ -z "$TMUX"  ]] && exec tmux
```

## Get the number of lines in a file

Alternative to `wc -l`.

**Example Function (bash 4):**

```sh
lines() {
    # Usage: lines "file"
    mapfile -tn 0 lines < "$1"
    printf '%s\n' "${#lines[@]}"
}
```

**Example Function (bash 3):**

This method uses less memory than the `mapfile` method and works in `bash` 3 but it is slower for bigger files.

```sh
lines_loop() {
    # Usage: lines_loop "file"
    count=0
    while IFS= read -r _; do
        ((count++))
    done < "$1"
    printf '%s\n' "$count"
}
```

**Example Usage:**

```sh
$ lines ~/.bashrc
48

$ lines_loop ~/.bashrc
48
```

## Count files or directories in directory

This works by passing the output of the glob to the function and then counting the number of arguments.

**Example Function:**

```sh
count() {
    # Usage: count /path/to/dir/*
    #        count /path/to/dir/*/
    printf '%s\n' "$#"
}
```

**Example Usage:**

```sh
# Count all files in dir.
$ count ~/Downloads/*
232

# Count all dirs in dir.
$ count ~/Downloads/*/
45

# Count all jpg files in dir.
$ count ~/Pictures/*.jpg
64
```

## Create an empty file

Alternative to `touch`.

```sh
# Shortest.
>file

# Longer alternatives:
:>file
echo -n >file
printf '' >file
```

## Extract lines between two markers

**Example Function:**

```sh
extract() {
    # Usage: extract file "opening marker" "closing marker"
    while IFS=$'\n' read -r line; do
        [[ $extract && $line != "$3" ]] &&
            printf '%s\n' "$line"

        [[ $line == "$2" ]] && extract=1
        [[ $line == "$3" ]] && extract=
    done < "$1"
}
```

**Example Usage:**

```sh
# Extract code blocks from MarkDown file.
$ extract ~/projects/pure-bash/README.md '```sh' '```'
# Output here...
```

<!-- CHAPTER END -->

<!-- CHAPTER START -->

# File Paths

## Get the directory name of a file path

Alternative to the `dirname` command.

**Example Function:**

```sh
dirname() {
    # Usage: dirname "path"
    local tmp=${1:-.}

    [[ $tmp != *[!/]* ]] && {
        printf '/\n'
        return
    }

    tmp=${tmp%%"${tmp##*[!/]}"}

    [[ $tmp != */* ]] && {
        printf '.\n'
        return
    }

    tmp=${tmp%/*}
    tmp=${tmp%%"${tmp##*[!/]}"}

    printf '%s\n' "${tmp:-/}"
}
```

**Example Usage:**

```sh
$ dirname ~/Pictures/Wallpapers/1.jpg
/home/black/Pictures/Wallpapers

$ dirname ~/Pictures/Downloads/
/home/black/Pictures
```

## Get the base-name of a file path

Alternative to the `basename` command.

**Example Function:**

```sh
basename() {
    # Usage: basename "path" ["suffix"]
    local tmp

    tmp=${1%"${1##*[!/]}"}
    tmp=${tmp##*/}
    tmp=${tmp%"${2/"$tmp"}"}

    printf '%s\n' "${tmp:-/}"
}
```

**Example Usage:**

```sh
$ basename ~/Pictures/Wallpapers/1.jpg
1.jpg

$ basename ~/Pictures/Wallpapers/1.jpg .jpg
1

$ basename ~/Pictures/Downloads/
Downloads
```

<!-- CHAPTER END -->

<!-- CHAPTER START -->

# Variables

## Scoping

```sh
declare a b c
a=foo
b=bar
c=baz

foo() {
    local a=1 b=2 c=3

    echo "function scope"
    declare -p a
    declare -p b
    declare -p c
}

foo

echo "global scope"
declare -p a
declare -p b
declare -p c
```
Output:

```sh
function scope
declare -- a="1"
declare -- b="2"
declare -- c="3"
global scope
declare -- a="foo"
declare -- b="bar"
declare -- c="baz"
```
## Multiple assignment

```sh
$ var1=var2=var3="same value"
```

## Assign and access a variable using a variable

```sh
$ hello_world="value"

# Create the variable name.
$ var="world"
$ ref="hello_$var"

# Print the value of the variable name stored in 'hello_$var'.
$ printf '%s\n' "${!ref}"
value
```

Alternatively, on `bash` 4.3+:

```sh
$ hello_world="value"
$ var="world"

# Declare a nameref.
$ declare -n ref=hello_$var

$ printf '%s\n' "$ref"
value
```

## Name a variable based on another variable

```sh
$ var="world"
$ declare "hello_$var=value"
$ printf '%s\n' "$hello_world"
value
```

<!-- CHAPTER END -->

<!-- CHAPTER START -->

# Escape Sequences

Contrary to popular belief, there is no issue in utilizing raw escape sequences. Using `tput` abstracts the same ANSI sequences as if printed manually. Worse still, `tput` is not actually portable. There are a number of `tput` variants each with different commands and syntaxes (*try `tput setaf 3` on a FreeBSD system*). Raw sequences are fine.

## Text Colors

**NOTE:** Sequences requiring RGB values only work in True-Color Terminal Emulators.

| Sequence | Description | Value |
| -------- | ----------- | ----- |
| `\e[38;5;<NUM>m` | Set text foreground color | **0-255**
| `\e[48;5;<NUM>m` | Set text background color | **0-255**
| `\e[38;2;<R>;<G>;<B>m` | Set text foreground color to RGB color | **R**, **G**, **B**
| `\e[48;2;<R>;<G>;<B>m` | Set text background color to RGB color | **R**, **G**, **B**

## Text Attributes

**NOTE:** Prepend 2 to any code below to turn it's effect off
(examples: 21=bold text off, 22=faint text off, 23=italic text off).

| Sequence | Description |
| -------- | ----------- |
| `\e[m` | Reset text formatting and colors
| `\e[1m` | Bold text
| `\e[2m` | Faint text
| `\e[3m` | Italic text
| `\e[4m` | Underline text
| `\e[5m` | Blinking text
| `\e[7m` | Highlighted text
| `\e[8m` | Hidden text
| `\e[9m` | Strike-through text

## Cursor Movement

| Sequence | Description | Value |
| -------- | ----------- | ----- |
| `\e[<LINE>;<COLUMN>H` | Move cursor to absolute position. | **line**, **column**
| `\e[H` | Move cursor to home position (**0,0**) |
| `\e[<NUM>A` | Move cursor up N lines | **num**
| `\e[<NUM>B` | Move cursor down N lines | **num**
| `\e[<NUM>C` | Move cursor right N columns | **num**
| `\e[<NUM>D` | Move cursor left N columns | **num**
| `\e[s` | Save cursor position |
| `\e[u` | Restore cursor position |

## Erasing Text

| Sequence | Description |
| -------- | ------------|
| `\e[K` | Erase from cursor position to end of line
| `\e[1K` | Erase from cursor position to start of line
| `\e[2K` | Erase the entire current line
| `\e[J` | Erase from the current line to the bottom of the screen
| `\e[1J` | Erase from the current line to the top of the screen
| `\e[2J` | Clear the screen
| `\e[2J\e[H` | Clear the screen and move cursor to **0,0**

<!-- CHAPTER END -->

<!-- CHAPTER START -->

# Parameter Expansion

## Indirection

| Parameter | Description |
| --------- | ----------- |
| `${!VAR}` | Access a variable based on the value of **VAR**
| `${!VAR*}` | Expand to **IFS** separated list of variable names starting with **VAR**
| `${!VAR@}` | Expand to **IFS** separated list of variable names starting with **VAR**. If double-quoted, each variable name expands to a separate word

## Replacement

| Parameter | Description |
| --------- | ----------- |
| `${VAR#PATTERN}` | Remove shortest match of pattern from start of string
| `${VAR##PATTERN}` | Remove longest match of pattern from start of string
| `${VAR%PATTERN}` | Remove shortest match of pattern from end of string
| `${VAR%%PATTERN}` | Remove longest match of pattern from end of string
| `${VAR/PATTERN/REPLACE}` | Replace first match with string
| `${VAR//PATTERN/REPLACE}` | Replace all matches with string
| `${VAR/PATTERN}` | Remove first match
| `${VAR//PATTERN}` | Remove all matches

## Length

| Parameter | Description |
| --------- | ----------- |
| `${#VAR}` | Length of var in characters
| `${#ARR[@]}` | Length of array in elements

## Expansion

| Parameter | Description |
| --------- | ----------- |
| `${VAR:OFFSET}` | Remove first **N** chars from variable
| `${VAR:OFFSET:LENGTH}` | Get substring from **N** character to **N** character. <br> (**${VAR:10:10}**: Get sub-string from char **10** to char **20**)
| `${VAR:: OFFSET}` | Get first **N** chars from variable
| `${VAR:: -OFFSET}` | Remove last **N** chars from variable
| `${VAR: -OFFSET}` | Get last **N** chars from variable
| `${VAR:OFFSET:-OFFSET}` | Cut first **N** chars and last **N** chars. requires **bash 4.2+**

## Case Modification

| Parameter | Description | Notes |
| --------- | ----------- | ----- |
| `${VAR^}` | Uppercase first character | **bash 4+**
| `${VAR^^}` | Uppercase all characters | **bash 4+**
| `${VAR,}` | Lowercase first character | **bash 4+**
| `${VAR,,}` | Lowercase all characters | **bash 4+**
| `${VAR~}` | Reverse case of first character | **bash 4+**
| `${VAR~~}` | Reverse case of all characters | **bash 4+**

## Default Value

| Parameter | Description |
| --------- | ----------- |
| `${VAR:-STRING}` | If **VAR** is empty or unset, use **STRING** as its value
| `${VAR-STRING}` | If **VAR** is unset, use **STRING** as its value
| `${VAR:=STRING}` | If **VAR** is empty or unset, set the value of **VAR** to **STRING**
| `${VAR=STRING}` | If **VAR** is unset, set the value of **VAR** to **STRING**
| `${VAR:+STRING}` | If **VAR** is not empty, use **STRING** as its value
| `${VAR+STRING}` | If **VAR** is set, use **STRING** as its value
| `${VAR:?STRING}` | Display an error if empty or unset
| `${VAR?STRING}` | Display an error if unset

<!-- CHAPTER END -->

<!-- CHAPTER START -->

# Brace Expansion

## Ranges

```sh
# Syntax: {<START>..<END>}

# Print numbers 1-100.
echo {1..100}

# Print range of floats.
echo 1.{1..9}

# Print chars a-z.
echo {a..z}
echo {A..Z}

# Nesting.
echo {A..Z}{0..9}

# Print zero-padded numbers.
# NOTE: requires bash 4+
echo {01..100}

# Change increment amount.
# Syntax: {<START>..<END>..<INCREMENT>}
# NOTE: requires bash 4+
echo {1..10..2} # Increment by 2.
```

## String Lists

```sh
echo {apples,oranges,pears,grapes}

# Example Usage:
# Remove dirs Movies, Music and ISOS from ~/Downloads/.
rm -rf ~/Downloads/{Movies,Music,ISOS}
```

<!-- CHAPTER END -->

<!-- CHAPTER START -->

# Conditional Expressions

## File Conditionals

| Expression | Value  | Description |
| ---------- | ------ | ----------- |
| `-a` | `file` | If file exists
| `-b` | `file` | If file exists and is a block special file
| `-c` | `file` | If file exists and is a character special file
| `-d` | `file` | If file exists and is a directory
| `-e` | `file` | If file exists
| `-f` | `file` | If file exists and is a regular file
| `-g` | `file` | If file exists and its set-group-id bit is set
| `-h` | `file` | If file exists and is a symbolic link
| `-k` | `file` | If file exists and its sticky-bit is set
| `-p` | `file` | If file exists and is a named pipe (*FIFO*)
| `-r` | `file` | If file exists and is readable
| `-s` | `file` | If file exists and its size is greater than zero
| `-t` | `fd` | If file descriptor is open and refers to a terminal
| `-u` | `file` | If file exists and its set-user-id bit is set
| `-w` | `file` | If file exists and is writable
| `-x` | `file` | If file exists and is executable
| `-G` | `file` | If file exists and is owned by the effective group ID
| `-L` | `file` | If file exists and is a symbolic link
| `-N` | `file` | If file exists and has been modified since last read
| `-O` | `file` | If file exists and is owned by the effective user ID
| `-S` | `file` | If file exists and is a socket

## File Comparisons

| Expression | Description |
| ---------- | ----------- |
| `file -ef file2` | If both files refer to the same inode and device numbers
| `file -nt file2` | If **file** is newer than **file2** (*uses modification time*) or **file** exists and **file2** does not
| `file -ot file2` | If **file** is older than **file2** (*uses modification time*) or **file2** exists and **file** does not

## Variable Conditionals

| Expression | Value | Description |
| ---------- | ----- | ----------- |
| `-o` | `opt` | If shell option is enabled
| `-v` | `var` | If variable has a value assigned
| `-R` | `var` | If variable is a name reference
| `-z` | `var` | If the length of string is zero
| `-n` | `var` | If the length of string is non-zero

## Variable Comparisons

| Expression | Description |
| ---------- | ----------- |
| `var = var2` | Equal to
| `var == var2` | Equal to (synonym for **=**)
| `var != var2` | Not equal to
| `var < var2` | Less than (*in ASCII alphabetical order.*)
| `var > var2` | Greater than (*in ASCII alphabetical order.*)

<!-- CHAPTER END -->

<!-- CHAPTER START -->

# Arithmetic Operators

## Assignment

| Operators | Description |
| --------- | ----------- |
| `=`       | Initialize or change the value of a variable

## Arithmetic

| Operators | Description |
| --------- | ----------- |
| `+` | Addition
| `-` | Subtraction
| `*` | Multiplication
| `/` | Division
| `**` | Exponentiation
| `%` | Modulo
| `+=` | Plus-Equal (*Increment a variable.*)
| `-=` | Minus-Equal (*Decrement a variable.*)
| `*=` | Times-Equal (*Multiply a variable.*)
| `/=` | Slash-Equal (*Divide a variable.*)
| `%=` | Mod-Equal (*Remainder of dividing a variable.*)

## Bitwise

| Operators | Description |
| --------- | ----------- |
| `<<` | Bitwise Left Shift
| `<<=` | Left-Shift-Equal
| `>>` | Bitwise Right Shift
| `>>=` | Right-Shift-Equal
| `&` | Bitwise AND
| `&=` | Bitwise AND-Equal
| `\|` | Bitwise OR
| `\|=` | Bitwise OR-Equal
| `~` | Bitwise NOT
| `^` | Bitwise XOR
| `^=` | Bitwise XOR-Equal

## Logical

| Operators | Description |
| --------- | ----------- |
| `!` | NOT
| `&&` | AND
| `\|\|` | OR

## Miscellaneous

| Operators | Description | Example |
| --------- | ----------- | ------- |
| `,` | Comma Separator | `((a=1,b=2,c=3))`

<!-- CHAPTER END -->

<!-- CHAPTER START -->

# Arithmetic

## Simpler syntax to set variables

```sh
# Simple math
((var=1+2))

# Decrement/Increment variable
((var++))
((var--))
((var+=1))
((var-=1))

# Using variables
((var=var2*arr[2]))
```

## Ternary Tests

```sh
# Set the value of var to var2 if var2 is greater than var.
# var: variable to set.
# var2>var: Condition to test.
# ?var2: If the test succeeds.
# :var: If the test fails.
((var=var2>var?var2:var))
```

<!-- CHAPTER END -->

<!-- CHAPTER START -->

# Traps

Traps allow a script to execute code on various signals. In [pxltrm](https://github.com/dylanaraps/pxltrm) (*a pixel art editor written in bash*)  traps are used to redraw the user interface on window resize. Another use case is cleaning up temporary files on script exit.

Traps should be added near the start of scripts so any early errors are also caught.

**NOTE:** For a full list of signals, see `trap -l`.

## On script exit

```sh
# Clear screen on script exit.
trap 'printf \\e[2J\\e[H\\e[m' EXIT
```

## Ignoring interrupts (CTRL+C, SIGINT)

```sh
trap '' INT
```

## React to window resize

```sh
# Call a function on window resize.
trap 'code_here' SIGWINCH
```

## Pre-command execution

```sh
trap 'code_here' DEBUG
```

## On funcion or source completion

```sh
trap 'code_here' RETURN
```

<!-- CHAPTER END -->

<!-- CHAPTER START -->

# Performance

## Disable Unicode

If unicode is not required, it can be disabled for a performance increase. Results may vary however there have been noticeable improvements in [neofetch](https://github.com/dylanaraps/neofetch) and other programs.

```sh
# Disable unicode.
LC_ALL=C
LANG=C
```

<!-- CHAPTER END -->

<!-- CHAPTER START -->

# Obsolete Syntax

## Shebang

Use `#!/usr/bin/env bash` instead of `#!/bin/bash`.

- The former searches the user's `PATH` to find the `bash` binary.
- The latter assumes it is always installed to `/bin/` which can cause issues.

**NOTE:** There are times when one may have a good reason for using `#!/bin/bash` or another direct path to the binary.

```sh
# Right:

    #!/usr/bin/env bash

# Less right:

    #!/bin/bash
```

## Command Substitution

Use `$()` instead of `` ` ` ``.

```sh
# Right.
var="$(command)"

# Wrong.
var=`command`

# $() can easily be nested whereas `` cannot.
var="$(command "$(command)")"
```

## Function Declaration

Do not use the `function` keyword, it reduces compatibility with older versions of `bash`.

```sh
# Right.
do_something() {
    # ...
}

# Wrong.
function do_something() {
    # ...
}
```

<!-- CHAPTER END -->

<!-- CHAPTER START -->

# Internal Variables

## Get the location to the `bash` binary

```sh
"$BASH"
```

## Get the version of the current running `bash` process

```sh
# As a string.
"$BASH_VERSION"

# As an array.
"${BASH_VERSINFO[@]}"
```

## Open the user's preferred text editor

```sh
"$EDITOR" "$file"

# NOTE: This variable may be empty, set a fallback value.
"${EDITOR:-vi}" "$file"
```

## Get the name of the current function

```sh
# Current function.
"${FUNCNAME[0]}"

# Parent function.
"${FUNCNAME[1]}"

# So on and so forth.
"${FUNCNAME[2]}"
"${FUNCNAME[3]}"

# All functions including parents.
"${FUNCNAME[@]}"
```

## Get the host-name of the system

```sh
"$HOSTNAME"

# NOTE: This variable may be empty.
# Optionally set a fallback to the hostname command.
"${HOSTNAME:-$(hostname)}"
```

## Get the architecture of the Operating System

```sh
"$HOSTTYPE"
```

## Get the name of the Operating System / Kernel

This can be used to add conditional support for different Operating
Systems without needing to call `uname`.

```sh
"$OSTYPE"
```

## Get the current working directory

This is an alternative to the `pwd` built-in.

```sh
"$PWD"
```

## Get the number of seconds the script has been running

```sh
"$SECONDS"
```

## Get a pseudorandom integer

Each time `$RANDOM` is used, a different integer between `0` and `32767` is returned. This variable should not be used for anything related to security (*this includes encryption keys etc*).

```sh
"$RANDOM"
```

<!-- CHAPTER END -->

<!-- CHAPTER START -->

# Information About the Terminal

## Get the terminal size in lines and columns (*from a script*)

This is handy when writing scripts in pure bash and `stty`/`tput` can’t be
called.

**Example Function:**

```sh
get_term_size() {
    # Usage: get_term_size

    # (:;:) is a micro sleep to ensure the variables are
    # exported immediately.
    shopt -s checkwinsize; (:;:)
    printf '%s\n' "$LINES $COLUMNS"
}
```

**Example Usage:**

```sh
# Output: LINES COLUMNS
$ get_term_size
15 55
```

## Get the terminal size in pixels

> **_CAVEAT:_** This does not work in some terminal emulators.
{: .caveat }

**Example Function:**

```sh
get_window_size() {
    # Usage: get_window_size
    printf '%b' "${TMUX:+\\ePtmux;\\e}\\e[14t${TMUX:+\\e\\\\}"
    IFS=';t' read -d t -t 0.05 -sra term_size
    printf '%s\n' "${term_size[1]}x${term_size[2]}"
}
```

**Example Usage:**

```sh
# Output: WIDTHxHEIGHT
$ get_window_size
1200x800

# Output (fail):
$ get_window_size
x
```

## Get the current cursor position

This is useful when creating a TUI in pure bash.

**Example Function:**

```sh
get_cursor_pos() {
    # Usage: get_cursor_pos
    IFS='[;' read -p $'\e[6n' -d R -rs _ y x _
    printf '%s\n' "$x $y"
}
```

**Example Usage:**

```sh
# Output: X Y
$ get_cursor_pos
1 8
```

<!-- CHAPTER END -->

<!-- CHAPTER START -->

# Conversion

## Convert a hex color to RGB

**Example Function:**

```sh
hex_to_rgb() {
    # Usage: hex_to_rgb "#FFFFFF"
    #        hex_to_rgb "000000"
    : "${1/\#}"
    ((r=16#${_:0:2},g=16#${_:2:2},b=16#${_:4:2}))
    printf '%s\n' "$r $g $b"
}
```

**Example Usage:**

```sh
$ hex_to_rgb "#FFFFFF"
255 255 255
```

## Convert an RGB color to hex

**Example Function:**

```sh
rgb_to_hex() {
    # Usage: rgb_to_hex "r" "g" "b"
    printf '#%02x%02x%02x\n' "$1" "$2" "$3"
}
```

**Example Usage:**

```sh
$ rgb_to_hex "255" "255" "255"
#FFFFFF
```

# Code Golf

## Shorter `for` loop syntax

```sh
# Tiny C Style.
for((;i++<10;)){ echo "$i";}

# Undocumented method.
for i in {1..10};{ echo "$i";}

# Expansion.
for i in {1..10}; do echo "$i"; done

# C Style.
for((i=0;i<=10;i++)); do echo "$i"; done
```

## Shorter infinite loops

```sh
# Normal method
while :; do echo hi; done

# Shorter
for((;;)){ echo hi;}
```

## Shorter function declaration

```sh
# Normal method
f(){ echo hi;}

# Using a subshell
f()(echo hi)

# Using arithmetic
# This can be used to assign integer values.
# Example: f a=1
#          f a++
f()(($1))

# Using tests, loops etc.
# NOTE: ‘while’, ‘until’, ‘case’, ‘(())’, ‘[[]]’ can also be used.
f()if true; then echo "$1"; fi
f()for i in "$@"; do echo "$i"; done
```

## Shorter `if` syntax

```sh
# One line
# Note: The 3rd statement may run when the 1st is true
[[ $var == hello ]] && echo hi || echo bye
[[ $var == hello ]] && { echo hi; echo there; } || echo bye

# Multi line (no else, single statement)
# Note: The exit status may not be the same as with an if statement
[[ $var == hello ]] &&
    echo hi

# Multi line (no else)
[[ $var == hello ]] && {
    echo hi
    # ...
}
```

## Simpler `case` statement to set variable

The `:` built-in can be used to avoid repeating `variable=` in a case statement. The `$_` variable stores the last argument of the last command. `:` always succeeds so it can be used to store the variable value.

```sh
# Modified snippet from Neofetch.
case "$OSTYPE" in
    "darwin"*)
        : "MacOS"
    ;;

    "linux"*)
        : "Linux"
    ;;

    *"bsd"* | "dragonfly" | "bitrig")
        : "BSD"
    ;;

    "cygwin" | "msys" | "win32")
        : "Windows"
    ;;

    *)
        printf '%s\n' "Unknown OS detected, aborting..." >&2
        exit 1
    ;;
esac

# Finally, set the variable.
os="$_"
```

<!-- CHAPTER END -->

<!-- CHAPTER START -->

# Misc

## Use `read` as an alternative to the `sleep` command

Surprisingly, `sleep` is an external command and not a `bash` built-in.

**NOTE:** Requires `bash` 4+

**Example Function:**

```sh
read_sleep() {
    # Usage: read_sleep 1
    #        read_sleep 0.2
    read -rt "$1" <> <(:) || :
}
```

**Example Usage:**

```sh
read_sleep 1
read_sleep 0.1
read_sleep 30
```

For performance-critical situations, where it is not economic to open and close an excessive number of file descriptors, the allocation of a file descriptor may be done only once for all invocations of `read`:

(See the generic original implementation at https://blog.dhampir.no/content/sleeping-without-a-subprocess-in-bash-and-how-to-sleep-forever)

```sh
exec {sleep_fd}<> <(:)
while some_quick_test; do
    # equivalent of sleep 0.001
    read -t 0.001 -u $sleep_fd
done
```

## Check if a program is in the user's PATH

```sh
# There are 3 ways to do this and either one can be used.
type -p executable_name &>/dev/null
hash executable_name &>/dev/null
command -v executable_name &>/dev/null

# As a test.
if type -p executable_name &>/dev/null; then
    # Program is in PATH.
fi

# Inverse.
if ! type -p executable_name &>/dev/null; then
    # Program is not in PATH.
fi

# Example (Exit early if program is not installed).
if ! type -p convert &>/dev/null; then
    printf '%s\n' "error: convert is not installed, exiting..."
    exit 1
fi
```

## Get the current date using `strftime`

Bash’s `printf` has a built-in method of getting the date which can be used in place of the `date` command.

**NOTE:** Requires `bash` 4+

**Example Function:**

```sh
date() {
    # Usage: date "format"
    # See: 'man strftime' for format.
    printf "%($1)T\\n" "-1"
}
```

**Example Usage:**

```sh
# Using above function.
$ date "%a %d %b  - %l:%M %p"
Fri 15 Jun  - 10:00 AM

# Using printf directly.
$ printf '%(%a %d %b  - %l:%M %p)T\n' "-1"
Fri 15 Jun  - 10:00 AM

# Assigning a variable using printf.
$ printf -v date '%(%a %d %b  - %l:%M %p)T\n' '-1'
$ printf '%s\n' "$date"
Fri 15 Jun  - 10:00 AM
```

## Get the username of the current user

**NOTE:** Requires `bash` 4.4+

```sh
$ : \\u
# Expand the parameter as if it were a prompt string.
$ printf '%s\n' "${_@P}"
black
```

## Generate a UUID V4

> **_CAVEAT:_** The generated value is not cryptographically secure.
{: .caveat }

**Example Function:**

```sh
uuid() {
    # Usage: uuid
    C="89ab"

    for ((N=0;N<16;++N)); do
        B="$((RANDOM%256))"

        case "$N" in
            6)  printf '4%x' "$((B%16))" ;;
            8)  printf '%c%x' "${C:$RANDOM%${#C}:1}" "$((B%16))" ;;

            3|5|7|9)
                printf '%02x-' "$B"
            ;;

            *)
                printf '%02x' "$B"
            ;;
        esac
    done

    printf '\n'
}
```

**Example Usage:**

```sh
$ uuid
d5b6c731-1310-4c24-9fe3-55d556d44374
```

## Progress bars

This is a simple way of drawing progress bars without needing a for loop
in the function itself.

**Example Function:**

```sh
bar() {
    # Usage: bar 1 10
    #            ^----- Elapsed Percentage (0-100).
    #               ^-- Total length in chars.
    ((elapsed=$1*$2/100))

    # Create the bar with spaces.
    printf -v prog  "%${elapsed}s"
    printf -v total "%$(($2-elapsed))s"

    printf '%s\r' "[${prog// /-}${total}]"
}
```

**Example Usage:**

```sh
for ((i=0;i<=100;i++)); do
    # Pure bash micro sleeps (for the example).
    (:;:) && (:;:) && (:;:) && (:;:) && (:;:)

    # Print the bar.
    bar "$i" "10"
done

printf '\n'
```

## Get the list of functions in a script

```sh
get_functions() {
    # Usage: get_functions
    IFS=$'\n' read -d "" -ra functions < <(declare -F)
    printf '%s\n' "${functions[@]//declare -f }"
}
```

## Bypass shell aliases

```sh
# alias
ls

# command
# shellcheck disable=SC1001
\ls
```

## Bypass shell functions

```sh
# function
ls

# command
command ls
```

## Run a command in the background

This will run the given command and keep it running, even after the terminal or SSH connection is terminated. All output is ignored.

```sh
bkr() {
    (nohup "$@" &>/dev/null &)
}

bkr ./some_script.sh # some_script.sh is now running in the background
```

## Capture the return value of a function without command substitution

**NOTE:** Requires `bash` 4+

This uses local namerefs to avoid using `var=$(some_func)` style command substitution for function output capture.

```sh
to_upper() {
  local -n ptr=${1}

  ptr=${ptr^^}
}

foo="bar"
to_upper foo
printf "%s\n" "${foo}" # BAR
```

<!-- CHAPTER END -->

<!-- CHAPTER START -->

# References and Further Reading

- [The Bash-Hackers Wiki](https://web.archive.org/web/20230406205817/https://wiki.bash-hackers.org/) The most human-readable documentation of any kind about Bash `ARCHIVED`
- [Bash beginner's mistakes](https://web.archive.org/web/20230330234404/https://wiki.bash-hackers.org/scripting/newbie_traps) by the Bash-Hackers Wiki `ARCHIVED`
- [Bash Guide](http://mywiki.wooledge.org/BashGuide) Excellent guide by Lhunath
- [Bash FAQ](http://mywiki.wooledge.org/BashFAQ) Lhunath answers many common questions
- [Bash Pitfalls](http://mywiki.wooledge.org/BashPitfalls) Discusses common pitfalls beginners fall into, and how to avoid them.
- [Bash manual](http://www.gnu.org/software/bash/manual/) Official GNU Bourne-Again Shell manual.
- [Bash FAQ](http://tiswww.case.edu/php/chet/bash/FAQ) by Chet Ramey
- [Advanced Bash-Scripting Guide](http://tldp.org/LDP/abs/html/) An in-depth exploration of the art of shell scripting
- [Bash Guide for Beginners](http://www.tldp.org/LDP/Bash-Beginners-Guide/html/) by Machtelt Garrels
- [Bash Programming - Intro/How-to](http://tldp.org/HOWTO/Bash-Prog-Intro-HOWTO.html#toc) Intermediate scripting by TLDP
- [bash-handbook](https://github.com/denysdovhan/bash-handbook) A handbook for those who want to learn Bash without diving in too deeply
- [Google's Shell Style Guide](https://google.github.io/styleguide/shellguide.html) Reasonable advice about code style
- [Sobell's Book](http://www.sobell.com/CR3/index.html) A practical guide to commands, editors, and shell programming
- [WikiBooks: Bash Shell Scripting](https://en.wikibooks.org/wiki/Bash_Shell_Scripting) All the citations
- [Use the Unofficial Bash Strict Mode](http://redsymbol.net/articles/unofficial-bash-strict-mode/) _"Unless You Looove Debugging"_
- [learnyoubash](https://github.com/denysdovhan/learnyoubash) Interactive workshop for using the shell and writing your the first bash script
- [Defensive BASH Programming](https://web.archive.org/web/20180917174959/http://www.kfirlavi.com/blog/2012/11/14/defensive-bash-programming) Defend your code from breakages while keeping it clean `ARCHIVED`
- [explainshell](https://explainshell.com) A website that breaks down shell commands, including their flags and options
- [Safe ways to do things in bash](https://github.com/anordal/shellharden/blob/master/how_to_do_things_safely_in_bash.md) by the shellharden team
{: #refs }

<!-- CHAPTER END -->
