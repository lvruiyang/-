# MVVM的实现机制
  其实就是利用Object.defineProperty方法，给属性绑定get、set方法，在数据改变时触发set方法，从而调用更新函数。在以后的版本中，改成使用ES6中的Proxy方法。
  (1)、Object.defineProperty方法的使用方法：
