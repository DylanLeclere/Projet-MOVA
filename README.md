"use client";
import { useState, useEffect } from "react";
import { MapContainer, TileLayer, Marker, Popup, CircleMarker } from "react-leaflet";
import "leaflet/dist/leaflet.css";
import QRCode from "react-qr-code";

const lockers = [
  { id: 1, name: "MOVA - Les Papeteries", position: [45.9038, 6.1215], available: 2, total: 8 },
  { id: 2, name: "MOVA - Centre Courier", position: [45.8994, 6.1295], available: 0, total: 12 },
  { id: 3, name: "MOVA - Place de la Mairie", position: [45.8999, 6.1281], available: 7, total: 10 },
];

export default function MovaApp() {
  const [selectedLocker, setSelectedLocker] = useState(null);
  const [duration, setDuration] = useState(null);
  const [reserved, setReserved] = useState(false);
  const [qrValue, setQrValue] = useState("");
  const [timeLeft, setTimeLeft] = useState(null);

  // Compte à rebours
  useEffect(() => {
    if (!timeLeft || timeLeft <= 0) return;
    const timer = setInterval(() => setTimeLeft((t) => t - 1), 1000);
    return () => clearInterval(timer);
  }, [timeLeft]);

  const handleReserve = (locker) => {
    if (locker.available === 0) return alert("Ce casier est complet.");
    setSelectedLocker(locker);
  };

  const handlePayment = (minutes) => {
    // Simulation du paiement Apple Pay / Google Pay
    const durations = { 60: 1.5, 120: 2.5, 240: 4.0, 1440: 6.0 };
    alert(`Paiement de ${durations[minutes]} € validé ✅`);
    setDuration(minutes);
    const uniqueQR = `MOVA-${Date.now()}`;
    setQrValue(uniqueQR);
    setReserved(true);
    setTimeLeft(minutes * 60); // en secondes
  };

  const handleBackHome = () => {
    setReserved(false);
    setSelectedLocker(null);
    setDuration(null);
    setQrValue("");
    setTimeLeft(null);
  };

  return (
    <div
      style={{
        backgroundColor: "#0D0D0D",
        color: "#E3C6A5",
        fontFamily: "Poppins, sans-serif",
        height: "100vh",
        textAlign: "center",
      }}
    >
      <h1 style={{ padding: "1rem 0" }}>MOVA – Le casier qui vous suit partout 🏍️</h1>

      {/* Vue principale */}
      {!selectedLocker && !reserved && (
        <MapContainer center={[45.8994, 6.1295]} zoom={14} style={{ height: "85vh", width: "100%" }}>
          <TileLayer url="https://{s}.tile.openstreetmap.org/{z}/{x}/{y}.png" />
          {lockers.map((locker) => (
            <div key={locker.id}>
              <CircleMarker
                center={locker.position}
                radius={14}
                color={locker.available === 0 ? "#FF3B30" : "#34C759"}
                fillOpacity={0.9}
                eventHandlers={{ click: () => handleReserve(locker) }}
              />
              <Marker position={locker.position}>
                <Popup>
                  <h3>{locker.name}</h3>
                  <p>
                    Disponibles : <b>{locker.available}</b> / {locker.total}
                  </p>
                  {locker.available === 0 ? (
                    <span style={{ color: "#FF3B30" }}>❌ Complet</span>
                  ) : (
                    <button
                      onClick={() => handleReserve(locker)}
                      style={{
                        backgroundColor: "#E3C6A5",
                        border: "none",
                        color: "#0D0D0D",
                        padding: "0.5rem 1rem",
                        borderRadius: "6px",
                        cursor: "pointer",
                      }}
                    >
                      Réserver
                    </button>
                  )}
                </Popup>
              </Marker>
            </div>
          ))}
        </MapContainer>
      )}

      {/* Étape 2 – Choix durée */}
      {selectedLocker && !reserved && (
        <div style={{ marginTop: "2rem" }}>
          <h2>{selectedLocker.name}</h2>
          <p>Choisissez votre durée :</p>
          <div style={{ display: "flex", justifyContent: "center", gap: "1rem", flexWrap: "wrap" }}>
            <button onClick={() => handlePayment(60)}>1h – 1,50 €</button>
            <button onClick={() => handlePayment(120)}>2h – 2,50 €</button>
            <button onClick={() => handlePayment(240)}>4h – 4,00 €</button>
            <button onClick={() => handlePayment(1440)}>Journée – 6,00 €</button>
          </div>
          <button
            onClick={handleBackHome}
            style={{ marginTop: "2rem", background: "none", color: "#999", border: "none" }}
          >
            ← Retour à la carte
          </button>
        </div>
      )}

      {/* Étape 3 – QR & Timer */}
      {reserved && (
        <div style={{ marginTop: "2rem" }}>
          <h2>🎟️ Réservation confirmée</h2>
          <p>Scannez ce QR code pour accéder à votre casier :</p>
          <div style={{ background: "white", display: "inline-block", padding: "1rem", marginTop: "1rem" }}>
            <QRCode value={qrValue} />
          </div>
          <p style={{ marginTop: "1rem" }}>Code : {qrValue}</p>
          <p>⏳ Temps restant : {Math.floor(timeLeft / 60)} min {timeLeft % 60}s</p>

          {timeLeft <= 0 && (
            <div style={{ marginTop: "1rem" }}>
              <h3>Votre session est terminée.</h3>
              <button
                onClick={handleBackHome}
                style={{
                  marginTop: "1rem",
                  backgroundColor: "#E3C6A5",
                  color: "#0D0D0D",
                  border: "none",
                  padding: "0.5rem 1rem",
                  borderRadius: "6px",
                }}
              >
                Retour à l’accueil
              </button>
            </div>
          )}
        </div>
      )}

      <footer style={{ marginTop: "1rem", color: "#999" }}>© 2025 MOVA</footer>
    </div>
  );
}
