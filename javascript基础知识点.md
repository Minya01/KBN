

### 事件委托（event delegation)

对于具有绑定到许多元素的事件处理程序的Web应用程序来说，这非常棒，在DOM中动态创建和/或删除新元素。通过事件委托，事件绑定的数量可以通过将它们移动到一个共同的父元素而大大减少，动态创建新元素的代码可以与绑定它们的事件处理程序的逻辑分离。

事件委托的另一个好处是事件监听器使用的内存总量减少（因为事件绑定数量减少）。对于经常卸载的小页面（例如，用户经常导航到不同的页面）可能没有太大区别。但对于长期使用的应用程序来说，这可能很重要。当从DOM中移除的元素仍然声明内存（即它们泄漏）时，有一些真正难以追踪的情况，并且这种泄漏的内存通常与事件绑定有关。通过事件代表团，您可以自由地销毁子元素，而不会有任何忘记“解除”其事件监听器的风险（因为监听器位于祖先上）。这些类型的内存泄漏可以被包含（如果不能消除的话，这有时候很难做到，IE我正在看你）。



### 简述`JavaScript`中的`this`

1. 在调用函数时使用`new`关键字，函数内的`this`是一个全新的对象。
2. 如果`apply`、`call`或`bind`方法用于调用、创建一个函数，函数内的 this 就是作为参数传入这些方法的对象。
3. 当函数作为对象里的方法被调用时，函数内的`this`是调用该函数的对象。比如当`obj.method()`被调用时，函数内的 this 将绑定到`obj`对象。
4. 如果调用函数不符合上述规则，那么`this`的值指向全局对象（global object）。浏览器环境下`this`的值指向`window`对象，但是在严格模式下(`'use strict'`)，`this`的值为`undefined`。
5. 如果符合上述多个规则，则较高的规则（1 号最高，4 号最低）将决定`this`的值。
6. 如果该函数是 ES2015 中的箭头函数，将忽略上面的所有规则，`this`被设置为它被创建时的上下文。



### 解释原型继承（prototypal inheritance）的工作原理

