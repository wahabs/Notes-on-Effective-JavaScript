# Notes on *Effective JavaScript* by David Herman

Intended to be a supplement to anyone's studying of JavaScript, whether through
[Herman's book](http://www.amazon.com/Effective-JavaScript-Specific-Software-Development/dp/0321812182) or elsewhere. Does not attempt to be neutral or objective on matters of best practice.

Released under the [MIT License.][license]. Issues/pull requests welcome.

[license]: ./LICENSE.md

### Chapter 1 - Accustoming Yourself to JavaScript

#### Item 2: Understand JavaScript's Floating Point Numbers
All numbers in JS are doubles, classified as `number`. Integers are also classified under this type. Be aware of limitations in floating-point arithmetic.

#### Item 3: Beware of Implicit Coercions
A lot of the time, JavaScript doesn't raise errors when it should (a good reason to `"use strict";`). This can occur when doing operations that don't make sense, such as `'2' + 3`. Make sure you know the data types you're dealing with and not falling into an implicit coercion.

Make sure to always use `typeof` or comparison to `undefined` rather than truthiness to test for undefined values.

#### Item 4: Prefer Primitives to Object Wrappers
JavaScript's primitives are:

1. Booleans
2. Numbers
3. Strings
4. `null`
5. `undefined`

Technically you can use an object wrapper to define a primitive, e.g. `var s = new String("hello")`. The variable `s` is now a `typeOf` object, *not* a string in the primitive sense. This can cause unexpected behavior, such as the normal comparison `===` to an equivalent string would yield `false` because they are different objects. For this reason, you should prefer primitives to object wrappers.

The reason object wrappers exist then is for their implicit coercion. You can call predefined methods on primitives as if they were objects, e.g. `"hello".toUpperCase();`. JS will create an implicit wrapping, creating `new String("hello")` and then calling the `toUpperCase` method on it.

#### Item 5: Avoid using `==` with mixed types
The `==` comparison operator is less strict and will try to coerce types when comparing, so stick to `===`.

#### Item 6: Learn the limits of semicolon insertion
Ignore the book and just use semicolons explicitly.

#### Item 7: Think of Strings as Sequences of 16-Bit Code Units
In Unicode, every unit of text of all the world’s writing systems is assigned a unique integer between 0 and 1,114,111, known as a *code point* in Unicode terminology. In this sense it is similar to ASCII. The difference, however, is that while ASCII maps each index to a unique binary representation, Unicode allows multiple different binary encodings of code points. These encodings make trade offs between storage/performance, and there a several standard encodings of Unicode. The most popular are UTF-8, UTF-16, UTF-32.

As far as their properties and methods are concerned, JS strings behave like sequences of UTF-16 code units. This will cause problems for applications working with the full range of Unicode (i.e. dealing with special characters). They can’t rely on string methods, length values, indexed lookups, or many regular expression patterns. If you are working outside the BMP, it’s a good idea to look for help from code point-aware libraries. It can be tricky to get the details of encoding and decoding right, so it’s advisable to use an existing library rather than implement the logic yourself.


### Chapter 2 - Variable Scope

#### Item 8: Minimize use of the global object
Defining global variables pollutes the common namespace shared by everyone, introducing the possibility of accidental name collisions. And if a global variable changes in value, it's much more difficult to figure out what function affected the change (breaking principle of encapsulation). For these reasons global variables are reserved for static constants.

JavaScript's global object is bound to the `window` variable in browsers. We generally attach our own namespaces as objects to this (e.g. `window.App`).

#### Item 9: Always declare local variables
Meaning, declare them as local with `var`.

#### Item 10: Avoid `with`
Basically, `with` is a method intended to act as a shorthand for chaining a sequence of methods together on a specific object. JavaScript treats all variables the same: It looks them up in scope, starting with the innermost scope and working its way outward. If the property is not found in the object, then the search continues in outer scopes  (i.e. traversing the *lexical environment*).  The `with` statement treats an object as if it represented a variable scope, so inside the `with` block, variable lookup starts by searching for a property of the given variable name and traversing upwards until a variable definition is found.

The problem is that `with` intends to bring multiple variables and methods together, and so there can easily be a conflict in the namespace (especially if we're passing in module such as `Math` to the `with` block). Every reference to an outer variable in a `with` block implicitly assumes that there is no property of the same name in the `with` object—*or in any of its prototype objects.* And so the `with` block is sensitive to any clashes between the variable scope and the object namespace, and is therefore best avoided.

#### Item 11: Get comfortable with closures

A JavaScript function *encloses* (i.e. maintains a reference to) all variables immediately outside of its local scope. An inner function would enclose all of its outer function's variables (including arguments that were passed in) even if the outer function has already returned. This is because JavaScript functions are first-class objects (see Item 19). As an example:

```js
function sandwichMaker(magicIngredient) {
	return function(filling) {
		return magicIngredient + " and " + filling;
	};
}
var hamSandwich = sandwichMaker("ham");
hamSandwich("cheese"); // "ham and cheese"
sandwichMaker("turkey")("Swiss"); // "turkey and Swiss"
```

Note that because a closure retains references to the enclosed variables, it can also update those variables. So updates are visible to any other closures that have access to the variables. Example:

```js
function box() {
var val = undefined;
	return {
		set: function(newVal) { val = newVal; },
		get: function() { return val; },
		type: function() { return typeof val; }
	};
}
var b = box();
b.type(); // "undefined"
b.set(98.6);
b.get(); // 98.6
b.type(); // "number"
```

#### Item 12: Understand variable hoisting

JavaScript supports *lexical scoping:* With only a few exceptions, a reference to a variable `foo` is bound to the nearest scope in which `foo` was declared. However, JavaScript does not support block scoping: Variable definitions are not scoped to their nearest enclosing statement or block, but rather to their containing function.

In each function, variables are *hoisted*, meaning that their declaration (e.g. `var x;`) is made at the beginning of the function even if their assignment occurs elsewhere (e.g. `var x = 10;`). This is true even if the program never gets to the assignment, such that this:

```js
function foo() {
	if (false) { var x = 1; }
	return;
	var y = 1;
}
```

is equivalent to

```js
function foo() {
	var x, y;
	if (false) { x = 1; }
	return;
	y = 1;
}
```

Note that hoisting also affects functions, but if the function body is assigned to a variable only the variable declaration gets hoisted;

```js
foo(); // TypeError "foo is not a function"
bar(); // valid

var foo = function () {}; // anonymous function expression ('foo' gets hoisted)
function bar() {}; // function declaration ('bar' and the function body get hoisted)
```


Example: Note that in the following code:
```js
var foo = 1;
function bar() {
	if (!foo) {
		var foo = 10;
	}
	alert(foo);
}
bar();
```

The last line alerts `10` because the declaration of `foo` inside `bar()` gets hoisted, and so `foo` becomes an undefined local variable in `bar()` rather than an enclosed variable. And so we enter the `if` block, set `foo = 10`, and alert it.

The best way to avoid confusion is to hoist manually, i.e. simply declare all your variables at the top of a function under a single `var`, and only call functions after they've been defined in your code.


####  Item 13: Use Immediately Invoked Function Expressions (IIFEs) to Create Local Scopes

Observe the following buggy code:

```js
function wrapElements(a) {
	var result = [];
	for (var i = 0, n = a.length; i < n; i++) {
		result[i] = function() { return a[i]; };
	}
	return result;
}
var wrapped = wrapElements([10, 20, 30, 40, 50]);
var f = wrapped[0];
f(); // undefined, not 10!
```

What the programmer might have assumed was that in each iteration, they were storing a function that would return the array's value at `i`. However, *closures store their outer variables by reference, not by value.* So what happens here is that `i` gets hoisted, and since the value of `i` changes after each function is created, the inner functions end up seeing the final value of `i`. The final value of `i` is of course `a.length`, so that each function in `wrapped` would return `a[a.length]` which is undefined.

A solution here is to use an IIFE to create a local scope:

```js
function wrapElements(a) {
	var result = [];
	for (var i = 0, n = a.length; i < n; i++) {
		(function() {
		var j = i;
		result[i] = function() { return a[j]; };
		})();
	}
	return result;
}
```

Since the declaration of `j` is in a new function scope, we don't have to worry about the hoisting effect - it only lives within our IIFE. Now, at each iteration the IIFE is invoked, meaning the value from the enclosed reference to `i` gets assigned to a newly declared `j` and the function returning that value is assigned to `result[i]`. In this way, an IIFE can simulate a block-level scope by wrapping the block's code in an anonymous function.

#### Item 14: Beware of Unportable Scoping of Named Function Expressions

If you are shipping in properly implemented ES5 environments, you’ve got nothing to worry about.

#### Item 15: Beware of Unportable Scoping of Block-Local Function Declarations

Always keep function declarations at the outermost level of a program or a containing function to avoid unportable behavior. Use `var` declarations with conditional assignment instead of conditional function declarations.

#### Item 16: Avoid Creating Local Variables with `eval()`

`eval()` is a function that will parse a string passed in and execute it as JavaScript code. Ignore the book - you really shouldn't use it at all, as it's vulnerable to command injection.

#### Item 17: Prefer Indirect `eval()` to Direct `eval()`

See above.

### Chapter 3: Working with Functions

In JavaScript, functions alone play roles that other languages fulfill with multiple distinct features: procedures, methods, constructors, and even classes and modules.

#### Item 18: Understand the difference between function, method, and constructor calls

The normal way of calling a function in JavaScript is often referred to as *function-style.*

```js
function hello(name) { return "Hello I'm " + name; }
hello("Ed"); // "hello I'm Ed"
```

A method in JavaScript is no more than a property defined on an object (or its prototype) that happens to be a function. We call this function *method-style.*

```js
var person = {
	name: "Ed",
	hello: function() { return "Hello I'm " + this.name; }
}

person.hello(); // "Hello I'm Ed"
```

Technically all functions are called as methods on the global Object, though this is not typically something we can or should use to our advantage.

A function called *constructor-style* will pass a brand-new object as the value of `this`, and implicitly return the new object as its result. The constructor function’s primary role is to initialize the object.

```js
function Person(name, age) {
	this.name = name;
	this.age = age;
}

Person.prototype.greet = function() {
	return "Hello I'm " + this.name + "and I'm " + this.age
}

var p = new Person("Ed", 22);
p.greet(); // "Hello I'm Ed and I'm 22"
```

#### Item 19: Get comfortable using higher-order functions

Higher-order functions are functions that return functions or take in functions as arguments. Functions passed in as arguments are referred to as *callbacks* because they are called back by the higher-order function.

An example is the `sort` method, which accepts a comparator function as a callback.

```js
var arr = [3, 1, 4, 1, 5, 9];
// Note: this is the comparator `sort` is using anyway
arr.sort(function (x,y) {
	if (x < y) {
		return -1;
	}
	if (x > y) {
		return 1;
	}
	return 0;
}); // [1, 1, 3, 4, 5, 9]
```

If you have multiple chunks of code that are performing slightly different tasks, it is probably beneficial to consolidate them into a function that accepts a callback to execute the different behavior.

A more developed example:

```js
var readline = require('readline');
var reader = readline.createInterface({
  input: process.stdin,
  output: process.stdout
});

function addTwoNumbers (callback) {
  // You never return in async code since the caller will not wait for it

  reader.question("Enter #1", function (numString1) {
    reader.question("Enter #2", function (numString2) {
      var num1 = parseInt(numString1);
      var num2 = parseInt(numString2);

      callback(num1 + num2);
    });
  });
}

// will take in two numbers, add them, and log the corresponding string below
addTwoNumbers(function (result) {
  console.log("The result is: " + result);
  reader.close();
});
```

#### Item 20: Use `call` to call methods with a custom receiver

Functions come with a built-in `call` method for providing a custom receiver. Invoking a function like `myFunc.call(myObj, arg1, arg2)` allows `myFunc` to be called method-style on `myObj` (so that `this` is set appropriately) even when `myObj` does not have a `myFunc` method.

The `call` method can also be useful when defining higher-order functions. A common idiom for a higher-order function is to accept an optional argument to provide as the receiver for calling the function. For example, an object that represents a table of key-value bindings might provide a `forEach` method:

```js
function Table() {
	this.entries = [];
}

Table.prototype.addEntry = function(key, value) {
	this.entries.push({ key: key, value: value });
}

Table.prototype.forEach = function(receiver, callback) {
	var entries = this.entries;
	for (var i = 0; i < entries.length; i++) {
		var entry = entries[i];
		callback.call(receiver, entry.key, entry.value);
	}
}
```

This allows consumers of the object to use a method as the callback function `callback` of `table.forEach` and provide a sensible receiver for the method. For example, we can conveniently copy the contents of one table into another, e.g. `table1.forEach(table2, Table.prototype.addEntry)`.

#### Item 21: Use `apply` to call functions with different numbers of arguments

`apply` is equivalent to `call` except the arguments are passed in as an array. In general, `call` is more convenient when you know ahead of time what arguments you want to pass. `apply` is more useful when someone is going to give you an array of arguments to use.

#### Item 22: Use `arguments` to create variadic functions

In any JavaScript function definition, the number of arguments to be passed in is only a suggestion - the function can be called with fewer arguments (the remaining variables will remain `undefined`) or more arguments (the extra arguments won't get assigned to any explicit variables).

However, JavaScript provides every function with an implicit local variable called `arguments`. The `arguments` object provides an array-like interface to the *actual* (passed in) arguments: It contains indexed properties for each actual argument and a `length` property indicating how many arguments were provided. Like `this`, `arguments` is reset every time we enter a new function.

Observe the following custom `bind` method.
```js
Function.prototype.myBind = function myBind (context) {
  var that = this;
  // since `arguments` is indexed and has a `length` property, we can call `slice` on it
  var bindArgs = Array.prototype.slice.call(arguments, 1);
  return function() {
    var callArgs = Array.prototype.slice.call(arguments);
    return that.apply(context, bindArgs.concat(callArgs));
  };
};
```

A good rule of thumb is that whenever you provide a variable-arity function for convenience, you should also provide a fixed-arity version that takes an explicit array. This is usually easy to provide, because you can typically implement the variadic function as a small wrapper that delegates to the fixed-arity version.

```js
function average() {
	return averageOfArray(arguments);
}
```

This way, consumers of your functions don’t have to resort to the `apply` method, which can be less readable and often carries a performance cost.


#### Item 23: Never modify the `arguments` object

Within a function, named arguments are *aliases* to their corresponding indices in the `arguments` object. The relationship between the `arguments` object and the named parameters of a function is extremely brittle, and so modifying the former will usually lead to unexpected results.

#### Item 24: Use a variable to save a reference to `arguments`

Bind an explicitly scoped reference to `arguments` in order to refer to it from nested functions (like the `that = this` idiom).

#### Item 25: Use `bind` to extract methods with a fixed receiver

Note that extracting a method does not bind the method’s receiver to its object.

```js
function times(num, callback) {
  for (var i = 0; i < num; i++) {
    callback(); // call is made "function-style"
  }
}

var cat = {
  age: 5,
  ageOneYear: function () {
    this.age += 1;
  }
};

cat.ageOneYear(); // works
times(10, cat.ageOneYear); // does not work!
```

When `times` is called on the last line, `ageOneYear` is extracted from `cat` and then gets called function style, or method style on the global Object. Since the global Object does not have an `ageOneYear` method/property, we get an error. To make sure `ageOneYear` is called method style, we simply make the callback an anonymous function that does that explicitly.

```js
times(10, function() {
	cat.ageOneYear();
});
```

Now our anonymous function is being called function style, but that doesn't matter because it itself is calling `ageOneYear` method style. The shorthand for this is to set the caller via `bind`.

```js
times(10, cat.ageOneYear.bind(cat));
```

#### Item 26: Use `bind` to curry functions

Suppose we have the following function

```js
function simpleURL(protocol, domain, path) {
	return protocol + "://" + domain + "/" + path;
}
```
Now suppose we have an array `paths` that correspond to the HTTP protocol and are under the domain "example.com." To generate an array of appropriate URLs, we could use the `map` function like so:

```js
var urls = paths.map(function(path) {
	return simpleURL("http", "example.com", path);
});
```

However, we already know we can `bind` a function to a specific object and previously defined arguments. So we could refactor the above like so:

```js
var urls = paths.map(simpleURL.bind(null, "http", "example.com"))
```

The call to `simpleURL.bind` produces a new function that delegates to `simpleURL`. Since we don't need to bind to any objects, we can set the receiver as `null`. The arguments passed to `simpleURL` are constructed by concatenating the remaining arguments of `simpleURL.bind` to any arguments provided to the new function. In other words, when the result of `simpleURL.bind` is called with a single argument `path` (as it is in `map`), the function delegates to `simpleURL("http", siteDomain, path)`. This technique of binding a function to a subset of its arguments is known as *currying.* It can be a succinct way to implement function delegation with less boilerplate than explicit wrapper functions.

#### Item 27: Prefer closures to strings for encapsulating code

Never include local references in strings when sending them to APIs that execute them with `eval`. Prefer APIs that accept functions to call rather than strings to `eval`.

#### Item 28: Avoid relying on the `toString` method of functions

JavaScript functions have a `toString` parser that is not regulated by the ECMAScript standard. Very sophisticated uses of function source extraction should employ carefully crafted JavaScript parsers and processing libraries. But when in doubt, it’s safest to treat a JavaScript function as an abstraction that should not be broken.

#### Item 29: Avoid Nonstandard Call Stack Inspection Properties

The best policy is to avoid stack inspection (via methods such as `arguments.caller` and `arguments.callee`) altogether. If your reason for inspecting the stack is solely for debugging, it’s much more reliable to use an interactive debugger.

### Chapter 4: Objects and Prototypes

In many languages, every object is an instance of an associated class, which provides code shared between all its instances. JavaScript, by contrast, has no built-in notion of classes. Instead, objects inherit from other objects. Every object is associated with some other object, known as its *prototype.* Working with prototypes can be different from classes, although many concepts from traditional object-oriented languages still carry over.

#### Item 30: Understand the difference between `prototype`, `getPrototypeOf`, and `__proto__`

In JavaScript, what might be referred to as a "class" is more accurately the combination of a constructor function (e.g. `User`) and a prototype object used to share methods among instances of the constructor function (e.g. `User.prototype`). Consider the following:

```js
function User(name, passwordHash) {
	this.name = name;
	this.passwordHash = passwordHash;
}

User.prototype.toString = function() {
	return this.name;
}

User.prototype.checkPassword = function(password) {
	return hash(password) === this.passwordHash;
}

var u = User.new("Ed", hash("correcthorsebatterystaple"))
```

The `User` function comes with a default `prototype` property, containing an object that starts out more or less empty. In this example, we add two methods to the `User.prototype` object: `toString` and `checkPassword`. When we create an instance of `User` with the `new` operator, the resultant object `u` gets the object stored at `User.prototype` automatically assigned as *its* prototype object (and can therefore access the aforementioned methods). And so we have the following definitions:

* `C.prototype` is used to establish the prototype of objects created by `new C()`.
* `Object.getPrototypeOf(myObj)` is the standard ES5 mechanism for retrieving the prototype object of `myObj`
* `myObj.__proto__` is the actual pointer to the prototype of `myObj`, and can be used as a nonstandard mechanism for retrieving the prototype.

#### Item 31: Prefer `Object.getPrototypeOf` to `__proto__`

Wherever `Object.getPrototypeOf` is available, it is the more standard and portable approach to extracting prototypes. For JavaScript environments that do not provide the ES5 API, it is easy to implement in terms of `__proto__`:
```js
if (typeof Object.getPrototypeOf === "undefined") { // avoid overwriting
	Object.getPrototypeOf = function(obj) {
		var type = typeof obj;
		if (!obj || (type !== "object" && type !== "function")) {
			throw new TypeError("not an object");
		}
		return obj.__proto__;
	};
}
```

#### Item 32: Never modify `__proto__`

For creating new objects with a custom prototype link, you can use ES5’s `Object.create`, e.g. `childClass.prototype = Object.create(parentClass.prototype)`.

Here, `Object.create` takes in a prototype and returns a new object that inherits from it. That object gets assigned to the child class's prototype, so that a prototype chain exists from the child to the parent.

#### Item 33: Make your constructors `new`-agnostic

Protecting a constructor against misuse may not always be worth the trouble, especially when you are only using a constructor locally. Still, it’s important to understand how badly things can go wrong if a constructor is called in the wrong way. At the very least, it’s important to document when a constructor function expects to be called with new, especially when sharing it across a large codebase or from a shared library.

#### Item 34: Store methods on prototypes

Storing methods on a prototype makes them available to all instances without requiring multiple copies of the functions that implement them or extra properties on each instance object. You might expect that storing methods on instance objects could optimize the speed of method lookups, since it doesn’t have to search the prototype chain to find the implementation of the method. However, modern JavaScript engines heavily optimize prototype lookups, so copying methods onto instance objects is not necessarily guaranteed to provide noticeable speed improvements. And instance methods are almost certain to use more memory than prototype methods.

#### Item 35: Use closures to store private data

JavaScript’s object system does not particularly encourage or enforce information hiding. Nevertheless, some programs actually call for a higher degree of hiding. For example, a security-sensitive platform or application framework may wish to send an object to an untrusted application without risk of the application tampering with the internals of the object. For these situations, JavaScript does provide one very reliable mechanism for information hiding: the closure.

Closures store data in their enclosed variables without providing direct access to those variables. The only way to gain access to the internals of a closure is for the function to provide access to it explicitly. In other words, objects and closures have opposite policies: The properties of an object are automatically exposed, whereas the variables in a closure are automatically hidden. We can take advantage of this like so:

```js
function User(name, passwordHash) {
	this.toString = function() {
		return "[User " + name + "]";
	};
	this.checkPassword = function(password) {
		return hash(password) === passwordHash;
	};
}
```

This implementation of `User` has no instance `name` or `passwordHash` to access directly, and the only way to access the arguments to the constructor are through the methods that enclose them. The obvious downside is that we are not defining the methods on the prototype, so that multiple user instances will reproduce the same `toString` and `checkPassword` properties. Nevertheless, in situations where guaranteed information hiding is critical, it may be worth the additional cost.


#### Item 36: Store instance state only on instance objects

Understanding the one-to-many relationship between a prototype object and its instances is crucial to implementing objects that behave correctly. One of the ways this can go wrong is by accidentally storing per-instance data (i.e. instance variables) on a prototype. Methods are generally safe to share between multiple instances of a class because they are typically stateless, other than referring to instance state via references to `this`. In general, any immutable data is safe to share on a prototype, and stateful data can in principle be stored on a prototype, too, so long as it’s truly intended to be shared among all instances (i.e. a class variable).

#### Item 37: Recognize the implicit binding of `this`

The scope of `this` is always determined by its nearest enclosing function. Use a local variable, usually called `that`, to make a `this`-binding available to inner functions. As an example, suppose we define a class to parse data in Comma-Separated-Values (CSV) format:

```js
// for robustness, allow for any kind of delimiter
function CSVReader(separators) {
	this.separators = separators || [","];
	this.regexp =
		new RegExp(this.separators.map(function(sep) {
			return "\\" + sep[0]; // i.e. (sep = ",") => "\,"
		}).join("|"));
}
```

We can now define a `read` method that takes in a CSV string, splits it into lines, and then splits the line by the appropriate delimiter. Below are two ways to handle the issue of `this`-binding.

```js
CSVReader.prototype.read = function(str) {
	var lines = str.trim().split(/\n/);
	var that = this; // save a reference to outer this-binding
	return lines.map(function(line) {
		return line.split(that.regexp);
	});
};

CSVReader.prototype.read2 = function(str) {
	var lines = str.trim().split(/\n/);
	return lines.map(function(line) {
		return line.split(this.regexp);
	}.bind(this)); // bind to outer this-binding
};

var reader = new CSVReader();
reader.read("a,b,c\nd,e,f\n"); => // [["a","b","c"], ["d","e","f"]]
```

#### Item 38: `call` superclass constructors from subclass constructors

The superclass constructor should only be invoked from the subclass constructor, not when creating the subclass prototype.

```js
function Animal(size, intelligence) {
	this.size = size;
	this.intelligence = intelligence;
}

function Bird(size, intelligence, wingspan) {
	Animal.call(this, size, intelligence); // here we call the superclass
	this.wingspan = wingspan;
}

Bird.prototype = Object.create(Animal.prototype);
```

#### Item 39: Never reuse superclass property names

Any instance variable/property is stored on instance objects and named with a string. Because property lookup is done via traversing the prototype chain, two classes in an inheritance hierarchy sharing a property *name* will in fact share the same property. As a result, subclasses must always be aware of name-clashing with all properties used by their superclasses, even if those properties are conceptually private. Note that we can still have a subclass override a *method* of its superclass by simply redefining the method on the subclass's prototype.

#### Item 40:
