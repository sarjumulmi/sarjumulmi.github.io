---
layout: post
title:  "Arrow Function and (Lexical) *'this'*"
date:   2017-09-18 16:46:59 -0400
---


Normally functions will bind it’s *‘this’* to the object that calls or is bound to the function either implicitly or explicitly. Depending upon whether the function is invoked with a context object (eg: obj1.foo()), a new keyword (eg: var bar = new Foo()), or call, apply, or bind (foo.call(obj1), foo.bind(obj1)), *‘this’* of the function gets assigned to its callee object.

However, the same rule doesn’t apply to arrow functions. Syntactically, arrow functions are denoted by a fat arrow operator (=>) instead of a function keyword.  So, *‘this’* of an arrow does not get bound to the callee object, but rather is assigned from the enclosing or parent scope/function. If the enclosing or parent function is missing, *‘this’* will be assigned as *window*, *global* or *undefined*.  

Let’s look at a few examples with the traditional function vs arrow function syntax:

**Example 1:**
```
var obj1 = {
a :"inside obj1", 
b: function(){console.log(`Value of a is: ${this.a}`)}, 
c: function(){return function() {console.log(`Value of a is: ${this.a}`)}}
};

var obj2 = {a:"inside obj2"}
```
**Scenarios**:
```
a.	obj1.b() #=> Value of a is: inside obj1
b.	obj1.b.call(obj2) #=> Value of a is: inside obj2
c.	var boundB = obj1.b.bind(obj2)
d.	boundB() #=> Value of a is: inside obj2
e.	var unbound = obj1.b
f.	unbound() #=> Value of a is: undefined
g.	obj1.c()() #=> Value of a is: undefined
```
Note example 1(g); the nested function returned creates its own environment and hence *'this'* becomes *'window'*.
**Example 2:**
Now, if we replace *‘b’* with an arrow function: 
```
var obj3 = {
a :"inside obj3", b: ()=>console.log(`Value of a is: ${this.a}`)
}
```
**Scenario**:
```
a.	obj3.b() #=>Value of a is: undefined
```
That’s because *‘this’* for *‘b’* is not the callee object *‘obj3’* but the surrounding function/scope for *‘b’* which is *‘window’* in this case. This quirky property of arrow function might come handy when we have to pass *‘this’* to nested functions. As opposed to example 1(g) above, *‘this’* for nested arrow function will look up to the surround scope/function’s *‘this’* rather than create its own *‘this’*. 

For example:
**Example 3:**
```
var obj4 = {a :"inside obj4", b: function(){return ()=>console.log(`Value of a is: ${this.a}`)}}
```
**Scenarios**:
```
a.	obj4.b()() #=> Value of a is: inside obj4
b.	obj4.b().call(obj2) #=> Value of a is: inside obj4
c.	var boundB = obj4.b().bind(obj2)
d.	boundB() #=> Value of a is: inside obj4
```

**4.	Example 4:**
If we try to use arrow function to define both the outer & inner functions with arrow function syntax, we basically end up with the same situation as with example 2.
```
var obj5 = {a :"inside obj5", b: () => { return ()=>console.log(`Value of a is: ${this.a}`)}}
```
**Scenario**:
```
a.	obj5.b()() #=>Value of a is: undefined
```

**References**:
> 1.	[You Don’t Know Javascript](https://github.com/getify/You-Dont-Know-JS) - Kyle Simpson
> 2.	[http://wesbos.com/arrow-functions-this-javascript/](http://wesbos.com/arrow-functions-this-javascript/) - Wes Bos




