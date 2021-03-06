# 手写源码
> [设计模式相关](#2-设计模式相关)  
> [正则相关](#9-正则相关)  
> [数组相关](#10-数组相关)

##### 1. jQuery框架架构原理
  ```javascript
      var jQuery = window.$ = function(sel){
          return new jQuery.fn.init(sel);
      };
      jQuery.fn = jQuery.prototype = {
        init: function(sel){
            //把dom对象变成 jQuery对象
            Array.prototype.push.apply(this, document.querySelectorAll(sel));
            return this;
        },
        name: 'jQuery'
      }
      /*jQuery 的方式是通过原型传递解决问题，把 jQuery 的原型传递给jQuery.prototype.init.prototype
      所以通过这个方法生成的实例 this 所指向的仍然是 jQuery.fn，所以能正确访问 jQuery 类原型上的属性与方法*/
      jQuery.fn.init.prototype = jQuery.fn;

      //添加扩展方法
      jQuery.extend = jQuery.fn.extend = function(){
          for(let p in arguments[0]){
              this[p] = arguments[0][i];
          }
      }
      //$('#logo-default')
        init {0: div#logo-default, length: 1}
  ```
##### 2. 设计模式相关
- 发布-订阅模式是面向调度中心编程的，而观察者模式则是面向目标和观察者编程的。发布订阅模式相比观察者模式多了个事件通道，订阅者和发布者不是直接关联的
> ###### 1. 自定义事件（发布订阅模式）
  ```javascript
      // 自定义事件
      class EventObserver{
          constructor(){
              //不同type的函数的数组
              this.handleFun = [];
          }
          //添加type类型的函数
          add (...args){
              let [type, func] = args;
              this.handleFun[type] ? null : this.handleFun[type] = [];
              if(this.handleFun[type].includes(func)){
                  return;
              }else{
                  this.handleFun[type].push(func);
              }        
          }
          //移除type类型的函数
          remove (...args){
              let [type, func] = args;
              try{
                  let ind = this.handleFun[type].indexOf(func);
                  if(ind === -1){
                      throw new Error('function 不存在')
                  }else{
                      this.handleFun[type].splice(ind, 1);
                  }
              }catch(err){
                  console.log(err);
              }
          }
          emit (...args){
              //可能是多个参数，type或者指定的出发函数
              try{
                  let [type, func] = args;
                  if(func){
                      this.handleFun[type].forEach(item=>{
                          item();
                      }) 
                  }else{
                      let ind = this.handleFun[type].indexOf(func);
                      if(ind === -1){
                          throw new Error('function 不存在')
                      }else{
                          this.handleFun[type][index]();
                      } 
                  }

              }catch(err){
                  console.log(err);
              }
          }
          once (...args) {
              //就是在触发后，移除它
              this.emit(...args)?this.remove(...args):null;
          }
      }
      //用法示例
      let btn1 = document.getElementById("test1");
      let eventObs = new EventObserver();

      function handler1(){
          alert("handler1:");
      };

      function handler2(){
          alert("handler2:");
      };

      //订阅       
      eventObs.add("msgType", handler1);
      eventObs.add("msgType", handler2);


      //点击按钮触发发布消息（触发事件）    
      btn1.addEventListener("click",  function(){
          eventObs.emit("msgType");
      }, false);

  ```
> ###### 2. 观察者模式
  ```javascript
      //观察者模式
      //主题
      class Subject{
          constructor(){
              this._state = 0;
              this._observers = [];
          }

          getState(){
              return this._state;
          }

          setState(val){
              this._state = val;
              this.notifyAllObservers(this._state);
          }
          attch(obs){
              this._observers.push(obs);
          }
          notifyAllObservers(value){
              this._observers.forEach(item=>{
                  item.update(value);
              })
          }

      }
      //订阅
      class Observer{
          constructor(sub){
              this._subject = sub;
              this._subject.attch(this);
          }
          update(val){
              console.log('主题更新了:', val);
          }
      }
      let sub = new Subject();
      let ob1 = new Observer(sub);
      let ob2 = new Observer(sub);
      sub.setState(4);  
  ```
> ###### 3. 单例模式
  ```javascript
      //只生成一个实例
      class SingleObject{
          constructor(){
              this.instance = null;
          }
          login(){ 
          //... 
          }
          static getInstance(){
              if(!this.instance){
                  this.instance = new SingleObject();
              }        
              return this.instance
          }
      }
      
      let obj = SingleObject.getInstance();
  ```
##### 4. 函数柯里化
  ```javascript
      function curry(fn){
          let _args = [];   //收集参数
          let _callback = (...args) => {
              _args.push(...args);
              return _callback; //每次返回函数连续调用
          }
          //利用toString隐式转换的特性，当最后执行时隐式转换，并计算最终的值返回
          _callback.toString = function(){
              return fn.apply(null, _args); 
          }
          return _callback;
          
      }
      //使用
      let add = (...args) => {
          return args.reduce((a,b)=>a+b)
      }
      let addC = curry(add);
      addC(1)(2)(3)(4)(5,6,7)   // ƒ 28
  ```
##### 5. 函数防抖、函数节流
> 1. 函数防抖
>> - 概念：触发回调函数后，会在一定时间之后执行，如果在这段时间再次触发，那么重新开始计时;
>> - 场景：给按钮加函数防抖防止表单多次提交。判断scroll是否滑到底部，滚动事件+函数防抖
> 2. 函数节流
>> - 概念：触发回调函数后，在一定时间段内，函数只执行一次，即使多次触发，也是只执行一次；
>> - 场景：游戏中的刷新率
  ```javascript
    function pShake(fn,time){
        let timer = null;
        return function(...args){
            if(timer){
                clearTimeout(timer);
            }
            timer = setTimeout(function(){
                fn.apply(null, args);
            },time);
        }
    }
    
    function rottle(fn,time){
        let flag = true;
        return function(...args){        
            if(!flag) return;
            flag = !flag;
            setTimeout(function(){
                fn.apply(null, args);
                flag = true;
            },time);
        }
    }
    //使用
      function test (...args){
        console.log('测试', ...args);
      }
      let pshake = pShake(test, 1000);
      pshake（1,2,3,4）   // 测试 1 2 3 4;

  ```
##### 6. call,apply,bind的封装实现
> 1. call 改变了this,然后把调用的函数在当前上下文中执行了一遍；参数是单个的
> 2. apply 参数是个数组
> 3. bind
> 4. callee是一个指针，指向拥有这个argument对象的函数
> 5. caller这个属性保存着调用当前函数的函数的引用


  ```javascript
    Function.prototype.call = function(context,...args){
        if(!context){
           context = window;
        }
        let symbol = Symbol();
        context[symbol] = this;
        let fn = context[symbol](...args);
        delete context[symbol];
        return fn;
    } 
    
    Function.prototype.apply = function(context, args){
        if(!context){
           context = window;
        }
        let symbol = Symbol();
        context[symbol] = this;
        let fn = context[symbol](...args);
        delete context[symbol];
        return fn;        
    }
    Function.prototype.bind = function(context){
        if(!context){
           context = window;
        }
        let self = this;
        return function(...args){
            self.apply(context, args)
        }
    }
  ```
##### 7. new的实现
> 1. 创建一个空对象，把__proto__指向构造函数的原型，用apply获取构造函数的实例的方法，属性。返回这个对象。
  ```javascript
      function new(fn){
          return function(...args){
              let obj = {};
              obj.__proto__ = fn.prototype;
              fn.apply(obj,args);
              return obj;
          }
      }
  ```
##### 8. 深拷贝
  ```javascript
      function deepCopy(obj,newObj={}){
          for(let p in  obj){
              if(obj[p]&&typeof obj[p] === 'object'){
                  newObj[p] = obj[p].contructor === Object?deepCopy(obj[p]):deepCopy(obj[p],[])
              }else{
                  newObj[p] = obj[p];
              }
          }
          return newObj;
      }
  ```
##### 9. 正则相关
> ###### 1. 查找字符串中出现最多的字符和个数
>> 1. 捕获组：这种引用既可以是在正则表达式内部，也可以是在正则表达式外部。
>>> - 普通捕获组：`(Expression)`  命名捕获组：`(?<name>Expression)`
>>> - $0表示的是整个引用
>> 2. 反向引用: 正则表达式中，对前面捕获组捕获的内容进行引用，称为反向引用
  ```javascript
      function mostWord(str){
          //'akdfkladksd'
          let char = '';
          let num = 0;
          let newStr = str.split("").sort().join('');  //"aadddfkkkls";
          newStr.replace(/(\w)\1+/g,($0,$1)=>{
              if(num < $0.length){
                  char = $1;
                  num = $0.length;
              }
              return undefined;
          })
          return `字符最多的是${char}，出现了${num}次`;
      }
      let str = 'akdfkladksd';
      mostWord(str);  //"字符最多的是d，出现了3次"
  ```
> ###### 2. 驼峰命名
>> 1. 掌握replace，第二个参数如果是函数内参数具体代表的含义；
  ```javascript
      function hump(str){
          let reg = /\-(\w{1})/g;
          return str.replace(reg, function(a,b,c,d){
              //a表示的正则匹配的串
              //b表示第一个分组的值
              //c表示匹配串的指针
              //d表示整个串
              return b.toUpperCase();
          })
      }
      var s1 = "get-element-by-id"
      hump(s1)  // "getElementById"
  ```
> ###### 3. 处理cookies
>> 1. match	找到一个或多个正则表达式的匹配。依赖于 regexp 是否具有标志 g。
>>> - 没g，执行一次匹配。如果没有找到任何匹配的文本， match() 将返回 null。否则，它将返回一个数组，其中存放了与它找到的匹配文本有关的信息。
>> 2. search 找到匹配返回其下标，没有则返回-1
>> 3. reg.exec(string) 有返回数组，否则返回null [匹配的整串，分组匹配，index, input];
  ```javascript
      function getCookie(name){
          let reg = new RegExp("(^| )"+name+"=([^;]*)(;|$)",'g');
          return reg.exec(document.cookie)[2];
      }
      getCookie('username')  // 'jingyuan'
  ```
> ###### 4. ?=n	 ?!n
>> 1. ?=n	匹配任何其后紧接指定字符串 n 的字符串。
>> 2. ?!n	匹配任何其后没有紧接指定字符串 n 的字符串。
  ```javascript
      var reg = /^(?=[A-z]{1})(?=.*[\d])(?=.*[\w])(?=.*[!@#$])[\d\w!@#$]{5}$/i
      reg.test('A2!s2')  // true
  ```
  ##### 10. 数组相关
  > ###### 1. 数组去重
  ```javascript
      //ES6 set的key不能重复
      [...new Set(arr)]
      Array.from(new Set(arr))

      //ES5 filter去重 
      function uniq(arr){
          return arr.filter((item,index) => {
              return arr.indexOf(item) === index;
          })
      }

      //ES5 while 
      function uniq(arr){
          let newarr = [];
          while(arr.length){
              let cur = arr.shift();
              arr.includes(cur) ? null: newarr.push(cur);
          }
          return newarr;
      }

  ```
> ###### 1. [数组方法](https://www.runoob.com/jsref/jsref-obj-array.html)的实现
>> 1. array.filter(function(currentValue,index,arr), thisValue) //返回符合条件的数组
  ```javascript
      Array.prototype._filter = function(fn){
          let result = [];
          for (let i = 0; i < this.length; i++) {
              let res = fn(this[i],i,this);
              if(res){ result.push(this[i])}
          }
          return result;
      }
  ```
>> 2. 当数组中的元素在测试条件时返回 true 时, find() 返回符合条件的元素，之后的值不会再调用执行函数。
如果没有符合条件的元素返回 undefined
  ```javascript
      Array.prototype._find = function(fn){
          let result = undefined;
          for (let i = 0; i < this.length; i++) {
              let backRes = fn(this[i],i,this);
              if(backRes){
                  result = this[i];
                  break;
              }
          }
          return result;
      }
  ```
>> 3. forEach() 方法用于调用数组的每个元素，并将元素传递给回调函数。 返回值:    undefined  
  ```javascript
      Array.prototype._forEach = function(fn){
          for (let i = 0; i < this.length; i++) {
              fn(this[i],i,this);
          }
          return undefined;
      }  
  ```
>> 4. map()返回一个新数组，数组中的元素为原始数组元素调用函数处理后的值。不改变数组的长度
  ```javascript
      Array.prototype._map = function(fn){
          let result = [];
          for (let i = 0; i < this.length; i++) {
              let res = fn(this[i],i,this);
              result.push(res);
          }
          return result;
      } 
  ```
>> 5. array.reduce(function(total, currentValue, currentIndex, arr), initialValue)   返回计算结果
  ```javascript
      Array.prototype._reduce = function(fn, initialValue=0){
          let total = initialValue;
          for (let i = 0; i < this.length; i++) {
              total = fn(total, this[i], i)
          } 
          return total;
      }
  ```

