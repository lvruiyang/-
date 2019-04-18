1、new操作符具体干了什么 <br>
 (1)生成一个空对象var obj={}
 (2)设置原型链 obj._proto_=构造函数.prototype
 (3)改变this的指向并执行构造函数中的代码，
 (4)如果构造函数有返回值并且返回值类型为一个对象或一个函数，那么就优先返回原有的返回值，如果返回值是基本类型或无返回值默认返回新创建的新对象obj.
 
 2、改变this指向常用的方法call,apply,bind <br>
  (1)call的用法：函数.call(要绑定的上下文对象，参数，参数)注意参数要用逗号分隔开。
   内部实现方法
   ```javascript
   Funciton.prototype.callFn=function(context,...args){
    //注意此处不能使用箭头函数，因为箭头函数的this会绑定它定义时所在的上下文对象，也就是此处的Funciton.prototype对象
    //首先context可以传任意值基本数据类型、对象、null,undefined,若果为null或undefined，context默认指向window对象，其他的都会默认转换成对象类型
    //如 1=>Number,'aa'=>String
     context=context?Object(context):window;
     //然后把函数添加到上下文对象对象中
     //这里有个问题，如何防止添加的函数名fn不予对象原有的属性名冲突，可以先判断属性名是否存在，es6中的Symbol非常适合解决此类命名冲突问题
     var fn=fnFactory(context);
     context[fn]=this;
     //然后执行函数,然后删除函数
     context.fn(...args)
     delete context[fn]
   }
   //首先判断 context中是否存在属性 fn，如果存在那就随机生成一个属性fnxx，然后循环查询 context 对象中是否存在属性 fnxx。如果不存在则返回最终值
   function fnFactory(context) {
	    var unique_fn = "fn";
     while (context.hasOwnProperty(unique_fn)) {
    	 unique_fn = "fn" + Math.random(); // 循环判断并重新赋值
     }
     return unique_fn;
   }
   ```
   //这里有个问题,就是如图：
   ![image](https://github.com/lvruiyang/myKnowledge/blob/master/images/1555581079(1).jpg)
   
   (2)apply用法跟call一样，除了参数必须是数组类型的，即函数名.apply(要绑定的上下文对象,[参数，参数])
   (3)bind的用法，就是
