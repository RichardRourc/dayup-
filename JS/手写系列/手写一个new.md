
手写一个 new 

##### 原理
先来回顾下 js 的 new 操作符做了些什么呢

+ 创建一个新的空对象
+ 将新对象的prototype原型指向操作符的原型
+ 将构造函数的this指向新对象，并执行构造函数
+ 返回对象，判断构造函数返回的变量的类型是否对象类型，如果是就直接返回，否则返回创建的新对象


##### 实现
上面大致说完了原理，那么接下来就来说一下这个模拟实现的方法吧。

首先，写一个函数 
```js
    function myNew (constructor,...args) {}  // constructor-构造函数，...args-传的参数
```

新建一个对象，修改新对象的原型，执行构造函数，判断构造函数返回的类型，如果是对象就直接返回，否则返回新对象
```js
    function myNew (constructor,...args) {
        const obj = {};
        Object.setPrototypeOf(obj,constructor.prototype)

        const result = constructor.apply(obj,args)
        return Object.prototype.toString.call(result)==='[object Object]' ? result : obj
    }
```

##### 总结
了解 new 操作符做了什么，写一个 new 函数出来就很简单了，来复习下。
第一步：创建新对象；
第二步：修改新对象的原型指向构造函数的原型；
第三步：修改构造函数的 this 指向新对象，并执行构造函数；
第四步：判断构造函数返回的结果是否对象类型，如果是则直接返回，否则返回开头创建的新对象。

// PS: 之前突然在想为啥要修改this指向，现在想想，人家构造函数，不修改指向，如果构造函数自己代码不写返回对象，那做了个寂寞打的操作啊，修改this指向，一方面让新对象有了继承的效果，一方面让构造函数做的操作“有家可归”哈哈哈。