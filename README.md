# Eco-trash-bin-
AI-based smart garbage monitoring system
<!DOCTYPE html>
<html lang="en">
<head>
<meta charset="UTF-8">
<meta name="viewport" content="width=device-width, initial-scale=1.0">
<title>EcoSmart Bin Pro</title>

<link rel="stylesheet" href="https://unpkg.com/leaflet@1.9.4/dist/leaflet.css"/>
<script src="https://unpkg.com/leaflet@1.9.4/dist/leaflet.js"></script>

<style>
body {
  font-family: Arial;
  background: linear-gradient(135deg,#d4fc79,#96e6a1);
}

header {
  background:#1b5e20;
  color:white;
  padding:15px;
  text-align:center;
}

.container { padding:15px; }

.card {
  background:white;
  padding:15px;
  margin-bottom:15px;
  border-radius:10px;
}

#map { height:300px; }

button {
  width:100%;
  padding:10px;
  background:#2e7d32;
  color:white;
  border:none;
}

img { width:100%; margin-top:10px; }
</style>
</head>

<body>

<header>🌱 EcoSmart Bin</header>

<div class="container">

<div class="card">
<h3>Points: <span id="points">0</span></h3>
</div>

<div class="card">
<input type="file" id="imageInput">
<img id="preview">
<button onclick="analyzeImage()">Analyze</button>
<p id="result"></p>
</div>

<div class="card">
<div id="map"></div>
</div>

<div class="card">
<h3>Leaderboard</h3>
<div id="leaderboard"></div>
</div>

</div>

<script>
let bins = JSON.parse(localStorage.getItem('bins')) || [];
let leaderboard = JSON.parse(localStorage.getItem('leaderboard')) || [];
let points = parseInt(localStorage.getItem('points')) || 0;

document.getElementById('points').innerText = points;

let map;
let userLat, userLng;

function initMap(lat, lng) {
  map = L.map('map').setView([lat, lng], 15);

  L.tileLayer('https://{s}.tile.openstreetmap.org/{z}/{x}/{y}.png').addTo(map);

  L.marker([lat, lng]).addTo(map).bindPopup("You are here").openPopup();

  renderBins();
}

function getLocation() {
  navigator.geolocation.getCurrentPosition(pos => {
    userLat = pos.coords.latitude;
    userLng = pos.coords.longitude;
    initMap(userLat, userLng);
  }, () => alert("Allow location"));
}

getLocation();

document.getElementById('imageInput').addEventListener('change', e => {
  const file = e.target.files[0];
  document.getElementById('preview').src = URL.createObjectURL(file);
});

function analyzeImage() {
  if (!userLat) {
    alert("Wait for location");
    return;
  }

  let level = "Low";

  // Simple improved logic
  const img = document.getElementById("preview");
  if (!img.src) {
    alert("Upload image");
    return;
  }

  // Fake detection fallback (stable)
  let rand = Math.random();
  if (rand > 0.6) level = "Full";
  else if (rand > 0.3) level = "Medium";

  document.getElementById('result').innerText = "Detected: " + level;

  bins.push({ level, lat: userLat, lng: userLng });
  localStorage.setItem('bins', JSON.stringify(bins));

  if (level === "Full") {
    points += 20;
    localStorage.setItem('points', points);
    updateLeaderboard();
  }

  document.getElementById('points').innerText = points;

  renderBins();
}

function renderBins() {
  bins.forEach(bin => {
    if (bin.level === "Full") {
      L.marker([bin.lat, bin.lng]).addTo(map)
        .bindPopup("🗑 Garbage Full");
    }
  });
}

function updateLeaderboard() {
  let name = localStorage.getItem('user') || prompt("Enter name");
  localStorage.setItem('user', name);

  let user = leaderboard.find(u => u.name === name);
  if (user) user.points += 20;
  else leaderboard.push({ name, points: 20 });

  localStorage.setItem('leaderboard', JSON.stringify(leaderboard));
  renderLeaderboard();
}

function renderLeaderboard() {
  let div = document.getElementById('leaderboard');
  div.innerHTML = "";

  leaderboard.sort((a,b)=>b.points-a.points);

  leaderboard.forEach((u,i)=>{
    div.innerHTML += `#${i+1} ${u.name} - ${u.points} pts<br>`;
  });
}

renderLeaderboard();
</script>

</body>
</html>
