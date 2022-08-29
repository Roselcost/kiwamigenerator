<template>
  <div class="absoluteWrapper">
    <div class="wrapper">
      <h1 class="invisibleH1">Kiwami introduction generator</h1>
      <img alt="Kiwami Introduction Generator" class="title"
           src="../assets/title.png">
      <div class="controlsWrapper">
        <div class="controls">
          <div class="control">
            <label for="mainText">Title text</label>
            <input v-model="mainText" name="mainText" id="mainText">
          </div>
          <div class="control">
            <label for="subText">First line</label>
            <input v-model="subText" name="subText" id="subText">
          </div>
          <div class="control">
            <label for="secondLine">Second line</label>
            <input v-model="secondLine" name="secondLine" id="secondLine">
          </div>
          <div class="control">
            <label for="setShadows">Shadows</label>
            <vue-slider style="width: 275px; padding: 15px" name="setShadows" id="setShadows" v-model="shadows" :min="0"
                        :max="100" :disabled="dramatic"/>
          </div>
          <div class="control">
            <label for="setBrightness">Brightness</label>
            <vue-slider style="width: 275px; padding: 15px" name="setBrightness" id="setBrightness" v-model="brightness" :min="100"
                        :max="200" :disabled="dramatic"/>
          </div>
          <div class="control">
            <label for="setContrast">Contrast</label>
            <vue-slider style="width: 275px; padding: 15px" name="setContrast" id="setContrast" v-model="contrast" :min="150"
                        :max="200" :disabled="dramatic"/>
          </div>
          <div class="control" style="justify-content: flex-end">
            <button :disabled="dramatic" @click="setDefaults()">SET DEFAULTS</button>
          </div>
          <hr style="width: 100%">
          <div class="control" style="justify-content: center">
            <label class="inputField">
              <input type="file" accept="image/*" @change="uploadImage($event)"/>
              <span>UPLOAD IMAGE</span>
            </label>
            <button id="downloadImage" type="button" @click="downloadImage()">DOWNLOAD EDITED IMAGE</button>
          </div>
          <div class="control" style="justify-content: center">
            <button v-if="!dramatic" @click="dramaticEffect()">PLAY DRAMATIC EFFECT</button>
            <button v-if="dramatic" @click="stopDramaticEffect()">STOP DRAMATIC EFFECT</button>
          </div>
          <hr style="width: 100%">
          <div class="control" style="flex-direction: column; align-items: flex-start; height: unset">
            <div style="display: flex;cursor:pointer;user-select: none;margin-top:20px" @click="instructions = !instructions">
              <h5 style="margin:0">INSTRUCTIONS</h5><div style="margin-left: 5px;font-size:13px;">{{instructions ? '▲' : '▼'}}</div>
            </div>
            <div v-if="instructions" style="text-align: initial;line-height:25px;margin-top:20px">- Write your text. Second
              line is optional.<br>
              - Upload your image.<br>
              - Adjust shadows, brightness and contrast if you are not satisfied with the default values (image too dark, etc).<br>
              - Adjust zoom and crop the image with the sliders.<br>
              - Download your image<br>
              <hr>
              Note: Unfortunately, there is no way to download the animation. If you want to save it, I recommend you to use a program like OBS Studio to record the
              part of the screen corresponding to the animation.
            </div>
          </div>
          <div class="control">
            <h5>SHARE</h5>
            <div>
              <div style="margin:5px;display:inline-block;vertical-align: middle;height:20px">
                <a href="https://twitter.com/share?ref_src=twsrc%5Etfw" class="twitter-share-button"
                   data-text="Create your own cool character introduction like the ones in the Yakuza videogame series! #Yakuza #Yakuza0 #YakuzaKiwami #RyuGaGotoku"
                   data-url="https://roselcost.github.io/kiwamigenerator/"
                   data-show-count="false">Tweet</a>
              </div>
              <div style="margin:5px;line-height:20px;display:inline-block">
                <div class="fb-share-button" data-href="https://roselcost.github.io/kiwamigenerator/"
                     data-layout="button"
                     data-size="small">Create your own cool introducion like the ones in the Yakuza
                  videogame
                  series!<a target="_blank"
                            href="https://www.facebook.com/sharer/sharer.php?u=https%3A%2F%2Fkiwamigenerator.com%2F&amp;src=sdkpreparse"
                            class="fb-xfbml-parse-ignore">Share</a></div>
              </div>
            </div>
          </div>
        </div>
        <div class="imageEditor">
          <div style="display: flex;flex-direction: column;justify-content: center;align-items: center;">
            <div class="control">
              <div style="width:100%;margin-bottom:10px">
                Zoom
                <div style="width:100%;display:flex;justify-content: center">
                  <div style="width:30px">-</div>
                  <div style="width: 300px">
                    <vue-slider v-model="imageZoom" @change="changeLimits()"/>
                  </div>
                  <div style="width: 30px">+</div>
                </div>
              </div>
            </div>
            <div class="control" style="width:100%">
              <div class="offsetIcon">
                <img alt="offset" src="../assets/offset.png" style="height: 20px; width: 20px;">
              </div>
              <div class="hOffset">
                <vue-slider v-model="hOffset" :min="0"
                            @change="changeXAxis" :disabled="(imgWidth - viewportWidth)===0"
                            :style="{opacity: (imgWidth - viewportWidth) === 0 ? 0.5 : 1}"
                            :max="imgWidth - viewportWidth"/>
              </div>
            </div>
          </div>
          <div style="display:flex;justify-content: space-between">
            <div class="vOffset" style="justify-content: center">
              <div style="height:100%;">
                <vue-slider style="height:100%" v-model="vOffset"
                            @change="changeYAxis" :disabled="(imgHeight - viewportHeight)===0"
                            :style="{opacity: (imgHeight - viewportHeight) === 0 ? 0.5 : 1}"
                            :min="0" :max="imgHeight - viewportHeight"
                            direction="ttb"/>
              </div>
            </div>
            <div id="imgWrapper" class="imgWrapper">
              <div class="image">
                <div class="shadowEffect" :style="setShadowStyle()">
                  <div class="brightness" :style="{'filter': 'brightness(' + brightness + '%'}">
                    <div class="contrast" :style="{'filter': 'contrast(' + (dramatic ? dramaticContrast : contrast) + '%)'}">
                      <img alt="editingImage" id="editing" :style="setImageCoordinates()"
                           style="object-fit: cover;position:absolute"
                           :src="image">
                    </div>
                  </div>
                  <div :style="{opacity: textOpacity}" class="content">
                    <svg class="regularText" width="640" :height="secondLine === '' ? '110' : '140'">
                      <text class="text main" x="50%" y="50" text-anchor="middle"
                            fill="#aa0a0c" stroke="white" stroke-width="3px">{{ mainText }}
                      </text>
                      <text class="text" x="50%" y="95" text-anchor="middle" fill="#aa0a0c"
                            stroke="white" stroke-width="3px">{{
                          secondLine === '' ? subText : subText
                              + ','
                        }}
                      </text>
                      <text class="text" x="50%" y="120" text-anchor="middle" fill="#aa0a0c"
                            stroke="white" stroke-width="3px"
                            style="paint-order: stroke fill;font-family: Edo, sans-serif"><br>{{ secondLine }}
                      </text>
                    </svg>
                    <svg class="narrowText" width="320" :height="secondLine === '' ? '55' : '70'">
                      <text class="text main" x="50%" y="25" text-anchor="middle"
                            fill="#aa0a0c" stroke="white" stroke-width="1.5px">{{ mainText }}
                      </text>
                      <text class="text" x="50%" y="47.5" text-anchor="middle" fill="#aa0a0c"
                            stroke="white" stroke-width="1.5px">{{
                          secondLine === '' ? subText : subText
                              + ','
                        }}
                      </text>
                      <text class="text" x="50%" y="60" text-anchor="middle" fill="#aa0a0c"
                            stroke="white" stroke-width="1.5px"
                            style="paint-order: stroke fill;font-family: Edo, sans-serif"><br>{{ secondLine }}
                      </text>
                    </svg>
                  </div>
                </div>
              </div>
            </div>
          </div>
        </div>
      </div>
      <footer>
        <hr>
        <div style="font-size:12px">
          Made with ❤ by <a style="color:white" href="https://twitter.com/roselcost">Daniel Larrosa</a>
        </div>
      </footer>
    </div>
  </div>
