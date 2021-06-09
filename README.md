### 1. seutup()

​		**setup会在beforeCreate之后created之前执行，所有需要在模板中的数据都要return出去**

```vue
<template>
    <div>

    </div>
</template>

<script>
import { setup, getCurrentInstance } from "vue";
    // let ins = getCurrentInstance() 获取当前组件实例
export default {
    // 声明接收的数据
    props:{
        name: String
    }
    /*
    	props: 父组件传递过来的参数
    	context: 执行上下文(相当于Vue2.x的this)：改函数运行所支持的环境，如一些外部变量等，这些值的集合就是上下文
    */
    setup(props, context) { // setup(props, {attr, emit, slots})
        // ! 注意： props不能解构，解构后不是响应式了，相当于把外层的监听壳拿走了
        console.log(props.name)
        console.log(context)
        console.log(this) // undefined
    }
}
</script>
```



### 2. reactive()  和  ref()

​		**创建响应式数据，相当于2.x的data，只能在setUp中使用**

#### 2.1 reactive

```vue
<template>
    <div>{{state.name}}+{{state.size}}</div>
</template>

<script>
import { reactive, setup, h } from "vue";
export default {
    setup() {
        const state = reactive({
            size: 16,
            name: '张三'
        })
        // 第一种形式: 返回render函数的上下文对象
        return {
            state
        }
        // 第二种形式：返回渲染函数
        return () => h('div', state.size)
    }
}
</script>
```

```vue
<template>
    <div>{{name}}+{{size}}</div>
</template>

<script>
import { reactive, setup } from "vue";
export default {
    setup() {
        const state = reactive({
            size: 16,
            name: '张三'
        })
        return {
            ...state
        }
    }
}
</script>
```





#### 2.2 ref

​		**ref()函数返回一个对象，对象只有一个属性为value，.value只有在setup中使用，模板上并不需要**

```vue
<template>
    <div>{{count}}</div>
</template>

<script>
import { ref } from "vue";
export default {
    setup() {
        const count = ref(0)
    	count.value++
        return {
            count
        }
    }
}
</script>
```



#### 2.3 ref的DOM引用

```jsx
<template>
  <div>
    <!-- 声明一个ref,保证与创建的名称一直 -->
    <h3 ref="h3Ref">TemplateRefOne</h3>
  </div>
</template>

<script>
import { ref, onMounted } from '@vue/composition-api'

export default {
  setup() {
    // 创建一个 DOM 引用
    const h3Ref = ref(null)

    // 在 DOM 首次加载完毕之后，才能获取到元素的引用
    onMounted(() => {
      // 为 dom 元素设置字体颜色
      // h3Ref.value 是原生DOM对象
      h3Ref.value.style.color = 'red'
    })

    // 把创建的引用 return 出去
    return {
      h3Ref
    }
  }
}
</script>
```





### 3. isRef()

​		**判断一个数是否是ref创建的**

```vue
<template>
    <div></div>
</template>

<script>
import { ref, isRef } from "vue";
export default {
    setup() {
        let foo = ref(0)
        let coo = isRef(foo) ? foo.value : foo
            
        return {
            
        }
    }
}
</script>
```



### 4. toRefs()

​		**结合reactive使用，使展开的reactive为响应式：将reactive创建的响应式对象转换成一个普通对象，只不过这个普通对象里面的每一个属性都是 ref() 类型的响应式对象**

```vue
<template>
    <div>{{num1}}+{{num2}}</div>
	<button @click="add1">add1</button>
	<button @click="add2">add2</button>
</template>

<script>
import { reactive, setup } from "vue";
export default {
    setup() {
        const state1 = reactive({
            num1: 16,
        })
        const state2 = reactive({
            num2: 16,
        })
        
        const add1 = ()=>state1.num1++
        const add2 = ()=>state2.num2++
        
        // 此时点击 add1 num1并不会呗自动改变
        return {
            ...state1,
            toRefs(state2)
        }
    }
}
</script>
```



### 5. computed

​		**computed() 用来创建计算属性，返回值是一个 ref 的实例（不可手动修改），传入一个function**

#### 5.1  创建只读属性

```js
const count = ref(1)

// 根据 count 的值，创建一个响应式的计算属性 plusOne
// 它会根据依赖的 ref 自动计算并返回一个新的 ref
const plusOne = computed(() => count.value + 1)

console.log(plusOne.value) // 输出 2
plusOne.value++ // error
```



#### 5.2 创建可读可写属性

​		**利用get  set 创建**

```js
// 创建一个 ref 响应式数据
const count = ref(1)

// 创建一个 computed 计算属性
const plusOne = computed({
  // 取值函数
  get: () => count.value + 1,
  // 赋值函数
  set: val => {
    count.value = val - 1
  }
})

// 为计算属性赋值的操作，会触发 set 函数
plusOne.value = 9
// 触发 set 函数后，count 的值会被更新
console.log(count.value) // 输出 8
```



### 6. watch

#### 6.1 基本用法

