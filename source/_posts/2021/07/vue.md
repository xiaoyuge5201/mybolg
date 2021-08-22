---
title: vue学习
tags: vue
categories:
- 前端
- Vue
comments: false
translate_title: vue-learning
date: 2021-07-23 11:40:44
---
Object.freeze()，这会阻止修改现有的 property，也意味着响应系统无法再追踪变化。
### 1. export
---
用于规定模块的对外接口，export输出变量和方法、类

-   变量

    ```javascript
    // profile.js
    export var firstName = 'Michael';
    export var lastName = 'Jackson';
    export var year = 1958;
    
    //简写--优先使用
    export {firstName, lastName, year}
    ```

-   方法

    ```javascript
    //如果想为输入的变量重新命名， 可以使用AS 关键字重新命名
    import { buildMenus as buildMenus} from '@/api/menu';
    //import命令接受一对大括号，里面指定要从其他模块导入的变量名。大括号里面的变量名，必须与被导入模块（profile.js）对外接口的名称相同
    ```

### 2. export default

为模块指定默认输出， 使用import命令的时候，用户需要知道所要加载的变量名和函数名，否则无法加载；了解模块有哪些方法和属性比较麻烦，使用export default命令，为模块指定默认输出

```javascript
// export-default.js
export default function () {
  console.log('foo');
}
```

上面代码是一个模块文件export-default.js。默认输出1个函数；

与export命令的区别：其他模块加载该模块是，import命令可以为该匿名函数指定任意名字

```javascript
// import-default.js
import customName from './export-default';
customName(); // 'foo'
```

上面代码的`import`命令，可以用任意名称指向`export-default.js`输出的方法，这时就不需要知道原模块输出的函数名。需要注意的是，这时`import`命令后面，不使用大括号。

本质上，`export default`就是输出一个叫做`default`的变量或方法，然后系统允许你为它取任意名字。所以，下面的写法是有效的。

```javascript
// modules.js
function add(x, y) {
  return x * y;
}
export {add as default};
// 等同于
// export default add;

// app.js
import { default as foo } from 'modules';
// 等同于
// import foo from 'modules';
```

正是因为`export default`命令其实只是输出一个叫做`default`的变量，所以它后面不能跟变量声明语句。

**总结：**

-   export命令对外接口是有名称的且`import`命令从模块导入的变量名与被导入模块对外接口的名称相同，而export default命令对外输出的变量名可以是任意的，这时`import`命令后面，不使用大括号。

-   `export default`命令用于指定模块的默认输出。显然，一个模块只能有一个默认输出，因此`export default`命令只能使用一次。所以，import命令后面才不用加大括号，因为只可能唯一对应`export default`命令。

```javascript
//menu.js
//get请求获取所有的菜单信息
export function buildMenus() {
  return request({
    url: 'api/menus/build',
    method: 'get'
  })
}
//post 请求保存数据
export function add(data) {
  return request({
    url: 'api/menus',
    method: 'post',
    data
  })
}
//delete 请求删除数据
export function del(id) {
  return request({
    url: 'api/menus/' + id,
    method: 'delete'
  })
}
//put请求修改数据
export function edit(data) {
  return request({
    url: 'api/menus',
    method: 'put',
    data
  })
}

//app.vue
import { buildMenus } from '@/api/menu';
```

### 3. Const、var、let

 ES5 中作用域有：全局作用域、函数作用域。没有块作用域的概念。

 ES6 中新增了块级作用域。块作用域由 { } 包括，if语句和 for语句里面的{ }也属于块作用域

```javascript
{
  var a = 1;
  console.log(a); // 1
}
console.log(a); // 1
// 通过var定义的变量可以跨块作用域访问到。

(function A() {
  var b = 2;
  console.log(b); // 2
})();
// console.log(b); // 报错，
// 可见，通过var定义的变量不能跨函数作用域访问到

if(true) {
  var c = 3;
}
console.log(c); // 3
for(var i = 0; i < 4; i ++) {
  var d = 5;
};
console.log(i); // 4   (循环结束i已经是4，所以此处i为4)
console.log(d); // 5
// if语句和for语句中用var定义的变量可以在外面访问到，
// 可见，if语句和for语句属于块作用域，不属于函数作用域
```

三者的区别：

1.  var定义的变量，没有块的概念，可以跨块访问, 不能跨函数访问。
2.  let定义的变量，只能在块作用域里访问，不能跨块访问，也不能跨函数访问。
3.  const用来定义常量，使用时必须初始化(即必须赋值)，只能在块作用域里访问，而且不能修改。

