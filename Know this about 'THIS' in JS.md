First of all if you have ever worked with any other programming language you must have encountered most developers controversial topic - the 'THIS' keyword and to add on top of that if you are developer who loves to code in JavaScript - I FEEL YOU BRO! The complexities of 'this' in JavaScript often leaves most experienced developers wondering about its various behaviors.

Don't fret MABWOOY -`Insert ZNMD meme here.` 
![[Pasted image 20240214155643.png]]

I'm here to help and share how I got it sorted for me all thanks to the YDKJS series. If you live under a rock then come out and learn about the one of the best open and free book written on JavaScript simplifying various challenges and explaining its intricacies in detail. You don't Know JS series is the books name. Do give it a read. 

One thing which is simple to understand is that you show be able to detect the Call Site of where the 'this' keyword was invoked from. That's just the beginning of the complete mystery but it is an integral part of it. 
An example for the same would be.
```js
function baz() {
	// call-stack is: `baz`
	// so, our call-site is in the global scope
	console.log( "baz" );
	bar(); // <-- call-site for `bar`
}

function bar() {
	// call-stack is: `baz` -> `bar`
	// so, our call-site is in `baz`
	console.log( "bar" );
	foo(); // <-- call-site for `foo`
}

function foo() {
	// call-stack is: `baz` -> `bar` -> `foo`
	// so, our call-site is in `bar`
	console.log( "foo" );
}

baz(); // <-- call-site for `baz`
```

It looks hard to find the call site but it is just remembering some rules.
#### Default Binding 
- For standalone function invocations, it applies when everything else fails. Think of this as the default rule. 
   