所有 JS 对象都有一个`prototype`属性，指向它的原型对象。当试图访问一个对象的属性时，如果没有在该对象上找到，它还会搜寻该对象的原型，以及该对象的原型的原型，依次层层向上搜索，直到找到一个名字匹配的属性或到达原型链的末尾。这种行为是在模拟经典的继承，[但是与其说是继承，不如说是委托（delegation）](https://davidwalsh.name/javascript-objects)。



### AMD 和 CommonJS 的了解

它们都是实现模块体系的方式，直到 ES2015 出现之前，JavaScript 一直没有模块体系。CommonJS 是同步的，而 AMD（Asynchronous Module Definition）从全称中可以明显看出是异步的。CommonJS 的设计是为服务器端开发考虑的，而 AMD 支持异步加载模块，更适合浏览器。

我发现 AMD 的语法非常冗长，CommonJS 更接近其他语言 import 声明语句的用法习惯。大多数情况下，我认为 AMD 没有使用的必要，因为如果把所有 JavaScript 都捆绑进一个文件中，将无法得到异步加载的好处。此外，CommonJS 语法上更接近 Node 编写模块的风格，在前后端都使用 JavaScript 开发之间进行切换时，语境的切换开销较小。

我很高兴看到 ES2015 的模块加载方案同时支持同步和异步，我们终于可以只使用一种方案了。虽然它尚未在浏览器和 Node 中完全推出，但是我们可以使用代码转换工具进行转换。



### 下面代码为什么不能用作 IIFE：`function foo(){ }();`，需要作出哪些修改才能使其成为 IIFE？

IIFE（Immediately Invoked Function Expressions）代表立即执行函数。 JavaScript 解析器将 `function foo(){ }();`解析成`function foo(){ }`和`();`。其中，前者是函数声明；后者（一对括号）是试图调用一个函数，却没有指定名称，因此它会抛出`Uncaught SyntaxError: Unexpected token )`的错误。

修改方法是：再添加一对括号，形式上有两种：`(function foo(){ })()`和`(function foo(){ }())`。以上函数不会暴露到全局作用域，如果不需要在函数内部引用自身，可以省略函数的名称。

你可能会用到 `void` 操作符：`void function foo(){ }();`。但是，这种做法是有问题的。表达式的值是`undefined`，所以如果你的 IIFE 有返回值，不要用这种做法。例如：

```
// Don't add JS syntax to this code block to prevent Prettier from formatting it.
const foo = void function bar() { return 'foo'; }();

console.log(foo); // undefined

```



### `null`、`undefined`和未声明变量之间有什么区别？如何检查判断这些状态值？

当你没有提前使用`var`、`let`或`const`声明变量，就为一个变量赋值时，该变量是未声明变量（undeclared variables）。未声明变量会脱离当前作用域，成为全局作用域下定义的变量。在严格模式下，给未声明的变量赋值，会抛出`ReferenceError`错误。和使用全局变量一样，使用未声明变量也是非常不好的做法，应当尽可能避免。要检查判断它们，需要将用到它们的代码放在`try`/`catch`语句中。

```
function foo() {
  x = 1; // 在严格模式下，抛出 ReferenceError 错误
}

foo();
console.log(x); // 1
```

当一个变量已经声明，但没有赋值时，该变量的值是`undefined`。如果一个函数的执行结果被赋值给一个变量，但是这个函数却没有返回任何值，那么该变量的值是`undefined`。要检查它，需要使用严格相等（`===`）；或者使用`typeof`，它会返回`'undefined'`字符串。请注意，不能使用非严格相等（`==`）来检查，因为如果变量值为`null`，使用非严格相等也会返回`true`。

```
var foo;
console.log(foo); // undefined
console.log(foo === undefined); // true
console.log(typeof foo === 'undefined'); // true

console.log(foo == null); // true. 错误，不要使用非严格相等！

function bar() {}
var baz = bar();
console.log(baz); // undefined
```

`null`只能被显式赋值给变量。它表示`空值`，与被显式赋值 `undefined` 的意义不同。要检查判断`null`值，需要使用严格相等运算符。请注意，和前面一样，不能使用非严格相等（`==`）来检查，因为如果变量值为`undefined`，使用非严格相等也会返回`true`。

```
var foo = null;
console.log(foo === null); // true

console.log(foo == undefined); // true. 错误，不要使用非严格相等！
```

作为一种个人习惯，我从不使用未声明变量。如果定义了暂时没有用到的变量，我会在声明后明确地给它们赋值为`null`。



### 什么是闭包（closure），为什么使用闭包？

闭包是函数和声明该函数的词法环境的组合。词法作用域中使用的域，是变量在代码中声明的位置所决定的。闭包是即使被外部函数返回，依然可以访问到外部（封闭）函数作用域的函数。

**为什么使用闭包？**

- 利用闭包实现数据私有化或模拟私有方法。这个方式也称为[模块模式（module pattern）](https://addyosmani.com/resources/essentialjsdesignpatterns/book/#modulepatternjavascript)。
- [部分参数函数（partial applications）柯里化（currying）](https://medium.com/javascript-scene/curry-or-partial-application-8150044c78b8#.l4b6l1i3x).



#### 闭包

1.闭包允许从内部函数访问外部函数的作用于。

要使用闭包，只需在另一个函数内定义一个函数并将其暴露。要公开一个函数，将其返回或传递给另一个函数。

即使在外部函数返回后，内部函数也可以访问外部函数作用域中的变量。



2.除其他外，封闭通常用于为对象提供数据隐私。数据隐私是帮助我们编程接口而不是实现的重要属性。

在JavaScript中，闭包是用于启用数据隐私的主要机制。当您使用闭包实现数据隐私时，封闭变量只在包含（外部）函数的范围内。除了通过对象的**特权方法**外，您无法从外部范围获取数据。在JavaScript中，在闭包范围内定义的任何公开方法都是有特权的。例如：

```js
const getSecret = (secret) => {
  return {
    get: () => secret
  };
};

test('Closure for object privacy.', assert => {
  const msg = '.get() should have access to the closure.';
  const expected = 1;
  const obj = getSecret(1);

  const actual = obj.get();

  try {
    assert.ok(secret, 'This throws an error.');
  } catch (e) {
    assert.ok(true, `The secret var is only available
      to privileged methods.`);
  }

  assert.equal(actual, expected, msg);
  assert.end();
});
```



### 请说明`.forEach`循环和`.map()`循环的主要区别，它们分别在什么情况下使用？

为了理解两者的区别，我们看看它们分别是做什么的。

**forEach**

- 遍历数组中的元素。
- 为每个元素执行回调。
- 无返回值。

```
const a = [1, 2, 3];
const doubled = a.forEach((num, index) => {
  // 执行与 num、index 相关的代码
});

// doubled = undefined
```

**map**

- 遍历数组中的元素
- 通过对每个元素调用函数，将每个元素“映射（map）”到一个新元素，从而创建一个新数组。

```
const a = [1, 2, 3];
const doubled = a.map(num => {
  return num * 2;
});

// doubled = [2, 4, 6]
```

`.forEach`和`.map()`的主要区别在于`.map()`返回一个新的数组。如果你想得到一个结果，但不想改变原始数组，用`.map()`。如果你只需要在数组上做迭代修改，用`forEach`。



### 匿名函数的典型应用场景

匿名函数可以在 IIFE 中使用，来封装局部作用域内的代码，以便其声明的变量不会暴露到全局作用域。

```
(function() {
  // 一些代码。
})();
```

匿名函数可以作为只用一次，不需要在其他地方使用的回调函数。当处理函数在调用它们的程序内部被定义时，代码具有更好地自闭性和可读性，可以省去寻找该处理函数的函数体位置的麻烦。

```
setTimeout(function() {
  console.log('Hello world!');
}, 1000);
```

匿名函数可以用于函数式编程或 Lodash（类似于回调函数）。

```
const arr = [1, 2, 3];
const double = arr.map(function(el) {
  return el * 2;
});
console.log(double); // [2, 4, 6]
```



### 宿主对象（host objects）和原生对象（native objects）的区别是什么？

原生对象是由 ECMAScript 规范定义的 JavaScript 内置对象，比如`String`、`Math`、`RegExp`、`Object`、`Function`等等。

宿主对象是由运行时环境（浏览器或 Node）提供，比如`window`、`XMLHTTPRequest`等等。



### 下列语句有什么区别：`function Person(){}`、`var person = Person()`和`var person = new Person()`？

这个问题问得很含糊。我猜这是在考察 JavaScript 中的构造函数（constructor）。从技术上讲，`function Person(){}`只是一个普通的函数声明。使用 PascalCase 方式命名函数作为构造函数，是一个惯例。

`var person = Person()`将`Person`以普通函数调用，而不是构造函数。如果该函数是用作构造函数的，那么这种调用方式是一种常见错误。通常情况下，构造函数不会返回任何东西，因此，像普通函数一样调用构造函数，只会返回`undefined`赋给用作实例的变量。

`var person = new Person()`使用`new`操作符，创建`Person`对象的实例，该实例继承自`Person.prototype`。另外一种方式是使用`Object.create`，例如：Object.create(Person.prototype)`。

```
function Person(name) {
  this.name = name;
}

var person = Person('John');
console.log(person); // undefined
console.log(person.name); // Uncaught TypeError: Cannot read property 'name' of undefined

var person = new Person('John');
console.log(person); // Person { name: "John" }
console.log(person.name); // "john"
```



### `.call`和`.apply`有什么区别？

`.call`和`.apply`都用于调用函数，第一个参数将用作函数内 this 的值。然而，`.call`接受逗号分隔的参数作为后面的参数，而`.apply`接受一个参数数组作为后面的参数。一个简单的记忆方法是，从`call`中的 C 联想到逗号分隔（comma-separated），从`apply`中的 A 联想到数组（array）。

```
function add(a, b) {
  return a + b;
}

console.log(add.call(null, 1, 2)); // 3
console.log(add.apply(null, [1, 2])); // 3
```



### 请说明`Function.prototype.bind`的用法。

摘自[MDN](https://developer.mozilla.org/en/docs/Web/JavaScript/Reference/Global_objects/Function/bind)：

> `bind()`方法创建一个新的函数, 当被调用时，将其 this 关键字设置为提供的值，在调用新函数时，在任何提供之前提供一个给定的参数序列。

根据我的经验，将`this`的值绑定到想要传递给其他函数的类的方法中是非常有用的。在 React 组件中经常这样做。



### 什么时候会用到`document.write()`？

`document.write()`用来将一串文本写入由`document.open()`打开的文档流中。当页面加载后执行`document.write()`时，它将调用`document.open`，会清除整个文档（`<head>`和`<body>`会被移除！），并将文档内容替换成给定的字符串参数。因此它通常被认为是危险的并且容易被误用。

网上有一些答案，解释了`document.write()`被用于分析代码中，或者[当你想包含只有在启用了 JavaScript 的情况下才能工作的样式](https://www.quirksmode.org/blog/archives/2005/06/three_javascrip_1.html)。它甚至在 HTML5 样板代码中用于[并行加载脚本并保持执行顺序](https://github.com/paulirish/html5-boilerplate/wiki/Script-Loading-Techniques#documentwrite-script-tag)！但是，我怀疑这些使用原因是过时的，现在可以在不使用`document.write()`的情况下实现。如果我的观点有错，请纠正我。



### 请尽可能详细地解释 Ajax。

Ajax（asynchronous JavaScript and XML）是使用客户端上的许多 Web 技术，创建异步 Web 应用的一种 Web 开发技术。借助 Ajax，Web 应用可以异步（在后台）向服务器发送数据和从服务器检索数据，而不会干扰现有页面的显示和行为。通过将数据交换层与表示层分离，Ajax 允许网页和扩展 Web 应用程序动态更改内容，而无需重新加载整个页面。实际上，现在通常将 XML 替换为 JSON，因为 JavaScript 对 JSON 有原生支持优势。

`XMLHttpRequest` API 经常用于异步通信。此外还有最近流行的`fetch` API。



### 使用 Ajax 的优缺点分别是什么？

**优点**

- 交互性更好。来自服务器的新内容可以动态更改，无需重新加载整个页面。
- 减少与服务器的连接，因为脚本和样式只需要被请求一次。
- 状态可以维护在一个页面上。JavaScript 变量和 DOM 状态将得到保持，因为主容器页面未被重新加载。
- 基本上包括大部分 SPA 的优点。

**缺点**

- 动态网页很难收藏。
- 如果 JavaScript 已在浏览器中被禁用，则不起作用。
- 有些网络爬虫不执行 JavaScript，也不会看到 JavaScript 加载的内容。
- 基本上包括大部分 SPA 的缺点。



### 你使用过 JavaScript 模板吗？用过什么相关的库？

使用过。Handlebars、Underscore、Lodash、AngularJS 和 JSX。我不喜欢 AngularJS 中的模板，因为它在指令中大量使用了字符串，并且书写错误会被忽略。JSX 是我的新宠，因为它更接近 JavaScript，几乎没有什么学习成本。现在，可以使用 ES2015 模板字符串快速创建模板，而不需依赖第三方代码。

```
const template = `<div>My name is: ${name}</div>`;
```

但是，请注意上述方法中可能存在的 XSS，因为内容不会被转义，与模板库不同。



### 解释变量提升（hosting）。

变量提升（hoisting）是用于解释代码中变量声明行为的术语。使用`var`关键字声明或初始化的变量，会将声明语句“提升”到当前作用域的顶部。 但是，只有声明才会触发提升，赋值语句（如果有的话）将保持原样。我们用几个例子来解释一下。

```
// 用 var 声明得到提升
console.log(foo); // undefined
var foo = 1;
console.log(foo); // 1

// 用 let/const 声明不会提升
console.log(bar); // ReferenceError: bar is not defined
let bar = 2;
console.log(bar); // 2
```

函数声明会使函数体提升，但函数表达式（以声明变量的形式书写）只有变量声明会被提升。

```
// 函数声明
console.log(foo); // [Function: foo]
foo(); // 'FOOOOO'
function foo() {
  console.log('FOOOOO');
}
console.log(foo); // [Function: foo]

// 函数表达式
console.log(bar); // undefined
bar(); // Uncaught TypeError: bar is not a function
var bar = function() {
  console.log('BARRRR');
};
console.log(bar); // [Function: bar]
```



### 描述事件冒泡。

当一个事件在 DOM 元素上触发时，如果有事件监听器，它将尝试处理该事件，然后事件冒泡到其父级元素，并发生同样的事情。最后直到事件到达祖先元素。事件冒泡是实现事件委托的原理（event delegation）。



### “attribute” 和 “property” 之间有什么区别？

“Attribute” 是在 HTML 中定义的，而 “property” 是在 DOM 上定义的。为了说明区别，假设我们在 HTML 中有一个文本框：`<input type="text" value="Hello">`。

```
const input = document.querySelector('input');
console.log(input.getAttribute('value')); // Hello
console.log(input.value); // Hello
```

但是在文本框中键入“ World!”后:

```
console.log(input.getAttribute('value')); // Hello
console.log(input.value); // Hello World!
```



### 为什么扩展 JavaScript 内置对象是不好的做法？

扩展 JavaScript 内置（原生）对象意味着将属性或方法添加到其`prototype`中。虽然听起来很不错，但事实上这样做很危险。想象一下，你的代码使用了一些库，它们通过添加相同的 contains 方法来扩展`Array.prototype`，如果这两个方法的行为不相同，那么这些实现将会相互覆盖，你的代码将不能正常运行。

扩展内置对象的唯一使用场景是创建 polyfill，本质上为老版本浏览器缺失的方法提供自己的实现，该方法是由 JavaScript 规范定义的。



### document 中的`load`事件和`DOMContentLoaded`事件之间的区别是什么？

当初始的 HTML 文档被完全加载和解析完成之后，`DOMContentLoaded`事件被触发，而无需等待样式表、图像和子框架的完成加载。

`window`的`load`事件仅在 DOM 和所有相关资源全部完成加载后才会触发。



### `==`和`===`的区别是什么？

`==`是抽象相等运算符，而`===`是严格相等运算符。`==`运算符是在进行必要的类型转换后，再比较。`===`运算符不会进行类型转换，所以如果两个值不是相同的类型，会直接返回`false`。使用`==`时，可能发生一些特别的事情，例如：

```
1 == '1'; // true
1 == [1]; // true
1 == true; // true
0 == ''; // true
0 == '0'; // true
0 == false; // true
```

我的建议是从不使用`==`运算符，除了方便与`null`或`undefined`比较时，`a == null`如果`a`为`null`或`undefined`将返回`true`。

```
var a = null;
console.log(a == null); // true
console.log(a == undefined); // true
```



### 请解释关于 JavaScript 的同源策略。

同源策略可防止 JavaScript 发起跨域请求。源被定义为 URI、主机名和端口号的组合。此策略可防止页面上的恶意脚本通过该页面的文档对象模型，访问另一个网页上的敏感数据。



### 请使下面的语句生效：

```
duplicate([1, 2, 3, 4, 5]); // [1,2,3,4,5,1,2,3,4,5]
function duplicate(arr) {
  return arr.concat(arr);
}

duplicate([1, 2, 3, 4, 5]); // [1,2,3,4,5,1,2,3,4,5]
```



### 什么是`"use strict";`？使用它有什么优缺点？

'use strict' 是用于对整个脚本或单个函数启用严格模式的语句。严格模式是可选择的一个限制 JavaScript 的变体一种方式 。

**优点：**

- 无法再意外创建全局变量。
- 会使引起静默失败（silently fail，即：不报错也没有任何效果）的赋值操抛出异常。
- 试图删除不可删除的属性时会抛出异常（之前这种操作不会产生任何效果）。
- 要求函数的参数名唯一。
- 全局作用域下，`this`的值为`undefined`。
- 捕获了一些常见的编码错误，并抛出异常。
- 禁用令人困惑或欠佳的功能。

**缺点：**

- 缺失许多开发人员已经习惯的功能。
- 无法访问`function.caller`和`function.arguments`。
- 以不同严格模式编写的脚本合并后可能导致问题。

总的来说，我认为利大于弊，我从来不使用严格模式禁用的功能，因此我推荐使用严格模式。



### 为什么不要使用全局作用域？

每个脚本都可以访问全局作用域，如果人人都使用全局命名空间来定义自己的变量，肯定会发生冲突。使用模块模式（IIFE）将变量封装在本地命名空间中。



### 为什么要使用`load`事件？这个事件有什么缺点吗？你知道一些代替方案吗，为什么使用它们？

在文档装载完成后会触发`load`事件。此时，在文档中的所有对象都在 DOM 中，所有图像、脚本、链接和子框架都完成了加载。

DOM 事件`DOMContentLoaded`将在页面的 DOM 构建完成后触发，但不要等待其他资源完成加载。如果在初始化之前不需要装入整个页面，这个事件是使用首选。

TODO:



### 请解释单页应用是什么，如何使其对 SEO 友好。

以下摘自 [Grab Front End Guide](https://github.com/grab/front-end-guide)，碰巧的是，这正是我自己写的！

现如今，Web 开发人员将他们构建的产品称为 Web 应用，而不是网站。虽然这两个术语之间没有严格的区别，但网络应用往往具有高度的交互性和动态性，允许用户执行操作并接收他们的操作响应。在过去，浏览器从服务器接收 HTML 并渲染。当用户导航到其它 URL 时，需要整页刷新，服务器会为新页面发送新的 HTML。这被称为服务器端渲染。

然而，在现代的 SPA 中，客户端渲染取而代之。浏览器从服务器加载初始页面、整个应用程序所需的脚本（框架、库、应用代码）和样式表。当用户导航到其他页面时，不会触发页面刷新。该页面的 URL 通过 [HTML5 History API](https://developer.mozilla.org/en-US/docs/Web/API/History_API) 进行更新。浏览器通过 [AJAX](https://developer.mozilla.org/en-US/docs/AJAX/Getting_Started) 请求向服务器检索新页面所需的数据（通常采用 JSON 格式）。然后，SPA 通过 JavaScript 来动态更新页面，这些 JavaScript 在初始页面加载时已经下载。这种模式类似于原生移动应用的工作方式。

**好处：**

- 用户感知响应更快，用户切换页面时，不再看到因页面刷新而导致的白屏。
- 对服务器进行的 HTTP 请求减少，因为对于每个页面加载，不必再次下载相同的资源。
- 客户端和服务器之间的关注点分离。可以为不同平台（例如手机、聊天机器人、智能手表）建立新的客户端，而无需修改服务器代码。只要 API 没有修改，可以单独修改客户端和服务器上的代码。

**坏处：**

- 由于加载了多个页面所需的框架、应用代码和资源，导致初始页面加载时间较长。
- 服务器还需要进行额外的工作，需要将所有请求路由配置到单个入口点，然后由客户端接管路由。
- SPA 依赖于 JavaScript 来呈现内容，但并非所有搜索引擎都在抓取过程中执行 JavaScript，他们可能会在你的页面上看到空的内容。这无意中损害了应用的搜索引擎优化（SEO）。然而，当你构建应用时，大多数情况下，搜索引擎优化并不是最重要的因素，因为并非所有内容都需要通过搜索引擎进行索引。为了解决这个问题，可以在服务器端渲染你的应用，或者使用诸如 [Prerender](https://prerender.io/) 的服务来“在浏览器中呈现你的 javascript，保存静态 HTML，并将其返回给爬虫”。



### 你对 Promises 及其 polyfill 的掌握程度如何？

掌握它的工作原理。`Promise`是一个可能在未来某个时间产生结果的对象：操作成功的结果或失败的原因（例如发生网络错误）。 `Promise`可能处于以下三种状态之一：fulfilled、rejected 或 pending。 用户可以对`Promise`添加回调函数来处理操作成功的结果或失败的原因。

一些常见的 polyfill 是`$.deferred`、Q 和 Bluebird，但不是所有的 polyfill 都符合规范。ES2015 支持 Promises，现在通常不需要使用 polyfills。



### `Promise`代替回调函数有什么优缺点？

**优点：**

- 避免可读性极差的回调地狱。
- 使用`.then()`编写的顺序异步代码，既简单又易读。
- 使用`Promise.all()`编写并行异步代码变得很容易。

**缺点：**

- 轻微地增加了代码的复杂度（这点存在争议）。
- 在不支持 ES2015 的旧版浏览器中，需要引入 polyfill 才能使用。



### 用转译成 JavaScript 的语言写 JavaScript 有什么优缺点？

Some examples of languages that compile to JavaScript include CoffeeScript, Elm, ClojureScript, PureScript and TypeScript. 这些是转译成 JavaScript 的语言，包括 CoffeeScript、Elm、ClojureScript、PureScript 和 TypeScript。

**优点：**

- 修复了 JavaScript 中的一些长期问题，并摒弃了 JavaScript 不好的做法。
- 在 JavaScript 的基础上提供一些语法糖，使我们能够编写更短的代码，我认为 ES5 缺乏语法糖的支持，但 ES2015 非常好。
- 对于需要长时间维护的大型项目，静态类型非常好用（针对 TypeScript）。

**缺点：**

- 由于浏览器只运行 JavaScript，所以需要构建、编译过程，在将代码提供给浏览器之前，需要将代码转译为 JavaScript。
- 如果 source map 不能很好地映射到预编译的源代码，调试会很痛苦。
- 大多数开发人员不熟悉这些语言，需要学习它。如果将其用于项目，会增加团队成本。
- 社区比较小（取决于语言），这意味着资源、教程、图书和工具难以找到。
- 可能缺乏 IDE（编辑器）的支持。
- 这些语言将始终落后于最新的 JavaScript 标准。
- 开发人员应该清楚代码正在被编译到什么地方——因为这是实际运行的内容，是最重要的。

实际上，ES2015 已经大大改进了 JavaScript，编写体验很好。我现在还没有真正看到对 CoffeeScript 的需求。



### 你使用什么工具和技巧调试 JavaScript 代码？

- React 和 Redux
  - [React Devtools](https://github.com/facebook/react-devtools)
  - [Redux Devtools](https://github.com/gaearon/redux-devtools)
- Vue
  - [Vue Devtools](https://github.com/vuejs/vue-devtools)
- JavaScript
  - [Chrome Devtools](https://hackernoon.com/twelve-fancy-chrome-devtools-tips-dc1e39d10d9d)
  - `debugger`声明
  - 使用万金油`console.log`进行调试



### 你使用什么语句遍历对象的属性和数组的元素？

**对象：**

- `for`循环：`for (var property in obj) { console.log(property); }`。但是，这还会遍历到它的继承属性，在使用之前，你需要加入`obj.hasOwnProperty(property)`检查。
- `Object.keys()`：`Object.keys(obj).forEach(function (property) { ... })`。`Object.keys()`方法会返回一个由一个给定对象的自身可枚举属性组成的数组。
- `Object.getOwnPropertyNames()`：`Object.getOwnPropertyNames(obj).forEach(function (property) { ... })`。`Object.getOwnPropertyNames()`方法返回一个由指定对象的所有自身属性的属性名（包括不可枚举属性但不包括 Symbol 值作为名称的属性）组成的数组。

**数组：**

- `for` loops：`for (var i = 0; i < arr.length; i++)`。这里的常见错误是`var`是函数作用域而不是块级作用域，大多数时候你想要迭代变量在块级作用域中。ES2015 引入了具有块级作用域的`let`，建议使用它。所以就变成了：`for (let i = 0; i < arr.length; i++)`。
- `forEach`：`arr.forEach(function (el, index) { ... })`。这个语句结构有时会更精简，因为如果你所需要的只是数组元素，你不必使用`index`。还有`every`和`some`方法可以让你提前终止遍历。

大多数情况下，我更喜欢`.forEach`方法，但这取决于你想要做什么。`for`循环有更强的灵活性，比如使用`break`提前终止循环，或者递增步数大于一。



### 请解释可变对象和不可变对象之间的区别。

- 什么是 JavaScript 中的不可变对象的例子？
- 不变性有什么优点和缺点？
- 你如何在自己的代码中实现不变性？

**可变对象** 在创建之后是可以被改变的。

**不可变对象** 在创建之后是不可以被改变的。

1. 在 `JavaScript` 中，`string` 和 `number` 从设计之初就是不可变(Immutable)。
2. **不可变** 其实是保持一个对象状态不变，这样做的好处是使得开发更加简单，可回溯，测试友好，减少了任何可能的副作用。但是，每当你想添加点东西到一个不可变(Immutable)对象里时，它一定是先拷贝已存在的值到新实例里，然后再给新实例添加内容，最后返回新实例。相比可变对象，这势必会有更多内存、计算量消耗。
3. 比如：构造一个纯函数

```
const student1 = {
  school: 'Baidu',
  name: 'HOU Ce',
  birthdate: '1995-12-15',
};

const changeStudent = (student, newName, newBday) => {
  return {
    ...student, // 使用解构
    name: newName, // 覆盖name属性
    birthdate: newBday, // 覆盖birthdate属性
  };
};

const student2 = changeStudent(student1, 'YAN Haijing', '1990-11-10');

// both students will have the name properties
console.log(student1, student2);
// Object {school: "Baidu", name: "HOU Ce", birthdate: "1995-12-15"}
// Object {school: "Baidu", name: "YAN Haijing", birthdate: "1990-11-10"}
```



### 请解释同步和异步函数之间的区别。

同步函数阻塞，而异步函数不阻塞。在同步函数中，语句完成后，下一句才执行。在这种情况下，程序可以按照语句的顺序进行精确评估，如果其中一个语句需要很长时间，程序的执行会停滞很长时间。

异步函数通常接受回调作为参数，在调用异步函数后立即继续执行下一行。回调函数仅在异步操作完成且调用堆栈为空时调用。诸如从 Web 服务器加载数据或查询数据库等重负载操作应该异步完成，以便主线程可以继续执行其他操作，而不会出现一直阻塞，直到费时操作完成的情况（在浏览器中，界面会卡住）。