```js
const count = ref(0)

// 定义 watch，只要 count 值变化，就会触发 watch 回调
watch(count, () => console.log(count.value))
// 输出 0

setTimeout(() => {
  count.value++
  // 输出 1
}, 1000)
```



#### 6.2 监听ref属性

```js
let a = ref(0)

watch(
    a,
    (val, oldval) => {
        console.log(999)
    },
    { lazy: true } // 可用来取消组件初始化的调用
)
setTimeout(() => {
    a.value++
}, 2000)
```



#### 6.3 监听reactive属性

```js
const state = reactive({ num: 222 })
watch(
    () => state.num,
    (val, oldval) => {
        console.log(999)
    },
    { lazy: true }
)
setTimeout(() => {
    state.num++
}, 2000)
```



#### 6.4  监听多个数据源

**单个参数变成数组，回调接收的也变成数组**

```js
// 监听ref---------------------------------------------------------------
let a = ref(0), b=ref(1)

watch(
    [a, b],
    ([aVal, bVal], [aOldval, bOldval]) => {  // 解构赋值
        console.log(aVal, bVal)
    },
    { lazy: true } // 可用来取消组件初始化的调用
)
setTimeout(() => {
    a.value++
    b.value++
}, 2000)



// 监听reactive----------------------------------------------------------
let state = reactive({
    a: 1,
    b: 2
})

watch(
    [()=>state.a, ()=>state.b],
    (valArr, oldvalArr) => {
        console.log(valArr, oldvalArr)
    },
    { lazy: true } // 可用来取消组件初始化的调用
)
setTimeout(() => {
    state.a++
    state.b++
}, 2000)

```



#### 6.5  清除监视器

```js
let a = ref(0)

let stop = watch(
    a,
    (val, oldval) => {
        console.log(999)
    },
    { lazy: true } // 可用来取消组件初始化的调用
)
setInterval(() => {
    a.value++
}, 2000)
setTimeout(()=>{
    stop()
},5000)
```



#### 6.6  watch中的监听时间回调

**如果watch在1S内重复被触发，怎会触发该回调函数 ? 未确定** 

```vue
<template>
  <div>
    <input type="text" v-model="val" />
  </div>
</template>

<script>
import { setup, ref, watch } from "@vue/composition-api"

export default {
  name: "Home",
  setup(props, context) {
    let val = ref("")
    const asyncPrint = ()=>{
      return setTimeout(()=>{
        console.log(val.value)
      },500)
    }
    watch(val, (val, oldval, clear) => {
      let timeId = asyncPrint()
      // 如果1S内被重复触发，则会调用该回调函数
      clear(()=>clearTimeout(timeId))
    }, { lazy: true })

    return {
      val,
    }
  },
}
</script>

```



### 7. watchEffect

**副作用侦听器：立即执行传入的函数，并执行响应式依赖，当依赖更新时重新运行此函数**、

```js
const count = ref(0)
watchEffect(() => console.log(count.value))
setTimeout(() => {
    count.value++
}, 1000)
```



### 8.  生命周期

**所有的声明周期函数全部放在setup里面，不在放到外面**

**可多次注册，按顺序执行**

- ~~`beforeCreate`~~ -> use `setup()`
- ~~`created`~~ -> use `setup()`
- `beforeMount` -> `onBeforeMount`
- `mounted` -> `onMounted`
- `beforeUpdate` -> `onBeforeUpdate`
- `updated` -> `onUpdated`
- `beforeDestroy` -> `onBeforeUnmount`
- `destroyed` -> `onUnmounted`
- `errorCaptured` -> `onErrorCaptured`
- `onRenderTracked` -> `新增`
- `onRenderTriggered` -> `新增`



### 9.  provide&inject

​	**1. `provide()` 和  `inject()` 可以实现嵌套组件之间的数据传递。这两个函数只能在 `setup()` 函数中使用。父级组件中使用 `provide()` 函数向下传递数据；子级组件中使用 `inject()` 获取上层传递过来的数据**
​	**2. 可共享ref响应式数据**

```js
// 父组件
import { provide } from '@vue/composition-api'
export default {
  name: 'app',
  setup() {
    // 1. App 根组件作为父级组件，通过 provide 函数向子级组件共享数据（不限层级）
    //    provide('要共享的数据名称', 被共享的数据)
    provide('globalColor', 'red')
  },
  components: {
    LevelOne
  }
}

// 子组件
import { inject } from '@vue/composition-api'
export default {
  setup() {
    // 2. 调用 inject 函数时，通过指定的数据名称，获取到父级共享的数据
    const themeColor = inject('globalColor')

    // 3. 把接收到的共享数据 return 给 Template 使用
    return {
      themeColor
    }
  },
  components: {
    LevelTwo
  }
}
```



### 10. 自定义指令

```js
export default {
  directives: {
    focus: {
        beforeMounted(el){},
        mounted(el){},
        beforeUpdate(el){},
        updated(el){},
        beforeUnMounted(el){},
        unMounted(el){}
    }
  }
}
```



### 11. Teleport 传送

可以将当前模板的一部分内容移到DOM的其他节点下

```html
<teleport to="body">
    ...
</teleport>
```

