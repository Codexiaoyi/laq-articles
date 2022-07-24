# vue双向数据绑定的原理

## 原理

vue是采用数据劫持结合**发布者-订阅者模式**的方法，通过Object.defineProPerty()来劫持各个属性的setter，getter，在数据变动时发布消息给订阅者，触发相应的监听回调。主要步骤如下：

1. 需要observe的数据对象进行递归遍历，包括子属性对象的属性，都加上setter和getter这样的话，给这个对象的某个值赋值，就会出发setter，那么就能监听到了数据变化。
2. compile解析模版指令，将模版中的变量替换成数据，然后初始化渲染页面视图，并将每个指令对应的节点绑定更新函数，添加监听数据的订阅者，一旦数据有变动，订阅者收到通知就更新视图。
3. watcher订阅者是observer和compile之间通信的桥梁，主要做的事情是：1、在自身实例化时往属性订阅器中添加自己。2、自身有一个update（）方法。3、待属性变动订阅器通知时，能调用自身的update（）方法，并触发compile中绑定的回调
4. MVVM作为数据绑定的入口，整合observer、compile和watcher三者，通过observer来监听自己的model数据变化，通过compile解析编译模版，最终利用watcher搭起observer和compile之间的通信，达到双向绑定效果。

![image-20220407163009291](https://zzzqi-images.oss-cn-hangzhou.aliyuncs.com/202204071630385.png)

## vue2使用object.defineProperty()来进行数据劫持有什么缺点

在对一些属性进行操作时，使用这种方法无法拦截，比如通过下标方式修改数组数据或者给对象新增属性，这都不能触发组件的重新渲染，因为object.defineProperty不能拦截这些操作。更精确的来说，对数组而言，大部分操作都是拦截不到的，只是vue内部通过重写函数的方式解决了这个问题。

vue3使用proxy对对象进行代理，从而实现数据劫持。使用proxy的好处是它可以完美监听到任何方式的数据改变，唯一缺点是兼容性问题，因为proxy是es6的语法。



# object.defineProperty()函数

## 参数

- obj 需要定义属性的对象。
- prop 需被定义或修改的属性名。
- descriptor 需被定义或修改的属性的描述符。

## 描述符

数据描述符和存取描述符均具有以下可选键值：

- configurable: 仅当该属性的 configurable 为 true 时，该属性才能够被改变，也能够被删除。默认为 false
- enumerable: 仅当该属性的 enumerable 为 true 时，该属性才能够出现在对象的枚举属性中。默认为 false
- value: 该属性对应的值。可以是任何有效的 JavaScript 值（数值，对象，函数等）。默认为 undefined
- writable: 仅当仅当该属性的writable为 true 时，该属性才能被赋值运算符改变。默认为 false
- get: 一个给属性提供 getter 的方法，如果没有 getter 则为 undefined。该方法返回值被用作属性值。undefined
- set: 一个给属性提供 setter 的方法，如果没有 setter 则为 undefined。该方法将接受唯一参数，并将该参数的新值分配给该属性。默认值为undefined。

