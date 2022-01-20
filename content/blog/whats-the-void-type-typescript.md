+++
title = "What's the void type in TypeScript?"
date = "2020-01-03"
tags = ["typescript", "code"]
+++
One of the more confusing types in the TypeScript universe is the `void` type. The most common place it’s used is as the return type for a callback function. For example, the type of the callback you pass to the Array `forEach` function is this:

```ts
type CallbackFn<T> = 
	(value: T, index: number, array: readonly T[]) => void;
```

The `forEach` callback accepts three arguments: the current value (`T`), the current index, and the array (`readonly T[]`). Its return type is the `void` type.

Most people think that the `void` type means ‘returns undefined’, and that `void` and `undefined` are interchangeable. For example, TypeScript accepts both of these callback functions as valid:

```ts
const logNumbersVoid = (num: number): void => {
	console.log(num);
	return undefined;
}

const logNumbersUndefined = (num: number): undefined => {
	console.log(num);
	return undefined;
}

// TypeScript doesn't complain about either of these.
[1, 2, 3].forEach(logNumbersVoid);
[1, 2, 3].forEach(logNumbersUndefined);
```

These two functions (`logNumbersVoid` and `logNumbersUndefined`) are the same, except for their return type. Both accept a `number` as their parameter, log the number, and return `undefined`. The first function’s return type is `void`, and the second one is `undefined`. 

TypeScript allows us to use both of these as the callback to the `forEach` function. So, it seems like `void` and `undefined` are doing the same thing.

However, there are some cases where `void` allows some things that `undefined` _does not_:

```ts
const logNumbersVoid = (num: number): void {
	console.log(num);

	// Note that we're returning `num` in both functions.
	return num;
}

const logNumbersUndefined = (num: number): undefined {
	console.log(num);
	
	// TypeScript complains about this but not the other one???
	return num;
}
```

Here, we change both functions to return `num` instead of `undefined`. TypeScript complains about the `return` statement in `logNumbersUndefined`, but doesn’t complain about the `return` from the `void` function.

What’s going on here? Why are we allowed to return a number from a `void` function but not from an `undefined` function? Let’s dig deeper.

## Where does `void` come from?

Several constructs in TypeScript have similarly named constructs in JavaScript. For example, TypeScript’s `typeof` operator comes from the `typeof` operator in JavaScript, working similarly to it in type definitions. It’s useful to understand the JavaScript version of these TypeScript constructs. Knowing the behavior in JavaScript can help us predict how they’ll behave in TypeScript.

Just like `typeof`, TypeScript’s `void` type has a counterpart in JavaScript: the `void` keyword.

The `void` keyword in JavaScript is a keyword that can be put before any expression (something evaluating to a value). JavaScript will evaluate the expression and then return the value `undefined` for the entire expression. So in the example below, “It returned undefined!” would be logged to the console:

```ts
if (void "helloWorld" === undefined) {
	console.log("It returned undefined!");
}
```

`void` works similarly to `typeof` in that it evaluates the expression to the right of it. However, `void` _throws away the result of the expression_. In this way, `void` sort of acts like a trash can: **the value of any expression you give it will not be accessible again**. Hold on to that analogy for a minute.

### Use of `void`

It might seem like the `void` keyword doesn’t have much of a use case now, and you’d be right in thinking that. But before ES5 JavaScript, it had an important use case: getting the value `undefined`.

In JavaScript, there are two versions of `undefined`: the _value_ and the _variable_. For whatever goddamn reason, `undefined` is actually a _global variable_ in JavaScript, not a reserved word. By default, this variable points to the _value_ `undefined`. Tricky, I know. 

Before the ES5 standard, any script could modify the contents of the `undefined` variable. Running something like `window.undefined = "HAHA FUCK YOU"` could potentially screw up a large part of most working programs (e.g., conditions like `variableName === undefined` would return false).

Because of this, many JavaScript developers would use the expression `void 0` to obtain the `undefined` value. Contrary to the `undefined` variable, `void` is a reserved word that cannot be modified or changed. This meant that `variableName === void 0` would always return true if `variableName` was `undefined`, even if someone re-assigned the `undefined` variable.

The ES5 standard changed `undefined` so that it was read-only, so thankfully these kinds of bugs / exploits no longer exist. The resolution of this issue removed one of the main use cases for `void`. As a result, most JS programmers don’t know about it.

## `void` in TypeScript

[The TypeScript Handbook describes `void` like this](https://www.typescriptlang.org/docs/handbook/2/functions.html#void):

> `void` represents the return value of functions which don’t return a value. It’s the inferred type any time a function doesn’t have any return statements, or doesn’t return any explicit value from those `return` statements.

So `void` is a type used when our functions don’t have any return value. If we don’t have any return statements in the function, TypeScript will automatically assume that our function returns `void`. This explains the typing of the `Array.prototype.forEach` callback: `forEach` isn't concerned with the result of the callback, only the _parameters_ it needs to pass to it. 

If `void` is inferred by TypeScript only when nothing is returned from the function, why could we return `undefined` in the example at the beginning of the post? The handbook goes on:

> In JavaScript, a function that doesn’t return any value will implicitly return the value undefined. However, `void` and `undefined` are not the same thing in TypeScript.

In JavaScript, any function that doesn’t return a value automatically returns `undefined`. As consumers of a function, we have no way to tell whether the return of `undefined` was explicit (`return undefined;`) or implicit (no `return` statement). Therefore, TypeScript allows us to return `undefined` from a `void` function. As consumers of that function, we can’t tell the difference between the two.

But hold on a second, why does the handbook say that `void` and `undefined` are different? [Scrolling down to the bottom of the page](https://www.typescriptlang.org/docs/handbook/2/functions.html#return-type-void), we read this:

> Contextual typing with a return type of void does **not** force functions to **not** return something. Another way to say this is a contextual function type with a `void` return type (`type vf = () => void`), when implemented, can return _any_ other value, but it will be ignored.

This is saying that when we have an explicit return type of `void` (like in both of our functions before), we can return _any_ type from our function. In other words, _any_ type is assignable to `void` when we’re returning from a function.

This is not the case with a function with a return type of `undefined`: TypeScript will force us to either return nothing, or explicitly return `undefined`. Contrary to this, `void` doesn’t care. We can return implicit `undefined`, explicit `undefined`, or _any explicit value we want_.

Remember what we said about JavaScript’s void being a trash can? TypeScript’s `void` is the same way, but as a type. Once you assign something to `void`, TypeScript _won’t let you use it again_. You can put stuff into it, but you can’t get stuff back out. Once you tell TypeScript that something is `void`, it throws out whatever type it was before and starts treating it like nothing. It’s a trash can.

So that’s how `void` works in TypeScript. If you have any questions or comments, feel free to comment on the post below. I also have a newsletter you can subscribe to (I only send one email a week) if you’re interested in more posts like this. Last, feel free to [follow me on Twitter](https://twitter.com/samwightt) to keep up-to-date with the latest things I’m working on.