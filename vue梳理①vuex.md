---
title: 'vue进阶①vuex'
date: 2018-09-22 14:07:24
tags:
- vue
- 框架
categories:
- 前端
- vue进阶
---

##### 1. 状态管理模式

先举个栗子：

```json
<template>
	<div @click='add'>{{count}}</div>
</template>

<script>
export default {
  data () {
    return {
    	count: 1
		}
   }

	methods: {
    add () {
     	this.count++
    }
	}
}
</script>
```

笼统的说，就是 状态映射在视图上，视图上的交互触发方法，方法又改变状态。这是一种单向数据流的思想。

vuex即是一个全局的状态管理，所有组件都能使用。

我们再举一个例子：

```json
const store = new Vuex.Store({
  state: {
    count: 0
  },
  mutations: {
    increment (state) {
      state.count++
    }
  }
})
```

```json
store.commit('increment')
console.log(store.state.count) // -> 1
```

new Vuex表示创建一个vuex实例，store是vuex的一个核心方法，可以理解为创建一个仓库，里面有state，mutations，actions，getters，modules等对象。需要注意的是一个项目只能存在一个store实例。

另外，vuex和单纯的全局对象是有区别的：

① vuex是响应式的。意思就是当状态（state）发生变化时，相应的组件能在第一时间更新数据。 

② 若想改变vuex的状态，只能显式地提交（commit）mutation （触发方法，方法去改变状态）。这是为了能更加明确地追踪到状态的变化。



##### 2. state详解

vuex使用单一状态树，一个对象就包含了所有的状态（state）。因此它是整个项目的唯一数据源。那么如何在vue组件中获取vuex的状态（state）呢？

最好的办法是通过在根组件中注册store，store实例会注入到所有的子组件中，子组件（尽量在computed中）使用this.$store即可访问到。

```json
import store from './store/store'
// 在根组件中注册store
new Vue({
  el: '#app',
  store,
  components: { App },
  template: '<App/>'
})
```

```json
this.id = this.$store.state.Info // 获取状态state
this.$store.commit('updateInfo') // 显示提交，触发更改状态的方法
```

当一个组件需要获取多个状态时，有以下更加简洁的写法。

辅助函数：mapState

```json
 <div>
 	{{this.city}}
	{{this.name}}
 </div>
```

```json
import {mapState} from 'vuex'
export default {
  name: 'HomeHeader',
  computed: {
    ...mapState([
      'city'，// 相当于this.city = this.$store.state.city
      'name'
    ])
    // 重命名写法
    // ...mapState({
    //   myCity: 'city', 
	  //	 myName: 'name'
  	// })
  }
}
```

##### 3. getter详解

getter可以理解为store的计算属性，它有两个参数：（state, getters(非必须)）。举个例子：

```json
const store = new Vuex.store({
  state: {
    name: 'john',
    age: 18
  },
  getters: {
    isAdult: (state) => {
    	if (state.age >= 18) {
    		return true
  		} else {
  			return false
			}
  	}
  }
})
```

那么怎么访问getter对象呢？有两种方式：

① 通过属性访问（此时的getter是缓存的）

getter暴露成了***store.getters***对象：

```json
this.$store.getters.isAdult // true
```

当getter把其它getter作为第二个参数时，就是我计算我自己 。囧，例如： 

```json
getters: {
  adultNum: (state, getters) {
    return getters.isAdult.length
  }
}
```



② 通过方法访问(没有缓存，每次访问都会调用方法)

这种方法主要是为了给getter传参，一般用于操作store中的数组。再举个栗子：

```json
state: {
  list: [
    {name: 'a', age: 10}，
    {name: 'b', age: 14}，
    {name: 'c', age: 18}，
  ]
}
getters: {
  getNum (state) => (than) => {
  	return state.list.find(value => age === than)
	}
}
```

```json
this.$store.getters.getNum(18)
```

类似于mapState，getter也有自己的辅助函数：mapGetters，同样是将getters映射到组件。例子：

```json
import {mapGetters} from 'vuex'

export defalut {
  computed: {
    ...mapGetters([
      'getNum'
    ])
    // 重命名写法
    // ...mapGetters({
    //   getSomething: 'getNum'
  	// })
  }
}
```

```json
<p>通过属性访问getter：<br/>{{this.byProperty}}</p>
<p>通过方法访问getter：<br/>{{this.getNum(14)}}</p>
```

##### 4. mutation详解

mutation相当于vuex中的事件methods，vuex中的mutation都是同步事务。

每个mutation都有一个字符串的事件类型（type，即函数名）和一个回调函数（handler），这个回调函数是我们实际改变状态的地方。

我们已经知道显式提交是这样的：

```json
this.$store.commit('add')
```

**提交载荷**（payload）：

指显示提交的时候传入额外的参数，载荷形式栗子：

```json
mutations: {
  add (state, n) {
    state.count += n
  }
}
```

```json
this.$store.commit('add', 2)
```

其实载荷应该尽量写成一个对象来使用，这样可以包含多个字段，也更加规范。例如：

```json
mutations: {
  add (state, payload) {
    state.count += payload.amount
  }
}
```

```json
this.$store.commit('add', {
  amount: 2
})
```

另一种提交风格，对象形式(效果同上)：

```json
mutation: {
  add (state, payload) {
    state.count += payload.amount
  }
}
```

```json
this.$store.commit({
  type: 'add',
  amount: 2
})
//整个对象作为荷载传递给mutation函数
```

提交荷载大概如此。

接下来，当我们想要使用store中的方法（mutation）去改变store中的数据（state）中的**对象和数组**属性时，应该怎样操作呢？

有3种方法：

① set方法

