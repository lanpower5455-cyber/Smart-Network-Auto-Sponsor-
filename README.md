<!DOCTYPE html>
<html lang="ms">
<head>
<meta charset="UTF-8">
<title>SMART NETWORK AUTO SYSTEM</title>

<script src="https://www.gstatic.com/firebasejs/10.8.0/firebase-app-compat.js"></script>
<script src="https://www.gstatic.com/firebasejs/10.8.0/firebase-database-compat.js"></script>
<script src="https://www.gstatic.com/firebasejs/10.8.0/firebase-auth-compat.js"></script>

<style>
body{ font-family:Arial; background:#020617; color:white; margin:0; }
.container{ max-width:900px; margin:auto; padding:20px; }
h1{text-align:center;}
.banner{ text-align:center; line-height:1.6; margin-bottom:20px; }
.card{ background:white; color:#111; padding:20px; margin-top:20px; border-radius:10px; }
input,button{ width:100%; padding:10px; margin-top:10px; }
button{ background:#2563eb; color:white; border:none; cursor:pointer; }
button:hover{opacity:0.9;}
.box{ padding:10px; margin-top:10px; border-radius:8px; background:#e2e8f0; }
.green{background:#dcfce7;}
.red{background:#fecaca;}
.linkBox{ font-size:12px; margin-top:5px; }
.waBtn{ display:inline-block; margin-top:5px; padding:5px 10px; background:#22c55e; color:white; text-decoration:none; border-radius:5px; font-size:12px; }
.toggleBtn{ margin-top:10px; background:#0ea5e9; }
/* Admin Tree */
#adminPanel { display: none; border: 4px solid red; margin-top: 30px; }
.tree-node { margin-left: 20px; cursor: pointer; color: #1e293b; font-size: 14px; }
.tree-content { display: none; border-left: 1px dashed #94a3b8; margin-left: 10px; padding-left: 10px; }
.open > .tree-content { display: block; }
.has-kids { font-weight: bold; color: #2563eb; }
.has-kids::before { content: "▶ "; font-size: 10px; }
.open > .has-kids::before { content: "▼ "; }
/* Loading */
.loader { border: 3px solid #cbd5e1; border-top: 3px solid #2563eb; border-radius: 50%; width: 16px; height: 16px; animation: spin 1s linear infinite; display: inline-block; margin-right: 8px; vertical-align: middle; }
@keyframes spin { 0% { transform: rotate(0deg); } 100% { transform: rotate(360deg); } }
.loading-text { font-size: 13px; color: #64748b; font-style: italic; }
</style>
</head>

<body>
<div class="container">
<h1>SMART NETWORK AUTO SYSTEM</h1>
<div class="banner">
<strong>WORK SMART</strong><br><em>Trust The System</em><br><br>
Tambah penajaan (sponsor) secara automatik dengan usaha minimum.<br>
Cari sekurang-kurangnya 2 rakan yang serius untuk projek pendapatan pasif ini.<br><br>
<strong>TUNGGU HASILNYA</strong><br>
Sponsor anda akan bertambah secara automatik.
</div>

<div class="card">
<h3>LEADER</h3>
<div id="leaderBox"><div class="box"><div class="loader"></div> <span class="loading-text">Menghubungi server...</span></div></div>

<h3>UPLINE</h3>
<div id="uplineBox"><div class="box"><div class="loader"></div> <span class="loading-text">Menyemak rangkaian...</span></div></div>
</div>

<div class="card">
<h2>Daftar Peserta</h2>
<form id="form">
<input id="name" placeholder="Nama penuh" required>
<input id="phone" placeholder="Telefon (601...)" required>
<button type="submit" id="btnDaftar">Daftar</button>
</form>
<div id="result"></div>
</div>

<div class="card">
<h3>Referral Link Anda</h3>
<input id="reflink" readonly>
</div>

<div class="card">
<h3>Senarai Peserta</h3>
<div id="memberList"></div>
</div>

<div class="card">
<h3>LEADER AUTO SPONSOR</h3>
<div class="box">Jumlah Auto Sponsor: <b id="totalAutoSponsor">0</b></div>
<button id="toggleAutoBtn" class="toggleBtn" onclick="toggleAutoList()">Lihat Semua</button>
<div id="autoSponsorList"></div>
</div>

<div class="card" id="adminPanel">
<h3 style="color:red; text-align:center;">ADMIN PANEL</h3>
<div class="box">
<strong>🌳 Genealogy Tree (Klik nama untuk buka)</strong>
<div id="adminTreeDisplay" style="background:white; padding:10px; border-radius:5px; margin-top:10px; color:#111;">Memuatkan...</div>
</div>
<button onclick="resetSystem()" style="background:#dc2626; margin-top:10px;">Reset Semua Data</button>
</div>

<div class="card" id="leaderPanel" style="display:none; border: 4px solid #0ea5e9; margin-top: 30px;">
<h3 style="color:#0ea5e9; text-align:center;">LEADER PANEL</h3>
<div class="box" id="leaderStatsDisplay" style="background:#f0f9ff; border:1px solid #0ea5e9; margin-bottom:15px;"></div>
<div class="box">
<strong>🌳 Rangkaian Downline Anda (Klik nama untuk buka)</strong>
<div id="leaderTreeDisplay" style="background:white; padding:10px; border-radius:5px; margin-top:10px; color:#111;">Memuatkan...</div>
</div>
<button onclick="location.reload()" style="background:#64748b; margin-top:10px;">Tutup / Log Keluar</button>
</div>

<div style="padding: 0 20px;">
<button onclick="openLeaderLogin()" style="background:#0ea5e9; font-weight:bold;">🔍 SEMAK RANGKAIAN LEADER</button>
</div>
</div>

<script>
// ====================== CONFIG FIREBASE ======================
const firebaseConfig = {
  apiKey: "AIzaSyDkopModA3tIqLK1GYZfUZiq9AH4vuR-8w",
  authDomain: "work-smart-network.firebaseapp.com",
  databaseURL: "https://work-smart-network-default-rtdb.firebaseio.com",
  projectId: "work-smart-network",
  storageBucket: "work-smart-network.firebasestorage.app",
  messagingSenderId: "73259137430",
  appId: "1:73259137430:web:c3f44c576e4f438ce6c5b7"
};
firebase.initializeApp(firebaseConfig);

const auth = firebase.auth(); 
const db = firebase.database();
const ADMIN_PHONE = "601154226496"; 
const HQ_PHONE = "601154226496";

let members = [];
let showAllAuto = false;
let showAllMembers = false; // Status untuk senarai utama
let params = new URLSearchParams(location.search)
let ref = params.get("ref")

let BASE_LEADER = "Admin"
let BASE_PHONE = "601154226496"
let BASE_UPLINE = "Admin"
let BASE_UPLINE_PHONE = "601154226496"

let currentLeader = BASE_LEADER
let currentLeaderPhone = BASE_PHONE
let currentUpline = BASE_UPLINE
let currentUplinePhone = BASE_UPLINE_PHONE

// ====================== UTILITY FUNCTIONS ======================
function normalizePhone(phone){
  phone = phone.replace(/\D/g,'');
  if(phone.startsWith("0")) return "6"+phone;
  return phone;
}

function waLink(phone){ return "https://wa.me/" + phone; }
function generateRefLink(id){ return location.origin + location.pathname + "?ref=" + id; }
function toggleAutoList(){ showAllAuto = !showAllAuto; renderAutoSponsor(); }
function toggleMemberList(){ showAllMembers = !showAllMembers; renderMembers(); }

// ====================== ADMIN PANEL CHECK ======================
function checkAdminPanel() {
  if(normalizePhone(currentLeaderPhone) === ADMIN_PHONE) {
    document.getElementById("adminPanel").style.display = "block";
    renderAdminTree();
  } else {
    document.getElementById("adminPanel").style.display = "none";
  }
}

// ====================== RENDER MEMBERS ======================
function renderMembers(){
  let list = document.getElementById("memberList");
  let filtered = members.filter(m => m.currentLeader === currentLeader);
  
  let totalCount = filtered.length;
  let summaryHTML = `<div class="box" style="font-weight:bold; background:#f1f5f9;">Jumlah Keseluruhan Peserta: ${totalCount}</div>`;

  if(totalCount === 0){
    list.innerHTML = summaryHTML + "<div class='box'>Tiada peserta lagi</div>";
    renderAutoSponsor();
    checkAdminPanel();
    return;
  }

  // Susun peserta mengikut pendaftaran terkini di atas
  let sortedMembers = filtered.slice().reverse();
  
  // Hadkan paparan kepada 2 jika tidak klik 'Lihat Semua'
  let displayList = showAllMembers ? sortedMembers : sortedMembers.slice(0, 2);

  let itemsHTML = displayList.map((m) => {
    // Cari nombor pendaftaran asal dalam array filtered (mengikut urutan masa)
    let originalIndex = filtered.findIndex(x => x.id === m.id) + 1;
    
    return `<div class="box">
      <b>Peserta Ke-${originalIndex}: ${m.name}</b><br>
      Telefon: ${m.phone}<br>
      Sponsor: ${m.sponsor}<br>
      <a class="waBtn" href="${waLink(m.phone)}" target="_blank">WhatsApp</a>
      <div class="linkBox">${generateRefLink(m.id)}</div>
    </div>`;
  }).join("");

  // Butang lihat semua jika melebihi 2 orang
  let toggleBtn = totalCount > 2 ? 
    `<button class="toggleBtn" onclick="toggleMemberList()">${showAllMembers ? 'Sembunyikan' : 'Lihat Semua Peserta'}</button>` : '';

  list.innerHTML = summaryHTML + itemsHTML + toggleBtn;

  renderAutoSponsor();
  checkAdminPanel();
}

function renderAutoSponsor(){
  let list = document.getElementById("autoSponsorList")
  let filtered = members.filter(m => m.sponsor === currentLeader)
  document.getElementById("totalAutoSponsor").innerText = filtered.length

  if(filtered.length === 0){
    list.innerHTML = "<div class='box'>Tiada auto sponsor lagi</div>";
    document.getElementById("toggleAutoBtn").style.display = "none";
    return;
  }

  // Susun peserta mengikut pendaftaran terkini di atas
  let sortedAuto = filtered.slice().reverse();

  // Hadkan paparan kepada 4 jika tidak klik 'Lihat Semua'
  let displayList = showAllAuto ? sortedAuto : sortedAuto.slice(0, 2);

  list.innerHTML = displayList.map((m) => {
    let originalIndex = filtered.findIndex(x => x.id === m.id) + 1;
    return `<div class="box green">
      <b>Sponsor Ke-${originalIndex}: ${m.name}</b><br>
      Telefon: ${m.phone}<br>
      Sponsor: ${m.sponsor}<br>
      <a class="waBtn" href="${waLink(m.phone)}" target="_blank">WhatsApp</a>
      <div class="linkBox">${generateRefLink(m.id)}</div>
    </div>`;
  }).join("");

  let btn = document.getElementById("toggleAutoBtn");
  btn.style.display = filtered.length > 2 ? "block" : "none";
  btn.innerText = showAllAuto ? "Sembunyikan" : "Lihat Semua";
}

// ====================== TREE RENDERING ======================
function renderAdminTree() {
  const display = document.getElementById("adminTreeDisplay");
  function build(parent){
    let kids = members.filter(m => m.sponsor === parent);
    if(kids.length === 0) return "";
    let h = '<div class="tree-content">';
    kids.forEach(k => {
      let hasD = members.some(m => m.sponsor === k.name);
      h += `<div class="tree-node" onclick="event.stopPropagation(); this.classList.toggle('open')">
          <span class="${hasD ? 'has-kids' : ''}">${k.name}</span>
          ${build(k.name)}
      </div>`;
    });
    h += '</div>'; return h;
  }
  display.innerHTML = `<div class="tree-node open" onclick="this.classList.toggle('open')"><span class="has-kids">🏠 HQ</span>${build("Admin")}</div>`;
}

function renderLeaderTree(leaderName) {
  const display = document.getElementById("leaderTreeDisplay");
  function build(parent){
    let kids = members.filter(m => m.sponsor === parent);
    if(kids.length === 0) return "";
    let h = '<div class="tree-content">';
    kids.forEach(k => {
      let hasD = members.some(m => m.sponsor === k.name);
      h += `<div class="tree-node" onclick="event.stopPropagation(); this.classList.toggle('open')">
          <span class="${hasD ? 'has-kids' : ''}">${k.name}</span>
          ${build(k.name)}
      </div>`;
    });
    h += '</div>'; return h;
  }
  display.innerHTML = `<div class="tree-node open" onclick="this.classList.toggle('open')">
    <span class="has-kids">⭐ ${leaderName}</span>${build(leaderName)}
  </div>`;
}

// ====================== FORM SUBMISSION ======================
function genID() {
    const chars = '0123456789ABCDEFGHIJKLMNOPQRSTUVWXYZ';
    let result = '';
    for (let i = 0; i < 6; i++) {
        result += chars.charAt(Math.floor(Math.random() * chars.length));
    }
    return "M-" + result;
}
document.getElementById("form").addEventListener("submit", function(e){
  e.preventDefault();
// Kod baru yang sudah melalui 'tapis':
let rawName = document.getElementById('name').value;
let rawPhone = document.getElementById('phone').value;

// Kita panggil fungsi cuci yang kita buat tadi
let dataSiap = cuciInput(rawName, rawPhone);

// Sekarang pembolehubah kita adalah:
let name = dataSiap.namaBersih;
let phone = dataSiap.telBersih;

// --- GANTIKAN BARIS 268 DENGAN INI ---
  let wujud = members.find(m => normalizePhone(m.phone) === normalizePhone(phone));

  if (wujud) {
      alert("MAAF! Nombor " + phone + " sudah berdaftar di bawah sponsor: " + wujud.sponsor);
      return; 
  }

  let sameLeader = members.filter(m => m.currentLeader === currentLeader);
  let sponsorPhone = sameLeader.length === 0 ? currentUplinePhone : currentLeaderPhone;
  let sponsor = members.find(m => m.phone === sponsorPhone)?.name || (sameLeader.length === 0 ? currentUpline : currentLeader);

  let newId = genID();
  let newMember = {
    id: newId,
    name, phone,
    currentLeader, sponsor,
    uplineName: sponsor,
    uplinePhone: sponsorPhone
  };

db.ref("members/" + newId).set(newMember).then(() => {
    let link = generateRefLink(newId);
    
    // Link WhatsApp Sponsor dengan teks siap sedia
    let linkWaSponsor = `https://wa.me/${sponsorPhone}?text=${encodeURIComponent(`Salam Leader ${sponsor.toUpperCase()}, saya ${name.toUpperCase()} (ID: ${newId}). Saya sudah aktif, mohon arahan seterusnya!`)}`;

    // RENDER AYAT WHATSAPP (VERSI "BUTTON-STYLE" PnC)
    const teksWhatsApp = 
        `*PENGESAHAN SMART NETWORK* 🏆\n\n` +
        `*STATUS:* ✅ *AKAUN LEADER AKTIF*\n\n` +
        `👤 *NAMA:* ${name.toUpperCase()}\n` +
        `🆔 *ID AHLI:* ${newId}\n` +
        `🤝 *SPONSOR:* ${sponsor.toUpperCase()}\n\n` +
        `------------------------------------\n` +
        `🔐 *LINK BORANG PERIBADI (PnC):*\n` +
        `👇 KLIK UNTUK DAFTAR PESERTA\n` +
        `${link}\n` +
        `------------------------------------\n\n` +
        `🚀 *ARAHAN PENTING:*\n` +
        `1. *SIMPAN* link ini sebagai nota peribadi.\n` +
        `2. *GUNA* link ini untuk mendaftar prospek.\n` +
        `3. *PnC:* Jangan dedahkan link ini secara terbuka.\n\n` +
        `📲 *TINDAKAN SUSULAN:*\n` +
        `Hubungi sponsor untuk arahan seterusnya:\n\n` +
        `🟢 *KLIK DI SINI UNTUK WHATSAPP SPONSOR:* \n` +
        `${linkWaSponsor}\n\n` +
        `*WORK SMART, TRUST THE SYSTEM!*`;

    const url = `https://wa.me/${phone}?text=${encodeURIComponent(teksWhatsApp)}`;
    
    // Redirect terus ke WhatsApp Peserta Baru (Leader Baru)
    window.location.replace(url);
  });
}); // <--- Pastikan ada penutup kurungan ini sebelum fungsi resetSystem

function resetSystem(){
  if(!confirm("Reset semua data?")) return;
  db.ref("members").remove().then(()=>{ alert("Berjaya dipadam"); location.reload(); });
}

// ====================== LEADER PANEL ======================
function openLeaderLogin() {
  let tel = prompt("Masukkan No. Telefon (Contoh: 60123456789):");
  if(!tel) return;
  tel = normalizePhone(tel);
  const leader = members.find(m => m.phone === tel);
  if(leader){
    document.getElementById("leaderPanel").style.display = "block";
    let direct = members.filter(m => m.sponsor === leader.name).length;
    let total = members.filter(m => m.currentLeader === leader.name).length;
    document.getElementById("leaderStatsDisplay").innerHTML = `
      <b>Nama:</b> ${leader.name}<br>
      <b>Telefon:</b> ${leader.phone}<br>
      <b>Direct Sponsor:</b> ${direct} orang | <b>Total Group:</b> ${total} orang
    `;
    renderLeaderTree(leader.name);
    document.getElementById("leaderPanel").scrollIntoView({ behavior: 'smooth' });
  } else { alert("Maaf, nombor telefon tidak dijumpai."); }
}

// ====================== FIREBASE DATA WATCH ======================
auth.signInAnonymously();
db.ref("members").on("value", snapshot => {
  const data = snapshot.val();
  members = data ? Object.values(data) : [];

  currentLeader = BASE_LEADER;
  currentLeaderPhone = BASE_PHONE;
  currentUpline = BASE_UPLINE;
  currentUplinePhone = BASE_UPLINE_PHONE;

  if(ref){
    let found = members.find(m => m.id === ref);
    if(found){
      currentLeader = found.name;
      currentLeaderPhone = found.phone;
      currentUpline = found.uplineName || BASE_UPLINE;
      currentUplinePhone = found.uplinePhone || BASE_UPLINE_PHONE;
    }
  }

  document.getElementById("leaderBox").innerHTML = `<div class="box">Nama: <b>${currentLeader}</b><br>Telefon: ${currentLeaderPhone}</div>`;
  document.getElementById("uplineBox").innerHTML = `<div class="box">Nama: <b>${currentUpline}</b><br>Telefon: ${currentUplinePhone}</div>`;
  renderMembers();
});

document.getElementById("reflink").value = location.origin + location.pathname + (ref ? "?ref="+ref : "");
// --- TAMBAHKAN KOD INI DI BAWAH ---
function cuciInput(nama, tel) {
    // 1. Nama: Buang simbol pelik & tukar ke HURUF BESAR
    let namaBersih = nama.replace(/[^a-zA-Z ]/g, "").toUpperCase().trim();

    // 2. Telefon: Pastikan hanya nombor & bermula dengan 60
    let telBersih = tel.replace(/\D/g, ''); 
    if (telBersih.startsWith('0')) {
        telBersih = '6' + telBersih; 
    } else if (telBersih.startsWith('1')) {
        telBersih = '60' + telBersih;
    }
    
    return { namaBersih, telBersih };
}
</script>
</body>
</html>
