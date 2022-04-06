<template>
  <canvas style="margin: 0 auto" ref="canvas" @mousedown="mouseDown" @mousemove="mouseMove" @mouseup="mouseUp"
          @touchstart="touchStart" @touchmove="touchMove" @touchend="touchEnd"></canvas>
</template>

<script>
  export default {
    props: {
      width: {
        type: Number,
        default: 800
      },
      height: {
        type: Number,
        default: 300
      },
      //高宽比
      aspectRatio:{
        type:Number,
        default(){
          return null
        }
      },
      lineWidth: {
        type: Number,
        default: 4
      },
      lineColor: {
        type: String,
        default: '#000000'
      },
      bgColor: {
        type: String,
        default: ''
      },
      isCrop: {
        type: Boolean,
        default: false
      },
      targetWidth:{
        type:Number,
        default:-1
      },
      delayInitTime:{
        type:Number,
        default(){
          return 0;
        }
      },
      maxHeightPercent:{
        type:Number,
        default(){
          return .6
        }
      },
      //签字后的图片和未签字的图片的大小差值的最小值，小于该值则认为签字不合规
      signImageMinSize:{
        type:Number,
        default(){
          return -1
        }
      }
    },
    data () {
      return {
        hasDrew: false,
        resultImg: '',
        points: [],
        canvasTxt: null,
        startX: 0,
        startY: 0,
        isDrawing: false,
        sratio: 1,
        emptyData:null,
        emptySize:0
      }
    },
    computed: {
      ratio () {
        if(this.aspectRatio>0){
          return this.aspectRatio;
        }else{
          return this.height / this.width
        }
      },
      stageInfo () {
        return this.$refs.canvas.getBoundingClientRect()
      },
      myBg () {
        return this.bgColor ? this.bgColor : 'rgba(255, 255, 255, 0)'
      }
    },
    watch: {
      'myBg': function (newVal) {
        this.$refs.canvas.style.background = newVal
      }
    },
    beforeMount () {
      window.addEventListener('resize', this.$_resizeHandler)
    },
    beforeDestroy () {
      window.removeEventListener('resize', this.$_resizeHandler)
    },
    mounted () {
      let canvasSize = this.canvasSize();
      const canvas = this.$refs.canvas
      canvas.height = canvasSize.height
      canvas.width = canvasSize.width
      canvas.style.background = this.myBg
      if(this.delayInitTime>0){
        setTimeout(()=>{
          this.$_resizeHandler()
        },this.delayInitTime)
      }else{
        this.$_resizeHandler()
      }

      // 在画板以外松开鼠标后冻结画笔
      document.onmouseup = () => {
        this.isDrawing = false
      }
      this.emptyData = this.$refs.canvas.toDataURL();
      this.compressImage(this.emptyData,this.targetWidth,"image/jpeg",(base64)=>{
        this.emptySize = this.imageBlob(base64).size
        console.log("emptySize=",this.emptySize)
      })
    },
    methods: {
      $_resizeHandler () {
        let canvasSize = this.canvasSize();
        console.log(canvasSize)
        let width = canvasSize.width;
        const canvas = this.$refs.canvas
        canvas.style.width = width + "px"
        setTimeout(()=>{
          const realw = parseFloat(window.getComputedStyle(canvas).width)
          console.log("realw",realw)
          canvas.style.height = this.ratio * realw + "px";
          this.canvasTxt = canvas.getContext('2d')
          this.canvasTxt.scale(1 * this.sratio, 1 * this.sratio)
          this.sratio = realw / canvas.width
          this.canvasTxt.scale(1 / this.sratio, 1 / this.sratio)
        },this.delayInitTime)

      },
      // pc
      mouseDown (e) {
        e = e || event
        e.preventDefault()
        this.isDrawing = true
        this.hasDrew = true
        let obj = {
          x: e.offsetX,
          y: e.offsetY
        }
        this.drawStart(obj)
      },
      mouseMove (e) {
        e = e || event
        e.preventDefault()
        if (this.isDrawing) {
          let obj = {
            x: e.offsetX,
            y: e.offsetY
          }
          this.drawMove(obj)
        }
      },
      mouseUp (e) {
        e = e || event
        e.preventDefault()
        let obj = {
          x: e.offsetX,
          y: e.offsetY
        }
        this.drawEnd(obj)
        this.isDrawing = false
      },
      // mobile
      touchStart (e) {
        e = e || event
        e.preventDefault()
        if (e.touches.length >= 1) {
          this.hasDrew = true;
          let obj = {
            x: e.targetTouches[0].clientX - this.$refs.canvas.getBoundingClientRect().left,
            y: e.targetTouches[0].clientY - this.$refs.canvas.getBoundingClientRect().top
          }
          this.drawStart(obj)
        }
      },
      touchMove (e) {
        e = e || event
        e.preventDefault()
        if (e.touches.length >= 1) {
          let obj = {
            x: e.targetTouches[0].clientX - this.$refs.canvas.getBoundingClientRect().left,
            y: e.targetTouches[0].clientY - this.$refs.canvas.getBoundingClientRect().top
          }
          this.drawMove(obj)
        }
      },
      touchEnd (e) {
        e = e || event
        e.preventDefault()
        if (e.touches.length >= 1) {
          let obj = {
            x: e.targetTouches[0].clientX - this.$refs.canvas.getBoundingClientRect().left,
            y: e.targetTouches[0].clientY - this.$refs.canvas.getBoundingClientRect().top
          }
          this.drawEnd(obj)
        }
      },
      // 绘制
      drawStart (obj) {
        this.startX = obj.x
        this.startY = obj.y
        this.canvasTxt.beginPath()
        this.canvasTxt.moveTo(this.startX, this.startY)
        this.canvasTxt.lineTo(obj.x, obj.y)
        this.canvasTxt.lineCap = 'round'
        this.canvasTxt.lineJoin = 'round'
        this.canvasTxt.lineWidth = this.lineWidth * this.sratio
        this.canvasTxt.stroke()
        this.canvasTxt.closePath()
        this.points.push(obj)
      },
      drawMove (obj) {
        this.canvasTxt.beginPath()
        this.canvasTxt.moveTo(this.startX, this.startY)
        this.canvasTxt.lineTo(obj.x, obj.y)
        this.canvasTxt.strokeStyle = this.lineColor
        this.canvasTxt.lineWidth = this.lineWidth * this.sratio
        this.canvasTxt.lineCap = 'round'
        this.canvasTxt.lineJoin = 'round'
        this.canvasTxt.stroke()
        this.canvasTxt.closePath()
        this.startY = obj.y
        this.startX = obj.x
        this.points.push(obj)
      },
      drawEnd (obj) {
        this.canvasTxt.beginPath()
        this.canvasTxt.moveTo(this.startX, this.startY)
        this.canvasTxt.lineTo(obj.x, obj.y)
        this.canvasTxt.lineCap = 'round'
        this.canvasTxt.lineJoin = 'round'
        this.canvasTxt.stroke()
        this.canvasTxt.closePath()
        this.points.push(obj)
        this.points.push({x: -1, y: -1})
      },
      // 操作
      generate () {
        let _this = this;
        const pm =  new Promise((resolve, reject) => {

          if (!this.hasDrew||this.emptyData==this.$refs.canvas.toDataURL()) {
            reject({error:1,message:'未签字'})
            return
          }
          // let width = this.$refs.canvas.width;
          // let height = this.$refs.canvas.height;
          let width = parseFloat(window.getComputedStyle(this.$refs.canvas).width);
          let height = parseFloat(window.getComputedStyle(this.$refs.canvas).height)

          var resImgData = this.canvasTxt.getImageData(0, 0, width,height)
          this.canvasTxt.globalCompositeOperation = "destination-over"
          this.canvasTxt.fillStyle = this.myBg
          this.canvasTxt.fillRect(0,0,width ,height)
          this.resultImg = this.$refs.canvas.toDataURL()
          var resultImg = this.resultImg
          this.canvasTxt.clearRect(0, 0, width ,height)
          this.canvasTxt.putImageData(resImgData, 0, 0)
          this.canvasTxt.globalCompositeOperation = "source-over"
          if (this.isCrop) {
            const crop_area = this.getCropArea(resImgData.data)
            var crop_canvas = document.createElement('canvas')
            const crop_ctx = crop_canvas.getContext('2d')
            crop_canvas.width = crop_area[2] - crop_area[0]
            crop_canvas.height = crop_area[3] - crop_area[1]
            const crop_imgData = this.canvasTxt.getImageData(...crop_area)
            crop_ctx.globalCompositeOperation = "destination-over"
            crop_ctx.putImageData(crop_imgData, 0, 0)
            crop_ctx.fillStyle = this.myBg
            crop_ctx.fillRect(0, 0, crop_canvas.width , crop_canvas.height)
            resultImg = crop_canvas.toDataURL()
            crop_canvas = null
          }
          if(_this.targetWidth>0 && _this.targetWidth<width){
            _this.compressImage(resultImg,_this.targetWidth,"image/jpeg",(result)=>{
              let blob = _this.imageBlob(result);
              if(_this.signImageMinSize>0 && _this.signImageMinSize>(blob.size-_this.emptySize)){
                reject({error:2,message:'签字不符合要求'})
                return;
              }
              resolve({base64:result,blob})
            })
          }else{
            let blob = _this.imageBlob(resultImg);
            if(_this.signImageMinSize>0 && _this.signImageMinSize>(blob.size-_this.emptySize)){
              reject({error:2,message:'签字不符合要求'})
              return;
            }
            resolve({base64:resultImg,blob})
          }
        })
        return pm
      },
      compressImage(path, targetWidth,type, callback){
        let img = new Image();
        img.src = path;
        img.onload = function(){
          // 默认按比例压缩
          let width = img.width,
            height = img.height,
            scale = width / height;
          width = targetWidth || width;
          height =  (width / scale);
          let quality = 0.7;  // 默认图片质量为0.7
          console.log(width,height);
          //生成canvas
          let canvas = document.createElement('canvas');
          let ctx = canvas.getContext('2d');
          // 创建属性节点
          let anw = document.createAttribute("width");
          anw.nodeValue = width;
          let anh = document.createAttribute("height");
          anh.nodeValue = height;
          canvas.setAttributeNode(anw);
          canvas.setAttributeNode(anh);
          ctx.drawImage(img, 0, 0, width, height);
          // quality值越小，所绘制出的图像越模糊
          type = type || 'image/jpeg'
          let base64 = canvas.toDataURL(type, quality);
          // 回调函数返回base64的值
          callback(base64);
        }
      },
      reset () {
        this.canvasTxt.clearRect(
          0,
          0,
          parseFloat(window.getComputedStyle(this.$refs.canvas).width),
          parseFloat(window.getComputedStyle(this.$refs.canvas).height)
        )
        this.$emit('update:bgColor', '')
        this.$refs.canvas.style.background = this.myBg
        this.points = []
        this.hasDrew = false
        this.resultImg = ''
      },
      getCropArea (imgData) {
        var topX = this.$refs.canvas.width; var btmX = 0; var topY = this.$refs.canvas.height; var btnY = 0
        for (var i = 0; i < this.$refs.canvas.width; i++) {
          for (var j = 0; j < this.$refs.canvas.height; j++) {
            var pos = (i + this.$refs.canvas.width * j) * 4
            if (imgData[pos] > 0 || imgData[pos + 1] > 0 || imgData[pos + 2] || imgData[pos + 3] > 0) {
              btnY = Math.max(j, btnY)
              btmX = Math.max(i, btmX)
              topY = Math.min(j, topY)
              topX = Math.min(i, topX)
            }
          }
        }
        topX++
        btmX++
        topY++
        btnY++
        const data = [topX, topY, btmX, btnY]
        return data
      },
      canvasSize(){
        let width,height;
        let winWidth = document.documentElement.clientWidth;
        let winHeight = document.documentElement.clientHeight;
        //当设置了aspectRatio时候，则忽略设置的width和height，按照宽度最大为全屏宽，高度最大为全屏高的90%
        if(this.aspectRatio>0){
          //宽度全屏时候，高度超过屏幕90%，则设置高度全屏90%，宽度按比例来
          if(winWidth*this.aspectRatio>winHeight*this.maxHeightPercent){
            height = winHeight*this.maxHeightPercent;
            width = height/this.aspectRatio;
          }else{//宽度全屏，高度按比例来
            width = winWidth;
            height = winWidth*this.aspectRatio;
          }
        }else{
          width = this.width;
          height = this.height;
        }
        return {width,height}
      },
      imageBlob(base64){
        var bytes = window.atob( base64.split(',')[1]);
        var array = [];
        for(var i = 0; i < bytes.length; i++){
          array.push(bytes.charCodeAt(i));
        }
        var blob = new Blob([new Uint8Array(array)], {type: 'image/jpeg'});
        return blob;
      }
    }
  }
</script>

<style scoped>
  canvas {
    max-width: 100%;
    display: block;
  }
</style>
