

#### 背景
手写一个Promise，支持then catch

#### Promise A+规范
简介：用于规范Promise实现的规范，解决在不同环境下的兼容性问题。规定了三种状态，peding,fulfilled,rejected三种不可逆的状态，回调和链式调用

### 一个简易版，例如then里的回调函数本该是微任务队列里的，但是现在为了模拟实现和确保异步调用，使用的是setTimeout，会放进宏任务里


#### 具体实现
```js
    class MyPromise {
        constructor(executor){
            this.state='pending'; // 初始化promise状态
            this.value = undefined; // 成功的值
            this.reason = undefined; // 失败的原因
            this.onFulfilledCallbacks = []; // 存储成功回调
            this.onRejectedCallbacks = []; // 存储失败回调

            const resolve = (value)=>{
                if(this.state==='pending') {
                    this.state = 'fulfilled'; // 更新状态
                    this.value = value; // 保存成功的值
                    this.onFulfilledCallbacks.forEach(callback=>callback(value)); // 执行所有成功回调
                }
            }

            const reject = (reason) => {
                if(this.state==='pending') {
                    this.state = 'rejected';
                    this.reason = reason;
                    this.onRejectedCallbacks.forEach(callback=>callback(reason))
                }
            }

            try{
                executor(resolve,reject); // 执行传入的函数
            } catch(error) {
                reject(error); // 捕获错误
            }
        }

        then(onFulfilled,onRejected) {
            return new MyPromise((resolve,reject)=>{
                const handleFulfilled = ()=> {
                    try {
                        const result = onFulfilled(this.value); // 执行成功回调
                        resolve(result) // 将结果传递给下一个 Promise
                    } catch (error) {
                        reject(error); // 捕获错误并调用 reject
                    }
                }
                
                const handleRejected = ()=>{
                     try {
                        if(onRejected) {
                             const result = onRejected(this.reason) // 执行失败回调
                             resolve(result); // 将结果传递给下一个promise
                        }else {
                            reject(this.reason) // 如果没有提供失败回调，则直接 reject
                        }
                       
                     }  catch(error) {
                        reject(error); // 捕获错误
                     }
                }

                if(this.state==='fulfilled') {
                    // 不是微任务吗，如果是setTimeout 会放到宏任务里吧
                    setTimeout(handleFulfilled); // 确保异步执行
                }else if (this.state==='rejected') {
                    setTimeout(handleRejected); // 确保异步执行
                }else {
                    // 如果仍为 pending,则将回调保存
                    this.onFulfilledCallbacks.push(handleFulfilled);
                    this.onRejectedCallbacks.push(handleRejected);
                }
            })
        }

        catch(onRejected) {
            return this.then(null, onRejected); // 通过then的第二个参数实现catch
        }
    }
```
