
## Basic Level Questions

### Q1: What is asynchronous programming and why do we need it in JavaScript?

**Answer:**

Great question! So, JavaScript is single-threaded, which means it can only do one thing at a time. Imagine you're at a restaurant - if the waiter had to wait for the kitchen to finish cooking before taking any other orders, that would be terrible service, right?

Asynchronous programming is like having a smart waiter who takes your order, gives it to the kitchen, and while your food is being prepared, they continue serving other customers. They don't just stand there waiting.

In JavaScript terms:

```javascript
// Synchronous - blocking
console.log('Order taken');
// Imagine this takes 5 seconds
cookFood(); // Everything stops here
console.log('Next customer');

// Asynchronous - non-blocking
console.log('Order taken');
cookFood(() => console.log('Food ready!')); // Goes to background
console.log('Next customer'); // This runs immediately!
```

We need async programming for operations like:

- Fetching data from APIs (network calls)
- Reading files
- Database queries
- Timers and animations

Without it, your entire application would freeze waiting for these operations to complete, creating a terrible user experience.

---

### Q2: Can you explain what a callback is with a real-world example?

**Answer:**

Absolutely! A callback is simply a function that you pass to another function to be executed later when something finishes.

Think of it like this: You order pizza and give them your phone number. You don't wait at the store - you go home and do other things. When the pizza is ready, they **call you back**. That's exactly what a callback does!

```javascript
// Real-world pizza example
function orderPizza(flavor, callback) {
  console.log(`Ordering ${flavor} pizza...`);
  
  // Simulating pizza preparation time
  setTimeout(() => {
    console.log('Pizza is ready!');
    callback(flavor); // "Calling you back" when pizza is done
  }, 3000);
}

// You provide what to do when pizza arrives
orderPizza('Pepperoni', (flavor) => {
  console.log(`Enjoying my ${flavor} pizza! 🍕`);
});

console.log('Meanwhile, watching Netflix...');

// Output:
// Ordering Pepperoni pizza...
// Meanwhile, watching Netflix...
// (3 seconds later)
// Pizza is ready!
// Enjoying my Pepperoni pizza! 🍕
```

The callback function is your "instruction" for what to do when the async operation completes. It's a fundamental pattern that Promises and async/await are built upon.

---

### Q3: What is "Callback Hell" and why is it a problem?

**Answer:**

Ah, callback hell! Also known as the "Pyramid of Doom." It's when you have multiple asynchronous operations that depend on each other, creating deeply nested code that looks like a pyramid tipping over.

Let me show you why it's problematic:

```javascript
// Callback Hell example
getUser(userId, (user) => {
  getProfile(user.id, (profile) => {
    getPosts(profile.id, (posts) => {
      getComments(posts[0].id, (comments) => {
        getAuthor(comments[0].authorId, (author) => {
          console.log('Finally got the author!', author);
          // At this point, you're 5 levels deep!
        });
      });
    });
  });
});
```

**Problems with this approach:**

1. **Readability**: It's hard to follow what's happening
2. **Maintainability**: Changing or debugging this code is a nightmare
3. **Error Handling**: You need to handle errors at each level
4. **Code grows horizontally**: Instead of down, it grows to the right

Here's the same logic with Promises - much cleaner:

```javascript
// Same logic with Promises - no pyramid!
getUser(userId)
  .then(user => getProfile(user.id))
  .then(profile => getPosts(profile.id))
  .then(posts => getComments(posts[0].id))
  .then(comments => getAuthor(comments[0].authorId))
  .then(author => console.log('Got the author!', author))
  .catch(error => console.error('Error anywhere in chain:', error));
```

This is one of the main reasons Promises were introduced - to flatten the pyramid and make async code more readable.

---

## Intermediate Level Questions

### Q4: What is a Promise and what are its states? Explain with an example.

**Answer:**

A Promise is like a placeholder for a value that we don't have yet but will have in the future. Think of it as a receipt you get when you order something online - it's a promise that your item will arrive.

**A Promise has three states:**

1. **Pending**: The operation is still running (your package is being shipped)
2. **Fulfilled**: The operation succeeded (package delivered! ✅)
3. **Rejected**: The operation failed (delivery failed ❌)

Here's a practical example:

```javascript
function checkWeather(city) {
  return new Promise((resolve, reject) => {
    console.log(`Checking weather for ${city}...`);
    
    setTimeout(() => {
      const weather = Math.random() > 0.2 ? 'Sunny' : null;
      
      if (weather) {
        resolve(`${city} is ${weather} today! 🌞`); // Success
      } else {
        reject(`Couldn't fetch weather for ${city} 😞`); // Failure
      }
    }, 2000);
  });
}

// Using the Promise
checkWeather('New York')
  .then(result => {
    console.log('Success:', result);
  })
  .catch(error => {
    console.error('Failed:', error);
  })
  .finally(() => {
    console.log('Weather check completed!');
  });
```

**Why Promises are powerful:**

- They make async code look synchronous and easy to read
- Single error handling point with `.catch()`
- You can chain multiple operations cleanly
- They're the foundation for async/await syntax

---

### Q5: Explain Promise chaining and why it's better than nested callbacks.

**Answer:**

Promise chaining is when you link multiple asynchronous operations together in a sequence, where each step depends on the previous one. The magic is that each `.then()` returns a new Promise, allowing you to chain them.

Let me show you with a real scenario - ordering food online:

```javascript
function selectRestaurant() {
  return new Promise(resolve => {
    setTimeout(() => {
      console.log('1. Restaurant selected: Pizza Palace');
      resolve('Pizza Palace');
    }, 1000);
  });
}

function placeOrder(restaurant) {
  return new Promise(resolve => {
    setTimeout(() => {
      console.log('2. Order placed at', restaurant);
      resolve({ orderId: 123, restaurant });
    }, 1000);
  });
}

function trackDelivery(order) {
  return new Promise(resolve => {
    setTimeout(() => {
      console.log('3. Tracking delivery for order', order.orderId);
      resolve({ ...order, status: 'Delivered' });
    }, 1000);
  });
}

// Promise chaining - clean and readable!
selectRestaurant()
  .then(restaurant => placeOrder(restaurant))
  .then(order => trackDelivery(order))
  .then(finalOrder => {
    console.log('4. Enjoy your meal!', finalOrder);
  })
  .catch(error => {
    console.error('Something went wrong:', error);
  });
```

**Why this is better than callbacks:**

1. **Linear flow**: Reads top to bottom, not nested
2. **Single error handler**: One `.catch()` catches errors from any step
3. **Cleaner**: Each step is at the same indentation level
4. **Easy to modify**: Want to add a step? Just add another `.then()`

**Pro tip**: Always return something in your `.then()` handlers to keep the chain going!

---

### Q6: What's the difference between Promise.all(), Promise.race(), Promise.allSettled(), and Promise.any()?

**Answer:**

Great question! These are like different strategies for handling multiple Promises. Let me explain with a sports analogy:

**1. Promise.all() - Team Race** Everyone must finish successfully for the team to win. If anyone fails, everyone fails.

```javascript
// Like a relay race - everyone must complete
const promise1 = Promise.resolve('Runner 1 finished');
const promise2 = Promise.resolve('Runner 2 finished');
const promise3 = Promise.resolve('Runner 3 finished');

Promise.all([promise1, promise2, promise3])
  .then(results => {
    console.log('All finished:', results);
    // ['Runner 1 finished', 'Runner 2 finished', 'Runner 3 finished']
  })
  .catch(error => {
    console.log('One person failed, team loses:', error);
  });

// Use case: Loading multiple critical resources
Promise.all([fetchUser(), fetchPosts(), fetchComments()])
  .then(([user, posts, comments]) => {
    console.log('All data loaded!', { user, posts, comments });
  });
```

**2. Promise.race() - Sprint Race** First person to cross the finish line (or fall) determines the outcome.

```javascript
// First to complete wins
const slow = new Promise(resolve => 
  setTimeout(() => resolve('Slow runner'), 5000)
);
const fast = new Promise(resolve => 
  setTimeout(() => resolve('Fast runner'), 1000)
);

Promise.race([slow, fast])
  .then(winner => {
    console.log('Winner:', winner); // 'Fast runner'
  });

