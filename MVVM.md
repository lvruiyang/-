# MVVM的实现机制
  其实就是利用Object.defineProperty方法，给属性绑定get、set方法，在数据改变时触发set方法，从而调用更新函数。在以后的版本中，改成使用ES6中的Proxy方法。
  (1)、Object.defineProperty方法的使用方法：
  Object.defineProperty(obj对象, prop//对象属性, {
    configurable://当且仅当该属性的 configurable 为 true 时，该属性描述符才能够被改变，同时该属性也能从对应的对象上被删除。默认为 false，
    enumerable://当且仅当该属性的enumerable为true时，该属性才能够出现在对象的枚举属性中。默认为 false，
    value:属性的值，
    writable：当且仅当该属性的writable为true时，value才能被赋值运算符改变。默认为 false。
    或者
    get:读取属性时触发,
    set:设置属性值时触发
  })
  