```json
Vue.set(obj, 'newProp', newValue) // 对象
Vue.set(arr, index, newValue) // 数组
```

② 以新换旧（其本质是改变对象或数组的引用地址）

```json
state.obj = {...state.obj, newProp: 123}
```

③ 变异方法，即数组js操作方法

```json
state.list.push(newProp)
```

ok,完美。

**在组件中提交mutation**

我们之前说过显式提交

```json
this.$store.commit('xxx')
```

另一种方法是在组件中借助辅助函数mapMutations：

```json
methods: {
  ...mapMutations([
    'add',
    'reduce'
  ])
  // 重命名写法
  // ...mapMutations({
	//	 myAdd: 'add',
  //	 myReduce: 'reduce'
	// })
}
```

##### 5. action详解

我们说过，在vuex中，mutation都是同步事务。栗子：

```json
this.$store.commit('add')
// 显示提交后触发add 这个mutation，导致的状态（state）变更都应该在此时完成
```

那么怎么处理异步操作呢？action闪亮登场。

action与mutation类似，区别在于：

① action提交的是mutation而非直接改变状态。

② action可以包含任何异步操作。（<~~这就是使用action的原因）

先举个例子：

```json
const store = new Vuex.store({
  state: {
    count: 1
  },
  mutations: {
    secondAdd (state) {
      state.count ++
    }
  },
  actions: {
    firstAdd (context) {
      context.commit('secondAdd')
    }
  }
})
```

重点1：context特喵的是啥？

action接受两个参数，第一个是context，第二个和mutation类似，是提交载荷携带的额外的参数（非必须）。

其中，context与store实例具有相同的方法和属性。那么为什么用context而不直接用store呢？

重点2：action参数的解构赋值。

例子：

```json
actions: {
  firstAdd ({commit}) {
    commit('secondAdd')
  }
}
```

其实以上写法是以下写法的简写形式：

```json
actions: {
  firstAdd (context) {
    context.commit('secondAdd')
  }
}
```

为什么能对context进行解构赋值呢？我们将context打印出来：

```json
context = {
  dispatch: local.dispatch,
　commit: local.commit,
　getters: local.getters,
　state: local.state,
　rootGetters: store.getters,
　rootState: store.state   
}
```

所以，以下这种写法就能理解了吧：

```json
actions: {
  firstAdd ({commit} = context) {
    commit('secondAdd')
  }
}
// 解构赋值后commit = context.commit
```



为什么不直接操作mutation呢？因为action可以异步操作。

```json
actions: {
  firstAdd ({commit}) {
    setTimeout(() => {
      commit(secondAdd)
    }, 1000)
  })
}
```

action使用场景：涉及到调用异步api和分发多重mutation。

和mutation类似，action的分发同样支持载荷方式和对象方式：

```json
// 载荷方式
stote.dispatch('firstAdd', {
  amount: 10
})

// 对象方式
store.dispatch({
  type: 'firstAdd',
  amount: 10
})
```

当然，action也有辅助函数mapActions：

```json
methods: {
  ...mapActions(['firstAdd'])
  // 重命名写法
  // ...mapActions({
  // 	myAdd: 'firstAdd'
	// })
}
```



最后是action的异步操作时的同步写法，借助promise或者async/await：

```json
actions: {
  actionA ({commit}) {
    return new Promise((resolve, reject) => {
      setTimeout(() => {
        commit('add')
        resolve()
      }, 1000)
    })
  },
  // 运行actionB，会先执行actionA，然后执行actionB
  actionB ({dispatch, commit}) {
    return dispatch('actionA').then(() => {
      commit('reduce')
    })
  },
  actionC ({commit}) {
    
  }
}


// 组件中
this.$store.dispatch('actionA').then(() => {
  console.log('我是最后执行的')
})

// async/await写法
```

##### 5. module详解

当项目足够大时，store对象会变得十分臃肿。因此我们需要将store分割为各个模块（module），每个模块有自己的state，mutation，action，getter甚至嵌套模块。例子：

```json
const moduleA = {
  state: {...},
  mutations: {...},
  actions: {...},
  getters: {...}          
}

const moduleB = {
  state: {...},
  mutations: {...},
  actions: {...},
  getters: {...}          
}
            
const store = new Vuex.store({
  modules: {
    a: moduleA,
    b: modeleB
  }
})
              
this.$store.state.a // moduleA的状态
this.$store.state.b // moduleB的状态
```



模块的局部状态

在模块内部，

getter和mutation参数里的state指向当前模块内部的状态，即局部状态。模块外部的状态指rootState

```json
getter: ({state, getters, rootState}) {}
```

对于action，局部状态为context.state,根节点状态为context.rootState

```json
action: ({state, commit, rootState}) {}
```

好吧，我自己都不知道自己在讲什么了。。。先放这



命名空间namespaced

模块化后，在组件中调用store的时候我怎么知道调用的哪个呢？因此要使用namespaced：true

经过多次试验后，举出以下例子，应该是最简写法了：

```json
const state = {
  count: 0,
  gods: [
    {name: 'rulai', level: 100},
    {name: 'wukong', level: 98},
    {name: 'bajie', level: 95}
  ]
}
const mutations = {
  create (state) {
    state.count++
  }
}

export default{
  namespaced: true,
  state,
  mutations
}
```

```json
import { mapState, mapMutations } from 'vuex'
export default {
  name: 'VuexModuleTest',
  computed: {
    ...mapState('create', ['count', 'gods']),
    ...mapState('destroy', ['dead', 'evils'])
  },
  methods: {
    ...mapMutations('create', ['create']),
    ...mapMutations('destroy', ['destroy'])
  }
}
```





