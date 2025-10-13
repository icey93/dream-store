<template>
  <div class="birthday-app">
    <!-- 背景装饰 -->
    <div class="background-decorations">
      <div class="stars"></div>
      <div class="floating-balloons">
        <div class="balloon balloon-1"></div>
        <div class="balloon balloon-2"></div>
        <div class="balloon balloon-3"></div>
        <div class="balloon balloon-4"></div>
      </div>
    </div>

    <!-- 主要内容区域 -->
    <div class="main-content">
      <!-- 头像区域 -->
      <div class="avatar-section">
        <div class="avatar-container">
          <div class="avatar-frame">
            <img
              src="./assets/avatar.jpg"
              alt="生日快乐"
              class="avatar-image"
              @error="handleImageError"
            />
            <div class="avatar-glow"></div>
          </div>
          <div class="avatar-decoration">
            <div class="heart heart-1">💖</div>
            <div class="heart heart-2">💝</div>
            <div class="heart heart-3">💕</div>
            <div class="star star-1">⭐</div>
            <div class="star star-2">✨</div>
            <div class="star star-3">🌟</div>
          </div>
        </div>
      </div>

      <!-- 标题区域 -->
      <div class="title-section">
        <h1 class="birthday-title">
          <span
            class="title-char"
            v-for="(char, index) in titleChars"
            :key="index"
            :style="{ animationDelay: index * 0.1 + 's' }"
          >
            {{ char }}
          </span>
        </h1>
        <div class="subtitle">2025年10月13日</div>
      </div>

      <!-- 生日蛋糕 -->
      <div class="cake-container" @click="blowCandles">
        <div class="cake">
          <div class="cake-layer cake-bottom"></div>
          <div class="cake-layer cake-middle"></div>
          <div class="cake-layer cake-top"></div>
          <div class="candles">
            <div
              class="candle"
              v-for="n in 5"
              :key="n"
              :class="{ blown: candlesBlown }"
            >
              <div class="flame" :class="{ out: candlesBlown }"></div>
            </div>
          </div>
        </div>
        <div class="cake-hint" v-if="!candlesBlown">点击蛋糕许愿 🎂</div>
      </div>

      <!-- 祝福语 -->
      <!-- <div class="wishes-section" v-if="candlesBlown">
        <div
          class="wish-text"
          v-for="(wish, index) in wishes"
          :key="index"
          :style="{ animationDelay: index * 0.5 + 's' }"
          :class="{ show: showWishes }"
        >
          {{ wish }}
        </div>
      </div> -->

      <!-- 简易刮刮卡：点蛋糕后出现 -->
      <div class="scratch-card" v-if="candlesBlown">
        <div class="scratch-card-title">🎁 刮开查看祝福</div>
        <div class="scratch-card-container" ref="scratchContainer">
          <div class="scratch-under">
            <div class="scratch-text">
              <div>📷</div>
              <div>恭喜您获得了任意型号相机一台</div>
              <div>请凭截图兑换~</div>
            </div>
          </div>
          <canvas
            class="scratch-overlay"
            ref="scratchCanvas"
            @pointerdown="onPointerDown"
            @pointermove="onPointerMove"
            @pointerup="onPointerUp"
            @pointercancel="onPointerUp"
            @pointerleave="onPointerUp"
          ></canvas>
        </div>
        <div class="scratch-info">
          已刮开 {{ Math.round(scratchedPercent) }}%
          <button class="scratch-reset" @click="resetScratch">重置</button>
        </div>
      </div>

      <!-- 烟花效果 -->
      <div class="fireworks" v-if="showFireworks">
        <div
          class="firework"
          v-for="n in 6"
          :key="n"
          :style="getFireworkStyle(n)"
        ></div>
      </div>
    </div>

    <!-- 重置按钮 -->
    <button class="reset-btn" @click="resetAll" v-if="candlesBlown">
      🔄 重新开始
    </button>
  </div>
</template>

<script setup>
import { ref, computed, onMounted, onBeforeUnmount, nextTick } from "vue";

// 响应式数据
const candlesBlown = ref(false);
const showWishes = ref(false);
const showFireworks = ref(false);

