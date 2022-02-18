# Vue版本

Vue2.x

# Provide-Inject

​	vue中的 provide 跟 inject 可以实现组件的跨级通信，举个简单的示例：

**第一种**

```js
// 在父组件中这样定义
provide: {
	test: '123'
},
// 在子组件中就可以通过inject拿到
inject: ['test'],
```

**第二种**

```vue
// 在父组件中这样定义
provide() {
    return {
    	test: '123'
    }
},
// 在子组件中就可以通过inject拿到
inject:{
    test: {
        from: 'test', // 对应的provide中的参数
        default: '321' // 默认值
    }
},
```

# 响应式

​	vue2中的`provide` 和 `inject` 绑定并不是可响应的。这是刻意为之的。然而，如果你传入了一个可监听的对象，那么其对象的 property 还是可响应的。

​	比如：

```js
// 在父组件中声明一个对象
data() {
	return {
		test: {
            a: '1'
        }
	}
}
provide() {
    return {
    	test: this.test
    }
},
// 在子组件中通过inject拿到
inject:{
    test: {
        from: 'test', // 对应的provide中的参数
        default: {a: '222'} // 默认值
    }
},
// 当你改变父组件中test.a的值的话，在子组件中是响应式的。
```

# 参考

[官方](https://cn.vuejs.org/v2/api/#provide-inject)