// Use case: Request with timeout
Promise.race([
  fetch('https://api.example.com/data'),
  new Promise((_, reject) => 
    setTimeout(() => reject('Timeout!'), 5000)
  )
])
  .then(data => console.log('Got data:', data))
  .catch(err => console.log('Failed or timed out:', err));
```

**3. Promise.allSettled() - Everyone Gets a Medal** Wait for everyone to finish, regardless of success or failure. You get all results.

```javascript
const promises = [
  Promise.resolve('Success 1'),
  Promise.reject('Failed 1'),
  Promise.resolve('Success 2'),
  Promise.reject('Failed 2')
];

Promise.allSettled(promises)
  .then(results => {
    console.log('All attempts completed:', results);
    // [
    //   { status: 'fulfilled', value: 'Success 1' },
    //   { status: 'rejected', reason: 'Failed 1' },
    //   { status: 'fulfilled', value: 'Success 2' },
    //   { status: 'rejected', reason: 'Failed 2' }
    // ]
    
    // Great for showing which operations succeeded
    results.forEach((result, index) => {
      if (result.status === 'fulfilled') {
        console.log(`Task ${index + 1} succeeded:`, result.value);
      } else {
        console.log(`Task ${index + 1} failed:`, result.reason);
      }
    });
  });

// Use case: Batch operations where you need all results
```

**4. Promise.any() - First Success Wins** First successful completion wins. Failures are ignored unless all fail.

```javascript
const server1 = fetch('https://api1.example.com').catch(() => 
  Promise.reject('Server 1 down')
);
const server2 = fetch('https://api2.example.com').catch(() => 
  Promise.reject('Server 2 down')
);
const server3 = fetch('https://api3.example.com');

Promise.any([server1, server2, server3])
  .then(response => {
    console.log('Got response from fastest working server:', response);
  })
  .catch(error => {
    console.log('All servers failed:', error);
  });

// Use case: Multiple backup servers, use whoever responds first
```

**Quick Comparison:**

|Method|Resolves When|Rejects When|Use Case|
|---|---|---|---|
|`Promise.all()`|All succeed|Any fails|All data is critical|
|`Promise.race()`|First settles|First settles|Timeout scenarios|
|`Promise.allSettled()`|All settle|Never|Need all results|
|`Promise.any()`|First succeeds|All fail|Redundant resources|

---

## Advanced Level Questions

### Q7: What is async/await and how does it improve upon Promises?

**Answer:**

Async/await is syntactic sugar built on top of Promises that makes asynchronous code look and behave like synchronous code. It's like going from writing in English to writing in your native language - much more natural!

**The key insight**: Async functions always return a Promise, and `await` pauses execution until the Promise resolves.

Here's the evolution:

```javascript
// 1. Callback version (old school)
function getUserData(userId, callback) {
  fetchUser(userId, (user) => {
    fetchPosts(user.id, (posts) => {
      fetchComments(posts[0].id, (comments) => {
        callback({ user, posts, comments });
      });
    });
  });
}

// 2. Promise version (better)
function getUserData(userId) {
  return fetchUser(userId)
    .then(user => fetchPosts(user.id)
      .then(posts => fetchComments(posts[0].id)
        .then(comments => ({ user, posts, comments }))
      )
    );
}

// 3. Async/Await version (cleanest!)
async function getUserData(userId) {
  const user = await fetchUser(userId);
  const posts = await fetchPosts(user.id);
  const comments = await fetchComments(posts[0].id);
  return { user, posts, comments };
}
```

**Why async/await is better:**

1. **Reads like synchronous code**: Top to bottom, no mental gymnastics
2. **Error handling with try/catch**: Familiar pattern
3. **Easier debugging**: Stack traces make sense
4. **Less cognitive load**: No chains, no callbacks

**Real-world example:**

```javascript
// Making a cup of coffee
async function makeCoffee() {
  try {
    console.log('Starting coffee maker...');
    
    // These operations must happen in sequence
    await boilWater(); // Wait for water to boil
    console.log('Water boiled!');
    
    await grindBeans(); // Then grind beans
    console.log('Beans ground!');
    
    await brewCoffee(); // Finally brew
    console.log('Coffee ready! ☕');
    
    return 'Delicious coffee';
  } catch (error) {
    console.error('Coffee making failed:', error);
    return 'No coffee today 😢';
  }
}

// Usage is simple
makeCoffee().then(result => console.log(result));
```

**Pro tip**: The keyword `await` literally makes JavaScript wait there, but it doesn't block the entire thread - other code can still run!

---

### Q8: How do you handle errors in async/await? Show different approaches.

**Answer:**

Error handling in async/await is one of its best features because we can use the familiar try/catch pattern. But there are several approaches, each useful in different scenarios.

**Approach 1: Try/Catch Block (Most Common)**

```javascript
async function fetchUserProfile(userId) {
  try {
    const user = await fetch(`/api/users/${userId}`);
    const userData = await user.json();
    
    const posts = await fetch(`/api/posts/${userId}`);
    const postsData = await posts.json();
    
    return { user: userData, posts: postsData };
  } catch (error) {
    console.error('Failed to fetch profile:', error);
    
    // You can handle specific errors
    if (error.message.includes('404')) {
      throw new Error('User not found');
    }
    
    throw error; // Re-throw if you want caller to handle it
  }
}
```

**Approach 2: Multiple Try/Catch Blocks (Granular Control)**

```javascript
async function processPayment(orderId) {
  let order, payment, invoice;
  
  // Each step has its own error handling
  try {
    order = await fetchOrder(orderId);
  } catch (error) {
    console.error('Failed to fetch order:', error);
    return { success: false, reason: 'Order not found' };
  }
  
  try {
    payment = await processCharge(order.amount);
  } catch (error) {
    console.error('Payment failed:', error);
    return { success: false, reason: 'Payment declined' };
  }
  
  try {
    invoice = await generateInvoice(order, payment);
  } catch (error) {
    console.error('Invoice generation failed:', error);
    // Payment succeeded, but invoice failed - might need refund
    await refundPayment(payment.id);
    return { success: false, reason: 'Invoice error, payment refunded' };
  }
  
  return { success: true, invoice };
}
```

**Approach 3: .catch() Method (Mixing Styles)**

```javascript
async function getUserData(userId) {
  // Catch error inline without try/catch
  const user = await fetchUser(userId)
    .catch(error => {
      console.error('User fetch failed:', error);
      return null; // Return default value
    });
  
  if (!user) {
    return { error: 'User not found' };
  }
  
  const posts = await fetchPosts(user.id)
    .catch(() => []); // Return empty array on error
  
  return { user, posts };
}
```

**Approach 4: Higher-Order Function for Error Handling (DRY)**

```javascript
// Utility function - use everywhere!
function asyncHandler(fn) {
  return async function(...args) {
    try {
      return await fn(...args);
    } catch (error) {
      console.error('Async error:', error);
      // Log to error service
      // logToSentry(error);
      throw error;
    }
  };
}

// Use it to wrap async functions
const fetchUserSafe = asyncHandler(async (userId) => {
  const response = await fetch(`/api/users/${userId}`);
  return response.json();
});

