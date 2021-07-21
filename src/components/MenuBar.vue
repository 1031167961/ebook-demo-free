<template>
      <div class="menu-bar">
        <transition name="slide-up">
          <div class="menu-wrapper" v-show="ifTitleAndMenuShow" :class="{'hide-box-shadow': ifSettingShow || !ifTitleAndMenuShow}">
            <div class="icon-wrapper">
              <span class="icon-menu icon" @click="showSetting(3)"></span>
            </div>
            <div class="icon-wrapper">
              <span class="icon-progress icon" @click="showSetting(2)"></span>
            </div>
            <div class="icon-wrapper">
              <span class="icon-bright icon" @click="showSetting(1)"></span>
            </div>
            <div class="icon-wrapper">
              <span class="icon-a icon" @click="showSetting(0)">A</span>
            </div>
          </div>
        </transition>
        <transition name="slide-up">
          <div class="setting-wrapper" v-show="ifSettingShow">
            <!-- 字体设置 -->
            <div class="setting-font-size" v-if="showTag === 0">
              <div class="preview" :style="{fontSize: fontSizeList[0].fontSize + 'px'}">A</div>
              <div class="select">
                <div class="select-wrapper" v-for="(item, index) in fontSizeList" :key="index" @click="setFontSize(item.fontSize)">
                  <div class="line"></div>
                  <div class="point-wrapper">
                    <div class="point" v-show="defaultFontSize === item.fontSize">
                      <div class="small-point"></div>
                    </div>
                  </div>
                  <div class="line"></div>
                </div>
              </div>
              <div class="preview" :style="{fontSize: fontSizeList[fontSizeList.length - 1].fontSize + 'px'}">A</div>
            </div>
            <!-- 主题设置 -->
            <div class="setting-theme" v-else-if="showTag === 1">
              <div class="setting-theme-item" v-for="(item, index) in themeList" :key="index" @click="setTheme(index)">
                <div class="preview" :style="{background: item.style.body.background}" :class="{'no-border': item.style.body.background !=='#fff'}"></div>
                <div class="text" :class="{'slected': index === defaultTheme}">{{item.name}}</div>
              </div>
            </div>
            <!-- 进度条设置 -->
            <div class="setting-progress" v-else-if="showTag === 2">
              <div class="progress-wrapper">
                <input class="progress" type="range"
                                        max="100"
                                        min="0"
                                        step="1"
                                        @change="onProgressChange($event.target.value)"
                                        @input="onProgressInput($event.target.value)"
                                        :value="progress"
                                        :disabled="!bookAvailable"
                                        ref="progress">
                <div class="text-wrapper">
                  <span>{{bookAvailable ? progress + "%" : '加载中...'}}</span>
                </div>
              </div>
            </div>
          </div>
        </transition>
        <!-- 目录组件 -->
        <content-view :ifShowContent="ifShowContent"
                      v-show="ifShowContent"
                      :bookAvailable="bookAvailable"
                      :navigation="navigation"
                      @jumpTo="jumpTo"></content-view>
        <transition name="fade">
          <!-- 蒙版层 -->
          <div class="content-mask" v-show="ifShowContent" @click="hideContent"></div>
        </transition>
      </div>
</template>

<script>
/* eslint-disable keyword-spacing, spaced-comment */
import ContentView from '@/components/ContentView'
export default {
  components: {
    ContentView
  },
  props: {
    ifTitleAndMenuShow: {
      type: Boolean,
      default: false
    },
    fontSizeList: Array,
    defaultFontSize: Number,
    themeList: Array,
    defaultTheme: Number,
    bookAvailable: {
      type: Boolean,
      default: false
    },
    navigation: Object
  },
  data() {
    return {
      ifSettingShow: false,
      showTag: 0,
      progress: 0,
      ifShowContent: false
    }
  },
  methods: {
    // 拖动进度条触发事件
    onProgressInput(progress) {
      this.progress = progress
      this.$refs.progress.style.backgroundSize = `${this.progress}% 100%`
    },
    // 进度条松开后出发的事件，根据进度条数值跳转到指定位置
    onProgressChange(progress) {
      this.$emit('onProgressChange', progress)
    },
    // 设置主题
    setTheme(index) {
      this.$emit('setTheme', index)
    },
    // 设置字号
    setFontSize(fontSize) {
      // 调用父组件中的setFontSize方法
      this.$emit('setFontSize', fontSize)
    },
    showSetting(tag) {
      this.showTag = tag
      if(this.showTag === 3) {
        this.ifSettingShow = false
        this.ifShowContent = true
      } else {
        this.ifSettingShow = true
      }
    },
    // 隐藏设置栏
    hideSetting() {
      this.ifSettingShow = false
    },
    // 隐藏目录
    hideContent() {
      this.ifShowContent = false
    },
    // 调用父组件jumpTo()方法，跳转到目录指定位置
    jumpTo(target) {
      this.$emit('jumpTo', target)
    }
  }
}
</script>

