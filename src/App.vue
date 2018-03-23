<template>
  <div id="app" ref="app">
    <img src="./assets/background-point.png" class="m-background-point" />
    <div class="g-container"
         @click.self="reset">
      <img src="./assets/label1.png" class="f-label f-index5 m-label1"/>
      <img src="./assets/label2.png" class="f-label f-index4 m-label2"/>
      <img src="./assets/label3.png" class="f-label f-index4 m-label3"/>
      <img src="./assets/label4.png" class="f-label f-index5 m-label4"/>
      <img src="./assets/label5.png" class="f-label f-index4 m-label5"/>
      <img src="./assets/label6.png" class="f-label f-index4 m-label6"/>
      <div class="stage f-index3">
        <img src="./assets/label7.png" class="f-label f-index4 m-label7"/>
        <div class="container f-index5">
          <div class="longitude" style="transform: rotateY(0deg)"></div>
          <div class="longitude" style="transform: rotateY(30deg)"></div>
          <div class="longitude" style="transform: rotateY(60deg)"></div>
          <div class="longitude" style="transform: rotateY(90deg)"></div>
          <div class="longitude" style="transform: rotateY(120deg)"></div>
          <div class="longitude" style="transform: rotateY(150deg)"></div>
          <div class="latitude latitude0deg"></div>
          <div class="latitude latitude30deg nlatitude"></div>
          <div class="latitude latitude60deg nlatitude"></div>
          <div class="latitude latitude30deg slatitude"></div>
          <div class="latitude latitude60deg slatitude"></div>
        </div>
      </div>
      <img src="./assets/label8.png" class="f-label f-index3 m-label8"/>
      <img src="./assets/label9.png" class="f-label f-index3 m-label9"/>
      <img src="./assets/label10.png" class="f-label f-index3 m-label10"/>
      <img src="./assets/label11.png" class="f-label f-index2 m-label11"/>
      <div
        :ref="'ballContainer'"
        class="m-ball-container f-index5"
        :class="isPause ? 'paused' : ''">
        <div v-for="(i) in 10"
            :key="i"
            :class="`m-ball m-ball${i} f-index${10-i} ${isSelected && selectedIndex != i ? 'fadeOut' : ''}`"
            @click.stop="clickBall(i)"
            :ref="`ball${i}`">
          <div class="u-ball">
            <img :src="require(`./assets/ball${i}-outer.png`)" class="u-outer" />
            <img :src="require(`./assets/ball${i}-inner.png`)" class="u-inner"/>
            <img :src="require(`./assets/ball${i}-word.png`)" class="u-word"/>
          </div>
        </div>
      </div>
      <div class="m-video f-index6" :class="showVideo ? 'show' : ''">
        <video :src="videoSrc" controls="controls" preload="auto" class="u-video" ref="video">
        </video>
        <div class="u-controller">
          <img src="./assets/player.png" class="u-player"/>
          <a href="javascript:;" :class="!isPlaying ? 'u-start' : 'u-pause'" @click="switchStatus"></a>
        </div>
      </div>
    </div>
  </div>
</template>

