<!DOCTYPE html>
<html lang="ru">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Соня Defense</title>
    <style>
        * { margin: 0; padding: 0; box-sizing: border-box; }

        body {
            overflow: hidden;
            font-family: 'Arial', sans-serif;
            background: #000;
            color: #fff;
            user-select: none;
        }

        /* ============ HUD ============ */
        #hud {
            position: absolute; top: 20px; left: 20px;
            color: white; text-shadow: 2px 2px 6px #000;
            z-index: 10; display: none;
            padding: 15px 20px;
            background: rgba(0,0,0,0.35);
            border-left: 4px solid #66aaff;
            border-radius: 8px;
            backdrop-filter: blur(8px);
            min-width: 220px;
            transition: border-color 0.5s;
        }
        #health { font-size: 22px; font-weight: 700; }
        #health-bar {
            width: 200px; height: 8px;
            background: rgba(0,0,0,0.5);
            border-radius: 4px; margin-top: 6px;
            overflow: hidden;
        }
        #health-fill {
            height: 100%; width: 100%;
            background: linear-gradient(90deg, #ff3333, #ff6666);
            transition: width 0.3s;
            border-radius: 4px;
        }
        #score { font-size: 16px; margin-top: 12px; color: #ffdd44; }
        #coins { font-size: 16px; margin-top: 6px; color: #ffcc00; }
        #wave-info { font-size: 13px; margin-top: 8px; color: #88ccff; }
        #time-info {
            font-size: 14px; margin-top: 8px;
            color: #ffdd88; font-weight: 700;
            transition: color 0.5s;
        }

        /* ============ CROSSHAIR ============ */
        #crosshair {
            position: absolute; top: 50%; left: 50%;
            width: 22px; height: 22px;
            transform: translate(-50%, -50%);
            pointer-events: none; z-index: 10; display: none;
        }
        #crosshair::before, #crosshair::after {
            content: ''; position: absolute; background: rgba(255,255,255,0.9);
            box-shadow: 0 0 4px rgba(0,0,0,0.9);
        }
        #crosshair::before { width: 2px; height: 100%; left: 50%; transform: translateX(-50%); }
        #crosshair::after  { width: 100%; height: 2px; top: 50%; transform: translateY(-50%); }
        #crosshair-dot {
            position: absolute; top: 50%; left: 50%;
            width: 3px; height: 3px;
            background: #ff3333; border-radius: 50%;
            transform: translate(-50%, -50%);
            box-shadow: 0 0 6px #ff3333;
        }

        /* ============ DAMAGE FLASH ============ */
        #damage-flash {
            position: absolute; inset: 0;
            background: radial-gradient(circle, transparent 40%, rgba(255,0,0,0.6) 100%);
            pointer-events: none; z-index: 15;
            opacity: 0; transition: opacity 0.15s;
        }

        /* ============ NIGHT OVERLAY ============ */
        #night-overlay {
            position: absolute; inset: 0;
            pointer-events: none; z-index: 5;
            background: radial-gradient(circle at center, transparent 20%, rgba(0,0,20,0.55) 100%);
            opacity: 0; transition: opacity 1s;
        }

        /* ============ GAME OVER ============ */
        #gameover {
            display: none; position: absolute; top: 50%; left: 50%;
            transform: translate(-50%, -50%);
            color: #ff3333; font-size: 56px; font-weight: 900;
            text-shadow: 0 0 30px rgba(255,0,0,0.8), 3px 3px 6px #000;
            z-index: 20;
            background: rgba(0,0,0,0.9);
            padding: 40px 70px;
            border-radius: 15px; text-align: center;
            border: 2px solid #ff3333;
            backdrop-filter: blur(10px);
            letter-spacing: 4px;
            animation: gameover-in 0.5s ease-out;
        }
        @keyframes gameover-in {
            from { transform: translate(-50%, -50%) scale(0.5); opacity: 0; }
            to   { transform: translate(-50%, -50%) scale(1); opacity: 1; }
        }

        #controls {
            position: absolute; bottom: 15px; left: 50%;
            transform: translateX(-50%);
            color: #99aabb; font-size: 12px; z-index: 10;
            display: none;
            background: rgba(0,0,0,0.5);
            padding: 8px 16px; border-radius: 8px;
            letter-spacing: 1px;
        }

        /* ============ ГЛАВНОЕ МЕНЮ ============ */
        #main-menu {
            position: absolute; inset: 0;
            display: flex; flex-direction: column;
            justify-content: center; align-items: center;
            z-index: 100;
            background:
                radial-gradient(ellipse at center, #1a2a4a 0%, #0a0a15 70%, #000 100%);
            overflow: hidden;
        }
        #main-menu::before {
            content: '';
            position: absolute; inset: -50%;
            background:
                radial-gradient(circle at 20% 30%, rgba(255,50,50,0.15) 0%, transparent 40%),
                radial-gradient(circle at 80% 70%, rgba(50,100,255,0.15) 0%, transparent 40%),
                radial-gradient(circle at 50% 50%, rgba(150,50,255,0.1) 0%, transparent 50%);
            animation: rotate-bg 20s linear infinite;
            pointer-events: none;
        }
        @keyframes rotate-bg {
            from { transform: rotate(0deg); }
            to   { transform: rotate(360deg); }
        }

        .menu-title {
            font-size: 78px; font-weight: 900;
            letter-spacing: 6px; margin-bottom: 8px;
            background: linear-gradient(135deg, #ff4444 0%, #ffaa00 50%, #44ff88 100%);
            -webkit-background-clip: text; -webkit-text-fill-color: transparent;
            background-clip: text;
            text-shadow: 0 0 40px rgba(255,100,0,0.3);
            z-index: 1;
            animation: pulse-title 3s ease-in-out infinite;
        }
        @keyframes pulse-title {
            0%, 100% { filter: drop-shadow(0 0 10px rgba(255,100,0,0.5)); }
            50%      { filter: drop-shadow(0 0 25px rgba(255,100,0,0.7)); }
        }
        .menu-subtitle {
            font-size: 14px; color: #8899aa;
            letter-spacing: 8px; text-transform: uppercase;
            margin-bottom: 60px; z-index: 1;
        }
        .menu-buttons {
            display: flex; flex-direction: column;
            gap: 16px; z-index: 1;
        }
        .menu-btn {
            width: 320px; padding: 18px 30px;
            font-size: 20px; font-weight: 700;
            letter-spacing: 3px; text-transform: uppercase;
            color: #fff;
            background: linear-gradient(135deg, rgba(30,40,60,0.9), rgba(15,20,35,0.9));
            border: 2px solid rgba(100,150,255,0.4);
            border-radius: 12px; cursor: pointer;
            transition: all 0.25s ease;
            position: relative; overflow: hidden;
            backdrop-filter: blur(10px);
            box-shadow: 0 4px 20px rgba(0,0,0,0.5);
            font-family: inherit;
        }
        .menu-btn::before {
            content: ''; position: absolute;
            top: 0; left: -100%; width: 100%; height: 100%;
            background: linear-gradient(90deg, transparent, rgba(255,255,255,0.15), transparent);
            transition: left 0.5s;
        }
        .menu-btn:hover::before { left: 100%; }
        .menu-btn:hover {
            transform: translateY(-3px) scale(1.03);
            border-color: rgba(100,200,255,0.9);
            box-shadow: 0 8px 30px rgba(50,150,255,0.5), 0 0 20px rgba(50,150,255,0.3) inset;
            color: #aaddff;
        }
        .menu-btn:active { transform: translateY(0) scale(0.98); }
        .menu-btn.play {
            border-color: rgba(80,255,120,0.5);
            background: linear-gradient(135deg, rgba(20,60,35,0.9), rgba(10,30,20,0.9));
        }
        .menu-btn.play:hover {
            border-color: rgba(80,255,120,1);
            box-shadow: 0 8px 30px rgba(80,255,120,0.5), 0 0 20px rgba(80,255,120,0.3) inset;
            color: #aaffbb;
        }
        .menu-btn.shop {
            border-color: rgba(255,200,50,0.5);
            background: linear-gradient(135deg, rgba(60,45,15,0.9), rgba(30,22,8,0.9));
        }
        .menu-btn.shop:hover {
            border-color: rgba(255,200,50,1);
            box-shadow: 0 8px 30px rgba(255,200,50,0.5), 0 0 20px rgba(255,200,50,0.3) inset;
            color: #ffdd88;
        }
        .menu-footer {
            position: absolute; bottom: 20px;
            color: #445566; font-size: 12px;
            letter-spacing: 2px; z-index: 1;
        }

        /* ============ ПАНЕЛИ ============ */
        .panel {
            position: absolute; inset: 0;
            display: none; flex-direction: column;
            justify-content: center; align-items: center;
            z-index: 110;
            background: radial-gradient(ellipse at center, #1a2a4a 0%, #0a0a15 70%, #000 100%);
            padding: 40px;
        }
        .panel h2 {
            font-size: 48px; margin-bottom: 30px;
            background: linear-gradient(135deg, #66ccff, #aa66ff);
            -webkit-background-clip: text; -webkit-text-fill-color: transparent;
            background-clip: text; letter-spacing: 4px;
        }

        .panel-header {
            position: relative;
            width: 100%; max-width: 700px;
            display: flex; align-items: center;
            justify-content: center;
            margin-bottom: 30px;
        }
        .panel-header h2 {
            margin-bottom: 0;
            text-align: center;
            flex: 1;
        }
        #shop-coins-badge {
            position: absolute;
            right: 0;
            top: 50%;
            transform: translateY(-50%);
            display: flex;
            align-items: center;
            gap: 8px;
            padding: 12px 22px;
            background: linear-gradient(135deg, rgba(255,200,50,0.25), rgba(255,140,0,0.25));
            border: 2px solid rgba(255,200,50,0.8);
            border-radius: 30px;
            color: #ffdd44;
            font-size: 20px;
            font-weight: 900;
            letter-spacing: 1px;
            backdrop-filter: blur(10px);
            box-shadow: 0 0 25px rgba(255,200,50,0.5),
                        0 0 15px rgba(255,200,50,0.3) inset;
            text-shadow: 0 0 10px rgba(255,200,50,0.8);
            animation: coin-badge-pulse 2.5s ease-in-out infinite;
            white-space: nowrap;
        }
        #shop-coins-badge .coin-icon {
            font-size: 22px;
            filter: drop-shadow(0 0 6px rgba(255,200,50,0.9));
        }
        @keyframes coin-badge-pulse {
            0%, 100% {
                box-shadow: 0 0 25px rgba(255,200,50,0.5),
                            0 0 15px rgba(255,200,50,0.3) inset;
            }
            50% {
                box-shadow: 0 0 40px rgba(255,200,50,0.9),
                            0 0 25px rgba(255,200,50,0.5) inset;
            }
        }
        #shop-coins-badge.flash {
            animation: coin-flash 0.5s ease-out;
        }
        @keyframes coin-flash {
            0%   { transform: translateY(-50%) scale(1); }
            40%  { transform: translateY(-50%) scale(1.2); color: #ffffff; }
            100% { transform: translateY(-50%) scale(1); }
        }

        .panel-content {
            width: 100%; max-width: 700px;
            max-height: 70vh; overflow-y: auto;
            background: rgba(20,25,40,0.7);
            border: 2px solid rgba(100,150,255,0.3);
            border-radius: 15px; padding: 25px;
            backdrop-filter: blur(10px);
            box-shadow: 0 10px 40px rgba(0,0,0,0.5);
        }
        .panel-content::-webkit-scrollbar { width: 8px; }
        .panel-content::-webkit-scrollbar-track { background: rgba(0,0,0,0.3); border-radius: 4px; }
        .panel-content::-webkit-scrollbar-thumb { background: rgba(100,150,255,0.5); border-radius: 4px; }

        .shop-item {
            display: flex; justify-content: space-between; align-items: center;
            padding: 15px 20px; margin-bottom: 12px;
            background: rgba(30,40,60,0.6);
            border: 1px solid rgba(100,150,255,0.2);
            border-radius: 10px; transition: all 0.2s;
        }
        .shop-item:hover {
            background: rgba(40,55,80,0.8);
            border-color: rgba(100,200,255,0.5);
            transform: translateX(5px);
        }
        .shop-item.active-skin {
            border-color: #66ff88;
            background: rgba(30,70,45,0.7);
            box-shadow: 0 0 20px rgba(80,255,120,0.4);
        }
        .shop-item-info h3 { font-size: 17px; margin-bottom: 4px; color: #eef; }
        .shop-item-info p  { font-size: 13px; color: #889; }

        .skin-preview {
            width: 42px; height: 42px;
            border-radius: 50%;
            margin-right: 15px;
            flex-shrink: 0;
            box-shadow: 0 0 15px currentColor, inset -6px -6px 12px rgba(0,0,0,0.5);
            position: relative;
        }
        .skin-preview::after {
            content: '';
            position: absolute;
            top: 8px; left: 10px;
            width: 10px; height: 10px;
            background: rgba(255,255,255,0.5);
            border-radius: 50%;
            filter: blur(3px);
        }

        .buy-btn {
            padding: 10px 20px;
            background: linear-gradient(135deg, #ffaa00, #ff6600);
            border: none; border-radius: 8px;
            color: #fff; font-weight: 700; letter-spacing: 1px;
            cursor: pointer; transition: all 0.2s;
            box-shadow: 0 4px 15px rgba(255,120,0,0.4);
            white-space: nowrap; font-family: inherit;
        }
        .buy-btn:hover { transform: scale(1.05); box-shadow: 0 6px 20px rgba(255,120,0,0.7); }
        .buy-btn:active { transform: scale(0.97); }
        .buy-btn.bought { background: #2a4a2a; color: #6f6; cursor: default; box-shadow: none; }
        .buy-btn.bought:hover { transform: none; }
        .buy-btn.equipped {
            background: linear-gradient(135deg, #2a5aaa, #1a3a7a);
            color: #aaddff;
            box-shadow: 0 0 15px rgba(100,150,255,0.6);
        }
        .buy-btn.equipped:hover { transform: none; }

        .setting-row {
            display: flex; justify-content: space-between; align-items: center;
            padding: 16px 5px;
            border-bottom: 1px solid rgba(100,150,255,0.15);
            gap: 15px;
        }
        .setting-row:last-child { border-bottom: none; }
        .setting-row label { font-size: 15px; color: #ccd; min-width: 180px; }
        .setting-row input[type="range"] {
            flex: 1; accent-color: #66aaff; cursor: pointer;
        }
        .setting-value {
            color: #88aaff; font-weight: 700;
            min-width: 80px; text-align: right; font-size: 14px;
        }

        .quality-btns {
            display: flex; gap: 8px; flex: 1; justify-content: flex-end;
        }
        .quality-btn {
            padding: 8px 14px; border-radius: 8px;
            background: rgba(30,40,60,0.8);
            border: 2px solid rgba(100,150,255,0.3);
            color: #99aacc; font-size: 13px; font-weight: 700;
            cursor: pointer; transition: all 0.2s;
            font-family: inherit;
        }
        .quality-btn.active {
            background: linear-gradient(135deg, #2a5aaa, #1a3a7a);
            border-color: #66aaff; color: #fff;
            box-shadow: 0 0 15px rgba(100,150,255,0.5);
        }
        .quality-btn:hover { border-color: #66aaff; color: #fff; }

        .toggle {
            position: relative; width: 50px; height: 26px;
            background: rgba(30,40,60,0.8);
            border-radius: 13px; cursor: pointer;
            transition: background 0.2s; flex-shrink: 0;
            border: 2px solid rgba(100,150,255,0.3);
        }
        .toggle::after {
            content: ''; position: absolute;
            top: 2px; left: 2px;
            width: 18px; height: 18px;
            background: #667; border-radius: 50%;
            transition: all 0.2s;
        }
        .toggle.on { background: rgba(80,150,255,0.4); border-color: #66aaff; }
        .toggle.on::after { left: 26px; background: #66aaff; box-shadow: 0 0 10px #66aaff; }

        .back-btn {
            margin-top: 25px; padding: 14px 40px;
            font-size: 15px; font-weight: 700;
            letter-spacing: 2px; text-transform: uppercase;
            background: linear-gradient(135deg, rgba(30,40,60,0.9), rgba(15,20,35,0.9));
            border: 2px solid rgba(150,150,200,0.5);
            border-radius: 10px; color: #ccd;
            cursor: pointer; transition: all 0.2s;
            font-family: inherit;
        }
        .back-btn:hover {
            border-color: #aaddff; color: #fff;
            transform: translateY(-2px);
            box-shadow: 0 6px 20px rgba(100,150,255,0.4);
        }

        /* ============ ПАУЗА ============ */
        #pause-menu {
            position: absolute; inset: 0;
            display: none; flex-direction: column;
            justify-content: center; align-items: center;
            z-index: 90;
            background: rgba(0,0,0,0.75);
            backdrop-filter: blur(6px);
        }
        #pause-menu h2 {
            font-size: 56px; color: #fff; margin-bottom: 40px;
            letter-spacing: 6px; text-shadow: 0 0 30px rgba(100,150,255,0.8);
        }

        /* ============ KILL FEED ============ */
        #kill-feed {
            position: absolute; top: 20px; right: 20px;
            z-index: 10; display: none;
            flex-direction: column; gap: 6px;
            max-width: 250px;
        }
        .kill-msg {
            background: rgba(0,0,0,0.6);
            border-left: 3px solid #ff3333;
            padding: 8px 12px; border-radius: 5px;
            color: #ffcccc; font-size: 13px;
            animation: kill-in 0.3s ease-out;
            backdrop-filter: blur(4px);
        }
        @keyframes kill-in {
            from { transform: translateX(50px); opacity: 0; }
            to   { transform: translateX(0); opacity: 1; }
        }

        /* ============ TIME NOTIFICATION ============ */
        #time-notification {
            position: absolute; top: 20%; left: 50%;
            transform: translate(-50%, -50%) scale(0.8);
            font-size: 52px; font-weight: 900;
            letter-spacing: 8px;
            color: #fff;
            text-shadow: 0 0 40px currentColor, 4px 4px 8px #000;
            z-index: 12; display: none;
            transition: opacity 0.5s;
            pointer-events: none;
        }

        /* ============ LOADING ============ */
        #loading {
            position: absolute; inset: 0;
            display: flex; flex-direction: column;
            justify-content: center; align-items: center;
            z-index: 200; background: #000;
            transition: opacity 0.5s;
        }
        #loading.hidden { opacity: 0; pointer-events: none; }
        .loader {
            width: 60px; height: 60px;
            border: 4px solid rgba(100,150,255,0.2);
            border-top-color: #66aaff;
            border-radius: 50%;
            animation: spin 1s linear infinite;
        }
        @keyframes spin { to { transform: rotate(360deg); } }
        #loading p {
            margin-top: 20px; color: #66aaff;
            letter-spacing: 4px; font-size: 14px;
        }

        @media (max-width: 700px) {
            .panel-header { flex-direction: column; gap: 15px; }
            #shop-coins-badge { position: static; transform: none; }
            .panel h2 { margin-bottom: 0; }
        }

        @media (max-width: 500px) {
            .menu-title { font-size: 42px; letter-spacing: 2px; }
            .menu-btn { width: 260px; font-size: 17px; padding: 14px 20px; }
            .panel h2 { font-size: 32px; }
            .setting-row label { min-width: auto; font-size: 13px; }
            #time-notification { font-size: 32px; letter-spacing: 4px; }
            .skin-preview { width: 32px; height: 32px; }
            #shop-coins-badge { font-size: 16px; padding: 10px 16px; }
        }
    </style>
    <script src="https://cdnjs.cloudflare.com/ajax/libs/three.js/r128/three.min.js"></script>
</head>
<body>

    <div id="loading">
        <div class="loader"></div>
        <p>ЗАГРУЗКА...</p>
    </div>

    <div id="main-menu">
        <h1 class="menu-title">СОНЯ DEFENSE</h1>
        <p class="menu-subtitle">Выживи любой ценой</p>
        <div class="menu-buttons">
            <button class="menu-btn play" id="btn-play">▶ Играть</button>
            <button class="menu-btn shop" id="btn-shop">🛒 Магазин</button>
            <button class="menu-btn"      id="btn-settings">⚙ Настройки</button>
        </div>
        <div class="menu-footer">v2.7 • Fullscreen Lock • Made with Three.js</div>
    </div>

    <div class="panel" id="shop-panel">
        <div class="panel-header">
            <h2>🛒 СКИНЫ ПАТРОНОВ</h2>
            <div id="shop-coins-badge">
                <span class="coin-icon">💰</span>
                <span id="shop-coins-value">0</span>
            </div>
        </div>
        <div class="panel-content" id="shop-content"></div>
        <button class="back-btn" id="shop-back">← Назад</button>
    </div>

    <div class="panel" id="settings-panel">
        <h2>⚙ НАСТРОЙКИ</h2>
        <div class="panel-content">
            <div class="setting-row">
                <label>🎯 Чувствительность мыши</label>
                <input type="range" id="sens" min="0.3" max="3" step="0.1" value="1">
                <span class="setting-value" id="sens-val">1.0</span>
            </div>
            <div class="setting-row">
                <label>🔊 Громкость звука</label>
                <input type="range" id="vol" min="0" max="100" step="1" value="70">
                <span class="setting-value" id="vol-val">70</span>
            </div>
            <div class="setting-row">
                <label>🎨 Качество графики</label>
                <div class="quality-btns">
                    <button class="quality-btn" data-q="0">Низкое</button>
                    <button class="quality-btn active" data-q="1">Среднее</button>
                    <button class="quality-btn" data-q="2">Высокое</button>
                </div>
            </div>
            <div class="setting-row">
                <label>🌫️ Туман / Дальность</label>
                <input type="range" id="fog" min="0" max="100" step="1" value="50">
                <span class="setting-value" id="fog-val">50</span>
            </div>
            <div class="setting-row">
                <label>🌿 Детализация мира</label>
                <div class="toggle on" id="toggle-details"></div>
            </div>
            <div class="setting-row">
                <label>💡 Динамическое освещение</label>
                <div class="toggle on" id="toggle-lights"></div>
            </div>
            <div class="setting-row">
                <label>🖱️ Инверсия мыши (Y)</label>
                <div class="toggle" id="toggle-invert"></div>
            </div>
            <div class="setting-row">
                <label>📳 Тряска камеры</label>
                <div class="toggle on" id="toggle-shake"></div>
            </div>
            <div class="setting-row">
                <label>🌗 Цикл дня и ночи</label>
                <div class="toggle on" id="toggle-daynight"></div>
            </div>
        </div>
        <button class="back-btn" id="settings-back">← Назад</button>
    </div>

    <div id="pause-menu">
        <h2>ПАУЗА</h2>
        <div class="menu-buttons">
            <button class="menu-btn play" id="btn-resume">▶ Продолжить</button>
            <button class="menu-btn"      id="btn-to-menu">🏠 В главное меню</button>
        </div>
    </div>

    <div id="hud">
        <div id="health">❤️ 100 / 100</div>
        <div id="health-bar"><div id="health-fill"></div></div>
        <div id="score">💀 Убито: 0</div>
        <div id="coins">💰 Монеты: 0</div>
        <div id="wave-info">🌊 Волна: 1</div>
        <div id="time-info">☀️ День</div>
    </div>
    <div id="night-overlay"></div>
    <div id="crosshair"><div id="crosshair-dot"></div></div>
    <div id="damage-flash"></div>
    <div id="kill-feed"></div>
    <div id="time-notification"></div>
    <div id="gameover">ТЫ ПОГИБ!<br><span style="font-size:22px; color:white; letter-spacing:2px;">Обнови страницу для рестарта</span></div>
    <div id="controls">WASD — движение • Мышь — обзор • ЛКМ (зажать) — авто-стрельба • ESC — пауза</div>

    <script>
        // ==================== ПОЛНОЭКРАННЫЙ РЕЖИМ ====================
        let fullscreenLocked = false;

        function enterFullscreen() {
            const el = document.documentElement;
            const req = el.requestFullscreen ||
                        el.webkitRequestFullscreen ||
                        el.mozRequestFullScreen ||
                        el.msRequestFullscreen;
            if (req) {
                req.call(el).then(() => {
                    fullscreenLocked = true;
                }).catch(err => {
                    console.warn('Fullscreen отклонён:', err);
                });
            }
        }

        function exitFullscreen() {
            const exit = document.exitFullscreen ||
                         document.webkitExitFullscreen ||
                         document.mozCancelFullScreen ||
                         document.msExitFullscreen;
            if (exit && document.fullscreenElement) {
                exit.call(document);
            }
        }

        function handleFullscreenChange() {
            const isFs = !!(document.fullscreenElement ||
                            document.webkitFullscreenElement ||
                            document.mozFullScreenElement ||
                            document.msFullscreenElement);

            if (!isFs && fullscreenLocked) {
                setTimeout(() => {
                    enterFullscreen();
                }, 100);
            }
        }

        ['fullscreenchange', 'webkitfullscreenchange',
         'mozfullscreenchange', 'MSFullscreenChange'].forEach(evt => {
            document.addEventListener(evt, handleFullscreenChange);
        });

        function blockFullscreenExitKeys(e) {
            if (!fullscreenLocked) return;

            if (e.key === 'F11' || e.code === 'F11') {
                e.preventDefault();
                e.stopPropagation();
                return false;
            }
            if ((e.key === 'Escape' || e.code === 'Escape') && !document.pointerLockElement) {
                e.preventDefault();
                e.stopPropagation();
                return false;
            }
        }

        window.addEventListener('keydown', blockFullscreenExitKeys, true);
        window.addEventListener('keyup', blockFullscreenExitKeys, true);

        // ==================== НАСТРОЙКИ ====================
        const settings = {
            sensitivity: 1.0,
            volume: 70,
            quality: 1,
            fogDensity: 0.01,
            details: true,
            dynamicLights: true,
            invertY: false,
            screenShake: true,
            dayNightCycle: true
        };

        // ==================== СКИНЫ ПАТРОНОВ ====================
        const BULLET_SKINS = [
            { id: 'default', name: 'Обычный', desc: 'Стандартный жёлтый шарик',
              price: 0, cssColor: '#ffff00', modelType: 'sphere' },
            { id: 'poop', name: '💩 Какашка', desc: 'Коричневая спираль-какашка',
              price: 200, cssColor: '#8B4513', modelType: 'poop' },
            { id: 'trash', name: '🗑️ Зелёный мусор', desc: 'Мусорный бак с крышкой',
              price: 400, cssColor: '#44cc44', modelType: 'trash' },
            { id: 'agusha', name: '🥛 Агуша', desc: 'Молочная бутылочка Агуша',
              price: 2000, cssColor: '#ffffff', modelType: 'bottle' },
            { id: 'gold-agusha', name: '👑 Золотая Агуша', desc: 'Роскошная золотая бутылка',
              price: 20000, cssColor: '#ffcc00', modelType: 'goldBottle' }
        ];

        const ownedSkins = new Set(['default']);
        let activeSkinId = 'default';

        function getActiveSkin() {
            return BULLET_SKINS.find(s => s.id === activeSkinId) || BULLET_SKINS[0];
        }

        // ==================== ОПТИМИЗАЦИЯ: КЭШ ГЕОМЕТРИЙ И МАТЕРИАЛОВ ====================
        const BULLET_CACHE = {
            geometries: {},
            materials: {}
        };

        function getGeometry(key, factory) {
            if (!BULLET_CACHE.geometries[key]) {
                BULLET_CACHE.geometries[key] = factory();
            }
            return BULLET_CACHE.geometries[key];
        }

        function getMaterial(key, factory) {
            if (!BULLET_CACHE.materials[key]) {
                BULLET_CACHE.materials[key] = factory();
            }
            return BULLET_CACHE.materials[key];
        }

        // ==================== МОДЕЛИ ПАТРОНОВ (ОПТИМИЗИРОВАННЫЕ) ====================
        function createBulletModel(skin) {
            const group = new THREE.Group();

            switch (skin.modelType) {
                case 'sphere': {
                    const sphere = new THREE.Mesh(
                        getGeometry('sphere16', () => new THREE.SphereGeometry(0.16, 12, 12)),
                        getMaterial('yellow', () => new THREE.MeshBasicMaterial({ color: 0xffff00 }))
                    );
                    group.add(sphere);
                    const glow = new THREE.Mesh(
                        getGeometry('sphere28', () => new THREE.SphereGeometry(0.28, 12, 12)),
                        getMaterial('yellowGlow', () => new THREE.MeshBasicMaterial({ color: 0xffff00, transparent: true, opacity: 0.35 }))
                    );
                    group.add(glow);
                    break;
                }
                case 'poop': {
                    const mat = getMaterial('poop', () => new THREE.MeshStandardMaterial({ color: 0x6a3a1a, roughness: 0.8 }));
                    const mat2 = getMaterial('poopDark', () => new THREE.MeshStandardMaterial({ color: 0x4a2810, roughness: 0.9 }));
                    const b1 = new THREE.Mesh(getGeometry('poopB1', () => new THREE.SphereGeometry(0.2, 12, 12)), mat);
                    b1.position.y = -0.14; b1.scale.set(1, 0.7, 1); group.add(b1);
                    const b2 = new THREE.Mesh(getGeometry('poopB2', () => new THREE.SphereGeometry(0.16, 12, 12)), mat);
                    b2.position.y = 0.02; b2.scale.set(1, 0.75, 1); group.add(b2);
                    const b3 = new THREE.Mesh(getGeometry('poopB3', () => new THREE.SphereGeometry(0.12, 12, 12)), mat);
                    b3.position.y = 0.15; b3.scale.set(1, 0.8, 1); group.add(b3);
                    const tip = new THREE.Mesh(getGeometry('poopTip', () => new THREE.ConeGeometry(0.06, 0.14, 8)), mat2);
                    tip.position.y = 0.27; group.add(tip);
                    const glow = new THREE.Mesh(
                        getGeometry('poopGlow', () => new THREE.SphereGeometry(0.32, 10, 10)),
                        getMaterial('poopGlowMat', () => new THREE.MeshBasicMaterial({ color: 0x8B4513, transparent: true, opacity: 0.25 }))
                    );
                    group.add(glow);
                    break;
                }
                case 'trash': {
                    const bodyMat = getMaterial('trashBody', () => new THREE.MeshStandardMaterial({ color: 0x44cc44, roughness: 0.6, metalness: 0.2 }));
                    const lidMat = getMaterial('trashLid', () => new THREE.MeshStandardMaterial({ color: 0x2a8a2a, roughness: 0.5 }));
                    const body = new THREE.Mesh(getGeometry('trashBodyGeo', () => new THREE.CylinderGeometry(0.18, 0.15, 0.32, 10)), bodyMat);
                    group.add(body);
                    const lid = new THREE.Mesh(getGeometry('trashLidGeo', () => new THREE.CylinderGeometry(0.2, 0.2, 0.06, 10)), lidMat);
                    lid.position.y = 0.18; group.add(lid);
                    const handle = new THREE.Mesh(getGeometry('trashHandle', () => new THREE.TorusGeometry(0.06, 0.02, 6, 12)), lidMat);
                    handle.position.y = 0.23; handle.rotation.x = Math.PI / 2; group.add(handle);
                    const glow = new THREE.Mesh(
                        getGeometry('trashGlow', () => new THREE.SphereGeometry(0.3, 10, 10)),
                        getMaterial('trashGlowMat', () => new THREE.MeshBasicMaterial({ color: 0x44cc44, transparent: true, opacity: 0.3 }))
                    );
                    group.add(glow);
                    break;
                }
                case 'bottle': {
                    const bodyMat = getMaterial('bottleBody', () => new THREE.MeshStandardMaterial({ color: 0xffffff, roughness: 0.35 }));
                    const body = new THREE.Mesh(getGeometry('bottleBodyGeo', () => new THREE.CylinderGeometry(0.14, 0.13, 0.3, 12)), bodyMat);
                    group.add(body);
                    const neck = new THREE.Mesh(getGeometry('bottleNeckGeo', () => new THREE.CylinderGeometry(0.08, 0.08, 0.08, 12)), bodyMat);
                    neck.position.y = 0.19; group.add(neck);
                    const nipple = new THREE.Mesh(
                        getGeometry('bottleNippleGeo', () => new THREE.SphereGeometry(0.07, 10, 10)),
                        getMaterial('bottleNippleMat', () => new THREE.MeshStandardMaterial({ color: 0xffaa44, roughness: 0.5 }))
                    );
                    nipple.position.y = 0.28; nipple.scale.set(1, 1.3, 1); group.add(nipple);
                    const label = new THREE.Mesh(
                        getGeometry('bottleLabelGeo', () => new THREE.CylinderGeometry(0.145, 0.145, 0.1, 12, 1, true)),
                        getMaterial('bottleLabelMat', () => new THREE.MeshBasicMaterial({ color: 0x4488ff, side: THREE.DoubleSide }))
                    );
                    label.position.y = -0.02; group.add(label);
                    const glow = new THREE.Mesh(
                        getGeometry('bottleGlow', () => new THREE.SphereGeometry(0.28, 10, 10)),
                        getMaterial('bottleGlowMat', () => new THREE.MeshBasicMaterial({ color: 0xffffff, transparent: true, opacity: 0.35 }))
                    );
                    group.add(glow);
                    break;
                }
                case 'goldBottle': {
                    const bodyMat = getMaterial('goldBody', () => new THREE.MeshStandardMaterial({
                        color: 0xffcc00, roughness: 0.15, metalness: 0.9,
                        emissive: 0x553300, emissiveIntensity: 0.4
                    }));
                    const body = new THREE.Mesh(getGeometry('goldBodyGeo', () => new THREE.CylinderGeometry(0.14, 0.13, 0.3, 12)), bodyMat);
                    group.add(body);
                    const neck = new THREE.Mesh(getGeometry('goldNeckGeo', () => new THREE.CylinderGeometry(0.08, 0.08, 0.08, 12)), bodyMat);
                    neck.position.y = 0.19; group.add(neck);
                    const nipple = new THREE.Mesh(
                        getGeometry('goldNippleGeo', () => new THREE.OctahedronGeometry(0.09, 0)),
                        getMaterial('goldNippleMat', () => new THREE.MeshStandardMaterial({
                            color: 0xffeeaa, roughness: 0.05, metalness: 0.8,
                            emissive: 0xaa8800, emissiveIntensity: 0.6
                        }))
                    );
                    nipple.position.y = 0.3; group.add(nipple);
                    const label = new THREE.Mesh(
                        getGeometry('goldLabelGeo', () => new THREE.CylinderGeometry(0.148, 0.148, 0.1, 12, 1, true)),
                        getMaterial('goldLabelMat', () => new THREE.MeshStandardMaterial({
                            color: 0xaa7700, roughness: 0.2, metalness: 0.95,
                            side: THREE.DoubleSide
                        }))
                    );
                    label.position.y = -0.02; group.add(label);
                    const glow = new THREE.Mesh(
                        getGeometry('goldGlow', () => new THREE.SphereGeometry(0.32, 12, 12)),
                        getMaterial('goldGlowMat', () => new THREE.MeshBasicMaterial({ color: 0xffdd44, transparent: true, opacity: 0.45 }))
                    );
                    group.add(glow);
                    const halo = new THREE.Mesh(
                        getGeometry('goldHalo', () => new THREE.SphereGeometry(0.45, 12, 12)),
                        getMaterial('goldHaloMat', () => new THREE.MeshBasicMaterial({ color: 0xffaa00, transparent: true, opacity: 0.15 }))
                    );
                    group.add(halo);
                    // Убираем PointLight из каждой пули — это сильно бьёт по производительности
                    // const light = new THREE.PointLight(0xffcc00, 1.0, 5);
                    // group.add(light);
                    break;
                }
            }
            return group;
        }

        // ==================== ЗАГРУЗКА ====================
        window.addEventListener('load', () => {
            setTimeout(() => {
                document.getElementById('loading').classList.add('hidden');
            }, 500);
        });

        // ==================== 3D СЦЕНА ====================
        const scene = new THREE.Scene();
        const DAY_SKY = new THREE.Color(0x87CEEB);
        scene.background = DAY_SKY.clone();
        scene.fog = new THREE.FogExp2(0x87CEEB, settings.fogDensity);

        const camera = new THREE.PerspectiveCamera(75, window.innerWidth / window.innerHeight, 0.1, 500);
        camera.position.y = 1.6;
        camera.rotation.order = 'YXZ';

        const renderer = new THREE.WebGLRenderer({ antialias: settings.quality > 0, powerPreference: 'high-performance' });
        renderer.setSize(window.innerWidth, window.innerHeight);
        renderer.setPixelRatio(Math.min(window.devicePixelRatio, settings.quality === 2 ? 2 : 1));
        renderer.shadowMap.enabled = settings.quality > 0;
        renderer.shadowMap.type = THREE.PCFSoftShadowMap;
        document.body.appendChild(renderer.domElement);

        // ==================== ОСВЕЩЕНИЕ ====================
        const ambientLight = new THREE.AmbientLight(0xffffff, 0.5);
        scene.add(ambientLight);

        const hemiLight = new THREE.HemisphereLight(0x87CEEB, 0x556B2F, 0.6);
        scene.add(hemiLight);

        const sunLight = new THREE.DirectionalLight(0xffffff, 1.0);
        sunLight.position.set(80, 120, 50);
        sunLight.castShadow = settings.quality > 0;
        sunLight.shadow.mapSize.width = settings.quality === 2 ? 2048 : 1024;
        sunLight.shadow.mapSize.height = settings.quality === 2 ? 2048 : 1024;
        sunLight.shadow.camera.left = -60;
        sunLight.shadow.camera.right = 60;
        sunLight.shadow.camera.top = 60;
        sunLight.shadow.camera.bottom = -60;
        sunLight.shadow.camera.near = 1;
        sunLight.shadow.camera.far = 200;
        scene.add(sunLight);

        const moonLight = new THREE.DirectionalLight(0x8899ff, 0.0);
        moonLight.position.set(-80, 100, -50);
        scene.add(moonLight);

        // ==================== ИГРОВЫЕ ПЕРЕМЕННЫЕ ====================
        let health = 100;
        let maxHealth = 100;
        let score = 0;
        let coins = 0;
        let gameOver = false;
        let gameStarted = false;
        let paused = false;
        let enemiesKilled = 0;
        let waveNumber = 1;
        let cameraShake = 0;

        // ★★★ 6 ПАТРОНОВ В СЕКУНДУ = 1000/6 ≈ 166.67 мс ★★★
        const FIRE_RATE_MS = 166.67;
        let lastShotTime = 0;
        let isShooting = false;

        // ==================== ЦИКЛ ДНЯ / НОЧИ ====================
        const DAY_DURATION = 180;
        const NIGHT_DURATION = 120;
        const TOTAL_DURATION = DAY_DURATION + NIGHT_DURATION;
        let worldTime = 0;
        let currentPeriod = 'day';

        const daySky = new THREE.Color(0x87CEEB);
        const sunsetSky = new THREE.Color(0xff8855);
        const nightSky = new THREE.Color(0x0a0a2a);

        // ==================== ОКРУЖЕНИЕ ====================
        function createGroundTexture() {
            const canvas = document.createElement('canvas');
            canvas.width = 256; canvas.height = 256;
            const ctx = canvas.getContext('2d');
            ctx.fillStyle = '#3a5a2a';
            ctx.fillRect(0, 0, 256, 256);
            for (let i = 0; i < 1500; i++) {
                const x = Math.random() * 256;
                const y = Math.random() * 256;
                const shade = 40 + Math.random() * 60;
                ctx.fillStyle = `rgb(${shade - 20}, ${shade + 40}, ${shade - 30})`;
                ctx.fillRect(x, y, 2, 3);
            }
            return new THREE.CanvasTexture(canvas);
        }

        const groundTex = createGroundTexture();
        groundTex.wrapS = groundTex.wrapT = THREE.RepeatWrapping;
        groundTex.repeat.set(20, 20);

        const planeGeometry = new THREE.PlaneGeometry(200, 200);
        const planeMaterial = new THREE.MeshLambertMaterial({ map: groundTex });
        const plane = new THREE.Mesh(planeGeometry, planeMaterial);
        plane.rotation.x = -Math.PI / 2;
        plane.receiveShadow = settings.quality > 0;
        scene.add(plane);

        // ==================== ЗВЁЗДЫ ====================
        const stars = (() => {
            const geo = new THREE.BufferGeometry();
            const positions = [];
            for (let i = 0; i < 800; i++) {
                const r = 300 + Math.random() * 200;
                const theta = Math.random() * Math.PI * 2;
                const phi = Math.random() * Math.PI * 0.5;
                positions.push(
                    r * Math.sin(phi) * Math.cos(theta),
                    r * Math.cos(phi),
                    r * Math.sin(phi) * Math.sin(theta)
                );
            }
            geo.setAttribute('position', new THREE.Float32BufferAttribute(positions, 3));
            const mat = new THREE.PointsMaterial({
                color: 0xffffff, size: 1.5,
                transparent: true, opacity: 0.0, sizeAttenuation: false
            });
            const points = new THREE.Points(geo, mat);
            scene.add(points);
            return points;
        })();

        // ==================== ДЕТАЛИ МИРА ====================
        const worldDetails = new THREE.Group();
        scene.add(worldDetails);

        const colliders = [];

        function addCollider(x, z, radius) {
            colliders.push({ x, z, radius });
        }

        function createRock() {
            const geo = new THREE.DodecahedronGeometry(0.5 + Math.random() * 0.8);
            const positions = geo.attributes.position;
            for (let i = 0; i < positions.count; i++) {
                positions.setXYZ(i,
                    positions.getX(i) * (0.8 + Math.random() * 0.4),
                    positions.getY(i) * (0.7 + Math.random() * 0.3),
                    positions.getZ(i) * (0.8 + Math.random() * 0.4)
                );
            }
            geo.computeVertexNormals();
            const mat = new THREE.MeshStandardMaterial({
                color: 0x666a70, roughness: 0.9, flatShading: true
            });
            const rock = new THREE.Mesh(geo, mat);
            rock.castShadow = settings.quality > 0;
            rock.receiveShadow = settings.quality > 0;
            return rock;
        }

        function createTree() {
            const group = new THREE.Group();
            const trunkGeo = new THREE.CylinderGeometry(0.3, 0.45, 3, 8);
            const trunkMat = new THREE.MeshStandardMaterial({ color: 0x4a2a1a, roughness: 0.95 });
            const trunk = new THREE.Mesh(trunkGeo, trunkMat);
            trunk.position.y = 1.5;
            trunk.castShadow = settings.quality > 0;
            group.add(trunk);
            const leafMat = new THREE.MeshStandardMaterial({
                color: 0x2a5a2a, roughness: 0.9, flatShading: true
            });
            for (let i = 0; i < 4; i++) {
                const leafGeo = new THREE.IcosahedronGeometry(1.2 + Math.random() * 0.5, 0);
                const leaf = new THREE.Mesh(leafGeo, leafMat);
                leaf.position.set(
                    (Math.random() - 0.5) * 1.2,
                    3 + Math.random() * 1,
                    (Math.random() - 0.5) * 1.2
                );
                leaf.castShadow = settings.quality > 0;
                group.add(leaf);
            }
            return group;
        }

        function createBush() {
            const group = new THREE.Group();
            const mat = new THREE.MeshStandardMaterial({
                color: 0x2a5a2a, roughness: 1, flatShading: true
            });
            for (let i = 0; i < 3; i++) {
                const geo = new THREE.IcosahedronGeometry(0.5 + Math.random() * 0.3, 0);
                const bush = new THREE.Mesh(geo, mat);
                bush.position.set(
                    (Math.random() - 0.5) * 0.8, 0.4, (Math.random() - 0.5) * 0.8
                );
                bush.castShadow = settings.quality > 0;
                group.add(bush);
            }
            return group;
        }

        function populateWorld() {
            while (worldDetails.children.length) {
                worldDetails.remove(worldDetails.children[0]);
            }
            colliders.length = 0;

            const count = settings.details ? 60 : 15;
            for (let i = 0; i < count; i++) {
                const rock = createRock();
                let x, z;
                do {
                    x = Math.random() * 180 - 90;
                    z = Math.random() * 180 - 90;
                } while (Math.sqrt(x * x + z * z) < 8);
                rock.position.set(x, 0.2, z);
                rock.rotation.set(Math.random(), Math.random(), Math.random());
                worldDetails.add(rock);
                const box = new THREE.Box3().setFromObject(rock);
                const size = box.getSize(new THREE.Vector3());
                addCollider(x, z, Math.max(size.x, size.z) * 0.55);
            }

            const treeCount = settings.details ? 30 : 8;
            for (let i = 0; i < treeCount; i++) {
                const tree = createTree();
                let x, z;
                do {
                    x = Math.random() * 180 - 90;
                    z = Math.random() * 180 - 90;
                } while (Math.sqrt(x * x + z * z) < 12);
                tree.position.set(x, 0, z);
                tree.rotation.y = Math.random() * Math.PI * 2;
                worldDetails.add(tree);
                addCollider(x, z, 0.6);
            }

            const bushCount = settings.details ? 40 : 10;
            for (let i = 0; i < bushCount; i++) {
                const bush = createBush();
                bush.position.set(
                    Math.random() * 180 - 90, 0, Math.random() * 180 - 90
                );
                worldDetails.add(bush);
            }
        }

        populateWorld();

        // ==================== КОЛЛИЗИЯ ====================
        const PLAYER_RADIUS = 0.5;
        function resolveCollisions(newPos) {
            const LIMIT = 98;
            newPos.x = Math.max(-LIMIT, Math.min(LIMIT, newPos.x));
            newPos.z = Math.max(-LIMIT, Math.min(LIMIT, newPos.z));

            for (let i = 0; i < colliders.length; i++) {
                const c = colliders[i];
                const dx = newPos.x - c.x;
                const dz = newPos.z - c.z;
                const minDist = c.radius + PLAYER_RADIUS;
                const distSq = dx * dx + dz * dz;

                if (distSq < minDist * minDist && distSq > 0.0001) {
                    const dist = Math.sqrt(distSq);
                    const pushX = (dx / dist) * (minDist - dist);
                    const pushZ = (dz / dist) * (minDist - dist);
                    newPos.x += pushX;
                    newPos.z += pushZ;
                }
            }
            return newPos;
        }

        // ==================== 3D МОДЕЛИ ВРАГОВ (ОПТИМИЗИРОВАННЫЕ) ====================
        // Кэшируем общие геометрии и материалы для врагов
        const ENEMY_CACHE = {
            geometries: {},
            materials: {}
        };

        function getEnemyGeo(key, factory) {
            if (!ENEMY_CACHE.geometries[key]) {
                ENEMY_CACHE.geometries[key] = factory();
            }
            return ENEMY_CACHE.geometries[key];
        }

        function getEnemyMat(key, factory) {
            if (!ENEMY_CACHE.materials[key]) {
                ENEMY_CACHE.materials[key] = factory();
            }
            return ENEMY_CACHE.materials[key];
        }

        function createSmugModel() {
            const group = new THREE.Group();
            const bodyMat = getEnemyMat('smugBody', () => new THREE.MeshStandardMaterial({ color: 0x22cc44, roughness: 0.7 }));
            const eyeMat  = getEnemyMat('smugEye', () => new THREE.MeshStandardMaterial({ color: 0xffffff, roughness: 0.3 }));
            const pupilMat= getEnemyMat('smugPupil', () => new THREE.MeshBasicMaterial({ color: 0x000000 }));
            const smileMat = getEnemyMat('smugSmile', () => new THREE.MeshBasicMaterial({ color: 0x000000 }));
            const armMat = getEnemyMat('smugArm', () => new THREE.MeshStandardMaterial({ color: 0x118822, roughness: 0.7 }));
            const legMat = getEnemyMat('smugLeg', () => new THREE.MeshStandardMaterial({ color: 0x0a6a1a, roughness: 0.8 }));

            const body = new THREE.Mesh(getEnemyGeo('smugBodyGeo', () => new THREE.SphereGeometry(0.7, 12, 12)), bodyMat);
            body.position.y = 0.9; body.scale.set(1, 1.15, 1); body.castShadow = true;
            group.add(body);
            const head = new THREE.Mesh(getEnemyGeo('smugHeadGeo', () => new THREE.SphereGeometry(0.55, 12, 12)), bodyMat);
            head.position.y = 1.85; head.castShadow = true; group.add(head);
            const eyeL = new THREE.Mesh(getEnemyGeo('smugEyeGeo', () => new THREE.SphereGeometry(0.14, 10, 10)), eyeMat);
            eyeL.position.set(-0.2, 1.95, 0.45); group.add(eyeL);
            const eyeR = eyeL.clone(); eyeR.position.x = 0.2; group.add(eyeR);
            const pupilL = new THREE.Mesh(getEnemyGeo('smugPupilGeo', () => new THREE.SphereGeometry(0.07, 8, 8)), pupilMat);
            pupilL.position.set(-0.2, 1.95, 0.56); group.add(pupilL);
            const pupilR = pupilL.clone(); pupilR.position.x = 0.2; group.add(pupilR);
            const smileGeo = getEnemyGeo('smugSmileGeo', () => new THREE.TorusGeometry(0.25, 0.04, 8, 16, Math.PI));
            const smile = new THREE.Mesh(smileGeo, smileMat);
            smile.position.set(0, 1.75, 0.5); smile.rotation.z = Math.PI; group.add(smile);
            const armL = new THREE.Mesh(getEnemyGeo('smugArmGeo', () => new THREE.CylinderGeometry(0.15, 0.15, 0.9, 8)), armMat);
            armL.position.set(-0.8, 1.0, 0); armL.rotation.z = 0.3; armL.castShadow = true;
            group.add(armL);
            const armR = armL.clone(); armR.position.x = 0.8; armR.rotation.z = -0.3; group.add(armR);
            const legL = new THREE.Mesh(getEnemyGeo('smugLegGeo', () => new THREE.CylinderGeometry(0.18, 0.18, 0.7, 8)), legMat);
            legL.position.set(-0.3, 0.35, 0); legL.castShadow = true; group.add(legL);
            const legR = legL.clone(); legR.position.x = 0.3; group.add(legR);
            return group;
        }

        function createScreamerModel() {
            const group = new THREE.Group();
            const bodyMat = getEnemyMat('screamBody', () => new THREE.MeshStandardMaterial({ color: 0xffffff, roughness: 0.6 }));
            const mouthMat = getEnemyMat('screamMouth', () => new THREE.MeshBasicMaterial({ color: 0x330000 }));
            const eyeMat = getEnemyMat('screamEye', () => new THREE.MeshBasicMaterial({ color: 0x000000 }));
            const tearMat = getEnemyMat('screamTear', () => new THREE.MeshBasicMaterial({ color: 0x000000, transparent: true, opacity: 0.4 }));
            const armMat = getEnemyMat('screamArm', () => new THREE.MeshStandardMaterial({ color: 0xdddddd, roughness: 0.7 }));
            const legMat = getEnemyMat('screamLeg', () => new THREE.MeshStandardMaterial({ color: 0xcccccc, roughness: 0.8 }));

            const body = new THREE.Mesh(getEnemyGeo('screamBodyGeo', () => new THREE.SphereGeometry(0.75, 12, 12)), bodyMat);
            body.position.y = 1.0; body.scale.set(1, 1.1, 1); body.castShadow = true;
            group.add(body);
            const head = new THREE.Mesh(getEnemyGeo('screamHeadGeo', () => new THREE.SphereGeometry(0.6, 12, 12)), bodyMat);
            head.position.y = 1.95; head.scale.set(1, 1.1, 1); head.castShadow = true;
            group.add(head);
            const mouth = new THREE.Mesh(getEnemyGeo('screamMouthGeo', () => new THREE.SphereGeometry(0.4, 12, 12)), mouthMat);
            mouth.position.set(0, 1.75, 0.35); mouth.scale.set(1.1, 1.4, 0.6); group.add(mouth);
            const eyeL = new THREE.Mesh(getEnemyGeo('screamEyeGeo', () => new THREE.SphereGeometry(0.12, 10, 10)), eyeMat);
            eyeL.position.set(-0.28, 2.15, 0.45); group.add(eyeL);
            const eyeR = eyeL.clone(); eyeR.position.x = 0.28; group.add(eyeR);
            const lineL = new THREE.Mesh(getEnemyGeo('screamLineGeo', () => new THREE.SphereGeometry(0.06, 8, 8)), tearMat);
            lineL.position.set(-0.28, 2.02, 0.55); lineL.scale.set(0.5, 2.5, 0.3); group.add(lineL);
            const lineR = lineL.clone(); lineR.position.x = 0.28; group.add(lineR);
            const armL = new THREE.Mesh(getEnemyGeo('screamArmGeo', () => new THREE.CylinderGeometry(0.15, 0.15, 1.1, 8)), armMat);
            armL.position.set(-0.85, 1.4, 0); armL.rotation.z = -0.6; armL.castShadow = true;
            group.add(armL);
            const armR = armL.clone(); armR.position.x = 0.85; armR.rotation.z = 0.6; group.add(armR);
            const legL = new THREE.Mesh(getEnemyGeo('screamLegGeo', () => new THREE.CylinderGeometry(0.18, 0.18, 0.75, 8)), legMat);
            legL.position.set(-0.3, 0.38, 0); legL.castShadow = true; group.add(legL);
            const legR = legL.clone(); legR.position.x = 0.3; group.add(legR);
            return group;
        }

        function createCrierModel() {
            const group = new THREE.Group();
            const bodyMat = getEnemyMat('crierBody', () => new THREE.MeshStandardMaterial({ color: 0x5577aa, roughness: 0.7 }));
            const eyeMat = getEnemyMat('crierEye', () => new THREE.MeshBasicMaterial({ color: 0x000000 }));
            const tearMat = getEnemyMat('crierTear', () => new THREE.MeshStandardMaterial({
                color: 0x88ccff, transparent: true, opacity: 0.85,
                roughness: 0.1, metalness: 0.3
            }));
            const frownMat = getEnemyMat('crierFrown', () => new THREE.MeshBasicMaterial({ color: 0x000000 }));
            const armMat = getEnemyMat('crierArm', () => new THREE.MeshStandardMaterial({ color: 0x334466, roughness: 0.8 }));
            const legMat = getEnemyMat('crierLeg', () => new THREE.MeshStandardMaterial({ color: 0x223355, roughness: 0.8 }));

            const body = new THREE.Mesh(getEnemyGeo('crierBodyGeo', () => new THREE.SphereGeometry(0.65, 12, 12)), bodyMat);
            body.position.y = 0.85; body.scale.set(1, 1.1, 1); body.castShadow = true;
            group.add(body);
            const head = new THREE.Mesh(getEnemyGeo('crierHeadGeo', () => new THREE.SphereGeometry(0.5, 12, 12)), bodyMat);
            head.position.y = 1.7; head.castShadow = true; group.add(head);
            const eyeL = new THREE.Mesh(getEnemyGeo('crierEyeGeo', () => new THREE.SphereGeometry(0.13, 10, 10)), eyeMat);
            eyeL.position.set(-0.18, 1.8, 0.42); group.add(eyeL);
            const eyeR = eyeL.clone(); eyeR.position.x = 0.18; group.add(eyeR);
            const tearL = new THREE.Mesh(getEnemyGeo('crierTearGeo', () => new THREE.SphereGeometry(0.09, 8, 8)), tearMat);
            tearL.position.set(-0.18, 1.62, 0.5); tearL.scale.set(0.7, 1.4, 0.7); group.add(tearL);
            const tearR = tearL.clone(); tearR.position.x = 0.18; group.add(tearR);
            const frownGeo = getEnemyGeo('crierFrownGeo', () => new THREE.TorusGeometry(0.2, 0.035, 8, 16, Math.PI));
            const frown = new THREE.Mesh(frownGeo, frownMat);
            frown.position.set(0, 1.55, 0.45); group.add(frown);
            const armL = new THREE.Mesh(getEnemyGeo('crierArmGeo', () => new THREE.CylinderGeometry(0.13, 0.13, 0.85, 8)), armMat);
            armL.position.set(-0.75, 0.95, 0); armL.rotation.z = 0.15; armL.castShadow = true;
            group.add(armL);
            const armR = armL.clone(); armR.position.x = 0.75; armR.rotation.z = -0.15; group.add(armR);
            const legL = new THREE.Mesh(getEnemyGeo('crierLegGeo', () => new THREE.CylinderGeometry(0.16, 0.16, 0.65, 8)), legMat);
            legL.position.set(-0.28, 0.33, 0); legL.castShadow = true; group.add(legL);
            const legR = legL.clone(); legR.position.x = 0.28; group.add(legR);
            group.userData.tearL = tearL;
            group.userData.tearR = tearR;
            return group;
        }

        function createEnemyModel(type) {
            if (type === 'smug')     return createSmugModel();
            if (type === 'screamer') return createScreamerModel();
            return createCrierModel();
        }

        // ==================== ВРАГИ ====================
        class Enemy {
            constructor(type) {
                this.type = type;
                this.speed = 4.5;
                this.health = type === 'crier' ? 1 : type === 'smug' ? 5 : 10;
                this.maxHealth = this.health;
                this.damage = type === 'crier' ? 5 : type === 'smug' ? 10 : 15;
                this.hitCooldown = 0;
                this.walkPhase = Math.random() * Math.PI * 2;

                this.mesh = createEnemyModel(type);

                const angle = Math.random() * Math.PI * 2;
                const radius = 30 + Math.random() * 20;
                this.mesh.position.set(
                    Math.cos(angle) * radius, 0, Math.sin(angle) * radius
                );
                scene.add(this.mesh);

                const lightGeo = new THREE.CircleGeometry(1.0, 16);
                const lightMat = new THREE.MeshBasicMaterial({
                    color: type === 'smug' ? 0x44ff44 : type === 'screamer' ? 0xffffff : 0x6688ff,
                    transparent: true, opacity: 0.35, side: THREE.DoubleSide
                });
                this.lightRing = new THREE.Mesh(lightGeo, lightMat);
                this.lightRing.rotation.x = -Math.PI / 2;
                this.lightRing.position.y = 0.02;
                scene.add(this.lightRing);
            }

            update(playerPos, dt) {
                if (gameOver || paused) return;
                this.hitCooldown = Math.max(0, this.hitCooldown - dt);

                const dx = playerPos.x - this.mesh.position.x;
                const dz = playerPos.z - this.mesh.position.z;
                const distSq = dx*dx + dz*dz;

                if (this.type === 'smug') {
                    this.mesh.position.x += Math.sin(performance.now() * 0.005) * dt * 1.2;
                }

                if (distSq > 2.25) { // 1.5^2
                    const dist = Math.sqrt(distSq);
                    const invDist = 1 / dist;
                    this.mesh.position.x += dx * invDist * this.speed * dt;
                    this.mesh.position.z += dz * invDist * this.speed * dt;

                    const targetAngle = Math.atan2(dx, dz);
                    let diff = targetAngle - this.mesh.rotation.y;
                    while (diff > Math.PI)  diff -= Math.PI * 2;
                    while (diff < -Math.PI) diff += Math.PI * 2;
                    this.mesh.rotation.y += diff * Math.min(1, dt * 8);
                } else if (this.hitCooldown <= 0) {
                    damagePlayer(this.damage);
                    this.hitCooldown = 0.8;
                }

                this.lightRing.position.x = this.mesh.position.x;
                this.lightRing.position.z = this.mesh.position.z;

                const t = performance.now() * 0.001;
                const bob = Math.sin(t * 6 + this.walkPhase) * 0.05;
                this.mesh.position.y = Math.abs(bob);

                if (this.type === 'screamer') {
                    const s = 1 + Math.sin(t * 12) * 0.05;
                    this.mesh.scale.set(s, s, s);
                    this.mesh.rotation.z = Math.sin(t * 8) * 0.08;
                } else if (this.type === 'crier') {
                    this.mesh.rotation.z = Math.sin(t * 3) * 0.05;
                    if (this.mesh.userData.tearL) {
                        this.mesh.userData.tearL.position.y = 1.62 + Math.sin(t * 4) * 0.05;
                        this.mesh.userData.tearR.position.y = 1.62 + Math.sin(t * 4 + 0.5) * 0.05;
                    }
                } else if (this.type === 'smug') {
                    this.mesh.rotation.z = Math.sin(t * 4) * 0.06;
                }
            }

            takeDamage(dmg) {
                this.health -= dmg;
                if (this.health <= 0) {
                    coins += 10;
                    enemiesKilled++;
                    document.getElementById('coins').innerText = "💰 Монеты: " + coins;
                    document.getElementById('score').innerText = "💀 Убито: " + enemiesKilled;
                    updateShopCoinsBadge();
                    addKillFeed(this.type);
                    scene.remove(this.mesh);
                    scene.remove(this.lightRing);
                    // Освобождаем ресурсы? Нет — они из кэша, переиспользуются.
                    return true;
                }
                this.mesh.traverse(child => {
                    if (child.isMesh && child.material) {
                        if (!child.userData.origColor) {
                            child.userData.origColor = child.material.color.clone();
                        }
                        child.material.color.setHex(0xff5555);
                    }
                });
                setTimeout(() => {
                    this.mesh.traverse(child => {
                        if (child.isMesh && child.material && child.userData.origColor) {
                            child.material.color.copy(child.userData.origColor);
                        }
                    });
                }, 90);
                return false;
            }
        }

        // ==================== KILL FEED ====================
        const killFeed = document.getElementById('kill-feed');
        function addKillFeed(type) {
            const names = { smug: '😏 Смаг', screamer: '😱 Крикун', crier: '😢 Плакса' };
            const msg = document.createElement('div');
            msg.className = 'kill-msg';
            msg.textContent = `${names[type]} уничтожен (+10 💰)`;
            killFeed.appendChild(msg);
            setTimeout(() => {
                msg.style.transition = 'opacity 0.5s, transform 0.5s';
                msg.style.opacity = '0';
                msg.style.transform = 'translateX(50px)';
                setTimeout(() => msg.remove(), 500);
            }, 2000);
        }

        // ==================== ПУЛИ (ПУЛ ОБЪЕКТОВ) ====================
        const bullets = [];
        const bulletPool = [];
        const MAX_POOL_SIZE = 60;

        function acquireBullet() {
            if (bulletPool.length > 0) {
                return bulletPool.pop();
            }
            const skin = getActiveSkin();
            const bullet = createBulletModel(skin);
            bullet.userData.skinId = skin.id;
            return bullet;
        }

        function releaseBullet(bullet) {
            scene.remove(bullet);
            if (bulletPool.length < MAX_POOL_SIZE) {
                bulletPool.push(bullet);
            }
        }

        // Предзаполняем пул
        (function prewarmPool() {
            for (let i = 0; i < 30; i++) {
                const skin = BULLET_SKINS[0];
                const bullet = createBulletModel(skin);
                bullet.userData.skinId = skin.id;
                bulletPool.push(bullet);
            }
        })();

        function shoot() {
            if (!gameStarted || gameOver || paused) return;

            const now = performance.now();
            if (now - lastShotTime < FIRE_RATE_MS) return;
            lastShotTime = now;

            const skin = getActiveSkin();

            let bullet = acquireBullet();
            // Если сменили скин — пересоздаём модель
            if (bullet.userData.skinId !== skin.id) {
                bullet = createBulletModel(skin);
                bullet.userData.skinId = skin.id;
            }

            const forward = new THREE.Vector3(0, 0, -1).applyQuaternion(camera.quaternion);
            bullet.position.copy(camera.position).addScaledVector(forward, 0.8);

            const quat = new THREE.Quaternion().setFromUnitVectors(
                new THREE.Vector3(0, 1, 0), forward.clone().normalize()
            );
            bullet.quaternion.copy(quat);

            bullet.userData.velocity = forward.clone().multiplyScalar(40);
            bullet.userData.life = 0;

            scene.add(bullet);
            bullets.push(bullet);

            if (settings.screenShake) cameraShake = 0.1;
        }

        document.addEventListener('mousedown', (e) => {
            if (e.button === 0) {
                isShooting = true;
                lastShotTime = 0;
                shoot();
            }
        });

        document.addEventListener('mouseup', (e) => {
            if (e.button === 0) isShooting = false;
        });

        window.addEventListener('blur', () => { isShooting = false; });

        // ==================== УПРАВЛЕНИЕ ====================
        const enemies = [];
        const keys = {};
        let velocity = new THREE.Vector3();
        let yaw = 0;
        let pitch = 0;

        document.addEventListener('keydown', (e) => { keys[e.code] = true; });
        document.addEventListener('keyup', (e) => keys[e.code] = false);

        document.addEventListener('pointerlockchange', () => {
            if (!document.pointerLockElement && gameStarted && !gameOver && !paused) {
                togglePause();
            }
        });

        window.addEventListener('blur', () => {
            if (gameStarted && !gameOver && !paused) togglePause();
        });

        document.addEventListener('mousemove', (e) => {
            if (gameOver || !gameStarted || paused) return;

            const sens = 0.002 * settings.sensitivity;
            yaw -= e.movementX * sens;
            const invert = settings.invertY ? -1 : 1;
            pitch -= e.movementY * sens * invert;

            const maxPitch = Math.PI / 2 - 0.05;
            pitch = Math.max(-maxPitch, Math.min(maxPitch, pitch));

            camera.rotation.y = yaw;
            camera.rotation.x = pitch;
            camera.rotation.z = 0;
        });

        // ==================== УРОН / СПАВН ====================
        function damagePlayer(amount) {
            if (gameOver) return;
            health -= amount;
            updateHealthUI();
            if (settings.screenShake) cameraShake = 0.3;

            const flash = document.getElementById('damage-flash');
            flash.style.opacity = '0.6';
            setTimeout(() => flash.style.opacity = '0', 150);

            if (health <= 0) {
                gameOver = true;
                document.getElementById('gameover').style.display = 'block';
                if (document.pointerLockElement) document.exitPointerLock();
            }
        }

        function updateHealthUI() {
            const hp = Math.max(0, Math.round(health));
            document.getElementById('health').innerText = `❤️ ${hp} / ${maxHealth}`;
            document.getElementById('health-fill').style.width = (hp / maxHealth * 100) + '%';
        }

        function spawnEnemy() {
            if (gameOver) return;
            if (gameStarted && !paused) {
                const types = ['smug', 'screamer', 'crier'];
                const randomType = types[Math.floor(Math.random() * types.length)];
                enemies.push(new Enemy(randomType));

                if (enemiesKilled > 0 && enemiesKilled % 10 === 0) {
                    waveNumber = Math.floor(enemiesKilled / 10) + 1;
                    document.getElementById('wave-info').innerText = "🌊 Волна: " + waveNumber;
                }
            }
            setTimeout(spawnEnemy, Math.max(600, 1500 - waveNumber * 50) + Math.random() * 1500);
        }

        function updateBullets(dt) {
            for (let i = bullets.length - 1; i >= 0; i--) {
                let bullet = bullets[i];
                const vel = bullet.userData.velocity;
                bullet.position.x += vel.x * dt;
                bullet.position.y += vel.y * dt;
                bullet.position.z += vel.z * dt;
                bullet.userData.life += dt;
                bullet.rotateY(dt * 8);

                if (bullet.userData.life > 3) {
                    releaseBullet(bullet);
                    bullets.splice(i, 1);
                    continue;
                }

                for (let j = enemies.length - 1; j >= 0; j--) {
                    let enemy = enemies[j];
                    // Проверка по XY+Z (быстрая)
                    const dx = bullet.position.x - enemy.mesh.position.x;
                    const dy = bullet.position.y - (enemy.mesh.position.y + 1.0);
                    const dz = bullet.position.z - enemy.mesh.position.z;
                    if (dx*dx + dy*dy + dz*dz < 1.69) { // 1.3^2
                        releaseBullet(bullet);
                        bullets.splice(i, 1);
                        if (enemy.takeDamage(1)) enemies.splice(j, 1);
                        break;
                    }
                }
            }
        }

        // ==================== ЦИКЛ ДНЯ/НОЧИ ====================
        const timeInfo = document.getElementById('time-info');
        const nightOverlay = document.getElementById('night-overlay');
        const hudEl = document.getElementById('hud');
        const timeNotification = document.getElementById('time-notification');

        function lerpColor(a, b, t) {
            return new THREE.Color(
                a.r + (b.r - a.r) * t,
                a.g + (b.g - a.g) * t,
                a.b + (b.b - a.b) * t
            );
        }

        function showTimeNotification(text, color) {
            timeNotification.textContent = text;
            timeNotification.style.color = color;
            timeNotification.style.display = 'block';
            timeNotification.style.opacity = '1';
            timeNotification.style.transform = 'translate(-50%, -50%) scale(1.1)';
            setTimeout(() => {
                timeNotification.style.opacity = '0';
                timeNotification.style.transform = 'translate(-50%, -50%) scale(1.3)';
                setTimeout(() => { timeNotification.style.display = 'none'; }, 500);
            }, 2000);
        }

        function updateDayNight(dt) {
            if (!settings.dayNightCycle) return;

            worldTime += dt;
            if (worldTime >= TOTAL_DURATION) worldTime -= TOTAL_DURATION;

            let t = 0;
            let newPeriod;

            if (worldTime < DAY_DURATION) {
                newPeriod = 'day';
                const remaining = DAY_DURATION - worldTime;
                if (remaining < 30) t = (30 - remaining) / 30 * 0.7;
                else t = 0;
            } else {
                newPeriod = 'night';
                const nightTime = worldTime - DAY_DURATION;
                const remaining = NIGHT_DURATION - nightTime;
                if (remaining > NIGHT_DURATION - 30) {
                    const into = NIGHT_DURATION - remaining;
                    t = 1 - (into / 30) * 0.7;
                } else if (remaining < 30) {
                    t = 0.7 + (30 - remaining) / 30 * 0.3;
                } else t = 1;
            }

            if (newPeriod !== currentPeriod) {
                currentPeriod = newPeriod;
                if (newPeriod === 'night') showTimeNotification('🌙 НОЧЬ', '#aaccff');
                else showTimeNotification('☀️ ДЕНЬ', '#ffdd44');
            }

            let skyColor;
            if (t < 0.5) skyColor = lerpColor(daySky, sunsetSky, t * 2);
            else skyColor = lerpColor(sunsetSky, nightSky, (t - 0.5) * 2);

            scene.background.copy(skyColor);
            scene.fog.color.copy(skyColor);

            sunLight.intensity = Math.max(0, 1 - t * 1.2);
            moonLight.intensity = Math.max(0, t * 0.7 - 0.2);
            ambientLight.intensity = 0.5 - t * 0.35;
            hemiLight.intensity = 0.6 - t * 0.45;

            if (t < 0.7) sunLight.color.setHex(0xffffff);
            else sunLight.color.setHex(0xffaa66);

            stars.material.opacity = Math.max(0, (t - 0.6) * 2.5);
            stars.rotation.y += dt * 0.005;
            nightOverlay.style.opacity = t * 0.4;

            if (t < 0.3) {
                timeInfo.textContent = '☀️ День';
                timeInfo.style.color = '#ffdd88';
                hudEl.style.borderLeftColor = '#66aaff';
            } else if (t < 0.7) {
                timeInfo.textContent = '🌅 Закат';
                timeInfo.style.color = '#ff8844';
                hudEl.style.borderLeftColor = '#ff8844';
            } else {
                timeInfo.textContent = '🌙 Ночь';
                timeInfo.style.color = '#aaccff';
                hudEl.style.borderLeftColor = '#aaccff';
            }

            const currentPhaseTime = worldTime < DAY_DURATION
                ? DAY_DURATION - worldTime
                : NIGHT_DURATION - (worldTime - DAY_DURATION);
            const mins = Math.floor(currentPhaseTime / 60);
            const secs = Math.floor(currentPhaseTime % 60);
            timeInfo.textContent += ` (${mins}:${secs.toString().padStart(2, '0')})`;
        }

        // ==================== ГЛАВНЫЙ ЦИКЛ ====================
        const clock = new THREE.Clock();

        function animate() {
            requestAnimationFrame(animate);
            const dt = Math.min(clock.getDelta(), 0.1);

            if (cameraShake > 0) {
                cameraShake = Math.max(0, cameraShake - dt * 2);
                const shake = cameraShake * 0.3;
                camera.position.x += (Math.random() - 0.5) * shake;
                camera.position.z += (Math.random() - 0.5) * shake;
            }

            if (!gameOver && gameStarted && !paused) {
                const speed = 7.5;

                velocity.set(0, 0, 0);
                if (keys['KeyW']) velocity.z -= 1;
                if (keys['KeyS']) velocity.z += 1;
                if (keys['KeyA']) velocity.x -= 1;
                if (keys['KeyD']) velocity.x += 1;

                if (velocity.lengthSq() > 0) {
                    velocity.normalize();
                    velocity.applyEuler(new THREE.Euler(0, yaw, 0));
                    velocity.multiplyScalar(speed * dt);

                    const newPos = camera.position.clone().add(velocity);
                    resolveCollisions(newPos);
                    camera.position.x = newPos.x;
                    camera.position.z = newPos.z;
                }

                if (isShooting) shoot();

                updateDayNight(dt);
                enemies.forEach(enemy => enemy.update(camera.position, dt));
                updateBullets(dt);
            }

            renderer.render(scene, camera);
        }

        // ==================== UI ЛОГИКА ====================
        const mainMenu = document.getElementById('main-menu');
        const shopPanel = document.getElementById('shop-panel');
        const settingsPanel = document.getElementById('settings-panel');
        const pauseMenu = document.getElementById('pause-menu');
        const hud = document.getElementById('hud');
        const crosshair = document.getElementById('crosshair');
        const controls = document.getElementById('controls');
        const killFeedEl = document.getElementById('kill-feed');

        function updateShopCoinsBadge(flash = false) {
            const badge = document.getElementById('shop-coins-badge');
            const value = document.getElementById('shop-coins-value');
            if (!value) return;
            value.textContent = coins;
            if (flash) {
                badge.classList.remove('flash');
                void badge.offsetWidth;
                badge.classList.add('flash');
            }
        }

        function showMenu() {
            mainMenu.style.display = 'flex';
            shopPanel.style.display = 'none';
            settingsPanel.style.display = 'none';
            pauseMenu.style.display = 'none';
        }

        function startGame() {
            mainMenu.style.display = 'none';
            shopPanel.style.display = 'none';
            settingsPanel.style.display = 'none';
            pauseMenu.style.display = 'none';

            hud.style.display = 'block';
            crosshair.style.display = 'block';
            controls.style.display = 'block';
            killFeedEl.style.display = 'flex';

            gameStarted = true;
            gameOver = false;
            paused = false;
            health = maxHealth;
            score = 0;
            enemiesKilled = 0;
            waveNumber = 1;
            worldTime = 0;
            currentPeriod = 'day';
            isShooting = false;

            camera.position.set(0, 1.6, 0);
            yaw = 0;
            pitch = 0;
            camera.rotation.set(0, 0, 0);

            enemies.forEach(e => {
                scene.remove(e.mesh);
                scene.remove(e.lightRing);
            });
            enemies.length = 0;
            bullets.forEach(b => releaseBullet(b));
            bullets.length = 0;

            updateHealthUI();
            document.getElementById('score').innerText = "💀 Убито: 0";
            document.getElementById('coins').innerText = "💰 Монеты: " + coins;
            document.getElementById('wave-info').innerText = "🌊 Волна: 1";
            document.getElementById('gameover').style.display = 'none';

            enterFullscreen();
            document.body.requestPointerLock();
        }

        function togglePause() {
            paused = !paused;
            if (paused) {
                pauseMenu.style.display = 'flex';
                isShooting = false;
                for (const k in keys) keys[k] = false;
                if (document.pointerLockElement) document.exitPointerLock();
            } else {
                pauseMenu.style.display = 'none';
                document.body.requestPointerLock();
            }
        }

        pauseMenu.addEventListener('click', (e) => e.stopPropagation());

        document.getElementById('btn-play').addEventListener('click', startGame);
        document.getElementById('btn-shop').addEventListener('click', () => {
            mainMenu.style.display = 'none';
            shopPanel.style.display = 'flex';
            updateShopCoinsBadge();
            renderShop();
        });
        document.getElementById('btn-settings').addEventListener('click', () => {
            mainMenu.style.display = 'none';
            settingsPanel.style.display = 'flex';
        });

        document.getElementById('shop-back').addEventListener('click', showMenu);
        document.getElementById('settings-back').addEventListener('click', showMenu);

        document.getElementById('btn-resume').addEventListener('click', togglePause);
        document.getElementById('btn-to-menu').addEventListener('click', () => {
            paused = false;
            gameStarted = false;
            isShooting = false;
            pauseMenu.style.display = 'none';
            hud.style.display = 'none';
            crosshair.style.display = 'none';
            controls.style.display = 'none';
            killFeedEl.style.display = 'none';
            nightOverlay.style.opacity = '0';
            showMenu();
        });

        // ==================== МАГАЗИН ====================
        const shopContent = document.getElementById('shop-content');

        function renderShop() {
            shopContent.innerHTML = '';

            BULLET_SKINS.forEach(skin => {
                const item = document.createElement('div');
                item.className = 'shop-item' + (skin.id === activeSkinId ? ' active-skin' : '');

                const previewHTML = `
                    <div class="skin-preview" style="background: ${skin.cssColor}; color: ${skin.cssColor};"></div>
                `;

                let btnHTML;
                const isOwned = ownedSkins.has(skin.id);
                const isActive = skin.id === activeSkinId;
                const isDefault = skin.id === 'default';

                if (isActive) {
                    btnHTML = `<button class="buy-btn equipped" disabled>✓ Надет</button>`;
                } else if (isOwned || isDefault) {
                    btnHTML = `<button class="buy-btn" data-id="${skin.id}" data-action="equip">Надеть</button>`;
                } else {
                    btnHTML = `<button class="buy-btn" data-id="${skin.id}" data-action="buy" data-price="${skin.price}">${skin.price} 💰</button>`;
                }

                const priceLabel = isDefault ? 'Бесплатно' : `${skin.price} 💰`;

                item.innerHTML = `
                    ${previewHTML}
                    <div class="shop-item-info" style="flex:1;">
                        <h3>${skin.name}</h3>
                        <p>${skin.desc} • ${priceLabel}</p>
                    </div>
                    ${btnHTML}
                `;
                shopContent.appendChild(item);
            });

            shopContent.querySelectorAll('.buy-btn').forEach(btn => {
                if (btn.disabled) return;
                btn.addEventListener('click', () => {
                    const id = btn.dataset.id;
                    const action = btn.dataset.action;

                    if (action === 'buy') {
                        const price = parseInt(btn.dataset.price);
                        if (coins < price) {
                            const orig = btn.textContent;
                            btn.textContent = 'Мало 💰';
                            btn.style.background = 'linear-gradient(135deg, #aa2222, #661111)';
                            setTimeout(() => {
                                btn.textContent = orig;
                                btn.style.background = '';
                            }, 1000);
                            return;
                        }
                        coins -= price;
                        ownedSkins.add(id);
                        activeSkinId = id;
                        document.getElementById('coins').innerText = "💰 Монеты: " + coins;
                        updateShopCoinsBadge(true);
                        renderShop();
                    } else if (action === 'equip') {
                        activeSkinId = id;
                        renderShop();
                    }
                });
            });
        }

        // ==================== НАСТРОЙКИ ====================
        const sensInput = document.getElementById('sens');
        const sensVal = document.getElementById('sens-val');
        sensInput.addEventListener('input', () => {
            settings.sensitivity = parseFloat(sensInput.value);
            sensVal.textContent = settings.sensitivity.toFixed(1);
        });

        const volInput = document.getElementById('vol');
        const volVal = document.getElementById('vol-val');
        volInput.addEventListener('input', () => {
            settings.volume = parseInt(volInput.value);
            volVal.textContent = settings.volume;
        });

        const qualityBtns = document.querySelectorAll('.quality-btn');
        qualityBtns.forEach(btn => {
            btn.addEventListener('click', () => {
                qualityBtns.forEach(b => b.classList.remove('active'));
                btn.classList.add('active');
                settings.quality = parseInt(btn.dataset.q);
                applyQuality();
            });
        });

        function applyQuality() {
            renderer.setPixelRatio(Math.min(window.devicePixelRatio, settings.quality === 2 ? 2 : 1));
            renderer.shadowMap.enabled = settings.quality > 0;

            sunLight.castShadow = settings.quality > 0;
            sunLight.shadow.mapSize.width = settings.quality === 2 ? 2048 : 1024;
            sunLight.shadow.mapSize.height = settings.quality === 2 ? 2048 : 1024;
            if (sunLight.shadow.map) {
                sunLight.shadow.map.dispose();
                sunLight.shadow.map = null;
            }

            populateWorld();
        }

        const fogInput = document.getElementById('fog');
        const fogVal = document.getElementById('fog-val');
        fogInput.addEventListener('input', () => {
            const v = parseInt(fogInput.value);
            fogVal.textContent = v;
            settings.fogDensity = 0.001 + (v / 100) * 0.025;
            scene.fog.density = settings.fogDensity;
        });

        function setupToggle(id, key, onChange) {
            const el = document.getElementById(id);
            el.addEventListener('click', () => {
                el.classList.toggle('on');
                settings[key] = el.classList.contains('on');
                if (onChange) onChange(settings[key]);
            });
        }

        setupToggle('toggle-details', 'details', () => populateWorld());
        setupToggle('toggle-lights', 'dynamicLights', (on) => {
            hemiLight.visible = on;
            sunLight.visible = on;
            ambientLight.intensity = on ? 0.5 : 0.8;
        });
        setupToggle('toggle-invert', 'invertY');
        setupToggle('toggle-shake', 'screenShake');
        setupToggle('toggle-daynight', 'dayNightCycle', (on) => {
            if (!on) {
                scene.background.copy(daySky);
                scene.fog.color.copy(daySky);
                sunLight.intensity = 1.0;
                sunLight.color.setHex(0xffffff);
                moonLight.intensity = 0;
                ambientLight.intensity = 0.5;
                hemiLight.intensity = 0.6;
                stars.material.opacity = 0;
                nightOverlay.style.opacity = 0;
                timeInfo.textContent = '☀️ День';
                timeInfo.style.color = '#ffdd88';
                hudEl.style.borderLeftColor = '#66aaff';
            }
        });

        // ==================== СТАРТ ====================
        animate();
        setTimeout(spawnEnemy, 3000);

        window.addEventListener('resize', () => {
            camera.aspect = window.innerWidth / window.innerHeight;
            camera.updateProjectionMatrix();
            renderer.setSize(window.innerWidth, window.innerHeight);
        });

        renderer.domElement.addEventListener('click', () => {
            if (gameStarted && !gameOver && !paused && !document.pointerLockElement) {
                document.body.requestPointerLock();
            }
        });
    </script>
</body>
</html>
