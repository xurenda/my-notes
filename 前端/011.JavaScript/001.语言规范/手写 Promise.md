---
date: 2024-04-02 18:25
modified: 2024-04-03 10:51
---

# Promise A+ 规范

[Promise A+ 规范](https://promisesaplus.com/)

```js
class MyPromise {
  // 三种状态
  static _statuses = {
    pending: 'pending', // 等待
    fulfilled: 'fulfilled', // 成功
    rejected: 'rejected', // 失败
  }

  /**
   * 创建一个 Promise
   * @param {Function} executor 任务执行器，立即执行
   */
  constructor(executor) {
    // 一开始状态为 pending
    this._status = MyPromise._statuses.pending
    // 数据，fulfilled：值，rejected：失败原因
    this._value = undefined
    // then、catch 处理函数可能是多个，需要执行队列
    this._handlers = []

    try {
      // 同步立即执行
      executor(this._resolve.bind(this), this._reject.bind(this))
    } catch (error) {
      // 执行器抛出异常，标记为失败
      this._reject(error)
    }
  }

  /**
   * 标记当前任务完成
   * @param {any} value 任务完成后的值
   */
  _resolve(value) {
    this._changeStatus(MyPromise._statuses.fulfilled, value)
  }

  /**
   * 标记当前任务失败
   * @param {any} reason 任务失败的原因
   */
  _reject(reason) {
    this._changeStatus(MyPromise._statuses.rejected, reason)
  }

  /**
   * 更改任务状态
   * @param {string} status 新状态
   * @param {any} value 相关数据
   */
  _changeStatus(status, value) {
    // 如果状态已经改变，就不能再次更改
    if (this._status !== MyPromise._statuses.pending) {
      return
    }
    // 如果 value 是 Promise，则当前 Promise 与 value 一致
    if (isPromiseLike(value)) {
      value.then(this._resolve.bind(this), this._reject.bind(this))
      return
    }
    this._status = status
    this._value = value
    // 状态变化，执行处理函数队列
    this._runHandlers()
  }

  /**
   * Promise A+ 规范的 then
   * @param {Function} onFulfilled 任务成功的处理函数
   * @param {Function} onRejected 任务失败的处理函数
   * @returns {Promise} 新的 Promise，可以链式调用
   */
  then(onFulfilled, onRejected) {
    return new MyPromise((resolve, reject) => {
      // 添加成功和失败处理函数到队列
      // 并保存 resolve 和 reject 用于链式调用
      this._handlers.push(
        {
          executor: onFulfilled,
          status: MyPromise._statuses.fulfilled,
          resolve,
          reject,
        },
        {
          executor: onRejected,
          status: MyPromise._statuses.rejected,
          resolve,
          reject,
        },
      )
      // 调用 then 的时候，可能状态已经改变，所以需要执行
      this._runHandlers()
    })
  }

  /**
   * 执行处理函数
   */
  _runHandlers() {
    // 如果状态还是 pending，则不执行
    if (this._status === MyPromise._statuses.pending) {
      return
    }
    // 保证队列顺序，执行完就弹出
    while (this._handlers[0]) {
      const handler = this._handlers.shift()
      this._runHandle(handler)
    }
  }

  /**
   * 处理一个 handler
   * @param {Object} handler
   */
  _runHandle({ executor, status, resolve, reject }) {
    // 只执行状态和当前 Promise 状态一致的处理函数
    if (status !== this._status) {
      return
    }
    // 放到微任务队列中执行
    runMicroTask(() => {
      // 处理函数不是函数，则直接执行 resolve 或 reject
      if (typeof executor !== 'function') {
        switch (this._status) {
          case MyPromise._statuses.fulfilled:
            resolve(this._value)
            break
          case MyPromise._statuses.rejected:
            reject(this._value)
            break
        }
        return
      }
      // 处理函数是函数，根据处理函数的执行结果来执行 resolve 或 reject，保证链式调用的状态一致
      try {
        const result = executor(this._value)
        if (isPromiseLike(result)) {
          result.then(resolve, reject)
        } else {
          resolve(result)
        }
      } catch (error) {
        reject(error)
      }
    })
  }
}

/**
 * 运行一个微任务，把传递的函数放到微任务队列中执行
 * @param {Function} func 要在微任务中执行的函数
 */
function runMicroTask(func) {
  if (typeof queueMicrotask === 'function') {
    // node、browser
    queueMicrotask(func)
  } else if (process && typeof process.nextTick === 'function') {
    // node
    process.nextTick(func)
  } else if (typeof MutationObserver === 'function') {
    // browser
    const observer = new MutationObserver(func)
    const node = document.createTextNode('')
    observer.observe(node, { characterData: true })
    node.data = 'x'
  } else {
    // 上面的 API 都不支持，直接在下一轮事件循环中执行
    setTimeout(func, 0)
  }
}

/**
 * 判断一个数据是否是一个符合 Promise A+ 规范的 Promise
 * @param {any} value
 * @returns
 */
function isPromiseLike(value) {
  // 是对象或函数，且有 then 方法
  return value && typeof value.then === 'function'
}
```

# ECMAScript 规范

## `catch` 和 `finally` 成员方法

```js
class MyPromise {
  // ...

  /**
   * Promise A+ 规范的 then
   * @param {Function} onFulfilled 任务成功的处理函数
   * @param {Function} onRejected 任务失败的处理函数
   * @returns {Promise} 新的 Promise，可以链式调用
   */
  then(onFulfilled, onRejected) {
    // ...
  }

  /**
   * 仅注册任务失败的处理函数
   * @param {Function} onRejected 任务失败的处理函数
   * @returns {Promise} 新的 Promise，可以链式调用
   */
  catch(onRejected) {
    return this.then(undefined, onRejected)
  }

  /**
   * 注册任务结束（无论成功或失败）的处理函数
   * @param {Function} onFinally 任务结束（无论成功或失败）的处理函数
   * @returns {Promise} 新的 Promise，可以链式调用
   */
  finally(onFinally) {
    // onFinally 没有参数，返回值也会被忽略
    // 返回的 Promise 状态和当前 Promise 状态一致
    return this.then(
      value => {
        onFinally()
        return value
      },
      reason => {
        onFinally()
        throw reason
      },
    )
  }

  // ...
}
```

## `resolve` 和 `reject` 静态方法

```js
class MyPromise {
  // ...

  /**
   * 返回一个已完成的 Promise
   * @param {any} data
   * @returns {Promise} 新的 Promise，是 MyPromise 类型
   */
  static resolve(data) {
    // data 本身就是 MyPromise 对象，直接返回
    if (data instanceof MyPromise) {
      return data
    }
    return new MyPromise((resolve, reject) => {
      // data 是 PromiseLike（Promise A+），返回新的 Promise，状态和其保持一致即可
      if (isPromiseLike(data)) {
        data.then(resolve, reject)
      } else {
        resolve(data)
      }
    })
  }

  /**
   * 返回一个失败的 Promise
   * @param {any} reason 失败原因
   * @returns 新的失败的 Promise，是 MyPromise 类型
   */
  static reject(reason) {
    return new MyPromise((_resolve, reject) => {
      reject(reason)
    })
  }
}
```

## `all`、`allSettled`、`race` 静态方法

```js
class MyPromise {
  // ...

  /**
   * 得到一个新的 Promise，该 Promise 的状态取决于 promises 的执行：
   * promises 全部成功，则返回的 Promise 成功，数据是所有 Promise 成功数据的数组，并且顺序是按照 promises 的顺序排列
   * promises 中只要有一个失败，则返回的 Promise 失败，原因是第一个失败的 Promise 的原因
   * @param {Iterable} promises 迭代器，包含多个 Promise 或非 Promise
   * @returns 新的 Promise
   */
  static all(promises) {
    return new MyPromise((resolve, reject) => {
      try {
        let totalCount = 0 // Promise 的总数
        let fulfilledCount = 0 // 已完成的 Promise 数量
        const result = []
        // 因为 promises 是一个可迭代对象，所以需要用 for...of 遍历
        // 也不能直接读取 length
        for (const prom of promises) {
          const i = totalCount
          totalCount++
          // 传入的值可能不是 Promise，转成 Promise
          MyPromise.resolve(prom).then(value => {
            fulfilledCount++
            result[i] = value
            if (fulfilledCount === totalCount) {
              // 全部完成，执行 resolve
              resolve(result)
            }
          }, reject)
        }
        if (totalCount === 0) {
          // 迭代器无值，进不到 for...of，直接 resolve
          resolve(result)
        }
      } catch (error) {
        // 遍历过程中抛出异常，直接执行 reject
        reject(error)
      }
    })
  }

  /**
   * 得到一个新的 Promise，该 Promise 的状态一定成功
   * 数据是所有 Promise 数据的数组，并且顺序是按照 promises 的顺序排列
   * @param {Iterable} promises 迭代器，包含多个 Promise 或非 Promise
   * @returns 新的 Promise
   */
  static allSettled(promises) {
    const handledPromises = []
    for (const prom of promises) {
      handledPromises.push(
        // 传入的值可能不是 Promise，转成 Promise
        // 不管成功还是失败，都会成功，数据加了一个 status 以标识成功或失败
        MyPromise.resolve(prom).then(
          value => ({ status: MyPromise._statuses.fulfilled, value }),
          reason => ({ status: MyPromise._statuses.rejected, reason }),
        ),
      )
    }
    return MyPromise.all(handledPromises)
  }

  /**
   * 返回的 Promise 与第一个有结果的一致
   * @param {Iterable} promises 迭代器，包含多个 Promise 或非 Promise
   * @returns 新的 Promise
   */
  static race(promises) {
    return new MyPromise((resolve, reject) => {
      for (const prom of promises) {
        // 传入的值可能不是 Promise，转成 Promise
        // 多次改变状态，只有第一次会改变状态成功
        MyPromise.resolve(prom).then(resolve, reject)
      }
      // 如果迭代器无值，则进不到 for...of，会一直等待
    })
  }
}
```
