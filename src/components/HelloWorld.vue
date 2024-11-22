<template>
  <div class="camera_outer">
    <p class="square"></p>
    <!--        playsinline webkit-playsinline="true"-->
    <video
        ref="photo"
        webkit-playsinline="true"
        x-webkit-airplay="true"
        x5-playsinline="true"
        playsinline="true"
        id="videoCamera"
        :width="videoWidth"
        :height="videoHeight"
        autoplay
    >
    </video>
    <div class="board">
      <div class="border-wrap">
        <p class="tip">请按图示将人脸放入取景框中</p>
        <div class="circle-wrap">
          <div class="circle" @click="start()">{{ used ? '重新开始' : '开始' }}</div>
          <div style="width: 15px;height: 1px"></div>
          <div class="circle" @click="setImage()">拍照</div>
        </div>
      </div>
    </div>
    <canvas
        style="display: none"
        :width="canvasWidth"
        :height="canvasHeight"
        id="canvasCamera">
    </canvas>
    <div class="shade">
      <img :src="scan" alt="">
    </div>
    <div v-if="imgSrc" class="img_bg_camera">
      <img :src="imgSrc" alt="" class="tx_img"/>
    </div>
  </div>
</template>
<script>
import scan from "./scan.svg";//遮罩图片
export default {
  data() {
    return {
      used: false,
      scan: scan,
      canvasWidth: 600,
      canvasHeight: 600,
      videoWidth: 600,
      videoHeight: 600,
      imgSrc: "",
      mCanvas: null,
      mContext: null,
      mVideo: null,
    };
  },
  components: {},
  mounted() {
  },
  methods: {
    start() {
      this.getCompetence();
    },
    getCompetence() {
      let _this = this;
      this.videoWidth = this.$refs.photo.clientWidth  //宽度
      this.videoHeight = this.videoWidth;
      this.mCanvas = document.getElementById("canvasCamera");
      this.mContext = this.mCanvas.getContext("2d");
      this.mVideo = document.getElementById("videoCamera");
      // 旧版本浏览器可能根本不支持mediaDevices，我们首先设置一个空对象
      if (navigator.mediaDevices === undefined) {
        navigator.mediaDevices = {};
      }
      // 一些浏览器实现了部分mediaDevices，我们不能只分配一个对象
      // 使用getUserMedia，因为它会覆盖现有的属性。
      // 这里，如果缺少getUserMedia属性，就添加它。
      if (navigator.mediaDevices.getUserMedia === undefined) {
        navigator.mediaDevices.getUserMedia = function (constraints) {
          // 首先获取现存的getUserMedia(如果存在)
          let getUserMedia =
              navigator.webkitGetUserMedia ||
              navigator.mozGetUserMedia ||
              navigator.getUserMedia;
          // 有些浏览器不支持，会返回错误信息
          // 保持接口一致
          if (!getUserMedia) {
            return Promise.reject(
                new Error("getUserMedia is not implemented in this browser")
            );
          }
          // 否则，使用Promise将调用包装到旧的navigator.getUserMedia
          return new Promise(function (resolve, reject) {
            getUserMedia.call(navigator, constraints, resolve, reject);
          });
        };
      }
      if (window.stream) {//一进来的时候判断是否开着摄像头
        window.stream.getTracks().forEach(track => {
          track.stop();
        });
      }
      let constraints = {//配置默认前置摄像头,以及手机上摄像头取到的画面宽高
        audio: false,
        video: {
          width: this.videoWidth,
          height: this.videoHeight,
          transform: "scaleX(-1)",
          sourceId: 'default',
          // facingMode:  { exact: "user" }
          facingMode: "user"
        },
      };
      navigator.mediaDevices
          .getUserMedia(constraints)
          .then(function (stream) {
            // 旧的浏览器可能没有srcObject
            if ("srcObject" in _this.mVideo) {
              _this.$nextTick(() => {
                _this.mVideo.srcObject = stream;//that.video是video标签节点，请自行获取节点，updated周期里可以拿到！
                _this.mVideo.play();
              });
            } else {
              // 避免在新的浏览器中使用它，因为它正在被弃用。
              _this.mVideo.src = window.URL.createObjectURL(stream);
            }
            // _this.mVideo.onloadedmetadata = function (e) {
            //   _this.mVideo.play();
            // };
          })
          .catch((err) => {
            console.log("错误");
            console.log(err);
          });

      if (this.imgSrc) {
        this.imgSrc = ''
      }
      this.used = true;
    },
    //  绘制图片（拍照功能）
    setImage() {
      let _this = this;
      // 点击，canvas画图
      _this.mContext.scale(-1, 1)//如果你上传图片是镜像的加上这段，我这边图片上传后会镜像
      //镜像翻转要往负方向移动图片的距离0,
      _this.mContext.drawImage(_this.mVideo, -parseInt(_this.canvasWidth), 0, _this.canvasWidth, _this.canvasHeight);
      // 获取图片base64链接
      let image = this.mCanvas.toDataURL("image/jpeg", 0.5);
      _this.imgSrc = image;
      let file = _this.dataURLtoFile(image, 'pic')
      // _this.upload(file);

      // 停止摄像机
      _this.mVideo.pause();
      this.stopNavigator();
    },
    // base64转文件
    dataURLtoFile(dataurl, filename) {
      let arr = dataurl.split(",");
      let mime = arr[0].match(/:(.*?);/)[1];
      let bstr = atob(arr[1]);
      let n = bstr.length;
      let u8arr = new Uint8Array(n);
      while (n--) {
        u8arr[n] = bstr.charCodeAt(n);
      }
      return new File([u8arr], filename, {
        type: mime
      });
    },      // 上传图片
    // 关闭摄像头
    stopNavigator() {
      this.mVideo.srcObject.getTracks()[0].stop();
      this.mVideo.srcObject = null
    },
  },
}
</script>
<style lang="scss" scoped>  #videoCamera {
  width: 100%;
}

