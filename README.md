# Intro to JavaScript

JavaScript is a lightweight, dynamic, weakly typed language. It features prototype-based objects, first-class functions, and asynchronous events.

Being lightweight means that JavaScript is a small language. Dynamic means that instead of compiling down to machine code, its interpreted line by line at runtime. This means that if there is a syntax error half way through a program, all the code before the error will run, and then the program will fail.

Here's an example of Hello World in JavaScript.
```javascript
console.log("Hello world.");
```
Note that theres no main function, the code starts being interpreted at the top of the file and just runs. ```console.log``` is actually not part of Javascript, but since Javascript runs in a browser it doesn't have stdin and stdout, it instead uses the console object provided by the browser and writes to the browsers debugging console.

Here's an example of a function you probably recognize, isPalindrome, written in JavaScript. 

```javascript
function isPalindrom(word){
	var length = word.length;
	for(var i=0; i < length/2; i++){
		if(word.charAt(i) !== word.charAt(length-i)){
			return false;
		}
	}
	return true;
}

isPalindrome("racecar"); // true
isPalindrome("hello"); //false
```

The function takes a word, loops through half of the list and compares it to the other half of the list. When it finds an inconsistancy it returns false, otherwise it returns true.

Look at the first line of the function

```javascript
var length = word.length;
```

word.length clearly returns a number, but we declare length not as an int, but as a var, as in variable.

If we declared length to be a String it would look like this,

```javascript
var length = "hello";
```

If we declared length to be an array it would look like this,

```javascript
var length = [1, 2, 3];
```

In fact we could redeclare length to be all those things,

```javascript
var length = word.length;
length = "hello";
length = [1, 2, 3];
```

This is what it means that JavaScript is weakly-typed. It gives no fucks about what type the actual variable is, it's always of type "var".

This leads to the next feature of JavaScript. It has first-class functions. This means we can assign functions to variables and treat them like variables.

```javascript
var isPalindrom = function(word){
	var length = word.length;
	for(var i=0; i < length/2; i++){
		if(word.charAt(i) !== word.charAt(length-i)){
			return false;
		}
	}
	return true;
};

isPalindrome("racecar"); // => true
isPalindrome("hello"); // => false
```

Note how in this implementation, instead of saying 

```javascript
function isPalindrome(word){ ... }
```

we say

```javascript
var isPalindrome = function(word){ ... };
```

We're now declaring a variable called isPalindrome and setting it equal to an _anonymous function_.

__Definition:__ An _anonymous function_ is any function in the form ```function(){ .. };```. You can think of these as functions without names, even though they're commonly set to a named variable.

So anonymouse functions are part of why we say functions are _first-class citizens_ in JavaScript. They're treated like data and can be set into variables. The second, more useful power is that they can be passed into function just like you would any other data!

```javascript
var sayWorld = function(){
	console.log("world");
};

var sayHello = function(){
	console.log("hello");
}

var runFuns = function(fun1, fun2){
	fun1();
	fun2();
};

runFuns(sayHello, sayWorld);
// => "hello"
// => "world"


runFuns(sayWorld, sayHello);
// => "world"
// => "hello"


runFuns(sayHello, function(){
	console.log("devon");
});
// => "hello"
// => "devon"
```

Note how in that last one I just declared an anonymous function right in the execution of the runFuns function.

# The JavaScript Event Loop

Javascript by itself can't really do all that much. The strength of JavaScript comes from its abillity to use functions tha the browser gives it access to. One of those functions we will be using a lot is setTimeout. The purpose of setTimeout is to delay code from executing.

As we discussed in the previous section, functions can be passed into other functions in JavaScript. setTimout takes two parameters, a function, and then a time in miliseconds to be delayed before that function is run.

```javascript
setTimeout(function(){
	console.log("hello world");
}, 5000);
```

If we run this we see that setTimeout delays "hello world" from being printed for five seconds. You might think this is like the sleep function in C, but it's not. This is where things get crazy.

```javascript
setTimeout(function(){
	console.log("hello");
}, 5000);
console.log("world");
```

If setTimeout worked like sleep, this would log "hello" and then "world". __But it doesn't.__ Instead is logs "world" __and then__ "hello".

JavaScript was made as an event driven language. When something happens in the browser, some code should run. "Something happens in the browser" could be a form submission, or a keystroke, or even just a certain amount of time passing. As a programmer, we don't want to be responsible for constantly checking the time or button presses, so instead, in the background, JavaScript runs a loop constantly checking for new events that get queued up by the browser.

setTimeout is our way of telling the browser to simulate one of these events after a period of time. Whenever we create one of these _event listeners_ we write an anonymouse function called a _callback_ that gets called once the event happens.

Lets look at some examples:

```javascript
setTimeout(function(){
	console.log("world");
}, 5000);
console.log("hello");
```

This is the correct version of our function from before. It will say "hello" imediately and then five seconds later say "world".

Imagine we wanted to say "hello" after five seconds and then "world" five seconds after that. An incorrect way to do this would be...

```javascript
setTimeout(function(){
	console.log("hello");
}, 5000);
setTimeout(function(){
	console.log("world");
}, 5000);
```

This _wont_ display the messages five seconds appart. Instead both messages will display at the same time after five seconds. This is because we set the setTimeouts at the same time.

One possible solution is to tell our browser to just run the first setTimeout (_hello_) after five seconds, and our second setTimeout (_world_) after ten seconds.

```javascript
setTimeout(function(){
	console.log("hello");
}, 5000);
setTimeout(function(){
	console.log("world");
}, 10000);
```

This works... sometimes. Unfortunately the browser can't guarantee that it will call our code after the exact number of milliseconds we want since it also needs to handle browser things like loading tabs, pages, and executing other JavaScript. If it's really busy it might take more than ten seconds to realize how much time has passed and run them at the same time. The adding method is also subjectable to the programmers bad math errors.

The idea of one thing coming after the other maters to us because this is how we're going to animate things in our games.

The better way to do this is to set our setTimeout for "hello" first, and then in the anonymous function where we're saying "hello" we imideately after set a second seTimeout for five seconds later.

```javascript
setTimeout(function(){
	console.log("hello");
	setTimeout(function(){
		console.log("world");
	}, 5000);
}, 5000);
```

Note how I'm stacking callbacks inside of callbacks now. This can get pretty ugly pretty fast, but we can clean it a little by using callbacks ourselves in our own functions.

```javascript
var sayHello = function(time, cb){
	setTimeout(function(){
		console.log("hello");
		cb();
	}, time);
}

var sayWorld = function(time, cb){
	setTimeout(function(){
		console.log("world");
		cb();
	}, time);
}

sayHello(5000, function(){
	sayWorld(5000);
});
```
This is a much more modular way of structuring our events because now we can plug and play them in any order, or easily make things happen between them.

One last thing that you need to understand about the event queue is that it only runs one thing at a time. Consider the following code...

```javascript
setTimeout(function(){
	console.log("world");
}, 5000);
console.log("hello");
while(true){};
```

The browser will be told "in five seconds run this function that says world", and in five seconds the browser will put that function call into a queue that the event loop is constantly checking. But imediately after we say "hello" we've started a while(true) loop that will never end and thus never let the current bit of code finish. Until this code finishes running, the event loop will never check the queue and our ```console.log("world")``` will never run.


