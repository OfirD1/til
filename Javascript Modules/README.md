
# Javascript Modules

### Definitions

#### Module Systems

- **CommonJS**
  - A module specification for JavaScript widely used today, in particular for server-side JavaScript.
  - A widely used implemenation of CommonJS is Node.js (which only implements the Modules and Packages CommonJS proposals).  
  
    Some other example implementaions are:
    - CouchDB (in its [javascript query server](https://docs.couchdb.org/en/stable/query-server/javascript.html?highlight=commonjs#commonjs-modules))
    - [Common Node](https://github.com/olegp/common-node) (implements other CommonJS proposals on top of Node.js)
  - Can be recognized by the use of the `require()` function and `module.exports`.
  - Browsers don't support CommonJS modules. 
  
    To use a CommonJS module in the browser, it needs to be transpiled to a different form of a module, which would be browser-supported. 
  
- **ES Modules (aka ES6 modules, aka ES2015 modules)**
  - A module specification for JavaScript widely used today, in particular for client-side JavaScript. 
  - Can be recognized by the use of `import` and `export` statements.

- **AMD**
	- AMD (Asynchronous Module Definition) is designed to suit the browser environment.  
    It started as a spinoff of the CommonJS Transport format and evolved into its own module definition API. Hence the similarities between the two.  
    One major difference from CommonJS is that AMD specifies that modules are loaded asynchronously - that means modules are loaded in parallel, as opposed to blocking the execution by waiting for a load to finish.  
    In AMD, the `define()` function allows the module to declare its dependencies before being loaded.  
    RequireJS is probably the most popular implementation of AMD.


#### Module vs Package in Node.js
  - If it can be `require`d, it's a module.
  - If it is a folder with a `package.json`, it's a package.  
  
Notes:
  - Node.js documentation treats both [files](https://nodejs.org/api/modules.html#modules_modules_commonjs_modules) and [folders](https://nodejs.org/api/modules.html#modules_folders_as_modules) 
    as modules because they can be `require`d.   
    
    At the same time, it says that one form of such folder-as-module is a Package, which is indeed a folder with a `package.json`.
    NPM (*Node* Package Manager) [summarizes](https://docs.npmjs.com/about-packages-and-modules) it best:  
    
    > Note: Since modules are not required to have a `package.json` file, not all modules are packages. Only modules that have a `package.json` file are also packages.
  - Taking the [CommonJS specification](http://wiki.commonjs.org/wiki/Modules/1.1.1) stance, a Module is a code contract that has:
    1. A `require` function
    2. An `exports` objects
    3. A context
    4. Identifiers
    While a Package:
    1. Is a cohesive wrapping of a collection of modules, code and other assets into a single form.
    2. Provides a top-level package descriptor, "package.json".
      
#### Module Resolution
The best description is in [TypeScript's docs](https://www.typescriptlang.org/docs/handbook/module-resolution.html#node):

> Relative paths are fairly straightforward.
As an example, let's consider a file located at `/root/src/moduleA.js`, which contains the import `var x = require("./moduleB");`
Node.js resolves that import in the following order:
>1. Ask the file named `/root/src/moduleB.js`, if it exists.
> 2. Ask the folder `/root/src/moduleB` if it contains a file named `package.json` that specifies a `"main"` module.
   In our example, if Node.js found the file `/root/src/moduleB/package.json` containing `{ "main": "lib/mainModule.js" }`, then Node.js will refer to `/root/src/moduleB/lib/mainModule.js`.
>3. Ask the folder `/root/src/moduleB` if it contains a file named `index.js`.
   That file is implicitly considered that folder's "main" module.
>  
>However, resolution for a [non-relative module name](#relative-vs-non-relative-module-imports) is performed differently.
Node will look for your modules in special folders named `node_modules`.
A `node_modules` folder can be on the same level as the current file, or higher up in the directory chain.
Node will walk up the directory chain, looking through each `node_modules` until it finds the module you tried to load.
>
>Following up our example above, consider if `/root/src/moduleA.js` instead used a non-relative path and had the import `var x = require("moduleB");`.
Node would then try to resolve `moduleB` to each of the locations until one worked.
>1. `/root/src/node_modules/moduleB.js`  
>2. `/root/src/node_modules/moduleB/package.json` (if it specifies a `"main"` property)
>3. `/root/src/node_modules/moduleB/index.js`
><br/>
>4. `/root/node_modules/moduleB.js`
>5. `/root/node_modules/moduleB/package.json` (if it specifies a `"main"` property)
>6. `/root/node_modules/moduleB/index.js`
><br/>
>7. `/node_modules/moduleB.js`
>8. `/node_modules/moduleB/package.json` (if it specifies a `"main"` property)
>9. `/node_modules/moduleB/index.js`
>
>Notice that Node.js jumped up a directory in steps (4) and (7).

#### Some Comparisons

##### Using `module.exports` vs `exports` in CommonJS
Two basic facts should be understood:
1. Both `exports` and `module.exports` point to the same object, unless you reassign one. 
2. Only `module.exports` is returned (automatically, no need to that explicitly) (see this related [SO post](https://stackoverflow.com/questions/7137397/module-exports-vs-exports-in-node-js)).

Confirming fact #1 ([source](https://www.hacksparrow.com/nodejs/exports-vs-module-exports.html)):

```javascript
// run.js
console.log(module);
```
Running:
```sh
$ node run.js
```
Output:
```sh
Module { 
   id: '.', 
   exports: {}, <-- the 'exports' object
   parent: null, 
   filename: '/Users/yaapa/projects/hacksparrow.com/run.js', 
   loaded: false, 
   children: [], 
   paths: [ 
	   '/Users/yaapa/projects/hacksparrow.com/node_modules', 
	   '/Users/yaapa/projects/node_modules', 
	   '/Users/yaapa/node_modules', 
	   '/Users/node_modules', 
	   '/node_modules' 
   ] 
}
```
Now edit `run.js` and run it again:
```javascript
// run.js
exports.a = 'A'; 
exports.b = 'B';
```
Running:
```sh
$ node run.js
```
Output:
```sh
Module { 
   id: '.', 
   exports: { a: 'A', b: 'B' }, <-- updated 'exports' object
   ...
}
```
You can see that assigning properties to the  `exports`  object has added those properties to  `modules.exports`.

However, if you assign anything to  `module.exports` itself (as oppsed to as assigning its properties),  `exports`  is not no longer a reference to it, and  `exports`  loses all its power.  

The opposite is also true: if you assign anything to `exports` itself (as opposed to as assigning its properties), it is no longer a reference to `module.exports`.

For example:
```javascript
// run.js

module.exports = {a: 'A'};
exports.b = 'B';
console.log(exports === module.exports);
console.log(module)
```
Running:
```sh
$ node run.js
```
Output:
```sh
false
Module {
  id: '.',
  exports: { a: 'A' },
  ...
```
And in the opposite way:
```javascript
// run.js

module.exports.a = 'A';
exports = 'B';
console.log(exports === module.exports);
console.log(module)
```
Running:
```sh
$ node run.js
```
Output:
```sh
false
Module {
  id: '.',
  exports: { a: 'A' },
  ...
```

##### Using default export and import vs. named export and import in ES6  

[The following is a summary of [MDN export](https://developer.mozilla.org/en-US/docs/web/javascript/reference/statements/export), [MDN import](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Statements/import), and [this](https://stackoverflow.com/a/45698115/3002584) SO posts] 

There are two different types of export and import: **named** and **default**. 

- **`export`**  

	You can have multiple **named** exports per module, but only one **default** export. 
	- **If a default export is not present, then no default will be exported.**
	 
	Each type corresponds to one of the following syntax:

	- Named exports:

		```js
		// export features declared earlier
		export { myFunction, myVariable };

		// export individual features (can export var, let,
		// const, function, class)
		export let myVariable = Math.sqrt(2);
		export function myFunction() { ... };

		```


	- Default exports:

		```js
		// export feature declared earlier as default
		export { myFunction as default };

		// export individual features as default
		export default function () { ... }
		export default class { .. }

		// each export overwrites the previous one
		```


- **`import`**  

	During the import of named exports, it is mandatory to use the same name of the corresponding object.  
  
	However, a default export can be imported with any name.
	- Named imports:
	
		```js
		import {myExport} from '/modules/my-module.js';
		import {myExport as someExport} from '/modules/my-module.js';
		```
	
	- Default imports:
		```js
		import myDefaultExport from '/modules/my-module.js';
		```
	- **`import *`**
		This imports all the exports from a file:

		```javascript
		//a.js
		export function doSomething() {}
		```
		```javascript
		//b.js
		export function doSomething() {}
		export default function doSomethingDefault() {}
		```
		```javascript
		import * as a from 'a'
		import * as b from 'b'
		a // a = { doSomething: function }
		b // b = { doSomething: function, default: function })
		```


- **Additional Possibilities**  

	- **`export from`**   
  
		It is also possible to "import/export" from different modules in a parent module so that they are available to import from that module. In other words, one can create a single module concentrating various exports from various modules.

		This can be achieved with the "export from" syntax:
		```js
		export { default as function1,
		         function2 } from 'bar.js';
		```
		Which is comparable to a combination of import and export:
		```js
		import { default as function1,
		         function2 } from 'bar.js';
		export { function1 as default, function2 };
		```

	- **Import a module for its side effects only**  
		We can import an entire module for side effects only, without importing anything. This runs the module's global code, but doesn't actually import any values:
		```js
		import '/modules/my-module.js';
		```

##### Using CommonJS modules vs. ES6 modules  in Node.js  

- Node.js treats JavaScript code as CommonJS modules by default. Authors can tell Node.js to treat JavaScript code as ECMAScript modules via the `.mjs` file extension, the `package.json`  [`"type"`](https://nodejs.org/api/packages.html#packages_type) field, or the `--input-type` flag. See [Modules: Packages](https://nodejs.org/api/packages.html#packages_determining_module_system) for more details (also take a specific look at the [dual-package section](https://nodejs.org/api/packages.html#packages_approach_1_use_an_es_module_wrapper)).

- Don't confuse terminology (based on [this](https://stackoverflow.com/a/40295288/3002584) SO post): 

	- CommonsJS uses `exports` for exporting, while ES6 uses `export` for exporting, with an extra reserved keyword  `export default` for "deafult" export.   
	- Some people consider CommonJS's `module.exports = foo` to be equivalent to ES6's `export default foo` and CommonJS's `module.exports.bar...` to be equivalent to ES6's `export const bar = ...`.  
  
	    The reason is that using ES6's default import `import foo from ...` (or any other ES6 possible default import way) is the way to import a CommonJS's module exported with `module.exports = foo`, and using ES6's named import `import {bar} from ...` (or any other ES6 possible named import way) is the way to import a CommonJS's `module.exports`'s property exported with `module.exports.bar = ...`.  
	
      However, this is simply a mix-up of terminology, because CommonJS doesn't have the notion of "default" or "named" export or import as ES6 does. All it has is `module.exports` and `require`. 
    
      It is the *compatability* of CommonJS and ES6 that leads people to come up with this mix-up.
    
	- In addition to the last point, at least with how Babel transpile a ES6 module to a CommonJS module, both named and default exports are actually created as a `module.exports` property! 
	  
      Here's how Babel compiles named and default exports:
		```javascript  
		// input
		export const foo = 42;
		export default 21;
		
		// output
		"use strict";

		Object.defineProperty(exports, "__esModule", {
		  value: true
		});
		var foo = exports.foo = 42;
		exports.default = 21; 
		```

- Summary of syntax ([source](https://stackoverflow.com/a/36720932/3002584))
	 
	- **CommonJS export, ES6 import**
		```js
	    module.exports = 1    // a.js 
	    import one from './a' // b.js 
	    // one === 1
	    
	    module.exports = {one: 1}  // a.js 
	    import obj from './a'      // b.js 
	    // obj is {one: 1}
		// --- or: ---
	    import * as obj from './a' // b.js 
	    // obj.one === 1 ; however, check out the 'funky stuff' below
		// --- or: ---
	    import {one} from './a'    // b.js <-- named import
	    // one === 1
		```
	- **ES6 export, CommonJS require**
		```js
	    export default 1 // a.js
	    const one = require('./a').default
	    // one === 1
	    
	    export const one = 1
	    const one = require('./a').one
	    // one === 1
	    ```

	- **Funky stuff**
		```js
	    module.exports = 1          // a.js
	    import * as obj from './a'  // b.js
	    // obj is {default: 1}

	    module.exports = {one: 1}   // a.js
	    import * as obj from './a'  // b.js
	    // obj is {one: 1, default: {one: 1}}
	    ```
	  - Note that until Node v14.13.0, only default imports was possible ([source](https://stackoverflow.com/questions/61549406/how-to-include-commonjs-module-in-es6-module-node-app#comment113574429_61581222)).
- Although CommonJS and ES6 modules share similar (though not identical) syntax, they work in fundamentally different ways ([source](https://www.sitepoint.com/understanding-es6-modules)):
	- ES6 modules are pre-parsed in order to resolve further imports before code is executed.
	- CommonJS modules load dependencies on demand while executing the code.
		- Example:

		  - **CommonJS modules**  
  
		    Code:
		    ```javascript
		    // ---------------------------------
		    // one.js
		    console.log('running one.js');
		    const hello = require('./two.js'); <-- CommonJS require
		    console.log(hello);

		    // ---------------------------------
		    // two.js
		    console.log('running two.js');
		    module.exports = 'Hello from two.js'; <-- CommonJS exports
		    ```
		    Output:
		    ```
		    running one.js
		    running two.js
		    hello from two.js
		    ```
    
		  - **ES6 modules**
  
		    Code:
		    ```javascript
		    // ---------------------------------
		    // one.js
		    console.log('running one.js');
		    import { hello } from './two.js'; <-- ES6 import
		    console.log(hello);

		    // ---------------------------------
		    // two.js
		    console.log('running two.js');
		    export const hello = 'hello from two.js'; <-- ES6 export
		    ```
		    Output:
		    ```
		    running two.js <-- two is first!
		    running one.js
		    hello from two.js
		    ```
   


##### Using (`import`ing) a CommonJS module in a ES6 module to be loaded in a `<script>` tag
[That's not possible](https://stackoverflow.com/questions/55167994/is-it-possible-to-es6-import-a-commonjs-module) natively, but can be done using a module loader (like RequireJS) or a transpiler.

##### `require` vs.  RequireJS

- `require` is Node.js's way of implementing CommonJS specification to import modules, while [RequireJS](https://requirejs.org/docs/history.html) is a library that implements the AMD specification (and which has a `requirejs` and `require` functions itself).
- Importanatly, RequireJS is [competeble with CommonJS modules](https://requirejs.org/docs/whyamd.html#commonjscompat) (but the opposite is not true).
- see this [answer](https://stackoverflow.com/questions/16521471/relation-between-commonjs-amd-and-requirejs).
