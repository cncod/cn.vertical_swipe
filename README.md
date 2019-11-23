# cn.vertical_swipe
> #### 垂直方向的 swiper 轮播滑动组件
>
> #### 直接修改的 vue-swipe 源码（vue-swipe不支持垂直的滑动）



#### * 下载

```js
// 第一种下载方式
npm i cn.vertical_swipe --save

// 第二种下载方式

git clone https://github.com/cncod/cn.vertical_swipe.git

// 第三种下载方式，下载zip，解压即可
https://github.com/cncod/cn.vertical_swipe/archive/master.zip
```



#### * Vue-cli 中使用

```js
// js ...
import { Swipe, SwipeItem } from 'cn.vertical_swipe';
export default {
    components: {
        Swipe,
        SwipeItem
    }
}

// html 
<template>
    <div ref="swipe" class="user__album">
        <swipe class="my-swipe" :auto='0'>
            <swipe-item class="slide1"></swipe-item>
            <swipe-item class="slide2"></swipe-item>
            <swipe-item class="slide3"></swipe-item>
        </swipe>
    </div>
</template>

// css 
@import '../../../../node_modules/vertical_swipe/dist/vertical_swipe.css';
.my-swipe {
  width: 100%;
  height: 100%;
  color: #fff;
  font-size: 30px;
  text-align: center;
}
 
.slide1 {
  background-color: #0089dc;
  color: #fff;
}
 
.slide2 {
  background-color: #ffd705;
  color: #000;
}
 
.slide3 {
  background-color: #ff2d4b;
  color: #fff;
}
```

#### * 参数

```js
speed             number    动画速度                  默认300
defaultIndex      number    开始索引                  0
disabled          boolean   禁用用户拖动滑动项          false
auto              number    自行滑动延迟               3000
continuous        Boolean   无端点的滑块               true
showIndicators    Boolean   在滑块底部显示指示器         true
noDragWhenSingle  boolean   只有一个刷卡项目时不要拖动    true
prevent           boolean   防止触摸启动时出现默认值     false
propagation       boolean   解决嵌套                  false
disabled          boolean   禁止用户滑动               false

```

#### * 事件

```js
change            当前项发生改变后发生

```

#### * 接口

```js
// 手动跳转某项
this.$refs.swipe.goto(newIndex);

```

