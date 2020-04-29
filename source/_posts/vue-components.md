---
title: vue全局组件注册(uni-app)
date: 2020-04-08 16:57:13
tags:
---

##### 在main.js中注册全局组件

```bash
// 引入vuex 状态库
import store from "./store"

// 在main.js中注册全局组件
import toast from './components/toast/toast.vue'

Vue.component('my-toast', toast)

//挂在到Vue原型链上
Vue.prototype.$store = store

//是否显示加载中 的方法 调用store中的mutations方法
function loading(tf) {
    if (tf) {
        store.commit("switch_loading", tf)
    } else {
        store.commit("switch_loading")
    }
}

//也挂在到原型链上 方便在每个页面中  使用 this.$loading()  去显示加载中
Vue.prototype.$loading = loading

```

<!-- more -->

##### 在store/index.js中加上控制显示隐藏的方法

```bash
import Vue from 'vue'
import Vuex from 'vuex'
Vue.use(Vuex)

const store = new Vuex.Store({
    state: {
        loading: false
    },
    getters: {
        is_loading: state => {
            return state.loading
        }
    },
    mutations: {
        //tf作为主动控制的参数
        switch_loading(state, tf) {
            if (tf) {
                state.loading = tf
            } else {
                state.loading = !state.loading
            }
        }
    }
})

export default store

```

##### 在组件components/toast.vue中 加上控制方法 控制属性

``` bash
<template>
    <view class="loading_box" v-show="is_loading" @click="switch_loading">
        <view class="loading">
            <view class="loader loader-17">
              <view class="css-square square1"></view>
              <view class="css-square square2"></view>
              <view class="css-square square3"></view>
              <view class="css-square square4"></view>
              <view class="css-square square5"></view>
              <view class="css-square square6"></view>
              <view class="css-square square7"></view>
              <view class="css-square square8"></view>
            </view>
        </view>
    </view>
</template>

<script>
// 方法一：
export default {
    data() {
        return {
            
        }
    },
    methods:{
        switch_loading() {
            this.$store.commit("switch_loading")
        }
    },
    //实测直接在标签属性里写  $store.state.XX  拿不到数据  所以这里通过 计算属性去监听一下
    computed:{
        is_loading(){
            return this.$store.state.loading
        }
    }
}
</script>

<style lang="less" scoped>

</style>

```

---

``` bash
<script>
// 方法二：
import {mapGetters, mapMutations} from 'vuex'

export default {
    data() {
        return {
            
        }
    },
    computed: {
        ...mapGetters([
            'is_loading'
        ])
    },
    methods:{
        switch_loading() {
            this.switch_loading()
        },
        ...mapMutations([
            'switch_loading'
        ])
    }
}
</script>
```

#### 在页面中使用

```bash
<template>
    <view>
        <my-toast></my-toast>
    </view>
</template>

<script>
export default {
    data() {
        return {
            
        }
    },
    methods: {
        // 调用
        test() {
            this.loading()
            
            this.loading(false)
        }
    }
}
</script>
```
