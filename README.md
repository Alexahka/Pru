<!DOCTYPE html>
<html lang="ru">
<head>
  <meta charset="UTF-8">
  <meta name="viewport" content="width=device-width, initial-scale=1.0, maximum-scale=1.0, user-scalable=no">
  <title>Секретные карты 🃏</title>
  <style>
    /* Общие стили */
    body {
      font-family: Arial, sans-serif;
      background-color: #000; /* Черный фон */
      color: #ecf0f1; /* Белый текст */
      margin: 0;
      padding: 0;
      display: flex;
      justify-content: center;
      align-items: center;
      height: 100vh;
      overflow: hidden; /* Убираем полосы прокрутки */
    }

    #game-container {
      text-align: center;
      background-color: #1c1c1c; /* Темно-серый фон для контейнера */
      padding: 20px;
      border-radius: 10px;
      box-shadow: 0 4px 8px rgba(255, 255, 255, 0.2); /* Светлая тень */
      position: relative;
      width: 90%; /* Ширина контейнера для мобильных устройств */
      max-width: 400px; /* Максимальная ширина */
      overflow: hidden; /* Убираем переполнение для анимации */
    }

    /* Анимация светящихся линий */
    #game-container::before {
      content: '';
      position: absolute;
      top: 0;
      left: 0;
      right: 0;
      bottom: 0;
      border-radius: 10px;
      background: linear-gradient(90deg, transparent, rgba(255, 235, 59, 0.8), transparent);
      background-size: 300% 300%;
      animation: glowing-lines 3s linear infinite;
      z-index: -1;
    }

    @keyframes glowing-lines {
      0% {
        background-position: 0% 50%;
      }
      100% {
        background-position: 100% 50%;
      }
    }

    /* Светящиеся точки при ничьей */
    .glow-dots {
      position: absolute;
      width: 5px;
      height: 5px;
      background-color: #ffeb3b;
      border-radius: 50%;
      animation: glow-dots-animation 2s infinite;
      opacity: 0;
    }

    @keyframes glow-dots-animation {
      0% {
        transform: translateY(100%);
        opacity: 0;
      }
      50% {
        opacity: 1;
      }
      100% {
        transform: translateY(-100%);
        opacity: 0;
      }
    }

    #balance {
      margin-bottom: 20px;
      font-size: 16px;
    }

    #table {
      display: flex;
      flex-direction: column;
      align-items: center;
      margin: 20px 0;
      position: relative;
      height: 200px;
    }

    #dealer-area, #player-area {
      display: flex;
      gap: 10px;
      position: absolute;
      width: 100%;
      justify-content: center;
    }

    #dealer-area {
      top: 10px;
    }

    #player-area {
      bottom: 10px;
    }

    .card {
      width: 50px;
      height: 75px;
      background-color: white;
      border-radius: 5px;
      display: flex;
      justify-content: center;
      align-items: center;
      font-size: 14px;
      font-weight: bold;
      color: black;
      box-shadow: 0 2px 4px rgba(0, 0, 0, 0.2);
      position: relative;
      opacity: 0;
      transform: translateY(-50px);
      transition: all 0.5s ease;
    }

    .card.active {
      opacity: 1;
      transform: translateY(0);
    }

    .card.back {
      background-color: #2c3e50;
      background-image: url('https://i.imgur.com/6QZ5z6x.png');
      background-size: cover;
      transform: rotateY(180deg);
    }

    #controls {
      margin-top: 20px;
      display: flex;
      flex-direction: column;
      gap: 10px;
    }

    #controls button {
      padding: 10px;
      font-size: 16px;
      cursor: pointer;
      border: none;
      border-radius: 5px;
      background-color: #1abc9c;
      color: white;
      transition: transform 0.2s ease, box-shadow 0.2s ease;
      box-shadow: 0 4px 6px rgba(0, 0, 0, 0.2);
    }

    #controls button:hover {
      transform: scale(1.1);
      box-shadow: 0 6px 8px rgba(0, 0, 0, 0.3);
    }

    #controls button:active {
      transform: scale(0.95);
      box-shadow: 0 2px 4px rgba(0, 0, 0, 0.2);
    }

    #controls button:disabled {
      background-color: #bdc3c7;
      cursor: not-allowed;
      transform: none;
      box-shadow: none;
    }

    #result {
      margin-top: 20px;
      font-size: 16px;
    }

    #change-background {
      margin-top: 20px;
      padding: 10px;
      font-size: 16px;
      cursor: pointer;
      border: none;
      border-radius: 5px;
      background-color: #ff9900;
      color: white;
      transition: transform 0.2s ease, box-shadow 0.2s ease;
      box-shadow: 0 4px 6px rgba(0, 0, 0, 0.2);
    }

    #change-background:hover {
      transform: scale(1.1);
      box-shadow: 0 6px 8px rgba(0, 0, 0, 0.3);
    }

    #change-background:active {
      transform: scale(0.95);
      box-shadow: 0 2px 4px rgba(0, 0, 0, 0.2);
    }
  </style>
