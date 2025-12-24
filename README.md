<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8">
  <meta name="viewport" content="width=device-width, initial-scale=1.0, user-scalable=no">
  <title>Merry Christmas</title>
  <style>
    body {
      margin: 0;
      padding: 0;
      background-color: #000;
      overflow: hidden;
      font-family: 'Arial', sans-serif;
    }
    
    /* 悬浮文字层 */
    .overlay {
      position: absolute;
      top: 50%;
      left: 50%;
      transform: translate(-50%, -50%);
      text-align: center;
      pointer-events: none;
      z-index: 10;
      mix-blend-mode: overlay; /* 让文字融入背景光效 */
    }

    h1 {
      color: #fff;
      font-size: 3rem;
      margin: 0;
      text-shadow: 0 0 20px rgba(255, 255, 255, 0.8), 0 0 40px #00ffcc;
      letter-spacing: 5px;
      font-weight: 300;
    }
    
    p {
      color: #00ffcc;
      font-size: 1rem;
      letter-spacing: 3px;
      margin-top: 10px;
      text-shadow: 0 0 10px #00ffcc;
    }

  </style>
</head>
<body>

  <div class="overlay">
    <h1>Merry Christmas</h1>
    <p>Design for You</p>
  </div>
  
  <canvas id="canvas"></canvas>

<script>
  const canvas = document.getElementById('canvas');
  const ctx = canvas.getContext('2d');

  let w, h;
  
  // 适配屏幕
  function resize() {
    w = canvas.width = window.innerWidth;
    h = canvas.height = window.innerHeight;
  }
  window.addEventListener('resize', resize);
  resize();

  // 配置参数
  const COUNT = 3000; // 粒子数量，越多越密
  const DEPTH = 800;  // 视距
  
  // 粒子存储
  const stars = [];

  // 初始化粒子
  for (let i = 0; i < COUNT; i++) {
    stars.push({
      x: 0, y: 0, z: 0,
      angle: Math.random() * Math.PI * 2, // 初始角度
      radiusOffset: Math.random() * 50,   // 半径随机偏移，让树看起来不那么死板
      speed: Math.random() * 0.02 + 0.005, // 旋转速度
      h: i / COUNT, // 高度百分比 0(顶) -> 1(底)
      color: Math.random() // 颜色随机因子
    });
  }

  let rotation = 0; // 整体旋转角度

  function animate() {
    // 1. 拖尾效果：用半透明黑色覆盖，而不是完全清空
    ctx.fillStyle = 'rgba(0, 0, 0, 0.15)'; 
    ctx.fillRect(0, 0, w, h);

    // 2. 将坐标原点移到屏幕中心，并向下稍微移动一点，保证树在中间
    ctx.save();
    ctx.translate(w / 2, h / 2 + 50);

    // 开启发光模式（关键！这能让粒子重叠处变亮，产生光晕感）
    ctx.globalCompositeOperation = 'lighter';

    rotation += 0.008; // 自动旋转

    stars.forEach(p => {
      // --- 核心数学修正 ---
      // 树高范围：从 -300 (上) 到 300 (下)
      // 在 Canvas 中，负数是上方，正数是下方
      let logicY = -350 + p.h * 700; 
      
      // 树形半径修正：顶部(p=0)半径小，底部(p=1)半径大
      // 使用 p * p 让树尖更尖，树底更宽
      let logicR = p.h * p.h * 280 + 10; 

      // 螺旋运动
      let angle = p.angle + rotation + logicY * 0.005; // 加上 logicY 让螺旋扭曲
      
      // 算出 3D 坐标
      let x = Math.cos(angle) * (logicR + p.radiusOffset);
      let z = Math.sin(angle) * (logicR + p.radiusOffset);
      let y = logicY;

      // --- 3D 投影到 2D ---
      let scale = DEPTH / (DEPTH + z); // 透视公式
      let x2d = x * scale;
      let y2d = y * scale;
      let size = (1.5 + p.h) * scale; // 离得近的大，离得远的小

      if (scale > 0) {
        // 颜色计算：根据高度和随机因子生成 蓝-绿-金 渐变
        // 顶部偏金/白，底部偏绿/蓝
        let hue = 140 + p.h * 50; // 绿色范围
        let sat = 100;
        let light = 50 + (1-p.h)*40; // 顶部更亮
        
        // 偶尔闪烁的星星
        if (Math.random() < 0.01) { light = 100; size *= 2; }

        ctx.fillStyle = `hsl(${hue}, ${sat}%, ${light}%)`;
        ctx.beginPath();
        ctx.arc(x2d, y2d, size, 0, Math.PI * 2);
        ctx.fill();
      }
    });
    
    // 绘制顶部的星星 (永远在最上面)
    ctx.fillStyle = "#fff";
    ctx.shadowBlur = 20;
    ctx.shadowColor = "#fff";
    ctx.beginPath();
    ctx.arc(0, -350 * (DEPTH/(DEPTH-50)), 6, 0, Math.PI*2); // 简单的透视估算
    ctx.fill();
    ctx.shadowBlur = 0;

    ctx.restore();
    
    requestAnimationFrame(animate);
  }

  animate();
</script>
</body>
</html>
