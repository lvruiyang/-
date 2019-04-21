
# vue组件间通信的几种方式
## 1、props和emit
      vue父组件通过props向子组件传递值，子组件可以通过emit触发父组件上绑定的方法来传值.
## 2、$attrs和$listeners
      子组件可以通过$attrs获取父组件上作为 prop 被识别 (且获取) 的特性绑定 (class 和 style 除外)。当一个组件没有声明任何 prop 时，这里会包含所有父作用域的绑定 (class 和 style 除外)，$listeners可以获取父组件上除了native修饰符的事件监听。
## 3、$parent和$children获取父子组件实例对象，组件都有一个_uid属性
## 4、通过$refs获取实例
## 5、父组件来通过provider来提供变量,子组件通过inject来注入变量
## 6、可以通过eventBus，使用一个空vue实例绑定和监听事件
## 7、vuex管理