// Now you don't need try/catch everywhere
const user = await fetchUserSafe(123);
```

**Approach 5: Promise.allSettled for Parallel Operations**

```javascript
async function loadDashboard(userId) {
  // Run multiple operations, some might fail
  const results = await Promise.allSettled([
    fetchUser(userId),
    fetchPosts(userId),
    fetchNotifications(userId),
    fetchSettings(userId)
  ]);
  
  const [user, posts, notifications, settings] = results;
  
  return {
    user: user.status === 'fulfilled' ? user.value : null,
    posts: posts.status === 'fulfilled' ? posts.value : [],
    notifications: notifications.status === 'fulfilled' ? notifications.value : [],
    settings: settings.status === 'fulfilled' ? settings.value : {}
  };
  
  // Dashboard still works even if some parts fail!
}
```

**Best Practices:**

1. Always handle errors - don't let them silently fail
2. Use specific error messages for debugging
3. Consider what should happen on failure (default values, retries, user notification)
4. Don't overuse try/catch - only where you can actually handle the error
5. Log errors for monitoring in production

---

### Q9: What's the difference between sequential and parallel async operations? When would you use each?

**Answer:**

This is a crucial question because the difference can make your app 10x faster or slower! Let me break it down:

**Sequential**: One operation waits for the previous to complete **Parallel**: Multiple operations run simultaneously

**Sequential Example:**

```javascript
async function makeBreakfastSlow() {
  console.time('breakfast');
  
  await toastBread();      // Takes 3 seconds
  await boilEgg();         // Takes 5 seconds
  await brewCoffee();      // Takes 4 seconds
  
  console.timeEnd('breakfast'); // Total: 12 seconds! 😱
  
  return 'Breakfast ready';
}
```

**Parallel Example:**

```javascript
async function makeBreakfastFast() {
  console.time('breakfast');
  
  // Start all operations at the same time
  const [toast, egg, coffee] = await Promise.all([
    toastBread(),      // Takes 3 seconds
    boilEgg(),         // Takes 5 seconds  
    brewCoffee()       // Takes 4 seconds
  ]);
  
  console.timeEnd('breakfast'); // Total: 5 seconds! 🚀
  // (The longest operation, boiling egg)
  
  return 'Breakfast ready';
}
```

**Real-World API Example:**

```javascript
// ❌ BAD - Sequential (slow!)
async function loadUserDashboardSlow(userId) {
  const user = await fetchUser(userId);           // 200ms
  const posts = await fetchPosts(userId);         // 300ms
  const friends = await fetchFriends(userId);     // 250ms
  const notifications = await fetchNotifications(userId); // 150ms
  
  // Total time: 900ms
  return { user, posts, friends, notifications };
}

// ✅ GOOD - Parallel (fast!)
async function loadUserDashboardFast(userId) {
  const [user, posts, friends, notifications] = await Promise.all([
    fetchUser(userId),
    fetchPosts(userId),
    fetchFriends(userId),
    fetchNotifications(userId)
  ]);
  
  // Total time: 300ms (longest request)
  return { user, posts, friends, notifications };
}
```

**When to Use Sequential:**

1. **When operations depend on each other:**

```javascript
async function orderProcess() {
  const order = await createOrder();      // Need order first
  const payment = await processPayment(order.id); // Need order ID
  const invoice = await generateInvoice(payment.id); // Need payment ID
  return invoice;
}
```

2. **When order matters:**

```javascript
async function deployApp() {
  await buildApp();    // Build first
  await runTests();    // Then test
  await deployToServer(); // Finally deploy
}
```

3. **Rate limiting or avoiding overwhelming a service:**

```javascript
async function processLargeDataset(items) {
  const results = [];
  for (const item of items) {
    const result = await processItem(item); // One at a time
    results.push(result);
  }
  return results;
}
```

**When to Use Parallel:**

1. **Independent operations:**

```javascript
async function loadPage() {
  const [user, settings, theme] = await Promise.all([
    fetchUser(),
    fetchSettings(),
    fetchTheme()
  ]);
  // None depend on each other - run simultaneously!
}
```

2. **Fetching multiple resources:**

```javascript
async function getProductDetails(productId) {
  const [product, reviews, relatedProducts, inventory] = await Promise.all([
    fetchProduct(productId),
    fetchReviews(productId),
    fetchRelatedProducts(productId),
    checkInventory(productId)
  ]);
  return { product, reviews, relatedProducts, inventory };
}
```

**Hybrid Approach (Best of Both Worlds):**

```javascript
async function smartCheckout(userId, cartId) {
  // Step 1: Get user and cart in parallel
  const [user, cart] = await Promise.all([
    fetchUser(userId),
    fetchCart(cartId)
  ]);
  
  // Step 2: Process payment (needs user and cart)
  const payment = await processPayment(user, cart);
  
  // Step 3: Generate invoice and send email in parallel
  const [invoice, emailResult] = await Promise.all([
    generateInvoice(payment),
    sendConfirmationEmail(user.email, payment)
  ]);
  
  return { payment, invoice, emailResult };
}
```

**Performance Comparison:**

```javascript
// Let's measure the difference
async function measurePerformance() {
  // Sequential
  console.time('Sequential');
  await delay(100);
  await delay(100);
  await delay(100);
  console.timeEnd('Sequential'); // ~300ms
  
  // Parallel
  console.time('Parallel');
  await Promise.all([
    delay(100),
    delay(100),
    delay(100)
  ]);
  console.timeEnd('Parallel'); // ~100ms
}
```

**Pro Tip**: If you have mixed scenarios, start dependent operations together:

```javascript
async function optimized(userId) {
  // Start user fetch
  const userPromise = fetchUser(userId);
  
  // Start other independent operations
  const themePromise = fetchTheme();
  const settingsPromise = fetchSettings();
  
  // Wait for user
  const user = await userPromise;
  
  // Now fetch user-specific data in parallel with the rest
  const [posts, theme, settings] = await Promise.all([
    fetchPosts(user.id), // Depends on user
    themePromise,        // Already started
    settingsPromise      // Already started
  ]);
  
  return { user, posts, theme, settings };
}
```

The key is: **Use parallel whenever possible, sequential when necessary!**

---

### Q10: How would you implement retry logic with async/await?

**Answer:**

Excellent question! In real-world applications, network requests fail all the time - timeout, server errors, etc. Implementing retry logic shows production-ready thinking.

**Basic Retry Implementation:**

```javascript
async function fetchWithRetry(url, maxRetries = 3) {
  for (let i = 0; i < maxRetries; i++) {
    try {
      console.log(`Attempt ${i + 1} of ${maxRetries}...`);
      const response = await fetch(url);
      
      if (!response.ok) {
        throw new Error(`HTTP ${response.status}`);
      }
      
      return await response.json(); // Success!
    } catch (error) {
      console.log(`Attempt ${i + 1} failed:`, error.message);
      
      // If this was the last attempt, throw the error
      if (i === maxRetries - 1) {
        throw new Error(`Failed after ${maxRetries} attempts: ${error.message}`);
      }
      
      // Wait before retrying (simple delay)
      await new Promise(resolve => setTimeout(resolve, 1000));
    }
  }
}

// Usage
try {
  const data = await fetchWithRetry('https://api.example.com/data');
  console.log('Success:', data);
} catch (error) {
  console.error('All retries failed:', error);
}
```

**Advanced: Exponential Backoff (Industry Standard)**

```javascript
async function fetchWithExponentialBackoff(url, maxRetries = 5) {
  let delay = 1000; // Start with 1 second
  
  for (let i = 0; i < maxRetries; i++) {
    try {
      console.log(`Attempt ${i + 1}/${maxRetries}...`);
      const response = await fetch(url);
      
      if (response.ok) {
        return await response.json();
      }
      
      // Don't retry client errors (4xx)
      if (response.status >= 400 && response.status < 500) {
        throw new Error(`Client error: ${response.status}`);
      }
      
      throw new Error(`Server error: ${response.status}`);
    } catch (error) {
      console.log(`Attempt ${i + 1} failed:`, error.message);
      
      if (i === maxRetries - 1) {
        throw new Error(`Failed after ${maxRetries} attempts`);
      }
      
      // Exponential backoff: wait longer each time
      console.log(`Waiting ${delay}ms before retry...`);
      await new Promise(resolve => setTimeout(resolve, delay));
      delay *= 2; // 1s, 2s, 4s, 8s, 16s...
      
      // Optional: add jitter to prevent thundering herd
      delay += Math.random() * 1000;
    }
  }
}
```

**Production-Ready: Configurable Retry Function**

```javascript
async function retryAsync(
  fn,
  options = {
    maxRetries: 3,
    initialDelay: 1000,
    maxDelay: 30000,
    shouldRetry: (error) => true,
    onRetry: (error, attempt) => {}
  }
) {
  const {
    maxRetries,
    initialDelay,
    maxDelay,
    shouldRetry,
    onRetry
  } = options;
  
  let lastError;
  let delay = initialDelay;
  
  for (let attempt = 1; attempt <= maxRetries; attempt++) {
    try {
      return await fn(); // Execute the function
    } catch (error) {
      lastError = error;
      
      // Check if we should retry this error
      if (!shouldRetry(error)) {
        throw error;
      }
      
      if (attempt === maxRetries) {
        throw new Error(
          `Failed after ${maxRetries} attempts: ${error.message}`
        );
      }
      
      // Notify about retry
      onRetry(error, attempt);
      
      // Wait before next attempt
      await new Promise(resolve => setTimeout(resolve, delay));
      
      // Increase delay for next time (exponential backoff)
      delay = Math.min(delay * 2, maxDelay);
    }
  }
  
  throw lastError;
}

