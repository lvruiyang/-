## 1、new操作符具体干了什么 <br>
 (1)生成一个空对象var obj={}<br>
 (2)设置原型链 obj._proto_=构造函数.prototype<br>
 (3)改变this的指向并执行构造函数中的代码，<br>
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
    //构造函数<br>
    funciton Func(){}
    Func.prototype.constructor=Func;
    //实例对象<br>
    var obj=new Func();
    obj.constructor=Func;
    //对象的原型链<br>
    obj._proto_ => Func.prototype => Function.prototype => Object.prototype => null;
    //构造函数的原型链<br>
    Function.prototype => Object.prototype => null;
 ## 5、js的继承 [参考-JavaScript常用八种继承方案]（https://github.com/yygmind/blog/issues/7）
   继承说白了就是子类想要拥有父类对象和方法的使用权，要不把属性和方法复制到自己身上，要不可以通过原型链访问到。
   (1)原型链继承<br>
      就是Child.prototype= new Parent();
      缺点：父类上的引用类型属性被所以实例共享，容易被篡改，并且不能向父类构造函数中传值。<br>
   (2)借用构造函数继承<br>
    
    ```
    	function Parent(){}
	function Child(){
	        Parent.call(this);//通过调用call方法，把父类构造函数中的属性复制到子类实例上。
	}
	
    ```
       缺点：这种方法只能继承父类实例上的属性，不能继承父类原型链上的属性，并且父类上的方法定义在构造函数内部的话，无法实现复用，每个子类都有父类实例函数的副本，影响性能。<br>
 (3)组合继承<br>
 ```
        function Parent(){
     	}
	function Child(){
	        Parent.call(this);//通过调用call方法，把父类构造函数中的属性复制到子类实例上。
	}
	Child.prototype == new Parent();
```
	缺点：
	第一次调用SuperType()：给SubType.prototype写入两个属性name，color。
	第二次调用SuperType()：给instance1写入两个属性name，color。
	实例对象instance1上的两个属性就屏蔽了其原型对象SubType.prototype的两个同名属性。所以，组合模式的缺点就是在使用子类创建实例对象时，其原型中会存在两份相同的属性/方法。
  (4)寄生组合式继承 <br>
  ```
        function Parent(){
        }
	function Child(){
	        Parent.call(this);//通过调用call方法，把父类构造函数中的属性复制到子类实例上。
	}
	Child.prototype == Object.create(Parent.prototype);
	Child.prototype.constructor=Child;//把子类原型链构造函数指回
  ```
  (5)混入的方式实现继承，就是多次调用call或apply，将要继承的属性复制到子类实例上。
  (6)Es6实现继承
 ## 6、ES6的继承
 ES6中的继承链有两条
 (1)子类可以继承父类实例上和原型上的属性和方法 <br>
 	child._proto_ => Child.prototype => Parent.prototype => Function.prototype => Object.prototype => null <br>
	调用super().执行Parent.call(this),继承父类实例属性 <br>
 (2)子类构造函数可以访问父类构造函数上的静态方法
 	Child._proto_ => Parent <br>