```javascript
// 块作用域
{
  var a = 1;
  let b = 2;
  const c = 3;
  // c = 4; // 报错
  var aa;
  let bb;
  // const cc; // 报错
  console.log(a); // 1
  console.log(b); // 2
  console.log(c); // 3
  console.log(aa); // undefined
  console.log(bb); // undefined
}
console.log(a); // 1
// console.log(b); // 报错
// console.log(c); // 报错

// 函数作用域
(function A() {
  var d = 5;
  let e = 6;
  const f = 7;
  console.log(d); // 5
  console.log(e); // 6  
  console.log(f); // 7 
})();
// console.log(d); // 报错
// console.log(e); // 报错
// console.log(f); // 报错

```

注意：**const定义的对象属性是否可以改变**

```javascript
const person = {
  name : 'jiuke',
  sex : '男'
}
person.name = 'test'
console.log(person.name)//person对象的name属性确实被修改了
```

因为对象是引用类型的，person中保存的仅是对象的指针，这就意味着，const仅保证指针不发生改变，修改对象的属性不会改变对象的指针，所以是被允许的。也就是说const定义的引用类型只要指针不发生改变，其他的不论如何改变都是允许的。

然后我们试着修改一下指针，让person指向一个新对象，果然报错

```javascript
const person = {
   name : 'jiuke',
   sex : '男'
}
person = {
   name : 'test',
   sex : '男'
}
//报错
```

### 4. promise

promise用途：异步编程的一种解决方案。

优点：比传统的解决方案——回调函数和事件——更合理和更强大。

三种状态：`pending`（进行中）、`fulfilled`（已成功）和`rejected`（已失败）。

```javascript
//基本用法：
const promise = new Promise(function(resolve, reject) {
    resolve(value);//表示异步操作成功
    reject(error);//表示异步操作失败
});

//promise常用的几个方法
//1. 异步状态为成功时调用第一个函数，为失败时调用第二个函数。then方法的第二个参数可选。
promise.then(value => {},error => {});

//2. 异步状态为失败时调用。
promise.catch(error => {});

//3. promise异步状态为失败时或then方法中抛出错误都会执行catch方法。
promise.then(value => {},error => {}).catch(error => {});

//4. 不管状态如何都会执行的操作。
promise.finally(() => {});
```

### 5. 生命周期

<img src="https://cn.vuejs.org/images/lifecycle.png" alt="img" style="zoom:50%;" />

### 6. 模版语法

#### v-once

执行一次性插值，当数据变化的时候，该内容不会更新；可能会影响该节点其他的数据绑定

```html
<span v-once>这个将不会改变: {{ msg }}</span>
```

#### v-html

双大括号会将数据解释为普通文本，而非 HTML 代码。为了输出真正的 HTML，你需要使用v-html;

```html
var rawHtml = "<span>这是个使用v-htmls</span>"

<p>Using mustaches: {{ rawHtml }}</p>
<p>Using v-html directive: <span v-html="rawHtml"></span></p>
```

#### [Attribute](https://cn.vuejs.org/v2/guide/syntax.html#Attribute)