// Usage examples:
// Example 1: Simple retry
const data = await retryAsync(
  () => fetch('https://api.example.com/data').then(r => r.json())
);

// Example 2: With custom options
const userData = await retryAsync(
  async () => {
    const response = await fetch('/api/user/123');
    if (!response.ok) throw new Error(`HTTP ${response.status}`);
    return response.json();
  },
  {
    maxRetries: 5,
    initialDelay: 500,
    shouldRetry: (error) => {
      // Only retry server errors (5xx) and network errors
      return error.message.includes('5') || error.message.includes('network');
    },
    onRetry: (error, attempt) => {
      console.log(`Retry attempt ${attempt}: ${error.message}`);
      // Could log to monitoring service here
    }
  }
);
```

**Real-World Example: API Client with Retry**

```javascript
class APIClient {
  constructor(baseURL) {
    this.baseURL = baseURL;
  }
  
  async request(endpoint, options = {}) {
    const maxRetries = options.maxRetries || 3;
    let lastError;
    
    for (let attempt = 1; attempt <= maxRetries; attempt++) {
      try {
        const response = await fetch(`${this.baseURL}${endpoint}`, {
          ...options,
          headers: {
            'Content-Type': 'application/json',
            ...options.headers
          }
        });
        
        // Success!
        if (response.ok) {
          return await response.json();
        }
        
        // Don't retry client errors
        if (response.status >= 400 && response.status < 500) {
          throw new Error(`Client error: ${response.status}`);
        }
        
        // Retry server errors
        throw new Error(`Server error: ${response.status}`);
      } catch (error) {
        lastError = error;
        console.error(`Attempt ${attempt} failed:`, error.message);
        
        // Last attempt
        if (attempt === maxRetries) {
          throw new Error(`Request failed after ${maxRetries} attempts: ${error.message}`);
        }
        
        // Wait with exponential backoff
        const delay = Math.min(1000 * Math.pow(2, attempt - 1), 10000);
        await new Promise(resolve => setTimeout(resolve, delay));
      }
    }
    
    throw lastError;
  }
  
  async get(endpoint, options) {
    return this.request(endpoint, { ...options, method: 'GET' });
  }
  
  async post(endpoint, data, options) {
    return this.request(endpoint, {
      ...options,
      method: 'POST',
      body: JSON.stringify(data)
    });
  }
}

// Usage
const api = new APIClient('https://api.example.com');

try {
  const user = await api.get('/users/123', { maxRetries: 5 });
  console.log('User loaded:', user);
} catch (error) {
  console.error('Failed to load user:', error);
}
```

**When to Use Retry Logic:**

✅ **DO retry:**

- Network timeouts
- Server errors (5xx)
- Rate limiting (429)
- Temporary service unavailability

❌ **DON'T retry:**

- Client errors (4xx) - your request is wrong
- Authentication failures (401, 403)
- Not found errors (404)
- Validation errors (400)

**Pro Tips:**

1. **Exponential backoff prevents overwhelming the server**
2. **Add jitter (random delay) to prevent thundering herd problem**
3. **Set a maximum delay to avoid waiting too long**
4. **Log retry attempts for debugging**
5. **Consider circuit breaker pattern for repeated failures**

This shows you understand production systems and how to build resilient applications!

---

### Q11: Can you explain the event loop and how it relates to async JavaScript?

**Answer:**

Ah, the event loop! This is the heart of JavaScript's asynchronous behavior. Understanding this really shows you know how JavaScript works under the hood.

**The Simple Explanation:**

JavaScript is like a single worker in a restaurant kitchen who can only do one task at a time. But this worker is smart - instead of standing idle while water boils or the oven bakes, they set timers and move on to other tasks. When a timer goes off, they come back to finish that task.

**How It Actually Works:**

```javascript
console.log('1. Start');

setTimeout(() => {
  console.log('3. Timeout callback');
}, 0); // Even with 0 delay!

Promise.resolve().then(() => {
  console.log('2. Promise callback');
});

console.log('4. End');

// Output:
// 1. Start
// 4. End
// 2. Promise callback
// 3. Timeout callback
```

**Why this order?** Let me explain the event loop:

**The Event Loop Components:**

1. **Call Stack**: Where code executes (synchronous operations)
2. **Web APIs**: Browser features (setTimeout, fetch, DOM events)
3. **Microtask Queue**: Promise callbacks, queueMicrotask
4. **Macrotask Queue**: setTimeout, setInterval, I/O operations
5. **Event Loop**: Coordinator that moves tasks between queues

**Visual Walkthrough:**

```javascript
function walkthrough() {
  console.log('A'); // Goes to call stack, executes immediately
  
  setTimeout(() => {
    console.log('B'); // Goes to Web APIs, then Macrotask Queue
  }, 0);
  
  Promise.resolve().then(() => {
    console.log('C'); // Goes to Microtask Queue
  });
  
  console.log('D'); // Goes to call stack, executes immediately
}

walkthrough();

// Output: A, D, C, B

/* 
Execution flow:
1. Call Stack: console.log('A') → prints "A"
2. Web APIs: setTimeout registered
3. Microtask Queue: Promise.then registered
4. Call Stack: console.log('D') → prints "D"
5. Call Stack is empty → Event loop checks Microtask Queue
6. Microtask Queue: console.log('C') → prints "C"
7. Microtask Queue empty → Event loop checks Macrotask Queue
8. Macrotask Queue: console.log('B') → prints "B"
*/
```

**Priority Order (Important!):**

```
1. Synchronous code (Call Stack)
2. Microtasks (Promises, queueMicrotask)
3. Macrotasks (setTimeout, setInterval)
```

**Complex Example:**

```javascript
console.log('1. Script start');

setTimeout(() => {
  console.log('2. setTimeout 1');
  Promise.resolve().then(() => {
    console.log('3. Promise inside setTimeout');
  });
}, 0);

Promise.resolve()
  .then(() => {
    console.log('4. Promise 1');
    setTimeout(() => {
      console.log('5. setTimeout inside Promise');
    }, 0);
  })
  .then(() => {
    console.log('6. Promise 2');
  });

setTimeout(() => {
  console.log('7. setTimeout 2');
}, 0);

console.log('8. Script end');

// Output:
// 1. Script start
// 8. Script end
// 4. Promise 1
// 6. Promise 2
// 2. setTimeout 1
// 3. Promise inside setTimeout
// 7. setTimeout 2
// 5. setTimeout inside Promise
```

**Real-World Implication:**

```javascript
// This can cause performance issues!
async function processItems(items) {
  for (const item of items) {
    await processItem(item); // Blocks on each item
  }
}

// Better: Process in batches
async function processItemsBetter(items, batchSize = 10) {
  for (let i = 0; i < items.length; i += batchSize) {
    const batch = items.slice(i, i + batchSize);
    await Promise.all(batch.map(item => processItem(item)));
    
    // Yield to event loop between batches
    await new Promise(resolve => setTimeout(resolve, 0));
  }
}
```

**Why This Matters:**

Understanding the event loop helps you:

- Debug async code effectively
- Avoid blocking the UI
- Write performant async code
- Understand why certain things happen in a specific order

**Interview Power Move:** "The event loop is why JavaScript can be single-threaded yet still handle asynchronous operations efficiently. It's the reason we can make network requests without freezing the browser!"

---

### Q12: What are common mistakes when working with async/await and how do you avoid them?

**Answer:**

Great question! Even experienced developers make these mistakes. Let me show you the most common ones and how to avoid them:

**Mistake 1: Forgetting to use `await`**

```javascript
// ❌ WRONG - Returns a Promise, not the data!
async function getUser() {
  const user = fetchUser(); // Missing await!
  console.log(user); // Promise { <pending> }
  return user;
}

