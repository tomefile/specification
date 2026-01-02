# Tomefile specification

This document describes the syntax and features of Tomefile (edition 1.0, 02 Jan 2026).

## Table of Contents

<!-- vim-markdown-toc GFM -->

* [1 Introduction](#1-introduction)
* [2 Definitions](#2-definitions)
* [3 Syntax](#3-syntax)
    * [3.1 Comments](#31-comments)
        * [3.1.1 Generic comments](#311-generic-comments)
        * [3.1.2 Shebang](#312-shebang)
    * [3.2 Directives](#32-directives)
        * [3.2.1 Examples](#321-examples)
    * [3.3 Commands](#33-commands)
        * [3.3.1 Redirections](#331-redirections)
        * [3.3.2 Pipelines](#332-pipelines)
    * [3.4 Macros](#34-macros)
    * [3.5 Arguments](#35-arguments)
        * [3.5.1 Literal string](#351-literal-string)
        * [3.5.2 Variable string](#352-variable-string)
            * [3.5.2.1 Literal segment](#3521-literal-segment)
            * [3.5.2.2 Variable segment](#3522-variable-segment)
        * [3.5.3 Subcommand](#353-subcommand)
* [4 Features](#4-features)
    * [4.1 Includes](#41-includes)
    * [4.2 Tomes](#42-tomes)
    * [4.3 Sections](#43-sections)
    * [4.4 Asserts](#44-asserts)
    * [4.5 If statements](#45-if-statements)
    * [4.5 Else statements](#45-else-statements)
    * [4.6 Else-if statements](#46-else-if-statements)
    * [4.7 For loops](#47-for-loops)
    * [4.8 Async blocks](#48-async-blocks)
    * [4.9 Await](#49-await)
    * [4.10 Define](#410-define)
    * [4.11 Require](#411-require)
    * [4.12 Set](#412-set)
    * [4.13 Unset](#413-unset)
* [5 Addendum A - Not production ready](#5-addendum-a---not-production-ready)

<!-- vim-markdown-toc -->

______________________________________________________________________

# 1 Introduction

Tomefile is a command language created for more elegant and intuitive automation (scripting). It is **not** compatible with other shell languages; instead, the syntax is designed to be primitive enough to easily understand and use.

It was inspired by [Bash](<>) and [Make](<>) with the addition of [Tomes](<>) that allow defining multiple workflows within a single file.

# 2 Definitions

These are non-standard definitions used in this manual.

- `Name charset`: Set of characters that contains all Latin characters `a-z, A-Z`, all Arabic numerals `0-9`, underscore `_`, and hyphen-minus `-`.

- `True value`: A value that represents `true`. Can be `true`, `TRUE`, or, most commonly, `1`.

- `False value`: A value that represents `false`. Can be `false`, `FALSE`, or, most commonly, `0`.

# 3 Syntax

The script is written and read left-to-right, top-to-bottom. There exist 3 top-level types of statements: [Comments](#21-comments), [Commands](<>), and [Directives](<>).

All statements end with a newline character (`LF (\n)`), unless a backslash `\` is used.

Semicolons `;` may also be used to terminate statements.

```tome
this is a statement \
    and this is still a part \
    of the statement above

however this is not.
```

## 3.1 Comments

### 3.1.1 Generic comments

All comments begin with a `#` character, whether that be at the beginning of a line or after another statement. Comments will **not** affect the result of execution and exist solely for documentation, developer notes, and temporary safe removal of code.

### 3.1.2 Shebang

[Unix Shebang](<https://en.wikipedia.org/wiki/Shebang_(Unix)>) can be placed **at the beginning of the script** (the first line) to indicate which interpreter to use to execute the script in Unix-like systems.

It is recommended to use the following shebang for all of your scripts to indicate to both the system and other developers how to execute the script:

```bash
#!/bin/tome
```

## 3.2 Directives

All directives begin with a `:` character followed by a name using **Name charset**, optional arguments, and sometimes a body `{ ... }`.

### 3.2.1 Examples

```tome
:directive arg1 arg2 {
    # Body of the directive
}

:simple_directive arg1 arg2
```

## 3.3 Commands

Everything that doesn't match a [Directive](<>) or a [Comment](<>) is treated as a command within the `$PATH` environment variable to be executed.

Commands can contain arguments, use [Redirections](<>), or be a part of a [Pipeline](<>).

An exclamation point `!` can be appended to the identifier to make the script panic if the command returns a non-zero status code.

```tome
my_command! arg1 arg2
```

### 3.3.1 Redirections

Any given command can redirect `stdin`, `stdout`, and `stderr` using `<`, `>`, and `>>` respectfully. Redirection can only happen from and into a file path.

```tome
# This will read stdin.txt file and feed it as stdin into `command`.
# All stdout will be written to stdout.txt
# Any errors will be written to stderr.txt

command < stdin.txt > stdout.txt >> stderr.txt
```

### 3.3.2 Pipelines

A pipeline allows the program to pipe stdout of one command into stdin of another using the pipe `|` character.

Stderr can be piped into stdin by using the `>>|` character sequence.

```tome
# This will redirect stdout of `command_a` into stdin of `command_b`
# And redirect stderr of `command_b` into stdin of `command_c`
command_a | command_b >>| command_c
```

## 3.4 Macros

A macro uses similar syntax to [Commands](<>), with the addition of parentheses `()` after the command identifier.

Macros call functions that are defined in the script.

An exclamation point `!` can be appended to the identifier to make the script panic if the command returns a non-zero status code.

```tome
this_is_a_macro() arg1 arg2
```

## 3.5 Arguments

Both [Commands](<>) and [Directives](<>) can receive space-separated (` `) arguments of various types.

### 3.5.1 Literal string

A literal string is surrounded by the single-quotation character `'` from both sides. The contents will be provided as-is with no modifications.

```tome
'This is a literal string argument'
```

### 3.5.2 Variable string

An interpolated string made out of segments. It can optionally be surrounded with the double-quotation character `"` to allow for the usage of spaces.

#### 3.5.2.1 Literal segment

A string that will be left as-is with no modifications.

#### 3.5.2.2 Variable segment

A lazily evaluated variable with optionally applied modifiers.

```tome
$simple_variable
${optional_variable?}
${variable_with_modifiers:to_lower:trim_prefix "My prefix"}
```

A question mark `?` can be appended to the name to make the variable optional (resolve as an empty string when undefined).

Optional [Variable Modifiers](<>) can be chained within the variable expansion `:<identifier> <args?>`.

### 3.5.3 Subcommand

A subcommand is a [Command](<>) or a [Macro](<>) within an argument list. The stdout will be provided to the parent command as a [Literal string](<>) argument. The syntax is as follows: `$(...)`, exclamation points can be used to make the parent command fail if the subcommand returns a non-zero status code `$!()`.

```tome
my_command $(my_subcommand arg1 arg2)
my_command $!(my_subcommand if it fails the execution of the parent command is aborted)
```

# 4 Features

This chapter describes specific identifiers, their meaning, and their behavior.

## 4.1 Includes

An include [Directive](<>) is used to interpret the contents of the specified file and merge it with the current. It can be placed anywhere in the script, and the file name does not have to be known at parse time.

File paths prepended with an at-sign `@` are resolved as [Standard Library](<>) files.

```tome
:include @log  # /etc/tomefile/lib/log.tome on Unix-like systems

:include $dynamic_value  # NOTE: will panic if undefined

:include ${optional_value}  # NOTE: will skip if undefined
```

## 4.2 Tomes

A tome is a [Directive](<>) that defines an isolated section of the script which will only be executed if called specifically. It makes it possible to create dynamic workflows based on the input to the interpreter.

```tome
:tome build {
    # ...
}

:tome test "Running all tests, but not building" {
    # ...
}
```

Note that when executing a specific tome, only the contents of the tome and nothing else will be executed.

## 4.3 Sections

A section is a [Directive](<>) that defines a related region of code. It does not modify the code flow; instead, it prints a top-level log message and exports a variable to allow for all inner log messages to be printed as nested.

```tome
:include @log

:section "Hello World" {
    log_info() "This will be indented to show that it's nested"
}
```

## 4.4 Asserts

An assert [Directive](<>) checks that all arguments are equal to a [Definitions / True value](<>) and then panics if not. Used to ensure the environment state is correct to prevent unexpected behavior.

```tome
:assert ${build_dir:is_dir}  # Will make sure build_dir both exists and is a valid directory path
```

## 4.5 If statements

An if statement [Directive](<>) allows to execute the body only if all conditions are a [Definitions / True value](<>).

Subsequent [Else statements](<>) or [Else-if statements](<>) are allowed.

```tome
:if ${build_file:not:is_dir} {
    # ...
}
```

## 4.5 Else statements

An else statement [Directive](<>) must always go after an [If statement](<>). It will execute only if the previous statement has not met the conditions.

```tome
:if ${build_file:not:is_dir} {
    # ...
}
:else {
    # ...
}
```

## 4.6 Else-if statements

An else-if (written as `elif`) [Directive](<>) must always go after an [If statement](<>). It will conditionally execute if the previous statement has not met the conditions.

```tome
:if ${build_file:not:is_dir} {
    # ...
}
:elif ${tmp_file:is_file} {
    # ...
}
```

## 4.7 For loops

A for loop is used to iterate over space-separated values. The first argument must always be a [Variable name](<>) followed by an equals sign `=` and a value to iterate over.

```tome
:for $file = $(ls .) {
    echo $file
}
```

## 4.8 Async blocks

An async [Directive](<>) is used to run a chunk of code asynchronously from the rest of the program. An optional argument can be provided to assign an identifier to the task.

The script will always wait until all async blocks have finished execution.

```tome
:async {
    # ...
}

:async my_task {
    # ...
}
```

Note that tasks can share the same name, uniting them into groups.

## 4.9 Await

An await [Directive](<>) is used to wait for all of a specific task to complete. One is always explicitly included at the end of each script.

Waiting means that the script will be halted until the task(s) is/are finished.

```tome
:await  # Wait for all to finish

:async my_task {}

:await my_task my_task2  # Wait for specific tasks to finish
```

Note that tasks can share the same name, uniting them into groups.

## 4.10 Define

A define [Directive](<>) is used to define [Macros](<>). It requires an [Identifier](<>) argument and a body.

```tome
:define log_info {
    printf "INFO: %s" $text
}
```

## 4.11 Require

A require [Directive](<>) is used to require a specific input argument for a script/macro. It assigns the values in order and panics if no argument is provided.

It can be used at any point in the program. However, it is recommended to use it on the top of a function/script to make it clear what kinds of arguments are required.

A question mark `?` can be appended to the identifier if the argument is optional.

An ellipsis `...` can be prepended to the identifier if it should be variadic of 1 or more arguments. **NOTE: Can only have one variadic argument**

```tome
:define log {
    :require level    # the first argument
    :require ...text  # all next arguments space-separated
}

log() INFO "text"  # will work
log() INFO         # will fail
```

Note that positional arguments can also be referenced as `$0 (length of arguments)`, `$* (all arguments)`, `$1`, `$2`, etc.

## 4.12 Set

Sets a variable to the specified value.

```tome
:set entries $(ls .)
:set message "Hello World"

echo $entries $message
```

## 4.13 Unset

Unsets a variable if it's set.

```tome
:unset entries
:unset message
```

# 5 Addendum A - Not production ready

The language is still in development. I wrote this document after a lot of work had already been done, realizing a lot of mistakes I had made during the initial prototyping. All that to say that existing tools MIGHT be out of date with this document. I will remove this addendum once Tomefile becomes production-ready.
