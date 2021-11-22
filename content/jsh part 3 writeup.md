+++
tags = ["writeup", "tutorial"]
title = "jsh part 3"
index = "0010"
slug = "0010"
layout = "default"
+++

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

**Step 4.** moving onto actually dealing with the pipes for the processes, we need to add another field to our `io_config` structure from part 2. add an `int iopipe[2]` to your `io_config` structure. this is how we'll read input and write output to the pipes whenever we have to. in the initialization code for that struct, **make sure you call** `pipe(cfg->iopipe)` to actually create the pipe

**Step 5.** create a `Dllist` of `io_config` structures that we can iterate over along-side the `nargv` list. this should be as simple as doing something like
```c
Dllist cfgs = new_dllist();
Dllist tmp;
dll_traverse(tmp, nargv) {
  io_config *cfg = parse_io_ops((char**)tmp->val.v);
  dll_append(new_jval_v(cfg));
}
```

**Step 6.** this is actually where we do the piping! you should have a `Dllist` of `char**` pointers called `nargv` or something similar. each of those commands are a single command in a pipeline. first, add code to check if there's only one thing in the `Dllist`. **if so, do exactly the same thing as you did in part 2**. otherwise, you need to loop over the `Dllist` and open pipes for each command in the list before executing it. consider the following cases for each command:
```c
Dllist tmp;
Dllist tmp2 = dll_first(cfgs);
dll_traverse(tmp, nargv) {
  char **argv = tmp->val.v;

  // consider these cases
  
  // if tmp->blink is the head of the list
  // then we're at the very front of the list
  // if we're at the front of the list, then we ONLY have to open the output pipe
  // for the current process
  if (tmp->blink == nargv) {
    io_config *ops = tmp2->val.v;
    dup2(ops->iopipe[1], 1);
  }
  // if tmp->flink is the head of the list
  // then we're at the END of the list
  // so we only have to open the input pipe OF THE LAST PROCESS
  else if (tmp->flink == nargv) {
    io_config *preops = tmp2->blink->val.v;
    dup2(preops->iopipe[0], 0);
  }
  // otherwise, we need to open both pipes!
  else {
    io_config *ops = tmp2->val.v;
    io_config *preops = tmp2->blink->val.v;
    dup2(ops->iopipe[1], 1);
    dup2(preops->iopipe[0], 0);
  }
  tmp2 = dll_next(tmp2);

  // here, you also need to open redirects and close all pipes
  // see step 7 for closing all pipes
  // and finally execute the command!
}
```

**Step 7.** this is a very important step!! we **have to close all pipes in the child processes as well as the parent process**. just iterate over the `cfgs` list and call close on both ends of the pipe. that is `close(ops->iopipe[0]); close(ops->iopipe[1]);`

**Step 8.** add an integer to keep track of pid of the last process you spawned, and update it inside the `dll_traverse`. after that, move your wait code outside the `dll_traverse` and only wait on the last pid. if you've done all this correct (and i haven't forgotten anything in this writeup) you should have a working jsh part 3!