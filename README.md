# js

## syntax

Use consistent naming conventions.

- constructors must use `PascalCase`
- variables and methods must use `camelCase`
- privates should be `_camelCase` (preceeded by an underscore)
- consts shoud be `UPPER_CASE`

## wording

Use descriptive names.

to catch `this` into a variable :

- use `thisValue` if set as an argument
- use the instance descriptive name
(`User` class, `var user = this`)

Do not name variables with single letters.

```javascript
function User(options){
  var key
  for(key in options) {
    if(!options.hasOwnProperty(key)) {
      continue
    }
    this[key] = options[key]
  }
}

User.prototype.getName = function(){
  return this.name
}

User.prototype.setName = function(name){
  this.name = name
}

User.prototype.alertName = function(){
  var user = this
  setTimeout(function(){
    // yes, that is a silly example
    alert(user.getName())
  }, 1000)
}
```

## semicolon(-less)

This is not a troll, they are automatically inserted
so that you can remove this syntactic noise.

This is a language feature well interpreted everywhere.

Good uglifiers will parse them when necessary.

[» specificiation](http://es5.github.io/#x7.9)

## indentation

Indent with 2 spaces (never tabs).

## limits

Try to put yourself some limits for better maintainability :

- 80 chars per line
- 200 lines of code for a file

are some quite common ones.

If a function has  *long* arguments, break lines between each argument
for better readability :

```javascript
view.updateContents(
  document.querySelector(".js-MyBlock"),
  i18n.get("myKey"),
  function(){
    app.fire("block:translated")
  }
)
```

## module system

In order to separate your files, you should divide your script into modules.

If you concat your files in the end, do not use requirejs
but rather use browserify. This will make the codebase a little
lighter, and the syntax a little less confusing.

### synchronous

```javascript
var $ = require("jquery")
var BaseView = require("../baseView")

module.exports = BaseView.extend({
  events : {
    "click .js-TogglePanel" : "togglePanel"
  },
  togglePanel : function(eventObject){
    var target = eventObject.currentTarget
    var panel = document.getElementById(target.getAttribute("data-panel"))
    $(panel).toggleClass("Panel--visible")
  }
})
```

### asynchronous

If you use requirejs, do not place all your dependencies in
one single line, keep it clear :

```javascript
define(
  [
    "jquery",
    "../baseView"
  ],
  function($, BaseView){
    return BaseView.extend({
      events : {
        "click .js-TogglePanel" : "togglePanel"
      },
      togglePanel : function(eventObject){
        var target = eventObject.currentTarget
        var panel = document.getElementById(target.getAttribute("data-panel"))
        $(panel).toggleClass("Panel--visible")
      }
    })
  }
)
```

If you open-source a lib, just use the commonjs syntax. 
The browserify `standalone` option makes UMD builds. 

## loops

### arrays

Avoid `for(;;)` loops and use `while` instead.

This forces you to initialize your variables above and
take only the `condition` expression in the loop declaration.

Use `break` and `continue` statements to early-exit an iteration
or a loop.

```javascript
var index = -1
var length = array.length
var item

while(++index < length) {
  item = array[index]
  // …
}
```

### objects

Initialize your variables above.

use `break` and `continue` statements to early-exit an iteration
or a loop.

```javascript
var key
var item

for(key in object) {
  if(!object.hasOwnProperty(key)) {
    continue
  }
  item = object[key]
  // …
}
```

### labels

Labels are rarely used, but can be useful.

If you nest loops, you might want to exit an upper loop, use labels
for this :

```javascript
var index = -1
var length = array.length
var item
var key

arrayLoop:
while(++index < length) {
  item = array[index]
  for(key in validators) {
    if(!validators.hasOwnProperty(key)) {
      continue
    }
    if(!validators[key].test(item[key])) {
      break arrayLoop
    }
  }
}
```

## functions

### inside a block

Assign to a variable

The reason for this is that function declarations are made
available to the scope before the parent function execution.

```javascript
var transitionFn

if(supports("transition")) {
  transitionFn = function(){
    // …
  }
}
```

### outside of a block

declare the function

```javascript
function transitionFn(){
  // …
}
```

## variables

use multiple `var` statements.

```javascript
var foo = "foo"
var bar = "bar"
var baz = "baz"
```

## methods

do not mix getters and setters (oh hai jquery)

- getters should return a value
- setters should return `undefined`

## chaining

If you use chaining, make sure to indent to make it readable.

```javascript
gulp.src("foo")
  .pipe(bar())
  .pipe(gulp.dest("baz"))

request
  .then(function(item){
    model.extend(item)
    return model.isReady()
  })
  .then(function(){
    view.create(model)
  })
```

## DOM

### events

- use event delegation
- use `js-` prefixed classNames to prevent too much
mixing between style and behaviour.
- describe the method name in this className.

```javascript
var view = require("../lib/view")

module.exports = view.extend({
  events : [
    {
      type : "click",
      selector : ".js-TogglePanel",
      listener : "togglePanel"
    }
  ],
  togglePanel : function(eventObject){
    var target = eventObject.currentTarget
    var panel = target.querySelector(".js-Panel")
    panel.classList.toggle("Panel--visible")
  }
})
```

### templating

#### as strings

If you have to use an inline string template, use an array for
better presentation :

```javascript
function template(data){
  return [
    "<div class='Panel'>",
      "<h3 class='Panel-title'>",
        escapeHTML(data.title),
      "</h3>",
    "</div>"
  ].join("")
}
```

#### as nodes

- use document fragments if you have multiple elements to insert.

```javascript
function template(data){
  var fragment = document.createDocumentFragment()
  data.forEach(function(item){
    var panel = document.createElement("div")
    var title = document.createElement("h3")
    panel.className = "Panel"
    title.className = "Panel-title"
    title.appendChild(
      document.createTextNode(item.title)
    )
    panel.appendChild(title)
    fragment.appendChild(panel)
  })
  return fragment
}
```

## architecture

- separate data from view
- make your components communicate through global/local events for simple testing

## do not forget

- the `radix` argument in `parseInt`
- the `deep` argument in `Node.prototype.cloneNode`
- never to call a primitive constructor with `new` as it leads to
unexpected results (objects are all truthy)
- to test your code