// 标题文字
const titleText = "张总生日快乐~";
const titleChars = computed(() => titleText.split(""));

// 祝福语
const wishes = [
  "🎈 愿你永远年轻，永远热泪盈眶",
  "🌟 愿你的生活如诗如画，充满惊喜",
  "🎁 愿所有的美好都在这一刻为你绽放",
  "💖 愿你被这个世界温柔以待",
];

// 刮刮卡：可配置文案
const scratchText = ref("恭喜：您获得了");

// 刮刮卡 refs 与状态
const scratchCanvas = ref(null);
const scratchContainer = ref(null);
const isScratching = ref(false);
const lastPos = ref(null);
const scratchedPercent = ref(0);
const fullyRevealed = ref(false);

// 初始化/重绘覆盖层
const drawCover = () => {
  const canvas = scratchCanvas.value;
  const container = scratchContainer.value;
  if (!canvas || !container) return;
  const rect = container.getBoundingClientRect();
  canvas.width = Math.round(rect.width);
  canvas.height = Math.round(rect.height);
  const ctx = canvas.getContext("2d");
  // 简单灰色涂层
  ctx.globalCompositeOperation = "source-over";
  ctx.fillStyle = "#bdbdbd";
  ctx.fillRect(0, 0, canvas.width, canvas.height);
};

const resizeOverlay = () => {
  // 仅当未完全揭示时重绘覆盖层
  const keepReveal = fullyRevealed.value;
  drawCover();
  if (keepReveal) {
    // 如果已完全揭示，清空覆盖层
    const canvas = scratchCanvas.value;
    if (canvas) {
      const ctx = canvas.getContext("2d");
      ctx.clearRect(0, 0, canvas.width, canvas.height);
    }
  }
};

const getPos = (e) => {
  const canvas = scratchCanvas.value;
  const rect = canvas.getBoundingClientRect();
  const x = (e.touches ? e.touches[0].clientX : e.clientX) - rect.left;
  const y = (e.touches ? e.touches[0].clientY : e.clientY) - rect.top;
  return { x, y };
};

const eraseAt = (pos) => {
  const canvas = scratchCanvas.value;
  if (!canvas) return;
  const ctx = canvas.getContext("2d");
  ctx.globalCompositeOperation = "destination-out";
  // 简单橡皮擦：线条+圆点
  if (lastPos.value) {
    ctx.lineWidth = 24;
    ctx.lineCap = "round";
    ctx.lineJoin = "round";
    ctx.beginPath();
    ctx.moveTo(lastPos.value.x, lastPos.value.y);
    ctx.lineTo(pos.x, pos.y);
    ctx.stroke();
  }
  ctx.beginPath();
  ctx.arc(pos.x, pos.y, 12, 0, Math.PI * 2);
  ctx.fill();
  lastPos.value = pos;
};

const computeScratched = () => {
  const canvas = scratchCanvas.value;
  if (!canvas) return 0;
  const ctx = canvas.getContext("2d");
  const img = ctx.getImageData(0, 0, canvas.width, canvas.height);
  let transparent = 0;
  // 统计 alpha < 128 的像素
  for (let i = 3; i < img.data.length; i += 4) {
    if (img.data[i] < 128) transparent++;
  }
  const total = img.data.length / 4;
  return (transparent / total) * 100;
};

const onPointerDown = (e) => {
  if (fullyRevealed.value) return;
  isScratching.value = true;
  e.target.setPointerCapture?.(e.pointerId);
  eraseAt(getPos(e));
  scratchedPercent.value = computeScratched();
  if (scratchedPercent.value >= 50) revealAll();
};

const onPointerMove = (e) => {
  if (!isScratching.value || fullyRevealed.value) return;
  eraseAt(getPos(e));
  // 取样计算，避免每像素都算
  if (Math.random() < 0.25) {
    scratchedPercent.value = computeScratched();
    if (scratchedPercent.value >= 50) revealAll();
  }
};

const onPointerUp = (e) => {
  if (!isScratching.value) return;
  isScratching.value = false;
  lastPos.value = null;
  scratchedPercent.value = computeScratched();
  if (scratchedPercent.value >= 50) revealAll();
};

