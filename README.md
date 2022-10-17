# something

<details>
<summary>Vue.js 组件库模板</summary>

## 创建项目

```
npm create vite@latest vue-library-template --template vue-ts
```

## 创建组件

`src/components/l-buttton/index.vue`

```
<template>
  <button>{{ text }}</button>
</template>

<script lang="ts">
export default defineComponent({ name: "LButton" });
</script>

<script setup lang="ts">
import { defineComponent, defineProps } from "vue";
defineProps<{ text: string }>();
</script>
```

## 测试组件

`src/App.vue`

```
<template>
  <div>
    <l-button text="测试文本"></l-button>
  </div>
</template>
```

## 使用 app.use() 载入组件

`src/index.ts`

```
import LButton from "./components/l-buttton/index.vue";
import type { App } from "vue";

const components = [LButton];

export function install(app: App) {
  components.forEach((component) => app.component(component.name, component));
}
export default { install };
export { LButton };
```

`src/main.ts`

```
import LUI from './index';

createApp(App).use(LUI).mount("#app");
```

## 打包组件

`package.json`

```
{
  "main": "dist/vue-library-template.cjs.js",
  "module": "dist/vue-library-template.es.js",
}
```

`vite.config.ts`

```
import { defineConfig } from "vite";
import vue from "@vitejs/plugin-vue";

// https://vitejs.dev/config/
export default defineConfig({
  build: {
    lib: {
      entry: "src/index.ts",
      formats: ["cjs", "es"],
    },
    rollupOptions: {
      external: ["vue"],
    },
  },
  plugins: [vue()],
});
```

## 类型声明

`npm install vite-plugin-dts --save-dev`

`package.json`

```
{
  "types": "dist/index.d.ts",
}
```

`vite.config.ts`

```
import { defineConfig } from "vite";
import vue from "@vitejs/plugin-vue";
import dts from "vite-plugin-dts";

// https://vitejs.dev/config/
export default defineConfig({
  build: {
    lib: {
      entry: "src/index.ts",
      formats: ["cjs", "es"],
    },
    rollupOptions: {
      external: ["vue"],
    },
  },
  plugins: [vue(), dts({ insertTypesEntry: true, copyDtsFiles: false })],
});
```

## 打包发布

```
npm publish
```
</details>
  
<details>
<summary>Vue.js 移动端调用摄像头获取数据流</summary>

## 参考资料
getUserMedia方法属于WebRTC API的部分知识体系，可能需要你了解相关知识：
1. WebRTC API
2. WebRTC adapter
3. MediaDevices.getUserMedia
4. h5-scan-qrcode

## 1. 说明

本组件主要作用是通过 getUserMedia 调用浏览器摄像头。解析拍摄到的二维码，以事件的方式返回给父组件解析结果。

同理各位同学可以配合 jsqr 以外的其它解析库，调用摄像头获取视频流解析需要出结果。
## 2. 原理
### 2.1 调用摄像头获取数据流

通过 navigator.mediaDevices.getUserMedia 方法调用摄像头，我们所需要的是返回内容的 MediaStream，MediaStream 包含视频轨道、音频轨道也可能是其它轨道类型。常用使用方式：
```
navigator.mediaDevices.getUserMedia(constraints)
.then(function(stream) {
  // 使用这个stream
})
.catch(function(err) {
  // 处理error
})
```

### 2.2 将数据流绘制到canvas中
说到canvas大家并不陌生，在这里通过 CanvasRenderingContext2D.drawImage 方法将带有二维码的数据流绘制到cavas中，drawImage相关参数不再详细阐述可以点击资料链接。

### 2.3 获取canvas数据 
通过 CanvasRenderingContext2D.getImageData 方法获取ImageData格式的对象数据，此方法在在实际中经常用到，getImageData相关参数不再详细阐述可以点击资料链接。

### 2.4 解析二维码
获取到的二维码的ImageData对象再通过对应的解析库进行解析就能得到解析结果，本组件通过使用jsQR库进行解析。常用使用方式：
const code = jsQR(imageData, width, height, options?);
code && console.log("Found QR code", code);

