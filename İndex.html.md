<!DOCTYPE html>  
<html lang="tr">  
<head>  
    <meta charset="UTF-8">  
    <meta name="viewport" content="width=device-width, initial-scale=1.0">  
    <title>İklim Life | Personel Yönetim Sistemi</title>  
    <script src="https://cdnjs.cloudflare.com/ajax/libs/xlsx/0.18.5/xlsx.full.min.js"></script>  
    <script src="https://unpkg.com/html5-qrcode"></script>  
    <style>  
        :root {  
            --primary: #1e293b;  
            --accent: #f59e0b;  
            --success: #10b981;  
            --danger: #ef4444;  
            --info: #3b82f6;  
            --bg-gray: #f8fafc;  
        }  
        body { font-family: 'Inter', sans-serif; background: var(--bg-gray); margin: 0; display: flex; justify-content: center; min-height: 100vh; }  
        .app-container { width: 100%; max-width: 440px; background: white; min-height: 100vh; box-shadow: 0 10px 15px rgba(0,0,0,0.1); display: flex; flex-direction: column; }  
        .nav-header { display: flex; background: var(--primary); padding: 15px 10px; gap: 10px; }  
        .nav-btn { flex: 1; padding: 10px; border: none; background: rgba(255,255,255,0.1); color: white; border-radius: 8px; font-weight: 600; cursor: pointer; }  
        .nav-btn.active { background: white; color: var(--primary); }  
        .content { padding: 25px; flex: 1; }  
        .hidden { display: none !important; }  
        input { width: 100%; padding: 16px; border: 2px solid #e2e8f0; border-radius: 12px; font-size: 1.1rem; margin-bottom: 15px; box-sizing: border-box; text-align: center; }  
        .btn-main { width: 100%; padding: 16px; border: none; border-radius: 12px; font-weight: 700; cursor: pointer; }  
        .action-grid { display: grid; grid-template-columns: 1fr 1fr; gap: 15px; margin-top: 10px; }  
        .btn-act { padding: 20px 10px; border-radius: 15px; border: none; color: white; font-weight: 700; display: flex; flex-direction: column; align-items: center; gap: 8px; cursor: pointer; }  
        .bg-work { background: var(--info); }  
        .bg-break { background: var(--success); }  
        .bg-off { background: var(--primary); }  
        .bg-mola-off { background: var(--accent); }  
        #reader { width: 100%; border-radius: 20px; overflow: hidden; border: 4px solid var(--accent); }  
        table { width: 100%; border-collapse: collapse; font-size: 0.8rem; margin-top: 10px; }  
        th, td { padding: 10px; border-bottom: 1px solid #eee; text-align: left; }  
    </style>  
</head>  
<body>  
  
<div class="app-container">  
    <div class="nav-header">  
        <button id="t-staff" class="nav-btn active" onclick="showView('staff')">PERSONEL</button>  
        <button id="t-admin" class="nav-btn" onclick="showView('admin')">YÖNETİCİ</button>  
    </div>  
  
    <div class="content">  
        <div id="view-login">  
            <h2 style="text-align:center">Giriş Yap</h2>  
            <input type="password" id="pinCode" placeholder="••••" maxlength="4" style="font-size:2.5rem; letter-spacing:15px;">  
            <button class="btn-main" style="background:var(--primary); color:white;" onclick="processLogin()">Giriş</button>  
        </div>  
  
        <div id="view-panel" class="hidden">  
            <h2 id="userName" style="text-align:center">Merhaba</h2>  
            <div class="action-grid">  
                <button class="btn-act bg-work" onclick="triggerQR('İşe Giriş')">🚀<br>İşe Başla</button>  
                <button class="btn-act bg-off" onclick="triggerQR('İşten Çıkış')">🏠<br>Mesai Bitir</button>  
                <button class="btn-act bg-break" onclick="triggerQR('Mola Başlat')">☕<br>Mola Başlat</button>  
                <button class="btn-act bg-mola-off" onclick="triggerQR('Mola Bitir')">✅<br>Mola Bitir</button>  
            </div>  
            <button class="btn-main" style="background:#f1f5f9; color:#475569; margin-top:30px;" onclick="secureLogout()">Güvenli Çıkış</button>  
        </div>  
  
        <div id="view-qr" class="hidden">  
            <h2 style="text-align:center; color:var(--accent)">QR Doğrulama</h2>  
            <div id="reader"></div>  
            <button class="btn-main" style="background:var(--danger); color:white; margin-top:20px;" onclick="cancelAction()">İptal Et</button>  
        </div>  
  
        <div id="view-admin" class="hidden">  
            <div id="admin-gate">  
                <h2 style="text-align:center">Yönetici Girişi</h2>  
                <input type="password" id="adminKey" placeholder="Şifre">  
                <button class="btn-main" style="background:var(--primary); color:white;" onclick="unlockAdmin()">Paneli Aç</button>  
            </div>  
            <div id="admin-data" class="hidden">  
                <div style="display:flex; gap:10px; margin-bottom:20px;">  
                    <button class="btn-main" style="background:var(--success); color:white;" onclick="exportExcel()">Excel Al</button>  
                    <button class="btn-main" style="background:var(--danger); color:white;" onclick="wipeData()">Sıfırla</button>  
                </div>  
                <div style="overflow-x:auto">  
                    <table>  
                        <thead><tr><th>Tarih</th><th>Personel</th><th>Eylem</th><th>Saat</th></tr></thead>  
                        <tbody id="logTableBody"></tbody>  
                    </table>  
                </div>  
                <button class="btn-main" style="background:#f1f5f9; color:#475569; margin-top:20px;" onclick="lockAdmin()">Çıkış</button>  
            </div>  
        </div>  
    </div>  
</div>  
  
<script>  
    const STORE = {  
        CODE: "MAGAZA-12345",  
        PASS: "1674",  
        STAFF: {   
            "0600": "Kübra", "6173": "Nefise", "7349": "Burcu",   
            "9506": "Sultan", "4672": "Derya", "8672": "Yasemin",   
            "1510": "Eda", "4209": "Özgün"   
        }  
    };  
  
    let currentUser = null;  
    let currentAction = null;  
    let logs = JSON.parse(localStorage.getItem('iklim_v3_data')) || [];  
    const scanner = new Html5Qrcode("reader");  
  
    function showView(viewName) {  
        document.getElementById('t-staff').classList.toggle('active', viewName === 'staff');  
        document.getElementById('t-admin').classList.toggle('active', viewName === 'admin');  
        ['view-login', 'view-panel', 'view-qr', 'view-admin'].forEach(id => document.getElementById(id).classList.add('hidden'));  
  
        if(viewName === 'staff') {  
            if(currentUser) document.getElementById('view-panel').classList.remove('hidden');  
            else document.getElementById('view-login').classList.remove('hidden');  
        } else {  
            document.getElementById('view-admin').classList.remove('hidden');  
            lockAdmin();  
        }  
    }  
  
    function processLogin() {  
        const pin = document.getElementById('pinCode').value;  
        if(STORE.STAFF[pin]) {  
            currentUser = STORE.STAFF[pin];  
            document.getElementById('userName').innerText = `Hoş Geldin, ${currentUser}`;  
            showView('staff');  
        } else { alert("Hatalı PIN!"); }  
    }  
  
    function secureLogout() {  
        currentUser = null;  
        document.getElementById('pinCode').value = "";  
        showView('staff');  
    }  
  
    function triggerQR(action) {  
        currentAction = action;  
        document.getElementById('view-panel').classList.add('hidden');  
        document.getElementById('view-qr').classList.remove('hidden');  
        scanner.start({ facingMode: "environment" }, { fps: 15, qrbox: 250 }, (text) => {  
            if(text === STORE.CODE) {  
                scanner.stop().then(() => {  
                    const d = new Date();  
                    logs.push({  
                        date: d.toLocaleDateString('tr-TR'),  
                        user: currentUser,  
                        type: currentAction,  
                        time: d.getHours().toString().padStart(2,'0') + ":" + d.getMinutes().toString().padStart(2,'0')  
                    });  
                    localStorage.setItem('iklim_v3_data', JSON.stringify(logs));  
                    alert("Kayıt Başarılı!");  
                    secureLogout();  
                });  
            }  
        }).catch(e => alert("Kamera Hatası!"));  
    }  
  
    function cancelAction() { scanner.stop().then(() => showView('staff')); }  
  
    function unlockAdmin() {  
        if(document.getElementById('adminKey').value === STORE.PASS) {  
            document.getElementById('admin-gate').classList.add('hidden');  
            document.getElementById('admin-data').classList.remove('hidden');  
            const body = document.getElementById('logTableBody');  
            body.innerHTML = logs.slice().reverse().map(l => `<tr><td>${l.date}</td><td>${l.user}</td><td>${l.type}</td><td>${l.time}</td></tr>`).join('');  
        } else { alert("Hatalı Şifre!"); }  
    }  
  
    function lockAdmin() {  
        document.getElementById('adminKey').value = "";  
        document.getElementById('admin-gate').classList.remove('hidden');  
        document.getElementById('admin-data').classList.add('hidden');  
    }  
  
    function exportExcel() {  
        const ws = XLSX.utils.json_to_sheet(logs);  
        const wb = XLSX.utils.book_new();  
        XLSX.utils.book_append_sheet(wb, ws, "Personel_Raporu");  
        XLSX.writeFile(wb, "Iklim_Life_Rapor.xlsx");  
    }  
  
    function wipeData() {  
        if(confirm("Tüm veriler silinsin mi?")) {  
            logs = [];  
            localStorage.setItem('iklim_v3_data', '[]');  
            unlockAdmin();  
        }  
    }  
</script>  
</body>  
</html>  