// ✅ CORRECT
async function getUser() {
  const user = await fetchUser();
  console.log(user); // { id: 1, name: 'John' }
  return user;
}
```

**Mistake 2: Using `await` in loops unnecessarily (Sequential instead of Parallel)**

```javascript
// ❌ WRONG - Takes 5 seconds if each request takes 1 second
async function getUsers(userIds) {
  const users = [];
  for (const id of userIds) {
    const user = await fetchUser(id); // Waits for each!
    users.push(user);
  }
  return users;
}

// ✅ CORRECT - Takes 1 second (all run in parallel)
async function getUsers(userIds) {
  const promises = userIds.map(id => fetchUser(id));
  const users = await Promise.all(promises);
  return users;
}

// Or even cleaner:
async function getUsers(userIds) {
  return Promise.all(userIds.map(fetchUser));
}
```

**Mistake 3: Not handling errors**

```javascript
// ❌ WRONG - Unhandled errors crash the app
async function loadData() {
  const data = await fetchData(); // What if this fails?
  return data;
}

// ✅ CORRECT - Always handle errors
async function loadData() {
  try {
    const data = await fetchData();
    return data;
  } catch (error) {
    console.error('Failed to load data:', error);
    return null; // Or throw, depending on your needs
  }
}

// Alternative: Handle at call site
loadData()
  .then(data => console.log(data))
  .catch(error => console.error(error));
```

**Mistake 4: Forgetting that async functions return Promises**

```javascript
// ❌ WRONG - Trying to use the result synchronously
async function getTotal() {
  return 42;
}

const total = getTotal();
console.log(total + 10); // NaN! (Promise + 10)

// ✅ CORRECT
async function getTotal() {
  return 42;
}

getTotal().then(total => {
  console.log(total + 10); // 52
});

// Or within another async function:
async function calculate() {
  const total = await getTotal();
  console.log(total + 10); // 52
}
```

**Mistake 5: Creating unnecessary async functions**

```javascript
// ❌ WRONG - Unnecessary async wrapper
async function getUser(id) {
  return fetchUser(id); // Just returning a promise
}

// ✅ CORRECT - No need for async if just returning
function getUser(id) {
  return fetchUser(id);
}

// Async is needed when using await:
async function getUser(id) {
  const user = await fetchUser(id);
  return { ...user, timestamp: Date.now() }; // Processing result
}
```

**Mistake 6: Not waiting for async operations in constructors**

```javascript
// ❌ WRONG - Constructors can't be async!
class User {
  constructor(id) {
    this.data = await fetchUser(id); // Syntax error!
  }
}

// ✅ CORRECT - Use a static factory method
class User {
  constructor(data) {
    this.data = data;
  }
  
  static async create(id) {
    const data = await fetchUser(id);
    return new User(data);
  }
}

// Usage:
const user = await User.create(123);
```

**Mistake 7: Mixing callbacks and async/await**

```javascript
// ❌ WRONG - Confusing mix of patterns
async function processData() {
  getData((data) => {
    // Callback doesn't work with await/async flow
    await processItem(data); // This await doesn't work here!
  });
}

// ✅ CORRECT - Convert callback to promise
function getDataPromise() {
  return new Promise((resolve, reject) => {
    getData((error, data) => {
      if (error) reject(error);
      else resolve(data);
    });
  });
}

async function processData() {
  const data = await getDataPromise();
  await processItem(data);
}
```

**Mistake 8: Not handling Promise rejection in forEach**

```javascript
// ❌ WRONG - forEach doesn't work with async/await!
async function processUsers(users) {
  users.forEach(async (user) => {
    await updateUser(user); // Won't wait!
  });
  console.log('Done!'); // Prints immediately!
}

// ✅ CORRECT - Use for...of or Promise.all
async function processUsers(users) {
  for (const user of users) {
    await updateUser(user); // Sequential
  }
  console.log('Done!'); // Prints after all complete
}

// Or parallel:
async function processUsers(users) {
  await Promise.all(users.map(user => updateUser(user)));
  console.log('Done!');
}
```

**Mistake 9: Floating promises (not awaiting or handling)**

```javascript
// ❌ WRONG - Fire and forget (errors go unhandled)
async function saveData(data) {
  updateCache(data); // Not awaited or handled!
  return saveToDatabase(data);
}

// ✅ CORRECT - Either await or handle
async function saveData(data) {
  await updateCache(data);
  return saveToDatabase(data);
}

// Or if you truly want fire-and-forget:
async function saveData(data) {
  updateCache(data).catch(err => console.error('Cache update failed:', err));
  return saveToDatabase(data);
}
```

**Mistake 10: Async/await with array methods that expect sync callbacks**

```javascript
// ❌ WRONG - map returns promises, not values
async function getUserNames(userIds) {
  return userIds.map(async (id) => {
    const user = await fetchUser(id);
    return user.name;
  });
  // Returns: [Promise, Promise, Promise]
}

// ✅ CORRECT - Use Promise.all
async function getUserNames(userIds) {
  const promises = userIds.map(async (id) => {
    const user = await fetchUser(id);
    return user.name;
  });
  return Promise.all(promises);
}
```

**Bonus: Best Practices Checklist**

```javascript
// ✅ Complete example with all best practices
class DataService {
  async fetchUserData(userId) {
    // 1. Always use try/catch
    try {
      // 2. Run independent operations in parallel
      const [user, posts, friends] = await Promise.all([
        this.fetchUser(userId),
        this.fetchPosts(userId),
        this.fetchFriends(userId)
      ]);
      
      // 3. Process sequentially only when needed
      const enrichedPosts = [];
      for (const post of posts) {
        const comments = await this.fetchComments(post.id);
        enrichedPosts.push({ ...post, comments });
      }
      
      // 4. Return structured data
      return {
        user,
        posts: enrichedPosts,
        friends
      };
    } catch (error) {
      // 5. Handle errors appropriately
      console.error('Failed to fetch user data:', error);
      
      // 6. Re-throw or return default based on context
      throw new Error(`User data fetch failed: ${error.message}`);
    }
  }
  
  async fetchUser(id) {
    // 7. Add timeout to prevent hanging
    const timeout = new Promise((_, reject) =>
      setTimeout(() => reject(new Error('Timeout')), 5000)
    );
    
    return Promise.race([
      fetch(`/api/users/${id}`).then(r => r.json()),
      timeout
    ]);
  }
  
  // 8. Document async functions
  /**
   * Fetches user posts with pagination
   * @param {number} userId - The user ID
   * @param {number} page - Page number (default: 1)
   * @returns {Promise<Post[]>} Array of posts
   */
  async fetchPosts(userId, page = 1) {
    // Implementation
  }
}
```

Understanding these mistakes and how to avoid them shows you're a thoughtful developer who writes maintainable code!

---

### Q13: How would you implement a timeout for async operations?

**Answer:**

Timeouts are crucial for production apps - you can't wait forever for a response! Here are multiple approaches:

**Approach 1: Simple Timeout Wrapper**

```javascript
function timeout(promise, ms) {
  return new Promise((resolve, reject) => {
    const timer = setTimeout(() => {
      reject(new Error(`Operation timed out after ${ms}ms`));
    }, ms);
    
    promise
      .then(value => {
        clearTimeout(timer);
        resolve(value);
      })
      .catch(error => {
        clearTimeout(timer);
        reject(error);
      });
  });
}

// Usage:
async function fetchWithTimeout() {
  try {
    const data = await timeout(
      fetch('https://api.example.com/slow-endpoint'),
      5000 // 5 second timeout
    );
    return data.json();
  } catch (error) {
    console.error('Request failed or timed out:', error);
  }
}
```

**Approach 2: Using Promise.race (Cleaner)**

```javascript
function fetchWithTimeout(url, ms) {
  const timeoutPromise = new Promise((_, reject) =>
    setTimeout(() => reject(new Error('Request timeout')), ms)
  );
  
  const fetchPromise = fetch(url).then(response => response.json());
  
  return Promise.race([fetchPromise, timeoutPromise]);
}

// Usage:
try {
  const data = await fetchWithTimeout('https://api.example.com/data', 3000);
  console.log('Data received:', data);
} catch (error) {
  console.error('Failed:', error.message);
}
```

**Approach 3: Reusable Generic Timeout Function**

```javascript
async function withTimeout(asyncFn, timeoutMs, timeoutMessage) {
  return Promise.race([
    asyncFn(),
    new Promise((_, reject) =>
      setTimeout(
        () => reject(new Error(timeoutMessage || `Timeout after ${timeoutMs}ms`)),
        timeoutMs
      )
    )
  ]);
}

