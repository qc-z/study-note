1 reactive
**使用 proxy 对对象进行拦截，里面有 get，set，初次调用只初始化，不进行 get 或者 set，只进行拦截方法的设置**
​

2 effect
响应式原理：只有调用 effect，才能进行依赖收集，参数是一个函数，里面可以是依赖的赋值，
调用 effect 过程分析:
先创建 reactiveEffect 类，effect 收集进来先跑一编（调用 reactiveEffect 的 run 方法），把 this 指向
_1 调用 effect 会返回一个函数(runner)，调用该函数会运行一次 effect 内部的 fn 这里的 runner 相当于 reactiveEffect 的 run 方法——————> runner 的实现_
_2 调用 runner 会返回 fn 的返回值即 runner() ————————> 若有返回值就返回_
_3 调用 fn 会触发响应式对象的 get 方法，此处会进行依赖收集_
进行 run 方法之后，就会调用响应式的 get 方法，进行依赖收集
对应关系 targetMap(Map)->depsMap(Map)->dep(Set)
得到 dep 后，把当前的 activeEffect 收集进去*因为一个 key 对应一个 dep，一个 dep 可以收集很多 effect *
**这里注意为什么是当前的 activeEffect,例如下面例子，这样的当前是没有 effect，只是触发 get**

```javascript
const obj = reactive({ age: 20 })
console.log(obj.age)
```

​

依赖发生变化: reactive 发生变化，触发依赖收集即 set 方法,先进行*shouldTrack (应不应该触发)&& activeEffect(是不是当前 effect)判断， *后面补充，
找出当前的 dep，运行其中的 run 方法(如果有 scheduler 就调用，没有就调用 run）重新调用 run(这里是不是重新调用 get?)
​

3 reactiveEffect 类：
**收集传进来的 fn，包含 scheduler,stop,onstop,run 方法，stop 方法也是基于 reactiveEffect 的 stop 方法**
​

​