.close {
  position: absolute;
  top: 0.3rem;
  right: 0.3rem;
}

.vux-x-icon {
  fill: #000;
}

/*遮罩*/
.shade {
  width: 100%;
  position: absolute;
  top: 15vh;
}

.square {
  margin: 0;
  height: 15vh;
  background: #fff;
}

.shade img {
  width: 100%;
}

.board {
  background: #4b4956;
  height: 40vh;
  width: 100%;
  margin-top: -5px;

  .border-wrap {
    width: 100%;
    position: absolute;
    bottom: 0;
    display: flex;
    flex-wrap: wrap;
    justify-content: center;
    align-items: center;
  }

  .tip {
    width: 100%;
    height: 1em;
    text-align: center;
    font-size: 18px;
    color: #fff;
  }

  .circle-wrap {
    width: 100%;
    height: 120px;
    display: flex;
    flex-direction: row;
    justify-content: center;
    align-items: center;

    .circle {
      bottom: 1rem;
      background: #fff;
      padding: 4px 10px;
      //border-radius: 50%; /*transform: all 1s ;*/
    }

    .circle:active {
      width: 1rem;
      height: 1rem;
    }
  }
}

.camera_outer {
  height: 100vh;
  position: relative;
  overflow: hidden;
  background-size: 100%;

  video, canvas, .tx_img {
    -moz-transform: scaleX(-1);
    -webkit-transform: scaleX(-1);
    -o-transform: scaleX(-1);
    transform: scaleX(-1);
  }

  .btn_camera {
    position: absolute;
    bottom: 4px;
    left: 0;
    right: 0;
    height: 50px;
    background-color: rgba(0, 0, 0, 0.3);
    line-height: 50px;
    text-align: center;
    color: #ffffff;
  }

  .bg_r_img {
    position: absolute;
    bottom: 0;
    left: 0;
    right: 0;
    top: 0;
  }

  .img_bg_camera {
    position: absolute;
    top: 0;
    left: 0;
    width: 300px;
    height: 300px;
    z-index: 9999;

    img {
      width: 300px;
      height: 300px;
    }
  }
}
</style>
