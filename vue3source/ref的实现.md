因为不同于reactive，只能依靠创建一个class来进行代理，所以这就是为什么调用的时候要用.value
```javascript
class refImpl {
	get value() {
  	// 依赖收集
  }
  set value(){
    // 依赖触发
  }
}
```
其他工具函数
isRef
unRef
proxyRef
