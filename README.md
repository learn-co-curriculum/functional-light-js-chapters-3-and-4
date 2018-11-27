Chapter 3

Previously on Functional-Lite JS: Functions

This week: Managing Inputs (Arguments/Parameters)

“Arguments are the values you pass in, and parameters are the named variables inside the function that receive those passed-in values.”

##Many arguments into one

`unary()`

Utility that takes a function, utility will pass multiple arguments, however function takes only single arguments

Create helper function that enforces the unary-ness (taking only one argument)

##One argument, return argument

`identify()`

Returns argument passed in.

Useful as function argument for things like `filter()`. JS will coerce value into boolean.

`identity()` also used as default function argument to transform functions.

##No argument, return value

`constant()`

Return a constant value as a function.

Useful in `then()` in JS Promises. Requires function even if only returning value.

Many use arrow function, but consider not doing that.

##From Arguments to Params

`spreadArgs()`

Arrays are handy to pass in as arguments, especially using deconstruction `...args`.

What do you do when you can't change functions' arguments? Utilities, libraries, etc.

`spreadArgs()` allows to return multiple values in array, but use those values independantly to pass to other functions

`gatherArgs()`: Opposite of `spreadArgs()`

##Arguments not passed in at same time

`partial()`

Situation: API calls where urls are known, but data and callback for response come later.

Option: Write wrappers that call `ajax` with url set.

Can be good but gets out of hand with setting arguments as well:

Get in the habit of looking for patterns where same things are done repeatedly. Good opportunity for reusable utilties.

##Argument Application

Apply or appliation
When function is called, arguments are 'applied' to parameters.

Partial Application
`getOrder(data, cb)` is a 'partial application' of `ajax(url, data, cb)` as it applies a single argument into a parameter, but other arguments are left for later.

Partial application is a reduction of a function's arity.

Arity
“Arity is the number of parameters in a function declaration”

`partial()` takes in initial arguments, then returns a function that will take more arguments, then calls the intial function with both sets of arguments.

Why does this work? Closures!

`partiallyApplied()` 'closes' over `fn` and `presetArgs`.

##`bind()`: The JS built-in `partial()`. No, it's bad

`bind()` does two things
- presets `this` context
- partially applys arguments

Kyle's take: Misguided to have combine these two responsbilities. Sometimes you'll want one and not the other. Never needed both at same time.

###Reversing arguments

`reverseArgs()`

`ajax` signature is `ajax( url, data, cb)`.

Want to apply the callback `cb`, but have other args later? Using a reverse function, we can partially apply from the right, as opposed to the left.

##One at a time

Currying: a technique similiar to partial application, where multiple arguments are broken down into chained functions that are arity 1

`curriedAjax()` receives one argument at a time, each in a seperate call.

Currying partially applies another argument to the original function, until all arguments have been passed. Each step returns a function that only accepts the next argument.

Nesting dolls of functions.

So currying unwinds a single higher-arity function into a series of chained unary functions.

Example of currying with `add` function:

```
[1,2,3,4,5].map( curry( add )( 3 ) );
// [4,5,6,7,8]
```

Why choose `curry(add)(3)` vs `partial(add, 3)`?

When you know you want to use `add`, but don't know the value to be added.

Currying creates a more specialized function with each return that can be captured and reused, while `partial` needs to be recalled each time.

Both use closure.

Visualization of curried functions call chain:
```
function curriedSum(v1) {
    return function(v2){
        return function(v3){
            return function(v4){
                return function(v5){
                    return sum( v1, v2, v3, v4, v5 );
                };
            };
        };
    };
}

// ES6 arrow functions
curriedSum =
    v1 =>
        v2 =>
            v3 =>
                v4 =>
                    v5 =>
                        sum( v1, v2, v3, v4, v5 );

curriedSum = v1 => v2 => v3 => v4 => v5 => sum( v1, v2, v3, v4, v5 );
```

##Why would you use currying/partial application?

They both separate when and where arguments are specified, in both time and space.

Currying also allows composition of functions.

We can pass what arguments we know into a function, then later, in a different callsite, we can complete the argument list and get the final data back.

This also enforces semantic boundary: separating how we get data and what we do with data later. 

Haskell has automatic currying:
`foo 1 2 3`
vs
`foo(1)(2)(3)` in JS.

Some JS FP libraries can curry multiple arguments at once

Currying can also be undone with an `uncurry()`, though may not be the same as the original function that was curried.

##Order matters

We're juggling argument order, but this is all because of positional dependence of arguments. This creates a lot of mess in the code.

##Spreading Properties

In `spreadArgs()`, it depends of argument order to pull values.

With named arguments, there is guarantee of order when calling.

JS functions have `.toString()` that gives string representation, including argument signature.

Trade-offs: argument order vs named argument. 'first argument' vs 'argument named x'.

##Point-free style

Popular style in FP world is tacit programming (point-free). Point refers to the parameter not needing to set a return value to variable to pass it into another function.

Helper utilities like `not()` or `when()` to reduce points in a refactor.

Point-free is cool, but use your judgement. It can be a lot of jumping through hoops to improve readability and may degrade readability if taken too far.

#Chapter 4 - Composing Functions

Functional programming: seeing every function like a Lego piece. Small on its own, but useful to put together with others.

However, sometimes, one function/piece fits with another function/piece so often that they are more used together than apart. That is a new function that is easier to think about together.

##Output to Input

