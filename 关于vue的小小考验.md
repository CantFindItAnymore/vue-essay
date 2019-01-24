1.vue的双向绑定是什么原理？

2.vue的data为什么要写成return的形式？

3.history模式和hash模式

4.$.router和$.route的区别

5.vue的修饰符

6.vue单页面的优缺点

7.computed的实现方式





:grinning:下面揭晓答案↓

### 一. vue的双向绑定是什么原理？

vue采取数据劫持和发布-订阅者模式，通过Object.defineProperty（）来实现数据的双向绑定。

当数据变动时，发布消息给订阅者，触发相应的回调函数。

下面用js实现简单的双向绑定：

```javascript
<body>
  <div id="app">
    <input type="text" id="txt">
    <p id="show"></p>
	</div>
</body>
<script type="text/javascript">
  var obj = {}
  Object.defineProperty(obj, 'txt', {
    get: function () {
      return obj
    },
    set: function (newValue) {
      document.getElementById('txt').value = newValue
      document.getElementById('show').innerHTML = newValue
    }
  })
  document.addEventListener('keyup', function (e) {
    obj.txt = e.target.value
  })
</script>
```
------



### 二. vue的data为什么要写成return的形式？

------



### 三. history模式和hash模式

现代框架利用浏览器的两个特性：hash和history实现了**前端路由**。

关于SPA，我们知道，vue在跳转路由时更改了视图和URL，但不会重载页面。怎么做到的呢？↓



#### 1. 先说说hash

在**hash**中，#代表网页中的位置，#右边的字符代表位置信息。

```json
http://localhost：8080/cbuild/index.html#one
```

hash模式有以下特点：

**A.** 如上，就代表网页index.html的one位置。浏览器读取到这个URL时，会自动将one这个div放入到app这个div中，达到切换视图的效果。

**B.** 其次，http请求中不包含hash值。如上例代码，浏览器实际上发出的请求是 GET / index.html，而不包含#one。

**C.** 然后，改变URL的hash值，只会切换app这个div中的div→然后改变视图，并不会重新向服务器请求index.html。

**D.** 然后，每一次改变#会改变浏览器的访问历史，即在里面增加一个记录。点击回退按钮，可以回到上一个位置。

**E.** 然后，windows.location.hash能够读写URL的hash值。

**F.** ***onhashchange***

HTML5有一个新增的事件***onhashchange***，当hash值变化时，就会触发这个事件。（pushState()和replaceState()改变hash时不会触发）

hash模式的原理正是如此。栗子↓

```javascript
window.onhashchange = function(event){
  console.log(event.oldURL, event.newURL)
  let hash = location.hash.slice(1)
  document.body.style.color = hash
}
```

上面的代码通过hash变化来改变颜色。

**G. 关于404页面**

hash模式下，向后端请求时是不带hash的，所以就算hash值不匹配路由，后台也也不会返回404。那么页面会跳转到哪里呢？没错，空白页。怎么解决呢？:blonde_woman:↓

路由规则之一：路由是一条一条从上往下匹配的，若匹配成功，则不会再往下继续匹配。

因此，我们可以在前端router的最后加一条匹配所有的规则，并使用redirect使其跳转至你想让它去的地方。

```javascript
export default new Router({
  routes: [
    {
      path: '/',
      name: 'Home',
      component: Home
    },
    {
      path: '/swipe',
      name: 'Swipe',
      component: Swipe
    },
    {
      path: '*',
      redirect: '/404',
      name: '404',
      component: 404
    }
  ]
})
```


####  **2. history**

相对于hash模式，history模式给了前端更多的自由。要理解history，我们需要先了解几个概念：

1. 浏览器是怎么管理历史记录栈的。
2.  history的新特性。



##### 2.1 浏览器是怎么管理历史记录栈的

浏览器历史记录策略：

