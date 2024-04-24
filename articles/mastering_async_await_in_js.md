---
author: Ajike Emmanuel

website: Your website if available

avatar: https://avatars.githubusercontent.com/u/117521496?s=400&u=4e7cec76593d8c25a7b5fa073f2045d0d2abf85c&v=4

twitter: https://twitter.com/___emee_

linkedin: https://www.linkedin.com/in/emmanuel-ajike-687396257/
---

# Mastering async/await in javascript

Async/await is a JavaScript feature that allows you to wait or pause code execution until something has finished or in this case resolved before code execution can carry on

So in essence, it is like telling JavaScript to wait for this to finish before you can continue.

**I will assume you have some experience of Javascript and asynchronous programming.**

In the real world the closest thing to an Async/await operation is the regular traffic light that instructs drivers, pedestrians etc. to hold on this or the other lane is to pass, wait for your turn.

In the world of programming you would know that computers are so fast so you would probably think a simple api call gives work right away right?, But the conditions are not always perfect and that simple api call can be influenced by factors like bandwidth, data center outages etc. The async/await mechanism serves as a way to instruct the computer to wait for the anticipated result to arrive.

### Promises

An object which represents the favorable value or error of an async operation is a promise. It is a placeholder for a value not necessarily known when creating the promise. It provides asynchronous handlers for the eventual success (resolve) and failure (reject). It has 3 possible states. A promise is said to be _settled_ if it is either fulfilled or rejected, but not pending. Stop by https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Promise

An async function will always return a promise status of pending, resolved, or rejected. Resolved is when you get your value and rejected is when it throws an error. Together with the await keyword we can wait for the value to resolve.

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

A promise in javascript is a function that lets us resolve or reject based on the values we want

```js
// Syntax

new Promise((resolve, reject) => {
	// Returns the value and breaks out of this code block

	resolve("some value"); // Throws an error and breaks out of this code block

	reject("some error");
});
```

Some custom packages from npm most times do not make use of promises but are still using callback functions. Here is how we can create a custom promise based wait function using `setTimeout`

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

The `setTimeout` accepts two arguments: a function and the number of seconds to wait before calling the function. The wait function returns a promise allowing us to await it.

### Async Loops

There are so many repetitious functions in Javascript, which allow us to accomplish an operation for each item in an array of values. Some of these functions are `forEach`, `map`, `for...in`, in etc. But the problem is that not all of them are asynchronous

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

\*\*Note: This is not very important but it is worthy of notice. Just use `async/await` with the `Promise.all` if you are working with a lot of async operations.

### Generator Functions

They are capable of retaining context, kind of like a closure. It is a special type of function that uses the `function*` syntax. It can be used when implementing infinite scroll or handling an infinite stream of data. When used in conjunction with Promises we are able to deal with async logic. Instead, it returns a special type of iterator object called a generator object. This next function contains the value (current value), `next` function (calling it retrieves the next set of values).

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

The above is a custom iterator which gives us the flexibility to do more.

### Pitfalls to avoid

When fetching data it is important to avoid nesting your fetch calls to avoid a **waterfall request,**
if the requests are not dependent on each other then there is no reason to wait for the previous one to resolve before proceeding to the subsequent one.

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

#### Differences between `Promise.all` and `Promise.allSettled`

**Note:** one key difference between `Promise.all` and `Promise.allSettled`
is that `Promise.all` will fail fast while `Promise.allSettled` will not
regardless of whether they fulfill or reject. Depending on your use case they can come in handy.

