# baodacezhi.github.io
<!DOCTYPE html>
<html lang="zh-CN">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>射击小游戏：二号打一号</title>
    <!-- 引入 Tailwind CSS for modern styling -->
    <script src="https://cdn.tailwindcss.com"></script>
    <style>
        /* 使用 Inter 字体 */
        :root {
            font-family: 'Inter', sans-serif;
        }

        /* 核心摇晃动画 (用于目标被击中时) */
        @keyframes shake {
            0%, 100% { transform: translateX(0); }
            10%, 30%, 50%, 70%, 90% { transform: translateX(-8px); }
            20%, 40%, 60%, 80% { transform: translateX(8px); }
        }

        /* 摇晃类 */
        .shake {
            animation: shake 0.4s cubic-bezier(.36,.07,.19,.97) both;
            transform: translate3d(0, 0, 0);
            backface-visibility: hidden;
            perspective: 1000px;
        }

        /* 子弹的样式 */
        .bullet {
            position: absolute;
            width: 12px;
            height: 6px;
            background-color: #ff0000; /* 红色子弹 */
            border-radius: 6px;
            opacity: 0;
            transition: all 0.5s ease-in; /* 子弹飞行时间 */
            z-index: 10;
        }

        /* 子弹飞行时的状态 */
        .bullet-firing {
            transform: translateX(350px); /* 飞行距离 */
            opacity: 1;
        }

        /* 射击者和目标的容器 */
        .character-container {
            width: 150px;
            height: 150px;
            display: flex;
            justify-content: center;
            align-items: center;
        }

        .character-image {
            width: 100%;
            height: 100%;
            object-fit: contain;
            user-select: none;
            pointer-events: none;
        }
    </style>
