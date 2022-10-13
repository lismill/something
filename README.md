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
