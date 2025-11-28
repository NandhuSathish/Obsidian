
## Understanding Asynchronous Programming

JavaScript is **single-threaded**, meaning it can only execute one operation at a time. However, many operations (like API calls, file reading, timers) take time to complete. Asynchronous programming allows JavaScript to handle these time-consuming operations without blocking the main thread.

**Synchronous vs Asynchronous Example:**

```javascript
// Synchronous (blocking)
console.log('First');
console.log('Second');
console.log('Third');
// Output: First, Second, Third (in order)

// Asynchronous (non-blocking)
console.log('First');
setTimeout(() => console.log('Second'), 1000);
console.log('Third');
// Output: First, Third, Second (Third doesn't wait for setTimeout)
```

---

## 1. Callbacks

A **callback** is a function passed as an argument to another function, to be executed later when an operation completes.

### Basic Callback Example

```javascript
function fetchUser(userId, callback) {
  setTimeout(() => {
    const user = { id: userId, name: 'John Doe' };
    callback(user);
  }, 1000);
}

fetchUser(1, (user) => {
  console.log('User fetched:', user);
});
```

### Real-World Example: Sequential API Calls

```javascript
function getUser(userId, callback) {
  setTimeout(() => {
    callback({ id: userId, name: 'Alice' });
  }, 1000);
}

function getPosts(userId, callback) {
  setTimeout(() => {
    callback([
      { id: 1, title: 'Post 1' },
      { id: 2, title: 'Post 2' }
    ]);
  }, 1000);
}

function getComments(postId, callback) {
  setTimeout(() => {
    callback(['Comment 1', 'Comment 2']);
  }, 1000);
}

// Using callbacks - notice the nesting!
getUser(1, (user) => {
  console.log('User:', user);
  getPosts(user.id, (posts) => {
    console.log('Posts:', posts);
    getComments(posts[0].id, (comments) => {
      console.log('Comments:', comments);
    });
  });
});
```

### The Problem: Callback Hell (Pyramid of Doom)

When you have multiple asynchronous operations that depend on each other, callbacks lead to deeply nested code that's hard to read and maintain:

```javascript
getData((a) => {
  getMoreData(a, (b) => {
    getEvenMoreData(b, (c) => {
      getYetMoreData(c, (d) => {
        getFinalData(d, (e) => {
          console.log('Finally done!', e);
        });
      });
    });
  });
});
```

### Error Handling with Callbacks

```javascript
function fetchData(callback) {
  setTimeout(() => {
    const error = Math.random() > 0.5 ? null : new Error('Failed to fetch');
    const data = error ? null : { id: 1, value: 'data' };
    callback(error, data);
  }, 1000);
}

// Error-first callback pattern
fetchData((error, data) => {
  if (error) {
    console.error('Error:', error.message);
    return;
  }
  console.log('Success:', data);
});
```

---

## 2. Promises

**Promises** represent a value that may be available now, in the future, or never. They provide a cleaner way to handle asynchronous operations.

### Promise States

- **Pending**: Initial state, operation is ongoing
- **Fulfilled**: Operation completed successfully
- **Rejected**: Operation failed

### Creating a Promise

```javascript
const myPromise = new Promise((resolve, reject) => {
  setTimeout(() => {
    const success = true;
    if (success) {
      resolve('Operation successful!');
    } else {
      reject('Operation failed!');
    }
  }, 1000);
});

// Using the promise
myPromise
  .then((result) => {
    console.log(result); // 'Operation successful!'
  })
  .catch((error) => {
    console.error(error);
  });
```

### Converting Callbacks to Promises

```javascript
// Callback version
function getUserCallback(userId, callback) {
  setTimeout(() => {
    callback({ id: userId, name: 'Alice' });
  }, 1000);
}

// Promise version
function getUser(userId) {
  return new Promise((resolve, reject) => {
    setTimeout(() => {
      if (userId <= 0) {
        reject(new Error('Invalid user ID'));
      } else {
        resolve({ id: userId, name: 'Alice' });
      }
    }, 1000);
  });
}

// Using the promise
getUser(1)
  .then((user) => console.log('User:', user))
  .catch((error) => console.error('Error:', error));
```

### Promise Chaining

Promises can be chained to handle sequential asynchronous operations elegantly:

