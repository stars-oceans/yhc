### 一、使用reactive函数声明数组如何正确赋值
[hello world](https://github.com/stars-oceans/yhc/issues/1)

　　需求：将接口请求到的列表数据赋值给响应数据 array


```
const arr = reactive([]);
const load = () => {
  const res = [2, 3, 4, 5]; //假设请求接口返回的数据
  // 方法1 失败，直接赋值丢失了响应性
  // arr = res;
  // 方法2 这样也是失败
  // arr.concat(res);
  // 方法3 可以，但是很麻烦
  res.forEach(e => {
    arr.push(e);
  });
};
```


　　问题原因：这是因为 `arr = newArr`这行代码让arr失去了响应式。**vue3 使用`proxy`，对于对象和数组都不能直接整个赋值**。

> 具体原因：reactive声明的响应式对象被 arr 代理，操作代理对象需要有代理对象的前缀，直接覆盖会丢失响应式。

　　方法2为什么不行？只有push或者根据索引遍历赋值才可以保留reactive数组的响应性？如何方便的将整个数组拼接到响应式数据上？下面我们看下解决方案：



```
// 这几种办法都可以触发响应性，推荐第一种
// 方案1：创建一个响应式对象，对象的属性是数组
const state = reactive({
    arr: []
});
state.arr = [1, 2, 3]

// 方案2: 使用ref函数
const state = ref([])
state.value = [1, 2, 3]

// 方案3: 使用数组的push方法
const arr = reactive([])
arr.push(...[1, 2, 3])
```



### 二、script setup 语法糖中reactive + toRefs+解构如何优雅呈现

　　比如下面这样，我定义了一个 reactive() 声明的对象，想在模板上响应式的使用其值，如果不使用 setup 语法糖，就可以使用 toRefs 然后配合解构 return 出去。使用 setup 语法糖的话，就可以这样

```
let starData = reactive({
  total: 0,
  stars: Array<Star>(),
})
const { total, stars } = toRefs(starData)
```

### 三、Options API 与 Composition API 如何选择及混用是否对性能有影响

1、使用了 Vue3，是否都要遵循用 Composition API 的形式去写页面？

　　答案是否定的。

　　需要注意一点：Vue3 并没有废弃 Options API，甚至还会**全力支持兼容 Vue2 语法的工作**。

　　而 **CompositionAPI 出现的背景主要是为了解决逻辑抽象和和复用的问题，但不意味着它成为了 Vue3 的标准**。

　　因此如何区分场景使用 `Options API` or `Composition API`**主要看业务逻辑的复杂程序**，例如一些简单的 toast/button 等基础组件，用`options API`形式会更加清晰和简洁。而相对复杂的业务逻辑，可以用 `Composition API`，可以把单独一块逻辑抽离到一个模块，通过 **hook 函数**的方式去解决。

2、Vue3 中混用 Options API 和 Composition API 会不会对性能产生影响？

　　答案是不会。其实从问题 1 就可以明显地看出来并不会对性能产生任何影响。不应该被`option api`限制思维，而更多关注逻辑内聚问题。

### 四、关于 setup 中没有 this 的问题及 setup 的执行时机

　　vue 官方文档是这么解释的：在 setup() 内部，this 不会是该活跃实例的引用，因为 `setup()` 是在解析其它组件选项之前被调用的，所以 `setup()` 内部的 this 的行为与其它选项中的 this 完全不同。这在和其它选项式 API 一起使用 setup() 时可能会导致混淆。这意味着，**除了 props 之外，你将无法访问组件中声明的任何属性** —— 本地状态，计算属性/方法。

　　但是**从源码实现你会发现其实组件实例创建在前，函数之所以访问不到 this，是因为它在执行 setup 函数的时候，就没有把组件实例 instance 传给 setup。也没有把 this 指向实例 instance**。

　　因此执行顺序其实是：**组件实例创建在 setup 函数执行之前，但是 setup 执行的时候，组件还没有 mounted，而晚于 beforeCreate 钩子，早于 create 钩子**。
