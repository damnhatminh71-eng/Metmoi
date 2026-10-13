<!DOCTYPE html>
<html lang="vi">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Super Mario Mini</title>
    <style>
        body {
            margin: 0;
            padding: 0;
            background-color: #333;
            display: flex;
            justify-content: center;
            align-items: center;
            height: 100vh;
            font-family: Arial, sans-serif;
            color: white;
            flex-direction: column;
        }

        #game-container {
            position: relative;
            width: 800px;
            height: 400px;
            background-color: #5c94fc; /* Màu bầu trời Mario */
            overflow: hidden;
            border: 4px solid #fff;
            box-shadow: 0 10px 20px rgba(0,0,0,0.5);
        }

        /* Mặt đất */
        .ground {
            position: absolute;
            bottom: 0;
            width: 100%;
            height: 50px;
            background-color: #c84c0c;
            border-top: 4px solid #fcb438;
        }

        /* Mario */
        #mario {
            position: absolute;
            width: 32px;
            height: 40px;
            background-color: #e52521; /* Áo đỏ */
            bottom: 50px;
            left: 50px;
            border-radius: 4px;
        }

        /* Mũ Mario */
        #mario::before {
            content: '';
            position: absolute;
            top: 0;
            width: 100%;
            height: 12px;
            background-color: #e52521;
        }

        /* Quần Mario */
        #mario::after {
            content: '';
            position: absolute;
            bottom: 0;
            width: 100%;
            height: 20px;
            background-color: #0020c2; /* Quần xanh */
        }

        /* Nấm Goomba */
        .goomba {
            position: absolute;
            width: 32px;
            height: 32px;
            background-color: #a84400;
            bottom: 50px;
            border-radius: 50% 50% 0 0;
        }

        /* Khối gạch */
        .block {
            position: absolute;
            width: 32px;
            height: 32px;
            background-color: #fcb438;
            border: 2px solid #a84400;
            box-sizing: border-box;
        }

        /* Đồng xu */
        .coin {
            position: absolute;
            width: 20px;
            height: 24px;
            background-color: #f8d820;
            border-radius: 50%;
            border: 2px solid #d89000;
            box-sizing: border-box;
            animation: bounce 0.6s infinite alternate;
        }

        @keyframes bounce {
            from { transform: translateY(0); }
            to { transform: translateY(-5px); }
        }

        /* Điểm số & Bảng thông báo */
        #score-board {
            position: absolute;
            top: 10px;
            left: 10px;
            font-size: 20px;
            font-weight: bold;
            color: #fff;
            text-shadow: 2px 2px #000;
        }

        #game-over {
            display: none;
            position: absolute;
            top: 50%;
            left: 50%;
            transform: translate(-50%, -50%);
            font-size: 36px;
            color: red;
            text-shadow: 2px 2px #000;
            text-align: center;
        }
    </style>
