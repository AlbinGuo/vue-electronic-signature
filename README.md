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
      <h4>Electronic signature</h4>
      <ssit-mobile-esign ref="esign"  :width="600" :height="300" :is-crop="false" :line-width="3" line-color="#303030"
                                                       bg-color="lightgray" :target-width="300">
      </ssit-mobile-esign>
      <img :src=src>
      <button @click="generate">generate img</button>
      <button @click="reset">reset</button>
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

|  attr   | description  |  type   | default  |
|  ----  | ----  |  ----  | ----  |
| width  | Signature area width | Number  | 800 |
| height  | Signature area height | Number  | 300 |
| lineWidth  | The line width of the signature brush | Number  | 4 |
| lineColor  | Signature brush color | String  | '#000000' |
| bgColor  | Signature background color | String  | '' |
| isCrop  | Whether to crop out blank areas | Boolean  | false |
| targetWidth  | The width of the final generated signature image, -1 means the same as width | Number  | -1 |

|  method   | description  |  param   | return  |
|  ----  | ----  |  ----  | ----  |
| reset  | clear signing | null  | null | null |
| generate  | Generate signature image | null  | null | generate Is an asynchronous function，Get the result through the.then function |


For detailed explanation on how things work, consult the [docs for vue-loader](http://vuejs.github.io/vue-loader).
