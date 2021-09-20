# Leak report

Running valgrind on the initial implementation of this repo was complaining about these particular lines:

````c
char *result;
// some code here
result = calloc(size - num_spaces + 1, sizeof(char));
````

The problem was happening because memory allocated for variable result was allocated using `calloc` but was never freed, resulting in a memory leak.

One fix that I saw could solve the issue is to do the following:

1. Change the return type of function `strip()` to return a char instead of a pointer to a char.
2. Dereference variable `result` and save it to a variable to be returned.
3. Free up memory that was allocated for result.
4. Return the __value__ of result.

Consequently, I also had to modify line 71, `result = strcmp(str, cleaned);` to be `result = strcmp(str, &cleaned);` as the output from `strip()` is now a regular char and not a pointer to one.

This results in the following output from valgrind:

```
==6460== HEAP SUMMARY:
==6460==     in use at exit: 0 bytes in 0 blocks
==6460==   total heap usage: 7 allocs, 7 frees, 1,070 bytes allocated
==6460==
==6460== All heap blocks were freed -- no leaks are possible
```

And the expected output when running `./check_whitespace`:

```
The string 'Morris' is NOT clean.
The string '  stuff' is NOT clean.
The string 'Minnesota' is NOT clean.
The string 'nonsense  ' is NOT clean.
The string 'USA' is NOT clean.
The string '   ' is NOT clean.
The string '     silliness    ' is NOT clean.
```