```javascript
function getUser(userId) {
  return new Promise((resolve) => {
    setTimeout(() => resolve({ id: userId, name: 'Alice' }), 1000);
  });
}

function getPosts(userId) {
  return new Promise((resolve) => {
    setTimeout(() => {
      resolve([
        { id: 1, title: 'Post 1' },
        { id: 2, title: 'Post 2' }
      ]);
    }, 1000);
  });
}

function getComments(postId) {
  return new Promise((resolve) => {
    setTimeout(() => resolve(['Comment 1', 'Comment 2']), 1000);
  });
}

// Clean promise chaining (no callback hell!)
getUser(1)
  .then((user) => {
    console.log('User:', user);
    return getPosts(user.id);
  })
  .then((posts) => {
    console.log('Posts:', posts);
    return getComments(posts[0].id);
  })
  .then((comments) => {
    console.log('Comments:', comments);
  })
  .catch((error) => {
    console.error('Error anywhere in the chain:', error);
  });
```

### Promise Static Methods

#### Promise.all() - Wait for all promises to resolve

```javascript
const promise1 = Promise.resolve(3);
const promise2 = new Promise((resolve) => setTimeout(() => resolve('foo'), 1000));
const promise3 = fetch('https://api.example.com/data');

Promise.all([promise1, promise2, promise3])
  .then((values) => {
    console.log(values); // [3, 'foo', Response]
  })
  .catch((error) => {
    console.error('One of the promises failed:', error);
  });
```

**Important**: If any promise rejects, `Promise.all()` immediately rejects.

#### Promise.allSettled() - Wait for all promises to complete

```javascript
const promises = [
  Promise.resolve('Success'),
  Promise.reject('Error'),
  Promise.resolve('Another success')
];

Promise.allSettled(promises).then((results) => {
  console.log(results);
  // [
  //   { status: 'fulfilled', value: 'Success' },
  //   { status: 'rejected', reason: 'Error' },
  //   { status: 'fulfilled', value: 'Another success' }
  // ]
});
```

#### Promise.race() - First to complete wins

```javascript
const slow = new Promise((resolve) => setTimeout(() => resolve('Slow'), 3000));
const fast = new Promise((resolve) => setTimeout(() => resolve('Fast'), 1000));

Promise.race([slow, fast]).then((result) => {
  console.log(result); // 'Fast'
});
```

#### Promise.any() - First to fulfill wins

```javascript
const promises = [
  Promise.reject('Error 1'),
  new Promise((resolve) => setTimeout(() => resolve('Success'), 1000)),
  Promise.reject('Error 2')
];

Promise.any(promises)
  .then((result) => {
    console.log(result); // 'Success'
  })
  .catch((error) => {
    console.error('All promises rejected');
  });
```

---

## 3. Async/Await

**Async/Await** is syntactic sugar built on top of Promises, making asynchronous code look and behave more like synchronous code.

### Basic Syntax

```javascript
// Define an async function
async function fetchData() {
  return 'Data fetched!';
}

// Async functions always return a promise
fetchData().then((result) => console.log(result)); // 'Data fetched!'
```

### Using await

The `await` keyword can only be used inside `async` functions. It pauses execution until the promise resolves.

```javascript
function delay(ms) {
  return new Promise((resolve) => setTimeout(resolve, ms));
}

async function example() {
  console.log('Starting...');
  await delay(2000); // Wait for 2 seconds
  console.log('2 seconds later...');
}

example();
```

### Rewriting Promise Chains with Async/Await

```javascript
// Promise chain version
function fetchUserData() {
  return getUser(1)
    .then((user) => {
      console.log('User:', user);
      return getPosts(user.id);
    })
    .then((posts) => {
      console.log('Posts:', posts);
      return getComments(posts[0].id);
    })
    .then((comments) => {
      console.log('Comments:', comments);
    })
    .catch((error) => {
      console.error('Error:', error);
    });
}

// Async/await version (much cleaner!)
async function fetchUserData() {
  try {
    const user = await getUser(1);
    console.log('User:', user);

    const posts = await getPosts(user.id);
    console.log('Posts:', posts);

    const comments = await getComments(posts[0].id);
    console.log('Comments:', comments);
  } catch (error) {
    console.error('Error:', error);
  }
}
```

### Error Handling with Try/Catch

```javascript
async function fetchData() {
  try {
    const response = await fetch('https://api.example.com/data');
    if (!response.ok) {
      throw new Error(`HTTP error! status: ${response.status}`);
    }
    const data = await response.json();
    return data;
  } catch (error) {
    console.error('Fetch failed:', error);
    throw error; // Re-throw if you want calling code to handle it
  }
}
```

