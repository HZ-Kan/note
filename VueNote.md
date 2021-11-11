# Vue组件通信

https://segmentfault.com/a/1190000019208626

## 1.利用props/$emit

**简介：**

这种方式是父组件通过props的方式向子组件传递参数，子组件通过事件触发$emit向父组件传递参数。

**示例：**

父组件=>子组件

- 先创建两个vue文件分别是componentA.vue与component.vue分别作为父组件与子组件

- 编辑componentA.vue如下

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
              text: '组件A向组件B传值'
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

- 编辑componentB如下

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

![image-20211111220920170](D:/Typora/img/image-20211111220920170.png)

子组件=>父组件

- 

