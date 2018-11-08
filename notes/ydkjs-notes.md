# You Don't Know JS Notes

Notes for learning Javascript from book You Don't Know JS.

#Types & Grammar

## Chapter 1 : Types

Understanding types is essential to properly and accurately convert values to different types 

> Type Coercion (conversion) confusion is perhaps one of the most profound frustrations for JavaScript developers. It has often been criticized as being so *dangerous* as to be considered a flaw in the design of the language, to be shunned and avoided.

#### `typeof` operator

* original bug in JS that has existed for nearly two decades

```javascript
typeof null === "object"; // true
```

* compound `null` check

```javascript
var a = null;

(!a && typeof a === "object"); // true
```

* useful when validating properties in Global variables

```javascript
// oops, this would throw an error!
if (DEBUG) {
	console.log( "Debugging is starting" );
}

// this is a safe existence check
if (typeof DEBUG !== "undefined") {
	console.log( "Debugging is starting" );
}
```

* Use as **feature check** in API when sharing utility functions

```javascript
if (typeof atob === "undefined") {
	atob = function() { /*..*/ };
}
```

* but this does not throws `ReferenceError` since we are accessing property value not a reference variable

```javascript
// Only works in browser environment
// won't work in environments like NodeJS
if (window.DEBUG) {
	// ..
}

if (!window.atob) {
	// ..
}
```



## Chapter 2 : Values

> `array`s, `string`s, and `number`s are the most basic building-blocks of any program, but JavaScript has some unique characteristics with these types that may either delight or confound you.

#### Arrays

* arrays are containers for values that it can hold *any type* of value
* empty slots are auto-assigned as `undefined`

```javascript
var a = [ ];

a[0] = 1;
// no `a[1]` slot set here
a[2] = [ 3 ];

a[1];		// undefined

a.length;	// 3
```

* also accepts String keys as they are originally objects

```javascript
var a = [ ];

a[0] = 1;
a["foobar"] = 2;

a.length;		// 1
a["foobar"];	// 2
a.foobar;		// 2
```

* But number convert-able key strings are considered as numeric index

> Generally, it's not a great idea to add `string` keys/properties to `array`s. Use `object`s for holding values in keys/properties, and save `array`s for strictly numerically indexed values.

```javascript
var a = [ ];

a["13"] = 42;

a.length; // 14
```

* Array Conversion
  * some values (e.g `arguments` object) are **Array-like** so that they can be looped but cannot call Array methods like `indexOf`, `forEach`, etc.
  * conversion such as below code is nessary, Alternatively can use `Array.from(..)` in ES6.

```javascript
function foo() {
	var arr = Array.prototype.slice.call( arguments );
	arr.push( "bam" );
	console.log( arr );
}

foo( "bar", "baz" ); // ["bar","baz","bam"]
```



#### Strings

* immutable values. increases performance, safe concurrent access
* **array-like** (char array) but limited array methods (no `map`, `filter` )

```javascript
var a = "foo";
var b = ["f","o","o"];

a.length;							// 3
b.length;							// 3

a.indexOf( "o" );					// 1
b.indexOf( "o" );					// 1

var c = a.concat( "bar" );			// "foobar"
var d = b.concat( ["b","a","r"] );	// ["f","o","o","b","a","r"]

a === c;							// false
b === d;							// false

a;									// "foo"
b;									// ["f","o","o"]
```

* can borrow non-mutate array methods as below

```javascript
a.join;			// undefined
a.map;			// undefined

var c = Array.prototype.join.call( a, "-" );
var d = Array.prototype.map.call( a, function(v){
	return v.toUpperCase() + ".";
} ).join( "" );

c;				// "f-o-o"
d;
```

* but won't work in mutating array methods

```javascript
a = "foo";
a.reverse;		// undefined

b.reverse();	// ["!","o","O","f"]
b;				// ["!","o","O","f"]
```

* Quick'n dirty workaround exists but *not unicode-aware* (use [esrever](https://github.com/mathiasbynens/esrever) for unicode-aware string reverse)

```javascript
var c = a
	// split `a` into an array of characters
	.split( "" )
	// reverse the array of characters
	.reverse()
	// join the array of characters back to a string
	.join( "" );

c; // "oof"
```



#### Numbers

* only one numeric type `number` for both *integers* and *decimal* numbers.
* internally, `number`s are *floating-point* a.k.a *64 bit binary*
* can call number methods in `Number.prototype` (mind  `()` wraps around `number`)

```javascript
// invalid syntax:
42.toFixed( 3 );	// SyntaxError

// these are all valid:
(42).toFixed( 3 );	// "42.000"
0.42.toFixed( 3 );	// "0.420"
42..toFixed( 3 );	// "42.000"
```