### Parallel Execution with Async/Await

```javascript
// Sequential (slower - 6 seconds total)
async function sequentialFetch() {
  const user = await getUser(1); // Wait 2 seconds
  const posts = await getPosts(1); // Wait 2 seconds
  const comments = await getComments(1); // Wait 2 seconds
  return { user, posts, comments };
}

// Parallel (faster - 2 seconds total)
async function parallelFetch() {
  const [user, posts, comments] = await Promise.all([
    getUser(1),
    getPosts(1),
    getComments(1)
  ]);
  return { user, posts, comments };
}
```

### Real-World Example: API Call

```javascript
async function getUserProfile(userId) {
  try {
    // Fetch user data
    const userResponse = await fetch(`https://api.example.com/users/${userId}`);
    const user = await userResponse.json();

    // Fetch user's posts and followers in parallel
    const [postsResponse, followersResponse] = await Promise.all([
      fetch(`https://api.example.com/users/${userId}/posts`),
      fetch(`https://api.example.com/users/${userId}/followers`)
    ]);

    const posts = await postsResponse.json();
    const followers = await followersResponse.json();

    return {
      user,
      posts,
      followers,
      summary: `${user.name} has ${posts.length} posts and ${followers.length} followers`
    };
  } catch (error) {
    console.error('Failed to fetch user profile:', error);
    throw error;
  }
}

// Usage
getUserProfile(123)
  .then((profile) => console.log(profile))
  .catch((error) => console.error(error));
```

### Common Pitfalls and Best Practices

#### ❌ Don't forget await

```javascript
async function bad() {
  const data = fetchData(); // Missing await! Returns a Promise, not the data
  console.log(data); // Promise { <pending> }
}

async function good() {
  const data = await fetchData(); // Correct!
  console.log(data); // Actual data
}
```

#### ❌ Don't use await in loops unnecessarily

```javascript
// Bad - Sequential (slow)
async function processUsers(userIds) {
  const users = [];
  for (const id of userIds) {
    const user = await getUser(id); // Waits for each one
    users.push(user);
  }
  return users;
}

// Good - Parallel (fast)
async function processUsers(userIds) {
  const promises = userIds.map((id) => getUser(id));
  const users = await Promise.all(promises);
  return users;
}
```

#### ✅ Always handle errors

```javascript
// Using try/catch
async function safeOperation() {
  try {
    const result = await riskyOperation();
    return result;
  } catch (error) {
    console.error('Operation failed:', error);
    return null; // or throw, depending on your needs
  }
}

// Using .catch()
async function anotherWay() {
  const result = await riskyOperation().catch((error) => {
    console.error('Operation failed:', error);
    return null;
  });
  return result;
}
```

---

## Comparison Summary

|Feature|Callbacks|Promises|Async/Await|
|---|---|---|---|
|**Readability**|Poor (nested)|Good (chained)|Excellent (linear)|
|**Error Handling**|Error-first pattern|.catch()|try/catch|
|**Composability**|Difficult|Good|Excellent|
|**Debugging**|Hard|Moderate|Easy|
|**Browser Support**|All|Modern|Modern (ES2017+)|

---

## When to Use What?

- **Callbacks**: Simple one-off asynchronous operations, or when working with older libraries
- **Promises**: When you need to chain multiple async operations or use Promise utilities (Promise.all, etc.)
- **Async/Await**: Default choice for modern JavaScript - clearest syntax, easiest to read and maintain

---

## Practice Exercise

Try converting this callback-based code to both Promises and Async/Await:

```javascript
function fetchUserOrders(userId, callback) {
  setTimeout(() => {
    console.log('Fetching user...');
    const user = { id: userId, name: 'John' };
    
    setTimeout(() => {
      console.log('Fetching orders...');
      const orders = [
        { id: 1, item: 'Book', price: 20 },
        { id: 2, item: 'Pen', price: 5 }
      ];
      
      callback({ user, orders });
    }, 1000);
  }, 1000);
}

// Usage
fetchUserOrders(1, (result) => {
  console.log('Result:', result);
});
```

**Challenge**: Rewrite this using Promises, then using Async/Await!

---

## Additional Resources

- [MDN - Asynchronous JavaScript](https://developer.mozilla.org/en-US/docs/Learn/JavaScript/Asynchronous)
- [JavaScript.info - Promises](https://javascript.info/promise-basics)
- [MDN - async function](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Statements/async_function)