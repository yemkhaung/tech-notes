# Notes from reading Javascript : The Good Parts

Notes taken from reading the book **Javascript : The Good Parts** by *Douglas Crackford*.

## Chapter 1 : Analyzing Javascript

It's the *ONLY* language of The Web. You have no choice but to avoid bad parts and embrae the good parts or it is *really good* inspite of its bad parts.

**Good parts**

* functions as first-class objects with (mostly) lexical scoping
* loose-typing
* powerful object literal notation
* protypal inheritance

**Bad parts**

* depends on Global objects for linkage
* DOM integration (*DOM itself is designed bad*)

**Take Away**

> Javascript is language of contrasts with many errors and sharp edges. If you can embrace the good parts, Javascript's functional programming becomes productive and fun.



## Chapter 2 : Grammar

* **Literals**
  * convenient syntax for declaring new objects.
  * object properties can be either *literal names* or *strings*
* **Functions**
  * can have an *optional* name. 
  * can call iteself inside body *recursively*.
* **Comments** : Block comments are bad pattern since regex can include `/` and `*` and can conflict with block comments `/* */`.
* **Names** : Nothing special. Use alphanumeric `A-Za-z0-9` and underscore `_` for easy syntax and avoid keywords.
* **Numbers** : The Good parts 
  * integers are `64 bit floating point` *internally* i.e 1 and 1.0 are the same value. This solves overflow problems with shrot integers.
  * `NaN` is programatically abnormal number. Not equal to any value including itself. detected by *isNaN* function.
  * `Infinity`: all values greater than `1.79769313486231570e+308`.
  * `Math` object which contains handy functions for number operations
* **Strings** : Immutable values. Concatenate with `+` operator.
* **Statements** : Executable compilation units. Javascript has not [linkers](https://www.computerhope.com/jargon/l/linker.htm) so all compilation units (e.g every `<script>`) are thrown into a common *Global namespace*.
* **Falsy values** (as in making conditional decisions)
  * false, null, undefined, empty string `''`, number `0`, number `NaN`
* **typeof** operator produces `number`, `string`, `boolean`, `function`, `object` and `undefined`. 
  * Bad : if operand is `array` or `null`, **typeof** result is `object` which is wrong.
* **&&** and **||** :
  * `&&` operator produces first operand if it is *falsy* otherwise produces second operand value. (use case : *guard operator*)
  * `||` operator produces first operand value if it is *truthy* otherwise produces second operand value. (use case *Default value*)
* **Invocation operator** : executes the function with literal name before the operator. In Javascript, `function` will not execute the function but `function()` will.



## Chapter 3 : Objects

* **Everything is objects** except *numbers, strings, booleans, null and undefined*
* are **mutable** keyed collections
* objects are **passed by reference**, never by copies
* **Objects literals** are used for easy object creation by using just a pair of `{}`. Properties are key-value pairs. Key is any string. Value is any exept undefined.
* Every object is linked to a **Prototype** object to inherit properties. Standard root link is `Object.prototype`.
* Use **Object.create** instead of `new` to properly set prototype to new object. in this way, prototype link has no effect on updating and only used for retrieval. If a property is not found in object, it is searched in *prototype chain*. This behavior is called *delegation*.
* **Global Abatement** : Should avoid or minimize to a single name. Use of global variables makes your application to allow bad interaction with other libraries and widgets.
* **Reflection** : checking whether properties of object exist or not can be done by two ways
  *  `typeof` : operator that returns type of a property. But any property on prototype chain can produce a type such as  *function* and so can lead to undesired values.
  * `hasOwnProperty` : method returns true if property exists. Does not look at prototype chain. 
* **Enumeration** : 
  * `for in` loop over all properties in prototype chain and enumeration include everything even functions. Use `typeof` operator to filter out *functions*.
  * Use `for` loop since it does not look at prototype chain and keeps the correct order of properties.
* **Delete** : operator to remove properties from an object and does not look at prototype chain.



## Chapter 4 : Functions

* 