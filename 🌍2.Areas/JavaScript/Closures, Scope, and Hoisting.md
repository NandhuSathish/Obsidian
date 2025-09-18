---
tags:
  - javascript
---
#### Refferance videos

-   [Namaste JavaScript Episode 1 - 12](https://www.youtube.com/playlist?list=PLlasXeu85E9cQ32gLCvAvr9vNaUccPVNP)
-   [Lydia Hallie](https://www.youtube.com/@theavocoder)
-   [40 Days Of JavaScript](https://www.youtube.com/watch?v=t8QXF85YovE&list=PLIJrr73KDmRw2Fwwjt6cPC_tk5vcSICCu)

## [[Interview Q&A Closures, Scope, Hoisting|Interview Q&A]]

---

## Scope

Scope determines where variables and functions are accessible in your code. Think of it as "visibility rules" for your variables.

### Types of Scope

#### 1. Global Scope

Variables declared outside any function or block have global scope.

```javascript
var globalVar = "I'm global!";
let globalLet = "Me too!";

function anyFunction() {
    console.log(globalVar); // ✅ Accessible
    console.log(globalLet); // ✅ Accessible
}
```

#### 2. Function Scope

Variables declared inside a function are only accessible within that function.

```javascript
function myFunction() {
    var functionScoped = "Only inside this function";
    
    function innerFunction() {
        console.log(functionScoped); // ✅ Accessible (parent scope)
    }
    
    innerFunction();
}

console.log(functionScoped); // ❌ ReferenceError: functionScoped is not defined
```

#### 3. Block Scope (ES6+)

Variables declared with `let` and `const` inside `{}` are block-scoped.

```javascript
if (true) {
    var varVariable = "I'm function scoped";
    let letVariable = "I'm block scoped";
    const constVariable = "I'm also block scoped";
}

console.log(varVariable);    // ✅ "I'm function scoped"
console.log(letVariable);    // ❌ ReferenceError
console.log(constVariable);  // ❌ ReferenceError
```

### Lexical Scoping

JavaScript uses lexical scoping, meaning inner functions have access to variables in their outer scope.

```javascript
function outerFunction(x) {
    // Outer scope
    
    function innerFunction(y) {
        // Inner scope
        console.log(x + y); // Can access 'x' from outer scope
    }
    
    return innerFunction;
}

const addFive = outerFunction(5);
addFive(3); // Output: 8
```

---

## Hoisting

Hoisting is JavaScript's behavior of moving variable and function declarations to the top of their scope during compilation.

### Variable Hoisting

#### `var` Hoisting

```javascript
console.log(myVar); // undefined (not ReferenceError!)
var myVar = 5;

// This is how JavaScript interprets it:
// var myVar; // Declaration hoisted
// console.log(myVar); // undefined
// myVar = 5; // Assignment stays in place
```

#### `let` and `const` Hoisting

```javascript
console.log(myLet); // ❌ ReferenceError: Cannot access 'myLet' before initialization
let myLet = 5;

console.log(myConst); // ❌ ReferenceError: Cannot access 'myConst' before initialization  
const myConst = 10;
```

**Key Point**: `let` and `const` are hoisted but placed in a "temporal dead zone" until their declaration is reached.

### Function Hoisting

#### Function Declarations

```javascript
sayHello(); // ✅ "Hello!" - Works because of hoisting

function sayHello() {
    console.log("Hello!");
}
```

#### Function Expressions

```javascript
sayHi(); // ❌ TypeError: sayHi is not a function

var sayHi = function() {
    console.log("Hi!");
};

// Arrow functions behave the same way
greet(); // ❌ ReferenceError: Cannot access 'greet' before initialization
const greet = () => console.log("Greetings!");
```

### Practical Example: Understanding the Temporal Dead Zone

```javascript
function example() {
    console.log(a); // undefined (var hoisting)
    console.log(b); // ❌ ReferenceError (temporal dead zone)
    
    var a = 1;
    let b = 2;
}
```

---

## Closures

A closure is formed when an inner function has access to variables from its outer function's scope, even after the outer function has finished executing.

### Basic Closure Example

```javascript
function outerFunction(x) {
    // This is the outer function's scope
    
    return function innerFunction(y) {
        // This inner function has access to 'x'
        return x + y;
    };
}

const addTen = outerFunction(10);
console.log(addTen(5)); // 15

// Even though outerFunction finished executing,
// the inner function still remembers 'x'
```

### Why Closures Work

When `outerFunction` executes, it creates a new execution context. The inner function "closes over" variables from the outer scope, keeping them alive even after the outer function returns.

### Practical Closure Examples

#### 1. Private Variables (Data Encapsulation)

```javascript
function createCounter() {
    let count = 0; // Private variable
    
    return {
        increment: function() {
            count++;
            return count;
        },
        decrement: function() {
            count--;
            return count;
        },
        getCount: function() {
            return count;
        }
    };
}

const counter = createCounter();
console.log(counter.increment()); // 1
console.log(counter.increment()); // 2
console.log(counter.getCount());  // 2

// count is not directly accessible
console.log(counter.count); // undefined
```

#### 2. Function Factory

```javascript
function createMultiplier(multiplier) {
    return function(number) {
        return number * multiplier;
    };
}

const double = createMultiplier(2);
const triple = createMultiplier(3);

console.log(double(5)); // 10
console.log(triple(5)); // 15
```

#### 3. Module Pattern

```javascript
const userModule = (function() {
    // Private variables and functions
    let users = [];
    
    function validateUser(user) {
        return user.name && user.email;
    }
    
    // Public API
    return {
        addUser: function(user) {
            if (validateUser(user)) {
                users.push(user);
                return true;
            }
            return false;
        },
        
        getUsers: function() {
            return [...users]; // Return a copy
        },
        
        getUserCount: function() {
            return users.length;
        }
    };
})();

userModule.addUser({name: "John", email: "john@email.com"});
console.log(userModule.getUserCount()); // 1
```

---

## Real-World Applications

### 1. Event Handlers with Closures

```javascript
function attachListeners() {
    const buttons = document.querySelectorAll('.btn');
    
    for (let i = 0; i < buttons.length; i++) {
        buttons[i].addEventListener('click', function() {
            console.log(`Button ${i} clicked`); // ✅ Works correctly with 'let'
        });
    }
    
    // With 'var', this would log the final value of 'i' for all buttons
    // for (var i = 0; i < buttons.length; i++) {
    //     buttons[i].addEventListener('click', function() {
    //         console.log(`Button ${i} clicked`); // ❌ Always logs final 'i' value
    //     });
    // }
}
```

### 2. Debouncing with Closures

```javascript
function debounce(func, delay) {
    let timeoutId;
    
    return function(...args) {
        clearTimeout(timeoutId);
        
        timeoutId = setTimeout(() => {
            func.apply(this, args);
        }, delay);
    };
}

const debouncedSearch = debounce(function(query) {
    console.log('Searching for:', query);
}, 300);

// Usage
debouncedSearch('apple');
debouncedSearch('apple pie'); // Only this will execute after 300ms
```

### 3. Memoization

```javascript
function memoize(fn) {
    const cache = {};
    
    return function(...args) {
        const key = JSON.stringify(args);
        
        if (key in cache) {
            console.log('Cache hit!');
            return cache[key];
        }
        
        console.log('Computing...');
        const result = fn.apply(this, args);
        cache[key] = result;
        return result;
    };
}

const expensiveFunction = memoize(function(n) {
    // Simulate expensive computation
    let result = 0;
    for (let i = 0; i < n; i++) {
        result += i;
    }
    return result;
});

console.log(expensiveFunction(1000)); // Computing... then result
console.log(expensiveFunction(1000)); // Cache hit! then result
```

---

## Common Pitfalls & Best Practices

### 1. The Classic Loop Problem

**Problem:**

```javascript
for (var i = 0; i < 3; i++) {
    setTimeout(function() {
        console.log(i); // Logs: 3, 3, 3 (not 0, 1, 2)
    }, 100);
}
```

**Solutions:**

**Using `let` (ES6+):**

```javascript
for (let i = 0; i < 3; i++) {
    setTimeout(function() {
        console.log(i); // Logs: 0, 1, 2
    }, 100);
}
```

**Using IIFE (Immediately Invoked Function Expression):**

```javascript
for (var i = 0; i < 3; i++) {
    (function(index) {
        setTimeout(function() {
            console.log(index); // Logs: 0, 1, 2
        }, 100);
    })(i);
}
```

### 2. Memory Leaks with Closures

**Be careful with:**

```javascript
function problematicClosure() {
    const largeData = new Array(1000000).fill('data');
    
    return function() {
        // If this function keeps a reference to largeData
        // but doesn't use it, it still won't be garbage collected
        console.log('Hello');
    };
}
```

**Better approach:**

```javascript
function betterClosure() {
    const largeData = new Array(1000000).fill('data');
    const neededData = largeData.slice(0, 10); // Only keep what you need
    
    return function() {
        console.log(neededData[0]);
    };
    // largeData can now be garbage collected
}
```

### 3. Understanding `this` in Closures

```javascript
const obj = {
    name: 'MyObject',
    
    regularFunction: function() {
        console.log(this.name); // 'MyObject'
        
        const innerFunction = function() {
            console.log(this.name); // undefined (this refers to global object)
        };
        innerFunction();
    },
    
    arrowFunction: function() {
        console.log(this.name); // 'MyObject'
        
        const innerArrow = () => {
            console.log(this.name); // 'MyObject' (arrow function inherits this)
        };
        innerArrow();
    }
};
```

### Best Practices

1. **Use `const` and `let`** instead of `var` to avoid hoisting confusion
2. **Be mindful of memory usage** in closures
3. **Use arrow functions** when you want to preserve `this` context
4. **Consider using modules** instead of closures for large applications
5. **Use closures for data privacy** and creating specialized functions

---

## Summary

- **Scope** determines where variables are accessible (global, function, block)
- **Hoisting** moves declarations to the top of their scope (`var` vs `let/const` behave differently)
- **Closures** allow inner functions to access outer function variables even after the outer function returns
- These concepts work together to create powerful patterns like data encapsulation, function factories, and modules

Understanding these fundamentals will help you write more predictable JavaScript code and debug issues more effectively!