![](https://github.com/CantFindItAnymore/vue-essay/blob/master/imgs/o_3.png?raw=true)

由上图得到结论：

a.  浏览器有一个History栈。

b. 执行pushState函数会发生什么？

若当前指针在栈顶：压入设定的url至栈顶，同时修改当前指针 ，激活压入的url。

若当前指针在中间：清除当前指针之上的条目，然后压入设定的url至栈顶，同时修改当前指针 ，激活压入的url。

c.  当执行back，forward操作时，history栈大小并不会改变（history.length不变），仅仅移动当前指针的位置。



##### 2.2 history的新特性

在HTML5之前，**history**对象有这3个方法：

```
1. back()：相当于点击回退按钮。
2. forward()：相当于点击前进按钮。
3. go(Number)：根据参数加载某个具体的页面。参数是数字，代表要去的URL条目与当前条目的相对位置。
```

以上3个方法即浏览器的3个行为。可能有人会说没有跳转（go），其实在chorme中把鼠标放在前进按钮上就能看到当前的历史记录栈了。

------

HTML5 History API新增

​	2个方法：`history.pushState() `  和 `history.replaceState()`，

​	和1个事件：`window.onpopstate`。 

```json
history.pushState() // 向历史记录栈中添加条目。

history.replaceState() // 修改历史记录中的条目。

window.onpopstate() // 当历史记录栈中的激活条目变换时，就会触发popstate事件。
```

------

###### 2.2.1 **先谈popstate事件：**

popstate事件发生于浏览器前进后退时（可以类比于onClick，onLoad）。

注意触发条件必须是浏览器的行为，比如前进后退跳转 ，或者在js中调用back()，go()，foward()方法。

注意pushState()和replaceState()是不会触发popstate的。

注意页面初始化时没有popstate事件，此时可以用history.state取到状态对象state。

当历史记录栈中被激活的条目/是被pushState()添加的，或是被replaceState()修改过的，/则popstate事件/的事件对象event/的state属性/就是/当前条目的state对象的拷贝。

popstate事件的语法：

```json
window.onpopstate = funcRef //funcRef 是个函数名.
```

咱们举个例子，假如当前网页地址为http://example.com/example.html,则运行下述代码后: 

```javascript
window.onpopstate = function(event) {
  alert("location: " + document.location + ", state: " + JSON.stringify(event.state));
};
//绑定事件处理函数. 
history.pushState({page: 1}, "title 1", "?page=1");    
history.pushState({page: 2}, "title 2", "?page=2");    
history.replaceState({page: 3}, "title 3", "?page=3");
history.back(); // 弹出 "location: http://example.com/example.html?page=1, state: {"page":1}"
history.back(); // 弹出 "location: http://example.com/example.html, state: null
history.go(2);  // 弹出 "location: http://example.com/example.html?page=3, state: {"page":3}
```

分析：

1. 当前历史记录栈只有一个条目，就是http://example.com/example.html，条目索引为0。
2. 添加并激活一个历史记录条目 http://example.com/example.html?page=1，条目索引为1。
3. 添加并激活一个历史记录条目 http://example.com/example.html?page=2，条目索引为2。
4. 修改当前激活的历史记录条目 http://ex..?page=2 变为 http://ex..?page=3，条目索引为3，条目2你号没了 :smirk:。
5. 目前是条目3，回退到条目1。
6. 目前是条目1，回退到条目0，即http://example.com/example.html。
7. 目前是条目0，前进2步，到条目3。

总结：popstate就是history的原理所在。栗子↓

```javascript
history.pushState({color:'red'}, 'red', 'red'})

window.onpopstate = function(event){
  console.log(event.state)
  if(event.state && event.state.color === 'red'){
    document.body.style.color = 'red'
  }
}

history.back()
history.forward()
```

上面的代码通过回退又前进，触发了当前页面的popstate事件。然后在该事件中修改了颜色。

------



###### 2.2.2 **再谈pushState方法：**

该方法接收3个参数：

1. 状态对象（必填）state：

   是个js对象，这条是给popstate事件用的。注意state序列化后不能大于640KB，否则报错。若需要传大的对象，建议使用`sessionStorage` 以及 `localStorage` 。

2. 标题：就是个名字。

3. URL：

   新的URL地址，不能跨域。

   可以是绝对路径；也可以是相对路径，相对于当前URL。

举个栗子↓

```javascript
history.pushState({page: 1}, "title 1", "?page=1")
```

注意pushState()能但并非只能用于history模式，例如：

```javascript
history.pushState({}, "title 1", "#hash")
```

也就是说，state和URL都可以传值。一般history模式用state，hash用URL。

------

其实我们可以将pushState()与location.href=''比较，两者都是创建并激活一个条目。

pushState()的不同之处：

1. pushState()中的新URL可以是与当前URL同源的任意URL，都不会使页面重载。

   例如https://cn.vuejs.org/v2/guide/computed.html下，运行以下js：

   ```javascript
   history.pushState({page: 2}, "", "https://cn.vuejs.org/v2/guide")
   ```

   页面不会重载。

   那么怎么改变视图呢？在popstate事件中用ajax请求数据或运行其它js即可。

   ------

   而location.href是只修改了哈希值时才会保持同一个document，即不会重载。

   例如http://news.baidu.com下，运行以下js：

   ```json
   location.href='http://news.baidu.com#31'
   ```

   网页是不会重载的。而是在历史记录栈中添加并激活了一个条目。其他情况页面是会重载的。

2. 你可以为新的历史记录项关联任意数据。

   但是注意基于哈希值的方式，必须将所有相关数据编码到一个短字符串里，不能存储很多信息。 

3.  title是可以被浏览器使用的。



**关于404页面**：

history模式不怕前进后退跳转，怕刷新。

刷新时，如果后台没有配置，则会返回404。

因为刷新时浏览器会去请求服务器，并且是带着完整的URL，不像hash模式还不带哈希值一块玩的。

咋办，让后台给你配置一下所有的路径。这样跟hash模式比起来挺麻烦的，所以我们一般还是采取hash模式吧。

那history除了好看还有啥用？有的，人家也靠才华吃饭↓

当我们需要保存当前页面的阅读位置啊，滚动条位置啊这些东西时，可以用history模式实现。

因为pushState()方法里的state可以存储这些值。真叼~

------

再次提醒，

pushState()和replaceState()绝对不会触发hashchange事件，即使该方法改变了hash值。

前进后退跳转才会触发popstate事件，pushState()和replaceState()不会。

参考：https://blog.csdn.net/lla520/article/details/77894985

### 四. $.router和$.route的区别

------



### 五. vue的修饰符

.prevent

.stop

.self

.captrue

------

### 六. vue单页面的优缺点