const revealAll = () => {
  fullyRevealed.value = true;
  const canvas = scratchCanvas.value;
  if (!canvas) return;
  const ctx = canvas.getContext("2d");
  ctx.clearRect(0, 0, canvas.width, canvas.height);
  scratchedPercent.value = 100;
};

const resetScratch = async () => {
  fullyRevealed.value = false;
  scratchedPercent.value = 0;
  lastPos.value = null;
  await nextTick();
  drawCover();
};

onMounted(() => {
  nextTick(() => {
    resizeOverlay();
    window.addEventListener("resize", resizeOverlay, { passive: true });
  });
});

onBeforeUnmount(() => {
  window.removeEventListener("resize", resizeOverlay);
});

// 吹蜡烛
const blowCandles = () => {
  if (candlesBlown.value) return;

  candlesBlown.value = true;

  // 初始化刮刮卡覆盖层，避免出现底部文案短暂露出
  fullyRevealed.value = false;
  scratchedPercent.value = 0;
  lastPos.value = null;
  nextTick(() => {
    drawCover();
  });

  // 延迟显示祝福语
  setTimeout(() => {
    showWishes.value = true;
  }, 500);

  // 显示烟花
  setTimeout(() => {
    showFireworks.value = true;
    setTimeout(() => {
      showFireworks.value = false;
    }, 3000);
  }, 1000);
};

// 烟花样式
const getFireworkStyle = (index) => {
  const positions = [
    { left: "10%", top: "20%" },
    { left: "80%", top: "15%" },
    { left: "20%", top: "60%" },
    { left: "70%", top: "50%" },
    { left: "50%", top: "30%" },
    { left: "90%", top: "70%" },
  ];
  return {
    ...positions[index - 1],
    animationDelay: index * 0.3 + "s",
  };
};

// 重置所有状态
const resetAll = () => {
  candlesBlown.value = false;
  showWishes.value = false;
  showFireworks.value = false;
  // 同时重置刮刮卡
  resetScratch();
};

// 头像图片加载错误处理
const handleImageError = (event) => {
  console.warn("头像图片加载失败，使用默认占位符");
  event.target.style.display = "none";
  // 可以在这里添加默认头像或占位符
};
</script>

<style>
/* 全局CSS重置 - 消除浏览器默认样式 */
* {
  margin: 0;
  padding: 0;
  box-sizing: border-box;
}

html,
body {
  margin: 0;
  padding: 0;
  width: 100%;
  height: 100%;
}

#app {
  margin: 0;
  padding: 0;
}
</style>

