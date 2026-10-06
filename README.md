# lottery-emulator
Play the lottery for free


<html lang="zh-TW">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>幸運大樂透對獎模擬器 (動態累積版)</title>
    <style>
        :root {
            --primary-color: #e74c3c;
            --bg-color: #f5f6fa;
            --ball-color: #ffffff;
            --text-color: #2f3640;
            --selected-color: #f1c40f;
            --matched-color: #2ecc71;
            --special-color: #9b59b6;
        }
        body {
            font-family: 'Segoe UI', Tahoma, Geneva, Verdana, sans-serif;
            background-color: var(--bg-color);
            color: var(--text-color);
            display: flex;
            flex-direction: column;
            align-items: center;
            padding: 20px;
            margin: 0;
            min-height: 100vh;
        }
        h1 {
            color: var(--primary-color);
            margin-bottom: 20px;
            text-align: center;
            width: 100%;
        }

        /* 左右佈局主容器 */
        .main-layout {
            display: flex;
            gap: 20px;
            max-width: 1000px;
            width: 100%;
            align-items: flex-start;
        }

        /* 左側：獎金池看板 */
        .prize-board {
            flex: 0 0 280px; 
            background: linear-gradient(135deg, #1e3c72 0%, #2a5298 100%);
            color: white;
            border-radius: 12px;
            padding: 25px 20px;
            box-shadow: 0 4px 15px rgba(0,0,0,0.2);
            box-sizing: border-box;
            position: sticky;
            top: 20px;
            transition: all 0.3s ease;
        }
        .prize-board-title {
            text-align: center;
            font-size: 1.3rem;
            font-weight: bold;
            margin-bottom: 10px;
            color: #f1c40f;
            letter-spacing: 2px;
        }
        .prize-board-subtitle {
            text-align: center;
            font-size: 0.9rem;
            opacity: 0.9;
            margin-bottom: 20px;
            padding-bottom: 15px;
            border-bottom: 1px solid rgba(255,255,255,0.2);
            line-height: 1.6;
        }
        .prize-stats {
            display: flex;
            flex-direction: column;
            gap: 15px;
        }
        .stat-item {
            display: flex;
            justify-content: space-between;
            align-items: center;
            background: rgba(255, 255, 255, 0.1);
            padding: 15px;
            border-radius: 8px;
            border: 1px solid rgba(255,255,255,0.1);
            transition: all 0.3s ease;
        }
        .stat-item.highlight {
            background: rgba(241, 196, 15, 0.15);
            border: 1px solid #f1c40f;
            transform: scale(1.02);
            box-shadow: 0 0 10px rgba(241, 196, 15, 0.2);
        }
        /* 獎金更新時的動畫效果 */
        .stat-item.updating {
            background: rgba(46, 204, 113, 0.4);
            transform: scale(1.05);
        }
        .stat-item span:first-child {
            font-size: 1rem;
            color: #ecf0f1;
        }
        .stat-item span:last-child {
            font-size: 1.3rem;
            font-weight: bold;
            color: #f1c40f;
        }

        /* 右側：主要操作區 */
        .content-area {
            flex: 1;
            display: flex;
            flex-direction: column;
            gap: 20px;
            width: 100%;
        }

        .container {
            background: white;
            padding: 25px;
            border-radius: 12px;
            box-shadow: 0 4px 15px rgba(0,0,0,0.1);
            box-sizing: border-box;
            width: 100%;
        }
        .section-title {
            font-size: 1.2rem;
            font-weight: bold;
            margin-bottom: 15px;
            border-bottom: 2px solid var(--bg-color);
            padding-bottom: 10px;
            display: flex;
            justify-content: space-between;
            align-items: center;
        }
        
        /* 號碼球網格 */
        .grid {
            display: grid;
            grid-template-columns: repeat(7, 1fr);
            gap: 10px;
            margin-bottom: 20px;
            width: 100%;
        }
        .ball {
            aspect-ratio: 1;
            min-height: 45px; 
            border-radius: 50%;
            background-color: var(--ball-color);
            border: 2px solid #ddd;
            display: flex;
            justify-content: center;
            align-items: center;
            font-size: 1.2rem;
            font-weight: bold;
            cursor: pointer;
            transition: all 0.2s ease;
            user-select: none;
            box-sizing: border-box;
        }
        .ball:hover:not(.disabled) {
            transform: scale(1.1);
            border-color: var(--selected-color);
        }
        .ball.selected {
            background-color: var(--selected-color) !important;
            border-color: var(--selected-color) !important;
            color: white;
        }
        .ball.disabled {
            cursor: not-allowed;
            opacity: 0.6;
        }
        .ball.matched {
            background-color: var(--matched-color) !important;
            border-color: var(--matched-color) !important;
            color: white;
            box-shadow: 0 0 10px var(--matched-color);
        }
        .ball.special {
            background-color: var(--special-color) !important;
            border-color: var(--special-color) !important;
            color: white;
            box-shadow: 0 0 10px var(--special-color);
        }

        /* 按鈕區 */
        .controls {
            display: flex;
            gap: 10px;
            margin-bottom: 20px;
            justify-content: center;
        }
        button {
            padding: 10px 20px;
            font-size: 1rem;
            border: none;
            border-radius: 6px;
            cursor: pointer;
            font-weight: bold;
            transition: background 0.2s;
        }
        .btn-action { background-color: #3498db; color: white; flex: 1; }
        .btn-action:hover { background-color: #2980b9; }
        .btn-danger { background-color: #95a5a6; color: white; flex: 1; }
        .btn-danger:hover { background-color: #7f8c8d; }
        .btn-primary { 
            background-color: var(--primary-color); 
            color: white; 
            font-size: 1.2rem; 
            padding: 15px 30px;
            width: 100%;
            letter-spacing: 2px;
        }
        .btn-primary:hover:not(:disabled) { background-color: #c0392b; }
        .btn-primary:disabled { background-color: #bdc3c7; cursor: not-allowed; }
        
        /* 開獎顯示區 */
        #draw-area {
            display: flex;
            justify-content: center;
            gap: 15px;
            min-height: 80px;
            margin-top: 20px;
            flex-wrap: wrap;
        }
        .drawn-ball {
            width: 60px;
            height: 60px;
            border-radius: 50%;
            display: flex;
            justify-content: center;
            align-items: center;
            font-size: 1.5rem;
            font-weight: bold;
            color: white;
            opacity: 0;
            transform: translateY(-20px);
            transition: all 0.5s cubic-bezier(0.175, 0.885, 0.32, 1.275);
        }
        .drawn-ball.show {
            opacity: 1;
            transform: translateY(0);
        }
        .normal-ball { background-color: #e67e22; }
        .special-ball { background-color: var(--special-color); }
        
        #result {
            text-align: center;
            font-size: 1.5rem;
            font-weight: bold;
            margin-top: 20px;
            color: var(--primary-color);
            min-height: 80px;
        }
        .prize-money {
            color: #27ae60;
            font-size: 1.8rem;
            display: block;
            margin-top: 10px;
        }

        /* 響應式設計 */
        @media (max-width: 850px) {
            .main-layout { flex-direction: column; }
            .prize-board { flex: none; width: 100%; position: static; }
            .prize-stats { flex-direction: row; flex-wrap: wrap; }
            .stat-item { flex: 1 1 45%; flex-direction: column; gap: 5px; }
        }
        @media (max-width: 600px) {
            .grid { grid-template-columns: repeat(5, 1fr); }
            .ball { min-height: 50px; }
            .drawn-ball { width: 45px; height: 45px; font-size: 1.2rem; }
            #draw-area { gap: 8px; flex-wrap: wrap; }
            .stat-item { flex: 1 1 100%; }
        }
    </style>
</head>
<body>

    <h1>幸運大樂透對獎模擬器</h1>
    
    <div class="main-layout">
        <!-- 左側：動態累積獎金池看板 -->
        <aside class="prize-board">
            <div class="prize-board-title">💰 累積獎池 💰</div>
            <div class="prize-board-subtitle">
                每期模擬增加銷售額：NT$ 10,000<br>
                目前累積期數：第 <span id="board-rounds">1</span> 期
            </div>
            <div class="prize-stats">
                <div class="stat-item highlight" id="item-head">
                    <span>頭獎 (82%)</span>
                    <span id="val-head">$4,592</span>
                </div>
                <div class="stat-item" id="item-2nd">
                    <span>貳獎 (7%)</span>
                    <span id="val-2nd">$392</span>
                </div>
                <div class="stat-item" id="item-3rd">
                    <span>參獎 (6.5%)</span>
                    <span id="val-3rd">$364</span>
                </div>
                <div class="stat-item" id="item-4th">
                    <span>肆獎 (4.5%)</span>
                    <span id="val-4th">$252</span>
                </div>
            </div>
        </aside>

        <!-- 右側：選號與開獎區 -->
        <main class="content-area">
            <div class="container">
                <div class="section-title">
                    <span>第一步：請選擇 6 個號碼</span>
                    <span style="font-size: 1rem; color: #7f8c8d;">已選 <span id="count">0</span>/6</span>
                </div>
                <div class="controls">
                    <button class="btn-action" id="btn-random">隨機快選</button>
                    <button class="btn-danger" id="btn-clear">清除重選</button>
                </div>
                <div class="grid" id="number-grid"></div>
                <button class="btn-primary" id="btn-draw" disabled>開始開獎！</button>
            </div>

            <div class="container">
                <div class="section-title">開獎結果</div>
                <div id="draw-area">
                    <div style="color: #999; display: flex; align-items: center;">請先完成選號並點擊開始開獎</div>
                </div>
                <div id="result"></div>
            </div>
        </main>
    </div>

    <script>
        document.addEventListener('DOMContentLoaded', () => {
            const MAX_SELECTION = 6;
            let selectedNumbers = [];
            let isDrawing = false;
            
            // 動態獎金池狀態
            let totalRounds = 1;
            let pools = {
                head: 4592,   // 5600 * 82%
                second: 392,  // 5600 * 7% (已修正邏輯)
                third: 364,   // 5600 * 6.5% (已修正邏輯)
                fourth: 252   // 5600 * 4.5%
            };

            const grid = document.getElementById('number-grid');
            const countSpan = document.getElementById('count');
            const btnDraw = document.getElementById('btn-draw');
            const drawArea = document.getElementById('draw-area');
            const resultText = document.getElementById('result');

            // 初始化號碼球
            function initGrid() {
                if (!grid) return;
                grid.innerHTML = '';
                for (let i = 1; i <= 49; i++) {
                    const ball = document.createElement('div');
                    ball.classList.add('ball');
                    ball.textContent = i;
                    ball.dataset.number = i;
                    ball.addEventListener('click', () => toggleNumber(i, ball));
                    grid.appendChild(ball);
                }
            }

            // 更新左側看板數值與動畫
            function updateBoardUI() {
                document.getElementById('board-rounds').textContent = totalRounds;
                document.getElementById('val-head').textContent = '$' + pools.head.toLocaleString();
                document.getElementById('val-2nd').textContent = '$' + pools.second.toLocaleString();
                document.getElementById('val-3rd').textContent = '$' + pools.third.toLocaleString();
                document.getElementById('val-4th').textContent = '$' + pools.fourth.toLocaleString();

                // 加入短暫閃爍動畫提示累積增加
                ['head', '2nd', '3rd', '4th'].forEach(key => {
                    const item = document.getElementById(`item-${key}`);
                    item.classList.add('updating');
                    setTimeout(() => item.classList.remove('updating'), 300);
                });
            }

            function toggleNumber(num, element) {
                if (isDrawing) return;
                const index = selectedNumbers.indexOf(num);
                if (index > -1) {
                    selectedNumbers.splice(index, 1);
                    element.classList.remove('selected');
                } else {
                    if (selectedNumbers.length < MAX_SELECTION) {
                        selectedNumbers.push(num);
                        element.classList.add('selected');
                    }
                }
                updateStatus();
            }

            function updateStatus() {
                countSpan.textContent = selectedNumbers.length;
                document.querySelectorAll('.ball').forEach(ball => {
                    if (selectedNumbers.length >= MAX_SELECTION && !ball.classList.contains('selected')) {
                        ball.classList.add('disabled');
                    } else {
                        ball.classList.remove('disabled');
                    }
                });
                btnDraw.disabled = selectedNumbers.length !== MAX_SELECTION;
            }

            document.getElementById('btn-random').addEventListener('click', () => {
                if (isDrawing) return;
                document.getElementById('btn-clear').click();
                let pool = Array.from({length: 49}, (_, i) => i + 1);
                for (let i = 0; i < MAX_SELECTION; i++) {
                    let randomIndex = Math.floor(Math.random() * pool.length);
                    let num = pool.splice(randomIndex, 1)[0];
                    let ball = document.querySelector(`.ball[data-number="${num}"]`);
                    if (ball) toggleNumber(num, ball);
                }
            });

            document.getElementById('btn-clear').addEventListener('click', () => {
                if (isDrawing) return;
                selectedNumbers = [];
                document.querySelectorAll('.ball').forEach(ball => {
                    ball.classList.remove('selected', 'matched', 'special');
                    ball.style.backgroundColor = ''; 
                    ball.style.borderColor = '';
                });
                drawArea.innerHTML = '<div style="color: #999; display: flex; align-items: center;">請先完成選號並點擊開始開獎</div>';
                resultText.textContent = '';
                updateStatus();
            });

            function generateDrawNumbers() {
                let pool = Array.from({length: 49}, (_, i) => i + 1);
                let drawn = [];
                for (let i = 0; i < 7; i++) {
                    let randomIndex = Math.floor(Math.random() * pool.length);
                    drawn.push(pool.splice(randomIndex, 1)[0]);
                }
                return {
                    normal: drawn.slice(0, 6).sort((a, b) => a - b),
                    special: drawn[6]
                };
            }

            btnDraw.addEventListener('click', () => {
                if (selectedNumbers.length !== MAX_SELECTION) return;
                
                isDrawing = true;
                btnDraw.disabled = true;
                document.getElementById('btn-random').disabled = true;
                document.getElementById('btn-clear').disabled = true;
                resultText.textContent = '開獎中...';
                drawArea.innerHTML = '';

                document.querySelectorAll('.ball.selected').forEach(ball => {
                    ball.classList.remove('matched', 'special');
                });

                const drawData = generateDrawNumbers();
                const allDrawnNumbers = [...drawData.normal, drawData.special];
                
                allDrawnNumbers.forEach((num, index) => {
                    setTimeout(() => {
                        const ballDiv = document.createElement('div');
                        ballDiv.classList.add('drawn-ball');
                        if (index < 6) {
                            ballDiv.classList.add('normal-ball');
                        } else {
                            ballDiv.classList.add('special-ball');
                            const label = document.createElement('div');
                            label.textContent = '+';
                            label.style.color = 'var(--text-color)';
                            label.style.fontSize = '2rem';
                            drawArea.appendChild(label);
                        }
                        ballDiv.textContent = num;
                        drawArea.appendChild(ballDiv);
                        
                        setTimeout(() => ballDiv.classList.add('show'), 50);

                        if (index === 6) {
                            setTimeout(() => checkResult(drawData.normal, drawData.special), 800);
                        }
                    }, index * 600); 
                });
            });

            function checkResult(normalDrawn, specialDrawn) {
                let matchCount = 0;
                let specialMatch = false;

                selectedNumbers.forEach(num => {
                    const ballEl = document.querySelector(`.ball[data-number="${num}"]`);
                    if (normalDrawn.includes(num)) {
                        matchCount++;
                        if(ballEl) ballEl.classList.add('matched'); 
                    } else if (num === specialDrawn) {
                        specialMatch = true;
                        if(ballEl) ballEl.classList.add('special'); 
                    }
                });

                let prizeText = `你對中了 ${matchCount} 個一般號碼`;
                if (specialMatch) prizeText += ` + 特別號`;

                let resultMsg = "";
                let prizeMoney = 0;

                // 檢查是否中累積獎，若中獎則將該獎金發出，並將該獎池歸零
                if (matchCount === 6) {
                    resultMsg = "🎉 狂賀！抱走頭獎！";
                    prizeMoney = pools.head;
                    pools.head = 0; 
                } else if (matchCount === 5 && specialMatch) {
                    resultMsg = "🎊 貳獎！";
                    prizeMoney = pools.second;
                    pools.second = 0;
                } else if (matchCount === 5) {
                    resultMsg = "💰 參獎！";
                    prizeMoney = pools.third;
                    pools.third = 0;
                } else if (matchCount === 4 && specialMatch) {
                    resultMsg = "💵 肆獎！";
                    prizeMoney = pools.fourth;
                    pools.fourth = 0;
                } 
                // 固定獎金區
                else if (matchCount === 4) {
                    resultMsg = "💵 伍獎！";
                    prizeMoney = 2000;
                } else if (matchCount === 3 && specialMatch) {
                    resultMsg = "👍 陸獎！";
                    prizeMoney = 1000;
                } else if (matchCount === 2 && specialMatch) {
                    resultMsg = "👍 柒獎！";
                    prizeMoney = 400;
                } else if (matchCount === 3) {
                    resultMsg = "👍 普獎！";
                    prizeMoney = 400;
                } else {
                    resultMsg = "😭 沒中獎，獎金滾入下一期！";
                    prizeMoney = 0;
                }

                let finalHTML = `${prizeText}<br><span style="color: #e74c3c;">${resultMsg}</span>`;
                if (prizeMoney > 0) {
                    finalHTML += `<span class="prize-money">獲得獎金：NT$ ${prizeMoney.toLocaleString()}</span>`;
                }
                
                resultText.innerHTML = finalHTML;
                
                // 結算完畢後，為「下一期」注入新的銷售額獎金
                totalRounds++;
                pools.head += 4592;
                pools.second += 392;
                pools.third += 364;
                pools.fourth += 252;
                
                // 更新畫面顯示新的累積獎金
                updateBoardUI();

                isDrawing = false;
                document.getElementById('btn-random').disabled = false;
                document.getElementById('btn-clear').disabled = false;
            }

            initGrid();
        });
    </script>
</body>
</html>