// Usage with any async function:
const user = await withTimeout(
  () => fetchUser(123),
  5000,
  'User fetch timed out'
);

const posts = await withTimeout(
  () => fetchPosts(user.id),
  3000,
  'Posts fetch timed out'
);
```

**Approach 4: Timeout with Abort Controller (Modern & Best Practice)**

```javascript
async function fetchWithAbort(url, timeoutMs = 5000) {
  const controller = new AbortController();
  const timeout = setTimeout(() => controller.abort(), timeoutMs);
  
  try {
    const response = await fetch(url, {
      signal: controller.signal
    });
    clearTimeout(timeout);
    return await response.json();
  } catch (error) {
    clearTimeout(timeout);
    if (error.name === 'AbortError') {
      throw new Error(`Request timed out after ${timeoutMs}ms`);
    }
    throw error;
  }
}

// Usage:
try {
  const data = await fetchWithAbort('https://api.example.com/data', 3000);
  console.log(data);
} catch (error) {
  console.error('Request failed:', error.message);
}
```

**Approach 5: Production-Ready API Client with Timeout**

```javascript
class APIClient {
  constructor(baseURL, defaultTimeout = 10000) {
    this.baseURL = baseURL;
    this.defaultTimeout = defaultTimeout;
  }
  
  async request(endpoint, options = {}) {
    const {
      timeout = this.defaultTimeout,
      retries = 3,
      ...fetchOptions
    } = options;
    
    const controller = new AbortController();
    const timeoutId = setTimeout(() => controller.abort(), timeout);
    
    let lastError;
    
    for (let attempt = 1; attempt <= retries; attempt++) {
      try {
        const response = await fetch(`${this.baseURL}${endpoint}`, {
          ...fetchOptions,
          signal: controller.signal
        });
        
        clearTimeout(timeoutId);
        
        if (!response.ok) {
          throw new Error(`HTTP ${response.status}: ${response.statusText}`);
        }
        
        return await response.json();
      } catch (error) {
        lastError = error;
        
        if (error.name === 'AbortError') {
          throw new Error(`Request timeout after ${timeout}ms`);
        }
        
        if (attempt < retries) {
          console.log(`Attempt ${attempt} failed, retrying...`);
          await new Promise(resolve => setTimeout(resolve, 1000 * attempt));
        }
      }
    }
    
    throw lastError;
  }
  
  async get(endpoint, options) {
    return this.request(endpoint, { ...options, method: 'GET' });
  }
  
  async post(endpoint, data, options) {
    return this.request(endpoint, {
      ...options,
      method: 'POST',
      headers: { 'Content-Type': 'application/json' },
      body: JSON.stringify(data)
    });
  }
}

// Usage:
const api = new APIClient('https://api.example.com', 5000);

try {
  // This request has 5 second timeout (default)
  const user = await api.get('/users/123');
  
  // This request has custom 10 second timeout
  const data = await api.get('/large-dataset', { timeout: 10000 });
  
  // Post with timeout and retries
  const result = await api.post('/orders', orderData, {
    timeout: 8000,
    retries: 5
  });
} catch (error) {
  console.error('API request failed:', error.message);
}
```

**Approach 6: Multiple Operations with Individual Timeouts**

```javascript
async function loadDashboard(userId) {
  // Each operation has its own timeout
  const results = await Promise.allSettled([
    withTimeout(() => fetchUser(userId), 3000, 'User timeout'),
    withTimeout(() => fetchPosts(userId), 5000, 'Posts timeout'),
    withTimeout(() => fetchNotifications(userId), 2000, 'Notifications timeout'),
    withTimeout(() => fetchSettings(userId), 4000, 'Settings timeout')
  ]);
  
  return {
    user: results[0].status === 'fulfilled' ? results[0].value : null,
    posts: results[1].status === 'fulfilled' ? results[1].value : [],
    notifications: results[2].status === 'fulfilled' ? results[2].value : [],
    settings: results[3].status === 'fulfilled' ? results[3].value : {}
  };
}
```

**Key Takeaways:**

1. **Always have timeouts** for network requests in production
2. **AbortController is preferred** for fetch API (can actually cancel)
3. **Promise.race is simple** but doesn't cancel the underlying operation
4. **Different operations need different timeouts** (small queries: 2-3s, large data: 10-30s)
5. **Combine with retries** for resilience

This shows you understand production concerns like user experience and resource management!

---

## Bonus Questions

### Q14: What's the difference between `Promise.resolve()` and `new Promise()`?

**Answer:**

**Quick Answer:** `Promise.resolve()` immediately creates a fulfilled promise, while `new Promise()` lets you control when it resolves.

```javascript
// Promise.resolve() - Instant resolution
const promise1 = Promise.resolve('Immediate value');
promise1.then(value => console.log(value)); // 'Immediate value'

// new Promise() - You control when it resolves
const promise2 = new Promise((resolve) => {
  setTimeout(() => {
    resolve('Delayed value');
  }, 2000);
});
promise2.then(value => console.log(value)); // After 2 seconds

// Use cases:

// 1. Converting values to promises
async function getConfig() {
  const cached = localStorage.getItem('config');
  if (cached) {
    return Promise.resolve(JSON.parse(cached)); // Immediate
  }
  return fetch('/api/config').then(r => r.json()); // Async
}

// 2. Promise.resolve with
// 2. Promise.resolve with another promise const existingPromise = fetch('/api/data'); const wrapped = Promise.resolve(existingPromise); // Both are the same promise!

// 3. Flattening thenable objects const thenable = { then(resolve) { resolve(42); } }; Promise.resolve(thenable).then(value => console.log(value)); // 42

````

---

### Q15: Can you call an async function without await? What happens?

**Answer:**

Yes! And this is actually useful in certain scenarios:

```javascript
// Calling without await returns a Promise immediately
async function fetchData() {
  await delay(1000);
  return 'Data loaded';
}

// Fire and forget (not recommended usually)
fetchData(); // Returns Promise, but we ignore it

// Better: Handle the promise
fetchData().then(data => console.log(data));

// Or catch errors
fetchData()
  .then(data => console.log(data))
  .catch(err => console.error(err));

// Real use case: Parallel operations
async function loadApp() {
  // Start all at once (don't await yet)
  const userPromise = fetchUser();
  const configPromise = fetchConfig();
  const themePromise = fetchTheme();
  
  // Do some other work...
  initializeApp();
  setupEventListeners();
  
  // Now wait for all
  const [user, config, theme] = await Promise.all([
    userPromise,
    configPromise,
    themePromise
  ]);
  
  return { user, config, theme };
}
````

---

### Q16: How do you cancel an ongoing async operation?

**Answer:**

Great question! JavaScript doesn't have built-in cancellation, but we can use **AbortController**:

```javascript
// Example: Cancel a fetch request
const controller = new AbortController();

async function fetchWithCancel() {
  try {
    const response = await fetch('https://api.example.com/data', {
      signal: controller.signal
    });
    return await response.json();
  } catch (error) {
    if (error.name === 'AbortError') {
      console.log('Request was cancelled');
    } else {
      console.error('Request failed:', error);
    }
  }
}

// Start the request
const dataPromise = fetchWithCancel();

// Cancel it after 2 seconds
setTimeout(() => {
  controller.abort();
  console.log('Cancelled!');
}, 2000);

// Real-world: Search with cancellation
class SearchComponent {
  constructor() {
    this.currentRequest = null;
  }
  
  async search(query) {
    // Cancel previous request if exists
    if (this.currentRequest) {
      this.currentRequest.abort();
    }
    
    // Create new abort controller
    this.currentRequest = new AbortController();
    
    try {
      const response = await fetch(`/api/search?q=${query}`, {
        signal: this.currentRequest.signal
      });
      const results = await response.json();
      this.displayResults(results);
    } catch (error) {
      if (error.name !== 'AbortError') {
        console.error('Search failed:', error);
      }
    }
  }
  
  displayResults(results) {
    console.log('Search results:', results);
  }
}

