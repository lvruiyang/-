# MVVM的实现机制
  其实就是利用Object.defineProperty方法，给属性绑定get、set方法，在数据改变时触发set方法，从而调用更新函数。在以后的版本中，改成使用ES6中的Proxy方法。<br>
  (1)、Object.defineProperty方法的使用方法：<br>
  Object.defineProperty(obj对象, prop对象属性, {<br>
      configurable://当且仅当该属性的 configurable 为 true 时，该属性描述符才能够被改变，同时该属性也能从对应的对象上被删除。默认为 false，<br>
      enumerable://当且仅当该属性的enumerable为true时，该属性才能够出现在对象的枚举属性中。默认为 false，<br>
      value:属性的值，<br>
      writable：当且仅当该属性的writable为true时，value才能被赋值运算符改变。默认为 false。<br>
      或者<br>
      get:读取属性时触发,<br>
      set:设置属性值时触发<br>
  })<br>
  ## 1、数据订阅Observe：
  ```javascript
  class Observe{
    constructor(data){
      this.walk(data)
    }
    walk(data){
      if(!data || typeof data !='Object'){
        return;
      }
      Object.keys(data).map(key=>{
        this.defineObserve(data,key,data[key])
      })
    }
    defineObserve(vm,key,value){
      this.walk(value);
      let dep=new Dep();
      Object.defineProperty(data,key,{
        configurable: true,
            enumerable: true,
            get(){
              if(Dep.target){
                dep.addSub(Dep.target);
              }
              return value
            },
            set(val){
              if(val==value){
                return;
              }
              value=val;
              dep.notify()
            }
      })
    }
}
```
## 2、数据订阅管理器
```javascript
class Dep{
	constructor(){
		this.subs=[]
	}
	addSub(target){
		this.subs.push(target)
	}
	notify(){
		this.subs.map(item=>{
			item.update.apply(null,arguments)
		})
	}
}
  ```
## 3、观察者Watcher
   ```javascript
class Watcher{
	constructor(vm,key){
		watcherCount++;
		this.vm=vm;
		this.data=key;
		this.get();
	}
	get(){
		Dep.target=this;
		this.vm[this.data];
		Dep.target=null;
	}
	update(){
		console.log('执行更新函数')
		console.log(arguments[0]+"***"+arguments[1])
	}
}
   ```
   ## 4、模板解析compile
   ```javascript
function Compile(el, vm) {
    this.vm = vm;
    this.el = document.querySelector(el);
    this.fragment = null;
    this.init();
}

Compile.prototype = {
    init: function () {
        if (this.el) {
            this.fragment = this.nodeToFragment(this.el);
            this.compileElement(this.fragment);
            this.el.appendChild(this.fragment);
        } else {
            console.log('Dom元素不存在');
        }
    },
    nodeToFragment: function (el) {
        var fragment = document.createDocumentFragment();
        var child = el.firstChild;
        while (child) {
            // 将Dom元素移入fragment中
            fragment.appendChild(child);
            child = el.firstChild
        }
        return fragment;
    },
    compileElement: function (el) {
        var childNodes = el.childNodes;
        var self = this;
        [].slice.call(childNodes).forEach(function(node) {
            var reg = /\{\{(.*)\}\}/;
            var text = node.textContent;

            if (self.isElementNode(node)) {  
                self.compile(node);
            } else if (self.isTextNode(node) && reg.test(text)) {
                self.compileText(node, reg.exec(text)[1]);
            }

            if (node.childNodes && node.childNodes.length) {
                self.compileElement(node);
            }
        });
    },
    compile: function(node) {
        var nodeAttrs = node.attributes;
        var self = this;
        Array.prototype.forEach.call(nodeAttrs, function(attr) {
            var attrName = attr.name;
            if (self.isDirective(attrName)) {
                var exp = attr.value;
                var dir = attrName.substring(2);
                if (self.isEventDirective(dir)) {  // 事件指令
                    self.compileEvent(node, self.vm, exp, dir);
                } else {  // v-model 指令
                    self.compileModel(node, self.vm, exp, dir);
                }
                node.removeAttribute(attrName);
            }
        });
    },
    compileText: function(node, exp) {
        var self = this;
        var initText = this.vm[exp];
        this.updateText(node, initText);
        new Watcher(this.vm, exp, function (value) {
            self.updateText(node, value);
        });
    },
    compileEvent: function (node, vm, exp, dir) {
        var eventType = dir.split(':')[1];
        var cb = vm.methods && vm.methods[exp];

        if (eventType && cb) {
            node.addEventListener(eventType, cb.bind(vm), false);
        }
    },
    compileModel: function (node, vm, exp, dir) {
        var self = this;
        var val = this.vm[exp];
        this.modelUpdater(node, val);
        new Watcher(this.vm, exp, function (value) {
            self.modelUpdater(node, value);
        });

        node.addEventListener('input', function(e) {
            var newValue = e.target.value;
            if (val === newValue) {
                return;
            }
            self.vm[exp] = newValue;
            val = newValue;
        });
    },
    updateText: function (node, value) {
        node.textContent = typeof value == 'undefined' ? '' : value;
    },
    modelUpdater: function(node, value, oldValue) {
        node.value = typeof value == 'undefined' ? '' : value;
    },
    isDirective: function(attr) {
        return attr.indexOf('v-') == 0;
    },
    isEventDirective: function(dir) {
        return dir.indexOf('on:') === 0;
    },
    isElementNode: function (node) {
        return node.nodeType == 1;
    },
    isTextNode: function(node) {
        return node.nodeType == 3;
    }
}
   ```
