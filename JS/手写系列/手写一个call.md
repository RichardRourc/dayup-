# 手写一个call

## 回顾使用方式
```js
    fnA.call(objA,argsA,b,c)
```

函数上的原型的方法，参数 context,args1,args2...

## 做了什么
- 修改原函数this指向新对象,how? 利用Js的隐式修改this指向。 obj.fn() 这时fn函数体内this不久指向obj了。施展小计，把原来的this存到obj的属性里。这样既可以调用原函数，又能修改原函数的this指向。
- 使用参数调用原函数
- 返回结果

## 代码实现
```js
    Function.prototype.myCall = function(context=window,...args){
        const symbolFn = Symbol(); // 做一个唯一标识，免得和其他属性冲突了

        const context[symbolFn] = this; // 拿到原 this ，用于调用 ，因为使用方式是 fnA.myCall，此时的this是能拿到fnA的。this是个关键字，它的具体指向跟运行时的环境有关。隐式修改（a.fn()），显示修改(call,apply,bind),new,constructor里

        const result = context[symbolFn](...args); // 在 context[symbolFn](...args) 这一步，利用js的隐式修改this指向将这个函数的this指向了context。因为实际上等同于类似调用了 context.myFn() 这时myFn()里的this指向肯定就是context了对不。

        delete context[symbolFn]; // 结束之后，删除属性，避免内存泄漏。

        return result;
    }
```