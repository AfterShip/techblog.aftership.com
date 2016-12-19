---
title: JavaScript 102
author: Hans Lee
date: 2016-12-19 16:45
categories:
  - JavaScript
tags:
  - JavaScript
  - Lodash
  - Code Quality
---

JavaScript is essential to companies using `Node.js`. It's THE language of year 2016. However, it is also a fragile language could be easily abused.

<!-- more -->
## Road toward clean code (less bug)

In this article, I am going to demonstrate some best practices in JavaScript with examples.

## `sum` function
To begin with, we are required to write a simple sum function. Here is the requirement:

1. the function accepts **1** argument - an array of numbers
2. the function should return the sum of all the numbers inside the array

### for loop (not recommended)

``` javascript
// array = [1,2,3,4]
function sum(array) {
  let result = 0;
  for (var i = 0; i < array.length; i++) {
    result += array[i];
  }
  return result;
}
```

The first approach is intuitive: use traditional `for` loop. There is nothing wrong about it, but as a js developer, we could do better.

### for ... in loop (**DEPRECATED**, DO NOT USE)

And here is the famous `foo ... in` loop.

As you can see, the obvious drawback of `for ... in` loop is that it could iterate over all of the keys and all the keys on it's prototype chain inside an object (array is also an object in js), therefore we have this `hasOwnProperty` check.

an astonishing fact is that a lot of programmers from other languages really love this approach... However, the cold fact is due to the cumbersome `hasOwnProperty` check, `for ... in` loop is seldom used in most of the `Node.js` project, and even considered bad practice when iterate over a collection.

Here is the reason:

>  The for-in statement by itself is not a "bad practice", however it can be mis-used, for example, to iterate over arrays or array-like objects.

> The purpose of the for-in statement is to enumerate over object properties. This statement will go up in the prototype chain, also enumerating over inherited properties, a thing that sometimes is not desired.

> Also, the order of iteration is not guaranteed by the spec., meaning that if you want to "iterate" an array object, with this statement you cannot be sure that the properties (array indexes) will be visited in the numeric order.

``` javascript
// array = [1,2,3,4]
function sum(array) {
  let result = 0;
  for (let index in array) {
    if (array.hasOwnProperty(index)) {
      result += array[index];
    }
  }
  return result;
}
```

If one should iterate over every property in an `object` (`{}`) rather than `array` (`[]`), consider use those methods:

``` javascript
Object.keys           <-> _.keys
Object.values  (es7)  <-> _.values
Object.entries (es7)  <-> _.pairs
```

Those methods are not following the prototype chain to do an exhaustive search so they may even have better performance compared to the `for ... in` loop.

### for ... of loop (es6) (Okay.. not the best)

``` javascript
// array = [1,2,3,4]
function sum(array) {
  let result = 0;
  for (let number of array) {
    result += number;
  }
  return result;
}
```

this is a more recommended way to iterate a collection (or iterable to be precise).

`for ... of` will not follow the prototype chain as well, because it can only be used on `iterable object`. And `Array` in es6 is implemented as an iterable. [iterable-mdn](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Guide/Iterators_and_Generators)

### Array.prototype.forEach (Okay, not the best)

``` javascript
// array = [1,2,3,4]
function sum(array) {
  let result = 0;
  array.forEach(number => result += number);
  return result;
}
```

Nothing wrong about it, but we still need a `result` tmp var.


### Array.prototype.reduce (better)

``` javascript
// array = [1,2,3,4]
function sum(array) {
  return array.reduce((pre, cur) => {
    return pre + cur;
  });
}
```

this it the killer function of javascript, reduce function simply turns a `collection` in to a single value.

<div class="tip">
Remember, you MUST return the accumulated result in the reducer function
</div>

We finally get rid of the annoying tmp var `result`! Yeal~


### \_.sum (best)

``` javascript
// array = [1,2,3,4]
function sum(array) {
  return _.sum(array);
}
```

