+++
tags = ["writeup", "tutorial"]
title = "jsh part 1"
index = "0007"
slug = "0007"
layout = "default"
+++
for part 1, we just have to get a basic shell going on with support for the "&" operator.

**1.** focus first on getting input working. use the fields library to do this. make sure you pass `NULL` to `new_inputstruct` instead of `argv[1]` the relevant example is the second code block here on [this page](http://web.eecs.utk.edu/~jplank/plank/classes/cs360/360/notes/Fields/) after you're properly taking input, just make the program print out the input to the user after reading it

**2.** handle `argv`. here's the best way i can describe it:
```c
char *prompt = NULL;
if argc is 1 then
    prompt = strdup("jsh: ");
else if argc is 2 then
    if (argv[1] is "-") then
        set prompt = ""
    otherwise
        set prompt = strdup(argv[1])
else
    print not enough arguments and die
```

**3.** change your input loop to a `while(1)` and do something like this at the top of it:
```c
int glstatus = get_line(is);
if (glstatus <= 0) break;
if (is->NF == 0) continue;
printf("%s", prompt);
```

**4.** write a function called something like `create_argv` (i called mine `get_argv`) that basically just loops over `is->fields` and `strdup`s each field to a malloc'd array of strings. this is *very* similar to [forkcat1](http://web.eecs.utk.edu/~huangj/cs360/360/labs/lab7/forkcat1.c)

**5.** here's where we'll actually spawn a process. slightly modified from forkcat1
```c
int pid = fork();
if (pid == 0) { // child process
    char **argv = create_argv(is);
    execvp(argv[0], argv);
    perror(argv[0]);
    exit(1);
} else { // parent process
    // make sure you do this right!!
    // the reason we wait in a loop is beacuse 
    // wait(...) may end up grabbing a child we previously spawned
    // so we need to wait *again* to make sure we wait on the right child
    int waited_pid = wait(&status);
    while (waited_pid != pid) {
        waited_pid = wait(&status);
    }
}
```

**6.** almost done with part 1, we just need to make sure we're not waiting on lines that end with a `&`. to do this, just check if the last thing in `is->fields` is `"&"` and set a flag if it is. then in our parent process, we check our flag:
```c
if (wait_flag == true) {
    // wait here
}
```
you should be done with part 1 after you've finished all this. and you will pass the first 20 test cases.
NOTE: make sure that in your `create_argv` function, you're skipping over `"&"` when adding stuff to the malloc'd list