<style lang="scss" rel="stylesheet/scss" scoped>
@import '../assets/styles/global';
.menu-bar {
  .menu-wrapper {
    position: absolute;
    bottom: 0;
    left: 0;
    z-index: 101;
    display: flex;
    width: 100%;
    height: pr(48);
    background: #fff;
    box-shadow: 0 pr(-8) pr(8) rgba(0, 0, 0, 0.2);
    &.hide-box-shadow {
      box-shadow: none;
    }
    .icon-wrapper {
      flex: 1;
      @include center;
      .icon-progress {
        font-size: pr(28);
      }
      .icon-bright {
        font-size: pr(24);
      }
    }
  }
  .setting-wrapper {
    position: absolute;
    bottom: pr(48);
    left: 0;
    z-index: 101;
    width: 100%;
    height: pr(60);
    background: #fff;
    box-shadow: 0 pr(-8) pr(8) rgba(0, 0, 0, 0.2);
    // 字体设置
    .setting-font-size {
      display: flex;
      height: 100%;
      .preview {
        flex: 0 0 pr(40);
        @include center;
      }
      .select {
        display: flex;
        flex: 1;
        .select-wrapper {
          flex: 1;
          display: flex;
          align-items: center;
          &:first-child {
            .line {
              &:first-child {
                border-top: none;
              }
            }
          }
          &:last-child {
            .line {
              &:last-child {
                border-top: none;
              }
            }
          }
          .line {
            flex: 1;
            height: 0;
            border-top: pr(1) solid #ccc;
          }
          .point-wrapper {
            position: relative;
            flex: 0 0 0;
            width: 0;
            height: pr(7);
            border-left: pr(1) solid #ccc;
            .point {
              position: absolute;
              top: pr(-7);
              left: pr(-10);
              width: pr(20);
              height: pr(20);
              border-radius: 50%;
              background: #fff;
              border: pr(1) solid #ccc;
              box-shadow: 0 pr(4) pr(4) #00000033;
              @include center;
              .small-point {
                width: pr(5);
                height: pr(5);
                border-radius: 50%;
                background: #333;
              }
            }
          }
        }
      }
    }
    // 主题设置
    .setting-theme {
      height: 100%;
      display: flex;
      .setting-theme-item {
        flex: 1;
        display: flex;
        flex-direction: column;
        padding: pr(5);
        box-sizing: border-box;
        .preview {
          flex: 1;
          border: pr(1) solid #ccc;
          box-sizing: border-box;
          &.no-border {
            border: none;
          }
        }
        .text {
          flex: 0 0 pr(20);
          font-size: pr(14);
          color: #333;
          color: #ccc;
          @include center;
          &.slected {
            color: #333;
          }
        }
      }
    }
    // 进度条设置
    .setting-progress {
      position: relative;
      width: 100%;
      height: 100%;
      .progress-wrapper {
        width: 100%;
        height: 100%;
        @include center;
        padding: 0 pr(30);
        box-sizing: border-box;
        display: flex;
        flex-wrap: wrap;
        .progress {
          width: 100%;
          -webkit-appearance: none;
          height: pr(2);
          background: -webkit-linear-gradient(#999, #999) no-repeat, #ddd;
          background-size: 100%;
          &.focus {
            outline: none;
          }
          &::-webkit-slider-thumb {
            -webkit-appearance: none;
            height: pr(20);
            width: pr(20);
            border-radius: 50%;
            background: #fff;
            box-shadow: 0 pr(4) pr(4) 0 #00000033;
            border: pr(1) solid #ddd;
          }
        }
        .text-wrapper {
          font-size: pr(15);
        }
      }
    }
  }
  .content-mask {
    position: absolute;
    top: 0;
    left: 0;
    z-index: 101;
    display: flex;
    width: 100%;
    height: 100%;
    background: #000000cc;
  }
}
</style>
