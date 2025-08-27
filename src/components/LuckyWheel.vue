<template>
  <div class="wheel-container">
    <div class="canvas-wrapper">
      <canvas
        ref="wheelCanvas"
        width="600"
        height="600"
        @click="handleCanvasClick"
      ></canvas>
      <img class="pointer2" :src="pointerImg" alt="pointer2" />
    </div>
  </div>
</template>

<script setup>
import { ref, onMounted } from 'vue';
import pointerImg from '@/assets/pointer2.png';
import Swal from 'sweetalert2';

const wheelCanvas = ref(null);
const options = [
  '現折300元',
  '現折500元',
  '現折600元',
  '現折800元',
  '現折1000元',
  '現折1200元',
  '現折1500元',
  '現折2000元'
];
const colors = ['#ffffff', '#889CBB'];  


let angle = 0;
let spinning = false;

const drawWheel = () => {
  const canvas = wheelCanvas.value;
  const ctx = canvas.getContext('2d');
  const centerX = canvas.width / 2;
  const centerY = canvas.height / 2;
  const radius = 280;
  const arc = (2 * Math.PI) / options.length;

  ctx.clearRect(0, 0, canvas.width, canvas.height);

  for (let i = 0; i < options.length; i++) {
    const startAngle = angle + i * arc;

    ctx.beginPath();
    ctx.fillStyle = colors[i % 2];
    ctx.moveTo(centerX, centerY);
    ctx.arc(centerX, centerY, radius, startAngle, startAngle + arc);
    ctx.fill();

    ctx.save();
    ctx.translate(centerX, centerY);
    ctx.rotate(startAngle + arc / 2);
    ctx.textAlign = 'right';
    // 設定文字顏色：灰底（奇數）→ 白字；白底（偶數）→ 深藍字
    ctx.fillStyle = i % 2 === 1 ? '#ffffff' : '#0f3049';
    ctx.font = 'bold 38px sans-serif';
    ctx.fillText(options[i], radius - 10, 10);
    ctx.restore();
  }

  // START 藍色圓圈
  const gradient = ctx.createRadialGradient(centerX, centerY, 10, centerX, centerY, 30);
  gradient.addColorStop(0, '#fff9db');
  gradient.addColorStop(1, '#0f3049');
  // START 按鈕區塊 - 使用深藍色實心圓
  ctx.beginPath();
  ctx.fillStyle = '#0f3049'; // 深藍色，不用 gradient
  ctx.arc(centerX, centerY, 45, 0, 2 * Math.PI);
  ctx.fill();
  // START按鈕文字
  ctx.fillStyle = '#ffffff'; // 白字
  ctx.font = 'bold 25px Arial Rounded MT Bold';
  ctx.textAlign = 'center';
  ctx.fillText('START', centerX, centerY + 8);
};

const handleCanvasClick = (e) => {
  const canvas = wheelCanvas.value;
  const rect = canvas.getBoundingClientRect();
  const x = e.clientX - rect.left;
  const y = e.clientY - rect.top;
  const centerX = canvas.width / 2;
  const centerY = canvas.height / 2;
  const dx = x - centerX;
  const dy = y - centerY;
  const distance = Math.sqrt(dx * dx + dy * dy);

  if (distance <= 30) spin();
};

const spin = () => {
  if (spinning) return;
  spinning = true;

  const duration = 3000;
  let start = null;

  const validIndexes = options
    .map((opt, idx) => ({ opt, idx }))
    .filter(item => item.opt !== '現折300元' && item.opt !== '現折2000元')
    .map(item => item.idx);

  const totalAngle = 2 * Math.PI;
  const arc = totalAngle / options.length;
  const pointerOffset = (3 * Math.PI) / 2;

  const targetIndex = validIndexes[Math.floor(Math.random() * validIndexes.length)];
  const stopAngle = totalAngle - (targetIndex * arc) - arc / 2 + pointerOffset;
  const rotation = stopAngle + 2 * Math.PI * 5;

  const animate = (timestamp) => {
    if (!start) start = timestamp;
    const progress = timestamp - start;
    const easing = easeOut(progress / duration);
    angle = rotation * easing;
    drawWheel();

    if (progress < duration) {
      requestAnimationFrame(animate);
    } else {
    Swal.fire({
        title: 'Homelike 喜家居',
        html: `
          <div style="text-align: center;">
            <p style="font-size: 60px; margin: 0;">🎁 恭喜您 🎁</p>
            <p class="swal-golden-text">${options[targetIndex]}</p>
          </div>
        `,
        icon: 'success',
        confirmButtonText: '太棒了！',
        confirmButtonColor: '#d4af37',
        width: 1000,
        customClass: {
           title: 'custom-swal-title',
          popup: 'custom-swal-popup'
        }
      }).then(() => {
        spinning = false;
      });


    }
  };

  requestAnimationFrame(animate);
};

const easeOut = (t) => 1 - Math.pow(1 - t, 3);

onMounted(() => {
   // 讓初始轉盤就對準「現折300元」
  const targetIndex = options.indexOf('現折300元');
  const totalAngle = 2 * Math.PI;
  const arc = totalAngle / options.length;
  const pointerOffset = (3 * Math.PI) / 2;

  // 指針要對準的角度
  angle = totalAngle - (targetIndex * arc) - arc / 2 + pointerOffset;
  drawWheel();
});
</script>

<style>
.wheel-container {
  position: relative;
  width: 100%;
  max-width: 1366px;
  margin: 0 auto;
}
.canvas-wrapper {
  position: relative;
  width: 1365px;
  height: 768px;
  background-image: url('@/assets/newBG.jpg');
  background-size: cover;
  background-position: center;
}
canvas {
  position: absolute;
  top: 52%;
  left: 59%;
  transform: translate(-50%, -50%);
  width: 600px;
  height: 600px;
  border-radius: 50%;
}
.pointer2 {
  position: absolute;
  top: calc(52% - 350px); /* 可微調，目的是讓指針底部靠近圓盤上邊緣 */
  left: 59%;
  transform: translateX(-50%);
  width: 50px;   /* 調小才不會壓到轉盤 */
  height: auto;  /* 保持原圖比例 */
  pointer-events: none;
  z-index: 10;
}
/* SweetAlert2 樣式 (必須用全域) */
.custom-swal-popup {
  padding: 40px;
  text-align: center;
}
.swal-golden-text {
  font-size: 100px;
  font-weight: bold;
   background: linear-gradient(
    90deg,
    #89CFF0,   /* 淺藍 */
    #4682B4,   /* 鋼藍 */
    #0f3049,   /* 深藍 */
    #4682B4,
    #89CFF0
  );
  background-size: 200% auto;
  -webkit-background-clip: text;
  -webkit-text-fill-color: transparent;
  animation: shimmer 2s linear infinite;
  margin: 20px 0;
}
/* 漸層流動 */
@keyframes shimmer {
  0% { background-position: -200px 0; }
  100% { background-position: 200px 0; }
}
.custom-swal-title {
  font-size: 60px;      /* 調大標題字體 */
  font-weight: bold;    /* 加粗 */    
  margin-bottom: 20px;
}
</style>