</template>

<script>
import VueSlider from 'vue-slider-component'
import 'vue-slider-component/theme/antd.css'
import domtoimage from 'dom-to-image';

export default {
  name: 'main',
  props: {
    msg: String
  },
  components: {
    VueSlider
  },
  data: function () {
    let vw, vh;
    const audio = new Audio();
    if (this.narrow()) {
      vw = 320;
      vh = 180;
    } else {
      vw = 640;
      vh = 360;
    }
    return {
      mainText: 'Your Name',
      subText: 'CEO of Judgment Studios',
      secondLine: '',
      shadows: 50,
      contrast: 200,
      textOpacity: 1,
      grayscale: 95,
      brightness: 100,
      image: 'dojima.png',
      vOffset: 0,
      hOffset: 0,
      imageZoom: 0,
      imgWidth: 0,
      imgHeight: 0,
      initWidth: 0,
      initHeight: 0,
      viewportWidth: vw,
      viewportHeight: vh,
      xAxis: 50,
      yAxis: 50,
      isNarrow: this.narrow(),
      audio: audio,
      dramatic: false,
      dramaticShadows: 0,
      dramaticContrast: 0,
      instructions: false
    }
  },
  created: function () {
    window.addEventListener("resize", this.adaptResize);
  },
  mounted: function () {
    this.getImageDimensions();
  },
  methods: {
    narrow() {
      return window.innerWidth < 680;
    },
    uploadImage(event) {
      if (event.target.files[0].type.substring(0, 5) === 'image') {
        this.image = URL.createObjectURL(event.target.files[0]);
        this.getImageDimensions();
      } else {
        alert('This file is not an image!')
      }
    },
    adaptResize() {
      if (this.narrow()) {
        this.viewportWidth = 320;
        this.viewportHeight = 180;
        if (!this.isNarrow) {
          this.initWidth /= 2;
          this.initHeight /= 2;
          this.hOffset /= 2;
          this.vOffset /= 2;
          this.imgWidth /= 2;
          this.imgHeight /= 2;
        }
        this.isNarrow = true;
      } else {
        this.viewportWidth = 640;
        this.viewportHeight = 360;
        if (this.isNarrow) {
          this.initWidth *= 2;
          this.initHeight *= 2;
          this.hOffset *= 2;
          this.vOffset *= 2;
          this.imgWidth *= 2;
          this.imgHeight *= 2;

        }
        this.isNarrow = false;
      }
    },
    greaterThan16_9(width, height) {
      return width / height > (16 / 9);
    },
    getImageDimensions() {
      let image = document.getElementById('editing');
      let img = new Image();
      img.src = this.image;
      img.onload = () => {
        this.imageZoom = 0;
        this.vOffset = 0;
        this.hOffset = 0;
        if (this.greaterThan16_9(image.width, image.height)) {
          image.style.width = '';
          image.style.height = '100%';
        } else {
          image.style.width = '100%';
          image.style.height = '';
        }
        this.imgWidth = image.width;
        this.imgHeight = image.height;
        this.initWidth = image.width;
        this.initHeight = image.height;
        if (this.greaterThan16_9(this.imgWidth, this.imgHeight)) {
          this.hOffset = (this.imgWidth - this.viewportWidth) / 2;
        } else {
          this.vOffset = (this.imgHeight - this.viewportHeight) / 2;
        }
      };
    },
    setImageCoordinates() {
      if (this.greaterThan16_9(this.imgWidth, this.imgHeight)) {
        return {
          height: (100 + this.imageZoom) + '%',
          top: -this.vOffset + 'px',
          left: -this.hOffset + 'px',
          filter: 'grayscale(' + this.grayscale + '%)'
        }
      } else {
        return {
          width: (100 + this.imageZoom) + '%',
          top: -this.vOffset + 'px',
          left: -this.hOffset + 'px',
          filter: 'grayscale(' + this.grayscale + '%)'
        }
      }
    },
    changeLimits() {
      this.imgWidth = Math.round(this.initWidth * (1 + this.imageZoom / 100));
      this.imgHeight = Math.round(this.initHeight * (1 + this.imageZoom / 100));
      if (this.imageZoom === 0) {
        this.xAxis = 50;
        this.yAxis = 50;
      }
      this.hOffset = Math.max(0, Math.round((this.imgWidth - this.viewportWidth) * this.xAxis / 100));
      this.vOffset = Math.max(0, Math.round((this.imgHeight - this.viewportHeight) * this.yAxis / 100));
    },
    changeXAxis() {
      this.xAxis = (this.hOffset / (this.imgWidth - this.viewportWidth) * 100);
    },
    changeYAxis() {
      this.yAxis = (this.vOffset / (this.imgHeight - this.viewportHeight) * 100);
    },
    setShadowStyle() {
      let a = 139, b = 150, c = -0;
      if (this.narrow()) {
        a /= 2;
        b /= 2;
        c /= 2;
      }
      if (this.dramatic){
        return {
          'box-shadow': 'inset ' + a + 'px 0px ' + b + 'px ' + c + 'px rgba(0,0,0,' + this.dramaticShadows / 100 + '),' +
              ' inset -' + a + 'px 0px ' + b + 'px ' + c + 'px rgba(0,0,0,' + this.dramaticShadows / 100 + ')'}
      }
      else return {
        'box-shadow': 'inset ' + a + 'px 0px ' + b + 'px ' + c + 'px rgba(0,0,0,' + this.shadows / 100 + '),' +
            ' inset -' + a + 'px 0px ' + b + 'px ' + c + 'px rgba(0,0,0,' + this.shadows / 100 + ')'
      }
    },
    dramaticEffect() {
      this.audio.src = require('../assets/effect.mp3');
      this.dramatic = false;
      this.dramatic = true;
      const shadowValue = this.shadows;
      const contrastValue = this.contrast;
      let interval = -200;
      this.grayscale = 0;
      this.dramaticShadows = 0;
      this.dramaticContrast = 100;
      this.textOpacity = 0;
      const time = setInterval(() => {
        if (interval > 0){
          if (interval < 500){
            this.audio.play();
            this.dramaticShadows = 0;
            this.textOpacity = 0;
          }
          else{
            this.dramaticShadows = Math.min(((interval - 500) / 500) * shadowValue, shadowValue);
            this.textOpacity = Math.min(((interval - 500) / 500), 1);
          }
          this.dramaticContrast = Math.min(Math.max(100, (((interval) / 500)) * contrastValue), contrastValue);
          this.grayscale = Math.min(((interval) / 500) * 95, 95);
        }
        interval = interval + 20;
        if (interval > 4000 || !this.dramatic){
          this.dramatic = false;
          this.textOpacity = 1;
          this.grayscale = 95;
          clearInterval(time);
        }
      }, 20);
    },
    stopDramaticEffect(){
      this.audio.pause();
      this.audio.src = "";
      this.dramatic = false;
    },
    setDefaults(){
      this.contrast = 200;
      this.brightness = 100;
      this.shadows = 50;
    },
    downloadImage() {
      let scale = 'scale(2)';
      if (this.narrow()) {
        scale = 'scale(4)';
      }
      domtoimage.toPng(document.getElementById('imgWrapper'),
          {
            width: 1280, height: 720,
            style: {'transform': scale, 'transform-origin': 'top left'}
          })
          .then(function (dataUrl) {
            var img = new Image();
            img.src = dataUrl;
            let link = document.createElement('a');
            link.download = 'kiwamigenerator.png';
            link.href = dataUrl;
            link.click();
          })
          .catch(function (error) {
            console.error('oops, something went wrong!', error);
          });

    }
  }
}
</script>