The real killer in JavaScript is the `lodash` library. It has became the de-factor JavaScript utility library. So Whenever you starts to write a function, the first thing to come up is always:

can i achieve it in lodash?

can i achieve it in lodash?

can i achieve it in lodash?

(have to repeat 3 times to emphersize important things)

<div class="tip">
Always exploring lodash for data transformation tasks
</div>

## `clean` function

consider the `clean` function here:

``` javascript
// ['tag1', 'tag2', 'tag1  '] -> ['tag1', 'tag2']
function clean(tags) {
  let dedupe = {};
  for (let i = 0; i < tags.length; i++){
    tags[i] = tags[i].trim();
    const aTag = tags[i];

    if(dedupe[aTag]) {
      tags.splice(i, 1);
      i--;
    } else {
      dedupe[aTag] = true;
    }
  }
  return tags;
}
```

it takes some time to brain-parse this function, it seems to work but it has a potential bug: it changes the reference of the argument passed in.

### impurity

``` diff
// ['tag1', 'tag2', 'tag1  '] -> ['tag1', 'tag2']
function clean(tags) {
  let dedupe = {};
  for (let i = 0; i < tags.length; i++){
-    tags[i] = tags[i].trim(); // mutates the tags
    const aTag = tags[i];

    if(dedupe[aTag]) {
-      tags.splice(i, 1); // mutates the tags
      i--;
    } else {
      dedupe[aTag] = true;
    }
  }
  return tags;
}
```

without mentioning the mysterious `splice` method with a dynamic `i` variable you need to brain-parse, the highlighted 2 lines are directly mutating the argument.

Imagine the `tags` argument passed in could be used in somewhere else, then this function has a side effect that is really hard to predict and could lead to a bug that is extremely hard to trace.

This is called `side effect` in functional programming world. and the `clean` function above is defined as `impure`

### pure function
So we could come up with a better solution:

``` diff
// ['tag1', 'tag2', 'tag1  '] -> ['tag1', 'tag2']
function clean(tags) {
  let dedupe = {};
+ let result = [];
  for (let i = 0; i < tags.length; i++){
-   tags[i] = tags[i].trim(); // mutates the tags
+   const aTag = tags[i].trim();

    if(dedupe[aTag]) {
-     tags.splice(i, 1); // mutates the tags
-     i--;
+     result.push(aTag);
    } else {
      dedupe[aTag] = true;
    }
  }
  return tags;
}
```

with this implementation, at least we have get rid of the side effects and eliminated a potential caused by direct data mutation, but we could still do better:

### Concatenated loops

{% asset_img "loop-patterns.png" "Loop patterns" %}

As illustrated in the image above, generally a good practice is try to wring `Concatenated loop`.

To achieve this goal, we could follow a simple rule:

> Inside loop, try to do only 1 thing

try to apply that to the `clean` function, we could get

``` javascript
// ['tag1', 'tag2', 'tag1'] -> ['tag1', 'tag2']
function clean(tags) {
  let dedupe = {};
  let result = [];

  for (let i = 0; i < tags.length; i++) {
    dedupe[tags[i].trim()] = true;
  }

  for (let i = 0; i < tags.length; i++) {
    if (dedupe[tags[i].trim()]) {
      result.push(tags[i])
    }
  }

  return result;
}
```

Here inside those 2 loops, we could easily figure out what they are doing respectively.


### Lazy evaluation

However, some people may ask: this is a `O(2n)` compared to the previous `O(n)` function.

Here will introduce the killer feature of `lodash` - chainable api

it is basically implemented using `_.chain(collection)` method, or `_(collection)` as a shorthand, and could be optimized easily

``` javascript
const result = _(source).map(func1).map(func2).map(func3).value();
```

is transformed to something like following in normal mode.

``` javascript
let result = [], temp1 = [], temp2 = [], temp3 = [];

for (var i = 0; i < source.length; i++) {
 temp1[i] = func1(source[i]);
}

for (i = 0; i < source.length; i++) {
 temp2[i] = func2(temp1[i]);
}

for (i = 0; i < source.length; i++) {
 temp3[i] = func3(temp2[i]);
}

result = temp3;
```

