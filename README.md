<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <title>AE-圣诞树</title>
    <script src="https://code.jquery.com/jquery-3.6.0.min.js"></script>
    <style>
        html, body {
            margin: 0;
            width: 100%;
            height: 100%;
            overflow: hidden; /* 防止滚动条出现 */
            font-family: 'Arial', sans-serif;
        }

        #main {
            position: fixed;
            top: 0;
            left: 0;
            width: 100%;
            height: 100%;
        }

        #video1 {
            width: 100%;
            height: 100%;
            object-fit: cover; /* 确保视频填满屏幕且不变形 */
            z-index: 1;
        }

        .hidden {
            display: none !important;
        }

        /* --- 新增：祝福语区域样式 --- */
        #greetings-container {
            position: fixed;
            /* 位置调整：设置在右侧大约三分之一的位置，高度居中偏上 */
            top: 25%; 
            right: 5%;
            width: 35%; /* 限制宽度，不要太宽 */
            z-index: 10; /* 确保在视频上方 */
            text-align: center;
            color: #FFFDD0; /* 奶油色/金色文字，适合深色背景 */
            text-shadow: 2px 2px 8px rgba(0,0,0,0.8); /* 增加文字阴影，提高在视频上的可读性 */
            pointer-events: none; /* 让鼠标点击穿透文字，不影响视频交互 */
        }

        /* 模拟圣诞风格字体的备选方案 */
        .christmas-font {
            font-family: "Brush Script MT", "Comic Sans MS", "Lucida Handwriting", cursive, serif;
            font-weight: bold;
        }

        /* 第一排大字样式 */
        .greetings-title {
            font-size: 2.5em; /* 字体不用太大，但要突出 */
            margin: 0 0 15px 0;
            color: #e61919; /* 圣诞红 */
            text-shadow: 1px 1px 2px yellow, 0 0 1em red; /* 发光效果 */
        }

        /* 第二排祝福语样式 */
        .greetings-body {
            font-size: 1.2em; /* 较小的字体 */
            line-height: 1.6;
            font-weight: normal;
        }

        /* 淡入动画效果 */
        .fade-in {
            animation: fadeInAnimation ease 3s;
            animation-iteration-count: 1;
            animation-fill-mode: forwards;
        }
        @keyframes fadeInAnimation {
            0% { opacity: 0; transform: translateY(20px); }
            100% { opacity: 1; transform: translateY(0); }
        }

    </style>
</head>

<body id="app">
    <div id="main">
        <video id="video1" muted="muted" autoplay="autoplay" loop="loop" src="./video/chrismas.mp4">
            您的浏览器不支持 HTML5 video 标签。
        </video>
        <audio id="music1" loop="loop" preload="auto" controls class="hidden">
            <source src="./music/jinniandongtianpeizhewoba.mp3" type="audio/mpeg">
        </audio>

        <div id="greetings-container" class="hidden">
            <div id="text-en" class="greeting-text hidden">
                <h1 class="christmas-font greetings-title">Merry Christmas!</h1>
                <p class="christmas-font greetings-body">Spring wishes you and your family happiness and health, and a happy New Year.</p>
            </div>
            <div id="text-ru" class="greeting-text hidden">
                <h1 class="christmas-font greetings-title">Merry Christmas!</h1>
                <p class="christmas-font greetings-body">Леха желает вам и вашей семье счастья, здоровья и счастливого Нового года.</p>
            </div>
            <div id="text-es" class="greeting-text hidden">
                <h1 class="christmas-font greetings-title">Merry Christmas!</h1>
                <p class="christmas-font greetings-body">Spring les desea a usted y a su familia felicidad, salud y un próspero Año Nuevo.</p>
            </div>
            <div id="text-pt" class="greeting-text hidden">
                <h1 class="christmas-font greetings-title">Merry Christmas!</h1>
                <p class="christmas-font greetings-body">Spring deseja a você e sua família felicidade, saúde e um feliz Ano Novo.</p>
            </div>
        </div>

    </div>

