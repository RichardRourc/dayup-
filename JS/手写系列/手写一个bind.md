
手写一个js的bind。

为什么？
加深对js的函数上下文和闭包的理解

bind是什么？
创建一个新函数，将原来函数的上下文转到新的地方

手动实现bind的步骤

1：返回一个新函数
2：使用闭包捕获原始函数和新的函数上下文，用于最后正确执行函数
3：获得第一次生成函数时传参和第二次调用时传参




代码实现

```javascript

    // 挂在函数原型上的

    Function.prototype.myBind=function(context) {
        // 存储原始函数的引用
        const originalFunction = this
        // 从参数中获取绑定时的参数
        const bindArgs = Array.prototype.slice.call(arguments,1) // 第一个参数是context，后面的参数 详见后面举例


        return function() {
            // 新函数传的参数
            const funcArgs = Array.prototype.slice.call(arguments)
            // 返回执行结果
            return originalFunction.apply(context,bindArgs.concat(funcArgs))
        }

    }
```

例子

```js

// 示例使用
// 普通调用，新函数不传参
const person = {
    firstName: 'John',
    lastName: 'Doe',
    getFullName: function() {
        return this.firstName + ' ' + this.lastName;
    }
};

function logFullName() {
    console.log('Full name:',this.getFullName)
}

// 使用自定义的bind函数绑定logFullName函数到person对象
const boundLogFullName = logFullName.myBind(person)

boundLogFullName() // 输出 Full name: John Doe



// 另外传参的方式，新函数传参

const obj = {
    prop:'prop'
}

function originalFunction(){
    console.log('Test prop:',this.prop,'arg:',arguments)
}

const boundFunc = originalFunction.myBind(obj, 1, 2);

let additionalArg = 3

boundFunc(additionalArg);


```


另外arguments在es6里不推荐，可将其改为 ...args 放到形参里。

扩展运算符版本的实现
```js
Function.prototype.myBind = function(context,...args){
    const fn = this
    // const bindArgs = Array.prototype.slice.call(args,1)
    // 在arguments里，因为要获取实参但是又没有具体可以获取的，得通过slice这个类数组来转换来获取数组变量,但是在扩展运算符里因为用参数了就不用这么麻烦了。

    return function (...args1) {
        const allArgs = [...args,...args1]
        return fn.call(context,allArgs)
    }
}
```


总结：
挂载在函数原型链上，通过闭包保存原来函数调用和新指向的引用，并且考虑到后面参数传值将他们进行合并，最后通过原来函数的call转移到新指向上再使用