<style scoped>
.absoluteWrapper {
  display: flex;
  justify-content: center;
  flex-wrap: wrap;
}

.wrapper {
  max-width: 1500px;
}

.invisibleH1 {
  opacity: 0;
  width: 0;
  height: 0;
}

.title {
  width: calc(100% - 20px);
  margin-bottom: 30px;
  margin-left: 10px;
  margin-right: 10px;
}

.controlsWrapper {
  display: flex;
  flex-wrap: wrap;
  justify-content: center;
}

label.inputField input[type="file"] {
  position: absolute;
  top: -1000px;
}

.inputField {
  border: 1px solid #883c3c;
  background-color: #0000004f;
  display: inline-block;
  padding-top: 5px;
  padding-bottom: 2px;
  padding-left: 20px;
  padding-right: 20px;
  font-size: 13px;
  color: #d9d9d9;
  cursor: pointer;
  margin: 5px;
}

.inputField:hover {
  background: #5e0000;
  transition-duration: 0.5s;
  transition-timing-function: ease;
}

.inputField:active {
  background: #5e0000;
  transition-duration: 0.5s;
  transition-timing-function: ease;
}

button {
  background-color: #0000004f;
  border: 1px solid #883c3c;
  color: #d9d9d9;
  padding-top: 5px;
  padding-left: 20px;
  padding-right: 20px;
  padding-bottom: 2px;
  margin: 5px;
  font-family: "Myriad Pro Light", sans-serif;
  cursor: pointer;
}