<script type="text/javascript">
    // --- 原有的视频播放设置 ---
    var video = document.getElementById("video1");
    // 降低播放速度，可以根据需要调整或注释掉
    video.playbackRate *= 0.8;
    
    // 尝试播放视频 (现代浏览器可能会因为未静音而阻止自动播放，但您的标签里已经有 muted 了)
    video.play().catch(function(error) {
        console.log("Video autoplay failed initially:", error);
    });
    
    // 原有的 jQuery 代码块 (保持不变)
    $(function() {
        // var video = document.getElementById("video1");
        // video.src = "";
        // video.play();

        // var music = document.getElementById("music1");
        // music.src = "";
        // music.play();
    });
</script>

<script>
    document.addEventListener("DOMContentLoaded", function() {
        var videoElement = document.getElementById("video1");
        var greetingsContainer = document.getElementById("greetings-container");
        var hasTriggered = false; // 确保只触发一次显示

        // =================================================================
        // 【重要设置】请修改这里！
        // 请观看您的视频，找到星星亮起并呈现全景的那一秒。
        // 将下面的数字 5 修改为您视频中的具体秒数 (例如 8.5)。
        var triggerTime = 5; 
        // =================================================================

        // 1. 获取URL参数以确定语言 (例如：index.html?lang=ru)
        const urlParams = new URLSearchParams(window.location.search);
        // 默认为英语 'en'
        const currentLang = urlParams.get('lang') || 'en';
        console.log("Current Language selected:", currentLang);

        // 2. 监听视频播放时间更新事件
        videoElement.addEventListener("timeupdate", function() {
            // 如果当前时间大于设定时间，且尚未触发过
            if (!hasTriggered && videoElement.currentTime >= triggerTime) {
                showGreetings(currentLang);
                hasTriggered = true; // 标记为已触发
            }
        });

        // 显示祝福语的函数
        function showGreetings(langCode) {
            // 先确保所有文本都隐藏
            var allTexts = document.querySelectorAll('.greeting-text');
            allTexts.forEach(function(el) {
                el.classList.add('hidden');
            });

            // 获取对应语言的文本元素
            var selectedText = document.getElementById("text-" + langCode);
            
            // 如果找不到对应的语言（比如输入了错误的参数），则默认显示英语
            if (!selectedText) {
                selectedText = document.getElementById("text-en");
            }

            // 显示主容器
            greetingsContainer.classList.remove('hidden');
            // 显示选中的语言文本并添加淡入效果
            selectedText.classList.remove('hidden');
            selectedText.classList.add('fade-in');
        }

        // 如果视频循环播放，需要在视频重新开始时重置触发器
        videoElement.addEventListener("seeking", function() {
             if (videoElement.currentTime < triggerTime) {
                 greetingsContainer.classList.add('hidden');
                 var allTexts = document.querySelectorAll('.greeting-text');
                 allTexts.forEach(el => el.classList.remove('fade-in', 'hidden'));
                 hasTriggered = false;
             }
        });
    });
</script>

<script>
    // --- 原有的音频自动播放 Hack 脚本 (保持不变) ---
    var myAuto = document.getElementById('music1');
    var app = document.getElementById("app");
    
    // 监听鼠标移动事件来触发音频播放 (应对浏览器自动播放策略)
    var autoPlayHandler = function() {
        console.log("User interacted, attempting to play music.");
        myAuto.play().then(function() {
             console.log("Music playback started.");
             // 播放成功后移除监听器，避免重复触发
             app.removeEventListener("mousemove", autoPlayHandler, true);
             app.removeEventListener("touchstart", autoPlayHandler, true);
        }).catch(function(error) {
            console.log("Music playback failed:", error);
        });
    };

    // 同时监听鼠标和触摸事件，增加移动端的成功率
    app.addEventListener("mousemove", autoPlayHandler, true);
    app.addEventListener("touchstart", autoPlayHandler, true);

    // 原有的定时器模拟事件 (效果有限，但保留以兼容旧逻辑)
    setInterval(function() {
        if ("createEvent" in document) {
            var evt = document.createEvent("HTMLEvents");
            evt.initEvent("mousemove", false, true);
            app.dispatchEvent(evt);
        } else{
            // IE support (obsolete but kept as in original source)
            if (app.fireEvent) {
                 app.fireEvent("onmousemove");
            }
        }
    }, 1000);
</script>
</html>
