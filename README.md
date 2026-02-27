<!DOCTYPE html>
<html lang="ar">
<head>
<meta charset="UTF-8">
<meta name="viewport" content="width=device-width, initial-scale=1.0">
<title>حاسبة سقي الزيتون والطاقة الشمسية</title>
<script src="https://cdn.jsdelivr.net/npm/chart.js"></script>
<style>
body { font-family: Arial; background:#f4f4f4; padding:20px; direction:rtl; }
.container { max-width:900px; margin:auto; background:#fff; padding:20px; border-radius:10px; box-shadow:0 0 10px rgba(0,0,0,0.1); }
h1 { text-align:center; }
input, select { width:100%; padding:8px; margin:5px 0 15px 0; border-radius:5px; border:1px solid #ccc; }
button { width:100%; padding:12px; background:#2e7d32; color:white; border:none; border-radius:5px; font-size:16px; cursor:pointer; }
button:hover { background:#1b5e20; }
#results { margin-top:20px; padding:15px; background:#e8f5e9; border-radius:8px; }
canvas { margin-top:30px; }
</style>
</head>
<body>

<div class="container">
<h1>حاسبة سقي الزيتون + الطاقة الشمسية</h1>

<label>صنف الزيتون:</label>
<select id="cultivar">
<option value="picholine">Picholine</option>
<option value="bicala">Bicala</option>
<option value="arbiqwen">Arbiqwen</option>
</select>

<label>ET0 السنوي (mm):</label>
<input type="number" id="et0" value="500">

<label>التساقطات السنوية (mm):</label>
<input type="number" id="rainfall" value="400">

<label>عدد الأشجار:</label>
<input type="number" id="trees" value="100">

<label>ساعات تشغيل المضخة يومياً:</label>
<input type="number" id="pumpHours" value="4">

<label>قدرة المضخة (kW):</label>
<input type="number" id="pumpKW" value="1">

<label>ثمن الكيلوواط (درهم):</label>
<input type="number" id="price" value="1.2">

<label>ثمن اللوح (درهم):</label>
<input type="number" id="panelPrice" value="1000">

<button onclick="calculate()">احسب</button>

<div id="results"></div>

<canvas id="irrigationChart"></canvas>
<canvas id="energyChart"></canvas>

</div>

<script>

let irrigationChart;
let energyChart;

function calculate() {

let cultivar = document.getElementById("cultivar").value;
let ET0 = parseFloat(document.getElementById("et0").value);
let rainfall = parseFloat(document.getElementById("rainfall").value);
let trees = parseInt(document.getElementById("trees").value);
let pumpHours = parseFloat(document.getElementById("pumpHours").value);
let pumpKW = parseFloat(document.getElementById("pumpKW").value);
let pricePerKWh = parseFloat(document.getElementById("price").value);
let panelPrice = parseFloat(document.getElementById("panelPrice").value);

let Kc = 0.65;
if(cultivar === "bicala") Kc = 0.70;
if(cultivar === "arbiqwen") Kc = 0.60;

let ETc = ET0 * Kc;
let deficit = ETc - rainfall;
if(deficit < 0) deficit = 0;

let irrigationPerTree = deficit * 10 / 1000;
let totalIrrigation = irrigationPerTree * trees;

let dailyEnergy = pumpHours * pumpKW;
let yearlyEnergy = dailyEnergy * 365;

let panelProduction = 900;
let numPanels = Math.ceil(yearlyEnergy / panelProduction);

let installationCost = 4000;
let totalCost = numPanels * panelPrice + installationCost;

let yearlySaving = yearlyEnergy * pricePerKWh;
let payback = (totalCost / yearlySaving).toFixed(1);
let profit20 = (yearlySaving * 20 - totalCost).toFixed(0);

document.getElementById("results").innerHTML = `
<h3>نتائج السقي</h3>
ETc: ${ETc.toFixed(1)} mm<br>
العجز المائي: ${deficit.toFixed(1)} mm<br>
السقي لكل شجرة: ${irrigationPerTree.toFixed(2)} m³<br>
السقي الإجمالي: ${totalIrrigation.toFixed(2)} m³<br>

<h3>نتائج الطاقة الشمسية</h3>
الطاقة السنوية: ${yearlyEnergy.toFixed(1)} kWh<br>
عدد الألواح: ${numPanels}<br>
تكلفة المشروع: ${totalCost} درهم<br>
التوفير السنوي: ${yearlySaving.toFixed(1)} درهم<br>
مدة الاسترجاع: ${payback} سنة<br>
الربح بعد 20 سنة: ${profit20} درهم
`;

let months = ["يناير","فبراير","مارس","أبريل","مايو","يونيو","يوليو","غشت","شتنبر","أكتوبر","نونبر","دجنبر"];

let irrigationData = Array(12).fill(totalIrrigation/12);
let energyData = Array(12).fill(yearlyEnergy/12);

if(irrigationChart) irrigationChart.destroy();
if(energyChart) energyChart.destroy();

irrigationChart = new Chart(document.getElementById("irrigationChart"), {
type: "bar",
data: {
labels: months,
datasets: [{ label: "السقي الشهري (m³)", data: irrigationData }]
},
options: { responsive:true }
});

energyChart = new Chart(document.getElementById("energyChart"), {
type: "line",
data: {
labels: months,
datasets: [{ label: "الطاقة الشهرية (kWh)", data: energyData }]
},
options: { responsive:true }
});

}

</script>

</body>
</html>
