# Tomefile specification

This document describes the syntax, semantics and features of the Tomefile language.

Edition 1.1, 7 Jan 2026.

```bash
#!/bin/tome
# This is an example script inside of a file named `Tomefile`

:include ansi
:include log

printf "Hello World!\n"

:assert ${OS_ARCH:not:empty}
:match $OS_ARCH {
    :case "x64" {
        :goto build_x64
    }
    :case {
        :goto build
    }
}

:tome test "Running tests..." {
    ls ./test
    log_err* "not implemented"
    exit 1
}

:tome build "Building the project" {
    mkdir -p ./build
    # ...
}

:tome build_x64 {
    # ...
}
```

## Table of Contents

<!-- vim-markdown-toc GFM -->

* [1 Introduction](#1-introduction)
* [2 Definitions](#2-definitions)
            * [newline](#newline)
            * [identifier](#identifier)
            * [boolean](#boolean)
            * [exit code](#exit-code)
            * [filename](#filename)
            * [local scope](#local-scope)
* [3 Syntax](#3-syntax)
    * [3.1 Comment](#31-comment)
        * [3.1.1 Shebang](#311-shebang)
    * [3.2 Directive](#32-directive)
    * [3.3 Commands](#33-commands)
        * [3.3.1 Redirection](#331-redirection)
        * [3.3.2 Pipeline](#332-pipeline)
    * [3.4 Macro](#34-macro)
    * [3.5 Argument](#35-argument)
        * [3.5.1 Literal string](#351-literal-string)
        * [3.5.2 Interpolated string](#352-interpolated-string)
        * [3.5.3 Variable](#353-variable)
        * [3.5.4 Variable expansion](#354-variable-expansion)
            * [3.5.4.1 Modifier](#3541-modifier)
        * [3.5.5 Subcommand](#355-subcommand)
* [4 Features / Directives](#4-features--directives)
    * [4.1 Include](#41-include)
    * [4.2 Set](#42-set)
    * [4.3 Unset](#43-unset)
    * [4.4 Require](#44-require)
    * [4.5 Assert](#45-assert)
    * [4.6 Define](#46-define)
    * [4.7 Section](#47-section)
    * [4.8 Tome](#48-tome)
    * [4.9 If statement](#49-if-statement)
    * [4.10 Elif statement](#410-elif-statement)
    * [4.11 Else statement](#411-else-statement)
    * [4.12 Async](#412-async)
    * [4.13 Await](#413-await)
* [5 Features / Modifiers](#5-features--modifiers)
    * [5.1 Not](#51-not)
    * [5.2 To Lower](#52-to-lower)
    * [5.3 To Upper](#53-to-upper)
    * [5.4 To Snake](#54-to-snake)
    * [5.5 To Kebab](#55-to-kebab)
    * [5.6 To Camel](#56-to-camel)
    * [5.7 To Pascal](#57-to-pascal)
    * [5.8 To Delimited](#58-to-delimited)
    * [5.9 Exists](#59-exists)
    * [5.10 Is Empty](#510-is-empty)
    * [5.11 Is File](#511-is-file)
    * [5.12 Is Dir](#512-is-dir)
    * [5.13 Is Symlink](#513-is-symlink)
    * [5.14 Length](#514-length)
    * [5.15 Escape](#515-escape)
    * [5.16 Surround](#516-surround)
    * [5.17 Trim](#517-trim)
    * [5.18 Trim Prefix](#518-trim-prefix)
    * [5.19 Trim Suffix](#519-trim-suffix)
    * [5.20 Pad](#520-pad)
    * [5.21 Pad Left](#521-pad-left)
    * [5.22 Pad Right](#522-pad-right)
    * [5.23 Has Prefix](#523-has-prefix)
    * [5.24 Has Suffix](#524-has-suffix)
    * [5.25 Slice](#525-slice)
    * [5.26 Slice Fields](#526-slice-fields)
    * [5.27 Reverse](#527-reverse)
    * [5.28 Invert Case](#528-invert-case)
    * [5.29 Contains](#529-contains)

<!-- vim-markdown-toc -->

______________________________________________________________________

# 1 Introduction

Tomefile is a command language created for more elegant and intuitive automation (scripting). It is **not** compatible with other shell languages; instead, the syntax is designed to be primitive enough to be easily understood and used.

It was inspired by [Bash](<https://en.wikipedia.org/wiki/Bash_(Unix_shell)>) and [Make](<https://en.wikipedia.org/wiki/Make_(software)>) with the addition of [Tomes](<>) that allow defining multiple workflows within a single file.

This specification should be used as the definitive reference on the language behavior.

# 2 Definitions

#### newline

An LF character `\n`.

#### identifier

A string of characters that is made up of `a-z`, `A-Z`, `0-9`, `_`, `-`.

#### boolean

A string of characters that represents a binary `true` value (`true`, `TRUE`, or most commonly `1`) or a `false` value (`false`, `FALSE`, or most commonly `0`).

#### exit code

The value returned by a command to its caller. Any non-zero value indicates a failure.

#### filename

A string of characters used to identify a file.

#### local scope

A set of defined [variables](<>) and [macros](<>) inside of curly braces `{ ... }` or document.

# 3 Syntax

The script is made up of statements separated by a [newline](#newline) or a semicolon `;`. A backslash `\` can be used to escape those characters for multi-line statements.

```tome
statement 1; statement 2
one very long \
    statement that fits \
    multiple lines.

statement 4
```

There exist 4 types of statements: [Comment](<>), [Directive](<>), [Command](<>), and [Macro](<>).

## 3.1 Comment

```tome
# This is a comment
```

A comment causes all characters after `#` to be ignored during execution. It exists solely for the reader of the file or documentation tools.

### 3.1.1 Shebang

```bash
#!/bin/tome
```

The [Unix Shebang](<https://en.wikipedia.org/wiki/Shebang_(Unix)>) can be placed **at the very beginning of the script** to indicate which interpreter to use to execute the script in Unix-like systems.

## 3.2 Directive

```tome
:directive arg1 arg2 {
    # Body of the directive
}

:simple_directive arg1 arg2

:no_arg_directive
```

A directive begins with a `:` character followed by an [identifier](#identifier), optional [arguments](<>), and sometimes a [body](<>) `{ ... }`.

> [!NOTE]
> Refer to [Features / Directives](<>) for a detailed list of all existing directives.

## 3.3 Commands

```bash
printf* "Hello World!"
rm ./build  # do not continue if `rm` fails.
```

A command is a file path to an executable, often just the [filename](#filename) that can be located from the `$PATH` environment variable.

Commands can contain [arguments](<>), use [redirections](#331-redirection), or be a part of [pipeline](#331-pipeline).

An asterisk `*` can be appended to the command to prevent the script from exiting preemptively if the command returns a non-zero [exit code](#exit-code).

### 3.3.1 Redirection

```bash
# example of redirecting all streams from/into files:
# read from stdin.txt -> stdin
# stdout -> write to stdout.txt
# stderr -> write to stderr.txt

command < stdin.txt > stdout.txt >> stderr.txt
```

Any given command can redirect `stdin`, `stdout`, or `stderr` using `<`, `>`, and `>>` respectively, followed by a [filename](#filename).

### 3.3.2 Pipeline

```bash
# Pipe command_a into command_b
# Pipe stderr of command_b into command_c
command_a | command_b >>| command_c
```

A pipeline can be used to pipe output of one command into the input of another.

- `|` is used to pipe `stdout` of the left command into `stdin` of the right command.
- `>>|` is used to pipe `stderr` of the left command into `stdin` of the right command.

## 3.4 Macro

```tome
my_print! "Hello World!"
my_rm!* ./build  # continue if `rm` fails.
```

A macro is an [identifier](#identifier) that ends with an exclamation point `!`.

Calling a macro results in the interpreter going into the defined function body, creating a local scope context, executing, and then returning to the macro call.

> [!NOTE]
> Just like [commands](#33-commands):
>
> Macros can contain [arguments](<>), use [redirections](#331-redirection), or be a part of [pipeline](#331-pipeline).
>
> An asterisk `*` can be appended to the macro (after `!`) to prevent the script from exiting preemptively if the command returns a non-zero [exit code](#exit-code).

## 3.5 Argument

There exist ... types of arguments: ...

### 3.5.1 Literal string

```bash
'This is a literal string'
```

A string of characters contained within `'`. Any [newline](#newline) characters must be escaped with `\`. The string is kept as-is with no modifications.

### 3.5.2 Interpolated string

```bash
this_is_an_interpolated_string
"And so is this, with a $variable"
```

A string of characters optionally contained within `"`. The string is computed at runtime together with all [variables](<>), [variable expansions](<>), and [subcommands](<>) within.

> [!NOTE]
> Interpolated strings are always computed at runtime, even if their actual contents are static.
> This should **not** introduce any significant performance cost (depending on implementation), but it is good to keep that in mind.

### 3.5.3 Variable

```bash
$variable_name
```

A variable [identifier](#identifier) that comes from either [environment](https://en.wikipedia.org/wiki/Environment_variable) or [local scope](#local-scope).

It is computed at runtime, and **panics if the variable is undefined**. Use [variable expansion] with a `?` to prevent that behavior.

> [!NOTE]
> Every scope has the following additional variables exposed:
>
> - `$*` — All input arguments in the same string, space separated.
> - `$0` — The amount of input arguments.
> - `$1`, `$2` ... `$n` — The n-th input argument.

### 3.5.4 Variable expansion

```tome
printf ${variable_name} ${variable:not:empty} ${optional_var?}
```

Behaves like a [variable](#353-variable) but with applied modifiers.

Append `?` to the [identifier](#identifier) to make it optional. An optional variable returns an empty string instead of panicking.

[Modifiers](#3541-modifier) can be chained to transform the variable into desired format or apply basic logical operations on the value.

#### 3.5.4.1 Modifier

```tome
printf ${...:modifier 1: modifier 2}
```

A function that takes the input string and converts to output string.

It is used inside of [variable expansions](#3541-variable-expansion) by using the following format: `:[identifier]`.
Some modifiers accept [arguments](<>), which should be space separated after the identifier (e.g. `:[identifier] [arg1] [arg2]`).

> [!NOTE]
> Refer to [Features / Modifiers](<>) for a detailed list of all existing modifiers.

### 3.5.5 Subcommand

```tome
command_a $(command_b arg1 arg2)
```

A [command](<>) as part of the arguments list, placed between `$(` and `)`.

The `stdout` of the command will be read in-full and provided as a literal string.

# 4 Features / Directives

This chapter describes specific [directive](<>) [identifiers](#identifier), their meaning, and their behavior.

## 4.1 Include

Syntax: `:include [filepath]`

Placement: **anywhere**

```bash
:include @log  # /etc/tomefile/lib/log.tome on Unix-like systems

:include $dynamic_value  # NOTE: will panic if undefined

:include ${optional_value?}  # NOTE: will skip if undefined
```

Includes the specified file inside of the current document.

File paths that begin with an at-sign `@` are resolved as [standard library](<>) files.

## 4.2 Set

Syntax: `:set [identifier] [value]`

Placement: **anywhere**

```bash
:set welcome_message "Hello World!"
```

Sets a variable to the specified value. Can also be used to export [environment variables](https://en.wikipedia.org/wiki/Environment_variable) to be used by subprocesses.

## 4.3 Unset

Syntax: `:unset [identifier]`

Placement: **anywhere**

```bash
:unset welcome_message
```

Unsets a variable if it is set.

## 4.4 Require

Syntax: `:require [identifier]`

Placement: **anywhere**

```bash
:require input_arg1
:require input_arg2?
:require ...the_rest

# Analogous to (assuming it's at the beginning of the document)
:set input_arg1 $0
:set input_arg2 ${1?}
:set the_rest ${*:fields_slice 2 -1}
```

Requires that the next read positional input argument exists and sets the variable to the argument value.

A question mark `?` can be appended to the [identifier](#identifier) if the argument is optional.

## 4.5 Assert

Syntax: `:assert [variable]`, `:assert [variable] [==,!=] [value]`

Placement: **anywhere**

```bash
:assert $var
:assert $var == 123
```

Checks that the statement is true, panics if not. This is useful to ensure the correct state of the program to prevent unexpected behavior.

## 4.6 Define

Syntax: `:define [identifier] {...}`

Placement: **anywhere**

```bash
:define my_macro {
    # ...
}
```

Defines a [macro](<>) with its own local scope. Note that the language is interpreted, which means that definitions must come before they are referenced.

## 4.7 Section

Syntax: `:section [description] {...}`

Placement: **anywhere**

```bash
:section "Building the project..." {
    # ...
}
```

Creates a section of code that helps organize tasks with its own local scope.
Every section exports the `$TOME_SECTION` variable that is set to its description.

## 4.8 Tome

Syntax: `:tome [name] [description?] {...}`

Placement: **anywhere**

```bash
:tome build {
    # ...
}

:tome test "Running tests..." {
    # ...
}
```

Creates a tome with its own local scope. It acts as a separate file inside of the document.

An interpreter must have a way to run a specific tome using one of the following syntaxes:

- `[filename]@[tome]` — inline with the filename, allows for multiple filenames to be referenced.
- `[filename] -t [tome]` — as a short CLI option
- `[filename] --tome=[tome]` — as a long CLI option

## 4.9 If statement

Syntax: `:if [variable] {...}` or `:if [variable] [==,!=] [value]`

Placement: **anywhere**

```tome
:if ${path:is_file} {
    # ...
}

:if $path == "/home/tome" {
    # ...
}
```

Run the code inside of the body only if the statement is [true](#boolean).

## 4.10 Elif statement

Syntax: `:elif [variable] {...}` or `:elif [variable] [==,!=] [value]`

Placement: **only after an [if statement](<>)**

```tome
:if ${path:is_file} {}
:elif $path == "/home/tome" {
    # ...
}
```

Run the code inside of the body only if the previous statement returned [false](#boolean) and the current statement is [true](#boolean).

## 4.11 Else statement

Syntax: `:else {...}`

Placement: **only after an [if](<>) or [elif](<>) statement**

```tome
:if ${path:is_file} {}
:else {
    # ...
}
```

Run the code inside of the body only if all previous statements returned [false](#boolean).

## 4.12 Async

Syntax: `:async {...}` or `:async [identifier] {...}`

Placement: **anywhere**

```tome
:async {
    # ...
}

:async my_task {
    # ...
}
```

Run the code inside of the body asynchronously. Creates a named task group if `[identifier]` is provided.

## 4.13 Await

Syntax: `:await` or `:await [identifiers...]`

Placement: **anywhere**

```tome
:await

:await my_task1 my_task2
```

Wait for all or specified tasks to finish before continuing to the next line.

# 5 Features / Modifiers

This chapter describes specific modifier [identifiers](#identifier), their meaning, and their behavior.

## 5.1 Not

`:not` — inverts the result. Always gets applied last regardless of position. Any non-[boolean](#boolean) value returns [false](#boolean).

## 5.2 To Lower

`:to_lower` — transforms the string into all lowercase.

## 5.3 To Upper

`:to_upper` — transforms the string into all uppercase.

## 5.4 To Snake

`:to_snake` — transforms the string into [snake_case](https://en.wikipedia.org/wiki/Letter_case#Snake_case).

## 5.5 To Kebab

`:to_kebab` — transforms the string into [kebab-case](https://en.wikipedia.org/wiki/Letter_case#Kebab_case).

## 5.6 To Camel

`:to_camel` — transforms the string into [camelCase](https://en.wikipedia.org/wiki/Letter_case#Camel_case).

## 5.7 To Pascal

`:to_pascal` — transforms the string into [PascalCase](https://en.wikipedia.org/wiki/Letter_case#Camel_case).

## 5.8 To Delimited

`:to_delimited [char]` — transforms the string where words\* are separated with `[char]`.

- **word** depends on the input case i.e. PascalCase -> Pascal and Case, snake_case -> snake and case, my sentence -> my and sentence.

## 5.9 Exists

`:exists` — returns a [boolean](#boolean) based on whether the input string is a real path that points to an entry on the filesystem.

## 5.10 Is Empty

`:is_empty` — returns a [boolean](#boolean) based on whether the input string contains no characters.

## 5.11 Is File

`:is_file` — returns a [boolean](#boolean) based on whether the input string is a path to a file that exists on the filesystem.

## 5.12 Is Dir

`:is_dir` — returns a [boolean](#boolean) based on whether the input string is a path to a directory that exists on the filesystem.

## 5.13 Is Symlink

`:is_symlink` — returns a [boolean](#boolean) based on whether the input string is a path to a symlink that exists on the filesystem.

## 5.14 Length

`:length` — returns a number representing the length of the input string in unicode codepoints.

## 5.15 Escape

`:escape [chars...]` — returns the input string with specified characters escaped using `\`.

## 5.16 Surround

`:surround [string]` — returns the input string surrounded by `[string]` from both ends.

## 5.17 Trim

`:trim [string]` — returns the input string with `[string]` being trimmed (if possible) from both sides.

## 5.18 Trim Prefix

`:trim_prefix [string]` — returns the input string with `[string]` being trimmed (if possible) from the beginning.

## 5.19 Trim Suffix

`:trim_suffix [string]` — returns the input string with `[string]` being trimmed (if possible) from the end.

## 5.20 Pad

`:pad [number]` — returns the input string padded with `[number]` spaces from both sides.

## 5.21 Pad Left

`:pad_left [number]` — returns the input string padded with `[number]` spaces from the left side.

## 5.22 Pad Right

`:pad_right [number]` — returns the input string padded with `[number]` spaces from the righht side.

## 5.23 Has Prefix

`:has_prefix [string]` — returns a [boolean](#boolean) based on whether the input string begins with `[string]`.

## 5.24 Has Suffix

`:has_suffix [string]` — returns a [boolean](#boolean) based on whether the input string ends with `[string]`.

## 5.25 Slice

`:slice [from] [to]` — returns a slice of the input string starting from `[from]` to `[to]` both inclusive. An index can be negative to count from the end `-1 == last character`.

## 5.26 Slice Fields

`:slice_fields [from] [to]` — returns a slice of the input string as an array of space separated words starting from `[from]` to `[to]` both inclusive. An index can be negative to count from the end `-1 == last character`.

## 5.27 Reverse

`:reverse` — returns the input string reversed.

## 5.28 Invert Case

`:invert_case` — returns the input string with the case being inverted.

## 5.29 Contains

`:contains [strings...]` — returns a [boolean](#boolean) based on whether the input string contains all of `[strings...]`.