## 3. 注意事项
浏览器安全策略：现今手机浏览器内核大多WebKit内核、Chrome内核，这俩种浏览器在http浏览环境下会限制MediaDevices.getUserMedia方法，需要在https协议下才能调用改方法。
规避方法：
PC端可以通过配置浏览器实现方案链接
移动端的话可以将项目打包放在本地的nginx中，再通过nginx将http转为https(这是一个可行方案，可惜本人一直配置失败，配置成功的同学可以分享一下哦~)；或者直接打包放在一个配有hppts协议证书的服务器中。

## 4. 示例
```
<template>
    <div class="scan">
        <div class="nav">
            <van-icon name="arrow-left" @click="() => $router.go(-1)" />
            <p class="title">VScanQrcode示例</p>
        </div>
        <div class="scroll-container">
            <VScanQrcode
                :stop-on-scanned="true"
                :draw-on-found="true"
                :responsive="true"
                @code-scanned="codeScanned"
                @error-captured="capturedError"
            />
        </div>
    </div>
</template>
<script lang="ts">
import { Vue, Component } from "vue-property-decorator";
import { Icon, Toast } from "vant";
import VScanQrcode from "@/resources/components/v-scan-qrcode/index.vue";
@Component({
    name: "Scan",
    components: {
        VanIcon: Icon,
        VScanQrcode
    },
    mounted() {
        /* 判断是否能正常调用摄像头 */
        let str = navigator.userAgent.toLowerCase();
        let ver: any = str.match(/cpu iphone os (.*?) like mac os/);
        if (ver && ver[1].replace(/_/g, ".") < "10.3.3") {
            Toast.fail("相机调用失败，设备不支持");
        }
    }
})
export default class Scan extends Vue {
    errorMessage = "";
    scanned = "";
    /**
     * 扫码成功回调事件
     */
    codeScanned(code: string): void {
        this.scanned = code;
        setTimeout(() => {
            Toast.success(`扫码解析成功: ${code}`);
        }, 200);
    }
    /**
     * 无法调用摄像头的error返回事件
     */
    capturedError(error: any): void {
        switch (error.name) {
            case "NotAllowedError":
                this.errorMessage = "相机没有权限。";
                break;
            case "NotFoundError":
                this.errorMessage = "没有连接摄像头。";
                break;
            case "NotSupportedError":
                this.errorMessage = "看起来这个页面是在不安全的上下文中提供的。";
                break;
            case "NotReadableError":
                this.errorMessage = "无法访问你的摄像头。它已经在使用了吗？";
                break;
            case "OverconstrainedError":
                this.errorMessage = "约束条件不匹配任何安装的相机。";
                break;
            default:
                this.errorMessage = "UNKNOWN ERROR: " + error.message;
        }
        console.error(this.errorMessage);
        Toast.fail(`相机调用失败，${this.errorMessage}`);
    }
}
</script>
组件 `@/resources/components/v-scan-qrcode/index.vue`
<template>
  <div ref="scaner" class="v-scan-qrcode">
    <!-- 提示框：用于在不兼容的浏览器中显示提示语 -->
    <div
      v-if="showBanner"
      class="banner"
      :style="{ background: notifyBGColor }"
    >
      <p class="text">若当前浏览器无法扫码，请切换其他浏览器尝试</p>
      <van-icon name="cross" @click="() => (showBanner = false)" />
    </div>
    <!-- 扫码框：显示扫码动画 -->
    <div class="cover">
      <p class="line"></p>
      <span class="square top left"></span>
      <span class="square top right"></span>
      <span class="square bottom right"></span>
      <span class="square bottom left"></span>
      <p class="tips">将二维码放入框内，即可自动扫描</p>
    </div>
    <!-- 展示摄像头捕获视频流 -->
    <video
      v-show="showPlay"
      ref="video"
      class="source"
      :width="videoWH.width"
      :height="videoWH.height"
      controls
    ></video>
    <!-- 绘制视频帧，用于二维码识别 -->
    <canvas v-show="!showPlay" ref="canvas" />
    <button v-show="showPlay" @click="run">开始</button>
  </div>
</template>

<script lang="ts">
/* eslint-disable */
import { Vue, Component, Prop, Watch, Emit } from "vue-property-decorator";
import jsQR from "jsqr";
import { Icon } from "vant";
@Component({
  name: "VScanQrcode",
  components: { VanIcon: Icon },
  mounted(): void {
    (this as any).setup();
  },
  beforeDestroy(): void {
    (this as any).fullStop();
  }
})
export default class VScanQrcode extends Vue {
  showPlay = false /* 是否-视频播放形式进行周期性扫描 */;
  showBanner = true /* 通知栏显示状态 */;
  containerWidth = undefined /* 此组件的宽高集合 */;
  active = false /* 是否扫码成功 */;

  /* 通知栏背景颜色 */
  @Prop({ default: "#1989fa" })
  readonly notifyBGColor?: string;

  /* 使用后置相机 */
  @Prop({ default: true })
  readonly useBackCamera?: boolean;

  /* 扫描识别后停止 */
  @Prop({ default: true })
  readonly stopOnScaned?: boolean;

  @Prop({ default: true })
  readonly drawOnfound?: boolean;

  /* 线条颜色 */
  @Prop({ default: "#03C03C" })
  readonly lineColor?: string;

  /* 线条宽度 */
  @Prop({ default: 2 })
  readonly lineWidth?: number;

  /* 视频宽度 */
  @Prop({
    default: document.documentElement.clientWidth || document.body.clientWidth
  })
  readonly videoWidth?: number;

  /* 视频高度 */
  @Prop({
    default:
      document.documentElement.clientHeight - 48 ||
      document.body.clientHeight - 48
  })
  readonly videoHeight?: number;

  /* 获取整个盒子宽高 */
  @Prop({ default: false })
  readonly responsive?: boolean;

  /* 捕获无法唤起摄像头异常 */
  @Emit()
  errorCaptured(error: Error): Error {
    return error;
  }

  /* 返回二维码解析结果 */
  @Emit()
  codeScanned(code: string): string {
    alert(`1扫码解析成功: ${code}`);
    return code;
  }

  @Watch("active", { immediate: true })
  onWatchActive(newVal: boolean): void {
    if (!newVal) {
      this.fullStop();
    }
  }

  get videoWH(): { width: number | undefined; height: number | undefined } {
    if (this.containerWidth) {
      const width = this.containerWidth;
      const height = Number(width) * 0.75;
      return { width, height };
    }
    return { width: this.videoWidth, height: this.videoHeight };
  }

  /**
   * 二维码-动态线
   */
  drawLine(begin: any, end: any): void {
    (this as any).canvas.beginPath();
    (this as any).canvas.moveTo(begin.x, begin.y);
    (this as any).canvas.lineTo(end.x, end.y);
    (this as any).canvas.lineWidth = this.lineWidth;
    (this as any).canvas.strokeStyle = this.lineColor;
    (this as any).canvas.stroke();
  }

  /**
   * 二维码-扫描框
   */
  drawBox(location: any): void {
    if (this.drawOnfound) {
      this.drawLine(location.topLeftCorner, location.topRightCorner);
      this.drawLine(location.topRightCorner, location.bottomRightCorner);
      this.drawLine(location.bottomRightCorner, location.bottomLeftCorner);
      this.drawLine(location.bottomLeftCorner, location.topLeftCorner);
    }
  }
  /**
   * 周期性扫码识别
   */
  tick(): void {
    // 视频处于准备阶段，并且已经加载足够的数据
    if (
      this.$refs.video &&
      (this.$refs.video as any).readyState ===
        (this.$refs.video as any).HAVE_ENOUGH_DATA
    ) {
      // 开始在画布上绘制视频
      (this.$refs.canvas as any).height = this.videoWH.height;
      (this.$refs.canvas as any).width = this.videoWH.width;
      (this as any).canvas.drawImage(
        this.$refs.video,
        0,
        0,
        (this.$refs.canvas as any).width,
        (this.$refs.canvas as any).height
      );
      // getImageData() 复制画布上制定矩形的像素数据
      const imageData = (this as any).canvas.getImageData(
        0,
        0,
        (this.$refs.canvas as any).width,
        (this.$refs.canvas as any).height
      );
      let code = false;
      try {
        // 识别二维码
        code = jsQR(imageData.data, imageData.width, imageData.height) as any;
      } catch (e) {
        alert(`二维码识别失败=>${JSON.stringify(e)}`);
        console.error("识别二维码失败===>", e);
      }
      // 如果识别出二维码，绘制矩形框
      if (code) {
        this.drawBox((code as any).location);
        // 识别成功事件处理
        this.found((code as any).data);
      }
    }
    this.run();
  }

  /**
   * 初始化
   */
  setup(): any {
    if (this.responsive) {
      this.$nextTick(() => {
        this.containerWidth = (this.$refs.scaner as any).clientWidth;
      });
    }

    // 判断了浏览器是否支持挂载在MediaDevices.getUserMedia()的方法
    if (navigator.mediaDevices && navigator.mediaDevices.getUserMedia) {
      (this as any).previousCode = null;
      (this as any).parity = 0;
      this.active = true;
      (this as any).canvas = (this.$refs.canvas as any).getContext("2d");
      // 获取摄像头模式，默认设置是后置摄像头
      const facingMode = this.useBackCamera ? { exact: "environment" } : "user";
      // 摄像头视频处理
      const handleSuccess: any = (stream: any) => {
        if ((this.$refs.video as any).srcObject !== undefined) {
          (this.$refs.video as any).srcObject = stream;
        } else if ((window as any).videoEl.mozSrcObject !== undefined) {
          (this.$refs.video as any).mozSrcObject = stream;
        } else if (window.URL.createObjectURL) {
          (this.$refs.video as any).src = window.URL.createObjectURL(stream);
        } else if (window.webkitURL) {
          (this.$refs.video as any).src = window.webkitURL.createObjectURL(
            stream
          );
        } else {
          (this.$refs.video as any).src = stream;
        }
        // 不希望用户来拖动进度条的话，可以直接使用playsinline属性，webkit-playsinline属性
        (this.$refs.video as any).playsInline = true;
        const playPromise = (this.$refs.video as any).play();
        playPromise.catch(() => (this.showPlay = true));
        // 视频开始播放时进行周期性扫码识别
        playPromise.then(this.run);
      };

      // 捕获视频流
      navigator.mediaDevices
        .getUserMedia({ video: { facingMode } })
        .then(handleSuccess)
        .catch(() => {
          navigator.mediaDevices
            .getUserMedia({ video: true })
            .then(handleSuccess)
            .catch(error => {
              alert(`捕获视频流失败===>${JSON.stringify(error)}`);
              console.error("捕获视频流失败===>", error);
              this.errorCaptured(error);
            });
        });
    }
  }

  /**
   * 周期性扫描
   */
  run(): void {
    if (this.active) {
      // 浏览器在下次重绘前循环调用扫码方法
      requestAnimationFrame(this.tick);
    }
  }

  /**
   * 二维码识别成功事件处理
   */
  found(code: string): void {
    if ((this as any).previousCode !== code) {
      (this as any).previousCode = code;
    } else if ((this as any).previousCode === code) {
      (this as any).parity += 1;
    }
    if ((this as any).parity > 2) {
      this.active = (this as any).stopOnScanned ? false : true;
      (this as any).parity = 0;
      this.codeScanned(code);
    }
  }

  /**
   * 完全停止
   */
  fullStop(): void {
    if (this.$refs.video && (this.$refs.video as any).srcObject) {

      // 停止视频流序列轨道
      (this.$refs.video as any).srcObject
        .getTracks()
        .forEach((t: any) => t.stop());
    }
  }
}
</script>

<style lang="scss" scoped>
.v-scan-qrcode {
  background: #000000;
  position: fixed;
  top: 48px;
  left: 0;
  width: 100%;
  height: 100%;
  height: -webkit-calc(100% - 48px);
  height: -moz-calc(100% - 48px);
  height: -ms-calc(100% - 48px);
  height: -o-calc(100% - 48px);
  height: calc(100% - 48px);
  .banner {
    width: 90%;
    height: 40px;
    position: absolute;
    top: 32px;
    left: 50%;
    transform: translateX(-50%);
    border-radius: 16px;
    box-sizing: border-box;
    display: flex;
    justify-content: space-between;
    align-items: center;
    opacity: 0.9;
    box-shadow: 1px 1px 10px rgba(0, 0, 0, 0.2);
    padding: 0 12px;
    .text {
      padding: 0;
      margin: 0;
      color: #ffffff;
      font-size: 12px;
      height: 16px;
    }
    .van-icon-cross {
      height: 16px;
      width: 16px;
      border-radius: 50%;
      background: rgba(255, 255, 255, 0.5);
      &::before {
        width: 100%;
        height: 100%;
        display: flex;
        justify-content: center;
        align-items: center;
        font-size: 12px;
        text-align: center;
        color: #fff;
      }
    }
  }
  .cover {
    height: 222px;
    width: 222px;
    position: absolute;
    top: 50%;
    left: 50%;
    -webkit-transform: translate(-50%, -50%);
    -moz-transform: translate(-50%, -50%);
    -ms-transform: translate(-50%, -50%);
    -o-transform: translate(-50%, -50%);
    transform: translate(-50%, -50%);
    border: 0.5px solid #999999;
    z-index: 1111;
    .line {
      width: 200px;
      height: 1px;
      margin-left: 10px;
      background: #5f68e8;
      background: linear-gradient(
        to right,
        transparent,
        #5f68e8,
        #0165ff,
        #5f68e8,
        transparent
      );
      position: absolute;
      -webkit-animation: scan 1.75s infinite linear;
      -moz-animation: scan 1.75s infinite linear;
      -ms-animation: scan 1.75s infinite linear;
      -o-animation: scan 1.75s infinite linear;
      animation: scan 1.75s infinite linear;
      -webkit-animation-fill-mode: both;
      -moz-animation-fill-mode: both;
      -ms-animation-fill-mode: both;
      -o-animation-fill-mode: both;
      animation-fill-mode: both;
      border-radius: 2px;
    }
    .square {
      display: inline-block;
      height: 20px;
      width: 20px;
      position: absolute;
      &.top {
        top: 0;
        border-top: 1px solid #5f68e8;
      }
      &.left {
        left: 0;
        border-left: 1px solid #5f68e8;
      }
      &.bottom {
        bottom: 0;
        border-bottom: 1px solid #5f68e8;
      }
      &.right {
        right: 0;
        border-right: 1px solid #5f68e8;
      }
    }
    .tips {
      text-align: center;
      position: absolute;
      bottom: -48px;
      width: 100%;
      font-size: 12px;
      color: #ffffff;
      opacity: 0.8;
    }
  }
}
@-webkit-keyframes scan {
  0% {
    top: 0;
  }
  25% {
    top: 50px;
  }
  50% {
    top: 100px;
  }
  75% {
    top: 150px;
  }
  100% {
    top: 200px;
  }
}
@-moz-keyframes scan {
  0% {
    top: 0;
  }
  25% {
    top: 50px;
  }
  50% {
    top: 100px;
  }
  75% {
    top: 150px;
  }
  100% {
    top: 200px;
  }
}
@-o-keyframes scan {
  0% {
    top: 0;
  }
  25% {
    top: 50px;
  }
  50% {
    top: 100px;
  }
  75% {
    top: 150px;
  }
  100% {
    top: 200px;
  }
}
@keyframes scan {
  0% {
    top: 0;
  }
  25% {
    top: 50px;
  }
  50% {
    top: 100px;
  }
  75% {
    top: 150px;
  }
  100% {
    top: 200px;
  }
}
</style>
```
  
### 4.1 参数
|--参数--|--说明--|--类型--|--默认值--|
|----|----|----|----|
|notifyBGColor	|通知栏背景色	|string	|	#1989fa|
|use-back-camera|	使用后置摄像头|	boolean|		true|
|stop-on-scaned|	扫描识别后停止|	boolean|		true|
|draw-onfound|	是否开启扫描线|	boolean|		true|
|line-color|	线条颜色|	string|		#03C03C|
|line-width|	线条宽度|	number|		2|
|video-width|	视频尺寸宽度|	numner|		document.documentElement.clientWidth || document.body.clientWidth|
|video-height|	视频尺寸高度|	number|		document.documentElement.clientHeight || document.body.clientHeight|
|responsive|	宽度动态自适应|	boolean|		true|
  
### 4.2 事件
|事件|说明|返回值|
|----|----|----|
|error-captured	|唤起摄像头异常	|event:Error|
|code-scanned|	二维码解析结果|	code:string|

## 5. Enjoy
😃
</details>
