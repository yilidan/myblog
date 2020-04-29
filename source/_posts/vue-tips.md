---
title: Vue开发小技巧
date: 2020-03-05 16:02:02
tags:
---

### 深度选择器
##### 1. stylus, css使用方法：
``` bash
.parent >>> .child{}
```
##### 2. less使用方法：
``` bash
.parent {
  /deep/ .child{}
}
```
##### 3. sass使用方法：
``` bash
.parent /deep/ .child{}
```

### watch深度监听用法
##### 监听对象属性的变化（方式一）
``` bash
obj: {
  handler() {
      console.log(obj.a)
  },
  immediate: true,
  deep: true
}
```
#### 监听对象属性的变化（方法二）
``` bash
'obj.a': {
    handler(){
        console.log(obj.a)
    },
    immediate: true
}
1. 'immediate'：默认触发handler方法
2. 'deep': 深度监听对象中的属性
```

<!-- more -->

### keep-alive的使用
> 使用keep-alive缓存之后，页面的路由不走created和mounted，你可以在activted中调用数据；

### ref的使用
``` bash
<div ref="test"></div>

非组件中获取样式：
this.$refs.test.style[transform] = `translate3d(0, 0, 0)`
或
this.$refs.test.style.transform = `translate3d(0, 0, 0)`

组价中获取样式：
多了一个 $el 符号
this.$refs.test.$el.style[transform] = `translate3d(0, 0, 0)`

```

### vue定义全局方法，调用其他组件的方法
``` bash
vue-api: this.$root
# 比如想要在其他页面中调用弹窗
# components/dialog.vue
created () {
  this.$root.$on('showLoginDialog', (flag) => {
    this.openShow = flag
  })
}

# 调用
# pages/index.vue
this.$root.$emit('showLoginDialog', true)
```
### 解决vue-router中，当页面地址栏参数变化时，页面不刷新的问题
``` bash
<template>
  <router-view :key="key"></router-view>
</template>

<script>
  export default {
    computed: {
      key() {
        # 符号'+' 将该元素转换成 Number 类型
        return this.$route.name ? this.$route.name+ +new Date() : this.$route+ +new Date()
      }
    }
  }
</script>
```
### Vue-cli配置 - 开启Gzip
1.在config -> index.js中把 build.productionGzip 设置成 true；

2.安装插件 npm i compression-webpack-plugin@1.1.12 -D；

> 安装最新的话需要 webpack>4, node>8；

3.执行 npm run build 即可；

### Vue-router开启 history 模式
``` bash
1. 在 router -> index.js 中
mode: 'history'

2. 在 config -> index.js 中, build 配置中修改
// Paths
assetsRoot: path.resolve(__dirname, '../dist'),
assetsSubDirectory: 'static',
assetsPublicPath: '/',

3. 还需要后端配置
> 参考 https://router.vuejs.org/zh/guide/essentials/history-mode.html

4. 打包
npm run build
```

### 按钮点击态
``` bash
.btn{
  width: 100%;
  height: 80px;
  margin-bottom: 20px;
  text-align: center;
  color: #fff;
  background-color: #ff5052;
  font-size: 34px;
  border-radius: 10px;
}
.btn:active {
  opacity: 0.6;
}
```

### 1-100随机数（整数）
``` bash
function sum(m,n) {
  var num = Math.floor(Math.random()*(m - n) + n)
}

sum(1, 100)
```

### 关闭eslint检查（局部）
``` bash
/* eslint-disable */
  test() {

  }
/* eslint-enable */
```