---
title: Tuesday
date: 2021-01-19 14:04:42
categories: 日常
tags: ['go', 'css', 'customer']
---

[[toc]]

# 在 Markdown 中 使用 Vue

  ## 浏览器的 API 访问限制

  当你在开发一个 VuePress 应用时，由于所有的页面在生成静态 HTML 时都需要通过 Node.js 服务端渲染，因此所有的 Vue 相关代码都应当遵循 [编写通用代码 (opens new window)](https://ssr.vuejs.org/zh/universal.html)的要求。简而言之，请确保只在 `beforeMount` 或者 `mounted` 访问浏览器 / DOM 的 API。

  如果你正在使用，或者需要展示一个对于 SSR 不怎么友好的组件（比如包含了自定义指令），你可以将它们包裹在内置的 `<ClientOnly>` 组件中：

  ```md
  <ClientOnly>
    <NonSSRFriendlyComponent/>
  </ClientOnly>
  ```

  请注意，这并不能解决一些组件或库在**导入**时就试图访问浏览器 API 的问题 —— 如果需要使用这样的组件或库，你需要在合适的生命周期钩子中**动态导入**它们：

  ```vue
  <script>
  export default {
    mounted () {
      import('./lib-that-access-window-on-import').then(module => {
        // use code
      })
    }
  }
  </script>
  ```

  如果你的模块通过 `export default` 导出一个 Vue 组件，那么你可以动态注册它：

  ```vue
  <template>
    <component v-if="dynamicComponent" :is="dynamicComponent"></component>
  </template>
  <script>
  export default {
    data() {
      return {
        dynamicComponent: null
      }
    },
    mounted () {
      import('./lib-that-access-window-on-import').then(module => {
        this.dynamicComponent = module.default
      })
    }
  }
  </script>
  ```

  **参考:**

  - [Vue.js > 动态组件(opens new window)](https://cn.vuejs.org/v2/guide/components.html#动态组件)

  ## 模板语法

  ### 插值

  每一个 Markdown 文件将首先被编译成 HTML，接着作为一个 Vue 组件传入 `vue-loader`，这意味着你可以在文本中使用 Vue 风格的插值：

  **Input**

  ```md
  {{ 1 + 1 }}
  ```

  **Output**

  ```
  2
  ```

  ### 指令

  同样地，也可以使用指令:

  **Input**

  ```md
  <span v-for="i in 3">{{ i }} </span>
  ```

  **Output**

  ```
  1 2 3 
  ```

  ### 访问网站以及页面的数据

  编译后的组件没有私有数据，但可以访问 [网站的元数据](https://www.vuepress.cn/theme/writing-a-theme.html#网站和页面的元数据)，举例来说：

  **Input**

  ```md
  {{ $page }}
  ```

  **Output**

  ```json
  {
    "path": "/using-vue.html",
    "title": "Using Vue in Markdown",
    "frontmatter": {}
  }
  ```

  ## Escaping

  默认情况下，块级 (block) 的代码块将会被自动包裹在 `v-pre` 中。如果你想要在内联 (inline) 的代码块或者普通文本中显示原始的大括号，或者一些 Vue 特定的语法，你需要使用自定义容器 `v-pre` 来包裹：

  **Input**

  ```md
  ::: v-pre
  `{{ This will be displayed as-is }}`
  :::
  ```

  **Output**

  ```
  {{ This will be displayed as-is }}
  ```

  ## 使用组件

  所有在 `.vuepress/components` 中找到的 `*.vue` 文件将会自动地被注册为全局的异步组件，如：

  ```text
  .
  └─ .vuepress
     └─ components
        ├─ demo-1.vue
        ├─ OtherComponent.vue
        └─ Foo
           └─ Bar.vue
  ```