<!DOCTYPE html>
<html>
<head>
<meta charset="UTF-8">
<title>Cybersecurity Dashboard</title>
<script src="https://cdn.jsdelivr.net/npm/chart.js"></script>
<link rel="stylesheet" href="https://unpkg.com/leaflet/dist/leaflet.css"/>
<script src="https://unpkg.com/leaflet/dist/leaflet.js"></script>

<style>
body{
 background:#0d1117;
 color:white;
 font-family:Arial;
 padding:20px;
}
.card{
 background:#161b22;
 padding:20px;
 border-radius:10px;
 margin:10px;
}
#map{
 height:300px;
 margin-top:20px;
}
canvas{
 background:#161b22;
 padding:20px;
 border-radius:10px;
}
button{
 padding:10px 20px;
 background:#00ffcc;
 border:none;
 cursor:pointer;
}
</style>
</head>
<body>

<h1>Cybersecurity Dashboard</h1>

<div class="card">
  <h2>Login</h2>
  <input id="user" placeholder="Username">
  <input id="pass" type="password" placeholder="Password">
  <button onclick="login()">Login</button>
</div>

<div class="card">
  <h2>Threat Analytics</h2>
  <canvas id="threatChart"></canvas>
</div>

<div class="card">
  <h2>Intrusion Logs</h2>
  <ul id="logs"></ul>
</div>

<div id="map"></div>

<button onclick="exportReport()">Export Security Report</button>

<script>
let token = "";
let chart;
let map = L.map('map').setView([20,0],2);

L.tileLayer('https://{s}.tile.openstreetmap.org/{z}/{x}/{y}.png',{
 maxZoom:18
}).addTo(map);

async function login(){
 const res = await fetch("http://localhost:5000/login",{
   method:"POST",
   headers:{"Content-Type":"application/json"},
   body:JSON.stringify({
     username:document.getElementById("user").value,
     password:document.getElementById("pass").value
   })
 });
 const data = await res.json();
 token = data.token;
 loadEvents();
}

async function loadEvents(){
 const res = await fetch("http://localhost:5000/events");
 const events = await res.json();

 document.getElementById("logs").innerHTML = "";
 let counts = {Low:0, Medium:0, High:0};

 events.forEach(e=>{
   document.getElementById("logs").innerHTML +=
    `<li>${e.time} - ${e.ip} - ${e.threat} (${e.severity})</li>`;

   counts[e.severity]++;

   const coords = getCoordinates(e.country);
   L.marker(coords).addTo(map)
    .bindPopup(`${e.ip}<br>${e.threat}`);
 });

 updateChart(counts);
}

function updateChart(counts){
 const ctx = document.getElementById("threatChart");
 if(chart) chart.destroy();

 chart = new Chart(ctx,{
   type:"bar",
   data:{
     labels:["Low","Medium","High"],
     datasets:[{
       label:"Threat Levels",
       data:[counts.Low, counts.Medium, counts.High]
     }]
   }
 });
}

function exportReport(){
 window.open("http://localhost:5000/export");
}

function getCoordinates(country){
 const mapCoords = {
   USA:[37,-95],
   India:[20,78],
   Germany:[51,10],
   China:[35,103]
 };
 return mapCoords[country] || [0,0];
}

setInterval(loadEvents, 5000);
</script>

</body>
</html>
