# Webpack  

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
