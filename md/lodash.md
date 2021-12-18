[官方文档](https://www.lodashjs.com)

Lodash 是一个一致性、模块化、高性能的 JavaScript 实用工具库。Lodash 通过降低 array、number、objects、string 等等的使用难度从而让 JavaScript 变得更简单

```shell
$ npm i --save lodash
```

```js
// Load the full build.
var _ = require('lodash');
// Load the core build.
var _ = require('lodash/core');
// Load the FP build for immutable auto-curried iteratee-first data-last methods.
var fp = require('lodash/fp');
 
// Load method categories.
var array = require('lodash/array');
var object = require('lodash/fp/object');
 
// Cherry-pick methods for smaller browserify/rollup/webpack bundles.
var at = require('lodash/at');
var curryN = require('lodash/fp/curryN');
```


## 常用API


```js
var array = [1, 2, 3, 4];
var evens = _.remove(array, function(n) {
  return n % 2 == 0;
});
 
console.log(array);
// => [1, 3]
 
console.log(evens);
// => [2, 4]

```