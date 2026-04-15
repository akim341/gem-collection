<!DOCTYPE html>
<html>
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Ultimate Miner Tycoon</title>
    <style>
        * { box-sizing: border-box; }
        body { background: #0d0d12; color: #e0e0e0; font-family: 'Segoe UI', sans-serif; text-align: center; margin: 0; padding: 10px; }
        .container { max-width: 500px; margin: 0 auto; }
        
        .card { background: #1a1a24; padding: 25px; border-radius: 20px; border: 1px solid #333; margin-top: 20px; }
        input { width: 100%; padding: 12px; margin: 8px 0; border-radius: 10px; border: 1px solid #444; background: #08080a; color: white; }
        button { width: 100%; background: #ffd700; color: #000; border: none; padding: 14px; border-radius: 12px; cursor: pointer; font-weight: bold; margin: 5px 0; font-size: 16px; }
        button:disabled { background: #333; color: #777; cursor: not-allowed; }
        
        .stat-box { background: #111; padding: 15px; border-radius: 15px; border: 2px solid #ffd700; margin: 20px 0; font-size: 28px; font-weight: bold; color: #ffd700; }
        
        .shop-grid { display: grid; grid-template-columns: 1fr; gap: 15px; margin-top: 20px; }
        .item { background: #22222e; padding: 15px; border-radius: 15px; border: 1px solid #333; display: flex; align-items: center; justify-content: space-between; text-align: left; }
        .item img { width: 50px; height: 50px; border-radius: 8px; background: #000; margin-right: 15px; }
        .item-info { flex-grow: 1; }
        .item-info b { font-size: 18px; display: block; }
        .item-info span { font-size: 13px; color: #aaa; }
        .buy-btn { width: 100px; padding: 8px; font-size: 12px; }

        #inventory { display: flex; flex-wrap: wrap; gap: 8px; justify-content: center; margin: 15px 0; }
        .inv-badge { background: #333; padding: 4px 12px; border-radius: 15px; font-size: 12px; border: 1px solid #555; }
        
        .logout-btn { background: none; color: #666; font-size: 12px; text-decoration: underline; margin-top: 20px; }
    </style>
</head>
<body>

<div class="container">
    <div id="auth">
        <div class="card">
            <h2 style="color: #ffd700;">MINER LOGIN</h2>
            <input type="text" id="u" placeholder="Логін">
            <input type="password" id="p" placeholder="Пароль">
            <button onclick="doRegister()">СТВОРИТИ АКАУНТ</button>
            <button onclick="doLogin()" style="background: #fff;">УВІЙТИ</button>
        </div>
    </div>

    <div id="game" style="display:none;">
        <div id="user-display" style="font-size: 14px; color: #888; margin-bottom: 5px;"></div>
        <div class="stat-box">💰 <span id="money">0</span></div>
        
        <div id="inventory"></div>

        <div class="shop-grid">
            <div class="item">
                <img src="img/stone1.png" onerror="this.src='https://via.placeholder.com/50?text=Opal'">
                <div class="item-info">
                    <b>Опал</b>
                    <span>Ціна: 100 | +5/сек</span>
                </div>
                <button id="btn1" class="buy-btn" onclick="buyItem('Опал', 100, 5, 'btn1')">КУПИТИ</button>
            </div>

            <div class="item">
                <img src="img/stone2.png" onerror="this.src='https://via.placeholder.com/50?text=Diam'">
                <div class="item-info">
                    <b>Діамант</b>
                    <span>Ціна: 1,000 | +50/сек</span>
                </div>
                <button id="btn2" class="buy-btn" onclick="buyItem('Діамант', 1000, 50, 'btn2')">КУПИТИ</button>
            </div>

            <div class="item">
                <img src="img/stone3.png" onerror="this.src='https://via.placeholder.com/50?text=Rubin'">
                <div class="item-info">
                    <b>Рубін</b>
                    <span>Ціна: 10,000 | +600/сек</span>
                </div>
                <button id="btn3" class="buy-btn" onclick="buyItem('Рубін', 10000, 600, 'btn3')">КУПИТИ</button>
            </div>
        </div>

        <button onclick="doLogout()" class="logout-btn">Вийти з акаунта</button>
    </div>
</div>

<script>
    let curUser = null;
    const dbKey = 'miner_pro_db';

    function getDB() { return JSON.parse(localStorage.getItem(dbKey) || '{}'); }
    function saveDB(db) { localStorage.setItem(dbKey, JSON.stringify(db)); }

    function doRegister() {
        const u = document.getElementById('u').value.trim();
        const p = document.getElementById('p').value.trim();
        if(!u || !p) return;
        let db = getDB();
        if(db[u]) return alert('Логін зайнятий');
        db[u] = { pass: p, money: 0, perSec: 1, items: ["Вугілля"] };
        saveDB(db);
        alert('Успіх!');
    }

    function doLogin() {
        const u = document.getElementById('u').value.trim();
        const p = document.getElementById('p').value.trim();
        let db = getDB();
        if(db[u] && db[u].pass === p) {
            localStorage.setItem('miner_session', u);
            start(u);
        }
    }

    function start(u) {
        curUser = u;
        document.getElementById('auth').style.display = 'none';
        document.getElementById('game').style.display = 'block';
        document.getElementById('user-display').innerText = "Гравець: " + u;
        refresh();
    }

    function refresh() {
        let db = getDB();
        let data = db[curUser];
        document.getElementById('money').innerText = Math.floor(data.money);
        
        const inv = document.getElementById('inventory');
        inv.innerHTML = "";
        data.items.forEach(it => { inv.innerHTML += `<div class="inv-badge">${it}</div>`; });

        updateBtn('btn1', 100, 'Опал', data);
        updateBtn('btn2', 1000, 'Діамант', data);
        updateBtn('btn3', 10000, 'Рубін', data);
    }

    function updateBtn(id, price, name, data) {
        const btn = document.getElementById(id);
        if(data.items.includes(name)) {
            btn.innerText = "ВЖЕ Є";
            btn.disabled = true;
            btn.style.background = "#222";
        } else {
            btn.disabled = data.money < price;
            btn.style.background = btn.disabled ? "#333" : "#ffd700";
        }
    }

    function buyItem(name, price, power) {
        let db = getDB();
        let data = db[curUser];
        if(data.money >= price && !data.items.includes(name)) {
            data.money -= price;
            data.items.push(name);
            data.perSec += power;
            saveDB(db);
            refresh();
        }
    }

    setInterval(() => {
        if(!curUser) return;
        let db = getDB();
        let data = db[curUser];
        data.money += data.perSec / 10;
        saveDB(db);
        document.getElementById('money').innerText = Math.floor(data.money);
        if(Math.floor(data.money * 10) % 50 === 0) refresh();
    }, 100);

    function doLogout() {
        localStorage.removeItem('miner_session');
        location.reload();
    }

    window.onload = () => {
        let s = localStorage.getItem('miner_session');
        if(s) start(s);
    };
</script>

</body>
</html>
