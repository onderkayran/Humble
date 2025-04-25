<!DOCTYPE html>
<html lang="tr">
<head>
  <meta charset="UTF-8">
  <title>Humblee - 5 Kişilik</title>
  <style>
    body {
      font-family: Arial, sans-serif;
      background: #2b2b2b;
      color: #fff;
      margin: 0;
      padding: 0;
      overflow: hidden;
    }

    h1 {
      text-align: center;
      margin: 20px;
    }

    #table {
      position: relative;
      width: 700px;
      height: 700px;
      margin: 20px auto;
      border-radius: 50%;
      background: #206020;
      border: 10px solid #8B5A2B;
    }

    .player {
      position: absolute;
      text-align: center;
      width: 120px;
    }

    .hand {
      display: flex;
      justify-content: center;
      flex-wrap: wrap;
      gap: 4px;
      margin-top: 5px;
    }

    .card {
      background: #fff;
      color: #000;
      padding: 6px 12px;
      border-radius: 6px;
      border: 2px solid #333;
      font-size: 24px;
      cursor: pointer;
      transition: all 0.5s ease-in-out;
    }

    .card.animating {
      position: absolute;
      z-index: 1000;
      transition: all 0.8s ease-in-out;
    }

    #controls {
      text-align: center;
      margin-top: 10px;
    }

    button {
      padding: 10px 16px;
      margin: 5px;
      font-size: 16px;
      border: none;
      border-radius: 5px;
      cursor: pointer;
    }

    #status {
      text-align: center;
      margin-top: 10px;
      font-weight: bold;
      color: lightgreen;
    }
  </style>
</head>
<body>
  <h1>Humblee - 5 Kişilik</h1>
  <div id="table"></div>

  <div id="controls">
    <button id="give-card">Sağdakine Kart Ver</button>
    <button id="humblee">HUMBLEE!</button>
  </div>
  <div id="status"></div>

  <script>
    const animalSymbols = ["🐘", "🐎", "🐓", "🐢", "🦘"];
    const playerCount = 5;
    const players = [];
    let currentPlayer = 0;
    let selectedCardIndex = null;

    const table = document.getElementById("table");

    function createPlayers() {
      const angleStep = 360 / playerCount;
      for (let i = 0; i < playerCount; i++) {
        const div = document.createElement("div");
        div.className = "player";
        div.id = `player${i}`;
        div.innerHTML = `<div>${i === 0 ? "Sen" : "Bot" + i}</div><div class="hand" id="hand${i}"></div>`;
        table.appendChild(div);

        const angle = angleStep * i - 90;
        const radius = 280;
        const x = 350 + radius * Math.cos(angle * Math.PI / 180) - 60;
        const y = 350 + radius * Math.sin(angle * Math.PI / 180) - 30;

        div.style.left = `${x}px`;
        div.style.top = `${y}px`;

        players.push({ id: i, name: i === 0 ? "Sen" : `Bot${i}`, hand: [] });
      }
    }

    function shuffleDeck() {
      const deck = [];
      for (let symbol of animalSymbols) {
        for (let i = 0; i < playerCount; i++) {
          deck.push(symbol);
        }
      }
      return deck.sort(() => Math.random() - 0.5);
    }

    function dealCards(deck) {
      for (let i = 0; i < playerCount; i++) {
        players[i].hand = deck.slice(i * playerCount, (i + 1) * playerCount);
      }
    }

    function renderHands() {
      for (let i = 0; i < playerCount; i++) {
        const handDiv = document.getElementById(`hand${i}`);
        handDiv.innerHTML = "";
        players[i].hand.forEach((card, idx) => {
          const cardEl = document.createElement("div");
          cardEl.className = "card";
          cardEl.textContent = i === 0 ? card : "🎴";
          if (i === 0) {
            cardEl.onclick = () => {
              selectedCardIndex = idx;
              document.getElementById("status").textContent = `Seçilen kart: ${card}`;
            };
          }
          handDiv.appendChild(cardEl);
        });
      }
    }

    function animateCardTransfer(card, fromIndex, toIndex, callback) {
      const cardEl = document.createElement("div");
      cardEl.className = "card animating";
      cardEl.textContent = card;
      table.appendChild(cardEl);

      const fromBox = document.getElementById(`player${fromIndex}`).getBoundingClientRect();
      const toBox = document.getElementById(`player${toIndex}`).getBoundingClientRect();
      const tableBox = table.getBoundingClientRect();

      cardEl.style.left = (fromBox.left - tableBox.left + 50) + "px";
      cardEl.style.top = (fromBox.top - tableBox.top + 20) + "px";

      setTimeout(() => {
        cardEl.style.left = (toBox.left - tableBox.left + 50) + "px";
        cardEl.style.top = (toBox.top - tableBox.top + 20) + "px";
      }, 50);

      setTimeout(() => {
        table.removeChild(cardEl);
        callback();
      }, 900);
    }

    function giveCard(playerIndex) {
      const giver = players[playerIndex];
      const receiverIndex = (playerIndex + 1) % playerCount;
      const receiver = players[receiverIndex];

      let card;
      if (playerIndex === 0) {
        if (selectedCardIndex === null) {
          alert("Lütfen kart seç.");
          return;
        }
        card = giver.hand.splice(selectedCardIndex, 1)[0];
        selectedCardIndex = null;
      } else {
        // Bot zekası: En az olan kartı ver
        const counts = {};
        for (let c of giver.hand) counts[c] = (counts[c] || 0) + 1;
        let sorted = Object.entries(counts).sort((a, b) => a[1] - b[1]);
        const cardIndex = giver.hand.findIndex(c => c === sorted[0][0]);
        card = giver.hand.splice(cardIndex, 1)[0];
      }

      animateCardTransfer(card, playerIndex, receiverIndex, () => {
        receiver.hand.push(card);
        renderHands();
        checkHumblee(playerIndex);
        nextTurn();
      });
    }

    function nextTurn() {
      currentPlayer = (currentPlayer + 1) % playerCount;
      if (currentPlayer === 0) {
        document.getElementById("status").textContent = "Sıra sende.";
      } else {
        setTimeout(() => giveCard(currentPlayer), 1000);
      }
    }

    function checkHumblee(index) {
      const hand = players[index].hand;
      const allSame = hand.length === playerCount && hand.every(c => c === hand[0]);
      if (allSame) {
        document.getElementById("status").textContent = `🎉 ${players[index].name} HUMBLEE yaptı!`;
      }
    }

    document.getElementById("give-card").onclick = () => {
      if (currentPlayer !== 0) return;
      giveCard(0);
    };

    document.getElementById("humblee").onclick = () => {
      if (currentPlayer !== 0) return;
      const hand = players[0].hand;
      const allSame = hand.length === playerCount && hand.every(c => c === hand[0]);
      if (allSame) {
        document.getElementById("status").textContent = "🎉 Tebrikler! HUMBLEE yaptın!";
      } else {
        document.getElementById("status").textContent = "❌ Elindeki tüm kartlar aynı değil.";
      }
    };

    function startGame() {
      createPlayers();
      const deck = shuffleDeck();
      dealCards(deck);
      renderHands();
      document.getElementById("status").textContent = "Oyun başladı. Sıra sende.";
    }

    startGame();
  </script>
</body>
</html>
