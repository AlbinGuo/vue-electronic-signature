<template>
  <div id="app">
    <button @click="sss">签字{{isShowSign}}</button>
    <popup v-if="isShowSign" v-show="isShowSign"  position="bottom" :min-width="100" :min-height="100" title="手写签名" class="sign-area" popup-transition="aa">
      <div style="border:10px solid #ff82b0">
        <mobile-esign :sign-image-min-size="1000" ref="esign_p" bg-color="white" :target-width="200" :is-crop="false" :delay-init-time="300" :aspect-ratio="0.5"></mobile-esign>
      </div>
      <img :src="src1">
      <div>size:{{size}}</div>
      <button @click="resize">resize</button>
      <button @click="generate1" style="width: 1.4rem;height:.7rem" >generate</button>
      <button @click="clear1" style="width: 1.4rem;height:.7rem" >clear</button>
    </popup>
    <!--    <div style="border:10px solid #ff82b0">-->
    <!--      <mobile-esign ref="esign" bg-color="white" :target-width="500" :is-crop="false"></mobile-esign>-->
    <!--    </div>-->
    <!--    <img :src="src">-->
    <!--    <button @click="generate" style="width: 1.4rem;height:.7rem" >generate</button>-->
    <!--    <button @click="clear" style="width: 1.4rem;height:.7rem" >clear</button>-->
    <!--    <div class="aaaa">dasd</div>-->
  </div>
</template>

<script>
  import popup from "ssit-mobile-popup"
  import MobileEsign from "./mobileesign"
  export default {
    name: 'app',
    components:{
      MobileEsign,popup
    },
    data () {
      return {
        src:"",
        src1:"",
        isShowSign:false,
        msg: 'Welcome to Your Vue.js App',size:""
      }
    },
    methods:{
      resize(){
        this.$refs.esign_p.$_resizeHandler()
      },
      sss(){
        this.isShowSign=true
      },
      generate(){
        this.$refs.esign.generate().then(result=>{
          this.src = result;
        }).catch(err=>{
          console.log(err)
          alert("没有签名")
        })
      },
      generate1(){
        this.$refs.esign_p.generate().then(result=>{
          this.src1 = result.base64;
          // var bytes = window.atob(result.base64.split(',')[1]);
          // var array = [];
          // for(var i = 0; i < bytes.length; i++){
          //   array.push(bytes.charCodeAt(i));
          // }
          // var blob = new Blob([new Uint8Array(array)], {type: 'image/jpeg'});
          console.log(result.blob.size)
          this.size=result.blob.size+","+window.devicePixelRatio
        }).catch(err=>{
          console.log(err)
          alert(err.message)
        })
      },
      clear(){
        this.$refs.esign.reset()
      },
      clear1(){
        this.$refs.esign_p.reset()
      }
    }
  }
</script>
<style>
  .ssit-popup{
    top:0 !important;
  }
  .aaaa{
    position: fixed;
    top: 0;
    left: 0;
    width: 100vw;
    height: 100vh;
    color:red;
    background: #ff82b0;
  }
</style>
