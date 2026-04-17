# 在 Vue 中，<template> 里面的标签写内联样式有以下几种方式：

## 1. 使用 :style 绑定（推荐）
```js
<template>
  <div :style="{ color: 'red', fontSize: '16px' }">
    红色文字
  </div>
</template>
```js

## 2. 使用对象变量绑定
```js
<template>
  <div :style="styleObject">
    动态样式
  </div>
</template>

<script setup>
import { ref } from 'vue'

const styleObject = ref({
  color: 'blue',
  fontSize: '14px',
  backgroundColor: '#f0f0f0'
})
</script>
```

## 3. 使用数组绑定多个样式对象
```js
<template>
  <div :style="[baseStyles, activeStyles]">
    多重样式
  </div>
</template>

<script setup>
const baseStyles = {
  color: 'black',
  fontSize: '16px'
}

const activeStyles = {
  fontWeight: 'bold',
  backgroundColor: 'yellow'
}
</script>
```

## 4. 使用计算属性
```js
<template>
  <div :style="dynamicStyle">
    计算样式
  </div>
</template>

<script setup>
import { computed } from 'vue'

const isActive = ref(true)

const dynamicStyle = computed(() => ({
  color: isActive.value ? 'red' : 'gray',
  fontWeight: isActive.value ? 'bold' : 'normal'
}))
</script>
```

## 5. 支持 CSS 属性名的两种写法
```js
<template>
  <!-- 驼峰命名法 -->
  <div :style="{ fontSize: '16px', backgroundColor: 'red' }">
    驼峰写法
  </div>
  
  <!-- 短横线命名法（需要加引号） -->
  <div :style="{ 'font-size': '16px', 'background-color': 'red' }">
    短横线写法
  </div>
</template>
```

## 注意事项
⚠️ **不要直接在标签上使用 style 属性（这样是静态的，无法响应式更新）：**
```js
<!-- ❌ 不推荐 - 静态样式，无法动态更新 -->
<div style="color: red; font-size: 16px;">静态样式</div>

<!-- ✅ 推荐 - 动态样式，可以响应式更新 -->
<div :style="{ color: textColor, fontSize: textSize + 'px' }">动态样式</div>
```

## 实际示例
```js
<template>
  <div 
    :style="{
      padding: '20px',
      margin: '10px',
      backgroundColor: bgColor,
      borderRadius: '8px',
      boxShadow: '0 2px 8px rgba(0,0,0,0.1)'
    }"
  >
    <h2 :style="{ color: titleColor, marginBottom: '10px' }">
      {{ title }}
    </h2>
    <p :style="paragraphStyle">
      {{ content }}
    </p>
  </div>
</template>

<script setup>
import { ref, computed } from 'vue'

const title = ref('标题')
const content = ref('内容')
const bgColor = ref('#ffffff')
const titleColor = ref('#333333')

const paragraphStyle = computed(() => ({
  color: '#666666',
  lineHeight: '1.6',
  fontSize: '14px'
}))
</script>
```

**最佳实践： 优先使用 :style 绑定，这样可以享受 Vue 的响应式特性，样式变化时会自动更新视图。**
