## 说说 var、let、const 之间的区别

`var`、`let`、`const` 三者区别可以围绕下面五点展开：

- 变量提升
- 暂时性死区
- 块级作用域
- 重复声明
- 修改声明的变量
- 使用

### 变量提升

在 ES5 中，顶层对象的属性和全局变量是等价的，用 var 声明的变量既是全局变量，也是顶层变量  
注意：顶层对象，在浏览器环境指的是 `window` 对象，在 Node 指的是 `global` 对象  
var 声明的变量存在变量提升，即变量可以在声明之前调用，值为 undefined  
let 和 const 不存在变量提升，即它们所声明的变量一定要在声明后使用，否则报错  
let 命令所在的代码块内有效

### 暂时性死区

var 不存在暂时性死区  
let 和 const 存在暂时性死区，只有等到声明变量的那一行代码出现，才可以获取和使用该变量
只要块级作用域内存在 let 命令，这个区域就不再受外部影响

```js{3}
let a = 123;
if (true) {
  a = "abc"; // ReferenceError
  let a;
}
```

### 块级作用域

var 不存在块级作用域  
let 和 const 存在块级作用域

### 重复声明

var 允许重复声明变量  
let 和 const 在同一作用域不允许重复声明变量

### 修改声明的变量

var 和 let 可以  
const 声明一个只读的常量。一旦声明，常量的值就不能改变  
const 实际上保证的并不是变量的值不得改动，而是变量指向的那个内存地址所保存的数据不得改动  
对于简单类型的数据，值就保存在变量指向的那个内存地址，因此等同于常量  
对于复杂类型的数据，变量指向的内存地址，保存的只是一个指向实际数据的指针，const 只能保证这个指针是固定的，并不能确保改变量的结构不变

### 使用

能用 const 的情况尽量使用 const，其他情况下大多数使用 let，避免使用 var