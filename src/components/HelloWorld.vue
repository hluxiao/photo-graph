<template>
  <div class="register">
    <!-- 上传头像 -->
    <div>点击图片拍照</div>
    <div id="upImg" class="pos-re">
      <img :src="imgUrl" alt id="faceImg" ref="faceImg" />
      <input type="file" id="upimg" accept="image/*" class="pos-abs" @change="upLoad" />
    </div>
    <div>拍照获取的数据：{{pai}}</div>
    <div>最终上传图片大小:{{size}}k</div>
    <div>选择图片的base64数据：
      <div>
        <textarea name="" id="" cols="30" rows="20" :value='base64'>
          
        </textarea>
      </div>
    </div>
  </div>
</template>
<script>
export default {
  data() {
    return {
      pai:'',
      size:'--',
      base64:'',
      imgUrl: require("../assets/logo.png"),
      imgList: [],
      headimgurl: "",
      selectImg: "" //用户选择的头像
    };
  },
  methods: {
    //图片上传
    fileUpload() {
      
    },

    //图片上传相关
    upLoad(e) {
      let files = e.target.files || e.dataTransfer.files;
      if (!files.length) return;
      this.picavalue = files[0];
      console.log(this.picavalue);
      if (this.picavalue.size / 1024 > 10240) {
        alert("所选图片过大，请重新选择");
        return;
      } else {
        this.showWaiting = true;
        this.imgPreview(this.picavalue);
      }
    },
    imgPreview(file) {
      let that = this;
      let Orientation;
      //这里的Exif是判断图片旋转的
      that.Exif.getData(file, function() {
        Orientation = that.Exif.getTag(file, "Orientation");
        console.log(Orientation);
      });
      if (!file || !window.FileReader) return;
      if (/^image/.test(file.type)) {
        let reader = new FileReader();
        reader.readAsDataURL(file);
        reader.onloadend = function() {
          let result = this.result;
          let img = new Image();
          img.src = result;
          img.onload = () => {
            let data = that.compress(img, Orientation);
            that.imgUrl = result;
            that.imgUrl = data;
            that.base64 = data;
            that.$refs.faceImg.src = data;
            var str = data.replace("data:image/jpeg;base64,", "");
            var strLength = str.length;
            var fileLength = parseInt(strLength - (strLength / 8) * 2);
            // 由字节转换为KB
            var size = "";
            size = parseInt((fileLength / 1024).toFixed(2));
            that.size = size;
            //如果大于10M弹框提示
            let maxSize = 10240; //10M
            if (size > maxSize) {
              alert("所选图片过大，请重新选择");
              return;
            } else {
              //走上传接口
              that.selectImg = str;
              that.fileUpload();
            }
          };
        };
      }
    },
    compress(img, Orientation) {
      let canvas = document.createElement("canvas");
      let ctx = canvas.getContext("2d");
      let initSize = img.src.length;
      let width = img.width / 3.5;
      let height = img.height / 3.5;
      canvas.width = width;
      canvas.height = height;
      ctx.fillStyle = "#fff";
      ctx.fillRect(0, 0, canvas.width, canvas.height);
      ctx.drawImage(img, 0, 0, width, height);
      this.pai = Orientation
      //处理ios手机旋转角度问题
      if (navigator.userAgent.match(/iphone/i)) {
        if (Orientation != "" && Orientation != 1) {
          switch (Orientation) {
            case 6:
              this.rotateImg(img, "left", canvas);
              break;
            case 8:
              this.rotateImg(img, "right", canvas);
              break;
            case 3:
              this.rotateImg(img, "right", canvas); //转两次
              this.rotateImg(img, "right", canvas);
              break;
          }
        }
      } else {
        //处理其安卓类手机图片旋转的问题
        if (Orientation != "" && Orientation != 1) {
          switch (Orientation) {
            case 6:
              this.rotateImg(img, "left", canvas);
              break;
            case 8:
              this.rotateImg(img, "right", canvas);
              break;
            case 3:
              this.rotateImg(img, "right", canvas);
              this.rotateImg(img, "right", canvas);
              break;
          }
        }
      }
      // ndata 为base64格式的图片上传时可直接使用，根据清晰度的要求进行调整，这里我用的是0.4
      var ndata = canvas.toDataURL("image/jpeg", 0.4);
      return ndata;
    },
    rotateImg(img, direction, canvas) {
      //最小与最大旋转方向，图片旋转4次后回到原方向
      const min_step = 0;
      const max_step = 3;
      if (img == null) return;
      //img的高度和宽度不能在img元素隐藏后获取，否则会出错
      //这里因为图片像素太大进行了一个3.5倍的缩放
      let height = img.height / 3.5;
      let width = img.width / 3.5;
      let step = 2;
      if (step == null) {
        step = min_step;
      }
      if (direction == "right") {
        step++;
        //旋转到原位置，即超过最大值
        step > max_step && (step = min_step);
      } else {
        step--;
        step < min_step && (step = max_step);
      }
      //旋转角度以弧度值为参数
      let degree = (step * 90 * Math.PI) / 180;
      let ctx = canvas.getContext("2d");
      console.log(step);
      switch (step) {
        case 0:
          canvas.width = width;
          canvas.height = height;
          ctx.drawImage(img, 0, 0, width, height);
          break;
        case 1:
          canvas.width = height;
          canvas.height = width;
          ctx.rotate(degree);
          ctx.drawImage(img, 0, -height, width, height);
          break;
        case 2:
          canvas.width = width;
          canvas.height = height;
          ctx.rotate(degree);
          ctx.drawImage(img, -width, -height, width, height);
          break;
        case 3:
          canvas.width = height;
          canvas.height = width;
          ctx.rotate(degree);
          ctx.drawImage(img, -width, 0, width, height);
          break;
      }
    }
  },
  created() {},
  mounted() {}
};
</script>
<style lang="less" scoped>
#upImg {
  margin: 0 auto;
  margin-top: 10px;
  width: 130px;
  height: 140px;
  img {
    width: 100%;
    height: 100%;
  }
  #upimg {
    top: 0;
    left: 0;
    width: 100%;
    height: 100%;
    opacity:0;
  }
}
textarea{
  resize: none
}
</style>