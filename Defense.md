# Defensive Coding in JS

## The eval Function: Dangers and Alternatives

The `eval()` function in JavaScript takes a string as an argument and executes it as Javascript source code. Consider the following examples:

```Javascript
// This user input causes an infinite loop to run
const user_input = "while(true) ;";
eval(user_input);
```

```Javascript
// This user input closes the application
const user_input = "process.exit(0)";
eval(user_input);
```

The functions, `setInterval()`, `setTimeout()`, and `new Function()` use `eval()` in their implementations, and should be used with the same caution.

We might be able to mitigate this risk with npm packages like `safe-eval` and `expression-eval`. Both allow us to limit which methods and properties are available to `eval()`. Strings passed to safe-eval must be an expression, not a statement. This prevents injected code from being executed. The code below, for example, will throw an error since it does not have access to the process object.

```Javascript
// Using safeEval will throw an error
const user_input = "process.exit(0)";
safeEval(user_input);
```

Take a look at their documentation for more examples!

Note: While packages like `safe-eval` may be safer than using `eval`, they may still contain vulnerabilities.

Best practices with `eval` are:

- Avoid using it altogether!
- If you must use it, use a safer version, and only allow trusted, non-user input.

You should always do your own research when exploring packages to use in your applications.