Mustache ({}) 语法不能作用在 HTML attribute 上，遇到这种情况应该使用 [`v-bind` 指令](https://cn.vuejs.org/v2/api/#v-bind)：

```html
<div v-bind:id="dynamicId"></div>


//isButtonDisabled 的值是 null、undefined 或 false，则 disabled attribute 甚至不会被包含在渲染出来的 <button> 元素中
<button v-bind:disabled="isButtonDisabled">Button</button>
```

#### 三元表达式

```html
{{ number + 1 }}

{{ ok ? 'YES' : 'NO' }}

{{ message.split('').reverse().join('') }}

<div v-bind:id="'list-' + id"></div>

//这些表达式会在所属 Vue 实例的数据作用域下作为 JavaScript 被解析。有个限制就是，每个绑定都只能包含单个表达式，所以下面的例子都不会生效。
<!-- 这是语句，不是表达式 -->
{{ var a = 1 }}

<!-- 流控制也不会生效，请使用三元表达式 -->
{{ if (ok) { return message } }}
```

### 7. 指令Directives

指令 (Directives) 是带有 `v-` 前缀的特殊 attribute。指令 attribute 的值预期是**单个 JavaScript 表达式** (`v-for` 是例外情况，稍后我们再讨论)。指令的职责是，当表达式的值改变时，将其产生的连带影响，响应式地作用于 DOM。

```html
//v-if 指令将根据表达式 seen 的值的真假来插入/移除 <p> 元素。
<p v-if="seen">现在你看到我了</p>
```

#### 参数

一些指令能够接收一个“参数”，在指令名称之后以冒号表示。例如，`v-bind` 指令可以用于响应式地更新 HTML attribute

```html
//href 是参数，告知 v-bind 指令将该元素的 href attribute 与表达式 url 的值绑定
<a v-bind:href="url">...</a>

<a v-on:click="doSomething">...</a>
```

#### 动态参数

 2.6.0 开始，可以用方括号括起来的 JavaScript 表达式作为一个指令的参数

```html
<a v-bind:[attributeName] = "url"></a>
<!--
	这里的attributeName会被作为一个javascript表达式进行动态赋值，求得的值会作为最终的参数来使用
如果VUE实例有一个data. property. attributeName， 其值为href， 那么绑定将等价于v-bind:href
--->
```

绑定处理函数：

```html
<a v-on:[eventName]="dosomething"></a>
```

-   对动态参数的值的约束

    动态参数预期会求出一个字符串，异常情况下值为 `null`。这个特殊的 `null` 值可以被显性地用于移除绑定。任何其它非字符串类型的值都将会触发一个警告。

-   对动态参数表达式的约束

    动态参数表达式有一些语法约束，因为某些字符，如空格和引号，放在 HTML attribute 名里是无效的。例如：

#### 修饰符

修饰符（modifier）是以半角句号`.` 指明的特殊后缀，用于指出一个指令应该以特殊方式绑定；例如` .prevent`修饰符告诉v-on指令对触发的事件调用event.preventDefault();

```html
<form v-on:submit.prevent = "onSubmit">
  
</form>
```

#### 缩写

```html
<!-- 完整语法 -->
<a v-bind:href="url">...</a>

<!-- 缩写 -->
<a :href="url">...</a>

<!-- 动态参数的缩写 (2.6.0+) -->
<a :[key]="url"> ... </a>


<!-- 完整语法 -->
<a v-on:click="doSomething">...</a>

<!-- 缩写 -->
<a @click="doSomething">...</a>

<!-- 动态参数的缩写 (2.6.0+) -->
<a @[event]="doSomething"> ... </a>
```

`:` 与 `@` 对于 attribute 名来说都是合法字符，在所有支持 Vue 的浏览器都能被正确地解析。而且，它们不会出现在最终渲染的标记中。

### 8. 计算属性

```html
<div id="example">
  {{ message.split('').reverse().join('') }}
</div>
```

这里是想要显示变量 `message` 的翻转字符串。当你想要在模板中的多处包含此翻转字符串时，就会更加难以处理。

所以，对于任何复杂逻辑，你都应当使用**计算属性**

例如：

```html
<div id="example">
  <p>Original message: "{{ message }}"</p>
  <p>Computed reversed message: "{{ reversedMessage }}"</p>
</div>
```

```javascript
var vm = new Vue({
  el: '#example',
  data: {
    message: 'Hello'
  },
  computed: {
    // 计算属性的 getter
    reversedMessage: function () {
      // `this` 指向 vm 实例
      return this.message.split('').reverse().join('')
    }
  }
})

//页面显示：
//Original message: "Hello"
//Computed reversed message: "olleH"
```

声明了一个计算属性`reversedMessage`；我们提供的函数将用作property `vm.reversedMessage`的getter函数

```javascript
console.log(vm.reversedMessage) // olleH
vm.message = 'Goodbye'
console.log(vm.reversedMessage) // => 'eybdooG'
```

你可以打开浏览器的控制台，自行修改例子中的 vm。`vm.reversedMessage` 的值始终取决于 `vm.message` 的值。

你可以像绑定普通 property 一样在模板中绑定计算属性。Vue 知道 `vm.reversedMessage` 依赖于 `vm.message`，因此当 `vm.message` 发生改变时，所有依赖 `vm.reversedMessage` 的绑定也会更新。以声明的方式创建了这种依赖关系：计算属性的 getter 函数是没有副作用 (side effect) 的。

#### 计算属性 VS 方法

使用表达式中调用方法同样可以达到上面的结果

```html
<p>Reversed message: "{{ reversedMessage() }}"</p>
```

```javascript
// 在组件中
methods: {
  reversedMessage: function () {
    return this.message.split('').reverse().join('')
  }
}
```

我们可以将同一函数定义为一个方法而不是一个计算属性。两种方式的最终结果确实是完全相同的。然而，不同的是**计算属性是基于它们的响应式依赖进行缓存的**。只在相关响应式依赖发生改变时它们才会重新求值。这就意味着只要 `message` 还没有发生改变，多次访问 `reversedMessage` 计算属性会立即返回之前的计算结果，而不必再次执行函数。

这也同样意味着下面的计算属性将不再更新，因为 `Date.now()` 不是响应式依赖：

```
computed: {
  now: function () {
    return Date.now()
  }
}
```

相比之下，每当触发重新渲染时，调用方法将**总会**再次执行函数。

我们为什么需要缓存？假设我们有一个性能开销比较大的计算属性 **A**，它需要遍历一个巨大的数组并做大量的计算。然后我们可能有其他的计算属性依赖于 **A**。如果没有缓存，我们将不可避免的多次执行 **A** 的 getter！如果你不希望有缓存，请用方法来替代。

#### 计算属性 VS 侦听属性

侦听属性：vue提供了一种更通用的方式来观察和响应vue实例上的数据变动；当有一些数据需要随着其他数据变动而变动时；很容易滥用watch;**通常更好的做法是使用计算属性而不是命令式的watch回调**；

```html
<div id="demo">{{ fullName }}</div>
```

```javascript
var vm = new Vue({
  el: '#demo',
  data: {
    firstName: 'Foo',
    lastName: 'Bar',
    fullName: 'Foo Bar'
  },
  //侦听属性watch 
  watch: {
    firstName: function (val) {
      this.fullName = val + ' ' + this.lastName
    },
    lastName: function (val) {
      this.fullName = this.firstName + ' ' + val
    }
  },
  //计算属性
  computed: {
    fullName: function () {
      return this.firstName + ' ' + this.lastName
    }
  }
})

```

#### 计算属性的setter

计算属性默认只有getter，自己可以提供一个setter

```javascript
computed: {
  fullName: {
    // getter
    get: function () {
      return this.firstName + ' ' + this.lastName
    },
    // setter
    set: function (newValue) {
      var names = newValue.split(' ')
      this.firstName = names[0]
      this.lastName = names[names.length - 1]
    }
  }
}
```

现在再运行 `vm.fullName = 'John Doe'` 时，setter 会被调用，`vm.firstName` 和 `vm.lastName` 也会相应地被更新。

### 9. 侦听器

当需要在数据变化时执行异步或开销较大的操作时，watch是最有用的；同时也可以自定义侦听器；

```html
<div id="watch-example">
  <p>
    Ask a yes/no question:
    <input v-model="question">
  </p>
  <p>{{ answer }}</p>
</div>
```

```javascript
<!-- 因为 AJAX 库和通用工具的生态已经相当丰富，Vue 核心代码没有重复 -->
<!-- 提供这些功能以保持精简。这也可以让你自由选择自己更熟悉的工具。 -->
<script src="https://cdn.jsdelivr.net/npm/axios@0.12.0/dist/axios.min.js"></script>
<script src="https://cdn.jsdelivr.net/npm/lodash@4.13.1/lodash.min.js"></script>
<script>
var watchExampleVM = new Vue({
  el: '#watch-example',
  data: {
    question: '',
    answer: 'I cannot give you an answer until you ask a question!'
  },
  watch: {
    // 如果 `question` 发生改变，这个函数就会运行
    question: function (newQuestion, oldQuestion) {
      this.answer = 'Waiting for you to stop typing...'
      this.debouncedGetAnswer()
    }
  },
  created: function () {
    // `_.debounce` 是一个通过 Lodash 限制操作频率的函数。
    // 在这个例子中，我们希望限制访问 yesno.wtf/api 的频率
    // AJAX 请求直到用户输入完毕才会发出。想要了解更多关于
    // `_.debounce` 函数 (及其近亲 `_.throttle`) 的知识，
    // 请参考：https://lodash.com/docs#debounce
    this.debouncedGetAnswer = _.debounce(this.getAnswer, 500)
  },
  methods: {
    getAnswer: function () {
      if (this.question.indexOf('?') === -1) {
        this.answer = 'Questions usually contain a question mark. ;-)'
        return
      }
      this.answer = 'Thinking...'
      var vm = this
      axios.get('https://yesno.wtf/api')
        .then(function (response) {
          vm.answer = _.capitalize(response.data.answer)
        })
        //异常捕获
        .catch(function (error) {
          vm.answer = 'Error! Could not reach the API. ' + error
        })
    }
  }
})
</script>
```

使用 `watch` 选项允许我们执行异步操作 (访问一个 API)，限制我们执行该操作的频率，并在我们得到最终结果前，设置中间状态。这些都是计算属性无法做到的

### 10. class与style绑定

将 `v-bind` 用于 `class` 和 `style` 时，Vue.js 做了专门的增强。表达式结果的类型除了字符串之外，还可以是对象或数组

#### 10.1 绑定html class

##### 10.1.1对象语法

方式一：内联

```html
<div
  class="static"
  v-bind:class="{ active: isActive, 'text-danger': hasError }"
></div>
//data
data: {
  isActive: true,
  hasError: false
}
```

方式二：绑定的数据对象不必内联定义在模板里

```html
<div v-bind:class="classObject"></div>
//vue data
data: {
  classObject: {
    active: true,
    'text-danger': false
  }
}
```

方式三：绑定一个返回对象的计算属性（常用）

```html
<div v-bind:class="classObject"></div>

//vue data
data: {
  isActive: true,
  error: null
},
computed: {
  classObject: function () {
    return {
      active: this.isActive && !this.error,
      'text-danger': this.error && this.error.type === 'fatal'
    }
  }
}
```

##### 10.1.2 数组语法

