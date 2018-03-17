<template>
  <div id="app">
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
        <video src="" class="u-video"/>
        <div class="u-controller">
          <img src="./assets/player.png" class="u-player" />
          <a href="javascript:;" class="u-start"></a>
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
      balls: [],
      isPause: false,
      isSelected: false,
      selectedIndex: -1,
      showVideo: false
    }
  },
  mounted: function () {
    for (let i = 1; i < 11; i++) {
      this.balls.push(this.$refs[`ball${i}`][0])
    }
  },
  methods: {
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
      const d = 600
      const alpha = -60
      return `translate(${-Math.cos(this.calcArcFromDeg(90 + alpha - this.getContainerRotateDeg())) * d}px, ${-Math.sin(this.calcArcFromDeg(90 + alpha - this.getContainerRotateDeg())) * d}px) scale(1.3)`
    },
    clickBall (index) {
      this.isPause = true
      this.isSelected = true
      this.selectedIndex = index
      setTimeout(() => {
        this.$refs[`ball${index}`][0].style.transition = 'all 1s ease-in-out'
        this.$refs[`ball${index}`][0].style.transform = this.calcTransform()
      }, 800)
      setTimeout(() => {
        this.showVideo = true
      }, 2000)
    },
    reset () {
      if (this.isSelected) {
        this.showVideo = false
        setTimeout(() => {
          this.$refs[`ball${this.selectedIndex}`][0].style.transform = ''
        }, 800)
        setTimeout(() => {
          this.$refs[`ball${this.selectedIndex}`][0].style.transition = ''
          this.isPause = false
          this.isSelected = false
          this.selectedIndex = -1
        }, 1900)
      }
    }
  }
}
</script>

<style lang="scss">
$radius: 620px;
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
  position: relative;
  width: 2440px;
  height: 1514px;
  overflow: hidden;
  background: url('./assets/background.png') no-repeat center center;
  .f-label, .stage {
    pointer-events: none;
  }
  .m-background-point {
    width: 2383px;
    height: 1488px;
    position: absolute;
    left: 50%;
    top: 50%;
    margin-left: -1191px;
    margin-top: -744px;
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
      right: 116px;
      top: 0;
      width: 834px;
      height: 227px;
    }
    .m-label2 {
      right: 46px;
      bottom: 17px;
      width: 327px;
      height: 192px;
    }
    .m-label3 {
      width: 389px;
      height: 322px;
      right: 0;
      top: 28px;
    }
    .m-label4 {
      width: 292px;
      height: 750px;
      right: 93px;
      top: 312px;
    }
    .m-label5 {
      width: 762px;
      height: 411px;
      top: 200px;
      left: 30px;
    }
    .m-label6 {
      width: 401px;
      height: 402px;
      left: 16px;
      bottom: 39px;
    }
    .stage {
      perspective: 2000px;
      perspective-origin: center center;
      width: 1077px;
      height: 1077px;
      position:absolute;
      left: 120px;
      top: -40px;
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
      width: 539px;
      height: 455px;
      left: 20px;
      top: 0;
    }
    .m-label9 {
      width: 1338px;
      height: 1342px;
      left: 50%;
      top: 50%;
      margin-left: -669px;
      margin-top: -671px;
      animation: rotate 120s linear;
      animation-iteration-count: infinite;
    }
    .m-label10 {
      width: 1008px;
      height: 1006px;
      left: 50%;
      top: 50%;
      margin-left: -504px;
      margin-top: -503px;
      animation: rotate 120s linear reverse;
      animation-iteration-count: infinite;
    }
    .m-label11 {
      width: 2129px;
      height: 1096px;
      margin-left: -1064px;
      margin-top: -548px;
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
        top: 42px;
        left: 160px;
        pointer-events: none;
      }
    }
    .m-ball1 {
      width: 356px;
      height: 357px;
      margin-left: half(-356px);
      margin-top: half(-357px);
      transform: translate($radius, 0);
      &:hover {
        transform: translate($radius, 0) scale($scale);
      }
    }
    .m-ball2 {
      width: 356px;
      height: 357px;
      margin-left: half(-356px);
      margin-top: half(-357px);
      transform: translate(getLeft(1), getTop(1));
      &:hover {
        transform: translate(getLeft(1), getTop(1)) scale($scale);
      }
    }
    .m-ball3 {
      width: 318px;
      height: 317px;
      margin-left: half(-318px);
      margin-top: half(-317px);
      transform: translate(getLeft(2), getTop(2));
      &:hover {
        transform: translate(getLeft(2), getTop(2)) scale($scale);
      }
    }
    .m-ball4 {
      width: 336px;
      height: 308px;
      margin-left: half(-336px);
      margin-top: half(-308px);
      transform: translate(getLeft(3), getTop(3));
      &:hover {
        transform: translate(getLeft(3), getTop(3)) scale($scale);
      }
    }
    .m-ball5 {
      width: 339px;
      height: 339px;
      margin-left: half(-339px);
      margin-top: half(-339px);
      transform: translate(getLeft(4), getTop(4));
      &:hover {
        transform: translate(getLeft(4), getTop(4)) scale($scale);
      }
    }
    .m-ball6 {
      width: 318px;
      height: 317px;
      margin-left: half(-318px);
      margin-top: half(-317px);
      transform: translate(getLeft(5), getTop(5));
      &:hover {
        transform: translate(getLeft(5), getTop(5)) scale($scale);
      }
    }
    .m-ball7 {
      width: 318px;
      height: 317px;
      margin-left: half(-318px);
      margin-top: half(-317px);
      transform: translate(getLeft(6), getTop(6));
      &:hover {
        transform: translate(getLeft(6), getTop(6)) scale($scale);
      }
    }
    .m-ball8 {
      width: 336px;
      height: 308px;
      margin-left: half(-336px);
      margin-top: half(-308px);
      transform: translate(getLeft(7), getTop(7));
      &:hover {
        transform: translate(getLeft(7), getTop(7)) scale($scale);
      }
    }
    .m-ball9 {
      width: 339px;
      height: 339px;
      margin-left: half(-339px);
      margin-top: half(-339px);
      transform: translate(getLeft(8), getTop(8));
      &:hover {
        transform: translate(getLeft(8), getTop(8)) scale($scale);
      }
    }
    .m-ball10 {
      width: 318px;
      height: 317px;
      margin-left: half(-318px);
      margin-top: half(-317px);
      transform: translate(getLeft(9), getTop(9));
      &:hover {
        transform: translate(getLeft(9), getTop(9)) scale($scale);
      }
    }
    .m-video {
      position: absolute;
      left: 827px;
      top: 540px;
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
        width: 1135px;
        height: 635px;
        background-color: white;
      }
      .u-controller {
        margin-top: 53px;
        text-align: right;
        font-size: 0;
        .u-player {
          display: inline-block;
          margin-right: 10px;
          width: 237px;
          height: 95px;
        }
        .u-start {
          width: 239px;
          height: 100px;
          display: inline-block;
          background-image: url(./assets/start_normal.png);
          &:hover {
            background-image: url(./assets/start_hover.png);
          }
        }
      }
    }
  }
}
</style>
