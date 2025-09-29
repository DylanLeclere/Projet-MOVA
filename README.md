# Projet-MOVA
Application de casier autonome
<!DOCTYPE html>
<html lang="fr">
<head>
  <meta charset="UTF-8" />
  <meta name="viewport" content="width=device-width, initial-scale=1.0"/>
  <title>MOVA - Casiers du Futur</title>
  <link href="style.css" rel="stylesheet">
  <link rel="stylesheet" href="https://unpkg.com/leaflet@1.7.1/dist/leaflet.css"/>
</head>
<body>
  <header>
    <h1 class="logo">MOVA</h1>
    <p class="slogan">Le casier qui vous suit partout</p>
  </header>

  <main>
    <section class="hero">
      <button class="btn-primary" id="reserve-btn">🚀 Réserver un casier</button>
    </section>

    <section id="map-section" style="display:none;">
      <h2>📍 Localisation</h2>
      <div id="map"></div>
      <button class="btn-secondary" id="go-reservation">Continuer</button>
    </section>

    <section id="reservation-section" style="display:none;">
      <h2>📝 Réservation</h2>
      <form id="reservation-form">
        <input type="text" placeholder="Votre nom" required />
        <input type="email" placeholder="Votre email" required />
        <select id="duration">
          <option value="1">2h - 2€</option>
          <option value="2">5h - 3,5€</option>
          <option value="4">12h - 10€</option>
          <option value="ME">abonée - abonée</option>
        </select>
        <button type="submit" class="btn-primary">💳 Procéder au paiement</button>
      </form>
    </section>

    <section id="qr-section" style="display:none;">
      <h2>✅ Votre QR code</h2>
      <canvas id="qr-code"></canvas>
      <p class="timer">⏳ Temps restant : <span id="countdown"></span></p>
      <button id="unlock-btn" class="btn-secondary">🔓 Déverrouiller</button>
    </section>

    <section id="final-qr" style="display:none;">
      <h2>🔑 QR Code de déverrouillage</h2>
      <canvas id="unlock-qr"></canvas>
    </section>
  </main>

  <script src="https://unpkg.com/leaflet@1.7.1/dist/leaflet.js"></script>
  <script src="https://cdn.jsdelivr.net/npm/qrcode/build/qrcode.min.js"></script>
  <script src="script.js"></script>
</body>
</html>
body {
  background: #000;
  color: #fff;
  font-family: 'Inter', sans-serif;
  margin: 0;
  padding: 0;
  text-align: center;
}

header {
  padding: 20px;
}

.logo {
  font-size: 3em;
  font-weight: 800;
  color: #fddbb0;
  text-shadow: 0 0 20px rgba(253, 219, 176, 0.7);
}

.slogan {
  font-size: 1.2em;
  color: #aaa;
  margin-bottom: 20px;
}

h2 {
  color: #fddbb0;
  text-shadow: 0 0 15px rgba(253, 219, 176, 0.5);
}

#map {
  height: 300px;
  width: 100%;
  margin: 20px auto;
  border-radius: 10px;
  overflow: hidden;
}

form {
  display: flex;
  flex-direction: column;
  gap: 15px;
  padding: 20px;
}

input, select {
  padding: 12px;
  border: none;
  border-radius: 8px;
  font-size: 1em;
  outline: none;
}

.btn-primary {
  background: linear-gradient(90deg, #fddbb0, #ff9f50);
  color: #000;
  padding: 15px;
  font-size: 1.1em;
  border: none;
  border-radius: 8px;
  cursor: pointer;
  font-weight: bold;
  box-shadow: 0 0 20px rgba(253, 219, 176, 0.8);
  transition: all 0.3s ease;
}

.btn-primary:hover {
  transform: scale(1.05);
  box-shadow: 0 0 30px rgba(253, 219, 176, 1);
}

.btn-secondary {
  background: transparent;
  color: #fddbb0;
  border: 2px solid #fddbb0;
  padding: 12px;
  border-radius: 8px;
  cursor: pointer;
  transition: all 0.3s ease;
}

.btn-secondary:hover {
  background: #fddbb0;
  color: #000;
}

canvas {
  margin-top: 15px;
  filter: drop-shadow(0 0 10px rgba(253, 219, 176, 0.8));
}

.timer {
  margin-top: 15px;
  font-size: 1.2em;
  color: #fddbb0;
}

@media (max-width: 600px) {
  .logo {
    font-size: 2em;
  }
  #map {
    height: 200px;
  }
}
const reserveBtn = document.getElementById("reserve-btn");
const mapSection = document.getElementById("map-section");
const reservationSection = document.getElementById("reservation-section");
const reservationForm = document.getElementById("reservation-form");
const qrSection = document.getElementById("qr-section");
const finalQRSection = document.getElementById("final-qr");

const qrCanvas = document.getElementById("qr-code");
const unlockCanvas = document.getElementById("unlock-qr");
const unlockBtn = document.getElementById("unlock-btn");
const countdown = document.getElementById("countdown");

let durationMinutes = 120;

function generateQR(canvas, text) {
  QRCode.toCanvas(canvas, text, { width: 200 }, function (error) {
    if (error) console.error(error);
  });
}

// Flow navigation
reserveBtn.addEventListener("click", () => {
  mapSection.style.display = "block";
});

document.getElementById("go-reservation").addEventListener("click", () => {
  mapSection.style.display = "none";
  reservationSection.style.display = "block";
});

// Handle reservation
reservationForm.addEventListener("submit", (e) => {
  e.preventDefault();
  reservationSection.style.display = "none";
  qrSection.style.display = "block";

  durationMinutes = parseInt(document.getElementById("duration").value) * 60;

  generateQR(qrCanvas, "reservation-" + Date.now());

  // Timer
  let endTime = new Date(Date.now() + durationMinutes * 60000);
  let timerInterval = setInterval(() => {
    let now = new Date();
    let remaining = endTime - now;
    if (remaining <= 0) {
      countdown.innerText = "Temps écoulé";
      clearInterval(timerInterval);
      return;
    }
    let mins = Math.floor((remaining / 60000) % 60);
    let hrs = Math.floor((remaining / 3600000));
    countdown.innerText = `${hrs}h ${mins}min`;
  }, 1000);
});

// Unlock
unlockBtn.addEventListener("click", () => {
  finalQRSection.style.display = "block";
  generateQR(unlockCanvas, "unlock-" + Date.now());
});

// Map
const map = L.map('map', {
  zoomControl: false,
  attributionControl: false
}).setView([45.8992, 6.1294], 16);

L.tileLayer('https://{s}.tile.openstreetmap.org/{z}/{x}/{y}.png').addTo(map);

L.marker([45.8992, 6.1294]).addTo(map)
  .bindPopup("Casier MOVA – Centre Courier, Annecy").openPopup();
  
