<!DOCTYPE html>
<html lang="vi">
<head>
<meta charset="UTF-8">
<meta name="viewport" content="width=device-width, initial-scale=1.0">
<title>anhtubantool</title>

<style>
*{box-sizing:border-box}
body{
    margin:0;
    font-family:-apple-system,BlinkMacSystemFont,"Segoe UI",Roboto,Arial;
    background:linear-gradient(270deg,red,orange,yellow,green,cyan,blue,violet);
    background-size:1400% 1400%;
    animation:rainbow 18s ease infinite;
    color:#fff;
    min-height:100vh;
}
@keyframes rainbow{
    0%{background-position:0% 50%}
    50%{background-position:100% 50%}
    100%{background-position:0% 50%}
}

.app{max-width:420px;margin:auto;padding:12px}

.header{
    text-align:center;
    font-size:22px;
    font-weight:bold;
    padding:12px;
    border:2px solid #fff;
    border-radius:16px;
    margin-bottom:12px;
}

.tabs{display:flex;gap:6px;margin-bottom:10px}
.tab{
    flex:1;
    padding:10px;
    text-align:center;
    border:2px solid #fff;
    border-radius:12px;
    cursor:pointer;
    font-weight:bold;
}
.tab.active{background:rgba(255,255,255,0.25)}

.section{
    display:none;
    border:2px solid #fff;
    border-radius:16px;
    padding:14px;
    background:rgba(0,0,0,0.35);
}
.section.active{display:block}

input,button{
    width:100%;
    padding:12px;
    margin-top:8px;
    border-radius:12px;
    border:none;
    font-size:15px;
}
button{font-weight:bold;cursor:pointer}

.reset{
    margin-top:10px;
    background:#000;
    color:#fff;
    border:2px solid #fff;
}

/* ===== TABLE ===== */
table{
    width:100%;
    border-collapse:collapse;
    margin-top:10px;
    font-size:13px;
}
th,td{
    border:1px solid #fff;
    padding:6px;
    text-align:center;
}
th{background:rgba(255,255,255,0.2)}
</style>
</head>

<body>
<div class="app">

<div class="header">🌈 anhtubantool 🌈</div>

<div class="tabs">
    <div class="tab active" onclick="showTab(0)">SICBO</div>
    <div class="tab" onclick="showTab(1)">SUNWIN</div>
    <div class="tab" onclick="showTab(2)">CHẴN / LẺ</div>
</div>

<!-- ================= SICBO ================= -->
<div class="section active">
<h3>🎲 SICBO AI</h3>
<input id="md5Input" placeholder="Nhập MD5 32 ký tự">
<button onclick="analyzeSicbo()">PHÂN TÍCH</button>

<table>
<thead>
<tr><th>#</th><th>Xúc xắc</th><th>Tổng</th><th>KQ</th></tr>
</thead>
<tbody id="sicboHist"></tbody>
</table>

<button class="reset" onclick="resetHistory('sicbo_hist',renderSicbo)">♻️ RESET LỊCH SỬ</button>
</div>

<!-- ================= SUNWIN ================= -->
<div class="section">
<h3>🎲 SUNWIN AI</h3>
<div>⏳ Làm mới sau <b><span id="countdown">60</span></b> giây</div>
<p>Phiên: <span id="phien"></span></p>
<p>Xúc xắc: <span id="dice"></span></p>
<p>Tổng: <span id="tong"></span></p>
<p>Kết quả: <span id="ketqua"></span></p>
<p>Dự đoán: <span id="dudoan"></span></p>
<p>Cầu: <span id="cau"></span></p>
</div>

<!-- ================= CHẴN LẺ ================= -->
<div class="section">
<h3>🔐 MD5 → CHẴN / LẺ</h3>
<input id="md5CL" placeholder="Nhập MD5 32 ký tự">
<button onclick="analyzeCL()">PHÂN TÍCH</button>

<table>
<thead>
<tr><th>#</th><th>MD5</th><th>KQ</th></tr>
</thead>
<tbody id="clHist"></tbody>
</table>

<button class="reset" onclick="resetHistory('cl_hist',renderCL)">♻️ RESET LỊCH SỬ</button>
</div>

</div>

<script>
/* ===== TAB ===== */
function showTab(i){
    document.querySelectorAll('.tab').forEach((t,idx)=>t.classList.toggle('active',idx===i))
    document.querySelectorAll('.section').forEach((s,idx)=>s.classList.toggle('active',idx===i))
}

/* ===== RESET ===== */
function resetHistory(key,cb){
    localStorage.removeItem(key);
    cb();
}

/* ===== SICBO ===== */
function analyzeSicbo(){
    const md5=md5Input.value.trim().toLowerCase();
    if(!/^[0-9a-f]{32}$/.test(md5)) return alert("MD5 không hợp lệ");

    let seed=parseInt(md5.slice(0,8),16);
    let r=()=>Math.sin(seed++)*10000%1;
    let d1=Math.floor(r()*6)+1;
    let d2=Math.floor(r()*6)+1;
    let d3=Math.floor(r()*6)+1;
    let t=d1+d2+d3;
    let k=t>=11?"TÀI":"XỈU";

    let h=JSON.parse(localStorage.getItem("sicbo_hist")||"[]");
    h.unshift({d:`${d1}-${d2}-${d3}`,t,k});
    if(h.length>10) h.pop();
    localStorage.setItem("sicbo_hist",JSON.stringify(h));
    renderSicbo();
}

function renderSicbo(){
    let h=JSON.parse(localStorage.getItem("sicbo_hist")||"[]");
    sicboHist.innerHTML=h.map((i,idx)=>`<tr>
        <td>${idx+1}</td><td>${i.d}</td><td>${i.t}</td><td>${i.k}</td>
    </tr>`).join("");
}
renderSicbo();

/* ===== SUNWIN ===== */
let cd=60;
async function loadSun(){
    try{
        let r=await fetch("https://sunwinsaygex-tzz9.onrender.com/api/sun");
        let d=await r.json();
        phien.textContent=d.phien;
        dice.textContent=`${d.xuc_xac_1}-${d.xuc_xac_2}-${d.xuc_xac_3}`;
        tong.textContent=d.tong;
        ketqua.textContent=d.ket_qua;
        dudoan.textContent=d.du_doan;
        cau.textContent=d.cau;
    }catch{ketqua.textContent="Lỗi API";}
}
setInterval(()=>{
    countdown.textContent=cd--;
    if(cd<0){cd=60;loadSun()}
},1000);
loadSun();

/* ===== CHẴN LẺ ===== */
function analyzeCL(){
    const md5=md5CL.value.trim().toLowerCase();
    if(!/^[0-9a-f]{32}$/.test(md5)) return alert("MD5 không hợp lệ");
    let r=parseInt(md5.slice(-2),16)%2?"LẺ":"CHẴN";
    let h=JSON.parse(localStorage.getItem("cl_hist")||"[]");
    h.unshift({m:md5.slice(0,8)+"…",r});
    if(h.length>10) h.pop();
    localStorage.setItem("cl_hist",JSON.stringify(h));
    renderCL();
}

function renderCL(){
    let h=JSON.parse(localStorage.getItem("cl_hist")||"[]");
    clHist.innerHTML=h.map((i,idx)=>`<tr>
        <td>${idx+1}</td><td>${i.m}</td><td>${i.r}</td>
    </tr>`).join("");
}
renderCL();
</script>

</body>
</html>