</head>
<body>
  <div id="game-container">
    <h1>Секретные карты 🃏</h1>
    <div id="balance">Монеты: <span id="coin-count">100</span></div>
    <div id="table">
      <div id="dealer-area" class="cards"></div>
      <div id="player-area" class="cards"></div>
    </div>
    <div id="controls">
      <button id="start-game">Начать игру</button>
      <button id="hit" disabled>Взять карту</button>
      <button id="stand" disabled>Остановиться</button>
      <button id="surrender" disabled>Сдаться</button>
    </div>
    <div id="result"></div>
    <button id="change-background">Сменить фон</button>
  </div>

  <script>
    let coins = 100;
    let deck = [];
    let playerCards = [];
    let dealerCards = [];
    let bet = 0;

    const coinCount = document.getElementById('coin-count');
    const playerArea = document.getElementById('player-area');
    const dealerArea = document.getElementById('dealer-area');
    const resultDiv = document.getElementById('result');
    const startGameButton = document.getElementById('start-game');
    const hitButton = document.getElementById('hit');
    const standButton = document.getElementById('stand');
    const surrenderButton = document.getElementById('surrender');
    const gameContainer = document.getElementById('game-container');

    // Создание колоды карт
    function createDeck() {
      const suits = ['♠️', '♥️', '♣️', '♦️'];
      const ranks = ['2', '3', '4', '5', '6', '7', '8', '9', '10', 'J', 'Q', 'K', 'A'];
      return ranks.flatMap(rank => suits.map(suit => `${rank}${suit}`));
    }

    // Перемешивание колоды
    function shuffleDeck(deck) {
      for (let i = deck.length - 1; i > 0; i--) {
        const j = Math.floor(Math.random() * (i + 1));
        [deck[i], deck[j]] = [deck[j], deck[i]];
      }
    }

    // Подсчет очков
    function calculateScore(cards) {
      let score = 0;
      let aces = 0;
      cards.forEach(card => {
        const rank = card.slice(0, -1);
        if (['J', 'Q', 'K'].includes(rank)) {
          score += 10;
        } else if (rank === 'A') {
          score += 11;
          aces++;
        } else {
          score += parseInt(rank);
        }
      });
      while (score > 21 && aces > 0) {
        score -= 10;
        aces--;
      }
      return score;
    }

    // Анимация добавления карт
    function addCardWithAnimation(area, card) {
      const cardElement = document.createElement('div');
      cardElement.textContent = card;
      cardElement.classList.add('card', 'back');
      area.appendChild(cardElement);

      setTimeout(() => {
        cardElement.style.opacity = '1';
        cardElement.style.transform = 'translateY(0)';
      }, 10);

      setTimeout(() => {
        cardElement.classList.remove('back');
        cardElement.textContent = card;
      }, 500);
    }

    // Отображение карт
    function displayCards(area, cards) {
      area.innerHTML = '';
      cards.forEach(card => {
        addCardWithAnimation(area, card);
      });
    }

    // Начало игры
    startGameButton.addEventListener('click', () => {
      if (coins < 10) {
        alert('Недостаточно монет для игры! Минимальная ставка: 10 монет.');
        return;
      }

      deck = createDeck();
      shuffleDeck(deck);
      playerCards = [deck.pop(), deck.pop()];
      dealerCards = [deck.pop(), deck.pop()];
      bet = 10;
      coins -= bet;
      coinCount.textContent = coins;

      displayCards(playerArea, playerCards);
      displayCards(dealerArea, [dealerCards[0]]);
      resultDiv.textContent = '';

      startGameButton.disabled = true;
      hitButton.disabled = false;
      standButton.disabled = false;
      surrenderButton.disabled = false;
    });

    // Взять карту
    hitButton.addEventListener('click', () => {
      const newCard = deck.pop();
      playerCards.push(newCard);
      addCardWithAnimation(playerArea, newCard);

      const playerScore = calculateScore(playerCards);
      if (playerScore > 21) {
        endGame('lose');
      }
    });

    // Остановиться
    standButton.addEventListener('click', () => {
      while (calculateScore(dealerCards) < 17) {
        const newCard = deck.pop();
        dealerCards.push(newCard);
        addCardWithAnimation(dealerArea, newCard);
      }

      const playerScore = calculateScore(playerCards);
      const dealerScore = calculateScore(dealerCards);

      if (dealerScore > 21 || playerScore > dealerScore) {
        endGame('win');
      } else if (playerScore < dealerScore) {
        endGame('lose');
      } else {
        endGame('draw');
      }
    });

    // Сдаться
    surrenderButton.addEventListener('click', () => {
      coins += Math.floor(bet / 2);
      coinCount.textContent = coins;
      endGame('surrender');
    });

    // Завершение игры
    function endGame(result) {
      startGameButton.disabled = false;
      hitButton.disabled = true;
      standButton.disabled = true;
      surrenderButton.disabled = true;

      if (result === 'win') {
        coins += bet * 2;
        resultDiv.textContent = 'Вы выиграли! 🎉';
      } else if (result === 'lose') {
        resultDiv.textContent = 'Вы проиграли! 😔';
      } else if (result === 'draw') {
        coins += bet;
        resultDiv.textContent = 'Ничья! 🤝';
        showGlowDots();
      } else if (result === 'surrender') {
        resultDiv.textContent = 'Вы сдались. Половина ставки возвращена.';
      }

      coinCount.textContent = coins;
    }

    // Светящиеся точки при ничьей
    function showGlowDots() {
      for (let i = 0; i < 20; i++) {
        const dot = document.createElement('div');
        dot.classList.add('glow-dots');
        dot.style.left = `${Math.random() * 100}%`;
        gameContainer.appendChild(dot);

        setTimeout(() => {
          dot.remove();
        }, 2000);
      }
    }

    // Смена фона
    const changeBackgroundButton = document.getElementById('change-background');
    const colors = ['#1c1c1c', '#e74c3c', '#f1c40f']; // Цвета стола
    let currentColorIndex = 0;

    changeBackgroundButton.addEventListener('click', () => {
      currentColorIndex = (currentColorIndex + 1) % colors.length;
      gameContainer.style.backgroundColor = colors[currentColorIndex];
    });
  </script>
</body>
</html>
