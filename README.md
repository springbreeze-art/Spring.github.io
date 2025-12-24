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
      /* 背景升级：深色科技蓝调 + 精细网格，模拟PCB或数字蓝图基底 */
      background-color: #020617; 
      background-image: 
        linear-gradient(rgba(30, 144, 255, 0.05) 1px, transparent 1px),
        linear-gradient(90deg, rgba(30, 144, 255, 0.05) 1px, transparent 1px);
      background-size: 30px 30px; /* 网格大小 */
      overflow: hidden;
      display: flex;
      flex-direction: column;
      align-items: center;
      justify-content: center;
      height: 100vh;
      font-family: 'Arial', sans-serif;
    }
    canvas {
      display: block;
      position: absolute;
      top: 0;
      left: 0;
      z-index: 1;
    }
    /* 祝福语卡片样式 - 增加了一点科技感的边框光晕 */
    .message-card {
      position: relative;
      z-index: 10;
      width: 85%;
      max-width: 500px;
      /* 背景稍微加深，提高对比度 */
      background: rgba(0, 10, 30, 0.75); 
      padding: 25px;
      border-radius: 12px;
      /* 边框改为红绿交替的微光，更有电路感 */
      border: 2px solid transparent;
      background-image: linear-gradient(rgba(0,10,30,0.75), rgba(0,10,30,0.75)), 
                        linear-gradient(45deg, #ff3333, #00ff00, #3399ff);
      background-origin: border-box;
      background-clip: padding-box, border-box;
      
      box-shadow: 0 0 30px rgba(0, 150, 255, 0.2); /* 科技蓝光晕 */
      text-align: center;
      color: #fff;
      margin-bottom: 40px;
      backdrop-filter: blur(5px);
      display: none;
      animation: popIn 1s ease-out;
    }

    .message-card h2 {
      margin: 0 0 15px 0;
      font-size: 1.8rem;
      color: #ff4d4d; /* 更亮的红 */
      text-shadow: 0 0 10px rgba(255, 0, 0, 0.5);
      font-family: 'Georgia', serif;
      letter-spacing: 1px;
    }

    .message-content {
      font-size: 1.05rem;
      line-height: 1.7;
      color: #e0eaff; /* 微蓝白色文字，更清晰 */
      white-space: pre-line;
    }

    .message-sign {
      margin-top: 20px;
      font-weight: bold;
      color: #00ff88; /* 科技荧光绿 */
      font-size: 1.2rem;
      text-align: right;
      text-shadow: 0 0 10px rgba(0, 255, 136, 0.4);
    }

    .ps-note {
      margin-top: 20px;
      font-size: 0.9rem;
      color: #ffcc00; /* 金色 */
      border-top: 1px solid rgba(30, 144, 255, 0.3); /* 蓝光分割线 */
      padding-top: 15px;
      font-style: italic;
      text-align: left;
    }

    @keyframes popIn {
      0% { transform: scale(0.9) translateY(20px); opacity: 0; }
      100% { transform: scale(1) translateY(0); opacity: 1; }
    }
  </style>
</head>
<body>

  <div class="message-card" id="card">
    <h2 id="greetingTitle"></h2>
    <div class="message-content" id="greetingBody"></div>
    <div class="message-sign" id="greetingSign"></div>
    <div class="ps-note" id="greetingPS" style="display:none;"></div>
  </div>

  <canvas id="treeCanvas"></canvas>

<script>
  /* ================= 配置区域 ================= */
  const greetings = {
    // 新增英文版 (设为默认)
    en: {
      title: "Merry Christmas!",
      body: "Wishing you and your family a holiday season filled with joy, peace, and warmth.\nMay the coming year bring you new opportunities, success, and prosperity.",
      sign: "Warm regards,\nSpring",
      ps: "P.S. May the spirit of the season shine on your path like a guiding star, lighting up the year ahead."
    },
    ru: {
      title: "С Рождеством!",
      body: "От всего сердца желаю вам и вашей семье здоровья, благополучия и счастливого Нового года!",
      sign: "Искренне ваш,\nЛеха",
      ps: ""
    },
    es: {
      title: "¡Feliz Navidad!",
      body: "Espero que esta Nochebuena esté llena de alegría y calidez para usted y toda su familia.\nLes deseo un año nuevo lleno de éxitos, salud y nuevas bendiciones.",
      sign: "Un cordial saludo,\nSpring",
      ps: ""
    },
    pt: {
      title: "Feliz Natal!",
      body: "Que esta data seja repleta de alegria e união para você e sua família.\nDesejo um ano novo cheio de luz, saúde e conquistas brilhantes!",
      sign: "Com cordiais saudações,\nSpring",
      ps: "P.S. Que o calor desta época aqueça seus dias como o abraço do sol do verão brasileiro."
    }
  };

  /* ================= 逻辑区域 ================= */
  const urlParams = new URLSearchParams(window.location.search);
  // 将默认语言改为英文 'en'
  const lang = urlParams.get('lang') || 'en'; 

  const config = greetings[lang] || greetings['en'];
  document.getElementById('greetingTitle').innerText = config.title;
  document.getElementById('greetingBody').innerText = config.body;
  document.getElementById('greetingSign').innerText = config.sign;
  
  const psDiv = document.getElementById('greetingPS');
  if(config.ps) {
    psDiv.innerText = config.ps;
    psDiv.style.display = 'block';
  }
  
  document.getElementById('card').style.display = 'block';


  /* ================= 科技感圣诞树核心代码 ================= */
  const canvas = document.getElementById('treeCanvas');
  const ctx = canvas.getContext('2d');
  let width, height;
  
  function resize() {
    width = canvas.width = window.innerWidth;
    height = canvas.height = window.innerHeight;
  }
  window.addEventListener('resize', resize);
  resize();

  const TREES_LAYERS = 55; // 稍微增加层数更紧密
  const MAX_RADIUS = 220;  
  let angle = 0;           

  // 雪花粒子（现在是科技碎片）
  const snowFlakes = [];
  for(let i=0; i<180; i++) {
    snowFlakes.push({
      x: Math.random() * width,
      y: Math.random() * height,
      size: Math.random() * 2 + 1, // 使用 size 代替半径
      speed: Math.random() * 1 + 0.5,
      opacity: Math.random() * 0.5 + 0.3
    });
  }

  function drawTree() {
    const cx = width / 2;
    const cy = height / 2 + 130; 

    ctx.clearRect(0, 0, width, height);

    // 绘制背景科技碎片（方形雪花）
    for(let f of snowFlakes) {
      // 微蓝白色的科技碎片
      ctx.fillStyle = `rgba(220, 240, 255, ${f.opacity})`;
      // 使用 fillRect 绘制方形，看起来像元器件或像素
      ctx.fillRect(f.x, f.y, f.size, f.size);
      
      f.y += f.speed;
      // 偶尔横向漂移一下，像数据流
      f.x += (Math.random() - 0.5) * 0.5;

      if(f.y > height) {
         f.y = -5;
         f.x = Math.random() * width;
      }
    }

    // 绘制“元器件”圣诞树
    for (let i = 0; i < TREES_LAYERS; i++) {
      let p = i / TREES_LAYERS;
      let y = cy - p * 420;
      let r = MAX_RADIUS * (1 - p);
      let currentAngle = angle + i * 0.5; 
      
      // 每一层点的数量
      let pointsCount = 16; 

      for(let j=0; j<pointsCount; j++) {
        let branchAngle = currentAngle + (j * (Math.PI * 2) / pointsCount);
        let x = cx + Math.cos(branchAngle) * r;
        let z = Math.sin(branchAngle); 
        
        // 基础大小
        let sizeBase = 3 + (1-p)*2;
        let size = sizeBase + z; // 近大远小透视
        
        let alpha = 0.7 + z * 0.3;

        // 颜色逻辑：主体绿色，混入一些科技蓝，和随机的红色高亮
        let hue = 130 + i*1.5; // 偏青绿色
        let saturation = "90%";
        let lightness = "60%";
        
        // 随机闪烁红灯 (像电路板上的指示灯)
        if (Math.random() < 0.04) {
             hue = 0; saturation = "100%"; lightness = "60%"; alpha = 1;
        }
        // 随机闪烁科技蓝灯
        else if (Math.random() < 0.06) {
             hue = 210; saturation = "100%"; lightness = "70%"; alpha = 1;
        }

        // 树顶金色信标
        if (p > 0.96) {
            hue = 50; saturation = "100%"; lightness = "70%"; alpha = 1; size += 2;
        }

        ctx.fillStyle = `hsla(${hue}, ${saturation}, ${lightness}, ${alpha})`;
        
        // 【关键修改】将树的组成粒子也改为方形 (fillRect)，模拟SMD元器件
        // 为了让旋转看起来不那么生硬，稍微减小一点方形的尺寸感
        ctx.fillRect(x - size/2, y - size/2, size*0.9, size*0.9);
      }
    }

    angle += 0.02; 
    requestAnimationFrame(drawTree);
  }

  drawTree();
</script>
</body>
</html>
