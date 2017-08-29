---
layout: post
title: Module Not Found
tags: javascript react TIL
---

So, this started when I ran into a _#facepalm_ moment while working on my first React project. I suddenly got an error message like the one below.

![Module Not Found]({{ site.url }}/assets/images/module_not_found.png)
<div class="caption">Failed to compile. Error in {path}. Module not found</div>

It took me longer than I’d like to admit to figure out the issue causing this error. See if you can spot the culprit in the code below.

```
import React from 'react';
import ReactDOM from 'react-dom';
import App from 'App';
ReactDOM.render(
  <App />,
  document.getElementById('root')
);
```

If you saw the error right away, I’m guessing you’ve run into this before...if you didn’t, you’re not alone. The problem is that my `App` import should be a relative path with a leading dot-slash `./`
```
import App from './App';
```

This fixed my issue and I went on with my day; but, later, I was a little curious about how imports worked. Why can I import modules like react without giving it the path?

---

## Digging In.
If you want to follow along to get this error, start by installing [create-react-app](https://github.com/facebookincubator/create-react-app) - which, by the way, is amazing. If your just learning React or starting a new React project, I think this is a great way to start (it abstracts away the build stuff which can be intimidating).

I’m just going to start by creating a new react app.

```
create-react-app react-app
cd react-app
npm start
```

In `src/index.js` change just remove the leading `./` from `./App` and boom, your browser will show you the same problem I had.

So what gives, why can we import some modules without providing the path? Well, that happens due to the magic of [NodeJS](NodeJS) and [Webpack](Webpack). When you use `import` it is compiled to `require` by [Babel](Babel).

![Babel Transpilation]({{ site.url }}/assets/images/babel_transpilation.png)
<div class="caption">`import` compiled with Babel</div>

Webpack wraps the standard NodeJS `require` with [some additional functionality](https://webpack.github.io/docs/configuration.html#resolve), but the answer to our question lies in [the way NodeJS implements require](https://nodejs.org/dist/latest-v6.x/docs/api/modules.html).

>If the module identifier passed to require() is not a native module, and does not begin with '/', '../', or './', then Node.js starts at the parent directory of the current module, and adds /node_modules, and attempts to load the module from that location. Node will not append node_modules to a path already ending in node_modules.

Great, so now we understand why a leading `./` is needed and the mechanism that allows modules in `node_modules` to be loaded by name. I wasn’t quite satisfied yet though, I wanted to see if it is possible to load my own modules by name only.

Remember how I said Webpack wraps `require` with some additional functionality, well the part of Webpack that determines what gets exported is the [resolve section](https://webpack.github.io/docs/configuration.html#resolve). To see how create-react-app has its Webpack config structured, you can run `npm run eject` which is a custom script that will unpack the build configuration into your project’s directory.
You can then check `config/webpack.config.dev.js` for the resolve section which should look something like this.

```
resolve: {
    fallback: paths.nodePaths,
    extensions: ['.js', '.json', '.jsx', ''],
    alias: { 'react-native': 'react-native-web' },
}
```

The part that is interesting is the fallback property. There is a great note in the config that informs the use of this property is to ensure that modules found in `node_modules` "win" in the case of a name clash. This means that if we add to `paths.nodePaths`, our application code won’t take precedence over an npm module we’ve loaded with the same name.

Pushing my curiosity a bit further, I decided to add my application’s source to `paths.nodePaths`. To do this, we just need to look at the `config/paths.js` file.

```
var nodePaths = (process.env.NODE_PATH || '')
  .split(process.platform === 'win32' ? ';' : ':')
  .filter(Boolean)
  .filter(folder => !path.isAbsolute(folder))
  .map(resolveApp);
```

We can see there are two ways of adding to `nodePaths`.

a) Adding the `NODE_PATH` environment variable.
```
export NODE_PATH=src
```
b) Editing the empty string after `NODE_PATH`.
```
var nodePaths = (process.env.NODE_PATH || 'src')
  .split(process.platform === 'win32' ? ';' : ':')
  .filter(Boolean)
  .filter(folder => !path.isAbsolute(folder))
  .map(resolveApp);
```

If you do one of these options and restart your server to rebuild the app, you should see the error go away and the page load. Cool!

We can take this even further and create a new file and import it with just the name.
```
// ~/src/hello/index.js
import React, { Component } from 'react'
class Hello extends Component {
  render() {
    return (<p>Hello World</p>)
  }
}
export default Hello
// ~/src/App.js
import React, { Component } from 'react';
import logo from 'logo.svg';
import 'App.css';
import Hello from 'hello'
class App extends Component {
  render() {
    return (
      <div className="App">
        <div className="App-header">
          <img src={logo} className="App-logo" alt="logo" />
          <h2>Welcome to React</h2>
        </div>
        <p className="App-intro">
          To get started, edit <code>src/App.js</code> and save to reload.
        </p>
        <Hello />
      </div>
    );
  }
}
export default App;
```

Pretty nifty; this means if I have some deeply nested file I don’t have to use really long relative paths aka from `'../../../../../module'` etc. Everything exported in my app will be available as a module `from 'module'`. However, this also has the downside of potential name-clashing and a performance hit as Webpack now has to look in more places for a module — so use discretion on whether or not that is a good idea (the fact that its not a default in create-react-app probably suggests it not being the best idea).

---

## Curiosity quenched.
I learned a good bit by going through that exercise, hopefully it helps someone…and maybe, just maybe, reduces a _#facepalm_ moment.

## tl;dr

1. Make sure relative paths to code within your app starts with a dot ( `./` or `../` )

2. Webpack wraps the Node.js `require` functionality which allows you to require `node_modules` without providing the path.

3. You can load your own components as modules, but doing so may not be the best idea.

## Further Reading
### NodeJS — Modules
[https://nodejs.org/api/modules.html](https://nodejs.org/api/modules.html)
Especially the sections on module loading. Webpack (and other bundlers) seem to be heavily based on the way Node handles module loading.

### BabelJS
[http://babeljs.io/learn-es2015/](https://nodejs.org/api/modules.html)
“Allow[s] you to use new syntax, right now without waiting for browser support.” You may notice that Webpack and other docs rarely use the `import` keyword; this is because it is an ES2015 feature and not fully supported by browsers. Babel handles converting `import`'s to `require`'s and so much more. Their whole site is great and informative, you should definitely look around.
