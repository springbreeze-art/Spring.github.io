<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8">
  <title>Merry Christmas</title>
  <style>
    body {
      margin: 0;
      overflow: hidden;
      background: #000;
      cursor: pointer;
    }
    canvas {
      display: block;
    }
    .overlay {
      position: absolute;
      top: 50%;
      left: 50%;
      transform: translate(-50%, -50%);
      pointer-events: none;
      text-align: center;
      z-index: 10;
    }
    h1 {
      color: rgba(255, 255, 255, 0.8);
      font-family: 'Arial', sans-serif;
      font-weight: 300;
      font-size: 3rem;
      letter-spacing: 5px;
      text-shadow: 0 0 10px rgba(255,255,255,0.5);
      mix-blend-mode: screen;
      animation: fade 3s infinite alternate;
    }
    @keyframes fade {
      0% { opacity: 0.5; }
      100% { opacity: 1; }
    }
  </style>
</head>
<body>

<div class="overlay">
  <h1>Merry Christmas</h1>
</div>

<script>
  // 粒子特效核心代码
  const canvas = document.createElement('canvas');
  document.body.appendChild(canvas);
  const ctx = canvas.getContext('2d');

  let width, height;
  let particles = [];
  
  // 树的参数
  const PARTICLE_COUNT = 1200; // 粒子数量，越多越密
  
  function resize() {
    width = canvas.width = window.innerWidth;
    height = canvas.height = window.innerHeight;
  }
  window.addEventListener('resize', resize);
  resize();

  // 粒子类
  class Particle {
    constructor() {
      this.reset();
      this.y = Math.random() * height; // 初始随机位置
    }

    reset() {
      // 随机分布在圆锥体内
      this.angle = Math.random() * Math.PI * 2;
      this.h = Math.random(); // 高度百分比 0-1
      
      // 树的形状控制
      // 半径随着高度变小
      let maxR = 200 * (1 - this.h); 
      this.r = Math.random() * maxR;
      
      this.x = Math.cos(this.angle) * this.r;
      this.z = Math.sin(this.angle) * this.r;
      this.y = -300 + this.h * 600; // 树的高度范围
      
      // 粒子自身的属性
      this.size = Math.random() * 2 + 0.5;
      this.speed = Math.random() * 0.02 + 0.005;
      
      // 颜色：绿色为主，点缀红黄
      let rand = Math.random();
      if(rand > 0.95) this.color = '#ffcc00'; // 星星黄
      else if(rand > 0.9) this.color = '#ff3333'; // 彩灯红
      else this.color = `hsl(${100 + Math.random()*60}, 80%, 60%)`; // 树绿
    }

    update() {
      // 旋转动画
      this.angle += this.speed;
      this.x = Math.cos(this.angle) * this.r;
      this.z = Math.sin(this.angle) * this.r;
    }
  }

  // 初始化粒子
  for(let i=0; i<PARTICLE_COUNT; i++) {
    particles.push(new Particle());
  }

  // 鼠标交互
  let mouseX = 0;
  let mouseY = 0;
  document.addEventListener('mousemove', (e) => {
    mouseX = (e.clientX - width/2) * 0.001;
    mouseY = (e.clientY - height/2) * 0.001;
  });
  // 手机触摸交互
  document.addEventListener('touchmove', (e) => {
    mouseX = (e.touches[0].clientX - width/2) * 0.002;
    mouseY = (e.touches[0].clientY - height/2) * 0.002;
  });

  function animate() {
    ctx.fillStyle = '#000';
    ctx.fillRect(0, 0, width, height);

    particles.sort((a, b) => b.z - a.z); // 简单的深度排序

    const cx = width / 2;
    const cy = height / 2 + 50;

    for(let p of particles) {
      p.update();

      // 3D 投影到 2D
      // 简单的旋转矩阵应用（跟随鼠标）
      let x = p.x;
      let y = p.y;
      let z = p.z;

      // 绕X轴旋转
      let y1 = y * Math.cos(mouseY) - z * Math.sin(mouseY);
      let z1 = y * Math.sin(mouseY) + z * Math.cos(mouseY);
      y = y1; z = z1;

      // 绕Y轴旋转
      let x1 = x * Math.cos(mouseX) - z * Math.sin(mouseX);
      let z2 = x * Math.sin(mouseX) + z * Math.cos(mouseX);
      x = x1; z = z2;

      // 透视投影
      let scale = 400 / (400 + z); 
      let x2d = cx + x * scale;
      let y2d = cy + y * scale;

      if(z > -300) { // 裁剪掉太远的
        ctx.fillStyle = p.color;
        ctx.beginPath();
        ctx.arc(x2d, y2d, p.size * scale, 0, Math.PI*2);
        ctx.fill();
        
        // 发光效果
        if(Math.random() > 0.98) {
             ctx.shadowBlur = 10;
             ctx.shadowColor = p.color;
             ctx.fill();
             ctx.shadowBlur = 0;
        }
      }
    }

    requestAnimationFrame(animate);
  }

  animate();
</script>
</body>
</html>
