Functional Programming in JavaScript: Decorators
=
Consider a function to calculate the square of any number

    const square = (x) => x*x

But what if someone passes {} or a string to this function? We will get NaN as result, in that case. We don't want that to happen. The program should throw error when a non-numeric argument is passed to the function.

That's where function decorators come in. **Function decorators** take functions as arguments and augment them with addition functionality or checks. So let's write a decorator fuction to check for numeric types.

    const operateOnNumericInput = (fn) => (x) => typeof x === 'number' ? fn(x) : new Error('Input is not a number')

This function will throw an error, if a non-numeric argument is passed to 'fn' and returns fn(x) otherwise. Let's write a better implementation of our square() using the operateOnNumericInput() decorator.

    const goodSquare = operateOnNumericInput(square)

Sweet! Now if we pass {} or a string to goodSquare, we are bound to have an error thrown in our face. That's what we wanted.

    goodSquare(8)
        // 64
        
    goodSquare({})
        // Error: Input is not a number

Now that we've understood about function decorators. Let's take a step further.

Consider the following object literal, which has a public method 'calculate' to calculate simple
interest (don't worry if you don't know the math, it's irrelevant, anyway).

    const interest = {
        rate: 8,
        calculate: function(loan) {
            return (this.rate * loan) / 100;
        }
    }

From __JavaScript: The Good Parts:__
> When a function is stored as a property of an object, we call it a method.

So, what if we want to ensure that the 'calculate' method only handle numeric data and throw an error otherwise. 

Can we use our function decorator to do that? Let's try that out.

    interest.calculate = operateOnNumericInput(interest.calculate)
    
__or__ 
you can do the same thing using 'Object.defineProperty()'

    Object.defineProperty(interest, 'calculate', {
        value: operateOnNumericInput(interest.calculate)
    });
    
Now that we have made our 'calculate()' method to operate only on numeric input, let's calculate the interest on $3000:

    interest.calculate(3000)
        // Output: NaN
        
WHAT!?!?!?
We got _NaN_ as output, so what went wrong, exactly?

The thing is that, our function decorator 'operateOnNumericInput()' does not preserve the context, hence 'this.rate' is resolving to a global scope. Why? because if you've noticed that 'operateOnNumericInput' is invoked in a 'functional invocation' way, which always resolves to the global scope. 

If you are clueless about what I'm talking about, I would request you to go straight to Chapter 5 of 'JavaScript: The Good Parts'.

So, how can we make our decorator retain the context? Let's redefine our decorator like this:

    const operateOnNumericInput = (fn) => {
    	return function (x) {
    		return typeof x === 'number' ? fn.call(this, x) : new Error('Input is not a number');
    	}
    }
    
We did two things here

1. We converted the inner arrow function to a normal function, so that 'this' can bind correctly to the caller. *Hint: If we call a function like a.b(), 'this' is object 'a', inside b()'s body*.
2. We used .call() to pass the context i.e 'this' to our function which is getting decorated'.

Now, let's test our new implementation:

    interest.calculate(3000);
        // Output: 240

Voila! Our function decorator is finally behaving, as intended. Now, that our function decorator preserves the context and can be used on methods, we can say that we have developed a __Method decorator__.

_If you liked my essay, please go and read, [JavaScript Allonge](1). The book is jam packed with functional programming like this._

**Thank you for reading. Till next time.**

[1]:https://leanpub.com/javascriptallongesix
