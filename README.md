<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8">
  <meta name="viewport" content="width=device-width, initial-scale=1.0, user-scalable=no">
  <title>Merry Christmas</title>
  <style>
    body {
      margin: 0;
      background: #000;
      overflow: hidden;
      color: #fff;
      font-family: 'Courier New', Courier, monospace; /* 程序员专用字体 */
    }
    canvas {
      display: block;
      cursor: move; 
    }
    
    /* 悬浮文字样式 */
    .overlay {
      position: absolute;
      bottom: 20px;
      width: 100%;
      text-align: center;
      pointer-events: none;
      z-index: 10;
    }
    
    .title {
      font-size: 2.5rem;
      font-weight: bold;
      background: linear-gradient(to right, #ff0000, #ffff00, #00ff00, #00ffff, #0000ff, #ff00ff, #ff0000);
      -webkit-background-clip: text;
      color: transparent;
      animation: rainbow 5s linear infinite;
      text-shadow: 0 0 10px rgba(255,255,255,0.3);
    }

    .subtitle {
      margin-top: 10px;
      font-size: 1rem;
      color: #0f0; /* 黑客绿 */
      text-shadow: 0 0 5px #0f0;
    }

    @keyframes rainbow {
      0% { background-position: 0% 50%; }
      100% { background-position: 100% 50%; }
    }
  </style>
</head>
<body>

<div class="overlay">
  <div class="title">Merry Christmas</div>
  <div class="subtitle">Code by Spring</div>
</div>

<canvas id="canvas"></canvas>

<script>
  const canvas = document.querySelector('canvas');
  const ctx = canvas.getContext('2d');

  let width, height;
  let t = 0; // 时间变量

  function resize() {
    width = canvas.width = window.innerWidth;
    height = canvas.height = window.innerHeight;
  }
  window.addEventListener('resize', resize);
  resize();

  // 3D 投影参数
  const fov = 350;
  const viewDistance = 5;
  
  // 交互控制
  let rotationSpeed = 0.02;
  let angleY = 0;
  let angleX = 0; // 稍微俯视

  // 粒子生成器
  const particles = [];
  const TOTAL_PARTICLES = 2500; // 极高密度，显卡燃烧！

  for (let i = 0; i < TOTAL_PARTICLES; i++) {
    // 使用螺旋公式分布粒子
    // y: 从 -100 到 100 (树高)
    // r: 半径随高度变化
    const percent = i / TOTAL_PARTICLES;
    
    // 形状控制：圆锥体
    const y = (percent * 400) - 200; 
    const radius = (200 - (y + 200) * 0.5) * (0.8 + Math.random()*0.2); 
    
    // 黄金角度分布，防止重叠
    const theta = i * 2.4; 

    particles.push({
      x: Math.cos(theta) * radius,
      y: y,
      z: Math.sin(theta) * radius,
      baseX: Math.cos(theta) * radius,
      baseZ: Math.sin(theta) * radius,
      color: Math.random() > 0.95 ? [255, 215, 0] : [0, 255, 127], // 95% 绿色，5% 金色
      blinkOffset: Math.random() * 100
    });
  }

  // 鼠标/触摸交互
  let startX = 0;
  document.addEventListener('touchstart', e => startX = e.touches[0].clientX);
  document.addEventListener('touchmove', e => {
    const delta = e.touches[0].clientX - startX;
    angleY += delta * 0.005;
    startX = e.touches[0].clientX;
  });
  document.addEventListener('mousemove', e => {
    // 简单的鼠标跟随旋转
    angleY = (e.clientX / width - 0.5) * 2;
  });

  function draw() {
    // 背景：带一点极光蓝的深黑
    ctx.fillStyle = '#020510'; 
    ctx.fillRect(0, 0, width, height);
    
    // 绘制“极光”光晕背景
    const gradient = ctx.createRadialGradient(width/2, height/2, 0, width/2, height/2, 600);
    gradient.addColorStop(0, 'rgba(0, 50, 100, 0.2)');
    gradient.addColorStop(1, 'rgba(0, 0, 0, 0)');
    ctx.fillStyle = gradient;
    ctx.fillRect(0,0,width,height);

    ctx.save();
    ctx.translate(width / 2, height / 2 + 50);

    // 排序粒子，保证3D遮挡关系正确 (画家算法)
    particles.sort((a, b) => b.pz - a.pz);

    t += 0.05; // 时间流动

    particles.forEach(p => {
      // 1. 旋转变换
      let x = p.x;
      let y = p.y;
      let z = p.z;

      // 绕Y轴旋转 (自动 + 鼠标)
      const cosY = Math.cos(angleY + t * rotationSpeed);
      const sinY = Math.sin(angleY + t * rotationSpeed);
      
      let x1 = x * cosY - z * sinY;
      let z1 = z * cosY + x * sinY;
      
      // 2. 呼吸效果 (让树有生命感)
      // 利用正弦波让粒子在半径方向轻微浮动
      const breathe = 1 + Math.sin(t * 0.1 + p.y * 0.02) * 0.03;
      x1 *= breathe;
      z1 *= breathe;

      // 保存变换后的Z用于排序
      p.pz = z1;

      // 3. 3D 透视投影
      const scale = fov / (fov + z1 + 400);
      const x2d = x1 * scale;
      const y2d = y * scale;

      // 4. 绘制
      if (scale > 0) {
        // 闪烁逻辑
        const blink = Math.sin(t * 0.2 + p.blinkOffset);
        let alpha = scale * 0.8;
        if(blink > 0.8) alpha = 1; // 高亮时刻
        
        // 颜色处理
        const [r, g, b] = p.color;
        
        // 如果是金色粒子，画大一点
        if (r === 255) {
             ctx.fillStyle = `rgba(255, 255, 200, ${alpha})`;
             ctx.shadowBlur = 15;
             ctx.shadowColor = 'orange';
             ctx.beginPath();
             ctx.arc(x2d, y2d, 2.5 * scale, 0, Math.PI * 2);
             ctx.fill();
             ctx.shadowBlur = 0; // 重置，优化性能
        } else {
             // 普通绿色粒子，加一点科技蓝
             ctx.fillStyle = `rgba(${r}, ${g}, ${b + Math.sin(t)*50}, ${alpha})`;
             ctx.fillRect(x2d, y2d, 2 * scale, 2 * scale); // 用方形模拟像素感
        }
      }
    });

    // 绘制树顶星星
    ctx.shadowBlur = 40;
    ctx.shadowColor = "#ffff00";
    ctx.fillStyle = "#fff";
    ctx.beginPath();
    ctx.arc(0, -220 * (1 + Math.sin(t * 0.1)*0.03), 6, 0, Math.PI*2);
    ctx.fill();
    ctx.shadowBlur = 0;

    ctx.restore();
    requestAnimationFrame(draw);
  }

  draw();
</script>
</body>
</html>
