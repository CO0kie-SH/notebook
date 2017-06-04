From https://stackoverflow.com/posts/20279485/edit.

## What you should know about `this`

`this` (aka "the context") is a special keyword inside each function and its value only depends on *how* the function was called, not how/when/where it was defined. It is not affected by lexical scope, like other variables. Here are some examples:

    function foo() {
        console.log(this);
    }

    // normal function call
    foo(); // `this` will refer to `window`

    // as object method
    var obj = {bar: foo};
    obj.bar(); // `this` will refer to `obj`

    // as constructor function
    new foo(); // `this` will refer to an object that inherits from `foo.prototype`

To learn more about `this`, have a look at the [MDN documentation][this].


----------


## How to refer to the correct `this`


### Don't use `this`

You actually don't want to access `this` in particular, but *the object it refers to*. That's why an easy solution is to simply create a new variable that also refers to that object. The variable can have any name, but common ones are `self` and `that`.


    function MyConstructor(data, transport) {
        this.data = data;
        var self = this;
        transport.on('data', function() {
            alert(self.data);
        });
    }

Since `self` is a normal variable, it obeys lexical scope rules and is accessible inside the callback. This also has the advantage that you can access the `this` value of the callback itself.


### Explicitly set `this` of the callback - part 1

It might look like you have no control over the value of `this`, because its value is set automatically, but that is actually not the case. 

Every function has the method [`.bind` *<sup>[docs]</sup>*][bind], which returns a new function with `this` bound to a value. The function has exactly the same behavior as the one you called `.bind` on, only that `this` was set by you. No matter how or when that function is called, `this` will always refer to the passed value.


    function MyConstructor(data, transport) {
        this.data = data;
        var boundFunction = (function() { // parenthesis are not necessary
            alert(this.data);             // but might improve readability
        }).bind(this); // <- here we are calling `.bind()` 
        transport.on('data', boundFunction);
    }

In this case, we are binding the callback's `this` to the value of `MyConstructor`'s `this`.

**Note:** When binding context for jQuery, use [`jQuery.proxy` *<sup>[docs]</sup>*][proxy] instead. The reason to do this is so that you don't need to store the reference to the function when unbinding an event callback. jQuery handles that internally. 

### ECMAScript 6: Use [arrow functions][arrow]

ECMASCript 6 introduces *arrow functions*, which can be thought of as lambda functions. They don't have their own `this` binding. Instead, `this` is looked up in scope just like a normal variable. That means you don't have to call `.bind`. That's not the only special behavior they have, please refer to the MDN documentation for more information. 

    function MyConstructor(data, transport) {
        this.data = data;
        transport.on('data', () => alert(this.data));
    }


### Set `this` of the callback - part 2

Some functions/methods which accept callbacks also accept a value to which the callback's `this` should refer to. This is basically the same as binding it yourself, but the function/method does it for you. [`Array#map` *<sup>[docs]</sup>*][map] is such a method. Its signature is:

    array.map(callback[, thisArg])

The first argument is the callback and the second argument is the value `this` should refer to. Here is a contrived example:

    var arr = [1, 2, 3];
    var obj = {multiplier: 42};
    
    var new_arr = arr.map(function(v) {
        return v * this.multiplier;
    }, obj); // <- here we are passing `obj` as second argument

**Note:** Whether or not you can pass a value for `this` is usually mentioned in the documentation of that function/method. For example, [jQuery's `$.ajax` method *<sup>[docs]</sup>*][ajax] describes an option called `context`:

> This object will be made the context of all Ajax-related callbacks.


----------

## Common problem: Using object methods as callbacks / event handlers

Another common manifestation of this problem is when an object method is used as callback / event handler. Functions are first class citizens in JavaScript and the term "method" is just a colloquial term for a function that is a value of an object property. But that function doesn't have a specific link to its "containing" object.

Consider the following example:

    function Foo() {
        this.data = 42,
        document.body.onclick = this.method;
    }

    Foo.prototype.method = function() {
        console.log(this.data);
    };

The function `this.method` is assigned as click event handler, but if the body is clicked, the value logged will be `undefined`, because inside the event handler, `this` refers to the body, not the instance of `Foo`.   
As already mentioned at the beginning, what `this` refers to depends on how the function is **called**, not how it is **defined**.  
If the code was like the following, it might be more obvious that the function doesn't have an implicit reference to the object:

    function method() {
        console.log(this.data);
    }


    function Foo() {
        this.data = 42,
        document.body.onclick = this.method;
    }

    Foo.prototype.method = method;

**The solution** is the same as mentioned above: If available, use `.bind` to explicitly bind `this` to a specific value

    document.body.onclick = this.method.bind(this);

or explicitly call the function as a "method" of the object, by using an anonymous function has callback / event handler and assign the object (`this`) to another variable:

    var self = this;
    document.body.onclick = function() {
        self.method();
    };

or use an arrow function:

    document.body.onclick = () => this.method();

  [proxy]: http://api.jquery.com/jQuery.proxy/
  [this]: https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Operators/this
  [arrow]: https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Functions/Arrow_functions
  [bind]: https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Function/bind
  [map]: https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Array/map
  [ajax]: http://api.jquery.com/jQuery.ajax/
