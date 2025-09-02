# booking-app
Calendrier-reservation
booking-app/
 ├─ package.json
 ├─ server.js
 ├─ db.js
 ├─ sms.js
 ├─ routes/
 │   └─ bookings.js
 ├─ public/
 │   ├─ index.html
 │   ├─ main.js
 │   └─ styles.css
 ├─ .env.example
 └─ README.md
 {
  "name": "booking-app",
  "version": "1.0.0",
  "description": "Application de réservation avec calendrier et SMS",
  "main": "server.js",
  "scripts": {
    "start": "node server.js"
  },
  "dependencies": {
    "body-parser": "^1.20.2",
    "dotenv": "^16.3.1",
    "express": "^4.18.2",
    "sqlite3": "^5.1.6",
    "twilio": "^4.19.0"
  }
}const express = require('express');
const bodyParser = require('body-parser');
const dotenv = require('dotenv');
const path = require('path');
const bookingsRouter = require('./routes/bookings');
const { initDB } = require('./db');

dotenv.config();
const app = express();
const PORT = process.env.PORT || 3000;

app.use(bodyParser.json());
app.use(express.static(path.join(__dirname, 'public')));

// Routes
app.use('/api/bookings', bookingsRouter);

// Start DB and server
initDB();
app.listen(PORT, () => console.log(`🚀 Serveur démarré sur http://localhost:${PORT}`));
const sqlite3 = require('sqlite3').verbose();
const db = new sqlite3.Database('./bookings.db');

function initDB() {
  db.run(`
    CREATE TABLE IF NOT EXISTS bookings (
      id INTEGER PRIMARY KEY AUTOINCREMENT,
      name TEXT NOT NULL,
      phone TEXT NOT NULL,
      date TEXT NOT NULL
    )
  `);
}

function addBooking(name, phone, date, callback) {
  db.run(`INSERT INTO bookings (name, phone, date) VALUES (?, ?, ?)`,
    [name, phone, date],
    function (err) {
      callback(err, this?.lastID);
    });
}

function getBookings(callback) {
  db.all(`SELECT * FROM bookings`, [], (err, rows) => {
    callback(err, rows);
  });
}

module.exports = { initDB, addBooking, getBookings };
const twilio = require('twilio');
const dotenv = require('dotenv');
dotenv.config();

const client = twilio(process.env.TWILIO_SID, process.env.TWILIO_AUTH);

function sendSMS(to, message) {
  if (!process.env.TWILIO_SID || !process.env.TWILIO_AUTH || !process.env.TWILIO_PHONE) {
    console.log(`📩 SMS simulé vers ${to}: ${message}`);
    return;
  }

  client.messages.create({
    body: message,
    from: process.env.TWILIO_PHONE,
    to
  })
    .then(msg => console.log(`SMS envoyé: ${msg.sid}`))
    .catch(err => console.error(err));
}

module.exports = { sendSMS };
const express = require('express');
const router = express.Router();
const { addBooking, getBookings } = require('../db');
const { sendSMS } = require('../sms');

router.get('/', (req, res) => {
  getBookings((err, rows) => {
    if (err) return res.status(500).json({ error: err.message });
    res.json(rows);
  });
});

router.post('/', (req, res) => {
  const { name, phone, date } = req.body;
  if (!name || !phone || !date) return res.status(400).json({ error: 'Champs manquants' });

  addBooking(name, phone, date, (err, id) => {
    if (err) return res.status(500).json({ error: err.message });
    sendSMS(phone, `Votre réservation est confirmée pour le ${date}`);
    res.json({ success: true, id });
  });
});

module.exports = router;
<!DOCTYPE html>
<html lang="fr">
<head>
  <meta charset="UTF-8">
  <title>Réservation</title>
  <link rel="stylesheet" href="styles.css">
</head>
<body>
  <h1>Réserver une date</h1>
  <form id="bookingForm">
    <input type="text" id="name" placeholder="Votre nom" required>
    <input type="tel" id="phone" placeholder="Votre téléphone" required>
    <input type="date" id="date" required>
    <button type="submit">Réserver</button>
  </form>
  <h2>Réservations</h2>
  <ul id="bookings"></ul>
  <script src="main.js"></script>
</body>
</html>
const form = document.getElementById('bookingForm');
const bookingsList = document.getElementById('bookings');

async function loadBookings() {
  const res = await fetch('/api/bookings');
  const data = await res.json();
  bookingsList.innerHTML = data.map(b => `<li>${b.date} - ${b.name}</li>`).join('');
}

form.addEventListener('submit', async (e) => {
  e.preventDefault();
  const name = document.getElementById('name').value;
  const phone = document.getElementById('phone').value;
  const date = document.getElementById('date').value;

  await fetch('/api/bookings', {
    method: 'POST',
    headers: { 'Content-Type': 'application/json' },
    body: JSON.stringify({ name, phone, date })
  });

  form.reset();
  loadBookings();
});

loadBookings();
body {
  font-family: Arial, sans-serif;
  margin: 20px;
  background: #f7f7f7;
  color: #333;
}

form {
  margin-bottom: 20px;
}

input, button {
  padding: 8px;
  margin: 5px 0;
  display: block;
  width: 100%;
  max-width: 300px;
}

button {
  background: #4CAF50;
  color: white;
  border: none;
  cursor: pointer;
}

button:hover {
  background: #45a049;
}
TWILIO_SID=ton_sid_twilio
TWILIO_AUTH=ton_token_twilio
TWILIO_PHONE=+41xxxxxxx
# Booking App

Une application simple de réservation avec calendrier et notifications SMS.

## Installation
```bash
npm install
npm start






