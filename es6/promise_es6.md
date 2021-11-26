# es6

```js
const PENDING = 'pending'
const RESOLVED = 'resolved'
const REJECTED = 'rejected'

class Promise {

    constructor(excutor) {
        const self = this

        self.status = PENDING
        self.data = undefined
        self.callbacks = []

        function resolve(value) {
            if (self.status !== PENDING) {
                return
            }

            self.status = RESOLVED
            self.data = value

            if (self.callbacks.length > 0) {
                setTimeout(() => {
                    self.callbacks.forEach(callbacksObj => {
                        callbacksObj.onResolved12(value)
                    })
                })
            }
        }

        function reject(reason) {
            if (self.status !== PENDING) {
                return
            }

            self.status = REJECTED
            self.data = reason

            if (self.callbacks.length > 0) {
                setTimeout(() => {
                    self.callbacks.forEach(callbacksObj => {
                        callbacksObj.onRejected(reason)
                    })
                })
            }
        }

        try {
            excutor(resolve, reject)
        } catch (error) {
            reject(error)
        }
    }

    then(onResolved, onRejected) {
        onResolved = typeof onResolved === 'function' ?
            onResolved : value => value // 如果不是函数，也要让它成为函数，使得能够被调用callback(self.data)

        onRejected = typeof onRejected === 'function' ?
            onRejected : reason => {
                throw reason
            }

        const self = this // 这里一定是之前的this,后面返回的promise里面用的是上一个this

        return new Promise((resolve, reject) => {
            function handle(callback) {
                try {
                    const result = callback(self.data)
                    if (result instanceof Promise) {
                        // result.then(
                        //   value => resolve(value),  //当result成功,让return的promise也成功( x=>fn(x), 将fn(x)的返回值返回)
                        //   reason => reject(reason)  //当result失败,让return的promise也失败
                        // )
                        result.then(resolve, reject)
                    } else {
                        resolve(result)
                    }
                } catch (error) {
                    reject(error)
                }
            }

            if (self.status === PENDING) {
                self.callbacks.push({
                    onResolved12() {
                        handle(onResolved)
                    },
                    onRejected(reason) {
                        handle(onRejected)
                    }
                })
            } else if (self.status === RESOLVED) {
                setTimeout(() => { // 模拟微任务
                    handle(onResolved) // 本来在这里是直接执行onResolved(self.data)，这样写纯碎是为了封装函数
                })
            } else {
                setTimeout(() => {
                    handle(onRejected)
                })
            }
        })
    }

    catch(onRejected) {
        return this.then(undefined, onRejected)
    }

    static resolve(value) {
        return new Promise((resolve, reject) => {
            if (value instanceof Promise) {
                value.then(resolve, reject)
            } else {
                resolve(value)
            }
        })
    }

    static reject(reason) {
        return new Promise((resolve, reject) => {
            reject(reason)
        })
    }

    static all(promises) {
        const values = new Array(promises.length)
        let resolveCount = 0

        return new Promise((resolve, reject) => {
            promises.forEach((p, index) => {
                Promise.resolve(p).then( // 要明白then前面的操作，会先全部做完，最后执行then，因为then是异步回调
                    value => {
                        resolveCount++
                        values[index] = value
                        console.log('resolve的个数'+resolveCount)
                        if (resolveCount === promises.length) { // 进入resolve的次数与数组长度相同，则表明所有promise都走了resolve，此时才会resolve
                            resolve(values)
                        }
                    },
                    reason => {
                        reject(reason)
                    }
                )
            })
        })
    }

    static race(promises) {
        return new Promise((resolve, reject) => {
            promises.forEach((p, index) => {
                Promise.resolve(p).then(
                    value => {
                        resolve(value) // 一旦resolve,状态则已经改变，不可再变，所以后续循环的resolve(value)无效
                    },
                    reason => {
                        reject(reason)
                    }
                )
            })
        })
    }

    static resolveDelay(value, time) {
        return new Promise((resolve, reject) => {
            setTimeout(() => {
                if (value instanceof Promise) {
                    value.then(resolve, reject)
                } else {
                    resolve(value)
                }
            }, time)
        })
    }

    static rejectDelay(reason, time) {
        return new Promise((resolve, reject) => {
            setTimeout(() => {
                reject(reason)
            }, time)
        })
    }
}

const p = new Promise((resolve) => {
    resolve(1);
})

p.then(1).then(res => {
    console.log(res)
})

// const p1=new Promise((resolve) =>resolve(1))
// const p2=new Promise((resolve) =>resolve(2))
// const p3=Promise.resolve(3)
// const all = Promise.race([p1,p2,p3])
// all.then(res => {
//     console.log(res);
// },err => {
//     console.log(err);
// })
// console.log(all);


```

