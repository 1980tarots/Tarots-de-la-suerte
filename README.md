<!DOCTYPE html>
<html lang="it">
<head>
  <meta charset="UTF-8">
  <title>Lettura Tarocchi</title>
  <script src="https://www.paypal.com/sdk/js?client-id=YOUR_BUSINESS_CLIENT_ID&currency=EUR"></script>
  <style>
    body {
      background: url('https://images.unsplash.com/photo-1616715643794-b2bfa382246f?auto=format&fit=crop&w=1950&q=80') no-repeat center center fixed;
      background-size: cover;
      color: #fff;
      font-family: 'Georgia', serif;
      text-align: center;
      padding: 20px;
      backdrop-filter: brightness(0.8);
    }
    h1 {
      font-size: 2.5em;
      color: #e94560;
    }
    .card {
      width: 100px;
      height: 150px;
      background-image: url('https://upload.wikimedia.org/wikipedia/commons/thumb/5/53/RWS_Tarot_00_Fool.jpg/120px-RWS_Tarot_00_Fool.jpg');
      background-size: cover;
      border: 2px solid #e94560;
      display: inline-block;
      margin: 10px;
      cursor: pointer;
      border-radius: 8px;
    }
    .card.selected {
      border-color: #00adb5;
    }
    #results {
      margin-top: 30px;
    }
    #paypal-button-container {
      margin-top: 20px;
    }
    #lang-select {
      position: absolute;
      top: 10px;
      right: 10px;
      background-color: #0f3460;
      border: none;
      color: #fff;
      padding: 5px;
    }
    #user-question, #birthdate, #gender-select {
      width: 80%;
      margin: 10px auto;
      display: block;
      padding: 10px;
      border-radius: 5px;
      border: none;
      font-family: 'Georgia', serif;
    }
    footer {
      margin-top: 40px;
      font-size: 0.9em;
    }
    footer a {
      color: #00adb5;
      text-decoration: none;
    }
  </style>