<script>
export default {
  name: 'App',
  data: function () {
    return {
      balls: [''],
      isPause: false,
      isSelected: false,
      selectedIndex: -1,
      showVideo: false,
      videoSrc: '',
      isPlaying: false,
      canCancel: false
    }
  },
  watch: {
    isPlaying (newVal) {
      if (newVal) {
        this.$refs['video'].play()
      } else {
        this.$refs['video'].pause()
      }
    }
  },
  mounted: function () {
    for (let i = 1; i < 11; i++) {
      this.balls.push(this.$refs[`ball${i}`][0])
    }
    this.suit()
    window.addEventListener('resize', this.suit.bind(this))
    setInterval(() => {
      if (this.$refs.video && this.$refs.video.paused) {
        this.isPlaying = false
      } else {
        this.isPlaying = true
      }
    }, 300)
  },
  methods: {
    suit () {
      const documentWidth = document.documentElement.clientWidth
      const documentHeight = document.documentElement.clientHeight

      const targetWidth = 7250
      const targetHeight = 4500

      let scale = 1

      if (targetHeight / targetWidth < documentHeight / documentWidth) {
        scale = 1 / (targetWidth / documentWidth)
      } else {
        scale = 1 / (targetHeight / documentHeight)
      }

      const scaleWidth = targetWidth * scale
      const scaleHeight = targetHeight * scale

      const left = (documentWidth - scaleWidth) / 2
      const top = (documentHeight - scaleHeight) / 2

      this.$refs.app.style.transform = `scale(${scale})`
      this.$refs.app.style.left = `${left}px`
      this.$refs.app.style.top = `${top}px`
    },
    getContainerRotateDeg () {
      const matrix = new DOMMatrix(window.getComputedStyle(this.$refs.ballContainer).transform)
      const cosVal = matrix.a
      const sinVal = matrix.b
      if (sinVal > 0) {
        return this.calcDegFromArc(Math.acos(cosVal))
      } else {
        return 180 - this.calcDegFromArc(Math.acos(cosVal)) + 180
      }
    },
    calcDegFromArc (r) {
      return 180 * r / Math.PI
    },
    calcArcFromDeg (deg) {
      return Math.PI / 180 * deg
    },
    calcTransform () {
      const d = 1640
      const alpha = -70
      return `translate(${-Math.cos(this.calcArcFromDeg(90 + alpha - this.getContainerRotateDeg())) * d}px, ${-Math.sin(this.calcArcFromDeg(90 + alpha - this.getContainerRotateDeg())) * d}px) scale(1.3)`
    },
    clickBall (index) {
      if (!this.canCancel) {
        this.canCancel = false
        this.isPause = true
        this.isSelected = true
        this.selectedIndex = index
        this.videoSrc = `https://static.hduzplus.xyz/video/${index}.mp4`
        this.isPlaying = false
        setTimeout(() => {
          this.balls[index].style.transition = 'all 1s ease-in-out'
          this.balls[index].style.transform = this.calcTransform()
        }, 600)
        setTimeout(() => {
          this.showVideo = true
          setTimeout(() => {
            this.canCancel = true
          }, 1000)
        }, 1500)
      }
    },
    reset () {
      if (this.isSelected && this.canCancel) {
        this.canCancel = true
        this.showVideo = false
        this.isPlaying = false
        setTimeout(() => {
          if (typeof this.balls[this.selectedIndex] !== 'undefined') {
            this.balls[this.selectedIndex].style.transform = ''
          }
        }, 600)
        setTimeout(() => {
          if (typeof this.balls[this.selectedIndex] !== 'undefined') {
            this.balls[this.selectedIndex].style.transition = ''
            this.videoSrc = ''
          }
          this.isPause = false
          this.isSelected = false
          this.selectedIndex = -1
          setTimeout(() => {
            this.canCancel = false
          }, 1000)
        }, 1500)
      }
    },
    switchStatus () {
      if (this.isPlaying) {
        this.isPlaying = false
      } else {
        this.isPlaying = true
      }
    }
  }
}
</script>

<style lang="scss">
$radius: 1860px;
$scale: 1.1;
$rotateDuration: 180s;

@import "./assets/math.scss";
@function getTop($index) {
  @return $radius * sin($index * 36deg);
}
@function getLeft($index) {
  @return $radius * cos($index * 36deg);
}
@function half($val) {
  @return $val / 2
}

@keyframes fadeIn {
  from {
    opacity: 0;
    transform: translateY(-20px);
  }
  to {
    opacity: 1;
    transform: translateY(0);
  }
}

