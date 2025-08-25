# Auditoriju noslodze
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
      background: #f4f4f4;
    }
    header {
      display: flex;
      justify-content: space-between;
      align-items: center;
      padding: 15px 20px;
      background-color: #007bff;
      color: #fff;
      font-size: 18px;
    }
    .date-time {
      font-weight: bold;
    }
    main {
      max-width: 1200px;
      margin: 30px auto;
      padding: 20px;
      background: #fff;
      border-radius: 10px;
      box-shadow: 0 2px 6px rgba(0,0,0,0.2);
    }
    table {
      width: 100%;
      border-collapse: collapse;
      margin-top: 20px;
    }
    th, td {
      border: 1px solid #ccc;
      padding: 10px;
      text-align: center;
    }
    th {
      background-color: #f1f1f1;
    }
    button {
      padding: 8px 15px;
      margin-top: 10px;
      cursor: pointer;
      background-color: #28a745;
      color: white;
      border: none;
      border-radius: 5px;
      font-size: 14px;
    }
    select, input[type="text"] {
      width: 100%;
      padding: 5px;
      box-sizing: border-box;
    }
    input.invalid {
      border: 2px solid red;
    }
  </style>
</head>
<body>
  <header>
    <div>Mācību grafiks</div>
    <div class="date-time" id="dateTime"></div>
  </header>
  <main>
    <h2>Plānotie pasākumi</h2>
    <button onclick="addRow()">Pievienot rindu</button>
    <table id="dataTable">
      <thead>
        <tr>
          <th>Auditorija</th>
          <th>Laiks</th>
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
      const options = {
        timeZone: 'Europe/Riga',
        year: 'numeric',
        month: '2-digit',
        day: '2-digit',
        hour: '2-digit',
        minute: '2-digit',
        second: '2-digit'
      };
      const formatted = new Intl.DateTimeFormat('lv-LV', options).format(now);
      document.getElementById('dateTime').textContent = formatted;
    }
    setInterval(updateDateTime, 1000);
    updateDateTime();

    // TABULAS FUNKCIJAS
    const tableBody = document.querySelector("#dataTable tbody");
    const auditorijas = [
      "101", "102", "103", "104", "105", "106",
      "1. datortelpa", "2. datortelpa", "Zāle", "Sporta zāle"
    ];

    function addRow(data = { auditorija: "", laiks: "", macibu: "", lektors: "", riimc: "" }) {
      const row = document.createElement("tr");

      // Auditorija (select)
      const td1 = document.createElement("td");
      const select = document.createElement("select");
      auditorijas.forEach(aud => {
        const option = document.createElement("option");
        option.value = aud;
        option.textContent = aud;
        if (data.auditorija === aud) option.selected = true;
        select.appendChild(option);
      });
      select.onchange = saveData;
      td1.appendChild(select);

      // Laiks (HH:MM)
      const td2 = document.createElement("td");
      const inputTime = document.createElement("input");
      inputTime.type = "text";
      inputTime.placeholder = "12:00";
      inputTime.value = data.laiks;
      inputTime.oninput = () => {
        validateTime(inputTime);
        saveData();
      };
      td2.appendChild(inputTime);

      // Mācību pasākums
      const td3 = document.createElement("td");
      const inputMacibu = document.createElement("input");
      inputMacibu.type = "text";
      inputMacibu.value = data.macibu;
      inputMacibu.oninput = saveData;
      td3.appendChild(inputMacibu);

      // Lektors
      const td4 = document.createElement("td");
      const inputLektors = document.createElement("input");
      inputLektors.type = "text";
      inputLektors.value = data.lektors;
      inputLektors.oninput = saveData;
      td4.appendChild(inputLektors);

      // RIIMC speciālists
      const td5 = document.createElement("td");
      const inputRiimc = document.createElement("input");
      inputRiimc.type = "text";
      inputRiimc.value = data.riimc;
      inputRiimc.oninput = saveData;
      td5.appendChild(inputRiimc);

      row.appendChild(td1);
      row.appendChild(td2);
      row.appendChild(td3);
      row.appendChild(td4);
      row.appendChild(td5);
      tableBody.appendChild(row);
    }

    // Validācija HH:MM 24h formātā
    function validateTime(input) {
      const regex = /^([01]\d|2[0-3]):([0-5]\d)$/;
      if (!regex.test(input.value) && input.value !== "") {
        input.classList.add("invalid");
      } else {
        input.classList.remove("invalid");
      }
    }

    // LocalStorage saglabāšana un ielāde
    function saveData() {
      const rows = tableBody.querySelectorAll("tr");
      const data = [];
      rows.forEach(row => {
        const select = row.querySelector("select");
        const inputs = row.querySelectorAll("input");
        data.push({
          auditorija: select.value,
          laiks: inputs[0].value,
          macibu: inputs[1].value,
          lektors: inputs[2].value,
          riimc: inputs[3].value
        });
      });
      localStorage.setItem("grafiksData", JSON.stringify(data));
    }

    function loadData() {
      const data = JSON.parse(localStorage.getItem("grafiksData")) || [];
      data.forEach(row => addRow(row));
    }

    loadData();
  </script>
</body>
</html>
