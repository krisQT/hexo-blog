---
title: vue
tags: 
      - vue
      - js
comments: true
categories: web前端
---
###### 1.生命周期钩子函数 created update destroyed ，钩子函数的this指向vue实例， 不能使用箭头函数，因为箭头函数是和父级上下文绑定在一起的，这样this将不会指向vue。
######  2.绑定数据 
```
<span>Message: {{ msg }}</span>
```
      
###### 3.通过使用 v-once 指令，你也能执行一次性地插值，当数据改变时，插值处的内容不会更新。但请留心这会影响到该节点上所有的数据绑定：
```
<span v-once>这个数据将不会改变: {{msg}} </span>
```

     
######      4.表达式（单个表达才能生效。语句和控制流程没用）
 ```
{{ number + 1 }}
{{ ok ? 'YES' : 'NO' }}
{{ message.split('').reverse().join('') }}
<div v-bind:id="'list-' + id"></div>
```
######      5. 指令的职责是，当表达式的值改变时，将其产生的连带影响，响应式地作用于 DOM

```
<a v-bind:href="url"></a>
```

在这里 href 是参数，告知 v-bind 指令将该元素的 href 属性与表达式 url 的值绑定。

######      6. 我们可以将同一函数定义为一个方法而不是一个计算属性。对于最终的结果，两种方式确实是相同的。然而，不同的是计算属性是基于它们的依赖进行缓存的。计算属性只有在它的相关依赖发生改变时才会重新求值。这就意味着只要 message 还没有发生改变，多次访问 reversedMessage 计算属性会立即返回之前的计算结果，而不必再次执行函数。
######      7.v-if 是“真正的”条件渲染，因为它会确保在切换过程中条件块内的事件监听器和子组件适当地被销毁和重建。v-if也是惰性的：如果在初始渲染时条件为假，则什么也不做——直到条件第一次变为真时，才会开始渲染条件块。相比之下，v-show就简单得多——不管初始条件是什么，元素总是会被渲染，并且只是简单地基于 CSS 进行切换。一般来说，v-if有更高的切换开销，而v-show有更高的初始渲染开销。因此，如果需要非常频繁地切换，则使用v-show较好；如果在运行时条件不太可能改变，则使用 v-if 较好。

```   
<!-- 阻止单击事件冒泡 -->
<a v-on:click.stop="doThis"></a>
<!-- 提交事件不再重载页面 -->
<form v-on:submit.prevent="onSubmit"></form>
<!-- 修饰符可以串联 -->
<a v-on:click.stop.prevent="doThat"></a>
<!-- 只有修饰符 -->
<form v-on:submit.prevent></form>
<!-- 添加事件侦听器时使用事件捕获模式 -->
<div v-on:click.capture="doThis">...</div>
<!-- 只当事件在该元素本身 (比如不是子元素) 触发时触发回调 -->
<div v-on:click.self="doThat">...</div>
```

###### 8. HTML 特性是不区分大小写的。所以，当使用的不是字符串模板，camelCased (驼峰式) 命名的 prop 需要转换为相对应的 kebab-case (短横线隔开式) 命名：

 ```
Vue.component('child', {
  // camelCase in JavaScript
  props: ['myMessage'],
  template: '<span>{{ myMessage }}</span>'
})


<!-- kebab-case in HTML -->
<child my-message="hello!"></child>
动态绑定
 <child v-bind:my-message="parentMsg"></child>
 <child :my-message="parentMsg"></child>

传递对象可省略参数
 
todo: {
  text: 'Learn Vue',
  isComplete: false
}
<todo-item v-bind="todo"></todo-item>
等价于
 
<todo-item  v-bind:text="todo.text"  v-bind:is-complete="todo.isComplete"></todo-item>
```
###### 9单向数据流
1. prop 是单向绑定的：当父组件的属性变化时，将传导给子组件，但是不会反过来。这是为了防止子组件无意修改了父组件的状态——这会让应用的数据流难以理解。
另外，每次父组件更新时，子组件的所有 prop 都会更新为最新值。这意味着你不应该在子组件内部改变 prop。如果你这么做了，Vue 会在控制台给出警告。

1. 为什么我们会有修改 prop 中数据的冲动呢？通常是这两种原因：
-    prop 作为初始值传入后，子组件想把它当作局部数据来用；
-    prop 作为初始值传入，由子组件处理成其它数据输出。

######   解决办法1:定义一个局部变量，并用props初始化它

 ```
props: ['initialCounter'],
data: function () {
  return { counter: this.initialCounter }
}
```
######  解决办法2:定义一个计算属性，处理 prop 的值并返回。
```
  props: ['size'],
 computed: {
  normalizedSize: function () {
    return this.size.trim().toLowerCase()
  }
}
```

###### 10 props验证

