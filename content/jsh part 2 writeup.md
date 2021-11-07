+++
tags = ["writeup"]
title = "jsh part 2"
index = "0008"
slug = "0008"
layout = "default"
+++
for part 2, we have to add support for input/output redirection. this mean's we'll have to parse the operators <, >, and >>

**0.** i'm putting a step 0 in here since there's a modification you'll have to make to your part 1 code before you can get started on part 2. in your `create_argv` function, i assume you're skipping over the `"&"` symbol. make sure you're skipping over `"<"`, "`>`", and `">>"` too. you can do this the same way.

**1.** create a structure that looks *something* like this (you'll need to `#include <stdbool.h>` to get `bool`)
```c
typedef struct io_config {
    char *inputfile;
    char *outputfile;
    bool truncate;
} io_config;
```
we'll use this structure for storing information about where to redirect input and output to

**2.** write a function called something like `parse_io_ops(IS is, io_config *cfg)`. this function will be responsible for looping over `is->fields` and setting things in `cfg`. that loop will look like this:
```c
for (int i = 0; i < is->NF; ++i) {
    char *field = is->fields[i];
    if (strcmp(field, "<") == 0) {
        // input redirection
        // the lab doesn't check error handling. if you actually wanna do error handling, lemme know
        cfg->inputfile = strdup(is->fields[i + 1]);
        i += 1;
    } else if (strcmp(field, ">") == 0) {
        cfg->outputfile = strdup(is->fields[i + 1]);
        i += 1;
        cfg->truncate = true;
    } else if (strcmp(field, ">>") == 0) {
        cfg->outputfile = strdup(is->fields[i + 1]);
        i += 1;
        cfg->truncate = false;
    }
}
```

**3.** write another function called something like `do_redirect(io_config *cfg)` (i called mine `open_redirects`). this function will behave very similarly to [catf1f2](http://web.eecs.utk.edu/~huangj/cs360/360/notes/Dup/catf1f2.c). basically, check if `cfg->inputfile != NULL` and if it isn't `NULL`, then do the open, dup2, and close as in that link. for output, do the exact same thing **except for one thing**. if `cfg->truncate == true` then call open with `O_WRONLY | O_TRUNC | O_CREAT`, otherwise call it with `O_WRONLY | O_APPEND | O_CREAT`. do everything else as in the link and you'll be fine.

**4.** actually put everything in action: create an `io_config` in your read-loop in `main` and set `inputfile` and `outputfile` to `NULL` first. then, call your parse function from step **2**. then, in your if-branch where `pid == 0`, call the redirect function from step **3**. if you did all this right, and i didn't forget anything in this writeup, you should pass test cases up through 60. 