## The Problem

You have the classic JavaScript problem known as the *incorrect `this` context*.
The [`this` keyword in JavaScript][1] behaves differently than it does in other languages like C# or Java.

<!-- Call out the exact line here -->

### How `this` works

The `this` keyword, in a function, is determined as follows:
 * If the function was created through a call to `.bind`, the `this` value is the argument provided to `bind`
 * If the function was *invoked* through a method call, e.g. `expr.func(args)`, then `this` is `expr`
 * Otherwise
   * If the code is in [*strict mode*][2], `this` is `undefined`
   * Otherwise, `this` is `window` (in a browser)

Let's look at how this works in practice:

    class Foo {
        value = 10;
        doSomething() {
            // Prints 'undefined', not '10'
            console.log(this.value);
        }
    }
    let f = new Foo();
    window.setTimeout(f.doSomething, 100);

This code will print `undefined` (or, in strict mode, throw an exception).
This is because we ended up in the last branch of the decision tree above.
The `doSomething` function was invoked, the function wasn't a result of a `bind` call, and it wasn't invoked in a method syntax position.

We can't see the code for `setTimeout` to see what its invocation looks like, but we don't need to.
Something to realize is that all `doSomething` methods point to *the same function object*.
In other words:

    let f1 = new Foo();
    let f2 = new Foo();
    // 'true'
    console.log(f1.doSomething === f2.doSomething);

We know that `setTimeout` can only see the function we passed it, so when it invokes that function,
  there's no way for it to know which `this` to provide.
The `this` context has been lost due to our *referencing* the method without *invoking* it.

### The Red Flag

Once you know about `this` problems, they're easy to spot:

    class Foo {
        value = 10;
        method1() {
            doSomething(this.method2); // DANGER, method reference without invocation
        }	
        method2() {
            console.log(this.value);
        }
    }

## The Solution

You have a few options here, each with its own trade-offs.
The best option depends on how often the method in question is invoked from differing call sites.

### Arrow Function in Class Definition

Instead of using the normal method syntax, use an [arrow function][3] to initialize a per-instance member.

    class DemonstrateScopingProblems {
        private status = "blah";

        public run = () => {
            // OK
            console.log(this.status);
        }
    }
    let d = new DemonstrateScopingProblems();
    window.setTimeout(d.run); // OK

 * Good/bad: This creates an additional closure per method per instance of your class. If this method is usually only used in regular method calls, this is overkill. However, if it's used a lot in callback positions, it's more efficient for the class instance to capture the `this` context instead of each call site creating a new closure upon invoke.
 * Good: Impossible for external callers to forget to handle `this` context
 * Good: Typesafe in TypeScript
 * Good: No extra work if the function has parameters
 * Bad: Derived classes can't call base class methods written this way using `super.`
 * Bad: The exact semantics of which methods are "pre-bound" and which aren't create an additional non-typesafe contract between your class and its consumers.

### Function Expression at Reference Site

Shown here with some dummy parameters for explanatory reasons:

    class DemonstrateScopingProblems {
        private status = "blah";

        public something() {
            console.log(this.status);
        }

        public run(x: any, y: any) {
            // OK
            console.log(this.status + ': ' + x + ',' + y);
        }
    }
    let d = new DemonstrateScopingProblems();
    // With parameters
    someCallback((n, m) => d.run(n, m));
    // Without parameters
    window.setTimeout(() => d.something(), 100);

 * Good/bad: Opposite memory/performance trade-off compared to the first method
 * Good: In TypeScript, this has 100% type safety
 * Good: Works in ECMAScript 3
 * Good: You only have to type the instance name once
 * Bad: You'll have to type the parameters twice
 * Bad: Doesn't easily work with variadic parameters

  [1]: https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Operators/this
  [2]: https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Strict_mode
  [3]: https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Functions/Arrow_functions
