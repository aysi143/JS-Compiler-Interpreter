
# Javascript Interpreter Project

A brief description of what this project does and who it's for

- JS-Interpreter is a sandboxed JavaScript interpreter written in JavaScript.
- It allows for execution of arbitrary JavaScript code line by line.
- Execution is completely isolated from the main JavaScript environment.
- Multiple instances of the JS-Interpreter allow for multi-threaded concurrent JavaScript without the use of Web Workers.



## Tokenizer

Here is how we went through

```// ***Tokenizer***

// Accept input string of code

function tokenizer(input) {
  // Tracks our position in the code
  let current = 0;

  // Tokens array to push tokens to
  let tokens = [];

  while (current < input.length) {
    //Store the current character
    let char = input[current];

    // Check for open parenthesis:
    if (char === "(") {
      tokens.push({
        type: "paren",
        value: "(",
      });
      // Increment Current
      current++;

      // Onto the next Character if one exists!
      continue;
    }

    //Check for a closing parenthesis, add a new token,
    // increment `current`, and `continue`.
    if (char === ")") {
      tokens.push({
        type: "paren",
        value: ")",
      });
      current++;
      continue;
    }
    // So here we're just going to test for existence of Whitespace and if it does exist we're
    // going to just `continue` on.
    let WHITESPACE = /\s/;
    if (WHITESPACE.test(char)) {
      current++;
      continue;
    }

    let NUMBERS = /[0-9]/;
    if (NUMBERS.test(char)) {
      // We're going to create a `value` string that we are going to push
      // characters to.
      let value = "";

      // Then we're going to loop through each character in the sequence until
      // we encounter a character that is not a number, pushing each character
      // that is a number to our `value` and incrementing `current` as we go.
      while (NUMBERS.test(char)) {
        value += char;
        char = input[++current];
      }

      // After that we push our `number` token to the `tokens` array.
      tokens.push({ type: "number", value });

      // And we continue on.
      continue;
    }

    // We'll also add support for strings in our language which will be any
    // text surrounded by double quotes (").
    //
    //   (concat "foo" "bar")
    //            ^^^   ^^^ string tokens
    //
    // We'll start by checking for the opening quote:
    if (char === '"') {
      // Keep a `value` variable for building up our string token.
      let value = "";

      // We'll skip the opening double quote in our token.
      char = input[++current];

      // Then we'll iterate through each character until we reach another
      // double quote.
      while (char !== '"') {
        value += char;
        char = input[++current];
      }

      // Skip the closing double quote.
      char = input[++current];

      // And add our `string` token to the `tokens` array.
      tokens.push({ type: "string", value });

      continue;
    }
    // The last type of token will be a `name` token. This is a sequence of
    // letters instead of numbers, that are the names of functions in our lisp
    // syntax.
    //
    //   (add 2 4)
    //    ^^^
    //    Name token
    //
    let LETTERS = /[a-z]/i;
    if (LETTERS.test(char)) {
      let value = "";

      // Again we're just going to loop through all the letters pushing them to
      // a value.
      while (LETTERS.test(char)) {
        value += char;
        char = input[++current];
      }

      // And pushing that value as a token with the type `name` and continuing.
      tokens.push({ type: "name", value });

      continue;
    }
    // Finally if we have not matched a character by now, we're going to throw
    // an error and completely exit.
    throw new TypeError("I dont know what this character is: " + char);
  }
  return tokens;
}

/**
 * ============================================================================
 *                                 ヽ/❀o ل͜ o\ﾉ
 *                                THE PARSER!!!
 * ============================================================================
 */

function parser(tokens) {
  // Again we keep a `current` variable that we will use as a cursor.
  let current = 0;

  // But this time we're going to use recursion instead of a `while` loop. So we
  // define a `walk` function.
  function walk() {
    // Inside the walk function we start by grabbing the `current` token.
    let token = tokens[current];

    // We're going to split each type of token off into a different code path,
    // starting off with `number` tokens.
    //
    // We test to see if we have a `number` token.
    if (token.type === "number") {
      // If we have one, we'll increment `current`.
      current++;

      // And we'll return a new AST node called `NumberLiteral` and setting its
      // value to the value of our token.
      return {
        type: "NumberLiteral",
        value: token.value,
      };
    }

    // If we have a string we will do the same as number and create a
    // `StringLiteral` node.
    if (token.type === "string") {
      current++;

      return {
        type: "StringLiteral",
        value: token.value,
      };
    }

    // Next we're going to look for CallExpressions. We start this off when we
    // encounter an open parenthesis.
    if (token.type === "paren" && token.value === "(") {
      // We'll increment `current` to skip the parenthesis since we don't care
      // about it in our AST.
      token = tokens[++current];

      // We create a base node with the type `CallExpression`, and we're going
      // to set the name as the current token's value since the next token after
      // the open parenthesis is the name of the function.
      let node = {
        type: "CallExpression",
        name: token.value,
        params: [],
      };

      // We increment `current` *again* to skip the name token.
      token = tokens[++current];

      while (
        token.type !== "paren" ||
        (token.type === "paren" && token.value !== ")")
      ) {
        // we'll call the `walk` function which will return a `node` and we'll
        // push it into our `node.params`.
        node.params.push(walk());
        token = tokens[current];
      }

      // Finally we will increment `current` one last time to skip the closing
      // parenthesis.
      current++;

      // And return the node.
      return node;
    }

    // Again, if we haven't recognized the token type by now we're going to
    // throw an error.
    throw new TypeError(token.type);
  }

  // Now, we're going to create our AST which will have a root which is a
  // `Program` node.
  let ast = {
    type: "Program",
    body: [],
  };

  // And we're going to kickstart our `walk` function, pushing nodes to our
  // `ast.body` array.
  //
  // The reason we are doing this inside a loop is because our program can have
  // `CallExpression` after one another instead of being nested.
  //
  //   (add 2 2)
  //   (subtract 4 2)
  //
  while (current < tokens.length) {
    ast.body.push(walk());
  }

  // At the end of our parser we'll return the AST.
  return ast;
}

```

## Usage

Start by including the two JavaScript source files:

   ``` <script src="acorn.js"></script>
    <script src="interpreter.js"></script>
  
  ```

Alternatively, use the compressed bundle (80kb):

 ``` <script src="acorn_interpreter.js"></script> ```
  
Next, instantiate an interpreter with the JavaScript code that needs to be parsed:

 ```   var myCode = 'var a=1; for(var i=0;i<4;i++){a*=i;} a;';
    var myInterpreter = new Interpreter(myCode); ```
  
Additional JavaScript code may be added at any time (frequently used to interactively call previously defined functions):

  ```  myInterpreter.appendCode('foo();'); ```
  
To run the code step by step, call the step function repeatedly until it returns false:

 ```   function nextStep() {
      if (myInterpreter.step()) {
        window.setTimeout(nextStep, 0);
      }
    }
    nextStep(); 
```
  
Alternatively, if the code is known to be safe from infinite loops, it may be executed to completion by calling the run function once:
```
    myInterpreter.run();
 ```
In cases where the code encounters asynchronous API calls (see below), run will return true if it is blocked and needs to be reexecuted at a later time.

/**
 * ============================================================================
 *                                 Jun 2022
 *                               Gobez Ethiopia
 * ============================================================================
 */
```
    