</head>
<body>

    <h2 style="margin-bottom: 10px;">Super Mario HTML5 Canvas/DOM</h2>
    <p style="margin-top: 0;">Sử dụng phím ⬅️ ➡️ để di chuyển, ⬆️ hoặc Space để nhảy</p>

    <div id="game-container">
        <div id="score-board">ĐIỂM: <span id="score">0</span></div>
        <div id="game-over">GAME OVER<br><span style="font-size: 18px; color: white;">F5 để chơi lại</span></div>
        <div id="mario"></div>
        <div class="ground"></div>
    </div>

    <script>
        const mario = document.getElementById('mario');
        const container = document.getElementById('game-container');
        const scoreEl = document.getElementById('score');
        const gameOverEl = document.getElementById('game-over');

        // Thông số vật lý
        let marioX = 50;
        let marioY = 50; // Khoảng cách so với đáy
        let velocityY = 0;
        let isJumping = false;
        let score = 0;
        let isGameOver = false;

        const gravity = 1.2;
        const jumpPower = 18;
        const speed = 5;

        // Trạng thái phím bấm
        const keys = {
            ArrowLeft: false,
            ArrowRight: false,
            ArrowUp: false,
            Space: false
        };

        // Danh sách vật thể trong game
        const blocks = [
            { x: 200, y: 150 },
            { x: 232, y: 150 },
            { x: 264, y: 150 },
            { x: 450, y: 200 },
            { x: 482, y: 200 }
        ];

        const coins = [
            { x: 200, y: 190, collected: false },
            { x: 232, y: 190, collected: false },
            { x: 264, y: 190, collected: false },
            { x: 466, y: 240, collected: false }
        ];

        let goombas = [
            { x: 400, y: 50, speed: -2, element: null },
            { x: 700, y: 50, speed: -2.5, element: null }
        ];

        // Khởi tạo bản đồ
        function initMap() {
            // Tạo khối gạch
            blocks.forEach(b => {
                const block = document.createElement('div');
                block.className = 'block';
                block.style.left = b.x + 'px';
                block.style.bottom = b.y + 'px';
                container.appendChild(block);
            });

            // Tạo đồng xu
            coins.forEach((c, index) => {
                const coin = document.createElement('div');
                coin.className = 'coin';
                coin.id = 'coin-' + index;
                coin.style.left = c.x + 'px';
                coin.style.bottom = c.y + 'px';
                container.appendChild(coin);
            });

            // Tạo Goomba
            goombas.forEach(g => {
                const goomba = document.createElement('div');
                goomba.className = 'goomba';
                container.appendChild(goomba);
                g.element = goomba;
            });
        }

        // Bắt sự kiện phím
        window.addEventListener('keydown', (e) => {
            if (e.code === 'Space') keys.Space = true;
            if (e.key === 'ArrowLeft') keys.ArrowLeft = true;
            if (e.key === 'ArrowRight') keys.ArrowRight = true;
            if (e.key === 'ArrowUp') keys.ArrowUp = true;
        });

        window.addEventListener('keyup', (e) => {
            if (e.code === 'Space') keys.Space = false;
            if (e.key === 'ArrowLeft') keys.ArrowLeft = false;
            if (e.key === 'ArrowRight') keys.ArrowRight = false;
            if (e.key === 'ArrowUp') keys.ArrowUp = false;
        });

        // Vòng lặp chính của trò chơi (Game Loop)
        function gameLoop() {
            if (isGameOver) return;

            // 1. Di chuyển ngang
            if (keys.ArrowLeft && marioX > 0) {
                marioX -= speed;
            }
            if (keys.ArrowRight && marioX < 800 - 32) {
                marioX += speed;
            }

            // 2. Nhảy
            if ((keys.ArrowUp || keys.Space) && !isJumping) {
                velocityY = jumpPower;
                isJumping = true;
            }

            // 3. Áp dụng trọng lực & Di chuyển dọc
            marioY += velocityY;
            velocityY -= gravity;

            // Kiểm tra va chạm mặt đất
            if (marioY <= 50) {
                marioY = 50;
                isJumping = false;
                velocityY = 0;
            }

            // Kiểm tra va chạm gạch (đứng lên gạch)
            blocks.forEach(b => {
                if (marioX + 32 > b.x && marioX < b.x + 32) {
                    // Đứng lên gạch
                    if (marioY <= b.y + 32 && marioY + velocityY >= b.y + 32 && velocityY <= 0) {
                        marioY = b.y + 32;
                        isJumping = false;
                        velocityY = 0;
                    }
                }
            });

            // 4. Nhặt đồng xu
            coins.forEach((c, index) => {
                if (!c.collected) {
                    if (Math.abs(marioX - c.x) < 25 && Math.abs(marioY - c.y) < 25) {
                        c.collected = true;
                        document.getElementById('coin-' + index).style.display = 'none';
                        score += 100;
                        scoreEl.innerText = score;
                    }
                }
            });

            // 5. Di chuyển & Va chạm Goomba
            goombas.forEach(g => {
                g.x += g.speed;
                if (g.x <= 0 || g.x >= 768) g.speed *= -1; // Đổi chiều khi đụng tường
                g.element.style.left = g.x + 'px';
                g.element.style.bottom = g.y + 'px';

                // Kiểm tra va chạm với Mario
                if (Math.abs(marioX - g.x) < 25 && Math.abs(marioY - g.y) < 25) {
                    // Nếu Mario nhảy đè lên đầu Goomba
                    if (velocityY < 0 && marioY > g.y + 15) {
                        g.x = -100; // Xóa Goomba
                        velocityY = 10; // Đẩy Mario nảy lên
                        score += 200;
                        scoreEl.innerText = score;
                    } else {
                        // Mario thua
                        isGameOver = true;
                        gameOverEl.style.display = 'block';
                    }
                }
            });

            // Cập nhật vị trí Mario trên màn hình
            mario.style.left = marioX + 'px';
            mario.style.bottom = marioY + 'px';

            requestAnimationFrame(gameLoop);
        }

        // Bắt đầu game
        initMap();
        gameLoop();
    </script>
</body>
</html># Metmoi
Quá mệt
