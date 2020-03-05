---
title: 移动端开发小技巧
date: 2020-02-03 10:38:20
tags:
---

#### 获取移动端 ua ,来判断设备类型
``` bash
var ua = navigator.userAgent.toLowerCase();

// 是否ios
var bIsIphoneOs = ua.match(/iphone os/i) == "iphone os";

// 是否为android
var bIsAndroid = ua.match(/android/i) == "android";

// 是否在微信浏览器中
function is_weixin(){
  if (ua.match(/MicroMessenger/i) == 'micromessenger') {
      return true;
  } else {
      return false;
  }
}

// 是否在微博浏览器中
function is_weibo(){
  if (ua.match(/Weibo/i) == "weibo") {
      return true;
  } else {
      return false;
  }
}
```

#### 判断是否为移动端
``` bash
if (/Android|Windows Phone|webOS|iPhone|iPod|BlackBerry/i.test(navigator.userAgent)) {
  // 跳转到移动端的链接
    window.location.href=""
} else {
  // 不是移动端
  return
}
```

#### 在移动端中转化时间格式问题
``` bash
// 在移动端中，时间格式为 '2019/03/08 15:28' 这种时间格式，比如
var data = new Date('2019-03-08 15:28');
// 如上的时间格式，转换出来为NaN
// 需要把 '-' 替换成 '/'
var time = '2019-03-08 15:28';
time = time.replace(/-/g, '/');
var data = new Date(time).getTime();
// 才能得出毫秒数 1552030320000
```

#### 解决多个输入框在ios微信下点击错位导致弹窗层消失和无法点击问题
``` bash
scrollViews() {
  setTimeout(() => {
    const scrollT = document.documentElement.scrollTop || document.body.scrollTop || 0
    window.scrollTo(0, Math.max(scrollT - 1, 0))
  }, 200)
}
```

#### 移动端打开指定App或者下载App

``` bash
navToDownApp() {
  let u = navigator.userAgent
  if (/MicroMessenger/gi.test(u)) {
    // 如果是微信客户端打开，引导用户在浏览器中打开
    alert('请在浏览器中打开')
  }
  if (u.indexOf('Android') > -1 || u.indexOf('Linux') > -1) {
    // Android
    if (this.openApp('en://startapp')) {
      this.openApp('en://startapp') // 通过Scheme协议打开指定APP
    } else {
      //跳转Android下载地址
    }
  } else if (u.indexOf('iPhone') > -1) {
    if (this.openApp('ios--scheme')) {
      this.openApp('ios--scheme') // 通过Scheme协议打开指定APP
    } else {
      // 跳转IOS下载地址
    }
  }
},
openApp(src) {
  // 通过iframe的方式试图打开APP，如果能正常打开，会直接切换到APP，并自动阻止a标签的默认行为
  // 否则打开a标签的href链接
  let ifr = document.createElement('iframe')
  ifr.src = src
  ifr.style.display = 'none'
  document.body.appendChild(ifr)
  window.setTimeout(function() {
    // 打开App后移出这个iframe
    document.body.removeChild(ifr)
  }, 2000)
}
```

#### 上传图片把本地图片转换为url

``` bash
let imgUrl = URL.createObjectURL(file)
```

#### 移动端滑屏插件

[fullPage插件](https://alvarotrigo.com/fullPage/zh/)

#### 跨域保存Cookie

``` bash
1. 需要在.html文件中 <head></head> 标签里加上，填写的是主域名
<script>document.domain = 'datangzww.com'</script>

2. 设置Cookie
  # // expires 过期时间 （如一天为24*60*60*1000）
  export const setCookie = (name, value, domain, path, expires) => {
    if (expires) {
      expires = new Date(+new Date() + expires)
    }
    var tempcookie = name + '=' + escape(value) + ((expires) ? '; expires=' + expires.toGMTString() : '') + ((path) ? '; path=' + path : '') + ((domain) ? '; domain=' + domain : '')
    # // Ensure the cookie's size is under the limitation
    if (tempcookie.length < 4096) {
      document.cookie = tempcookie
    }
  }
  
  # // 调用
  # // 设置domain时，填写主域名，需要加个'.';
  setCookie('tf', 1, '.datangzww.com', '/', 0)
    
3. 获取Cookie
  export const getCookie = name => {
    var arr, reg = new RegExp("(^| )" + name + "=([^;]*)(;|$)")
    if (arr = document.cookie.match(reg)) {
      return (arr[2])
    } else {
      return null
    }
  }
  
  # // 调用
  getCookie('tf')
```

### 未完待续...
