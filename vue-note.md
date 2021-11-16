# Vue组件通信

**[参考](https://segmentfault.com/a/1190000019208626)**

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

## 3.vux



## 总结

当项目比较大时，可以选择vuex

