---
title: How node.js modules are loaded
date: 2017-10-29 22:11:00 Z
categories:
- node.js
tags:
- node.js
---

---
title: How node.js modules are loaded
date: 2017-10-28 00:00:00 Z
layout: post
---

Have you ever though how node.js modules are loaded? For example, if you are requiring same module in lots of different places, does node.js load it every time you require it? 

!Let's find out.

dummy-module.js:
```javascript
console.log('start of dummy js')
function Dummy() {
	console.log('in the dummy() function')
	this.PI = 3

	this.add = function (x,y){
		return x + y
	}

	this.mult = function (x,y){
		return x * y
	}
}
console.log('after dummy() function')

module.exports = new Dummy()
console.log('after module.exports')
```

index.js:
```javascript
console.log('start of index.js')
const dummy = require('./dummy-module')
console.log('dummy module is ',dummy)
console.log('end of index.js')
```

We have just 3 lines in index.js file. Now if we start index.js with node:

```bash
$ node index.js
start of index.js
start of dummy js
after dummy() function
in the dummy() function
after module.exports
dummy is Dummy { PI: 3, add: [Function], mult: [Function] }
end of index.js
```

It may seem weird to you of seeing the `in the dummy() function` after the `after dummy() function` output.
Let's elaborate the situation:
1. When our dummy module is required, dummy-module.js is run by node.js.
2. Before running dummy-module.js, JavaScript compiler hoists variables declared as var (since we don't have this, nothing happens).
3. Also in compilation phase, each function is assigned a name. So even if the function is called before it's declaration, it is still accessible and function is called as it's meant to be.
4. JavaScript engine runs the code.

So in our dummy module, Dummy() function is run in this line:
```javascript
module.exports = new Dummy()
```

And this line is just before this line :
```javascript
console.log('start of index.js')
```

This makes `in the dummy() function` text come after `after dummy() function` text.
Remember, in JavaScript functions doesn't get called unless you call them :) 
That's why engine just skips from start of the dummy module to end of the Dummy() function.

So what about the last line of dummy module? Does it run after the module is exported or before?
It seems like it is run before module is exported. But why? Didn't we already exported the module bu using module.exports? No. Because JavaScript exports the module when it finishes running the module. Think module.exports as a variable which is exported when the engine finished running the module. That means you may have as many lines as you like with the module.exports but only the last value will be exported:

```javascript
module.exports = 'first value'
module.exports = 'second value'
module.exports = 'third value'
```

In the example above, only the `third value` will be exported. See, module.exports is just a placeholder for what will be exported. Another question, what if we put some delay between those lines? Something like this?

dummy-module.js:
```javascript
console.log('start of dummy js')
function Dummy() {
	console.log('in the dummy() function')
	this.PI = 3

	this.add = function (x, y){
		return x + y
	}

	this.mult = function (x, y){
		return x * y
	}
}
console.log('after dummy() function')

module.exports = new Dummy()
module.exports = 'second value'
setTimeout(()=> module.exports = 'third value', 100)

console.log('after module.exports')
```

In this case, only 'second value' will be JavaScripte engine never waits to return a value, by the time engine reaches end of the dummy module, the current value of the module.exports is returned to index.js instance and we see this output:
```bash
$ node index.js
start of index.js
start of dummy js
after dummy() function
in the dummy() function
after module.exports
dummy module is second value
end of index.js
```
Even after the `setTimeout` callback is called, the dummy variable won't change in index.js :

index.js:
```javascript
console.log('start of index.js')
const dummy = require('./dummy-module')
setTimeout(()=>console.log('dummy module is',dummy), 200)
console.log('end of index.js')
```

The result will be almost same as above, the only difference is that the order of the output will be changed:
```bash
$ node index.js
start of index.js
start of dummy js
after dummy() function
in the dummy() function
after module.exports
end of index.js
dummy module is second value
```

Now we came to the first question, *"**does node.js load a module every time you require it**"*?

We can easily find out if node.js loads a module twice or not. For this. we should have another JavaScript file that requires our same dummy module. But since node.js can run only one process, we can't run them both in the same time unless we require one from another. With the dummy module code being the same, consider following codes:

side-module.js:
```javascript
console.log('* start of side-module')
const dummy = require('./dummy-module')
console.log('* dummy module from side-module:',dummy)
function Side() {
	console.log('* in the Side() function')
	this.sidePI = dummy.PI
}
console.log('* after the Side() function')
module.exports = new Side()
console.log('* end of side-module')
```
index.js:
```javascript
console.log('start of index.js')
const dummy = require('./dummy-module')
const side= require('./side-module')
console.log('dummy module from index.js:',dummy)
console.log('side module from index.js:',side)
console.log('end of index.js')
```
Now run index.js:
```bash
$ node index.js
start of index.js
start of dummy js
after dummy() function
in the dummy() function
after module.exports
* start of side-module
* dummy module from side-module: second value
* after the Side() function
* in the Side() function
* end of side-module
dummy module from index.js: second value
side module from index.js: Side { sidePI: undefined }
end of index.js
```

We only see `start of dummy js` once. That means dummy-module.js is run only once. If a JavaScript module is already loaded with a require, it is never run again. Compiler uses same module.exports value for all require calls. It is like singleton pattern is already embedded in node.js for modules.

What is the module is mutable? Meaning module returns different things depending on the parameter you provide? 

dummy-module.js:
```javascript
console.log('start of dummy js')
function Dummy(newPI) {
	console.log('in the dummy() function')
	this.PI = newPI || 3

	this.add = function (x,y){
		return x+y
	}

	this.mult = function (x,y){
		return x* y
	}
}
console.log('after dummy() function')
module.exports = Dummy
setTimeout(()=> module.exports = 'second value', 100)
console.log('after module.exports')
```
side-module.js:
```javascript
console.log('* start of side-module')
const dummy = require('./dummy-module')
const dummyObject = new dummy(3.14)
console.log('* dummy PI from side-module:',dummyObject.PI)
function Side() {
	console.log('* in the Side() function')
	this.sidePI = dummyObject.PI
}
console.log('* after the Side() function')
module.exports = new Side()
console.log('* end of side-module')
```
index.js:
```javascript
console.log('start of index.js')
const dummy = require('./dummy-module')
const dummyObject  = new dummy()
const side= require('./side-module')
console.log('dummy module from index.js:',dummyObject)
console.log('side module from index.js:',side)
console.log('end of index.js')
```


Now run index.js:
```bash
start of index.js
start of dummy js
after dummy() function
after module.exports
in the dummy() function
* start of side-module
in the dummy() function
* dummy PI from side-module: 3.14
* after the Side() function
* in the Side() function
* end of side-module
dummy module from index.js: Dummy { PI: 3, add: [Function], mult: [Function] }
side module from index.js: Side { sidePI: 3.14 }
end of index.js
```

The `in the dummy() function` text is shown twice, that's good. Because there has to be two different dummy modules depending on the parameter value. 