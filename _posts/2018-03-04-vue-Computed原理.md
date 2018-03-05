   

本来 vue 的响应式应该才是重中之重。但是网上的文章很多很多。在看 computed 的实现之前。肯定还是要把 vue 的响应式如何实现好好看一下。或者说两者根本就是一样的东西。这边推荐几篇文章关于 vue 的响应式。

[vue 响应式简单实现](http://www.cnblogs.com/caizhenbo/p/6710174.html)

[vue 慕课响应式手记](http://www.imooc.com/article/14466)

还是看看官网对于响应式的解释：
[![](https://sfault-image.b0.upaiyun.com/317/284/3172844570-5947d2cda3491_articlex)](https://sfault-image.b0.upaiyun.com/317/284/3172844570-5947d2cda3491_articlex)

总的来说。vue 实现响应式的关键有三个：watcher,dep,observe;

1.  observe: 遍历 data 中的属性。在 get,set 方法中设置核心数据劫持

2.  dep：每个属性都有一个自己的 dep（消息订阅起）用于订制该属性上的所有观察者

3.  watcher：观察者，通过 dep 实现对响应属性的监听观察。观察得到结果后，主动触发自己的回调

可以去看看 vue2.3 的这三部分源码。中间还是有很多精美的设计。比如一个全局唯一的 Dep.target，在任何时候都是唯一的值。以确保同一时间只有一个观察者在订阅。再比如，watcher 中也会存下相关的订阅器，实现去重和实现同一个观察者的分组（这里是实现 computed 的关键），再如。watcher 中的 id 也会唯一。用于异步更新的时候不同时出发相同的订阅。仔细看看会收获不小。改天我把所有的响应式的代码也整理一下。

在理解了响应式的情况下。我们来看看 computed 的实现。最简单的一个 demo 如下：

```javascript
new Vue({
  el: '#app',
  data: function () {
    return {
      firstName: 111,
      lastName: 222
    }
  },
  computed: {
    computeA: function () {
      return this.firstName + ' ' + this.lastName
    }
  },
  created(){
    setTimeout(
      () => {
        this.firstName = 333;
      },1000
    )
  }
})
```
我们来从源码的角度看看发生了什么：

1.  在初始化实例创建响应式的时候。对 options 中的 computed 做了特殊处理：
```javascript
function initComputed (vm, computed) {
  var watchers = vm._computedWatchers = Object.create(null);

  for (var key in computed) {
    var userDef = computed[key];
    var getter = typeof userDef === 'function' ? userDef : userDef.get;
    {
      if (getter === undefined) {
        warn(
          ("No getter function has been defined for computed property \"" + key + "\"."),
          vm
        );
        getter = noop;
      }
    }
    // create internal watcher for the computed property.
    watchers[key] = new Watcher(vm, getter, noop, computedWatcherOptions);//为每一个computed项目订制一个watcher

    // component-defined computed properties are already defined on the
    // component prototype. We only need to define computed properties defined
    // at instantiation here.
    if (!(key in vm)) {
      defineComputed(vm, key, userDef);
    } else {
      if (key in vm.$data) {
        warn(("The computed property \"" + key + "\" is already defined in data."), vm);
      } else if (vm.$options.props && key in vm.$options.props) {
        warn(("The computed property \"" + key + "\" is already defined as a prop."), vm);
      }
    }
  

function defineComputed (target, key, userDef) {
  if (typeof userDef === 'function') {
    sharedPropertyDefinition.get = createComputedGetter(key);
    sharedPropertyDefinition.set = noop;
  } else {
    sharedPropertyDefinition.get = userDef.get
      ? userDef.cache !== false
        ? createComputedGetter(key)
        : userDef.get
      : noop;
    sharedPropertyDefinition.set = userDef.set
      ? userDef.set
      : noop;
  }
  Object.defineProperty(target, key, sharedPropertyDefinition);
}

function createComputedGetter (key) {//构造该computed的get函数
  return function computedGetter () {
    var watcher = this._computedWatchers && this._computedWatchers[key];
    if (watcher) {
      if (watcher.dirty) {
        watcher.evaluate();//收集该watcher的订阅
      }
      if (Dep.target) {
        watcher.depend();//同一为这一组订阅再加上组件re-render的订阅（该订阅负责更新组件）
      }
      return watcher.value
    }
  }
}
```

总的来说。理解了响应式的构建之后。再来看 computed 的实现还是很直观的。组件初始化的时候。computed 项和 data 中的分别建立响应式。data 中的数据直接对属性的 get,set 做数据拦截。而 computed 则建立一个新的 watcher，在组件渲染的时候。先 touch 一下这个 computed 的 getter 函数。将这个 watcher 订阅起来。这里相当于这个 computed 的 watcher 订阅了 firstname 和 lastname。touch 完后。Dep.target 此时又变为之前那个用于更新组件的。再通过 watcher.depend() 将这个组统一加上这个订阅。这样一旦 firstname 和 lastname 变了。同时会触发两个订阅更新。其中一个便是更新组件。重新 re-render 的函数。感觉看的还不够细啊