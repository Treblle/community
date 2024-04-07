---
author: Ajike Emmanuel
website: Your website if available
avatar: https://avatars.githubusercontent.com/u/117521496?s=400&u=4e7cec76593d8c25a7b5fa073f2045d0d2abf85c&v=4
twitter: https://twitter.com/___emee_
linkedin: https://www.linkedin.com/in/emmanuel-ajike-687396257/
---

# Mastering async await in javascript

Async/await is a JavaScript feature that allows you to wait or pause code execution until something has finished, or in this case, resolved, before code execution can carry on.

In other words, it's like saying JavaScript will wait for this to finish before you can continue.

**I will assume you have some experience with JavaScript and asynchronous programming.**

In the real world, the closest thing to an async/await operation is a regular traffic light. It instructs drivers, pedestrians, etc., to hold on while the other lane gets to pass. You wait for your turn.

In the world of programming, you would know that computers are very fast. So, you would probably think a simple API call gives results right away, right? But the conditions are not always perfect. That simple API call can be influenced by factors like bandwidth, data center outages, etc. So, the concept of async/await is here to help us instruct the computer to hold on until we can get the value we were expecting.

### Promises

A promise is an object which represents the possible completion or failure of an asynchronous operation and it's value. It is a placeholder or proxy for a value not necessarily known when the promise is created. It provides asynchronous handlers for the eventual success (resolve) and failure (reject). It has 3 possible states. A promise is said to be *settled* if it is either fulfilled or rejected, but not pending. Read more at [MDN](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Promise)

An async function will always return a promise: pending, resolved, or rejected. Pending is the initial state neither resolved or rejected. Resolved is when you get your value, and rejected is when it throws an error. The `await` keyword is used to wait for the value.

```js
function fetchPokemonData() {
	let request1 = fetch("https://pokeapi.co/api/v2/pokemon/ditto").then(
		(response) => response.json()
	);
	let request2 = fetch("https://pokeapi.co/api/v2/pokemon/pikachu").then(
		(response) => response.json()
	);

	let response1 = request1.name;
	let response2 = request2.name;
	return [response1, response2];
}

// Alternatively

async function fetchPokemonData() {
	// The fetch API returns a promise, so we wait for it to resolve before we continue.
	let request1 = await fetch("https://pokeapi.co/api/v2/pokemon/ditto");
	let request2 = await fetch("https://pokeapi.co/api/v2/pokemon/pikachu");

	let response1 = await request1.json();
	let response2 = await request2.json();

	return [response1.name, response2.name];
}

// Even better

async function fetchPokemonData() {
	let [request1, request2] = await Promise.all([
		fetch("https://pokeapi.co/api/v2/pokemon/ditto"),
		fetch("https://pokeapi.co/api/v2/pokemon/pikachu"),
	]);

	let [response1, response2] = await Promise.all([
		request1.json(),
		request2.json(),
	]);
	return [response1.name, response2.name];
}
```

### Custom Promises

A promise in JavaScript is a function that lets us resolve or reject based on the values we want.

```js
// Syntax
new Promise((resolve, reject) => {
	// Returns the value and breaks out of this code block
	resolve("some value");

	// Throws an error and breaks out of this code block
	reject("some error");
});
```

Sometimes, when working with custom packages from npm, some of them might not implement promises but still use callback statements. Here's how we can create a custom promise-based wait function using `setTimeout`:

```js
const wait = (seconds) => {
	return new Promise((resolve) => {
		return setTimeout(resolve, seconds);
	});
};

async function run() {
	await wait(6000);
	console.log("Run was called after 6 seconds");
}

run();
```

The `setTimeout` function accepts two arguments: a function and the number of seconds to wait before calling the function. The `wait` function returns a promise, allowing us to await it.

### Async Loops

There are many iterative methods or functions in JavaScript that allow you to perform an operation for each item in an array or collection of values. Some of these functions are `forEach`, `map`, `for...in`, etc. However, the problem is that not all of them are asynchronous.

