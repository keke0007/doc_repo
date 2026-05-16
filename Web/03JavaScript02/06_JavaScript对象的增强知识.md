JavaScript对象的增强知识

目录

```javascript
Object.defineProperty
```

content 数据属性描述符 存取属性描述符

```javascript
Object.defineProperties
```

对象的其他方法补充

## 对属性操作的控制

■在前面我们的属性都是直接定义在对象内部，或者直接添加到对象内部的：

- 但是这样来做的时候我们就不能对这个属性进行一些限制：比如这个属性是否是可以通过delete删除的？这个属性是否在for
in遍历的时候被遍历出来呢？

```javascript
var obj = {
```

name:- "why", age: 18, height: 1.88 如果我们想要对一个属性进行比较精准的操作控制，那么我们就可以使用属性描述符。

- 通过属性描述符可以精准的添加或修改对象的属性；
- 属性描述符需要使用Object.defineProperty来对属性进行添加或者修改；

## Object.defineProperty

1Object.definePropertyO方法会直接在一个对象上定义一个新属性，或者修改一个对象的现有属性，并返回此对象。

```javascript
Object.defineProperty(obj,prop,descriptor)
```

可接收三个参数：

- obj要定义属性的对象;
- prop要定义或修改的属性的名称或Symbol;
- descriptor要定义或修改的属性描述符；
返回值：

- 被传递给函数的对象。

## 属性描述符分类

属性描述符的类型有两种：

- 数据属性(Data Properties)描述符(Descriptor);
- 存取属性(Accessor访问器Properties)描述符(Descriptor);
configurable enumerable value writable get set 不可 不可 数据描述符 可以 可以 可以 可以 以 以 存取描述符 可以 可以 不可以 不可以 可以 可以

## 数据属性描述符

数据数据描述符有如下四个特性： [Configurable]]：表示属性是否可以通过delete删除属性，是否可以修改它的特性，或者是否可以将它修改为存取属性描述符；

- 当我们直接在一个对象上定义某个属性时，这个属性的[Configurable]]为true;
- 当我们通过属性描述符定义一个属性时，这个属性的[Configurable]]默认为false;
[[Enumerable]]：表示属性是否可以通过for-in或者Object.keysO返回该属性；

- 当我们直接在一个对象上定义某个属性时，这个属性的[[Enumerable]]为true;
- 当我们通过属性描述符定义一个属性时，这个属性的[[Enumerable]默认为false；
[[Writable]]：表示是否可以修改属性的值；

- 当我们直接在一个对象上定义某个属性时，这个属性的[Writable]]为true;
- 当我们通过属性描述符定义一个属性时，这个属性的[[Writable]]默认为false;
```javascript
[value]]：属性的value值，读取属性时会返回该值，修改属性时，会对其进行修改;
```

- 默认情况下这个值是undefined；

## 数据属性描述符测试代码

```javascript
var obj = { 1·自己定义的属性 /.1.测试enumerable为false
```

- /·这种方式访问时看不到属性
```javascript
name: "why", Object.defineProperty(obj,·"address",·{ console.log(obj)
age: 18, configurable:·false, console.log(object.keys(obj))
height:·1.88 enumerable:·false, for (var key in obj)·{
console. log(key)
```

- /·writable:·false,
默认是可以配置 value：·"北京市" 1/·这种方式是可以访问的

```javascript
delete obj.name }) console.log("address" in obj)
console.log(obj.has0wnProperty('address'))
console.log(obj) console.log(obj.address)
for (var key in obj) { //·2.测试writable，·修改address的值
console. log(key) obj.address =·"广州市"
```

- /·北京市，·并且在严格模式下会报错
```javascript
console.log(obj.address)
console.log(object.keys(obj)) //·3.测试configurable
```

1/·不可以删除

```javascript
obj.name = "kobe" delete obj.address
console.log(obj) 1/·不可以重新修改
object.defineProperty(obj,·'address'，·{
```

configurable: true

```javascript
})
console.log(obj.address)
```

## 存取属性描述符

数据数据描述符有如下四个特性：

```javascript
[Configurable]]：表示属性是否可以通过delete删除属性，是否可以修改它的特性，或者是否可以将它修改为存取属性描述符;
```

- 和数据属性描述符是一致的；
- 当我们直接在一个对象上定义某个属性时，这个属性的[Configurable]]为true;
- 当我们通过属性描述符定义一个属性时，这个属性的[[Configurable]]默认为false;
[[Enumerable]]：表示属性是否可以通过for-in或者Object.keysO返回该属性；

- 和数据属性描述符是一致的；
- 当我们直接在一个对象上定义某个属性时，这个属性的[[Enumerable]]为true;
- 当我们通过属性描述符定义一个属性时，这个属性的[[Enumerable]默认为false;
[[get]]：获取属性时会执行的函数。默认为undefined [[set]]：设置属性时会执行的函数。默认为undefined

## 存储属性描述符测试代码

"use strict"

```javascript
var obj = {
```

name: "why", age: 18

```javascript
var address =·"北京市"
Object.defineProperty(obj,·'address', {
```

configurable: true, enumerable: true,

```javascript
get: function()·{
return address
set:·function(value)·{
address = value
console.log(obj.address)
obj.address =·"广州市"
console.log(obj.address)
```

## 同时定义多个属性

1Object.definePropertiesO方法直接在一个对象上定义多个新的属性或修改现有属性，并且返回该对象。

```javascript
var obj = {
```

_age: 18

```javascript
Object.defineProperties(obj,°{
name:{
```

writable:·true, value: "why" age:

```javascript
get: function() ·{
return·this._age
```

## 对象方法补充

获取对象的属性描述符： getOwnPropertyDescriptor getOwnPropertyDescriptors 禁止对象扩展新属性：preventExtensions

- 给一个对象添加新的属性会失败(在严格模式下会报错)；
密封对象，不允许配置和删除属性：seal

- 实际是调用preventExtensions
- 并且将现有属性的configurable：false
冻结对象，不允许修改现有属性：freeze

- 实际上是调用seal
- 并且将现有属性的writable：false
