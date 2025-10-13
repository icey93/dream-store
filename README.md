# 🎂 生日祝福页面 - Dream Store

一个精美的移动端生日祝福页面，使用 Vue 3 + Vite 构建，包含丰富的动画效果和互动体验。

## ✨ 功能特性

### 🎨 视觉效果

- **渐变背景**: 美丽的紫色渐变背景，营造梦幻氛围
- **闪烁星空**: 动态闪烁的星星背景装饰
- **浮动气球**: 四个彩色气球缓慢浮动动画
- **文字动画**: 标题文字逐字弹跳出现效果

### 🎂 互动蛋糕

- **三层生日蛋糕**: 精美的分层蛋糕设计
- **闪烁蜡烛**: 5 支蜡烛带有逼真的火焰闪烁效果
- **许愿功能**: 点击蛋糕可以"吹灭"蜡烛并许愿
- **烟花庆祝**: 吹灭蜡烛后触发绚丽的烟花动画

### 💝 祝福语展示

- **渐进显示**: 祝福语逐条从下方滑入
- **毛玻璃效果**: 祝福卡片采用半透明毛玻璃设计
- **精美文案**: 包含 4 条温馨的生日祝福语

<!-- 刮刮卡功能已下线 -->

### 📱 移动端优化

- **响应式设计**: 完美适配各种移动设备屏幕
- **触摸优化**: 针对触摸设备优化的交互体验
- **性能优化**: 流畅的动画和高效的渲染

## 🎮 使用方法

1. **页面加载**: 自动播放标题动画和背景效果
2. **许愿**: 点击生日蛋糕吹灭蜡烛
3. **查看祝福**: 蜡烛熄灭后自动显示祝福语
4. **查看祝福**: 欣赏页面动画与祝福内容
5. **重新开始**: 点击右下角的重置按钮重新体验

## 🛠️ 技术栈

- **Vue 3**: 使用 Composition API 和 `<script setup>` 语法
- **Vite**: 快速的构建工具和开发服务器
<!-- Canvas API（刮刮卡相关）已移除 -->
- **CSS3**: 丰富的动画效果和响应式设计
- **JavaScript ES6+**: 现代 JavaScript 语法

## 🎯 核心功能实现

### 动画系统

- 使用 CSS `@keyframes` 实现各种动画效果
- `transform` 和 `opacity` 属性创建流畅的过渡
- 通过 `animation-delay` 实现错时动画序列

<!-- 刮刮卡实现章节已移除 -->

### 响应式状态管理

```javascript
// 使用 Vue 3 响应式 API
const candlesBlown = ref(false);
const showWishes = ref(false);
const isRevealed = ref(false);
```

## 🎨 设计理念

- **用户体验优先**: 直观的交互设计，无需学习成本
- **情感化设计**: 通过动画和色彩营造温馨的生日氛围
- **渐进式体验**: 分步骤的互动流程，保持用户参与感
- **细节打磨**: 精心设计的微动画和过渡效果

## 🚀 项目设置

### 安装依赖

```sh
pnpm install
```

### 开发模式

```sh
pnpm dev
```

### 构建生产版本

```sh
pnpm build
```

### 预览构建结果

```sh
pnpm preview
```

## 🌟 开发环境推荐

### IDE 设置

[VS Code](https://code.visualstudio.com/) + [Vue (Official)](https://marketplace.visualstudio.com/items?itemName=Vue.volar) (请禁用 Vetur)

### 浏览器设置

- **Chromium 系浏览器** (Chrome, Edge, Brave 等):
  - [Vue.js devtools](https://chromewebstore.google.com/detail/vuejs-devtools/nhdogjmejiglipccpnnnanhbledajbpd)
  - [开启 Custom Object Formatter](http://bit.ly/object-formatters)
- **Firefox**:
  - [Vue.js devtools](https://addons.mozilla.org/en-US/firefox/addon/vue-js-devtools/)
  - [开启 Custom Object Formatter](https://fxdx.dev/firefox-devtools-custom-object-formatters/)

## 📄 配置参考

更多配置选项请参考 [Vite 配置文档](https://vite.dev/config/)

## 🎊 特别说明

这个项目专为移动端设计，建议在手机浏览器中体验最佳效果。

---

_用心制作，传递温暖的生日祝福 💖_
