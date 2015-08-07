Welcome to this short tutorial in which you'll learn how to validate form input in an Express app using an open-source Node module called [express-validator](https://github.com/ctavan/express-validator).

### Installation

Installing express-validator is easy when you use [npm](https://docs.npmjs.com/getting-started/what-is-npm). Just input the following command:

```
npm install express-validator
```

This will download the latest version of the [express-validator npm package](https://www.npmjs.com/package/express-validator).

Once the command has done executing - if you were to to look inside the _"node modules"_ folder, you'd see that express-validator has a _transitive dependency_ on another module called [validator](https://github.com/chriso/validator.js):

![](https://i.imgur.com/1tabli4.png)

You'll see why this is important later.

### Setup

For the purposes of this tutorial, I'll assume you already have a script whose contents look something like this:

    // dependencies
    var express = require('express');
    var bodyParser = require('body-parser');

    var app = express();

    // middleware
    app.use(bodyParser.urlencoded({ extended: false }));

(If you don't, you can quickly create a application skeleton using the [application generator tool](http://expressjs.com/starter/generator.html).)

express-validator is middleware which you can mount by calling [`app.use`](http://expressjs.com/4x/api.html#app.use):

<pre>
<code>// dependencies
var express = require('express');
var bodyParser = require('body-parser');
<strong>var validator = require('express-validator');</strong>

var app = express();

// middleware
app.use(bodyParser.urlencoded({ extended: false }));
<strong>app.use(validator());</strong></code>
</pre>

There is only one thing you need to watch out for and that is, you must `use` express-validator *after* you `use` the [body parser middleware](https://github.com/expressjs/body-parser)*. If you do not, express-validator will not work. This is because without the body parser middleware, it would be impossible for express-validator to parse the values to validate!

### Validation

In order to illustrate validation in action, we'll imagine that we have a request handler:

    app.post('/pages/create', function(req, res) {
    });

And that the request body looks like this:

    {
      leader_email: "leader@something.edu",
      leader_mobile_no: "017171",
      team_twitter: "http://twitter.com/teamA",
      member_count: 10,
      page_color: "#FFFFFF",
      page_accent_colour: "#FFFFFF"
    }

Here we want to validate that...

- **leader_email** is a valid email address*
- **leader\_mobile_no** is a valid UK mobile phone number
- **team_twitter** is a valid Twitter handle
- **member_count** is both a number and divisble by 2
- **page\_color** and **page\_accent_color** are valid hex color codes

<p style="font-size:15px">*Remember. Validation is not the same as verification. Validation merely confirms that the email is in the correct format. Verification confirms that that the user has access to the email. If you need to verify the email, consider sending a verification email.</p>

Validating an email is a common requirement so we may as well start there.

Inside of the request handler, make a call to the `req.checkBody` function. Pass to this function the name of the input you want to validate (in this case, _"leader\_email"_) and the error message you want to return if the input is invalid:

<pre>
<code>app.post('/pages/create', function(req, res) {
    <strong>req.checkBody("leader_email", "Enter a valid email address.");</strong>
});</code>
</pre>

Next, tell the middleware which validators to apply by _chaining_ one or more validation functions:

<pre>
<code>app.post('/pages/create', function(req, res) {
  req.checkBody("leader_email", "Enter a valid email address.")<strong>.isEmail()</strong>;
});</code>
</pre>

Hopefully now you can see why express-validator depending on [validator](https://github.com/chriso/validator.js) is not merely an implementation detail. You almost always end up calling functions from the validator library of validation functions. You can see a full list of validation functions [here](https://github.com/chriso/validator.js#validators).

You might be wondering where exactly the `checkBody` function comes from. It comes from the express-parser middleware. Express middleware can add any property it wants to the `req` argument. In addition to the `checkBody` function, express-validator adds two more functions namely, `checkQuery` and `checkParams` which can be used to validate query parameters and route parameters respectively. See the [documentation](https://github.com/ctavan/express-validator/blob/master/README.md) for more information on those.



For completeness, here is how I would validate the remaining input elements:

```
req.checkBody("leader_email", "Enter a valid email address.").isEmail();

req.checkBody(
  "leader_mobile_no",
  "Enter a valid UK phone number.").isMobilePhone("en-GB");

req.checkBody(
  "team_twitter",
   "Enter a valid Twitter URL").optional().matches("http://twitter.com/*");

req.checkBody(
  "contestant_count",
  "Contestant count must be a number and one that is divisible by 2"
).isNumber().isDivisibleBy(2);

req.checkBody(
  "page_color",
  "Page colour must be a valid hex color"
).isHexColor();

req.checkBody(
  "page_color_accent",
  "Page colour accent must be a valid hex color").isHexColor();
```

Hopefully by now the above listing is intuitive. If you are unsure about what any of the functions do, you can refer to their descriptions [here](https://github.com/chriso/validator.js#validators). Aside from that, there are just a couple of things to note about this code:

- In a couple of places we _chain_ more than one validation function. This is possible thanks to [_fluent interfaces_](https://en.wikipedia.org/wiki/Fluent_interface).
- We chain the `optional` function before chaining the `matches` function. The `optional` functions says _"this value is optional so do not attempt to validate it with `matches` **unless** the user actually entered something_". In other words, if the input is empty, `matches` won't be called.

Remember. All we've done so far is define some rules. We are yet to enforce these rules. We'll do that now.

In order to determine whether the input is valid according to the rules, we call a function called `validationErrors`. If any input is invalid, this function returns an array of errors; otherwise - if all input is valid - it returns `undefined` (which indicates _"no errors"_). If the function returns an array of errors, we'll show them to the user:

<pre>
<code>app.post('/pages/create', function(req, res) {
  req.checkBody("leader_email", "Enter a valid email address.").isEmail();

  <strong>var errors = req.validationErrors();
  if (errors) {
    res.send(errors);
    return;
  } else {
    // normal processing here
  }</strong>
});
</code>
</pre>

![](https://i.imgur.com/tJO47Fj.png)

(Note that I have removed most of the rules I wrote before for brevity)

Whilst emitting errors in JSON format might be appropriate for a web service, for most websites, this is not very nice. In such a case we can send `errors` back with a template:

<pre>
<code>app.post('/pages/create', function(req, res) {
  req.checkBody("leader_email", "Enter a valid email address.").isEmail();

  var errors = req.validationErrors();
  if (errors) {
    <strong>res.render('create', { errors: errors });</strong>
    return;
  } else {
    // normal processing here
  }
});
</code>
</pre>

And then in the template, show the errors conditionally:

    if errors  
      ul
        for error in errors
          li!= error.msg

![](https://i.imgur.com/N022l73.png)

This is an example using the [Jade template engine](https://github.com/jadejs/jade). I know not everyone digs Jade so for  good measure, here is another example, this time using the [Handlebars template engine](http://handlebarsjs.com/):

    {{ #if errors }}
      <ul>
        {{ #each errors }}
          <li>{{ this.msg }}</li>
        {{ /each }}
      </ul>
    {{ /if }}


### Extending express-validator
Sometimes your validation rules will be too complex to convey using the built-in validation functions alone. Fortunately, express-validator allows you to define custom validation functions.

To illustrate custom validation functions, imagine that the user is asked to enter exactly two, _unique_, comma-delimited values.

Valid inputs would include:

- wibble, wobble
- wubble, flob

Invalid inputs would include:

- wibble, wibble
- wibble
- wibble, wobble, wubble

Although it *might* be possible to create a regular expression for this rule, I speculate that doing so would come at too much of a cost to readability. So, let us define a custom validation function.

Go back to the script in which you mounted the express-validator middleware and replace...

    app.use(validator());

With...

    app.use(validator({
      customValidators: {
        containsTwoTags: function (value) {
          var tags = value.split(',');
          // Remove empty tags
          tags = tags
            .filter(function(tag) { return /\S/.test(tag) });
          // Remove duplicate tags
          tags = tags
            .filter(function(item, pos, self) {
              return self.indexOf(item) == pos;
            });
          return tags.length <= 2;
        }
      }
    })

I shan't belabor the details of this custom function because those details are not likely to be of relevance to you. What you _do_ need to know is that the function returns `true` if the input is valid; otherwise, it returns `false`. When you input your own validation function, you'll need to ensure that it does the same.

You can use this custom validation function just like you would any other validation function, like this:

    req.checkBody('tags', 'Enter exactly two distinct tags.').containsTwoTags();


### Conclusion
This has been an introduction to input validation in Express with express-validator. Whilst I hope I have covered enough for you to hit the ground running in your application, if you encounter any problems, please consult the [documentation](httpss://github.com/ctavan/express-validator/blob/master/README.md) or [get in touch](https://twitter.com/bookercodes) and I'll be happy to help :).

**P.S. If you read this far, you might want to follow me on [Twitter](https://twitter.com/bookercodes) and [GitHub](https://github.com/alexbooker), and subscribe to my [YouTube channel](https://www.youtube.com/channel/UCcQsDUZiK1GWDcP7BpVO_kw) and [blog](https://booker.codes/rss/).**