```  
Vue.component('example', {
  props: {
    // 基础类型检测 (`null` 意思是任何类型都可以)
    propA: Number,
    // 多种类型
    propB: [String, Number],
    // 必传且是字符串
    propC: {
      type: String,
      required: true
    },
    // 数字，有默认值
    propD: {
      type: Number,
      default: 100
    },
    // 数组/对象的默认值应当由一个工厂函数返回
    propE: {
      type: Object,
      default: function () {
        return { message: 'hello' }
      }
    },
    // 自定义验证函数
    propF: {
      validator: function (value) {
        return value > 10
      }
    }
  }
})
```


###### 11子组件与父组件通信采用自定义事件

 ***html***
 ```
<div id="counter-event-example">
  <p>{{ total }}</p>
  <button-counter v-on:increment="incrementTotal"></button-counter>
  <button-counter v-on:increment="incrementTotal"></button-counter>
</div>
```
1. v-on：监听子组件事件
1. $emit：触发事件

***js***
 ```
Vue.component('button-counter', {

  template: '<button v-on:click="incrementCounter">{{ counter }}</button>',
  data: function () {
    return {
      counter: 0
    }
  },
  methods: {
    incrementCounter: function () {
      this.counter += 1
      this.$emit('increment')
    }
  },
})

new Vue({
  el: '#counter-event-example',
  data: {
    total: 0
  },
  methods: {
    incrementTotal: function () {
      this.total += 1
    }
  }
})
 ```
此时子组件已经完全与外界解耦，唯一要做的就是报告自己的内部事件，至于父组件是否关心与他无关

###### 12路由

***html***
 ```
<script src="https://unpkg.com/vue/dist/vue.js"></script>
<script src="https://unpkg.com/vue-router/dist/vue-router.js"></script>

<div id="app">
  <h1>Hello App!</h1>
  <p>
    <!-- 使用 router-link 组件来导航. -->
    <!-- 通过传入 `to` 属性指定链接. -->
    <!-- <router-link> 默认会被渲染成一个 `<a>` 标签 -->
    <router-link to="/foo">Go to Foo</router-link>
    <router-link to="/bar">Go to Bar</router-link>
  </p>
  <!-- 路由出口 -->
  <!-- 路由匹配到的组件将渲染在这里 -->
  <router-view></router-view>
</div>
 ```
***js***

 <!--如果使用模块化机制编程，導入Vue和VueRouter，要调用 Vue.use(VueRouter)-->

 1. 定义（路由）组件。
```
// 可以从其他文件 import 进来
const Foo = { template: '<div>foo</div>' }
const Bar = { template: '<div>bar</div>' }
 ```
 2. 定义路由:
 每个路由应该映射一个组件。 其中"component" 可以是
 通过 Vue.extend() 创建的组件构造器，
 或者，只是一个组件配置对象。
 我们晚点再讨论嵌套路由。
 ```
const routes = [
  { path: '/foo', component: Foo },
  { path: '/bar', component: Bar }
]
 ```
 3. 创建 router 实例，然后传 `routes` 配置你还可以传别的配置参数, 不过先这么简单着吧。

 ```
const router = new VueRouter({
  routes // （缩写）相当于 routes: routes
})
 ```
 4. 创建和挂载根实例。
记得要通过 router 配置参数注入路由， 从而让整个应用都有路由功能
 ```
const app = new Vue({
  router
}).$mount('#app')
 ```
现在，应用已经启动了！
要注意，当 <router-link> 对应的路由匹配成功，将自动设置 class 属性值 .router-link-active。

###### 13动态路由匹配
***html***
 ```
<div id="app">
  <p>
  <router-link to="/user/fo">/user/fo</router-link>
  <router-link to="/user/bar">/user/bar</router-link>
  </p>
  <router-view></router-view>
</div>
 ```
***js***
 ```
const User = {
  template: '<div>User</div>'
}

const router = new VueRouter({
  routes: [
    // 动态路径参数 以冒号开头
    { path: '/user/:id', component: User }
  ]
})
 ```