To compose: pass output of first function as input to second function.

arrayValue <-- map <-- unary <-- parseInt
//right-to-left is common in FP and makes sense later.

`parseInt()` is input to `unary()`. `unary()` output is input to `map()`. `map()` output is `arrayValue`. Composition of map and unary

Example workflow: wordsUsed <-- unique <-- words <-- text

##A composition-making machine

If compositions are good, is there a way to create these compositions without needing to rewrite functions to accept output?

Keep parameter order in mind, `fn2` before `fn1`. The functions compose right-to-left, so argument order is listed in that order. Very common for FP libraries to define `compose()` right-to-left.

Execution order is right-to-left, but code order is left-to-right.

```
var uniqueWords = compose2( unique, words );
```

##Composition Variation

Functions can be composed the other way as well. Sometimes, it works, but we wary.

##General Composition

With composition, you could combine any number of functions.

Most powerful tricks of Function Programming!
- A combination of partial application and function composing allows some arguments to be set while still creating specialized variations. Could also `curry` but may need to reverse args.

##Alternative Implementations

Better to use the FP libraries versions of utilties especially `compose`, however understand how it works. There may be performance implications.

Potential implementations:
`reduce`
- 'fancy loop', reduces array to a single value
- reduce([1,2,3,4,5]) // ((((1+2)+3)+4)+5) = 15
- we can reduce a list of functions to make `compose`
- each result step is passed as input to next call

Eager vs lazy implentation can have very different performance.

In initial version of compose, `[...fns].reverse().reduce(...)` calls reduce on every function, while `fns.reverse().reduce(...)` calls reduce once and defers calculation until the end.

Recursion offers another way. Recursion vs repetitive action with running result. YMMV.

##Reordered composition

Right-to-left is standard `compose` order.
- pros
    - same order they appear in if composing manually
- cons
    - functions are in reverse order of Execution
    - using partial application like `partialRight` to set first function is difficult

Left-to-right is called `pipe`.

From Unix where programs are strung together using the pipe (|) operator. Output is passed to next program.

- pros
    - functions are listed in execution order
    - partial application is clearer
- cons
    - ?

Pipe is the reverse arguments of `compose`
```
var pipe = reverseArgs( compose );
```

##Abstration

Pull out generality between two or more tasks

DRY code helps keep behavior common across tasks. Also keeps from needing to update logic in multiple places.

Abstration can be taken too far.

"Composition is helpful even if there's only once occurence of something"

Separation Enables Focus

"… abstraction is a process by which the programmer associates a name with a potentially complicated program fragment, which can then be thought of in terms of its purpose of function, rather than in terms of how that function is achieved. By hiding irrelevant details, abstraction reduces conceptual complexity, making it possible for the programmer to focus on a manageable subset of the program text at any particular time."
    -Michael L. Scott, Programming Language Pragmatics

Separate two pieces of functionality so you can focus on each piece independently of the other. Not black boxes that we never look into. Is about focus.

"We’re not abstracting to hide details; we’re separating details to improve focus."

Through focus via FP, code can be more readable and understandable.

Abstration creates a 'semantic boundary' between two pieces of functionality. Example, name of function. Separating the *how* from *what*.

Imperative vs declaritive code

Imperative: how to accomplish a tasks
Declaritive: what the outcome should be

ES6 example of syntax that pushes declarative code: destructuring.

Destructuring describes how a compound value, like object or array, is taken apart.


Discussion Topics:
- https://github.com/lodash/lodash/wiki/FP-Guide#mapping
- How are people feeling about the longer functions vs ES6 versions? One easier to read?
- Always use FP utilities? How do you remember, find those?
- Helpful to see the 'returned function' version of functions like `partial()` implementation. Useful in normal development?
- Benefits of currying? When would you create your own?
- Trade-offs: argument order vs named argument. 'first argument' vs 'argument named x'. Both require knowledge about the function.
- Kyle's style of naming could be anonymous functions...
```
function compose2(fn2,fn1) {
    return function composed(origValue){
        return fn2( fn1( origValue ) );
    };
}
```
find myself ignore what `composed` is called. Does naming everything help? Naming shows importance/usefulness so sometimes seems distracting?
- In 'Machine Making', what level of utilities should we be making? Is 2 an optimal number for function composition?
- In 'Composition Variation', it's pointed out that composing functions can sometimes be switched for a different purpose. Is this a valuable learning? Just a novel idea?
```
var uniqueWords = compose2( unique, words );
//versus
var letters = compose2( words, unique );
```
- In Alternate Composition, eager vs lazy vs recursion was discussed. Good ways to figure out performance characteristcs? Benchmark? Use returned function version?
- Talking about pipes, doesn't really list a negative. Is there an instance where pipe is not good to use?
- How much do agree with this 'overly-DRY' code:
```
function conditionallyStoreData(store,location,value,checkFn) {
    if (checkFn( value, store, location )) {
        store[location] = value;
    }
}

function notEmpty(val) { return val != ""; }

function isUndefined(val) { return val === undefined; }

function isPropUndefined(val,obj,prop) {
    return isUndefined( obj[prop] );
}

function saveComment(txt) {
    conditionallyStoreData( comments, comments.length, txt, notEmpty );
}

function trackEvent(evt) {
    conditionallyStoreData( events, evt.name, evt, isPropUndefined );
}
```
- Composition is valuable if there is only one occurence vs DRY. Agree?