</head>
<body class="bg-gray-100 min-h-screen flex flex-col items-center justify-center p-4">

    <!-- 游戏主容器 -->
    <div class="bg-white p-8 rounded-xl shadow-2xl w-full max-w-4xl">
        <h1 class="text-3xl font-bold text-center mb-6 text-gray-800">🎯 二号打一号 🎯</h1>
        
        <!-- 游戏场景 -->
        <div id="game-scene" class="relative flex justify-between items-end h-64 md:h-80 border-b-4 border-gray-300 pb-4">
            
            <!-- 二号 (射击者) -->
            <div id="shooter" class="flex flex-col items-center">
                <div class="character-container">
                    <img id="player-two" 
                         class="character-image" 
                         src="uploaded:6647236adfa9a93194f956e4f7cd36ef.png-d39abe0e-77f7-41ae-a99b-34fde43ba6b1" 
                         alt="二号 (射击者)">
                </div>
                <div class="text-lg font-semibold mt-2">二号 (Shooter)</div>
                <!-- 枪口位置的模拟和子弹起点 -->
                <div id="gun-muzzle" class="absolute top-1/2 left-[120px] -translate-y-1/2 flex items-center">
                    <span class="text-3xl rotate-90" style="color:#555">🔫</span>
                </div>
            </div>

            <!-- 子弹 -->
            <div id="bullet" class="bullet"></div>

            <!-- 一号 (目标) -->
            <div id="target-container" class="flex flex-col items-center">
                <div class="character-container" id="target-wrapper">
                    <img id="player-one" 
                         class="character-image" 
                         src="uploaded:0f95b5be287e7d2104ab3a17bd98279f.png-d35ffc68-e2c3-4b98-b88a-cfb7b5df6c0c" 
                         alt="一号 (目标)">
                </div>
                <div class="text-lg font-semibold mt-2">一号 (Target)</div>
            </div>
        </div>

        <!-- 控制按钮 -->
        <div class="flex justify-center mt-8">
            <button id="shoot-button" 
                    class="px-8 py-4 bg-red-600 hover:bg-red-700 text-white font-extrabold text-xl rounded-full shadow-lg transition duration-200 transform hover:scale-105 active:scale-95 disabled:bg-gray-400">
                点击射击 (Shoot!)
            </button>
        </div>

        <!-- 状态和提示 -->
        <p id="status-message" class="text-center mt-4 text-sm text-gray-600">点击按钮发射子弹！</p>
    </div>

    <script>
        // 获取DOM元素
        const shootButton = document.getElementById('shoot-button');
        const bullet = document.getElementById('bullet');
        const playerOne = document.getElementById('player-one');
        const gunMuzzle = document.getElementById('gun-muzzle');
        const scene = document.getElementById('game-scene');
        const statusMessage = document.getElementById('status-message');
        
        // 子弹飞行时间 (需与CSS中的 transition 时间保持一致)
        const BULLET_FLIGHT_TIME = 500; // 毫秒 (0.5s)
        const SHAKE_TIME = 400; // 毫秒 (0.4s)

        // 状态变量，防止连击时动画错乱
        let isShooting = false;

        // 获取二号的枪口位置
        function getMuzzlePosition() {
            const sceneRect = scene.getBoundingClientRect();
            const muzzleRect = gunMuzzle.getBoundingClientRect();
            
            // 计算子弹的起始位置 (相对于场景的左上角)
            // x: 枪口右侧，y: 枪口中心高度
            const x = muzzleRect.left + muzzleRect.width - sceneRect.left;
            const y = muzzleRect.top + muzzleRect.height / 2 - sceneRect.top;
            
            return { x, y };
        }

        // 射击函数
        function shoot() {
            if (isShooting) {
                statusMessage.textContent = '子弹正在飞行中，请稍候...';
                return;
            }
            
            isShooting = true;
            shootButton.disabled = true;
            statusMessage.textContent = '💥 子弹发射！';

            // 1. 设置子弹的初始位置
            const muzzlePos = getMuzzlePosition();
            bullet.style.left = `${muzzlePos.x}px`;
            bullet.style.top = `${muzzlePos.y}px`;
            
            // 确保子弹回到起点并隐藏 (重置动画)
            bullet.classList.remove('bullet-firing');
            playerOne.classList.remove('shake'); 
            
            // 强制浏览器重绘，确保子弹在下一帧开始动画
            void bullet.offsetWidth; 

            // 2. 触发子弹飞行动画
            // 目标距离为整个场景的宽度减去目标物体的宽度，以确保子弹击中目标的大致位置
            // 这里我们直接在 CSS 中使用固定的 translateX(350px) 来模拟
            bullet.classList.add('bullet-firing');

            // 3. 子弹击中目标
            setTimeout(() => {
                // 击中效果：目标摇晃
                playerOne.classList.add('shake');
                statusMessage.textContent = '🎯 击中目标！一号开始摇晃！';

                // 隐藏子弹 (移出屏幕后或设置为透明)
                bullet.classList.remove('bullet-firing');
                bullet.style.opacity = '0'; 

                // 4. 重置状态
                setTimeout(() => {
                    playerOne.classList.remove('shake');
                    shootButton.disabled = false;
                    isShooting = false;
                    statusMessage.textContent = '点击按钮发射子弹！';
                }, SHAKE_TIME);

            }, BULLET_FLIGHT_TIME);
        }

        // 初始化子弹位置
        window.onload = function() {
            // 延迟确保所有元素加载完成
            setTimeout(() => {
                const initialPos = getMuzzlePosition();
                bullet.style.left = `${initialPos.x}px`;
                bullet.style.top = `${initialPos.y}px`;
            }, 100); 
        };
        
        // 绑定射击事件
        shootButton.addEventListener('click', shoot);

        // 监听窗口大小变化以重新定位枪口和子弹（增强响应性）
        window.addEventListener('resize', () => {
            if (!isShooting) {
                const newPos = getMuzzlePosition();
                bullet.style.left = `${newPos.x}px`;
                bullet.style.top = `${newPos.y}px`;
            }
        });
    </script>
</body>
</html>
