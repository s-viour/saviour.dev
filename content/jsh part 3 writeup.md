+++
tags = ["writeup", "tutorial"]
title = "jsh part 3 (in progress)"
index = "0010"
slug = "0010"
layout = "default"
+++
**this writeup is still in progress! i'll update it when i finish the program**

in parts 2 and 3, the user could only enter one command at a time. in part 3, they can **chain commands** with the pipe operator (`|`). this means we'll have to create a **list** of `argv` lists and `io_config` structures

**Step 1.** in `main.c`, right after you read from the `IS`. create a `Dllist` called something like `argvs`. this list will store each invididual `argv` list for each command. create another `Dllist` called `current_argv` or something like that as well as an `int current_argc = 0`. then, iterate over `is->fields` and do the following:

**if you see a pipe character**:\
turn `current_argv` into a `char**` by doing something like:
```c
// i wrote this code for this article, and it is untested
// it should work fine, but i may have made a mistake
char** nargv = malloc(sizeof(char *) * (current_argc + 1));
Dllist tmp;
int i = 0;
dll_traverse(tmp, current_argv) {
  nargv[i++] = strdup(tmp->val.s);
}
nargv[current_argc] = NULL;
```
and then free memory (optional), clear `current_argv` and `current_argc`, and append `nargv` to `argvs`.

**otherwise**\
`dll_append` the current field we're looking at to `current_argv`

make sure you test that `nargv` holds the correct values before continuing! i did this by doing this by printing out the commands i'd parsed and then immediately `exit(0)`'d

**Step 2.** after you're sure you're parsing commands correctly, that is, if i were to enter `echo hello | head -n 1` you'd end up with two commands: `echo hello` and `head -n 1`. modify your `create_argv` function to take a `char**` instead of an `IS` so you can pass one of your `argv`s to it

**Step 3.** do the same thing you did in (2) for your `parse_io_ops` function. test steps 2 and 3 by traversing your `argvs` list and calling each function on the entry. if all values are good, then you're parsing correctly!