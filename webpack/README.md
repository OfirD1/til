# Webpack  

### Basic Bundle Creation (Which only works for use in `<script>` tag)

([source](https://webpack.js.org/guides/getting-started/), [demo](https://stackblitz.com/github/webpack/webpack.js.org/tree/master/examples/getting-started?file=README.md&terminal=)) 

Say that we have some `.js` file that imports some library, and an `.html` file that needs to load that `.js` file.  
Our `.html` file shouldn't care what libraries that `.js` file loads, and just wants to load that one file and nothing more.

To make that work, we need to create a **bundle** out of our `index.js` and the imported library (which in the following example is lodash).

So, here's our initial folder structure:

```git
webpack-demo
|- /dist
  |- index.html
|- /src
  |- index.js
|- package.json
|- webpack.config.js
```

And here are our files, one by one:

**`index.js`**  
```js
import _ from 'lodash'; // <-- importing lodash
function component() {
  const element = document.createElement('div');
  element.innerHTML = _.join(['Hello', 'Webpack'], ' '); // <-- using lodash
  return element;
}
// appends "Hello Webpack" to document.body
document.body.appendChild(component());
```

**`index.html`**  
```html
<!DOCTYPE html>
<html>
  <head>
    <meta charset="utf-8" />
    <title>Getting Started</title>
  </head>
  <body>
    <script src="main.js"></script> <-- loading 
  </body>
</html>
```

**`webpack.config.js`**  
```js
const path = require('path');

module.exports = {
  entry: './src/index.js',
  output: {
    filename: 'main.js',
    path: path.resolve(__dirname, 'dist'),
  },
};
```

**`package.json`**
```json
{
  "scripts": {
    "build": "webpack",
    "start": "serve dist"
  },
  "dependencies": {
    "lodash": "^4.17.21"
  },
  "devDependencies": {
    "serve": "^12.0.0",
    "webpack": "^5.38.1",
    "webpack-cli": "^4.7.2"
  }
}
```

We now run:
1. `npm install` 
2. `npm run build`

And now this is how our folder stucture look:

```git
webpack-demo
|- /dist
  |- index.html
  |- main.js     <-- created
|- /src
  |- index.js
|- /node_modules <-- created
|- package.json
|- webpack.config.js
```

Finally, we run:
`npm run start`
And the result is a `"Hello Webpack"` div.

#### However, this only works when including the bundle in a `<script>` tag!

The reason is that by default, Webpack creates the bundle with a global variable named as the exported value.

Read again: our exported variable is NOT exported, but simply declared as a global variable!

This is the meaning of the somewhat vague description in the [documentation](https://webpack.js.org/configuration/output/#expose-via-object-assignment):

> **Expose Via Object Assignment**
>
>These options assign the return value of the entry point (e.g. whatever the entry point exported) to a specific object under the name defined by `output.library.name`.
> [...]
> 
> **Warning**
>
> Note that not setting a `output.library.name` will cause all properties returned by the entry point to be assigned to the given object.

And somewhat more explicitly, though still quite obscure description in [expose-the-library](https://webpack.js.org/guides/author-libraries/#expose-the-library):

>[Using only `library.name` or omitting it] only works when it's referenced through script tag, it can't be used in other environments like CommonJS, AMD, Node.js, etc.

 
### Exposing a bundle as a global variable  


Suppose `demo.html` needs to use a ***single*** `<script>` tag to load a bundle of `.js` files:  

```
webpack-demo
 |- demo.html
 |- hello1.js
 |- hello2.js
```
    
Where each `.js` file has a `default` export in a slightly different way:  
    
```javascript
// hello1.js
module.exports = {
  hello1: function () { console.log('hello1'); }
}; 
```
```javascript
// hello2.js
var Hello2 = {
  hello2: function () { console.log('hello2'); }
}
module.exports = Hello2;
```
        
* *Note:*  
  *1. `module.exports = foo` is equivalent to `export default foo`* (see [here](https://stackoverflow.com/a/34795237/3002584)).  
  *2. Using `export default` as a practice is arguable, but I came across it when working with `CoffeeScript` generated files.*   
  
       
Here's how to bundle these exports using Webpack:  
     
   1. Create `index.js`:  
        
      *Let's suffix the names with an `X` to easily differ the original objects from the current ones*  
       ```javascript
       module.exports = {
         hello1X: require("./hello1"),
         hello2X: require("./hello2")
       };
       ```
     
   2. Create `webpack.config.js`:  
     
      ```javascript
      const path = require('path');
      module.exports = {
        entry: './index.js',
        output: {
          filename: 'bundle.js',
          path: path.resolve(__dirname, 'dist'),
          library: 'hello'
        }
      };
      ```

Great! this is our updated folder structure:
      
```
webpack-demo
  |- demo.html
  |- hello1.js
  |- hello2.js
  |- index.js
  |- webpack.config.js
```
Now, run `webpack index.js`.  The resultant folder structure is:
```
webpack-demo
  |- dist
    |- bundle.js
  |- demo.html
  |- hello1.js
  |- hello2.js
  |- index.js
  |- webpack.config.js
```
Finally, in our `demo.html` file:
```html
<script src="./dist/bundle.js"></script>
<script>
   hello.hello1X.hello1(); // prints "hello1"
   hello.hello2X.hello2(); // prints "hello2"
</script>
```

### Build an ES module bundle, and consume that bundle in a Node.js ES module

([source](https://stackoverflow.com/questions/68913996/use-webpack-5-to-build-an-es-module-bundle-and-consume-that-bundle-in-a-node-js) 

It's possible to use Webpack 5 to build an ES module bundle, and then consume that bundle in a Node.js ES module.

To do that, `webpack.config.js` will *also* need to be written in ES Modules syntax.

So, here's our initial folder structure:  

    |- webpack.config.js
    |- package.json
    |- /src
      |- index.js

**`index.js`**

    function util() {
    	return "I'm a util!";
    }
    export default util;

**`webpack.config.js`**

    import path from "path";
    import { fileURLToPath } from "url";

    const __dirname = path.dirname(fileURLToPath(import.meta.url));

    export default {
      mode: "development",     
      entry: "./src/index.js",
      output: {
        filename: "bundle.js",
        path: path.resolve(__dirname, "dist"),
        library: {
          type: "module",
        },
      },
      experiments: {
        outputModule: true,
      },
    };

**`package.json`**  

    {
      "name": "exporter",
      "version": "1.0.0",
      "main": "dist/bundle.js",
      "scripts": {
        "build": "webpack"
      },
      "devDependencies": {
        "webpack": "^5.51.1",
        "webpack-cli": "^4.8.0"
      },
      "type": "module"
    }

After running `npm run build`, we get:

    |- webpack.config.js
    |- package.json
    |- /src
      |- index.js
    |- /dist
      |- bundle.js

Great, now we want to just create some "demo" file `importer.js` which would import `util` and use it. For convenience, let's create it on the same folder:

    |- importer.js
    |- webpack.config.js
    |- package.json
    |- /src
      |- index.js
    |- /dist
      |- bundle.js

**`importer.js`**  

    import util from './dist/bundle.js';
    
    console.log(util());
    
Finally, run:
```bash
⚡  node importer.js 
I'm a util!
```
