
```js
async function async1() {
    console.log('async1 start')
    await async2()
    console.log('async1 end')
};
async function async2(){
    console.log('async2')
};
console.log('scriptstart');
setTimeout(function(){console.log('setTimeout')},0);
async1();
new Promise(function(resolve) {
    console.log('promise1')
    resolve()
    }).then(function(){console.log('promise2')})
console.log('script end')

```

输出
scriptstart
async1 start
async2
promise1
script end
async1 end
promise2
setTimeout

解析
注意 await 会阻塞下面的代码执行，后面的函数会正常在当前宏任务队列执行，下面的则是相当于放进了该次微任务队列里，而setTimeout是放在下一轮的宏任务队列里的。