```js
const asyncOperation = (ms, value) => {
	return new Promise((resolve, reject) => {
		return setTimeout(() => {
			if (value === "error") {
				reject(new Error(value));
			} else {
				resolve(value);
			}
		}, ms);
	});
};

async function fetch1() {
	return await asyncOperation(1000, "Data fetched successfully");
}
async function fetch2() {
	return await asyncOperation(1000, "error");
}
async function fetch3() {
	return await asyncOperation(1000, "Data fetched successfully");
}

async function main1() {
	try {
		let [result1, result2, result3] = await Promise.all([
			fetch1(),
			fetch2(),
			fetch3(),
		]);

		console.log(result1);
		console.log(result2);
		console.log(result3);
	} catch (e) {
		console.log("Promise.all Error:", e.message);
	}
}

async function main2() {
	try {
		let [result1, result2, result3] = await Promise.allSettled([
			fetch1(),
			fetch2(),
			fetch3(),
		]);

		console.log(result1);
		console.log(result2);
		console.log(result3);
	} catch (e) {
		console.log("Promise.allSettled Error:", e.message);
	}
}

main1();
main2();
```

### Error handling

Asynchronous error handling can be a little confusing, but with the right set of tools and strategies provided to us by javascript we are able to handle errors properly.

In JavaScript, runtime errors are caught with the use of `try...catch` blocks. The code between the `try` braces is executed and errors are caught with the `catch` statement.

```js
try {
	throw new Error("This is a runtime failure");
} catch (error) {
	console.error("Error: ", error.message);
}
```

Avoid nested try catch statements, try to be as DRY (Don't Repeat Yourself) as possible. Nesting `try...catch` statements complicates the code. So one way to be DRY is to create a `try...catch` function which accepts a function, its arguments, and returns either the error or the resolved value. This is one way to handle the error in a cleaner and simplified manner.

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

	throw new Error(`Could not find user with id: ${id}`); // Success path (commented out for demonstration) // return `The blog data was fetched for user by id: ${id}`;
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

As you can notice it is much cleaner and readable.

### Cancellation of Asynchronous Operations

Asynchronous operations are very useful in computing whether frontend or backend etc. They allow us to fetch data, perform complex logic without blocking the UI, and create a more responsive user experience. Issues can arise in react when trying to fetch data but the component has already unmounted. So this is where cancellation comes in.

Cancellation gives us the power to gracefully skip an ongoing async operation. Imagine a user has already navigated away from the page, there is not a single reason to keep fetching the user data. So in this scenario, it makes sense to terminate or cancel the fetch request and improve user experience.

Below, I will explore how to achieve the same kind of logic as seen in some fetch requests in the browser. By using the `AbortController` a custom class extending the `EventEmitter` class, I will be replicating the logic but using our simple `wait` function.

```js
function wait(time, signal) {
	return new Promise((resolve, reject) => {
		const timeoutId = setTimeout(() => {
			console.log("API was called, operation was not aborted");

			resolve();
		}, time);

		if (signal) {
			signal.abort(timeoutId, resolve); // Resolve or Reject as you need
		}
	});
}
```

I will create an 'AbortController' class which will extend 'EventEmitter'. A core Javascript module for creating custom events.

```js
const EventEmitter = require("events");

class AbortController extends EventEmitter {
	constructor(options) {
		super(options);

		this.addListener();
	}

	addListener() {
		this.on("abort", () => {
			console.log("Listener added");
		});

		return this;
	}

	abort(timeoutId, handler) {
		clearTimeout(timeoutId);

		handler();

		console.log("Event aborted");
	}
}
```

The 2 methods of the `AbortController` are the following:

- `addListener`: This method will add an event listener, listening for the `abort` event

- `abort`: this is event which our event listener will listen for. And from there execute necessary logic to end the function

an instance of the `AbortController` class to be passed to the `wait` function

#### Putting it all together

In the main function, we create an instance of `AbortController`. This is not a 1:1 clone of how it was achieved in the fetch api on how to cancel a fetch request

```js
async function main() {
	try {
		let signal = new AbortController(); // Option 1: Using addListener for explicit control

		await wait(10000, signal.addListener()); // Option 2: Passing the AbortController instance directly // await wait(10000, signal); // await wait(10000); // Without abort controller

		console.log("Resolved request");
	} catch (e) {
		console.log("Rejected request");
	}
}

main();
```

### Conclusion

There are many topics related to mastering async/await that are impossible to cover in a single blog post. However, I believe the information above provides a solid overview to enhance your understanding.