</head>
<body>
  <select id="lang-select" onchange="changeLanguage()">
    <option value="it">Italiano</option>
    <option value="en">English</option>
    <option value="es">Español</option>
  </select>

  <h1 id="title">Lettura dei Tarocchi</h1>
  <p id="birthLabel">Inserisci la tua data di nascita:</p>
  <input type="date" id="birthdate">

  <p id="genderLabel">Seleziona il tuo genere:</p>
  <select id="gender-select">
    <option value="male">Uomo</option>
    <option value="female">Donna</option>
    <option value="nonbinary">Non binario</option>
  </select>

  <p id="questionLabel">Scrivi (facoltativo) la tua domanda o il tema della tua lettura:</p>
  <textarea id="user-question" rows="3" placeholder="Es. Amore, lavoro, spiritualità..."></textarea>

  <p id="chooseLabel">Scegli 3 carte:</p>
  <div id="deck"></div>
  <button id="readBtn" onclick="handlePayment()">Paga 5€ per Scoprire il tuo destino</button>
  <div id="paypal-button-container" style="display:none;"></div>
  <div id="results"></div>

  <footer>
    <p>Seguici su <a href="https://www.facebook.com/profile.php?id=61575162406639" target="_blank">Facebook</a></p>
  </footer>

  <audio id="bg-audio" loop autoplay>
    <source src="https://www.bensound.com/bensound-music/bensound-slowmotion.mp3" type="audio/mp3">
  </audio>

  <script>
    const texts = {
      it: {
        title: "Lettura dei Tarocchi",
        birthLabel: "Inserisci la tua data di nascita:",
        chooseLabel: "Scegli 3 carte:",
        readBtn: "Paga 5€ per Scoprire il tuo destino",
        questionLabel: "Scrivi (facoltativo) la tua domanda o il tema della tua lettura:",
        genderLabel: "Seleziona il tuo genere:",
        positions: ["Passato", "Presente", "Futuro"]
      },
      en: {
        title: "Tarot Reading",
        birthLabel: "Enter your birth date:",
        chooseLabel: "Choose 3 cards:",
        readBtn: "Pay €5 to Reveal Your Fate",
        questionLabel: "Optionally write your question or reading theme:",
        genderLabel: "Select your gender:",
        positions: ["Past", "Present", "Future"]
      },
      es: {
        title: "Lectura del Tarot",
        birthLabel: "Introduce tu fecha de nacimiento:",
        chooseLabel: "Elige 3 cartas:",
        readBtn: "Paga 5€ para Revelar tu Destino",
        questionLabel: "Escribe (opcional) tu pregunta o el tema de tu consulta:",
        genderLabel: "Selecciona tu género:",
        positions: ["Pasado", "Presente", "Futuro"]
      }
    };

    let currentLang = 'it';
    let selected = [];

    const tarotCards = [
      { name: {it: "Il Matto", en: "The Fool", es: "El Loco"}, meaning: {it: "Nuovi inizi", en: "New beginnings", es: "Nuevos comienzos"} },
      { name: {it: "Il Mago", en: "The Magician", es: "El Mago"}, meaning: {it: "Potere personale", en: "Personal power", es: "Poder personal"} },
      { name: {it: "La Papessa", en: "The High Priestess", es: "La Sacerdotisa"}, meaning: {it: "Intuizione", en: "Intuition", es: "Intuición"} },
      { name: {it: "L’Imperatrice", en: "The Empress", es: "La Emperatriz"}, meaning: {it: "Abbondanza", en: "Abundance", es: "Abundancia"} },
      { name: {it: "L’Imperatore", en: "The Emperor", es: "El Emperador"}, meaning: {it: "Stabilità", en: "Stability", es: "Estabilidad"} }
    ];

    function shuffleDeck() {
      selected = [];
      const shuffled = [...tarotCards].sort(() => 0.5 - Math.random());
      const deck = document.getElementById("deck");
      deck.innerHTML = "";
      shuffled.forEach((card) => {
        const div = document.createElement("div");
        div.className = "card";
        div.onclick = () => selectCard(card, div);
        deck.appendChild(div);
      });
    }

    function selectCard(card, element) {
      if (selected.length >= 3 || element.classList.contains("selected")) return;
      element.classList.add("selected");
      selected.push(card);
    }

    function changeLanguage() {
      currentLang = document.getElementById("lang-select").value;
      const t = texts[currentLang];
      document.getElementById("title").innerText = t.title;
      document.getElementById("birthLabel").innerText = t.birthLabel;
      document.getElementById("chooseLabel").innerText = t.chooseLabel;
      document.getElementById("readBtn").innerText = t.readBtn;
      document.getElementById("questionLabel").innerText = t.questionLabel;
      document.getElementById("genderLabel").innerText = t.genderLabel;
    }

    function handlePayment() {
      document.getElementById("paypal-button-container").style.display = "block";
      document.getElementById("readBtn").style.display = "none";
      paypal.Buttons({
        createOrder: function(data, actions) {
          return actions.order.create({
            purchase_units: [{ amount: { value: '5.00' } }]
          });
        },
        onApprove: function(data, actions) {
          return actions.order.capture().then(function(details) {
            showInterpretation();
          });
        }
      }).render('#paypal-button-container');
    }

    function showInterpretation() {
      if (selected.length !== 3) {
        alert("Seleziona esattamente 3 carte.");
        return;
      }
      const question = document.getElementById("user-question").value;
      const resultDiv = document.getElementById("results");
      resultDiv.innerHTML = `<h2>${texts[currentLang].title}</h2>`;
      if (question.trim()) {
        resultDiv.innerHTML += `<p><em>${question}</em></p>`;
      }
      selected.forEach((card, i) => {
        resultDiv.innerHTML += `<p><strong>${texts[currentLang].positions[i]}:</strong> ${card.name[currentLang]} - ${card.meaning[currentLang]}</p>`;
      });
    }

    shuffleDeck();
  </script>
</body>
</html>