button:hover {
  background-color: #5e0000;
  transition-duration: 0.5s;
  transition-timing-function: ease;
}
button:disabled{
  color: #555;
  background-color: #00000054;
  border: 1px solid #533232;
}

.content {
  display: flex;
  justify-content: flex-end;
  align-content: flex-end;
  flex-direction: column;
  position: absolute;
  bottom: 0;
  width: 100%;
}

input {
  width: 270px;
  margin: 10px;
  background-color: #0000004f;
  border: 1px solid #883c3c;
  color: #d9d9d9;
  font-family: "Myriad Pro Light", sans-serif;
  height: 25px;
  padding-left: 5px;
}

.image {
  width: 100%;
  height: 100%;
  position: relative;
}

.shadowEffect {
  width: 100%;
  height: 100%;
}
.contrast{
  width: 100%;
  height: 100%;
  position: relative;
  z-index: -3;
}
.brightness{
  width: 100%;
  height: 100%;
  position: relative;
  z-index: -3;
}

@media (min-width: 680px) {
  .offsetIcon {
    width: 40px;
  }

  .hOffset {
    width: 100%;
    height: 30px;
  }
  .vOffset{
    width: 38px;
    display: flex;
  }

  input {
    width: 270px;
    margin: 10px;
    background-color: #0000004f;
    border: 1px solid #883c3c;
    color: #d9d9d9;
    font-family: "Myriad Pro Light", sans-serif;
    height: 25px;
    padding-left: 5px;
  }

  .control {
    width: 400px;
    display: flex;
    justify-content: space-between;
    align-items: center;
    height: 40px;
  }

  .imgWrapper {
    position: relative;
    overflow: hidden;
    width: 640px;
    height: 360px;
  }

  .imageEditor {
    display: flex;
    width: 680px;
    flex-direction: column;
  }

  .text {
    font-family: Edo, sans-serif;
    color: #c20c0e;
    -webkit-text-stroke: 3px white;
    font-size: 20px;
    font-weight: 500;
    paint-order: stroke fill;
  }

  .text.main {
    font-size: 45px;
  }

  .narrowText {
    display: none;
  }
}

