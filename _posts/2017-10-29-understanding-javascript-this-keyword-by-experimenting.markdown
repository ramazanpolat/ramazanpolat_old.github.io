---
title: Understanding Javascript this keyword by experimenting
date: 2017-10-29 22:32:00 Z
tags:
- javascript
---

I'm not gonna teach you what `this` is, I'm gonna show you how I learned what `this` is by experimenting. You can do the same to understand `this`. In fact, you can use this method to understand lots of things. I call it 'learning by experimenting'.

Open up Chrome Developer Tools (F12 on Windows) and type these(you literally need go and type these, just reading won't be enough!):
```javascript
this
console.log(this)
this == window //true
```
As you can see, `this` is the global context in the browser, which is `window` object.
So what we learned?
**We learned that `this` gives you the context you are bound to.** 
Good, now let's continue learning by experimenting, because the above statement is not enough to explaint JavaScript `this`.

```javascript
var ob1 = {
	name: 'object1'
}
ob1 == this.ob1 // true
if (ob1 == window.ob1)
	console.log("ob1 belongs to window")
else
	console.log("ob1 doesn't belong to window")
// outputs: ob1 belongs to window
```
Let's say we want to write a function that checks if a variable belongs to window or not.
```javascript
//silly non-parametric function
function is_ob1_member_of_window(){
	return ob1 == window.ob1
}
is_ob1_member_of_window() //true

//using IIFE
(function(){return ob1 == window.ob1})() //true

//semi parametric function, still silly
function is_member_of_window(variable){
	return variable == window.variable
}
is_member_of_window(ob1) //false!
```
We are clearly doing something wrong! How do we get false?
Well, because we are accessing `window.variable`, not `window.ob1`! 
Since `window.variable` is `undefined`and result is false.

Attempt to fix:
```javascript
// most cumbersome function, ever!
function is_member_of_window(variable, variable_name){
	return variable == window[variable_name]
}
is_member_of_window(ob1,'ob1') // true
```

Better fix:
```javascript
// most cumbersome function, ever!
function is_member_of_window(variable_name){
	if (this.hasOwnProperty(variable_name))
		if (window.hasOwnProperty(variable_name))
			return this[variable_name] == window[variable_name]
	return false;
}
is_member_of_window('ob1') // true
```
That has gone too far from out point. Need to get back on track. The last thing we learned was that: **`this` gives you the context you are bound to.** 

**Whenever JavaScript executes a method, the context is the object that method belongs to.**
This brings some questions: What makes a function method? What is the difference?

The method is a function attached to an object and called usually referenced with a dot after the object. 
```javascript
var myMathObject = {
	pi: 3.14,
	add: function(a,b) {
		return a + b
	}
}
myMathObject.add(3,4) // 7
```

In the above example, `add` is a method of `myMathObject`, that's why we are referencing it with a dot between the object name and the function as `myMathObject.add`.  In order to call it, you also need to pass parenthesis `()` to the referenced function. Ok, so far so good. But is this a method?

```javascript
function add(a,b) {
	return a + b
}
add(3,4) // 7
window.add(3,4) // 7
```
Yes it is! It's the method of the current execution context, which is `windows`, just because we are already in that context, we don't need to use dot to reference the variable(remember functions are also variables in FP). This brings another question: are all functions also a method? No, they are not and I will explain this another post.

Continuing, if `this`  gives the global object when we are in global context, then what does it hold if we call it with another object? Let's experiment it!

```javascript
var myMathObject = {
	pi: 3.14,
	add: function(a,b) {
		return a + b
	},
	printContext: function(){
		console.log(this)	
	}
}
myMathObject.printContext()
// {pi: 3.14, add: ƒ, printContext: ƒ}
```
Wow, this time it hold the object of the method it belong to. 
```javascript
var myStringObject= {
	str: 'String'
}
myStringObject.printContext = myMathObject.printContext
myStringObject.printContext()
// {str: "String", printContext: ƒ}
```
And this made it clear that it gets changed depending on where it is called.
Our conclusion is :
### Whenever you call a method of an object, the value of `this` is set to the object, whose method is being called.