访问的时候直接使用 /user/id ，可以跳转到user/id ，通过  $route.params.id 可以获取id,通过 $route.query 可以获取URl，还有 $route.hash等 ，[$route参数文档](https://router.vuejs.org/zh-cn/api/route-object.html)


###### 14嵌套路由
***html***
 ```
const User = {
  template: `
    <div class="user">
      <h2>User {{ $route.params.id }}</h2>
      <router-view></router-view>
    </div>
  `
}
 ```

***js***
 ```
const router = new VueRouter({
  routes: [
    { path: '/user/:id', component: User,
      children: [
        // 当 /user/:id 匹配成功，
        // UserHome 会被渲染在 User 的 <router-view> 中
        { path: '', component: UserHome },
       
          {
          // 当 /user/:id/profile 匹配成功，
          // UserProfile 会被渲染在 User 的 <router-view> 中
          path: 'profile',
          component: UserProfile
        },
        {
          // 当 /user/:id/posts 匹配成功
          // UserPosts 会被渲染在 User 的 <router-view> 中
          path: 'posts',
          component: UserPosts
        }
      ]
    }
  ]
})
 ```


###### 15.vuex

- 1.store
通过在根实例中注册 store 选项，该 store 实例会注入到根组件下的所有子组件中，且子组件能通过 this.$store访问到
 ```
const Counter = {
  template: `<div>{{ count }}</div>`,
  computed: {
    count () {
      return this.$store.state.count
    }
  }
}
 ```
使用 Vuex 并不意味着你需要将所有的状态放入 Vuex。虽然将所有的状态放到 Vuex 会使状态变化更显式和易调试，但也会使代码变得冗长和不直观。如果有些状态严格属于单个组件，最好还是作为组件的局部状态。你应该根据你的应用开发需要进行权衡和确定。

- 2.getters
当我们在子组件定义对store计算属性时，如果某些计算属性函数是多个子组件都需要，此时需要我们在多个子组件中定义，那么会非常麻烦，就可以选择使用 getters 来解决这个问题
Vuex 允许我们在 store 中定义『getters』（可以认为是 store 的计算属性）。就像计算属性一样，getters的返回值会根据它的依赖被缓存起来，且只有当它的依赖值发生了改变才会被重新计算。
 ```
const store = new Vuex.Store({
  state: {
    todos: [
      { id: 1, text: '...', done: true },
      { id: 2, text: '...', done: false }
    ]
  },
  getters: {
    doneTodos: state => {                    //第一个参数是state
      return state.todos.filter(todo => todo.done)
    }
  }
})

//Getters 会暴露为 store.getters 对象：

store.getters.doneTodos // -> [{ id: 1, text: '...', done: true }]

//Getters接受其他getters作为第二个参数

getters: {
  // ...
  doneTodosCount: (state, getters) => {
    return getters.doneTodos.length
  }
}

store.getters.doneTodosCount // -> 1

mapGetters 辅助函数仅仅是将 store 中的 getters 映射到局部计算属性：

import { mapGetters } from 'vuex'

export default {
  // ...
  computed: {
  // 使用对象展开运算符将 getters 混入 computed 对象中
    ...mapGetters([
      'doneTodosCount',
      'anotherGetter',
      // ...
    ])
  }
}
 ```
###### 3.更改store中的状态唯一的办法是提交mutations.
 ```
const store = new Vuex.Store({
  state: {
    count: 1
  },
  mutations: {
    increment (state) {
      // 变更状态
      state.count++
    }
  }
})
 ```
- mutations像是注册了一个事件，但不能直接使用，需要通过调用  store.commit('increment')
 ```
// ...
mutations: {
  increment (state, payload) {
    state.count += payload.amount
  }
}

store.commit('increment', {
  amount: 10
})

store.commit({ //另一种方式提交 type作为mutations 函数名传入
  type: 'increment',
  amount: 10
})
 ```
- 你可以在组件中使用 this.$store.commit('xxx') 提交 mutation
一条重要的原则就是要记住 mutation 必须是同步函数

- 你可以在组件中使用 this.$store.commit('xxx') 提交 mutation，或者使用 mapMutations 辅助函数将组件中的 methods 映射为 store.commit 调用（需要在根节点注入 store）。
 ```
import { mapMutations } from 'vuex'

export default {
  // ...
  methods: {
    ...mapMutations([
      'increment' // 映射 this.increment() 为 this.$store.commit('increment')
    ]),
    ...mapMutations({
      add: 'increment' // 映射 this.add() 为 this.$store.commit('increment')
    })
  }
}
 ```

###### 4.Actions
Action 类似于 mutation，不同在于：
  - Action 提交的是 mutation，而不是直接变更状态。
  - Action 可以包含任意异步操作。
 ```
const store = new Vuex.Store({
  state: {
    count: 0
  },
  mutations: {
    increment (state) {
      state.count++
    }
  },
  actions: {
    increment (context) {
      context.commit('increment')
    }
  }
})
 ```
- Action 函数接受一个与 store 实例具有相同方法和属性的 context 对象，因此你可以调用 context.commit 提交一个 mutation，或者通过 context.state 和 context.getters 来获取 state 和 getters。
- 你在组件中使用 this.$store.dispatch('xxx') 分发 action，或者使用 mapActions 辅助函数将组件的 methods 映射为 store.dispatch 调用（需要先在根节点注入 store）：
 ```
import { mapActions } from 'vuex'

export default {
  // ...
  methods: {
    ...mapActions([
      'increment' // 映射 this.increment() 为 this.$store.dispatch('increment')
    ]),
    ...mapActions({
      add: 'increment' // 映射 this.add() 为 this.$store.dispatch('increment')
    })
  }
}

 ```