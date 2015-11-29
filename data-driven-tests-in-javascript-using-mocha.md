*Welcome to this article in which I'll teach you how to write data-driven tests using [Mocha](https://mochajs.org/).*

*If you are already familiar with the data-driven test pattern, you may wish to skip the proceeding section and jump straight to the [sections on implementation](#implementation).*

## The Case for Data-Driven Tests

Sometimes it's desirable to run essentially the same test many times with many slightly different inputs. 

Imagine for a second that you're writing a function called `isPrime` that takes a number. This function should return `true` if the number is prime; otherwise, it should return `false`. Here is what our tests for such a function might look like, using Mocha and [Chai](http://chaijs.com/):

```javascript
import { expect } from "chai";
import isPrime from "./";

describe("isPrime()", function() {
  it("should return true when number is a prime number", function() {
    const actual = isPrime(2);
    expect(actual).to.be.true;
  });

  test("should return false when number is a composite number", function() {
    const actual = isPrime(4);
    expect(actual).to.be.false;
  });
});
```

What do you think about these tests? Do you think that they're adequate?

I don't. think these tests are very good at all. Here's why:

At the moment, all these tests prove is that the function returns `true` if the input is `2`, or `false` if if the input is `4`. 

The `isPrime` function's implementation could be as erroneous as: 

```javascript
export default function isPrime(num) {
  if (num === 2) {
    return true;
  } else {
    return false;
  }
}
```

And the tests would still pass.

Clearly our test coverage is inadequate - we'll need to add more tests if we are ever going to trust our test suite to catch [regressions](https://en.wikipedia.org/wiki/Software_regression).  

Now, what we *could* do is duplicate the tests a handful of times, changing the input slightly for each test:

```javascript
import { expect } from "chai";
import isPrime from "./";

describe("isPrime", function() {
  test("given a prime number, isPrime() returns true", function() {
    const actual = isPrime(2);
    expect(actual).to.be.true;
  });

  test("given a prime number, isPrime() returns true", function() {
    const actual = isPrime(5);
    expect(actual).to.be.true;
  });

  test("given a prime number, isPrime() returns true", function() {
    const actual = isPrime(727);
    expect(actual).to.be.true;
  });
  
  test("given a prime number, isPrime() returns true", function() {
    const actual = isPrime(1223);
    expect(actual).to.be.true;
  });
});
```

Whilst this approach provides much better code coverage of functionality, it also incurs a high cost to test maintainability, due to the repetitious nature of the tests. Repetitious tests are untenable because any changes made to one of the tests must be propagated to all of the similar tests. In other words, this approach violates the  [DRY principle](), which is no good.

This is a common problem, for which a solution already exists. Enter the *data-driven test pattern*.

<h2 id="implementation">Implementing a Data-Driven Test Using Mocha</h2>

Whilst certain test frameworks for other technology stacks inherently support data-driven tests, unfortunately, Mocha does not. That being said, it is entirely possible to write data-driven tests using Mocha, like so:

```javascript
var primeNumbers = [2, 3, 5, 53, 443, 977];
primeNumbers.forEach(function(primeNumber) {
  it("given prime number, isPrime() returns true", function() {
    const actual = isPrime(primeNumber);
    expect(actual).to.be.true;
  });
});
```

The above tests pass and produce the following output:

![](https://i.imgur.com/tHDKHLx.png)

This is a sound solution, albeit a little naïve.

The problem with the implementation above is that, should a test fail, as is the case here:

![](https://i.imgur.com/3OCsga0.png)

You won't be able to tell which input caused the test to fail at a glance. 

As I am sure you'll all agree, a quality test should make it immediately clear why the test has failed. How to improve the assertion error message is the subject of the next section.

## A Better Data-Driven Test

In order to improve the test output, we can leverage our assertion libraries to  report a custom assertion failure message.

Here I am using [Chai's BDD DSL](http://chaijs.com/api/bdd/), but any assertion library worth it's salt enables custom assertion failure messages:

```javascript
var primeNumbers = [2, 3, 5, 53, 443, 977];
primeNumbers.forEach(function(primeNumber) {
  it("should return true if number is prime number", function() {
    const actual = isPrime(primeNumber);
    expect(actual, `num=${primeNumber}`)
      .to
      .be
      .true;
  });
});
```
Now, when one or more tests fail, we get a useful error message:

![](https://i.imgur.com/JkuHnr9.png)

This is a solid implementation that I use for the majoroty of my data-driven tests. There is, however, one more slight variation of this implementation that I would like to share with you in this article. 

## Custom Test Names

One limitation of the aforementioned implementation is that the test names are all the same and therefore, very general. 


You could, if you wanted to, associate descriptions with each of your tests, like so:

```javascript
import { expect } from "chai";
import usernameValidator from "./";

describe("usernameValidator", function() {

  const invalidUsernames = {
    "empty username": "",
    "username shorter than 3 chars": "us",
    "username containing symbols": "username$"
    "username containing spaces": "user name"
  };

  for (let prop in invalidUsernames) {
    it(`given ${prop}, validateUsername() should return false`, function () {
      const username = invalidUsernames[prop];
      const actual = validateUsername(username);
      expect(actual).to.be.false;
    });
  }
});
```

Which would yield the following output:

![](https://i.imgur.com/QebBEBI.png)

Understand that this implementation is not always applicable.  Because this implementation requires extra code, it incurs a cost to maintaibility. It is for this reason that you need to be judicious when applying it - you need to make sure that the cost is justifable. I would not use this implementation for the `isPrime` function tests, for example.
### Conclusion

Good code coverage of functionality is important, but so is test maintainability. The data-driven test pattern affords you a way to attain sufficient code coverage without negatively impacting the maintainability of your tests.

Whilst test frameworks for other stacks inherently support this pattern through [theory attributes](http://xunit.github.io/docs/getting-started-desktop.html#write-first-theory), most JavaScript test frameworks do not. Fortunately, it isn't too much trouble to implement this pattern using simple JavaScript constructs.

**P.S. If you read this far, you might want to follow me on [Twitter](https://twitter.com/bookercodes) and [GitHub](https://github.com/alexbooker), or [subscribe](https://booker.codes/rss/) to my blog.**

