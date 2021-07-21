<template>
  <div class="ebook">
    <title-bar :ifTitleAndMenuShow="ifTitleAndMenuShow"></title-bar>
    <div class="read-wrapper">
      <div id="read"></div>
      <div class="mask">
        <div class="left" @click="prevPage"></div>
        <div class="center" @click="toggleTitleAndMenu"></div>
        <div class="right" @click="nextPage"></div>
      </div>
    </div>
    <menu-bar :ifTitleAndMenuShow="ifTitleAndMenuShow"
              :fontSizeList="fontSizeList"
              :defaultFontSize="defaultFontSize"
              @setFontSize="setFontSize"
              :themeList="themeList"
              :defaultTheme="defaultTheme"
              @setTheme="setTheme"
              :bookAvailable="bookAvailable"
              @onProgressChange="onProgressChange"
              :navigation="navigation"
              @jumpTo="jumpTo"
              ref="menuBar"></menu-bar>
  </div>
</template>

<script>
/* eslint-disable keyword-spacing, comma-dangle, quotes, semi, no-trailing-spaces */
import TitleBar from '@/components/TitleBar';
import MenuBar from '@/components/MenuBar';
import Epub from "epubjs";
const DOWNLOAD_URL = "/static/零售的哲学.epub";
export default {
  components: {
    TitleBar,
    MenuBar
  },
  data() {
    return {
      ifTitleAndMenuShow: false,
      fontSizeList: [
        { fontSize: 12 },
        { fontSize: 14 },
        { fontSize: 16 },
        { fontSize: 18 },
        { fontSize: 20 },
        { fontSize: 22 },
        { fontSize: 24 }
      ],
      defaultFontSize: 16,
      themeList: [
        {
          name: 'default',
          style: { body: { 'color': '#000', 'background': '#fff' } }
        },
        {
          name: 'eye',
          style: { body: { 'color': '#000', 'background': '#ceeaba' } }
        },
        {
          name: 'night',
          style: { body: { 'color': '#fff', 'background': '#000' } }
        },
        {
          name: 'gold',
          style: { body: { 'color': '#000', 'background': '#f1ece2' } }
        },
      ],
      defaultTheme: 0,
      // 图书是否处于可用状态
      bookAvailable: false,
      navigation: {}
    }
  },
  methods: {
    // 电子书解析和渲染
    showEpub() {
      // 1.生成book对象
      this.book = new Epub(DOWNLOAD_URL);
      console.log(this.book);
      // 2.通过book对象生成Rendition对象(Book.renderTo)
      this.rendition = this.book.renderTo("read", {
        width: window.innerWidth,
        height: window.innerHeight,
      });
      // 3.通过Rendition.display渲染电子书
      this.rendition.display();

      // 获取rendition对象中的Theme对象
      this.themes = this.rendition.themes
      // 设置默认字体
      this.setFontSize(this.defaultFontSize)
      // 设置主题
      // this.themes.register(name. styes)-->将主题注册到theme对象中
      // this.themes.select(name)-->通过名称切换主题
      this.registerTheme()
      this.setTheme(this.defaultTheme)// 设置默认主题
      // 获取Locations对象
      // 生成locations对象比较消耗性能，所以默认不会生成locations对象
      // 可以通过epubjs的.ready钩子函数来实现(电子书解析完成后会回调的方法，返回一个promise对象)
      this.book.ready.then(() => {
        return this.book.locations.generate()
      }).then(result => {
        // 目录
        this.navigation = this.book.navigation
        // 进度
        this.locations = this.book.locations
        this.bookAvailable = true
      })
    },
    // 翻页功能
    prevPage() {
      // Rendition.prev
      if (this.rendition) {
        this.rendition.prev();
      }
    },
    nextPage() {
      // Rendition.next
      if (this.rendition) {
        this.rendition.next();
      }
    },
    // 点击页面中间，唤醒菜单栏与标题栏
    toggleTitleAndMenu() {
      this.ifTitleAndMenuShow = !this.ifTitleAndMenuShow
      if(!this.ifTitleAndMenuShow) {
        this.$refs.menuBar.hideSetting()
      }
    },
    // 功能1：设置字号方法
    setFontSize(fontSize) {
      this.defaultFontSize = fontSize
      if(this.themes) {
        this.themes.fontSize(fontSize + 'px')
      }
    },
    // 功能2：设置主题
    // 注册主题
    registerTheme() {
      this.themeList.forEach(theme => {
        this.themes.register(theme.name, theme.style)
      })
    },
    // 选择主题
    setTheme(index) {
      this.themes.select(this.themeList[index].name)
      this.defaultTheme = index
    },
    // 功能3：进度条功能 [progress：进度条的数值（0-100）]
    onProgressChange(progress) {
      const percentage = progress / 100
      const location = percentage > 0 ? this.locations.cfiFromPercentage(percentage) : 0
      this.rendition.display(location)
    },
    // 功能4：根据目录链接跳转到指定位置
    jumpTo(href) {
      this.rendition.display(href)
      this.hideTitleAndMenu()
    },
    hideTitleAndMenu() {
      // 隐藏标题栏和菜单栏
      this.ifTitleAndMenuShow = false
      // 隐藏菜单弹出的设置栏
      this.$refs.menuBar.hideSetting()
      // 隐藏目录
      this.$refs.menuBar.hideContent()
    }
  },
  mounted() {
    this.showEpub();
  },
};
</script>

<style lang="scss" scoped>
@import "/assets/styles/global";
.ebook {
  position: relative;
  .read-wrapper {
    .mask {
      position: absolute;
      top: 0;
      left: 0;
      z-index: 100;
      display: flex;
      width: 100%;
      height: 100%;
      .left {
        flex: 0 0 pr(100);
      }
      .center {
        flex: 1;
      }
      .right {
        flex: 0 0 pr(100);
      }
    }
  }
}
</style>
