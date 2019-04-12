### vue的修饰符

**注意：**修饰符可以同时使用多个,但是可能会因为顺序而有所不同。 顺序是**从左往右~** 

#### 1. 表单修饰符

1. **.lazy**

   ```javascript
   <div>
      <input type="text" v-model.lazy="value">
      <p>{{value}}</p>
   </div>
   ```

   光标离开input的时候才更新视图，取消实时性。

2. **.trim**

   ```javascript
   <input type="text" v-model.trim="value">
   ```

   过滤首尾的空格。

3. **.number**

   如果你先输入数字，那它就会限制你输入的只能是数字。 

   如果你先输入字符串，那它就相当于没有加.number 。

#### 2. 事件修饰符

1. **.stop**

   由于事件冒泡的机制，我们给元素绑定点击事件的时候，也会触发父级的点击事件。 

   ```javascript
   <div @click="shout(2)">
     <button @click="shout(1)">ok</button>
   </div>
   
   //js
   shout(e){
     console.log(e)
   }
   //1
   //2
   ```

   ```javascript
   <div @click="shout(2)">
     <button @click.stop="shout(1)">ok</button>
   </div>
   //只输出1
   ```

2. **.prevent**

   用于阻止事件的默认行为，例如，当点击提交按钮时阻止对表单的提交。

   相当于调用了event.preventDefault()方法。 

   ```javascript
   <!-- 提交事件不再重载页面 -->
   <form v-on:submit.prevent="onSubmit"></form>
   ```

3. **.self**

   跟.stop有点类似。

   ```javascript
   <div class="blue" @click.self="shout(2)">
     <button @click="shout(1)">ok</button>
   </div>
   ```

   ```javascript
   <div @click="shout(2)">
     <button @click.stop="shout(1)">ok</button>
   </div>
   //只输出1
   ```

   上面的两段代码是一样的效果。

   所以，.self的作用是只有当前元素才能触发当前元素绑定的事件。

4. **.once**

   只能用一次，绑定了事件以后只能触发一次，第二次就不会触发。 

   ```javascript
   //键盘按坏都只能shout一次
   <button @click.once="shout(1)">ok</button>
   ```

5. **.captrue**

   emmmm，先补充一下冒泡机制的知识点。

   ![冒泡机制](G:\myRoadOfWeb\vue\我的文章\imgs\冒泡机制.png)

   假如我们点击一个div, 实际上是先点击document，然后点击事件传递到div,而且并不会在这个div就停下，div有子元素就还会向下传递，最后又会冒泡传递回document 。

   默认的呢，事件触发是从目标开始往上冒泡。 当我们加了这个.capture以后呢，我们就反过来了，事件触发从包含这个元素的顶层开始往下触发。 

   ```javascript
   <div @click.capture="shout(1)">
     obj1
     <div @click.capture="shout(2)">
       obj2
       <div @click="shout(3)">
         obj3
         <div @click="shout(4)">
           obj4
         </div>
       </div>
     </div>
   </div>
   // 1 2 4 3 
   ```

   从上面这个例子我们点击obj4的时候，就可以清楚地看出区别，obj1，obj2在捕获阶段就触发了事件，因此是先1后2，后面的obj3，obj4是默认的冒泡阶段触发，因此是先4然后冒泡到3~ 

6. **.passive**

   当我们在监听元素滚动事件的时候，会一直触发onscroll事件，在pc端是没啥问题的，但是在移动端，会让我们的网页变卡，因此我们使用这个修饰符的时候，相当于给onscroll事件整了一个.lazy修饰符 。

   ```javascript
   <!-- 滚动事件的默认行为 (即滚动行为) 将会立即触发 -->
   <!-- 而不会等待 `onScroll` 完成  -->
   <!-- 这其中包含 `event.preventDefault()` 的情况 -->
   <div v-on:scroll.passive="onScroll">...</div>
   ```

7. **.native**

   我们经常会写很多的小组件，有些小组件可能会绑定一些事件，但是，像下面这样绑定事件是不会触发的 .

   ```javascript
   <My-component @click="shout(3)"></My-component>
   ```

   必须使用.native来修饰这个click事件（即<My-component @click.native="shout(3)"></My-component>），可以理解为该修饰符的作用就是把一个vue组件转化为一个普通的HTML标签， 注意：**使用.native修饰符来操作普通HTML标签是会令事件失效的** 。

#### 3. 鼠标按键修饰符

刚刚我们讲到这个click事件，我们一般是会用左键触发，有时候我们需要更改右键菜单啥的，就需要用到右键点击或者中间键点击，这个时候就要用到鼠标按钮修饰符 

1. **.left** 左键点击 
2. **.right** 右键点击 
3. **.middle** 中键点击 

```javascript
<button @click.right="shout(1)">ok</button>
```

#### 4. 键值修饰符

1. **.keyCode**

   如果不用keyCode修饰符，那我们每次按下键盘都会触发shout，当我们想指定按下某一个键才触发这个shout的时候，这个修饰符就有用了，具体键码查看[键码对应表](https://zhidao.baidu.com/question/266291349.html) 

   ```javascript
   <input type="text" @keyup.keyCode="shout(4)">
   ```

   为了方便我们使用，vue给一些常用的键提供了别名 :

   ```javascript
   //普通键
   .enter 
   .tab
   .delete //(捕获“删除”和“退格”键)
   .space
   .esc
   .up
   .down
   .left
   .right
   
   //系统修饰键(不能单独使用，必须配合其他键或鼠标)
   .ctrl
   .alt
   .meta
   .shift
   ```

   

2. **.exact**

   

#### 5. v-bind修饰符（实在不知道叫啥名字）

1. **.sync**
2. **.prop**
3. **.camel**