// Usage: User types fast, only last search executes
const search = new SearchComponent();
search.search('java'); // Gets cancelled
search.search('javascript'); // Gets cancelled  
search.search('javascript async'); // This one completes!
```

---

### Q17: What happens if you don't handle a rejected promise?

**Answer:**

**Unhandled Promise Rejection** - this causes warnings in console and can crash Node.js apps!

```javascript
// ❌ BAD - Unhandled rejection
async function riskyOperation() {
  throw new Error('Something went wrong!');
}

riskyOperation(); // Unhandled promise rejection warning!

// ✅ GOOD - Always handle
riskyOperation()
  .catch(error => console.error('Handled:', error));

// Or with try/catch
async function safeOperation() {
  try {
    await riskyOperation();
  } catch (error) {
    console.error('Handled:', error);
  }
}

// Global handlers (last resort)
// In browser:
window.addEventListener('unhandledrejection', (event) => {
  console.error('Unhandled rejection:', event.reason);
  event.preventDefault();
});

// In Node.js:
process.on('unhandledRejection', (reason, promise) => {
  console.error('Unhandled rejection at:', promise, 'reason:', reason);
});
```

**Why it matters:** In production, unhandled rejections can crash your Node.js server or leave your app in a broken state!

---

### Q18: How would you convert a callback-based function to use Promises?

**Answer:**

This is a common task when working with older libraries. Here's the pattern:

```javascript
// Old callback-based function
function readFileCallback(filename, callback) {
  setTimeout(() => {
    if (!filename) {
      callback(new Error('Filename required'), null);
    } else {
      callback(null, `Contents of ${filename}`);
    }
  }, 1000);
}

// Convert to Promise
function readFilePromise(filename) {
  return new Promise((resolve, reject) => {
    readFileCallback(filename, (error, data) => {
      if (error) {
        reject(error);
      } else {
        resolve(data);
      }
    });
  });
}

// Now use with async/await!
async function example() {
  try {
    const contents = await readFilePromise('data.txt');
    console.log(contents);
  } catch (error) {
    console.error('Error:', error);
  }
}

// Reusable utility (Node.js has util.promisify for this!)
function promisify(fn) {
  return function(...args) {
    return new Promise((resolve, reject) => {
      fn(...args, (error, result) => {
        if (error) reject(error);
        else resolve(result);
      });
    });
  };
}

// Usage:
const readFileAsync = promisify(readFileCallback);
const data = await readFileAsync('file.txt');
```

---

### Q19: Explain microtasks vs macrotasks with an example

**Answer:**

This shows deep understanding of JavaScript's event loop!

**Microtasks** (Higher Priority):

- Promise callbacks (.then, .catch, .finally)
- queueMicrotask()
- MutationObserver

**Macrotasks** (Lower Priority):

- setTimeout / setInterval
- I/O operations
- UI rendering

```javascript
console.log('1. Sync start');

setTimeout(() => {
  console.log('2. Macrotask (setTimeout)');
}, 0);

Promise.resolve()
  .then(() => {
    console.log('3. Microtask (Promise)');
  })
  .then(() => {
    console.log('4. Another microtask');
  });

queueMicrotask(() => {
  console.log('5. Microtask (queueMicrotask)');
});

setTimeout(() => {
  console.log('6. Another macrotask');
}, 0);

console.log('7. Sync end');

/* Output order:
1. Sync start
2. Sync end
3. Microtask (Promise)
4. Another microtask
5. Microtask (queueMicrotask)
6. Macrotask (setTimeout)
7. Another macrotask
*/

// Rule: All microtasks run BEFORE the next macrotask
```

**Complex Example:**

```javascript
console.log('Start');

setTimeout(() => {
  console.log('Timeout 1');
  Promise.resolve().then(() => console.log('Promise in timeout'));
}, 0);

Promise.resolve()
  .then(() => {
    console.log('Promise 1');
    setTimeout(() => console.log('Timeout in promise'), 0);
  })
  .then(() => console.log('Promise 2'));

console.log('End');

/* Output:
Start
End
Promise 1
Promise 2
Timeout 1
Promise in timeout
Timeout in promise
*/

// Explanation:
// 1. Sync code runs: "Start", "End"
// 2. Microtask queue processes: "Promise 1", "Promise 2"
// 3. First macrotask: "Timeout 1"
// 4. Microtask created in macrotask: "Promise in timeout"
// 5. Next macrotask: "Timeout in promise"
```

**Why This Matters:** Understanding this helps you predict execution order and avoid bugs where timing matters!

---

### Q20: How do you handle multiple async operations where some are optional?

**Answer:**

Great real-world scenario! Sometimes you want to load data but some parts are optional and shouldn't block the main flow:

```javascript
async function loadUserDashboard(userId) {
  try {
    // Critical data - must succeed
    const user = await fetchUser(userId);
    
    // Optional data - don't let failures block the dashboard
    const [posts, notifications, stats] = await Promise.allSettled([
      fetchPosts(userId).catch(() => null),
      fetchNotifications(userId).catch(() => []),
      fetchStats(userId).catch(() => ({}))
    ]);
    
    return {
      user, // Critical
      posts: posts.status === 'fulfilled' ? posts.value : null,
      notifications: notifications.status === 'fulfilled' ? notifications.value : [],
      stats: stats.status === 'fulfilled' ? stats.value : {},
      hasPartialData: [posts, notifications, stats].some(r => r.status === 'rejected')
    };
  } catch (error) {
    console.error('Critical data failed:', error);
    throw error;
  }
}

// Usage:
const dashboard = await loadUserDashboard(123);
if (dashboard.hasPartialData) {
  console.warn('Some data failed to load');
}
```

**Approach 2: With Fallbacks**

```javascript
async function loadPageData(pageId) {
  // Helper function for optional data with fallback
  const fetchOptional = async (fetchFn, fallback) => {
    try {
      return await fetchFn();
    } catch (error) {
      console.warn('Optional data fetch failed:', error);
      return fallback;
    }
  };
  
  // Critical data
  const page = await fetchPage(pageId);
  
  // Optional data with fallbacks in parallel
  const [comments, related, author] = await Promise.all([
    fetchOptional(() => fetchComments(pageId), []),
    fetchOptional(() => fetchRelatedPages(pageId), []),
    fetchOptional(() => fetchAuthor(page.authorId), null)
  ]);
  
  return { page, comments, related, author };
}
```

**Approach 3: Priority-Based Loading**

```javascript
async function smartLoad(userId) {
  // Phase 1: Critical data first (fast response)
  const user = await fetchUser(userId);
  
  // Show something to user immediately
  renderUserProfile(user);
  
  // Phase 2: Important but not critical (load in background)
  Promise.all([
    fetchPosts(userId).then(posts => updatePosts(posts)),
    fetchFriends(userId).then(friends => updateFriends(friends))
  ]).catch(error => console.error('Secondary data failed:', error));
  
  // Phase 3: Nice to have (lowest priority)
  setTimeout(() => {
    fetchRecommendations(userId)
      .then(recs => updateRecommendations(recs))
      .catch(() => {}); // Fail silently
  }, 2000);
  
  return user;
}
```

---

## BONUS: Tricky Interview Questions

### Q21: What will this code output and why?

```javascript
async function question() {
  console.log('1');
  
  const promise = new Promise((resolve) => {
    console.log('2');
    resolve('3');
  });
  
  console.log('4');
  
  promise.then(console.log);
  
  console.log('5');
}

question();
console.log('6');
```

**Answer:**

Output: `1, 2, 4, 5, 6, 3`

**Explanation:**

1. `console.log('1')` - synchronous, prints "1"
2. Promise executor runs synchronously, prints "2"
3. Promise resolves with "3", `.then()` callback queued as microtask
4. `console.log('4')` - synchronous, prints "4"
5. `console.log('5')` - synchronous, prints "5"
6. `question()` completes, `console.log('6')` runs, prints "6"
7. Call stack empty, microtask queue processes, prints "3"

**Key insight:** Promise executors run immediately, but `.then()` callbacks are always asynchronous!

---

### Q22: What's wrong with this code?

```javascript
async function processFiles(files) {
  const results = files.map(async (file) => {
    return await processFile(file);
  });
  console.log('Results:', results);
  return results;
}
```

**Answer:**

**Problem:** `results` is an array of Promises, not the actual values!

```javascript
// ❌ WRONG - returns Promise array
async function processFiles(files) {
  const results = files.map(async (file) => {
    return await processFile(file);
  });
  console.log('Results:', results); // [Promise, Promise, Promise]
  return results;
}