@keyframes rotate {
  from {
    transform: rotateZ(0deg)
  }
  to {
    transform: rotateZ(360deg)
  }
}
#app {
  transform-origin: left top;
  position: absolute;
  width: 7250px;
  height: 4500px;
  overflow: hidden;
  background: url('./assets/background.png') no-repeat center center;
  transition: transform .5s linear;
  .f-label, .stage {
    pointer-events: none;
  }
  .m-background-point {
    width: 7077px;
    height: 4420px;
    position: absolute;
    left: 50%;
    top: 50%;
    margin-left: -3538px;
    margin-top: -2210px;
    animation: fadeIn 4s;
    pointer-events: none;
  }
  .f-label {
    position: absolute;
    display: block;
  }
  .f-index10 {
    z-index: 10;
  }
  .f-index9 {
    z-index: 9;
  }
  .f-index8 {
    z-index: 8;
  }
  .f-index7 {
    z-index: 7;
  }
  .f-index6 {
    z-index: 6;
  }
  .f-index5 {
    z-index: 5;
  }
  .f-index4 {
    z-index: 4;
  }
  .f-index3 {
    z-index: 3;
  }
  .f-index2 {
    z-index: 2;
  }
  .f-index1 {
    z-index: 1;
  }
  .g-container {
    position: relative;
    width: 100%;
    height: 100%;
    overflow: hidden;
    .m-label1 {
      right: 345px;
      top: 0;
      width: 2475px;
      height: 672px;
    }
    .m-label2 {
      right: 137px;
      bottom: 35px;
      width: 971px;
      height: 567px;
    }
    .m-label3 {
      width: 1154px;
      height: 953px;
      right: 0;
      top: 85px;
    }
    .m-label4 {
      width: 864px;
      height: 2277px;
      right: 280px;
      top: 935px;
    }
    .m-label5 {
      width: 2260px;
      height: 1217px;
      top: 600px;
      left: 90px;
    }
    .m-label6 {
      width: 1187px;
      height: 1193px;
      left: 50px;
      bottom: 117px;
    }
    .stage {
      perspective: 2000px;
      perspective-origin: center center;
      width: 1077px;
      height: 1077px;
      position:absolute;
      left: 120px;
      top: -40px;
      transform: scale(3);
      transform-origin: left top;
      .m-label7 {
        left: -30px;
        top: 50px;
        width: 1077px;
        height: 912px;
        opacity: .2;
      }
      .container {
        width:100%;
        height:100%;
        position: relative;
        transform-style: preserve-3d;
        animation: circle 30s linear infinite;
      }
      .longitude {
        position: absolute;
        width:100%;
        height:100%;
        border:3px solid #323f5c;
        border-radius: 50%;
      }
      .latitude {
        width: 100%;
        height: 100%;
        border-radius: 50%;
        position: absolute;
        border:3px solid #323f5c;
      }
      .latitude.latitude0deg {
        transform: rotateX(90deg);
      }
      .latitude.latitude30deg.nlatitude {
        transform: translateY(25%) rotateX(90deg) scale(0.866);
      }
      .latitude.latitude30deg.slatitude {
        transform: translateY(-25%) rotateX(90deg) scale(0.866);
      }
      .latitude.latitude60deg.nlatitude {
        transform: translateY(43.3%) rotateX(90deg) scale(0.5);
      }
      .latitude.latitude60deg.slatitude {
        transform: translateY(-43.3%) rotateX(90deg) scale(0.5);
      }
      @keyframes circle {
        from {
          transform: rotateX(-13deg) rotateY(0);
        }
        to {
          transform: rotateX(-13deg) rotateY(360deg);
        }
      }
    }
    .m-label8 {
      width: 1596px;
      height: 1351px;
      left: 60px;
      top: 0;
    }
    .m-label9 {
      width: 3974px;
      height: 3986px;
      left: 50%;
      top: 50%;
      margin-left: -1987px;
      margin-top: -1993px;
      animation: rotate 120s linear;
      animation-iteration-count: infinite;
    }
    .m-label10 {
      width: 2990px;
      height: 2990px;
      left: 50%;
      top: 50%;
      margin-left: -1495px;
      margin-top: -1495px;
      animation: rotate 120s linear reverse;
      animation-iteration-count: infinite;
    }
    .m-label11 {
      width: 6324px;
      height: 3174px;
      transform: translate(-50%, -50%);
      left: 50%;
      top: 50%;
    }
    .m-ball-position {
      position: absolute;
      left: 50%;
      top: 50%;
      transition: all .7s linear;
    }
    .m-ball-container {
      position: absolute;
      left: 50%;
      top: 50%;
      animation: rotate $rotateDuration linear infinite;
      &.paused {
        animation-play-state: paused;
        .m-ball {
          pointer-events: none;
        }
        .u-ball {
          animation-play-state: paused;
        }
      }
      .u-ball {
        animation: rotate $rotateDuration linear reverse infinite;
      }
    }
    .m-ball {
      position: absolute;
      left: 0;
      top: 0;
      transform-origin: center center;
      transition: all .7s linear;
      cursor: pointer;
      &.fadeOut {
        opacity: 0;
      }
      &.m-ball-fake {
        margin-left: 0 !important;
        margin-top: 0 !important;
        transform: none;
      }
      .u-outer {
        width: 100%;
        height: 100%;
        display: block;
        animation: rotate 30s linear infinite;
      }
      .u-inner {
        position: absolute;
        left: 50%;
        top: 50%;
        transform: translate(-50%, -50%);
      }
      .u-word {
        position: absolute;
        top: 126px;
        left: 480px;
        pointer-events: none;
      }
    }
    .m-ball1 {
      width: 1055px;
      height: 1056px;
      margin-left: half(-1055px);
      margin-top: half(-1056px);
      transform: translate($radius, 0);
      &:hover {
        transform: translate($radius, 0) scale($scale);
      }
    }
    .m-ball2 {
      width: 1056px;
      height: 1044px;
      margin-left: half(-1056px);
      margin-top: half(-1044px);
      transform: translate(getLeft(1), getTop(1));
      &:hover {
        transform: translate(getLeft(1), getTop(1)) scale($scale);
      }
    }
    .m-ball3 {
      width: 941px;
      height: 941px;
      margin-left: half(-941px);
      margin-top: half(-941px);
      transform: translate(getLeft(2), getTop(2));
      &:hover {
        transform: translate(getLeft(2), getTop(2)) scale($scale);
      }
    }
    .m-ball4 {
      width: 998px;
      height: 911px;
      margin-left: half(-998px);
      margin-top: half(-911px);
      transform: translate(getLeft(3), getTop(3));
      &:hover {
        transform: translate(getLeft(3), getTop(3)) scale($scale);
      }
    }
    .m-ball5 {
      width: 1005px;
      height: 1005px;
      margin-left: half(-1005px);
      margin-top: half(-1005px);
      transform: translate(getLeft(4), getTop(4));
      &:hover {
        transform: translate(getLeft(4), getTop(4)) scale($scale);
      }
    }
    .m-ball6 {
      width: 941px;
      height: 942px;
      margin-left: half(-941px);
      margin-top: half(-942px);
      transform: translate(getLeft(5), getTop(5));
      &:hover {
        transform: translate(getLeft(5), getTop(5)) scale($scale);
      }
    }
    .m-ball7 {
      width: 941px;
      height: 942px;
      margin-left: half(-941px);
      margin-top: half(-942px);
      transform: translate(getLeft(6), getTop(6));
      &:hover {
        transform: translate(getLeft(6), getTop(6)) scale($scale);
      }
    }
    .m-ball8 {
      width: 998px;
      height: 911px;
      margin-left: half(-998px);
      margin-top: half(-911px);
      transform: translate(getLeft(7), getTop(7));
      &:hover {
        transform: translate(getLeft(7), getTop(7)) scale($scale);
      }
    }
    .m-ball9 {
      width: 1005px;
      height: 1005px;
      margin-left: half(-1005px);
      margin-top: half(-1005px);
      transform: translate(getLeft(8), getTop(8));
      &:hover {
        transform: translate(getLeft(8), getTop(8)) scale($scale);
      }
    }
    .m-ball10 {
      width: 941px;
      height: 942px;
      margin-left: half(-941px);
      margin-top: half(-942px);
      transform: translate(getLeft(9), getTop(9));
      &:hover {
        transform: translate(getLeft(9), getTop(9)) scale($scale);
      }
    }
    .m-video {
      position: absolute;
      left: 2465px;
      top: 1612px;
      transform: translateY(-40px);
      opacity: 0;
      pointer-events: none;
      transition: all 1s;
      &.show {
        transform: translateY(0);
        opacity: 1;
        pointer-events: auto;
      }
      .u-video {
        width: 3405px;
        // background-color: white;
      }
      .u-controller {
        margin-top: 100px;
        text-align: right;
        font-size: 0;
        .u-player {
          display: inline-block;
          margin-right: 90px;
          width: 700px;
          height: 280px;
        }
        .u-start, .u-pause {
          width: 707px;
          height: 293px;
          display: inline-block;
          background-size: 707px 293px;
        }
        .u-start {
          background-image: url(./assets/start_normal.png);
          &:hover {
            background-image: url(./assets/start_hover.png);
          }
        }
        .u-pause {
          background-image: url(./assets/pause_normal.png);
          &:hover {
            background-image: url(./assets/pause_hover.png);
          }
        }
      }
    }
  }
}
</style>
