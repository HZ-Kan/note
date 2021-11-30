# Vue组件通信

**[参考](https://segmentfault.com/a/1190000019208626)**   **[参考](https://www.cnblogs.com/barryzhang/p/10566515.html)**

## 1.props/$emit

**简介：**

这种方式是父组件通过props的方式向子组件传递参数，子组件通过事件触发$emit向父组件传递参数。**一般只适用于父子组件的传值。**

当然这里只是简单的介绍他们的常见的用法，更多用法请移步[**官方文档**](https://cn.vuejs.org/v2/guide/components.html#%E7%9B%91%E5%90%AC%E5%AD%90%E7%BB%84%E4%BB%B6%E4%BA%8B%E4%BB%B6)。

**示例：**

**父组件=>子组件**

我们会将AtransB传递到子组件中，并在子组件中利用props接收并展示。

- 先创建两个vue文件分别是componentA.vue与component.vue分别作为父组件与子组件

- 编辑父组件（componentA.vue）如下

  ```vue
  <template>
      <div class="comA">
          我是父组件
          <componentB :AtransB='text'/>
      </div>
  </template>
  
  <script>
  import componentB from './componentB';
  export default {
      name: 'comA',
      components:{
          componentB
      },
      data(){
          return{
              text: '父组件向子组件传值'
          }
      }
  }
  </script>
  
  <style scoped>
  .comA{
      border: 1px solid black;
  }
  </style>
  ```

- 编辑子组件（componentB.vue）如下

  ```vue
  <template>
    <div class="comB">
      我是子组件
      {{ AtransB }}
    </div>
  </template>
  
  <script>
  export default {
    name: 'comB',
    props:{  // 在子组件中利用props来接收父组件传过来的参数
      AtransB:{  // 定义参数
        type: String,  
        required: true
      }
    }
  }
  </script>
  
  <style scoped>
  .comB{
    border: 1px solid pink;
  }
  </style>
  ```

- 运行截图

   ![image-20211111223707525](D:/Typora/img/image-20211111223707525.png)

**子组件=>父组件**

利用子组件的点击事件来触发[$emit](https://cn.vuejs.org/v2/api/#vm-emit)然后在父组件中利用事件接收并赋值给对应的属性。

- 编辑父组件内容

  ```vue
  <template>
      <div class="comA">
          我是父组件{{ text }}
          <componentB @BtransA='changeText'/>
      </div>
  </template>
  
  <script>
  import componentB from './componentB';
  export default {
      name: 'comA',
      components:{
          componentB
      },
      data(){
          return{
              text: ''
          }
      },
      methods:{
          changeText(data){
              this.text = data;
          }
      }
  }
  </script>
  
  <style scoped>
  .comA{
      border: 1px solid black;
  }
  </style>
  ```

- 编辑子组件中的内容

  ```vue
  <template>
    <div class="comB" @click="changeText">
      我是子组件
    </div>
  </template>
  
  <script>
  export default {
    name: 'comB',
    methods:{
      changeText(){
        this.$emit('BtransA','子组件向父组件传值');
      }
    }
  }
  </script>
  
  <style scoped>
  .comB{
    border: 1px solid pink;
  }
  </style>
  ```

- 点击子组件运行截图

  ![kfm7n-l92cd](D:/Typora/img/kfm7n-l92cd.gif)

## 2.$emit/$on

这种方法通过一个空的Vue实例作为中央事件总线（事件中心），用它来触发事件和监听事件，巧妙而轻量的实现了任何组件间的通信（包括父子，兄弟，跨级）。但一个弊端就是，这种方式并不会自动销毁，所以为了避免回调函数重复执行，还要在 `onUnmounted` 中去移除回调函数。这就要求我们在创建回调函数的时候不能使用匿名函数。（这里并不做回调函数的销毁）

**！！！Vue3中弃用了$on[具体见此](https://v3.vuejs.org/guide/migration/events-api.html#_3-x-update)**。因此这里我们使用的是vue2版本。

**[$on用法](https://cn.vuejs.org/v2/api/?#vm-on)**

这里只提供父子组件之间的传值示例。

- 创建config.js文件，添加代码

  ```js
  import Vue from 'vue';
  
  const bus = new Vue(); //创建Vue空实例
  
  export default bus; //对外暴露
  ```

- 创建父组件，添加代码

  ```vue
  <template>
      <div class="comA">
          我是父组件{{ text }}
          <componentB />
      </div>
  </template>
  
  <script>
  import componentB from './componentB';
  import bus from './config' //引入bus
  export default {
      name: 'comA',
      components:{
          componentB
      },
      data(){
          return{
              text: ''
          }
      },
      mounted(){
          bus.$on('BtransA',text =>{
              this.text = text;
          })
      }
  }
  </script>
  
  <style scoped>
  .comA{
      border: 1px solid black;
  }
  </style>
  ```
  
- 创建子组件，添加代码

  ```vue
  <template>
    <div class="comB" @click="changeText">
      我是子组件
    </div>
  </template>
  
  <script>
  import bus from './config';
  export default {
    name: 'comB',
    methods:{
      changeText(){
        bus.$emit('BtransA','子组件向父组件传值');
      }
    }
  }
  </script>
  
  <style scoped>
  .comB{
    border: 1px solid pink;
  }
  </style>
  ```

- 效果截图

  ![kfm7n-l92cd](D:/Typora/img/kfm7n-l92cd.gif)

## 3.$attrs/$listeners

这里介绍另外一种跨级通信的方式。

- `$attrs`：包含了父作用域中不被 prop 所识别 (且获取) 的特性绑定 (class 和 style 除外)。当一个组件没有声明任何 prop 时，这里会包含所有父作用域的绑定 (class 和 style 除外)，并且可以通过 v-bind="$attrs" 传入内部组件。通常配合[ interitAttrs](https://blog.csdn.net/qq_38211541/article/details/105824684) 选项一起使用。
- `$listeners`：包含了父作用域中的 (不含 .native 修饰器的) v-on 事件监听器。它可以通过 v-on="$listeners" 传入内部组件

简单来说：`$attrs`与`$listeners` 是两个对象，`$attrs` 里存放的是父组件中绑定的非 Props 属性，`$listeners`里存放的是父组件中绑定的非原生事件。

**Vue3已废弃[$listeners](https://v3.vuejs.org/guide/migration/listeners-removed.html#overview)**。下面使用$attrs进行演示。

**示例：**

- 创建祖先组件A，添加代码

  ```vue
  <template>
      <div class="comA">
          我是父组件{{ text }}
          <componentB :='text' :age='age' :gender='gender' />
      </div>
  </template>
  
  <script>
  import componentB from './componentB';
  export default {
      name: 'comA',
      components:{
          componentB
      },
      data(){
          return{
              text: 'A组件中的数据',
              age: 18,
              gender: 'man'
          }
      }
  }
  </script>
  
  <style scoped>
  .comA{
      border: 1px solid black;
  }
  </style>
  ```
  
- 创建子组件B，添加代码

  ```vue
  <template>
    <div class="comB">
      {{text}} 我是子组件B {{ $attrs }}
      <componentC v-bind="$attrs" /> 
    </div>
  </template>
  
  <script>
  import componentC from './componentC.vue'
  export default {
    name: 'comB',
    inheritAttrs: false, // 可以关闭自动挂载到组件根元素上的没有在props声明的属性
    components: {
      componentC
    },
    props: {
      text: String
    }
  }
  </script>
  ```

- 创建子组件C，添加代码

  ```vue
  <template>
      <div class="comC">
          我是子组件C {{ $attrs }}
      </div>
  </template>
  
  <script>
  export default{
      name: 'comC',
  
  }
  </script>
  ```

**运行截图：**

![image-20211121191113419](D:/Typora/img/image-20211121191113419.png)

## 4.Provide/Inject

​		以允许一个祖先组件向其所有子孙后代注入一个依赖，不论组件层次有多深，并在起上下游关系成立的时间里始终生效。一言而蔽之：祖先组件中通过provider来提供变量，然后在子孙组件中通过inject来注入变量。			provide / inject API 主要解决了跨级组件间的通信问题，不过它的使用场景，主要是子组件获取上级组件的状态，跨级组件间建立了一种主动提供与依赖注入的关系。

**示例：**

- 创建父组件A，添加代码

  ```vue
  <template>
      <div class="comA">
          我是父组件A
          <componentB />
      </div>
  </template>
  
  <script>
  import componentB from './componentB';
  export default {
      name: 'comA',
      components:{
          componentB
      },
      provide: {
          text: 'A组件中的数据'
      },
  }
  </script>
  
  <style scoped>
  .comA{
      border: 1px solid black;
  }
  </style>
  ```

- 创建子组件B，添加代码

  ```vue
  <template>
    <div class="comB">
      我是子组件B 
      <componentC /> 
    </div>
  </template>
  
  <script>
  import componentC from './componentC.vue'
  export default {
    name: 'comB',
    components: {
      componentC
    }
  }
  </script>
  ```

- 创建子组件C，添加代码

  ```vue
  <template>
      <div class="comC">
          我是子组件C {{ text }}
      </div>
  </template>
  
  <script>
  export default{
      name: 'comC',
      inject: ['text'],
  }
  </script>
  ```

**运行截图：**

![image-20211121192410395](D:/Typora/img/image-20211121192410395.png)

**需要注意的是**provide 和 inject 绑定并不是可响应的。这是刻意为之的。然而，如果你传入了一个可监听的对象，那么其对象的属性还是可响应的。简而言之当组件A中的text发生变化时，组件C中的text不变。想要实现响应式，可参考[官方文档](https://v3.cn.vuejs.org/guide/component-provide-inject.html#%E5%A4%84%E7%90%86%E5%93%8D%E5%BA%94%E6%80%A7)。

## 5.$parent / $children与 ref

**[$children](https://v3.cn.vuejs.org/guide/migration/children.html#children)在Vue3中已经被移除**（官网这样写到：在 3.x 中，`$children` property 已被移除，且不再支持。如果你需要访问子组件实例，我们建议使用 [$refs](https://v3.cn.vuejs.org/guide/component-template-refs.html#模板引用)。）

- `ref`：如果在普通的 DOM 元素上使用，引用指向的就是 DOM 元素；如果用在子组件上，引用就指向组件实例
- `$parent` / `$children`：访问父 / 子实例

需要注意的是：这两种都是直接得到组件实例，使用后可以直接调用组件的方法或访问数据。我们来看个用 `ref`来访问组件的例子：

- 创建父组件A，添加代码

  ```vue
  <template>
      <div class="comA">
          我是父组件A {{text}}
          <componentB ref="comB" />
      </div>
  </template>
  
  <script>
  import componentB from './componentB';
  export default {
      name: 'comA',
      components:{
          componentB
      },
      data(){
          return{
              text: '',
              aaa: '组件A中的数据'
          }
      },
      mounted(){
          const comB = this.$refs.comB;
          this.text = comB.text;
          comB.aaa = this.aaa;
          comB.sayHello(); 
      }
  }
  </script>
  
  <style scoped>
  .comA{
      border: 1px solid black;
  }
  </style>
  ```

- 创建子组件B，添加代码

  ```vue
  <template>
    <div class="comB">
      我是子组件B {{aaa}}
    </div>
  </template>
  
  <script>
  export default {
    name: 'comB',
    data(){
      return{
        text: '组件B中的数据',
        aaa: ''
      }
    },
    methods:{
      sayHello(){
        window.alert('Hello');
      }
    }
  }
  </script>
  ```

**运行截图：**

![image-20211121195237065](D:/Typora/img/image-20211121195237065.png)

不过，**这两种方法的弊端是，无法在跨级或兄弟间通信**。

## 6.vuex

![image](https://segmentfault.com/img/remote/1460000019208632)

**1.简要介绍Vuex原理**

Vuex实现了一个单向数据流，在全局拥有一个State存放数据，当组件要更改State中的数据时，必须通过Mutation进行，Mutation同时提供了订阅者模式供外部插件调用获取State数据的更新。而当所有异步操作(常见于调用后端接口异步获取更新数据)或批量的同步操作需要走Action，但Action也是无法直接修改State的，还是需要通过Mutation来修改State的数据。最后，根据State的变化，渲染到视图上。

**2.简要介绍各模块在流程中的功能：**

- Vue Components：Vue组件。HTML页面上，负责接收用户操作等交互行为，执行dispatch方法触发对应action进行回应。
- dispatch：操作行为触发方法，是唯一能执行action的方法。
- actions：**操作行为处理模块,由组件中的`$store.dispatch('action 名称', data1)`来触发。然后由commit()来触发mutation的调用 , 间接更新 state**。负责处理Vue Components接收到的所有交互行为。包含同步/异步操作，支持多个同名方法，按照注册的顺序依次触发。向后台API请求的操作就在这个模块中进行，包括触发其他action以及提交mutation的操作。该模块提供了Promise的封装，以支持action的链式触发。
- commit：状态改变提交操作方法。对mutation进行提交，是唯一能执行mutation的方法。
- mutations：**状态改变操作方法，由actions中的`commit('mutation 名称')`来触发**。是Vuex修改state的唯一推荐方法。该方法只能进行同步操作，且方法名只能全局唯一。操作之中会有一些hook暴露出来，以进行state的监控等。
- state：页面状态管理容器对象。集中存储Vue components中data对象的零散数据，全局唯一，以进行统一的状态管理。页面显示所需的数据从该对象中进行读取，利用Vue的细粒度数据响应机制来进行高效的状态更新。
- getters：state对象读取方法。图中没有单独列出该模块，应该被包含在了render中，Vue Components通过该方法读取全局state对象。

**3.Vuex与localStorage**

vuex 是 vue 的状态管理器，存储的数据是响应式的。但是并不会保存起来，刷新之后就回到了初始状态，**具体做法应该在vuex里数据改变的时候把数据拷贝一份保存到localStorage里面，刷新之后，如果localStorage里有保存的数据，取出来再替换store里的state。**

## 7.[v-model](https://blog.csdn.net/kzj0916/article/details/120477925)

点击链接查看

## 总结

常见使用场景可以分为三类：

- 父子通信：

  props/$emit;（推荐）

  ref ；

  provide / inject API；

  $attrs;

  $emit/$on；

  Vuex;

  v-model

- 兄弟通信：

  Vuex;

  $emit/$on；

- 跨级通信：

  Vuex；（大型项目推荐）

  provide / inject API;

  $attrs;

  $emit/$on；

# Vue插槽

​		**插槽(slot)**可以让我们在自定义组件的时候把一部分内容预留出来，留给使用该组件的人去自定义，同时我们还可以传递一些数据供其使用。插槽分为三类匿名插槽，具名插槽，作用域插槽。

## 1.匿名插槽（默认插槽）

​	name可以不写，默认值为"default"，这种插槽称为**匿名插槽**或者**默认插槽**。

**示例**

- 创建组件A，添加代码

  ```vue
  <template>
   <div class="comA">
      <comB>
          <p>一段文字</p>
      </comB>
  </div>
  </template>
  
  <script>
  import comB from './componentB.vue'
  export default {
      name: 'comA',
      components: {
          comB,
      }
  }
  </script>
  ```

- 创建组件B，添加代码

  ```vue
  <template>
      <div class="comB">
          <p>123</p>
          <slot></slot>
      </div>
  </template>
  
  <script>
  export default {
      name: 'comB',
  }
  </script>
  ```

- 运行结果

  ![image-20211127164206424](D:/Typora/img/image-20211127164206424.png)

## 2.具名插槽

​		顾名思义就是给这个插槽起个名字,插槽在使用 `name` 属性绑定名字后, 在组件被调用的时候,组件内的标

签通过 `slot` 属性来决定某个标签具体渲染哪一个插槽。

利用[**v-slot**](https://cn.vuejs.org/v2/guide/components-slots.html)实现

**示例**

- 创建组件A，添加代码

  ```vue
  <template>
      <div class="comA">
          <comB>
              <template v-slot:body> 
                  <p>body</p>
              </template> 
              <template v-slot:header> 
                  <p>header</p>
              </template> 
              <template v-slot:footer> 
                  <p>footer</p>
              </template>   
          </comB>
      </div>
  </template>
  
  <script>
  import comB from './componentB.vue'
  export default {
      name: 'comA',
      components: {
          comB,
      }
  }
  </script>
  ```

- 创建组件B，添加代码

  ```vue
  <template>
      <div class="comB">
          <p>123</p>
          <slot name="header">备用内容</slot>
          <slot name="body"></slot>
          <slot name="footer"></slot>
      </div>
  </template>
  
  <script>
  export default {
      name: 'comB',
  }
  </script>
  ```

- 运行结果

  ![image-20211127165415057](D:/Typora/img/image-20211127165415057.png)

## 3.作用域插槽

​		简单来说就是一种可以访问子组件中数据的插槽。

通过v-slot+v-bind实现。**#是v-slot的语法糖。**

**示例**

- 创建组件A，添加代码

  ```vue
  <template>
      <div class="comA">
          <comB>
              <template #body='{text}'> 
                  <p>{{text.a}}</p>
              </template>  
          </comB>
      </div>
  </template>
  
  <script>
  import comB from './componentB.vue'
  export default {
      name: 'comA',
      components: {
          comB,
      }
  }
  </script>
  ```

- 创建组件B，添加代码

  ```vue
  <template>
      <div class="comB">
          <p>123</p>
          <slot name='body' :text='text'>123</slot>
      </div>
  </template>
  
  <script>
  export default {
      name: 'comB',
      data(){
          return{
              text:{
                  a: '组件B中a的值'
              }
          }
      }
  }
  </script>
  ```

- 运行截图

  ![image-20211127173352980](D:/Typora/img/image-20211127173352980.png)

[参考](http://caibaojian.com/vue-slot.html)

# Vue动态组件

​		有的时候，需要在不同组件之间进行动态的切换。比如tab栏的切换，可以利用Vue的**[is](https://cn.vuejs.org/v2/api/#is)**属性来实现。

**示例**

- 效果展示

  ![yaryi-t16ib](D:/Typora/img/yaryi-t16ib.gif)

- 创建组件A，添加代码

  ```vue
  <template>
   <div class="comA">
       <h1>is 动态组件</h1>
       <br />
       <button @click='changeCom'>切换组件</button>
       <div :is='com'></div>
  </div>
  </template>
  
  <script>
  import comB from './comB.vue'
  import comC from './comC.vue'
  import comD from './comD.vue'
  export default {
      name: 'comA',
      data(){
          return{
              index: 0,
              arr:['comB','comC','comD'],
              com: 'comB'
          }
      },
      components: {
          comB,
          comC,
          comD
      },
      methods:{
          changeCom(){
              this.index = this.index === 2 ? 0 : this.index + 1;
              this.com = this.arr[this.index];
          }
      }
  }
  </script>
  ```

- 创建组件B，添加代码

  ```vue
  <template>
      <div class="comB">
          组件B
      </div>
  </template>
  
  <script>
  export default {
      name: 'comB',
  }
  </script>
  ```

- 创建组件C，添加代码

  ```vue
  <template>
    <div class="comC">
      组件C
    </div>
  </template>
  
  <script>
  export default {
    name: 'comC',
  }
  </script>
  ```

- 创建组件D，添加代码

  ```vue
  <template>
    <div class="comD">
      组件D
    </div>
  </template>
  
  <script>
  export default {
    name: 'comD',
  }
  </script>
  ```

# Vuex

# Vue路由

# Vue双向绑定

[参考](https://www.jianshu.com/p/e7ebb1500613)
