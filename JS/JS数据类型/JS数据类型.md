

#### JS数据类型

1.JS原始数据类型有哪些？引用数据类型有哪些？

在JS中，存在着七种原始值，分别是：
+ boolean
+ null
+ undefined
+ number
+ string
+ symbol
+ bigint

引用数据类型：对象Object（包含普通对象-Object，数组对象-Array，正则对象-RegExp，日期对象-Date，数字函数-Math，函数对象-Function)

2.说出下面运行的结果，解释原因。

```js
    function test(person) {
        person.age=26;
        person={
            name:'hzj',
            age:18
        }

        return person
    }
    const p1 = {
        name:'fyq',
        age:19
    }
    const p2 = test(p1);
    console.log(p1);
    console.log(p2);
```

结果
```js
    p1 = {
        name:'fyq',
        age:26
    }

    p2 = {
        name:'hzj',
        age:18
    }
```

原因
在函数传参的时候传递的是对象在堆中的内存地址值，test函数中的实参person是p1对象的内存地址，通过调用person.age=26确实改变了p1的值，但随后person变成了另一块内存空间的地址，并且在最后将这另外一份内存空间的地址返回，赋值给了p2。

3.null是对象吗？为什么？
结论：null不是对象。
解释：虽然 typeof null 会输出 object，但是这只是 JS 存在的一个悠久 Bug。在 JS 的最初版本中使用的是 32 位的系统，为了性能考虑使用低位存储变量的类型信息，000开头代表是对象然而 null 表示为全零，所以将它错误的判断为 object。