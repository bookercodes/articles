The [ES2015 specification](http://www.ecma-international.org/ecma-262/6.0/), which  defines the JavaScript programming language, was finalized back in June of 2015. Unfortunately, it'll be a while yet before all the major JavaScript engines finish implementing the updated specification, and even longer yet before those engines see wide adoption, both on the client _and_ on the server. Yes, even Node 5, which supports a good portion of ES6, has a relatively small adoption at this time ([source](https://docs.google.com/spreadsheets/d/1AY1GbB1WGix4CZXY6L-6QEFZlArN1C_Ew3jMMWQ1XpQ/edit#gid=0)).

It is due to the simple fact that most platforms don't yet support ES6, that it isn't feasible to publish **native** ES6 modules today. That being said, it is possible - and quite easy, in fact - to publish **transpiled** ES6 modules, and in this article, I'll teach you how.

## Transpilation

An ES6 transpiler is essentially a tool that takes ES6 source code as input, and outputs equivalent ES5 code, which is a much more widely supported version of the JavaScript language:

![](http://i.imgur.com/iPMIuDp.png)

The most popular ES6 transpiler, and the one we're going to use today is [Babel](https://github.com/babel/babel). Babel is used by many large technology companies including Facebook, and has an awesome community behind it. It also has an online [interactive environment](https://babeljs.io/repl/), if you want to experiment with the tool.

The fundamental idea behind publishing transpiled ES6 modules is that, you create two folders: a `source` folder, and a `distribution` folder:

![](http://i.imgur.com/V8ja3p3.png)

The `source` folder will contain your ES6 source files. You can then use Babel to transpile each source file, and direct the transpiled ES5 output to the `distribution` folder. The contents of this `distribution` folder can then be published to npm.

Because you'll ultimately be distributing vanilla ES5 code, your package will work seamlessly on any platform that supports ES5. ES5 has been around for a long time now, and so that's most platforms. Do bear in mind, though, that transpilation only relates to **language features**, and that you'll still need to rely on [ES6 polyfill](https://babeljs.io/docs/usage/polyfill/) if your module depends on ES6 **APIs** like like `Map`, `Set` and `Promise`.

## Installing Babel

I'm going to assume that you already have a suitable `package.json` file. If you don't, you can quickly create one using the [`npm init` command](https://docs.npmjs.com/cli/init).

Since version 6, Babel has become a very plugin-centric tool. In practice, this means you need to install two packages: **1,** [`babel-cli`](https://www.npmjs.com/package/babel-cli), which is essentially the core Babel tool, and **2,** [`babel-preset-es2015`](https://www.npmjs.com/package/babel-preset-es2015), which is a preset for all ES6 plugins.

Although it's possible to install Babel CLI globally on your machine, I suggest that you heed [Babel's advice](https://babeljs.io/docs/usage/cli/), and install both packages locally:

```
npm install --save-dev babel-cli@6 babel-preset-es2015@6
```

One tangible benefit of Babel's plugin architecture is that you can optionally install plugins for non-standard features like [`async` functions](https://jakearchibald.com/2014/es7-async-functions/). Another benefit is that, later down the road, when ES6 is commonplace and you inevitably want to use the next iteration of the language, all you'll need to do is replace your Babel preset!


## Write some ES6, today!

For demonstration purposes, I'm going to be publishing a trivial Node package that calculates the percentage of a given value.

Remember, all source files belong in the `source` folder, so create your main `index.js` script in that folder:

```javascript
module.exports = function({percent = 100, amount}) {
  const percentOff = (percent / 100) * amount;
  return percentOff;
}
```

I call this module _offof_ and because I'm writing this tutorial retroactively, you can find it on both [npm](https://www.npmjs.com/package/offof) and [GitHub](https://github.com/alexbooker/offof) now.

As you can see, I'm using a handful of ES6 features, including a [_destructing assignment_](https://developer.mozilla.org/en/docs/Web/JavaScript/Reference/Operators/Destructuring_assignment)  and [_`const` declaration_](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Statements/const).

You might be wondering why I'm using `module.exports` instead of an ES6 [_`export` statement_](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Statements/export). This question leads to an important point which is that, if you expect your module to be imported using CommonJS `require` - which, if you're targeting ES5, you probably do -  then you should export your module using CommonJS `module.exports`. If you don't, developers will have to go out of their way to use your module, which isn't good. For a more thorough explanation, and some alternative solutions, see [this post](https://medium.com/@kentcdodds/misunderstanding-es6-modules-upgrading-babel-tears-and-a-solution-ad2d5ab93ce0#.y8ewd1vb5) by [@kentcdodds](https://twitter.com/kentcdodds).

If you were to distribute this code as-is, most environments wouldn't understand it. How to transpile this code is the subject of the upcoming sections.

## Setting up Babel

To transpile the contents of the `source` folder, firstly add the following script to `package.json`:

```
"scripts": {
  "build": "babel source --presets babel-preset-es2015 --out-dir distribution"
}
```

In a nutshell, this script tells Babel to **1,** take the ES6 source files in the `source` folder, **2,** transpile them using the ES2015 preset, and **3,** output the transpiled ES5 files in the `distribution` folder.

> **Tip:** If, in the future, you want to use more than one preset, you may find declaring them in-line to be a bit unwieldy, in which case you could opt to create a [`.babelrc` file](https://babeljs.io/docs/usage/babelrc/).

To invoke the script, execute `npm run build` in your terminal.

After invoking this script, you'll be able to observe that the `distribution` folder has been populated with the transpiled file:

![](http://i.imgur.com/HtRRYiy.png)

To keep things simple, I'm only transpiling a single file. It is, however, possible to transpile multiple files. Also note that if no `distribution` folder exists, Babel will create one.

## Overriding the main module

Usually, Node will look for a main file within the module folder called `index.js`. When transpiling source files into a `distribution` folder, there shouldn't be a file at that location, so it is neccesary to overide this behaviour.

To specify an alternative path, `package.json` must be updated to contain a key named `main` that specifies the path to the main file. In this case, `./distribution/index.js`:

<pre>
<code>{
  <strong>"main": "./distribution/index.js",</strong>
   "scripts": {
     "build": ...
   }
}</code></pre>

## Ignoring files

You're almost ready to publish your Node module, but first you're going to want to tell npm to not upload the `source` folder by creating an `.npmignore` file with the following contents:

```
source
```

Conversely, you'll want to tell Git to not upload the `distribution` folder. You can do this by adding the below lines to your `.gitignore` file:

```
node_modules
distribution
```

**Heads up!** It is very important that if you have a `.gitignore` file, that you also have an `.npmignore` file. If you don't create an `.npmignore` file, npm will actually use your `.gitignore` as the `.npmignore`, which means that the `distribution` folder won't be published.

### Publishing to npm

Before publishing to npm, you'll obviously want to firstly build your source files using the `build` script you defined earlier. Instead of doing this manually, I _highly_ recommend that you create a special script recognized by npm called `prepublish` that npm will automatically execute before publishing the package:

<pre>
<code>"scripts": {
  "build": "babel-cli --preset xxx",
  <strong>"prepublish": "npm run build"</strong>
}</code></pre>

Now, when you run `npm publish`, npm will automatically run the `build` script. This is both convenient and helps to avoid errors.

Whilst I have found `prepublish` to work fine for my needs, I feel it important to mention that there is a [known UX issue](https://github.com/npm/npm/issues/10074#issue-112707857) with `prepublish` whereby the script is implicitly run when the package is installed. This may or may not be a problem for you.

If you haven't done so already, use [these instructions](https://gist.github.com/coolaj86/1318304) by [@coolaj86](https://github.com/coolaj86) to set your npm author information. Then, simply run the following command whilst in the module folder:

```
npm publish ./
```

After a few seconds, if you did everything correctly, your module should be published to the npm registry.

## Testing it out
Before I conclude, I want to show you that developers can seamlessly install and use your module, despite the fact that it was coded using ES6.

First of all, I can install the [`offof`](https://www.npmjs.com/package/offof) package just like I would any other npm package:

```
npm install
```
Then, from within Node 4 - an environment that has little to no notion of ES6 - I can import and use the module as though it were written in ES5:

![](http://i.imgur.com/2LMbpzi.png)

You might also find it instructive to observe the `node_modules` directory:

![](http://i.imgur.com/ZIggJnv.png)

As you can see, there's no trace of the `source` folder; there's only the `distribution` folder.

## Conclusion

In this article you learned how to utilize the syntactic niceties afforded by ES6 whilst still maintaining support for platforms that don't yet support it. You also learned about some potential pitfalls along the way, like how you should probably use `module.exports` instead of an ES6 export statement.

_Special thanks to: [Maarten Van Giel](https://twitter.com/maartenvangiel), [Romaine Rose-Cameron](https://twitter.com/R0ma1ne), [partycoder](https://www.livecoding.tv/partycoder/), and [kevin-DL](https://github.com/kevin-DL)._

**P.S. If you read this far, you might want to follow me on [Twitter](https://twitter.com/bookercodes) and [GitHub](https://github.com/alexbooker), or [subscribe](https://booker.codes/rss/) to my blog.**