But in the lazy mode (could be accomplished with `lazy.js`):

``` javascript
let result = [];
for (var i = 0; i < source.length; i++) {
  result[i] = func3(func2(func1(source[i])));
}
```

as you could see, `O(kn)` algorithm has been optimized to `O(n)`. With pure `for` loop, this optimization is not easy to be achived, but with `lodash` and a sequence of function transformation, this could be easily achieved.

### Lodash Chainable API: rewrite `clean` function

``` javascript
// ['tag1', 'tag2  ', 'tag1 '] -> ['tag1', 'tag2']
function clean(tags) {
  return _(tags)
    .map(_.trim)
    .uniq()
    .value();
}
```

this is so much cleaner than the first version of above!

read the documentation on [lodash chain](https://lodash.com/docs/4.17.2#chain) for more black magic

## `mergeAndOverwrite` function

consider the function below:

``` javascript
function mergeAndOverwrite(target, source) {
  target = _.merge(_.cloneDeep(target), source.merge);

  for (let config_key in source.overwrite) {
    if (source.overwrite.hasOwnProperty(config_key)) {
      target[config_key] = source.overwrite[config_key];
    }
  }

  return target;
}
```

there are 2 gotchas:
1. 1st parameter in `_.merge` could be an empty object, and `_.merge` is a recursive deep operation already, so `_.cloneDeep` is somewhat a cumbersome call.
2. the for ... in loop could be entirely replaced by `Object.assign`


### \_.merge vs Object.assign

Those 2 functions are similar but do things in totally different ways:

1. \.merge is a recursive call, so it traverses **every** property all object and merge them
2. Object.assign only overwrite the 1st layer of right hand-side to left hands-side

``` javascript
// => { "a": { "a": "a","b":"bb" }}
_.merge({}, {a:{a:'a'}}, {a:{b:'bb'}})
// => { "a": { "b": "bb" }}
Object.assign({}, {a:{a:'a'}}, {a:{b:'bb'}})

// => { "a": [ "bb" ] }
_.merge({}, {a:['a']}, {a:['bb']})
// => { "a": [ "bb" ] }
Object.assign({}, {a:['a']}, {a:['bb']})
```

### rewrite the `mergeAndOverwrite` function

``` javascript
function mergeAndOverwrite(target, source) {
  const merged = _.merge(
    {},
    target,
    source.merge
  );
  return Object.assign(
    merged,
    source.overwrite
  );
}
```

<div class="tip">
Read through Lodash doc before using \_.cloneDeep
</div>

## Final challenge: indentation hell

To wrap up, consider the following function

``` javascript
function filterAck(acl_hash) {
  // 0
  let filtered_api_acl = {};
  for (let api_version in acl_hash) {
    // 1
    if (acl_hash.hasOwnProperty(api_version)) {
      // 2
      if (config.api_key.default_user_api_acl[api_version]) {
        // 3
        filtered_api_acl[api_version] = [];
        if (_.isArray(acl_hash[api_version])) {
          // 4
          for (let i = 0; i < acl_hash[api_version].length; i++) {
            // 5
            if (config.api_key.default_user_api_acl[api_version].indexOf(acl_hash[api_version][i]) !== -1) {
              // 6
              filtered_api_acl[api_version].push(acl_hash[api_version][i]);
            }
          }
        }
      }
    }
  }
  return filtered_api_acl;
}
```

<div class="tip">
We human beings are not good at reading deeply nested indentation hells: Research shows we only have 7 buffers for short term memory... So code like below will literally burn our brain out.
</div>

{% asset_img "indentation-hell.jpg" "indentation hell" %}

Whenever writing code, try your best to keep the indentation level <= 3 is generally a good practice, **this could vastly improve maintainability of code.**

### 1. use `Object.keys` to iterate an object

indentation level `-1`, current `5`

``` diff
function filterAck(acl_hash) {
  let filtered_api_acl = {};
- for (let api_version in acl_hash) {
-   if (acl_hash.hasOwnProperty(api_version)) {
+ Object.keys(acl_hash).forEach(api_version => {
-     if (config.api_key.default_user_api_acl[api_version]) {
+   if (config.api_key.default_user_api_acl[api_version]) {
-       filtered_api_acl[api_version] = [];
+     filtered_api_acl[api_version] = [];
-       if (_.isArray(acl_hash[api_version])) {
+     if (_.isArray(acl_hash[api_version])) {
-         for (let i = 0; i < acl_hash[api_version].length; i++) {
+       for (let i = 0; i < acl_hash[api_version].length; i++) {
-           if (config.api_key.default_user_api_acl[api_version].indexOf(acl_hash[api_version][i]) !== -1) {
+         if (config.api_key.default_user_api_acl[api_version].indexOf(acl_hash[api_version][i]) !== -1) {
-             filtered_api_acl[api_version].push(acl_hash[api_version][i]);
            // 5
+           filtered_api_acl[api_version].push(acl_hash[api_version][i]);
-           }
+         }
-         }
+       }
-       }
+     }
-     }
+   }
- }
+ });
  return filtered_api_acl;
}
```

### 2. use `_.intersction` to simplify replace the inner loop

indentation level `-2`, current: `3`

``` diff
function filterAcl(acl_hash) {
  let filtered_api_acl = {};
  Object.keys(acl_hash).forEach(api_version => {
    if (config.api_key.default_user_api_acl[api_version]) {
+     filtered_api_acl[api_version] = [];
      if (_.isArray(acl_hash[api_version])) {
-       for (let i = 0; i < acl_hash[api_version].length; i++) {
-         if (config.api_key.default_user_api_acl[api_version].indexOf(acl_hash[api_version][i]) !== -1) {
-           filtered_api_acl[api_version].push(acl_hash[api_version][i]);
-         }
-       }
        // 3
+       filtered_api_acl[api_version] = _.intersction(
+         config.api_key.default_user_api_acl[api_version],
+         acl_hash
+       )
      }
    }
  })
  return filtered_api_acl;
}
```

### 3. return in advance to reduce indentation level

indentation level `-1`, current `2`

``` javascript
function filterAcl(acl_hash) {
  let filtered_api_acl = {};
  Object.keys(acl_hash).forEach(api_version => {
    if (!config.api_key.default_user_api_acl[api_version]) {
      // 2
      return;
    }

    if (!_.isArray(acl_hash[api_version])) {
      return;
    }

    filtered_api_acl[api_version] = [];

    filtered_api_acl[api_version] = _.intersection(
      config.api_key.default_user_api_acl[api_version],
      acl_hash[api_version]
    );
  });

  return filtered_api_acl;
}
```

### 4. final step: use lodash to wrap everything

indentation level `-2`, current `0`

``` javascript
function filterAcl(acl_hash) {
  return _
    .chain(Object.keys(acl_hash))
    .filter(api_version => config.api_key.default_user_api_acl[api_version])
    .filter(api_version => Array.isArray(acl_hash[api_version]))
    .map(api_version => ([
      api_version,
      _.intersection(
        config.api_key.default_user_api_acl,
        acl_hash[api_version]
      )
    ]))
    .fromPairs()
    .value();
}
```

## Final Words

<div class="tip">
Use Lodash!
</div>

#### references:

* [lodash-doc](https://lodash.com/docs/4.17.2)
* [lazy-js-intro](http://philosopherdeveloper.com/posts/introducing-lazy-js.html)
* [for-in-stackoverflow](http://stackoverflow.com/questions/500504/why-is-using-for-in-with-array-iteration-a-bad-idea)
* [simple-rules-for-simple-code](https://laracasts.com/series/simple-rules-for-simpler-code)