<style scoped>
/* 组件样式 */
.birthday-app {
  min-height: 100vh;
  background: linear-gradient(135deg, #667eea 0%, #764ba2 100%);
  position: relative;
  overflow-x: hidden;
  padding: 20px;
  font-family: "Arial", sans-serif;
}

/* 简易刮刮卡样式 */
.scratch-card {
  margin: 30px auto 0;
}

.scratch-card-title {
  color: #fff;
  font-size: 1.1rem;
  margin-bottom: 10px;
}

.scratch-card-container {
  position: relative;
  width: 320px;
  height: 160px;
  margin: 0 auto;
  border-radius: 14px;
  overflow: hidden;
  box-shadow: 0 6px 16px rgba(0, 0, 0, 0.2);
  background: #fff;
}

.scratch-under {
  position: absolute;
  inset: 0;
  display: grid;
  place-items: center;
  padding: 16px;
  z-index: 1;
}

.scratch-text {
  color: #3a0ca3;
  text-align: center;
  font-weight: 700;
  display: flex;
  flex-direction: column;
  align-items: center;
  justify-content: center;
  gap: 8px;
  line-height: 1.25;
  text-shadow: 0 1px 0 rgba(255, 255, 255, 0.5);
}

.scratch-text > div:first-child {
  font-size: 2rem;
}

.scratch-text > div:nth-child(2) {
  font-size: 1.15rem;
}

.scratch-text > div:nth-child(3) {
  font-size: 0.95rem;
  opacity: 0.85;
}

.scratch-overlay {
  position: absolute;
  inset: 0;
  width: 100%;
  height: 100%;
  touch-action: none;
  cursor: crosshair;
  z-index: 2;
}

.scratch-info {
  margin-top: 10px;
  color: #fff;
}

.scratch-reset {
  margin-left: 12px;
  background: rgba(255, 255, 255, 0.2);
  color: #fff;
  border: 1px solid rgba(255, 255, 255, 0.4);
  border-radius: 10px;
  padding: 6px 10px;
  cursor: pointer;
}

@media (max-width: 480px) {
  .scratch-card-container {
    width: 280px;
    height: 150px;
  }

  .scratch-text > div:first-child {
    font-size: 1.7rem;
  }

  .scratch-text > div:nth-child(2) {
    font-size: 1.05rem;
  }

  .scratch-text > div:nth-child(3) {
    font-size: 0.9rem;
  }
}

/* 背景装饰 */
.background-decorations {
  position: fixed;
  top: 0;
  left: 0;
  width: 100%;
  height: 100%;
  pointer-events: none;
  z-index: 1;
}

.stars {
  position: absolute;
  width: 100%;
  height: 100%;
  background-image: radial-gradient(2px 2px at 20px 30px, #fff, transparent),
    radial-gradient(2px 2px at 40px 70px, #fff, transparent),
    radial-gradient(1px 1px at 90px 40px, #fff, transparent),
    radial-gradient(1px 1px at 130px 80px, #fff, transparent),
    radial-gradient(2px 2px at 160px 30px, #fff, transparent);
  background-repeat: repeat;
  background-size: 200px 100px;
  animation: twinkle 4s infinite;
}

@keyframes twinkle {
  0%,
  100% {
    opacity: 0.3;
  }
  50% {
    opacity: 1;
  }
}

/* 气球动画 */
.floating-balloons {
  position: absolute;
  width: 100%;
  height: 100%;
}

.balloon {
  position: absolute;
  width: 40px;
  height: 50px;
  border-radius: 50% 50% 50% 50% / 60% 60% 40% 40%;
  animation: float 6s ease-in-out infinite;
}

.balloon-1 {
  background: #ff6b6b;
  left: 10%;
  top: 10%;
  animation-delay: 0s;
}

.balloon-2 {
  background: #4ecdc4;
  right: 10%;
  top: 20%;
  animation-delay: 1s;
}

.balloon-3 {
  background: #45b7d1;
  left: 20%;
  bottom: 30%;
  animation-delay: 2s;
}

.balloon-4 {
  background: #f9ca24;
  right: 20%;
  bottom: 20%;
  animation-delay: 3s;
}

@keyframes float {
  0%,
  100% {
    transform: translateY(0px) rotate(0deg);
  }
  33% {
    transform: translateY(-20px) rotate(1deg);
  }
  66% {
    transform: translateY(-10px) rotate(-1deg);
  }
}

/* 主要内容 */
.main-content {
  position: relative;
  z-index: 2;
  max-width: 400px;
  margin: 0 auto;
  text-align: center;
}

/* 头像区域样式 */
.avatar-section {
  margin-bottom: 30px;
  position: relative;
}

.avatar-container {
  position: relative;
  display: inline-block;
  animation: avatarEntrance 2s ease-out;
}

.avatar-frame {
  position: relative;
  width: 150px;
  height: 150px;
  margin: 0 auto;
  border-radius: 50%;
  padding: 8px;
  background: linear-gradient(45deg, #ff6b6b, #feca57, #48dbfb, #ff9ff3);
  background-size: 400% 400%;
  animation: gradientShift 3s ease-in-out infinite;
  box-shadow: 0 0 30px rgba(255, 255, 255, 0.3),
    0 0 60px rgba(255, 107, 107, 0.2), inset 0 0 20px rgba(255, 255, 255, 0.1);
}

.avatar-image {
  width: 100%;
  height: 100%;
  border-radius: 50%;
  object-fit: cover;
  border: 3px solid rgba(255, 255, 255, 0.8);
  transition: transform 0.3s ease;
}

.avatar-image:hover {
  transform: scale(1.05);
}

.avatar-glow {
  position: absolute;
  top: -10px;
  left: -10px;
  right: -10px;
  bottom: -10px;
  border-radius: 50%;
  background: radial-gradient(
    circle,
    rgba(255, 255, 255, 0.3) 0%,
    transparent 70%
  );
  animation: pulse 2s ease-in-out infinite;
  pointer-events: none;
}

.avatar-decoration {
  position: absolute;
  top: 50%;
  left: 50%;
  transform: translate(-50%, -50%);
  width: 200px;
  height: 200px;
  pointer-events: none;
}

.heart,
.star {
  position: absolute;
  font-size: 20px;
  animation: float 3s ease-in-out infinite;
}

.heart-1 {
  top: -10px;
  left: 20px;
  animation-delay: 0s;
}

.heart-2 {
  top: 30px;
  right: -10px;
  animation-delay: 1s;
}

.heart-3 {
  bottom: 20px;
  left: 10px;
  animation-delay: 2s;
}

.star-1 {
  top: 10px;
  right: 20px;
  animation-delay: 0.5s;
}

.star-2 {
  bottom: -5px;
  right: 30px;
  animation-delay: 1.5s;
}

.star-3 {
  bottom: 40px;
  right: -5px;
  animation-delay: 2.5s;
}

/* 标题区域 */
.title-section {
  margin-bottom: 40px;
}

.birthday-title {
  font-size: 2.5rem;
  font-weight: bold;
  color: #fff;
  text-shadow: 2px 2px 4px rgba(0, 0, 0, 0.3);
  margin-bottom: 10px;
}

.title-char {
  display: inline-block;
  animation: bounceIn 1s ease-out forwards;
  opacity: 0;
}

@keyframes bounceIn {
  0% {
    opacity: 0;
    transform: scale(0.3) translateY(-50px);
  }
  50% {
    opacity: 1;
    transform: scale(1.1) translateY(0);
  }
  100% {
    opacity: 1;
    transform: scale(1) translateY(0);
  }
}

.subtitle {
  color: #fff;
  font-size: 1.1rem;
  opacity: 0.9;
}

/* 蛋糕样式 */
.cake-container {
  margin: 40px 0;
  cursor: pointer;
}

.cake {
  position: relative;
  margin: 0 auto;
  width: 200px;
  height: 120px;
}

.cake-layer {
  position: absolute;
  left: 50%;
  transform: translateX(-50%);
  border-radius: 10px;
}

.cake-bottom {
  width: 180px;
  height: 40px;
  background: #8b4513;
  bottom: 0;
  box-shadow: 0 4px 8px rgba(0, 0, 0, 0.2);
}

.cake-middle {
  width: 140px;
  height: 35px;
  background: #dda0dd;
  bottom: 35px;
}

.cake-top {
  width: 100px;
  height: 30px;
  background: #ffb6c1;
  bottom: 65px;
}

.candles {
  position: absolute;
  top: 25px;
  left: 50%;
  transform: translateX(-50%);
  display: flex;
  gap: 15px;
}

.candle {
  width: 6px;
  height: 25px;
  background: #fff;
  border-radius: 3px;
  position: relative;
  transition: all 0.3s ease;
}

.flame {
  position: absolute;
  top: -8px;
  left: 50%;
  transform: translateX(-50%);
  width: 8px;
  height: 10px;
  background: radial-gradient(circle, #ffa500 0%, #ff4500 70%);
  border-radius: 50% 50% 50% 50% / 60% 60% 40% 40%;
  animation: flicker 1s ease-in-out infinite alternate;
}

.flame.out {
  opacity: 0;
  transform: translateX(-50%) scale(0);
}

@keyframes flicker {
  0% {
    transform: translateX(-50%) rotate(-1deg) scale(1);
  }
  100% {
    transform: translateX(-50%) rotate(1deg) scale(1.1);
  }
}

.cake-hint {
  margin-top: 20px;
  color: #fff;
  font-size: 1rem;
  animation: pulse 2s infinite;
}

@keyframes pulse {
  0%,
  100% {
    opacity: 0.7;
  }
  50% {
    opacity: 1;
  }
}

/* 祝福语样式 */
.wishes-section {
  margin: 40px 0;
}

.wish-text {
  color: #fff;
  font-size: 1.2rem;
  margin: 15px 0;
  padding: 15px;
  background: rgba(255, 255, 255, 0.1);
  border-radius: 15px;
  backdrop-filter: blur(10px);
  opacity: 0;
  transform: translateY(30px);
  animation: slideInUp 0.8s ease-out forwards;
}

.wish-text.show {
  opacity: 1;
  transform: translateY(0);
}

@keyframes slideInUp {
  to {
    opacity: 1;
    transform: translateY(0);
  }
}

/* 烟花效果 */
.fireworks {
  position: fixed;
  top: 0;
  left: 0;
  width: 100%;
  height: 100%;
  pointer-events: none;
  z-index: 3;
}

.firework {
  position: absolute;
  width: 4px;
  height: 4px;
  background: #fff;
  border-radius: 50%;
  animation: firework 2s ease-out forwards;
}

@keyframes firework {
  0% {
    transform: scale(1);
    opacity: 1;
    box-shadow: 0 0 0 0 #ff0080, 0 0 0 0 #00ff80, 0 0 0 0 #8000ff,
      0 0 0 0 #ff8000;
  }
  100% {
    transform: scale(20);
    opacity: 0;
    box-shadow: 30px 0 0 -2px #ff0080, -30px 0 0 -2px #00ff80,
      0 30px 0 -2px #8000ff, 0 -30px 0 -2px #ff8000, 21px 21px 0 -2px #ff0080,
      -21px -21px 0 -2px #00ff80, 21px -21px 0 -2px #8000ff,
      -21px 21px 0 -2px #ff8000;
  }
}

/* 重置按钮 */
.reset-btn {
  position: fixed;
  bottom: 20px;
  right: 20px;
  background: rgba(255, 255, 255, 0.2);
  color: #fff;
  border: 2px solid rgba(255, 255, 255, 0.3);
  border-radius: 50px;
  padding: 12px 20px;
  font-size: 1rem;
  cursor: pointer;
  backdrop-filter: blur(10px);
  transition: all 0.3s ease;
  z-index: 4;
}

.reset-btn:hover {
  background: rgba(255, 255, 255, 0.3);
  transform: scale(1.05);
}

/* 移动端优化 */
@media (max-width: 480px) {
  .birthday-app {
    padding: 15px;
  }

  /* 头像响应式 */
  .avatar-frame {
    width: 120px;
    height: 120px;
    padding: 6px;
  }

  .avatar-decoration {
    width: 160px;
    height: 160px;
  }

  .heart,
  .star {
    font-size: 16px;
  }

  .birthday-title {
    font-size: 2rem;
  }

  .cake {
    width: 160px;
    height: 100px;
  }

  .cake-bottom {
    width: 150px;
    height: 35px;
  }

  .cake-middle {
    width: 120px;
    height: 30px;
    bottom: 30px;
  }

  .cake-top {
    width: 90px;
    height: 25px;
    bottom: 55px;
  }

  .wish-text {
    font-size: 1.1rem;
    padding: 12px;
  }
}

/* 触摸设备优化 */
@media (hover: none) and (pointer: coarse) {
  .cake-container:active {
    transform: scale(0.95);
  }

  .reset-btn:active {
    transform: scale(0.95);
  }
}

/* 刮刮卡相关样式已移除 */

/* 头像动画关键帧 */
@keyframes avatarEntrance {
  0% {
    opacity: 0;
    transform: scale(0.3) rotate(-180deg);
  }
  50% {
    transform: scale(1.1) rotate(-90deg);
  }
  100% {
    opacity: 1;
    transform: scale(1) rotate(0deg);
  }
}

@keyframes gradientShift {
  0% {
    background-position: 0% 50%;
  }
  50% {
    background-position: 100% 50%;
  }
  100% {
    background-position: 0% 50%;
  }
}

@keyframes pulse {
  0%,
  100% {
    opacity: 0.3;
    transform: scale(1);
  }
  50% {
    opacity: 0.8;
    transform: scale(1.1);
  }
}

@keyframes float {
  0%,
  100% {
    transform: translateY(0px) rotate(0deg);
  }
  33% {
    transform: translateY(-10px) rotate(5deg);
  }
  66% {
    transform: translateY(5px) rotate(-3deg);
  }
}
</style>
