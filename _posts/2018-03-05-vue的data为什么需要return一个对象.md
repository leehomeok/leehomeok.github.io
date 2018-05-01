在创建或注册模板的时候，传入一个data属性作为用来绑定的数据。但是在组件中，data必须是一个函数，而不能直接把一个对象赋值给它。
```javascript
Vue.component('my-component', {
template: '<div>OK</div>',
data() {
  return {} // 返回一个唯一的对象，不要和其他组件共用一个对象进行返回
  },
})
```
你在前面看到，在new Vue()的时候，是可以给data直接赋值为一个对象的。这是怎么回事，为什么到了组件这里就不行了。
你要理解，上面这个操作是一个简易操作，实际上，它首先需要创建一个组件构造器，然后注册组件。注册组件的本质其实就是建立一个组件构造器的引用。使用组件才是真正创建一个组件实例。所以，注册组件其实并不产生新的组件类，但会产生一个可以用来实例化的新方式。
理解这点之后，再理解js的原型链：
```javascript
var MyComponent = function() {}
MyComponent.prototype.data = {
a: 1,
b: 2,
}
// 上面是一个虚拟的组件构造器，真实的组件构造器方法很多
var component1 = new MyComponent()
var component2 = new MyComponent()
```