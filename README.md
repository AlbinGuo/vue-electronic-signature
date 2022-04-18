# vue-electronic-signature

Mobile electronic signature.

<h1 align="center">
Vue-electronic-signature
</h1>
<p align="center">
vue-electronic-signature is a Mobile electronic signature component.
<p>
<p align="center">
  <a href="https://www.npmjs.com/package/@guonei001/vue-electronic-signature"><img src="https://img.shields.io/npm/v/@guonei001/vue-electronic-signature?color=729B1B&label="></a>
<p>

#### Description
A Vue.js component for creating polls, 
voting and showing results. It’s easy to implement and easy to customize.
It`s based on vue2+

#### Install
```
npm i vue-vote
```

## Build Setup

``` bash
<ClientOnly>
<template>
  <div class="mobile-container">
      <h4>电子签名</h4>
      <ssit-mobile-esign ref="esign"  :width="600" :height="300" :is-crop="false" :line-width="3" line-color="#303030"
                                                       bg-color="lightgray" :target-width="300">
      </ssit-mobile-esign>
      <img :src=src>
      <button @click="generate">生成图片</button>
      <button @click="reset">重置</button>
  </div>
</template>
<script>
export default {
    data() {
        return {
          src:""
        }
    },
    methods:{
        generate(){
            this.$refs.esign.generate().then(res=>{
              this.src = res;
            });
        },
        reset(){
          this.$refs.esign.reset();
        }
        
    }

}
</script>
<style>
</style>
</ClientOnly>
```

|  属性   | 说明  |  类型   | 默认值  |
|  ----  | ----  |  ----  | ----  |
| width  | 签名区宽度 | Number  | 800 |
| height  | 签名区高度 | Number  | 300 |
| lineWidth  | 签名画笔的线条宽度 | Number  | 4 |
| lineColor  | 签名画笔颜色 | String  | '#000000' |
| bgColor  | 签名背景颜色 | String  | '' |
| isCrop  | 是否裁剪掉空白区域 | Boolean  | false |
| targetWidth  | 最终生成的签名图片的宽度,-1表示与width一致 | Number  | -1 |

|  方法   | 说明  |  参数   | 返回值  |
|  ----  | ----  |  ----  | ----  |
| reset  | 清除签名 | 无  | 无 | 无 |
| generate  | 生成签名图片 | 无  | 无 | generate是异步函数，通过.then函数拿结果 |


For detailed explanation on how things work, consult the [docs for vue-loader](http://vuejs.github.io/vue-loader).