ES5实现方法：
```
    function Parent(){
     }
     function Child(){
	Parent.call(this);//通过调用call方法，把父类构造函数中的属性复制到子类实例上。
      }
      Child.prototype == Object.create(Parent.prototype);
      Child.prototype.constructor=Child;//把子类原型链构造函数指回
      Child._proto_=Parent;//或者使用Object.setPrototypeOf(Child, Parent)
```
## 7、js的内存空间
(1)js中内存空间分为堆内存与栈内存。
   1、基本类型 --> 保存在栈内存中，因为这些类型在内存中分别占有固定大小的空间，通过按值来访问。基本类型一共有6种：Undefined、Null、Boolean、Number 、String和Symbol <br>
   2、引用类型 --> 保存在堆内存中，因为这种值的大小不固定，因此不能把它们保存到栈内存中，但内存地址大小是固定的，因此保存在堆内存中，在栈内存中存放的只是该对象的访问地址。当查询引用类型的变量时， 先从栈中读取内存地址， 然后再通过地址找到堆中的值。对于这种，我们把它叫做按引用访问。<br>
   基本类型的赋值都是值复制，引用类型的赋值都是内存地址的复制，可以有多个变量指向同一个对象。<br>
   ```
   var a = 20;
   var b = a;
   b = 30;
   console.log(a)//因为基本数据类型是值复制，因此两者之间互不影响，所以打印20
   var a = { name: '前端开发' }
   var b = a;
   b.name = '进阶';
   console.log(a.name)//输出'进阶'，因为引用数据类型赋值是内存地址赋值，同一内存地址指向同一个对象，a和b操作的都是一个对象;
   //一个很好的面试题
   var a = {n: 1};
   var b = a;
   a.x = a = {n: 2};
   
   a	// --> {n:2}
   b	// --> {n:1,x:n: 2}
   1、优先级。.的优先级高于=，所以先执行a.x，堆内存中的{n: 1}就会变成{n: 1, x: undefined}，改变之后相应的b.x也变化了，因为指向的是同一个对象。
   2、赋值操作是从右到左，所以先执行a = {n: 2}，a的引用就被改变了，然后这个返回值又赋值给了a.x，需要注意的是这时候a.x是第一步中的{n: 1, x: undefined}那个对象，其实就是b.x，相当于b.x = {n: 2}
   ```
  ## 8、深拷贝与浅拷贝
  对于基本类型来说，拷贝是值复制，因此两者之间互不影响。对于引用数据类型来说，拷贝是地址拷贝，相同的内存地址指向同一个对象。<br>
  (1)浅拷贝就是基本数据类型进行值复制，引用数据类型进行内存地址拷贝。拷贝过后的对象操作基本数据类型时互不影响，操作引用数据类型时，因为指向的是同一个对象，因此会影响到拷贝前的对象.<br>
  ES6中的Object.assign、...拓展运算符都是属于浅拷贝。<br>
 (2)深拷贝就是基本数据类型进行值复制，引用数据类型会把内部所有的引用数据类型属性，重新创建一个堆内存空间放入旧值，然后使用新的内存地址，拷贝前后内存地址发生变化，指向两个属性相同但内存地址不同的对象，各自操作不会相互影响。<br>
   最简单实现：JSON.parse(JSON.stringify(Object)),该方法缺点：
	1、会忽略 undefined
	2、会忽略 symbol
	3、不能序列化函数
	4、不能解决循环引用的对象
	5、不能正确处理new Date()
	6、不能处理正则
实现一个深拷贝方法:
```
funciton deepCopy(target){
	let child;
	if(!target || typeof target !=Object){
		return target;//拷贝基本数据类型的话就直接返回自身
	}else{
		//如果是对象的话，判断是数组还是对象
		child=Object.prototype.toString(target) == '[object Array]':[]:{};
		for(let key in target){
			child[key] = deepCopy(target[key])
		}
		return child;
	}
}
```
## 9、js内存泄漏
	(1)内存泄漏：就是在不在需要后，变量依然保留在内存中，内存空间没有释放，没有被js垃圾回收机制清除的情况。
	(2)js垃圾回收机制：
		1、标记清除(最常用)：标记清除算法将“不再使用的对象”定义为“无法到达的对象”。即从根部（在JS中就是全局对象）出发定时扫描内存中的对象，凡是能从根部到达的对象，保留。那些从根部出发无法触及到的对象被标记为不再使用，稍后进行回收。
		2、引用计数:当对一个变量的引用为0时，才会被回收。
	ES6 新出的两种数据结构：WeakSet 和 WeakMap，表示这是弱引用，它们对于值的引用都是不计入垃圾回收机制的。他们只接受对象为键值，不能遍历。
## 10、变量作用域与变量提升
	(1)js变量作用域分为全局作用域与局部作用域（一般指函数作用域）
	(2)变量的访问查找:变量的访问只能往上进行查找，即当在局部作用域里找不到时，继续向全局作用域里找，再找不到的话就报错：Uncaught ReferenceError: 变量 is not defined。
	(3)变量声明提升：在非严格模式下，变量在声明时会自动将声明提升到当前所在作用域的顶部，而赋值语句留在当前位置。而函数声明则会自动提升到所在作用域的最顶端。函数参数会形成一个局部作用域。
	```
	var a=1;
	function test(a){
		console.log(a)//输出：undefined
		a=3;
		console.log(a)//输出：3
	}
	test()
	console.log(a)//输出：1
	
	```
