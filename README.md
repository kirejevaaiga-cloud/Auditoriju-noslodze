<!DOCTYPE html>
<html lang="lv">
<head>
<meta charset="UTF-8">
<meta name="viewport" content="width=device-width, initial-scale=1.0">
<title>Mājaslapa ar datumu un grafiku</title>
<style>
body { font-family: Arial; margin:0; padding:0; background:#f4f4f4; }
header { display:flex; justify-content:space-between; align-items:center; padding:10px 20px; background:#007bff; color:#fff; font-size:18px; }
header img { height:40px; }
.date-time { font-weight:bold; }
main { max-width:1200px; margin:30px auto; padding:20px; background:#fff; border-radius:10px; box-shadow:0 2px 6px rgba(0,0,0,0.2);}
table { width:100%; border-collapse:collapse; margin-top:20px;}
th, td { border:1px solid #ccc; padding:10px; text-align:center;}
th { background:#f1f1f1;}
button { padding:8px 15px; margin-top:10px; cursor:pointer; background:#28a745; color:white; border:none; border-radius:5px; font-size:14px;}
select, input[type="text"] { width:100%; padding:5px; box-sizing:border-box;}
input.invalid { border:2px solid red;}
.time-range { display:flex; gap:5px; justify-content:center; }
.time-range input { width:50%; }
</style>
</head>
<body>
<header>
<img src="logo.png" alt="Logo">
<div class="date-time" id="dateTime"></div>
</header>
<main>
<h2>Mācību pasākumi</h2>
<button onclick="addRow()">Pievienot rindu</button>
<table id="dataTable">
<thead>
<tr>
<th>Auditorija</th>
<th>Laiks (no–līdz)</th>
<th>Mācību pasākums</th>
<th>Lektors</th>
<th>RIIMC atbildīgais speciālists</th>
</tr>
</thead>
<tbody>
</tbody>
</table>
</main>

<script>
// DATUMA UN LAIKA ATJAUNOŠANA UTC+2
function updateDateTime() {
  const now = new Date();
  const options = { timeZone:'Europe/Riga', year:'numeric', month:'2-digit', day:'2-digit', hour:'2-digit', minute:'2-digit', second:'2-digit'};
  document.getElementById('dateTime').textContent = new Intl.DateTimeFormat('lv-LV', options).format(now);
}
setInterval(updateDateTime,1000);
updateDateTime();

// TABULAS FUNKCIJAS
const tableBody = document.querySelector("#dataTable tbody");
const auditorijas = ["101","102","103","104","105","106","1. datortelpa","2. datortelpa","Zāle","Sporta zāle"];

function addRow(data={auditorija:"", laiksNo:"", laiksLidz:"", macibu:"", lektors:"", riimc:""}){
  const row = document.createElement("tr");

  // Auditorija
  const td1=document.createElement("td");
  const select=document.createElement("select");
  auditorijas.forEach(a=>{ const option=document.createElement("option"); option.value=a; option.textContent=a; if(data.auditorija===a) option.selected=true; select.appendChild(option);});
  select.onchange=saveData;
  td1.appendChild(select);

  // Laiks (no–līdz)
  const td2=document.createElement("td");
  const divRange=document.createElement("div"); divRange.className="time-range";
  const inputNo=document.createElement("input"); inputNo.type="text"; inputNo.placeholder="12:00"; inputNo.value=data.laiksNo;
  const inputLidz=document.createElement("input"); inputLidz.type="text"; inputLidz.placeholder="13:00"; inputLidz.value=data.laiksLidz;
  inputNo.oninput=()=>{validateTime(inputNo); saveData();}
  inputLidz.oninput=()=>{validateTime(inputLidz); saveData();}
  divRange.appendChild(inputNo); divRange.appendChild(inputLidz);
  td2.appendChild(divRange);

  // Mācību pasākums
  const td3=document.createElement("td");
  const inputMacibu=document.createElement("input"); inputMacibu.type="text"; inputMacibu.value=data.macibu; inputMacibu.oninput=saveData; td3.appendChild(inputMacibu);

  // Lektors
  const td4=document.createElement("td");
  const inputLektors=document.createElement("input"); inputLektors.type="text"; inputLektors.value=data.lektors; inputLektors.oninput=saveData; td4.appendChild(inputLektors);

  // RIIMC
  const td5=document.createElement("td");
  const inputRiimc=document.createElement("input"); inputRiimc.type="text"; inputRiimc.value=data.riimc; inputRiimc.oninput=saveData; td5.appendChild(inputRiimc);

  row.appendChild(td1); row.appendChild(td2); row.appendChild(td3); row.appendChild(td4); row.appendChild(td5);
  tableBody.appendChild(row);
}

// Validācija HH:MM
function validateTime(input){
  const regex=/^([01]\d|2[0-3]):([0-5]\d)$/;
  if(!regex.test(input.value)&&input.value!==""){ input.classList.add("invalid"); } else { input.classList.remove("invalid"); }
}

// LocalStorage
function saveData(){
  const rows=tableBody.querySelectorAll("tr");
  const data=[];
  rows.forEach(row=>{
    const select=row.querySelector("select");
    const inputs=row.querySelectorAll("input");
    data.push({ auditorija:select.value, laiksNo:inputs[0].value, laiksLidz:inputs[1].value, macibu:inputs[2].value, lektors:inputs[3].value, riimc:inputs[4].value });
  });
  localStorage.setItem("grafiksData",JSON.stringify(data));
}

function loadData(){
  const data=JSON.parse(localStorage.getItem("grafiksData"))||[];
  data.forEach(row=>addRow(row));
}

loadData();
</script>
</body>
</html>
