<template>
  <div class="wheel-container">
    <div class="canvas-wrapper">
      <canvas
        ref="wheelCanvas"
        width="600"
        height="600"
        @click="handleCanvasClick"
      ></canvas>
      <img class="pointer1" :src="pointerImg" alt="pointer1" />
    </div>
  </div>
</template>

<script setup>
import { ref, onMounted } from 'vue';
import pointerImg from '@/assets/pointer1.png';
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
const colors = ['#ffffff', '#aaaaaa'];  


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
    ctx.fillStyle = colors[i % 2] === '#aaaaaa' ? '#fff' : '#000';
    ctx.font = 'bold 38px sans-serif';
    ctx.fillText(options[i], radius - 10, 10);
    ctx.restore();
  }

  // GO 金色圓圈
  const gradient = ctx.createRadialGradient(centerX, centerY, 10, centerX, centerY, 30);
  gradient.addColorStop(0, '#fff9db');
  gradient.addColorStop(1, '#d4af37');

  ctx.beginPath();
  ctx.fillStyle = gradient;
  ctx.arc(centerX, centerY, 45, 0, 2 * Math.PI);
  ctx.fill();
  
  ctx.fillStyle = '#000';
  ctx.font = 'bold 30px Arial Rounded MT Bold';
  
  ctx.textAlign = 'center';
  ctx.fillText('GO!', centerX, centerY + 8);
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
    .filter(item => item.opt !== '現折1500元' && item.opt !== '現折2000元')
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
          <p style="font-size: 20px;">🎉 恭喜您 !！</p>
           <p style="font-size: 30px; font-weight: bold; color: #b22222;">
            ${options[targetIndex]}
          </p>
        `,
        icon: 'success',
        confirmButtonText: '太棒了！',
        confirmButtonColor: '#d4af37'
      }).then(() => {
        spinning = false;
      });
    }
  };

  requestAnimationFrame(animate);
};

const easeOut = (t) => 1 - Math.pow(1 - t, 3);

onMounted(() => {
  drawWheel();
});
</script>

<style scoped>
.wheel-container {
  display: flex;
  justify-content: center;
  align-items: center;
  height: 100vh;
  background-color: #f2f2f2;
}
.canvas-wrapper {
  position: relative;
  width: 1365px;
  height: 768px;
  background-image: url('@/assets/BG.jpg');
  background-size: cover;
  background-position: center;
}
canvas {
  position: absolute;
  top: 52%;
  left: 58%;
  transform: translate(-50%, -50%);
  width: 600px;
  height: 600px;
  border-radius: 50%;
}
.pointer1 {
  position: absolute;
  top: calc(52% - 340px);
  left: 58%;              /* ✅ 與 canvas 一樣，才能水平對齊 */
  transform: translateX(-50%);
  width: 60px;
  height: 120px;
  pointer-events: none;
}

</style>