@media (max-width: 680px) {
  .offsetIcon {
    width: 40px;
    text-align: left;
  }

  .hOffset {
    width: 90%;
    height: 30px;
  }
  .vOffset{
    width: 20px;
    display:flex;
  }

  input {
    width: 190px;
    margin: 10px;
    background-color: #0000004f;
    border: 1px solid #883c3c;
    color: #d9d9d9;
    font-family: "Myriad Pro Light", sans-serif;
    height: 25px;
    padding-left: 5px;
  }

  .control {
    width: 300px;
    display: flex;
    justify-content: space-between;
    align-items: center;
    height: 40px;
  }

  .imgWrapper {
    position: relative;
    overflow: hidden;
    width: 320px;
    height: 180px;
  }

  .imageEditor {
    display: flex;
    width: 340px;
    flex-direction: column;
  }

  .text {
    font-family: Edo, sans-serif;
    color: #c20c0e;
    -webkit-text-stroke: 3px white;
    font-size: 10px;
    font-weight: 500;
    paint-order: stroke fill;
  }

  .text.main {
    font-size: 22.5px;
  }

  .regularText {
    display: none;
  }
}

.controls {
  display: flex;
  justify-content: center;
  align-items: center;
  flex-direction: column;
  margin: 25px;
}

img {
  z-index: -3;
}
hr{
  color: #6c2e2e;
  border: 0.6666px solid;
}

footer {
  margin-top: 50px;
}

@keyframes appear {
  0% {
    opacity: 0;
  }
  100% {
    opacity: 1;
  }
}
</style>
