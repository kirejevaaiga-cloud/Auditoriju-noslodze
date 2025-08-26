<!DOCTYPE html>
<html lang="lv">
<head>
<meta charset="UTF-8">
<meta name="viewport" content="width=device-width, initial-scale=1.0">
<title>Mājaslapa ar datumu un grafiku</title>
<style>
body {
  font-family: Arial, sans-serif;
  margin: 0;
  padding: 0;
  background: #f4f4f4 url('images/fons.jpg') no-repeat center center fixed;
  background-size: cover;
}

header {
  display: flex;
  justify-content: space-between;
  align-items: center;
  padding: 10px 20px;
  background: #007bff;
  color: #fff;
  font-size: 18px;
}

header img { height: 40px; }
.date-time { font-weight: bold; }

main {
  max-width: 1200px;
  margin: 30px auto;
  padding: 20px;
  background: #fff;
  border-radius: 10px;
  box-shadow: 0 2px 6px rgba(0,0,0,0.2);
}

h2 { margin-top: 0; }

table { width: 100%; border-collapse: collapse; margin-top: 20px;}
th, td { border: 1px solid #ccc; padding: 10px; text-align: center;}
th { background: #f1f1f1;}
button { padding: 8px 15px; margin-top: 10px; cursor: pointer; background: #28a745; color:white; border:none; border-radius:5px; font-size:14px;}
select, input[type="text"], input[type="date"] { width:100%; padding:5px; box-sizing:border-box;}
input.invalid { border:2px solid red;}
.time-range { display:flex; gap:5px; justify-content:center; }
.time-range input { width:50%; }
.delete-btn { background:#dc3545; color:white; border:none; padding:2px 6px; border-radius:50%; cursor:pointer; font-weight:bold; margin-left:5px; }
#filterDiv { margin-top: 20px; margin-bottom: 10px; }
</style>
</head>
<body>

<header>
<img src="logo.png" alt="Logo">
<div class="date-time" id="dateTime"></div>
</header>

<main>
<h2>Pasākumu pievienošana</h2>
<button onclick="addRow()">Pievienot rindu</button>
<table id="dataTable">
<thead>
<tr>
<th>Datums</th>
<th>Auditorija</th>
<th>Laiks</th>
<th>Mācību pasākums</th>
<th>Lektors</th>
<th>RIIMC atbildīgais speciālists</th>
</tr>
</thead>
<tbody></tbody>
</table>

<hr>
<h2>Pasākumu skatīšanās</h2>
<div id="filterDiv">
  <label for="filterDate">Izvēlies datumu:</label>
  <input type="date" id="filterDate">
</div>
<table id="viewTable">
<thead>
<tr>
<th>Auditorija</th>
<th>Laiks</th>
<th>Mācību pasākums</th>
<th>Lektors</th>
<th>RIIMC atbildīgais speciālists</th>
</tr>
</thead>
<tbody></tbody>
</table>
</main>

<script>
// DATUMA UN LAIKA ATJAUNOŠANA
function updateDateTime() {
  const now = new Date();
  const options = { timeZone:'Europe/Riga', year:'numeric', month:'2-digit', day:'2-digit', hour:'2-digit', minute:'2-digit', second:'2-digit'};
  document.getElementById('dateTime').textContent = new Intl.DateTimeFormat('lv-LV', options).format(now);
}
setInterval(updateDateTime,1000);
updateDateTime();

// DATU DEFINĪCIJAS
const tableBody = document.querySelector("#dataTable tbody");
const viewBody = document.querySelector("#viewTable tbody");
const auditorijas = ["101","102","103","104","105","106","1. datortelpa","2. datortelpa","Zāle","Sporta zāle"];
const riimcList = [
  "Aiga Kirejeva, t. 67105540",
  "Aleksejs Lukjančikovs, t. 67105271",
  "Dace Sondare, t. 67105541",
  "Daina Keidāne, t. 67105550",
  "Daina Kupča, t. 67105543",
  "Ilze Kadiķe, t. 67105545",
  "Inga Draveniece, t. 67105547",
  "Inga Liepniece, t. 67105544",
  "Iveta Razumovska, t. 67105534",
  "Jeļena Griezne-Dubrovska, t.67105129",
  "Kitija Čipāne, t. 67105546",
  "Kristīne Kalniņa, t. 67105579",
  "Sarmīte Katkeviča, t. 67105581"
];

// PIEVIENO RINDU
function addRow(data={date:"", auditorija:"", laiksNo:"", laiksLidz:"", macibu:"", lektors:"", riimc:""}) {
  const row = document.createElement("tr");

  // Datums
  const tdDate = document.createElement("td");
  const inputDate = document.createElement("input");
  inputDate.type = "date";
  inputDate.value = data.date;
  inputDate.onchange = saveData;
  tdDate.appendChild(inputDate);

  // Auditorija
  const tdAud = document.createElement("td");
  const selectAud = document.createElement("select");
  auditorijas.forEach(a=>{
    const option=document.createElement("option");
    option.value=a; option.textContent=a;
    if(data.auditorija===a) option.selected=true;
    selectAud.appendChild(option);
  });
  selectAud.onchange=saveData;
  tdAud.appendChild(selectAud);

  // Laiks
  const tdTime = document.createElement("td");
  const divRange = document.createElement("div"); divRange.className="time-range";
  const inputNo = document.createElement("input"); inputNo.type="text"; inputNo.placeholder="HH:MM"; inputNo.value=data.laiksNo;
  inputNo.oninput=()=>{validateTime(inputNo); saveData();}
  const inputLidz=document.createElement("input"); inputLidz.type="text"; inputLidz.placeholder="HH:MM"; inputLidz.value=data.laiksLidz;
  inputLidz.oninput=()=>{validateTime(inputLidz); saveData();}
  divRange.appendChild(inputNo); divRange.appendChild(inputLidz);
  tdTime.appendChild(divRange);

  // Mācību pasākums
  const tdMacibu=document.createElement("td");
  const inputMacibu=document.createElement("input"); inputMacibu.type="text"; inputMacibu.value=data.macibu; inputMacibu.oninput=saveData; tdMacibu.appendChild(inputMacibu);

  // Lektors
  const tdLektors=document.createElement("td");
  const inputLektors=document.createElement("input"); inputLektors.type="text"; inputLektors.value=data.lektors; inputLektors.oninput=saveData; tdLektors.appendChild(inputLektors);

  // RIIMC + dzēst poga
  const tdRiimc=document.createElement("td");
  const selectRiimc=document.createElement("select");
  riimcList.forEach(r=>{
    const option=document.createElement("option"); option.value=r; option.textContent=r;
    if(data.riimc===r) option.selected=true;
    selectRiimc.appendChild(option);
  });
  selectRiimc.onchange=saveData;
  tdRiimc.appendChild(selectRiimc);

  const deleteBtn=document.createElement("button"); deleteBtn.textContent="×"; deleteBtn.className="delete-btn";
  deleteBtn.onclick=()=>{ row.remove(); saveData(); renderView(); };
  tdRiimc.appendChild(deleteBtn);

  row.appendChild(tdDate); row.appendChild(tdAud); row.appendChild(tdTime); row.appendChild(tdMacibu); row.appendChild(tdLektors); row.appendChild(tdRiimc);
  tableBody.appendChild(row);
  saveData(); renderView();
}

// HH:MM validācija
function validateTime(input){
  const regex=/^([01]\d|2[0-3]):([0-5]\d)$/;
  if(!regex.test(input.value)&&input.value!==""){ input.classList.add("invalid");} else{ input.classList.remove("invalid"); }
}

// LocalStorage saglabāšana
function saveData(){
  const rows=tableBody.querySelectorAll("tr");
  const data=[];
  rows.forEach(row=>{
    const inputs=row.querySelectorAll("input");
    const selects=row.querySelectorAll("select");
    data.push({
      date: inputs[0].value,
      auditorija: selects[0].value,
      laiksNo: inputs[1].value,
      laiksLidz: inputs[2].value,
      macibu: inputs[3].value,
      lektors: inputs[4].value,
      riimc: selects[1].value
    });
  });
  localStorage.setItem("grafiksData", JSON.stringify(data));
}

// LocalStorage ielāde
function loadData(){
  const data=JSON.parse(localStorage.getItem("grafiksData"))||[];
  data.forEach(row=>addRow(row));
}

loadData();

// Filtrs un tabulas atjaunošana
const filterDate=document.getElementById("filterDate");
filterDate.onchange=renderView;

function renderView(){
  const filter = filterDate.value;
  const data=JSON.parse(localStorage.getItem("grafiksData"))||[];
  // Filtrē pēc datuma
  let filtered=data.filter(d=>d.date===filter);
  // Kārto pēc auditorijas un pēc laika
  filtered.sort((a,b)=>{
    if(a.auditorija<b.auditorija) return -1;
    if(a.auditorija>b.auditorija) return 1;
    return a.laiksNo.localeCompare(b.laiksNo);
  });
  // Atjauno tabulu
  viewBody.innerHTML="";
  filtered.forEach(d=>{
    const row=document.createElement("tr");
    row.innerHTML=`<td>${d.auditorija}</td><td>${d.laiksNo}-${d.laiksLidz}</td><td>${d.macibu}</td><td>${d.lektors}</td><td>${d.riimc}</td>`;
    viewBody.appendChild(row);
  });
}
