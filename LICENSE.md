<!DOCTYPE html>
<html lang="fr">
<head>
<meta charset="UTF-8">
<title>Lean Training – Voltaira</title>
<meta name="viewport" content="width=device-width, initial-scale=1.0">

<!-- Librairies -->
<script src="https://cdnjs.cloudflare.com/ajax/libs/jspdf/2.5.1/jspdf.umd.min.js"></script>
<script src="https://cdnjs.cloudflare.com/ajax/libs/qrcodejs/1.0.0/qrcode.min.js"></script>

<style>
body{
  margin:0;font-family:'Segoe UI',Arial;
  background:linear-gradient(135deg,#0f1724,#1e293b);
  color:#e6eef6;
  display:flex;justify-content:center;align-items:center;min-height:100vh
}
.box{
  background:#1e293b;padding:20px;border-radius:16px;
  width:95%;max-width:700px;box-shadow:0 15px 40px rgba(0,0,0,.6)
}
header{
  display:flex;align-items:center;gap:15px;
  border-bottom:1px solid rgba(255,255,255,.1);
  padding-bottom:10px;margin-bottom:15px
}
header img{height:45px}
header h2{margin:0;color:#06b6d4}
header span{font-size:13px;color:#9ca3af}
input,button,select,textarea{
  width:100%;padding:12px;margin-top:10px;border-radius:10px;border:none;font-size:15px
}
input,select,textarea{
  background:#0f1724;color:#fff;border:1px solid rgba(255,255,255,.1)
}
button{background:#06b6d4;color:#041027;font-weight:700;cursor:pointer}
.card{background:rgba(255,255,255,.05);padding:14px;border-radius:12px;margin-top:14px}
.q{border-bottom:1px solid rgba(255,255,255,.15);padding:10px 0}
.q small{color:#cbd5e1}
.q button{width:48%}
.q .vrai.active{background:#22c55e}
.q .faux.active{background:#ef4444}
.hidden{display:none}
.footer{text-align:center;margin-top:15px;font-size:12px;color:#9ca3af}
</style>
</head>

<body>

<!-- LOGIN -->
<div id="loginBox" class="box">
  <header>
    <img src="logo.png">
    <div>
      <h2>Lean Training</h2>
      <span>Application de formation opérateurs</span>
    </div>
  </header>

  <input id="pass" type="password" placeholder="Mot de passe">
  <button id="btnLogin">Entrer</button>

  <div style="text-align:center;margin-top:20px">
    <p>Scanner ce QR pour entrer</p>
    <div id="qrcode"></div>
  </div>

  <div class="footer">Voltaira © Formation Interne</div>
</div>

<!-- APP -->
<div id="appBox" class="box hidden">

<header>
  <img src="logo.png">
  <div>
    <h2>Lean Quiz – PRO</h2>
    <span>Lean | Qualité | HSE</span>
  </div>
</header>

<div class="card">
  <input id="nom" placeholder="Nom & Prénom *">
  <input id="matricule" placeholder="Matricule *">
</div>

<div class="card">
  <select id="theme"></select>
  <button id="btnStart">Commencer Quiz</button>
</div>

<div class="card" id="quizBox"></div>
<button id="btnSubmit">Soumettre</button>
<p id="result"></p>

<button id="btnFormateur">👨🏫 Mode Formateur</button>

<div id="formateurBox" class="card hidden">
<h3>Mode Formateur</h3>

<select id="fTheme"></select>

<textarea id="qFR" placeholder="Question (Français)"></textarea>
<textarea id="qAR" placeholder="السؤال (العربية)"></textarea>

<select id="fAnswer">
  <option value="true">Vrai</option>
  <option value="false">Faux</option>
</select>

<button id="btnAddQ">➕ Ajouter</button>

<h4>Questions existantes</h4>
<div id="fList"></div>

<button id="btnExportAudit">📑 Export Audit IATF</button>
</div>

<div class="footer">Evidence de formation – Génération PDF automatique</div>
</div>

<script>
const { jsPDF } = window.jspdf;

const APP_PASSWORD="1234";
const FORMATEUR_PASSWORD="form123";
const PASS_PERCENT = 70;

// DOM
const el={
loginBox:loginBox,appBox:appBox,pass:pass,btnLogin:btnLogin,
nom:nom,matricule:matricule,theme:theme,btnStart:btnStart,
quizBox:quizBox,btnSubmit:btnSubmit,result:result,
btnFormateur:btnFormateur,formateurBox:formateurBox,
fTheme:fTheme,qFR:qFR,qAR:qAR,fAnswer:fAnswer,
btnAddQ:btnAddQ,fList:fList,btnExportAudit:btnExportAudit,
qr:qrcode
};

// QR
new QRCode(el.qr,{
text:"https://voltaira.com/leanquiz",
width:128,height:128,colorDark:"#06b6d4",colorLight:"#fff"
});

// QUESTIONS
let QUESTIONS=JSON.parse(localStorage.getItem("QUESTIONS_FR_AR"))||{
Lean:[{fr:"Le Lean vise à éliminer les gaspillages.",ar:"اللين يهدف إلى القضاء على الهدر.",a:true}],
Qualite:[{fr:"La qualité se contrôle seulement à la fin.",ar:"الجودة تتم مراقبتها فقط في النهاية.",a:false}],
HSE:[{fr:"Le port des EPI est obligatoire.",ar:"ارتداء معدات الوقاية الشخصية إجباري.",a:true}]
};
localStorage.setItem("QUESTIONS_FR_AR",JSON.stringify(QUESTIONS));

// LOGIN
btnLogin.onclick=()=>{
if(pass.value===APP_PASSWORD){
loginBox.classList.add("hidden");
appBox.classList.remove("hidden");
loadThemes();
}else alert("Mot de passe incorrect");
};

function loadThemes(){
theme.innerHTML="";fTheme.innerHTML="";
Object.keys(QUESTIONS).forEach(t=>{
theme.innerHTML+=`<option>${t}</option>`;
fTheme.innerHTML+=`<option>${t}</option>`;
});
refreshF();
}

let current=[],answers=[];

// START QUIZ
btnStart.onclick=()=>{
current=QUESTIONS[theme.value];
quizBox.innerHTML="";answers=[];
current.forEach((q,i)=>{
quizBox.innerHTML+=`
<div class="q">
<b>${q.fr}</b><br><small>${q.ar}</small><br><br>
<button class="vrai" onclick="answer(${i},true,this)">Vrai</button>
<button class="faux" onclick="answer(${i},false,this)">Faux</button>
</div>`;
});
};

// ANSWER
function answer(i,v,b){
if(answers[i]!==undefined)return;
answers[i]=v;b.classList.add("active");
}
window.answer=answer;

// SUBMIT
btnSubmit.onclick=()=>{
if(!nom.value||!matricule.value){alert("Nom & Matricule obligatoires");return;}
if(answers.length<current.length){alert("Répondez à toutes les questions");return;}

let correct=0;
current.forEach((q,i)=>{if(answers[i]===q.a)correct++;});

const percent=Math.round((correct/current.length)*100);
const status=percent>=PASS_PERCENT?"VALIDÉ":"NON VALIDÉ";

result.innerText=`Résultat : ${percent}% – ${status}`;
generatePDF(percent,status);

const save=JSON.parse(localStorage.getItem("OPERATEURS_RESULTS"))||[];
save.push({nom:nom.value,matricule:matricule.value,theme:theme.value,percent,status,date:new Date().toLocaleDateString()});
localStorage.setItem("OPERATEURS_RESULTS",JSON.stringify(save));
};

// PDF
function generatePDF(p,s){
const pdf=new jsPDF();
pdf.text("FICHE DE FORMATION OPERATEUR",105,30,{align:"center"});
pdf.text(`Nom : ${nom.value}`,20,50);
pdf.text(`Matricule : ${matricule.value}`,20,60);
pdf.text(`Formation : ${theme.value}`,20,70);
pdf.text(`Résultat : ${p}% - ${s}`,20,80);
pdf.text(`Date : ${new Date().toLocaleDateString()}`,20,90);
pdf.save(`Formation_${matricule.value}.pdf`);
}

// FORMATEUR
btnFormateur.onclick=()=>{
if(prompt("Mot de passe Formateur")===FORMATEUR_PASSWORD){
formateurBox.classList.toggle("hidden");refreshF();
}else alert("Accès refusé");
};

function refreshF(){
fList.innerHTML="";
(QUESTIONS[fTheme.value]||[]).forEach((q,i)=>{
fList.innerHTML+=`${q.fr}<br><small>${q.ar}</small>
<button onclick="delQ('${fTheme.value}',${i})">❌</button><br>`;
});
}

btnAddQ.onclick=()=>{
QUESTIONS[fTheme.value].push({fr:qFR.value,ar:qAR.value,a:fAnswer.value==="true"});
localStorage.setItem("QUESTIONS_FR_AR",JSON.stringify(QUESTIONS));
qFR.value="";qAR.value="";refreshF();
};

function delQ(t,i){
QUESTIONS[t].splice(i,1);
localStorage.setItem("QUESTIONS_FR_AR",JSON.stringify(QUESTIONS));
refreshF();
}
window.delQ=delQ;

// EXPORT AUDIT
btnExportAudit.onclick=()=>{
const pdf=new jsPDF();
pdf.text("Audit IATF – Résultats Formation",105,20,{align:"center"});
let y=40;
(JSON.parse(localStorage.getItem("OPERATEURS_RESULTS"))||[]).forEach(r=>{
pdf.text(`${r.nom} | ${r.matricule} | ${r.theme} | ${r.percent}% | ${r.status} | ${r.date}`,10,y);
y+=10;if(y>280){pdf.addPage();y=20;}
});
pdf.save("Audit_IATF.pdf");
};
</script>

</body>
</html>