```js
let arr = [1, 2, 3, 4, 5, 6];

// It waits for 3 seconds and continues execution
arr.map(async (val, index) => {
	await wait(3000);
	console.log(val);
});

// But a for loop waits 3 seconds each time
for (let i = 0; i < arr.length; i++) {
	await wait(3000);
	console.log(arr[i]);
}
```

**Note:** This approach might not be ideal for large arrays due to performance reasons. It's generally recommended to use asynchronous alternatives like `async/await` with `Promise.all` for better performance when dealing with multiple asynchronous operations.

### Generator Functions

A generator function in JavaScript is a special type of function that allows you to pause and resume its execution. It's defined using an asterisk () after the function keyword. When you call a generator function, it doesn't run the entire function at once. Instead, it returns a special type of iterator object called a generator object. This next function contains the value (current value), and a `next` function (calling it retrieves the next set of values).

```js
function* iterator(array) {
	let index = 0;
	let arrayLength = array.length;
	while (index < arrayLength) {
		yield array[index++];
	}
}

const arr = [1, 2, 3, 4, 5];
const generator = iterator(arr);

async function processGenerator() {
	let nextIteration = generator.next();

	while (!nextIteration.done) {
		await wait(3000);
		console.log(nextIteration.value);
		nextIteration = generator.next();
	}
}

processGenerator();
```

The above code snippet showcases a custom iterator that provides more flexibility.

### Pitfalls to Avoid

When fetching data, it's important to avoid nesting your fetch calls to prevent a **waterfall request**. If the requests are not dependent on each other, there's no reason to wait for the previous one to resolve before proceeding with the subsequent one.

```js
// Avoid this pitfall request
async function fetchPokemonData() {
	let analytics = await fetch("https://example.com/analytics/1092");
	let userData = await fetch("https://example.com/api/user/123");

	let response1 = await userData.json();
	let response2 = await analytics.json();

	return [response1.name, response2.name];
}

// Much better, causes the requests to be executed in parallel
async function fetchUserData() {
	let [userData, analytics] = await Promise.all([
		fetch("https://example.com/api/user/123"),
		fetch("https://example.com/analytics/1092"),
	]);

	let [response1, response2] = await Promise.all([
		userData.json(),
		analytics.json(),
	]);
	return [response1.name, response2.name];
}
```

### Error Handling

Handling errors for asynchronous code can be a bit tricky, but with the right set of tools and strategies provided by JavaScript, we can handle errors properly.

In JavaScript, we handle runtime errors using `try...catch` blocks. The code between the `try` braces is executed, and errors are caught with the `catch` statement.

```js
try {
	throw new Error("This is a runtime failure");
} catch (error) {
	console.error("Error: ", error.message);
}
```

It's generally recommended to avoid deeply nested `try...catch` statements. Aim to be as DRY (Don't Repeat Yourself) as possible. Nested `try...catch` statements can be hard to follow and read. Here's one way to achieve DRY principles: create a `tryCatch` function that accepts a function, its arguments, and returns either the error or the resolved value. This allows for cleaner error handling.

```js
const tryCatch = async (cb, ...args) => {
	try {
		let data = await cb(...args);
		return [null, data];
	} catch (error) {
		return [error];
	}
};

async function fetchData(id) {
	// Error path
	throw new Error(`Could not find user with id: ${id}`);

	// Success path (commented out for demonstration)
	// return `The blog data was fetched for user by id: ${id}`;
}

async function getUserId() {
	let [error, value] = await tryCatch(fetchData, 123);

	if (error) {
		console.error(error.message);
		return;
	}

	console.log(value);
}

getUserId();
```

### Conclusion

There are many topics related to mastering async/await that are impossible to cover in a single blog post. However, I believe the information above provides a solid overview to enhance your understanding.
