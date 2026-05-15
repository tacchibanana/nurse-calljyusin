<!DOCTYPE html>
<html lang="ja">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>管理・受信ボード - nursecallAPP</title>
    <script src="https://cdnjs.cloudflare.com/ajax/libs/qrcodejs/1.0.0/qrcode.min.js"></script>
    <style>
        :root {
            --bg-color: #fdfaf5;
            --card-bg: #ffffff;
            --text-color: #3e3a39;
            --primary-color: #5d627b;
            --accent-red: #d9534f;
            --accent-orange: #f0ad4e;
            --accent-green: #5cb85c;
            --tab-bg: #e9e4d9;
            --border-color: #dcd6c8;
        }

        body.night-mode {
            --bg-color: #1a1a1a;
            --card-bg: #2d2d2d;
            --text-color: #e0e0e0;
            --tab-bg: #333333;
            --border-color: #444444;
            --primary-color: #8fa1cc;
        }

        body {
            font-family: "Helvetica Neue", Arial, sans-serif;
            background-color: var(--bg-color);
            color: var(--text-color);
            margin: 0;
            transition: background-color 0.3s;
        }

        /* --- ヘッダー --- */
        .header-bar {
            display: flex; justify-content: space-between; align-items: center;
            background: var(--tab-bg); padding: 5px 20px; border-bottom: 1px solid var(--border-color);
            position: sticky; top: 0; z-index: 1000;
        }
        .nav-icons { display: flex; gap: 10px; }
        .icon-btn {
            width: 45px; height: 45px; border-radius: 50%; border: none;
            background: transparent; cursor: pointer; font-size: 1.4rem;
            display: flex; align-items: center; justify-content: center;
            transition: background 0.2s; color: var(--text-color);
        }
        .icon-btn.active { background: var(--card-bg); box-shadow: 0 2px 5px rgba(0,0,0,0.1); }
        
        .header-controls { display: flex; align-items: center; gap: 10px; }
        
        .stop-alarm-btn {
            display: none; width: 45px; height: 45px; border-radius: 50%;
            background: #e60012; color: white; border: none; font-size: 1.4rem;
            cursor: pointer; animation: pulse 1s infinite alternate;
        }
        @keyframes pulse { from { transform: scale(1); } to { transform: scale(1.1); } }

        .night-toggle { 
            width: 45px; height: 45px; border-radius: 50%; border: 1px solid var(--border-color);
            background: var(--card-bg); color: var(--text-color); cursor: pointer;
            display: flex; align-items: center; justify-content: center; font-size: 1.2rem;
        }

        .content-section { display: none; padding: 20px; }
        .content-section.active { display: block; }

        .board { display: grid; grid-template-columns: repeat(auto-fill, minmax(280px, 1fr)); gap: 20px; }
        .card { background: var(--card-bg); padding: 20px; border-radius: 12px; box-shadow: 0 4px 15px rgba(0,0,0,0.05); border-left: 10px solid #ccc; }
        .card.urgent { border-left-color: var(--accent-red); }
        .card.normal { border-left-color: var(--accent-orange); }
        .card.consult { border-left-color: var(--accent-green); }
        .done-btn { width: 100%; margin-top: 15px; padding: 12px; background: var(--primary-color); color: white; border: none; border-radius: 8px; font-weight: bold; cursor: pointer; }

        /* --- 印刷用 --- */
        #print-area { display: none; }
        @media print {
            @page { size: A4 portrait; margin: 0; }
            .no-print { display: none !important; }
            #print-area { display: block !important; width: 210mm; height: 297mm; padding: 15mm; box-sizing: border-box; background: white; color: black; }
            .print-container { border: 2px solid #eee; height: 100%; display: flex; flex-direction: column; justify-content: space-between; padding: 10mm; box-sizing: border-box; }
            .step-num { background: #d9534f !important; -webkit-print-color-adjust: exact; color: white !important; }
            .instructions { padding: 25px; border-radius: 20px; text-align: left; background: #f9f9f9; border: 2px dashed #ccc; }
            .step { display: flex; align-items: center; margin-bottom: 20px; }
            .step-num { width: 45px; height: 45px; border-radius: 50%; display: flex; align-items: center; justify-content: center; font-size: 1.6rem; font-weight: bold; margin-right: 20px; }
            .step-text { font-size: 1.6rem; font-weight: bold; }
        }

        #start-overlay { position: fixed; top:0; left:0; width:100%; height:100%; background: var(--bg-color); display: flex; justify-content: center; align-items: center; z-index: 2000; }
        .start-btn { padding: 20px 40px; font-size: 1.5rem; border-radius: 50px; cursor: pointer; background: var(--primary-color); color: white; border: none; }
    </style>
</head>
<body>

<div id="print-area">
    <div class="print-container">
        <div style="text-align:center;">
            <p style="font-size: 1.4rem;">用件入力ナースコール</p>
            <h1 style="font-size:4.5rem; margin:10px 0;" id="out-room">---</h1>
            <p id="out-name" style="font-size: 2rem;"></p>
            <div id="qrcode" style="display:flex; justify-content:center; margin:30px 0;"></div>
            <div class="instructions">
                <h2 style="text-align:center; color:#d9534f; margin-bottom:25px;">つかいかた</h2>
                <div class="step"><div class="step-num">1</div><div class="step-text">カメラでQRを読み取る</div></div>
                <div class="step"><div class="step-num">2</div><div class="step-text">今の状況と用件をえらぶ</div></div>
                <div class="step"><div class="step-num">3</div><div class="step-text">「スタッフを呼ぶ」をおす</div></div>
            </div>
        </div>
    </div>
</div>

<div class="no-print">
    <div id="start-overlay">
        <button class="start-btn" onclick="startApp()">受信を開始する</button>
    </div>

    <div class="header-bar">
        <div class="nav-icons">
            <button class="icon-btn active" id="btn-board" onclick="switchTab('board-section', this)">📋</button>
            <button class="icon-btn" id="btn-qr" onclick="switchTab('qr-section', this)">➕</button>
        </div>
        <div class="header-controls">
            <button id="stop-alarm-btn" class="stop-alarm-btn" onclick="stopAlarm()">🔕</button>
            <button class="night-toggle" onclick="toggleNightMode()">🌙</button>
        </div>
    </div>

    <div id="board-section" class="content-section active">
        <div class="board" id="board"></div>
    </div>

    <div id="qr-section" class="content-section">
        <div style="max-width: 500px; margin: auto; background: var(--card-bg); padding: 30px; border-radius: 12px; box-shadow: 0 4px 10px rgba(0,0,0,0.05);">
            <h2>QRコード作成</h2>
            <input type="text" id="in-room" placeholder="部屋番号" style="width:100%; padding:10px; margin-bottom:10px;">
            <input type="text" id="in-name" placeholder="お名前（任意）" style="width:100%; padding:10px; margin-bottom:10px;">
            <input type="text" id="sender-url" value="https://nursecallapp-bac29.web.app" style="width:100%; padding:10px; margin-bottom:20px;">
            <button class="done-btn" onclick="preparePrint()" style="background:#28a745;">🖨️ A4サイズで印刷する</button>
        </div>
    </div>
</div>

<script type="module">
    import { initializeApp } from "https://www.gstatic.com/firebasejs/10.7.1/firebase-app.js";
    import { 
        getFirestore, collection, query, orderBy, onSnapshot, 
        deleteDoc, doc, limit, initializeFirestore 
    } from "https://www.gstatic.com/firebasejs/10.7.1/firebase-firestore.js";

    const firebaseConfig = {
        apiKey: "AIzaSyBeL7xd049PjR-Sl18C-EJ2HgCkjzKI0SM",
        authDomain: "nursecallapp-bac29.firebaseapp.com",
        projectId: "nursecallapp-bac29",
        storageBucket: "nursecallapp-bac29.firebasestorage.app",
        messagingSenderId: "690697442852",
        appId: "1:690697442852:web:75b17f1abc8b7b20f6aab1"
    };

    const app = initializeApp(firebaseConfig);
    const db = initializeFirestore(app, { experimentalForceLongPolling: true });

    const normalAudio = new Audio('https://assets.mixkit.co/active_storage/sfx/2869/2869-preview.mp3');
    const urgentAudio = new Audio('https://assets.mixkit.co/active_storage/sfx/995/995-preview.mp3');
    urgentAudio.loop = true;

    // --- UI用関数群 ---
    window.toggleNightMode = () => {
        document.body.classList.toggle('night-mode');
    };

    window.switchTab = (id, el) => {
        document.querySelectorAll('.content-section').forEach(s => s.classList.remove('active'));
        document.querySelectorAll('.icon-btn').forEach(b => b.classList.remove('active'));
        document.getElementById(id).classList.add('active');
        el.classList.add('active');
    };

    window.stopAlarm = () => {
        urgentAudio.pause(); urgentAudio.currentTime = 0;
        document.getElementById('stop-alarm-btn').style.display = 'none';
    };

    window.preparePrint = () => {
        const room = document.getElementById('in-room').value;
        if (!room) return alert("部屋番号を入力してください");
        document.getElementById('out-room').innerText = room;
        document.getElementById('out-name').innerText = document.getElementById('in-name').value;
        const qrContainer = document.getElementById('qrcode');
        qrContainer.innerHTML = "";
        new QRCode(qrContainer, { text: `${document.getElementById('sender-url').value}?room=${encodeURIComponent(room)}`, width: 250, height: 250 });
        setTimeout(() => window.print(), 600);
    };

    window.completeCall = async (id) => {
        if(confirm("対応完了として削除しますか？")) {
            await deleteDoc(doc(db, "calls", id));
            stopAlarm();
        }
    };

    // --- メイン受信処理 ---
    window.startApp = () => {
        document.getElementById('start-overlay').style.display = 'none';
        const q = query(collection(db, "calls"), orderBy("createdAt", "desc"), limit(300));
        
        onSnapshot(q, { includeMetadataChanges: false }, (snapshot) => {
            const board = document.getElementById('board');
            
            snapshot.docChanges().forEach((change) => {
                const data = change.doc.data();
                const id = change.doc.id;

                if (change.type === "added") {
                    const card = document.createElement('div');
                    card.className = `card ${data.priority}`;
                    card.id = `card-${id}`;
                    card.innerHTML = `
                        <div style="font-size:1.6rem; font-weight:bold;">${data.room}</div>
                        <div style="margin:10px 0; font-size:1.2rem;">${data.menu.join(' / ')}</div>
                        <button class="done-btn" onclick="completeCall('${id}')">対応完了</button>
                    `;
                    // 常に最新が上に来るように挿入
                    board.insertBefore(card, board.firstChild);

                    if (data.priority === 'urgent') {
                        urgentAudio.play().catch(() => {});
                        document.getElementById('stop-alarm-btn').style.display = 'block';
                    } else {
                        normalAudio.play().catch(() => {});
                    }
                }
                if (change.type === "removed") {
                    const card = document.getElementById(`card-${id}`);
                    if (card) card.remove();
                }
            });
        });
    };
</script>
</body>
</html>