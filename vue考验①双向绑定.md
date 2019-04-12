### vue的双向绑定是什么原理？

vue采取数据劫持和发布-订阅者模式，通过Object.defineProperty（）来实现数据的双向绑定。

当数据变动时，发布消息给订阅者，触发相应的回调函数。

下面用js实现简单的双向绑定：

```javascript
<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8">
  <meta name="viewport" content="width=device-width, initial-scale=1.0">
  <meta http-equiv="X-UA-Compatible" content="ie=edge">
  <title>Document</title>
</head>
<body>
  <div id="app">
    <input type="text" id="txt">
    <p id="show"></p>
    <button id="button">在p前加一个~</button>
  </div>

  <script type="text/javascript">
    var obj = {}
    Object.defineProperty(obj, 'txt', {
      get: function () {
        return document.getElementById('show').innerHTML
      },
      set: function (newValue) {
        document.getElementById('txt').value = newValue
        document.getElementById('show').innerHTML = newValue
      }
    })
    document.getElementById('txt').addEventListener('keyup', function (e) {
      obj.txt = e.target.value
    })
    document.getElementById('button').addEventListener('click', function () {
      var val = obj.txt
      obj.txt = '~' + val
    })
  </script>
</body>
</html>
```

分析（请先看完全文再回来看这个）：

我们的目的是让input和p的值动态一致，任何一方变化了另一方都会跟着变。

而通过Object.defineProperty定义的对象就相当于是一个独立的中央控制室R。假设变化的双方分别是A和B，A变化了告诉R，R就把变化后的值重新赋值给A和B。B变化也一样。就是这样~



set函数里的逻辑就是赋值的逻辑。在Object.defineProperty外部，我们去监听A，B的事件->回调函数里面改变R对象的值->导致set函数执行。就可以了。



反正核心就是，**去触发set函数**。



------

要理解上面的代码，我们先搞定 `Object.definePropety()`。

1. 如何定义一个对象？

   ```javascript
   var obj = new Object;  //obj = {}
   obj.name = "张三";  //添加描述
   obj.say = function(){};  //添加行为
   ```

   

2. 除了第一种添加属性的方式，还可以使用**Object.defineProperty**定义新属性或修改原有的属性。 

   **语法**：

   ```javascript
   Object.defineProperty(obj, prop, descriptor)
   ```

   **参数说明**：

   - obj：必需。目标对象  
   - prop：必需。需定义或修改的属性的名字 （字符串）
   - descriptor：必需。目标属性所拥有的特性  (对象)

   **返回值**：

   定义或修改后的对象obj。

   ```javascript
   var obj = {}
   var a = Object.defineProperty(obj, 'test', {
     value: 'rj love study' // 键只能写成value，下面详述
   })
   console.log(a)
   ```

   

3. 重点来了:last_quarter_moon_with_face::last_quarter_moon_with_face::last_quarter_moon_with_face::last_quarter_moon_with_face::last_quarter_moon_with_face::last_quarter_moon_with_face::last_quarter_moon_with_face::last_quarter_moon_with_face::last_quarter_moon_with_face::last_quarter_moon_with_face::last_quarter_moon_with_face::last_quarter_moon_with_face::last_quarter_moon_with_face::last_quarter_moon_with_face::last_quarter_moon_with_face::last_quarter_moon_with_face::last_quarter_moon_with_face::last_quarter_moon_with_face::last_quarter_moon_with_face::last_quarter_moon_with_face:

   针对对象的属性，我们可以设置一些特性，比如：

   - 属性的值。

   - 是否只读。
   - 当该对象某个属性被存取时，调用set/get方法。
   - 是否可枚举（可以被*`for..in`*或`Object.keys()`遍历）。
   - 是否可以被删除或修改。

   给对象的属性添加特性描述，目前提供两种形式：

   - **数据描述**
   - **存取器描述**。

4. **数据描述**

   当修改或定义对象的某个属性的时候，给这个属性添加一些特性。

   - **value**：设置属性的值。默认为undefined。
   - **writable**：值是否可以被重写 true/false。
   - **enumerable**：该属性是否可枚举 true/false。
   - **configurable**：该属性是否可被删除或修改 true/false。

   ```javascript
   var obj = {
       test:"hello"
   }
   //对象已有的属性添加特性描述
   Object.defineProperty(obj,"test",{
       configurable:true | false,
       enumerable:true | false,
       value:任意类型的值,
       writable:true | false
   });
   //对象新添加的属性的特性描述
   Object.defineProperty(obj,"newKey",{
       configurable:true | false,
       enumerable:true | false,
       value:任意类型的值,
       writable:true | false
   });
   ```

   

5. **存取器描述**

   当使用存取器描述属性的特性的时候，允许设置以下特性属性： 

   ```javascript
   var obj = {};
   Object.defineProperty(obj,"newKey",{
       get:function (){} | undefined,
       set:function (value){} | undefined
       configurable: true | false
       enumerable: true | false
   });
   ```

   *注意设置了get或set后，就不能使用value和writable属性了。

   *注意get和set并不是一定要共存。

   get函数用来定义->该属性->被外界获取值时->做的事情；

   set函数用来定义->该属性->被外界修改时->做的事情。

   ```javascript
   var obj = {};
   var initValue = 'hello';
   Object.defineProperty(obj,"newKey",{
       get:function (){
           //当获取值的时候触发的函数
           return initValue;    
       },
       set:function (value){
           //当设置值的时候触发的函数,设置的新值通过参数value拿到
           initValue = value;
       }
   });
   //获取值
   console.log( obj.newKey );  //hello
   
   //设置值
   obj.newKey = 'change value';
   
   console.log( obj.newKey ); //change value
   ```