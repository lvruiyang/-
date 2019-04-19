## 1、new操作符具体干了什么 <br>
 (1)生成一个空对象var obj={}
 (2)设置原型链 obj._proto_=构造函数.prototype
 (3)改变this的指向并执行构造函数中的代码，
 (4)如果构造函数有返回值并且返回值类型为一个对象或一个函数，那么就优先返回原有的返回值，如果返回值是基本类型或无返回值默认返回新创建的新对象obj.
 
 ## 2、改变this指向常用的方法call,apply,bind <br>
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
   //这里有个问题,就是如图：多个call连续调用的时候，明明是函数，调用时却报错 <br>
   ![image](https://github.com/lvruiyang/myKnowledge/blob/master/images/1555581079(1).jpg)
   
   (2)apply用法跟call一样，除了参数必须是数组类型的，即函数名.apply(要绑定的上下文对象,[参数，参数])
   (3)bind的用法，就是返回一个绑定目标上下文对象的函数，即函数名.bind(要绑定的上下文，参数，参数);
   简易实现方法：
   ```
   Function.prototype.bindFn=function(context,...args){
   	var self=this;
	return function(newArgs){
		return self.apply(context,args.concat[newArgs]);	
	}
   }
   //另外因为bind函数返回的函数还可以当做构造函数使用，所以需要把上面的改写一下
   Function.prototype.bindFn2=function(context,...args){
   	var self=this;
	var fn= function(newArgs){
		return self.apply(this instanceof bindFn2 ? this : context,args.concat[newArgs]);	
	}
	//fn.prototype=Object.create(this.prototype);下面是Object.create的实现方法
	var emptyFn=function(){};
	emptyFn.prototype=this.protoytpe;
	fn.prototype=new emptyFn();
	return fn;
   }
   ```
 ## 3、this的指向问题
  (1)this的默认指向是，默认指向调用时所在的那个对象，如果没有的话，指向全局对对象window
  (2)箭头函数的绑定this。箭头函数绑定它定义时所在的那个对象，this指向不能更改，包括call和apply,另外箭头函数不能当做构造函数，内部也不能使用arguments对象。
  (3)call,apply.bind强制改变this.
  (4)new操作符默认改变this，指向构造函数的原型对象<br>
 ## 4、原型与原型链
  (1)原型的概念<br>
	js中万物皆对象,而所有的对象除了null，都有一个内部的_proto_属性，这就是对象的原型。而所有的函数都有一个prototype属性，这是函数的原型对象。
       函数同时具有_proto_属性与prototype原型对象,对象只有_proto_原型。<br>
  (2)原型链的概念<br>
    对象的原型_proto_默认指向它的构造函数的原型prototype,而构造函数的原型对象作为一个对象，也拥有自己_proto_,指向所有对象的构造函数Object,最终Object的原型_proto_指向null,这样形成指向链的就是原型链。
    实例对象读取自身属性时，现在自己内部查找，如果查找不到的话，就继续沿着原型链查找，直到找到属性为止，都找不到的话才返回undefined.
    in 操作符就是判断属性是否在实例对象的内部和原型链上存在。instanceof也是判断原型链上是否有该构造函数的原型，for in 遍历操作符能遍历出原型链上的属性。<br>
    ```
    //构造函数
    funciton Func(){}
    Func.prototype.constructor=Func;
    //实例对象
    var obj=new Func();
    obj.constructor=Func;
    //对象的原型链<br>
    obj._proto_ => Func.prototype => Function.prototype => Object.prototype => null;
    //构造函数的原型链
    Function.prototype => Object.prototype => null;
    ```
 ##5、js的继承
    继承说白了就是子类想要拥有父类对象和方法的使用权，要不把属性和方法复制到自己身上，要不可以通过原型链访问到。
    (1)原型继承
      就是Child.prototype= new Parent();
