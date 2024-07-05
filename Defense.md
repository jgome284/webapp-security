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

## The exec method: Dangers and Alternatives

In this exercise, we will discuss the exec() method, its risks, and alternatives.

The exec() method takes a string as an argument and runs it as a shell command, enabling shell syntax within JavaScript. The danger is that unrestricted commands can access, modify, and delete files. For example:

```Javascript
user_input = "cat *.js";
exec(user_input);
```

exec(), combined with the user input above, allows an attacker to print out all the JavaScript files in the current directory.

The execFile() method is an alternative that works similarly to exec() but requires separation of the commands and its arguments. This prevents piped commands and path variable access. Consider the following example:

```Javascript
import { exec, execFile } from "child_process";

// Spawns a shell with the input as is
exec("ls -lah /tmp");

// Requires a command and specified arguments to execute
execFile("ls", ["-lah", "/tmp"]);
```

The arguments for the command ls must be separated in the execFile() method call. This separation ensures that an attacker cannot inject their malicious commands. Whereas exec will allow for additional unintended commands in the input, execFile will detect an error.

## Dangers and Secure Use of the fs Module

The file system, or fs, module in Node.js enables file system operations. It gives us access to methods like:

- chmod() to change file permissions
- mkdir() to create directories
- rmdir() to delete directories
- And many more

The fs module coupled with improperly sanitized user input gives attackers access to our entire file system and exposes it to path traversal and file inclusion vulnerabilities. Take a look at the code below:

```Javascript
const user_input = "./example.txt";
fs.unlinkSync(user_input);
```

The above example uses `unlink()` to delete the file defined by the user’s input. It could be any file on our system! To mitigate the risk, we can tweak our code to restrict traversal scope to a directory of our choice:

```Javascript
const user_input = "example.txt"
const root_directory = process.cwd();   // Hard-code path to restrict scope
const filePath = path.join(root_directory , fileName);
fs.unlinkSync(filePath);
```

We use the `join()` method of the path variable to combine our desired directory with the user-provided file name. Hiding the directory the server is operating on makes it tougher to reach valuable information.

## Dangers and Secure Use of Regular Expressions

Regular Expressions are used in almost every single programming language to validate whether user input adheres to an expected condition. Attackers can make use of insecure regex expressions to trigger a [Regular expression Denial of Service (ReDoS)](https://owasp.org/www-community/attacks/Regular_expression_Denial_of_Service_-_ReDoS).

The RegEx engine can take a large amount of time on poorly defined Regex expressions. Consider the RegEx ([0-9]+)+\#. The table below shows how the number of backtracking steps increases exponentially as the location of the unmatched character increases.

| String                 | Number of Digits | Number of Steps |
| ---------------------- | ---------------- | --------------- |
| 123#                   | 3                | 6               |
| 123456789123456789...# | 180              | 6               |
| 1c#                    | 1                | 5               |
| 1234567o#              | 7                | 755             |
| 123456789123456d#      | 15               | 196587          |
| 1234567891234567e#     | 16               | TIMEOUT ERROR   |

To prevent this danger, we can use the validator npm package. It provides a library of string validators and sanitizers for things like IP addresses, emails, and phone numbers. For Regex expression we must write ourselves, we can use tools like the safe-regex npm package to detect dangerous regular expressions.

We encourage you to take a look at some examples in the safe-regex documentation.

## Secure Your Code:

### Strict Mode

Now that you have learned about some dangerous functions and regular expressions to avoid, let’s learn about some defensive tools. One of them is JavaScript’s strict mode. Using [strict mode](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Strict_mode) throws errors that would otherwise be silent, which can help reveal vulnerabilities. To invoke strict mode, simply put "use strict"; in single or double quotes on top of your JavaScript file.

For example, strict mode catches assignments to undefined variables:

```Javascript
// Runs fine without strict mode
x = "codecademy";
```

```Javascript
// Throws “ReferenceError” with strict mode
"use strict";
x = "codecademy";
```

```Javascript
// Runs fine with strict mode if the variable is declared with let, var, or const
"use strict";
var x = "codecademy";
```

We can use strict mode to find the errors in our workspace code!

### Static Code Analysis

Static Code Analysis evaluates a code without executing it. A lint, or linter, is a static code analysis tool used to improve source code by finding and flagging programming errors, bugs, and patterns that may compromise security. Some of the most popular JavaScript linters are:

- ESLint
- JSLint
- JSHint

We can customize the linter rules to fit our needs using configuration files or third-party plugins. `eslint-plugin-security` is a plugin that adds rules to detect several security vulnerabilities.