// ✅ CORRECT - wait for all promises
async function processFiles(files) {
  const promises = files.map(async (file) => {
    return await processFile(file);
  });
  const results = await Promise.all(promises);
  console.log('Results:', results); // [result1, result2, result3]
  return results;
}

// ✅ EVEN BETTER - more concise
async function processFiles(files) {
  return Promise.all(files.map(file => processFile(file)));
}
```

---

### Q23: Explain this behavior:

```javascript
const promise = new Promise((resolve) => {
  resolve('First');
  resolve('Second'); // What happens here?
});

promise.then(console.log); // What prints?
```

**Answer:**

**Output:** `"First"`

**Explanation:** Once a Promise is resolved or rejected, it's **settled** and cannot change state. Subsequent calls to `resolve()` or `reject()` are ignored.

```javascript
const promise = new Promise((resolve, reject) => {
  resolve('Success');
  reject('Error');     // Ignored!
  resolve('Another');  // Ignored!
});

promise
  .then(value => console.log('Resolved:', value)) // "Resolved: Success"
  .catch(error => console.log('Rejected:', error)); // Never runs

// This is important for error handling:
function fetchData() {
  return new Promise((resolve, reject) => {
    const timeout = setTimeout(() => reject('Timeout'), 5000);
    
    fetch('/api/data')
      .then(response => {
        clearTimeout(timeout);
        resolve(response); // First to execute wins!
      })
      .catch(error => {
        clearTimeout(timeout);
        reject(error);
      });
  });
}
```

---

### Q24: How can you implement a sleep/delay function using Promises?

**Answer:**

```javascript
// Simple sleep function
function sleep(ms) {
  return new Promise(resolve => setTimeout(resolve, ms));
}

// Usage:
async function demo() {
  console.log('Starting...');
  await sleep(2000); // Wait 2 seconds
  console.log('2 seconds later!');
}

// Practical use case: Polling with delay
async function pollUntilComplete(taskId, maxAttempts = 10) {
  for (let i = 0; i < maxAttempts; i++) {
    const status = await checkTaskStatus(taskId);
    
    if (status === 'complete') {
      return 'Task completed!';
    }
    
    console.log(`Attempt ${i + 1}: Still processing...`);
    await sleep(2000); // Wait 2 seconds between checks
  }
  
  throw new Error('Task did not complete in time');
}

// Advanced: Sleep with ability to cancel
function cancellableSleep(ms) {
  let timeoutId;
  let rejectFn;
  
  const promise = new Promise((resolve, reject) => {
    rejectFn = reject;
    timeoutId = setTimeout(resolve, ms);
  });
  
  promise.cancel = () => {
    clearTimeout(timeoutId);
    rejectFn(new Error('Sleep cancelled'));
  };
  
  return promise;
}

// Usage:
const sleepPromise = cancellableSleep(5000);
sleepPromise.then(() => console.log('Slept 5 seconds'));

// Cancel after 2 seconds
setTimeout(() => sleepPromise.cancel(), 2000);
```

---

### Q25: Final Boss Question - Implement Promise.all from scratch

**Answer:**

This shows you truly understand Promises!

```javascript
function myPromiseAll(promises) {
  return new Promise((resolve, reject) => {
    // Handle edge case: empty array
    if (promises.length === 0) {
      resolve([]);
      return;
    }
    
    const results = [];
    let completedCount = 0;
    
    promises.forEach((promise, index) => {
      // Convert non-promise values to promises
      Promise.resolve(promise)
        .then(value => {
          results[index] = value;
          completedCount++;
          
          // All promises completed?
          if (completedCount === promises.length) {
            resolve(results);
          }
        })
        .catch(error => {
          // If any promise fails, reject immediately
          reject(error);
        });
    });
  });
}

// Test it:
const p1 = Promise.resolve(1);
const p2 = Promise.resolve(2);
const p3 = Promise.resolve(3);

myPromiseAll([p1, p2, p3])
  .then(results => console.log('Results:', results)) // [1, 2, 3]
  .catch(error => console.error('Error:', error));

// Test with failure:
const p4 = Promise.reject('Failed!');
myPromiseAll([p1, p2, p4])
  .then(results => console.log('Results:', results))
  .catch(error => console.error('Error:', error)); // "Error: Failed!"
```

**Bonus: Implement Promise.allSettled**

```javascript
function myPromiseAllSettled(promises) {
  return new Promise((resolve) => {
    if (promises.length === 0) {
      resolve([]);
      return;
    }
    
    const results = [];
    let completedCount = 0;
    
    promises.forEach((promise, index) => {
      Promise.resolve(promise)
        .then(value => {
          results[index] = { status: 'fulfilled', value };
        })
        .catch(reason => {
          results[index] = { status: 'rejected', reason };
        })
        .finally(() => {
          completedCount++;
          if (completedCount === promises.length) {
            resolve(results);
          }
        });
    });
  });
}

// Test:
myPromiseAllSettled([
  Promise.resolve('Success'),
  Promise.reject('Error'),
  Promise.resolve('Another success')
]).then(results => console.log(results));
// [
//   { status: 'fulfilled', value: 'Success' },
//   { status: 'rejected', reason: 'Error' },
//   { status: 'fulfilled', value: 'Another success' }
// ]
```

---

## Interview Tips & Key Takeaways

### What Interviewers Look For:

1. **Understanding fundamentals** - Event loop, call stack, callback queue
2. **Practical experience** - Error handling, performance optimization
3. **Production awareness** - Timeouts, retries, error monitoring
4. **Code quality** - Clean, readable async code
5. **Problem-solving** - Choosing the right approach for the situation

### Power Phrases to Use:

- "In production, I always add timeouts to prevent hanging operations"
- "I prefer async/await for readability, but I know it's just syntactic sugar over Promises"
- "I'd use Promise.all here since the operations are independent"
- "This could cause a race condition, so we need to..."
- "Let me add proper error handling with try/catch"
- "The event loop processes microtasks before macrotasks"
- "I'd implement exponential backoff for the retry logic"

### Common Red Flags (What NOT to Do):

❌ Using async/await without understanding Promises ❌ Forgetting error handling ❌ Sequential operations when parallel would work ❌ Not considering edge cases (empty arrays, timeouts, etc.) ❌ Mixing callbacks and Promises unnecessarily ❌ Floating promises (unhandled async operations)

### Quick Reference Cheat Sheet:

```javascript
// Sequential
const a = await fetchA();
const b = await fetchB(); // Waits for A

// Parallel
const [a, b] = await Promise.all([fetchA(), fetchB()]);

// Race (first wins)
const result = await Promise.race([fast(), slow()]);

// AllSettled (all results, even failures)
const results = await Promise.allSettled([a(), b(), c()]);

// Error handling
try {
  await riskyOperation();
} catch (error) {
  handleError(error);
}

// Timeout
await Promise.race([
  operation(),
  timeout(5000)
]);

// Retry
for (let i = 0; i < 3; i++) {
  try {
    return await operation();
  } catch (error) {
    if (i === 2) throw error;
    await sleep(1000 * Math.pow(2, i));
  }
}
```

---

## Final Words

Mastering asynchronous JavaScript is crucial for modern web development. Remember:

1. **Start with callbacks** to understand the foundation
2. **Learn Promises** for cleaner async code
3. **Master async/await** for the best developer experience
4. **Understand the event loop** to debug effectively
5. **Think about edge cases** - timeouts, errors, empty data
6. **Write production-ready code** - always handle errors, add timeouts, implement retries

**Pro tip for interviews:** When given an async problem, think out loud:

- "First, I'll check if these operations can run in parallel..."
- "I'll add error handling with try/catch..."
- "This needs a timeout to prevent hanging..."
- "Let me add retry logic with exponential backoff..."

This demonstrates not just knowledge, but **production-ready thinking** that impresses interviewers!

Good luck with your interviews! 🚀# Asynchronous JavaScript Interview - Questions & Answers

