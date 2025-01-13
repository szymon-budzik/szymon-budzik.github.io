<!DOCTYPE html>
<html lang="pl">

<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Blackjack - Szymon Budzik</title>
    <style>
        body {
            font-family: Arial, sans-serif;
            text-align: center;
            background-color: rgb(49, 1, 1);
            color: white;
            font-weight: bold;
        }

        .container {
            margin: 20px auto;
            max-width: 600px;
            padding: 20px;
            background-color: #35654d;
            border-radius: 10px;
            box-shadow: 0 4px 8px rgba(0, 0, 0, 0.2);
        }

        button {
            padding: 10px 20px;
            margin: 5px;
            font-size: 16px;
            background-color: #FFD700;
            border: none;
            border-radius: 5px;
            cursor: pointer;
        }

        button:hover {
            background-color: #FFA500;
        }

        .cards {
            display: flex;
            justify-content: center;
            gap: 10px;
            margin: 10px 0;
        }

        .card {
            width: 60px;
            height: 80px;
            border: 2px solid white;
            border-radius: 5px;
            display: flex;
            justify-content: center;
            align-items: center;
            font-size: 18px;
            background-color: white;
            color: black;
        }

        .bet-input {
            margin: 10px 0;
        }
    </style>
</head>

<body>
    <div class="container">
        <h1>Blackjack</h1>
        <p id="message">Podaj swój zakład i kliknij "Deal" aby zacząć grę!</p>

        <div class="bet-input">
            <label for="bet">Kwota zakładu: $</label>
            <input type="number" id="bet" min="1" max="1000" value="100">
        </div>

        <div>
            <button id="deal"><b>Deal</b></button>
            <button id="hit"><b>Hit</b></button>
            <button id="stand"><b>Stand</b></button>
            <button id="double"><b>Double</b></button>
            <button id="split"><b>Split</b></button>
        </div>

        <h2>Twoje karty</h2>
        <div class="cards" id="player-cards"></div>
        <p>Punkty gracza: <span id="player-total">0</span></p>

        <h2>Karty Krupiera</h2>
        <div class="cards" id="dealer-cards"></div>
        <p>Punkty krupiera: <span id="dealer-total">0</span></p>

        <h2>Twoje saldo</h2>
        <p>$<span id="balance">1000</span></p>
    </div>

    <script>
        // Pobieranie referencji do elementów HTML
        const dealButton = document.getElementById('deal');
        const hitButton = document.getElementById('hit');
        const standButton = document.getElementById('stand');
        const doubleButton = document.getElementById('double');
        const splitButton = document.getElementById('split');
        const betInput = document.getElementById('bet');
        const playerCardsDiv = document.getElementById('player-cards');
        const dealerCardsDiv = document.getElementById('dealer-cards');
        const playerTotalSpan = document.getElementById('player-total');
        const dealerTotalSpan = document.getElementById('dealer-total');
        const balanceSpan = document.getElementById('balance');
        const messageParagraph = document.getElementById('message');

        // Inicjalizacja zmiennych gry
        let playerCards = [];
        let dealerCards = [];
        let deck = [];
        let balance = 1000;
        let currentBet = 0;
        let canDouble = false;
        let canSplit = false;

        // Funkcja tworząca nową talię kart
        function createDeck() {
            const suits = ['♠', '♥', '♦', '♣'];
            const values = [2, 3, 4, 5, 6, 7, 8, 9, 10, 'J', 'Q', 'K', 'A'];
            let newDeck = [];
            for (let suit of suits) {
                for (let value of values) {
                    newDeck.push({ value, suit });
                }
            }
            return newDeck;
        }

        // Funkcja tasująca talię kart
        function shuffleDeck(deck) {
            for (let i = deck.length - 1; i > 0; i--) {
                const j = Math.floor(Math.random() * (i + 1));
                [deck[i], deck[j]] = [deck[j], deck[i]];
            }
        }

        // Funkcja obliczająca sumę punktów kart
        function calculateTotal(cards) {
            let total = 0;
            let aces = 0;
            for (let card of cards) {
                if (typeof card.value === 'number') {
                    total += card.value;
                } else if (['J', 'Q', 'K'].includes(card.value)) {
                    total += 10;
                } else if (card.value === 'A') {
                    total += 11;
                    aces++;
                }
            }
            while (total > 21 && aces > 0) {
                total -= 10;
                aces--;
            }
            return total;
        }

        // Funkcja renderująca karty w podanym kontenerze
        function renderCards(cards, container) {
            container.innerHTML = '';
            for (let card of cards) {
                const cardDiv = document.createElement('div');
                cardDiv.className = 'card';
                cardDiv.textContent = `${card.value}${card.suit}`;
                if (card.suit === '♥' || card.suit === '♦') {
                    cardDiv.style.color = 'red';
                } else {
                    cardDiv.style.color = 'black';
                }
                container.appendChild(cardDiv);
            }
        }

        // Funkcja rozpoczynająca nową grę
        function startGame() {
            // Wyświetlenie komunikatu startowego
            setMessage('Hit, Stand, Double, lub Split?', 'transparent');

            // Walidacja zakładu
            currentBet = parseInt(betInput.value, 10);
            if (currentBet > balance || currentBet <= 0) {
                setMessage('Zła kwota zakładu!', 'rgba(255, 0, 0, 0.5)');
                return;
            }

            // Aktualizacja salda i inicjalizacja gry
            balance -= currentBet;
            balanceSpan.textContent = balance;

            deck = createDeck();
            shuffleDeck(deck);
            playerCards = [deck.pop(), deck.pop()];
            dealerCards = [deck.pop(), deck.pop()];

            renderCards(playerCards, playerCardsDiv);
            renderCards(dealerCards.slice(0, 1), dealerCardsDiv);

            const playerTotal = calculateTotal(playerCards);
            playerTotalSpan.textContent = playerTotal;
            dealerTotalSpan.textContent = '?';

            // Ustawienie dostępnych opcji gracza
            canDouble = true;
            canSplit = playerCards[0].value === playerCards[1].value;
            hitButton.disabled = false;
            standButton.disabled = false;
            doubleButton.disabled = !canDouble;
            splitButton.disabled = !canSplit;

            // Automatyczna akcja przy blackjacku
            if (playerTotal === 21) {
                setMessage('Masz Blackjacka! Stand automatyczny.', 'rgba(0, 255, 0, 0.5)');
                stand();
            }
        }

        // Funkcja obsługująca przycisk "Hit"
        function hit() {
            playerCards.push(deck.pop());
            renderCards(playerCards, playerCardsDiv);

            const playerTotal = calculateTotal(playerCards);
            playerTotalSpan.textContent = playerTotal;

            if (playerTotal === 21) {
                messageParagraph.textContent = 'Masz 21 punktów! Stand automatyczny.';
                stand();
                return;
            }

            if (playerTotal > 21) {
                setMessage('Krupier wygrywa!', 'rgba(255, 0, 0, 0.5)');
                endGame();
            }

            canDouble = false;
            doubleButton.disabled = true;
        }

        // Funkcja obsługująca przycisk "Stand"
        function stand() {
            renderCards(dealerCards, dealerCardsDiv);
            let dealerTotal = calculateTotal(dealerCards);
            dealerTotalSpan.textContent = dealerTotal;

            while (dealerTotal < 17) {
                dealerCards.push(deck.pop());
                dealerTotal = calculateTotal(dealerCards);
                renderCards(dealerCards, dealerCardsDiv);
                dealerTotalSpan.textContent = dealerTotal;
            }

            const playerTotal = calculateTotal(playerCards);

            // Porównanie wyników gracza i krupiera
            if (dealerTotal > 21 || playerTotal > dealerTotal) {
                setMessage('Wygrywasz!', 'rgba(0, 255, 0, 0.5)');
                balance += currentBet * 2;
            } else if (playerTotal < dealerTotal) {
                setMessage('Krupier wygrywa!', 'rgba(255, 0, 0, 0.5)');
            } else {
                setMessage('Remis!', 'rgba(255, 255, 0, 0.5)');
                balance += currentBet;
            }

            balanceSpan.textContent = balance;
            endGame();
        }

        // Funkcja obsługująca przycisk "Double"
        function doubleDown() {
            if (!canDouble) return;
            currentBet *= 2;
            balance -= currentBet / 2;
            balanceSpan.textContent = balance;

            playerCards.push(deck.pop());
            renderCards(playerCards, playerCardsDiv);

            const playerTotal = calculateTotal(playerCards);
            playerTotalSpan.textContent = playerTotal;

            if (playerTotal > 21) {
                messageParagraph.textContent = 'Krupier wygrywa';
                endGame();
                return;
            }

            stand();
        }

        // Funkcja obsługująca przycisk "Split"
        function split() {
            if (!canSplit) return;

            // Rozdzielenie kart
            const firstCard = playerCards[0];
            const secondCard = playerCards[1];

            // Resetowanie ręki gracza i dodanie kart do nowej ręki
            playerCards = [firstCard];
            const splitHand = [secondCard];

            // Aktualizacja salda
            balance -= currentBet;
            balanceSpan.textContent = balance;

            // Dobranie kart dla obu rąk
            playerCards.push(deck.pop());
            splitHand.push(deck.pop());

            renderCards(playerCards, playerCardsDiv);
            const playerTotal = calculateTotal(playerCards);
            playerTotalSpan.textContent = playerTotal;

            messageParagraph.textContent = 'Split udany! Zagraj każdą rękę.';
        }

        // Funkcja ustawiająca komunikat
        function setMessage(text, backgroundColor = 'transparent') {
            messageParagraph.textContent = text;
            messageParagraph.style.backgroundColor = backgroundColor;
        }

        // Funkcja kończąca grę
        function endGame() {
            // Wyłączenie przycisków po zakończeniu gry
            hitButton.disabled = true;
            standButton.disabled = true;
            doubleButton.disabled = true;
            splitButton.disabled = true;

            // Obsługa sytuacji, gdy saldo wynosi 0
            if (balance <= 0) {
                setTimeout(() => {
                    alert('Nie masz już pieniędzy, udajesz się do bankomatu po 1000$.');
                    balance = 1000;
                    balanceSpan.textContent = balance;
                    messageParagraph.textContent = 'Postaw zakład aby kontynuować grę.';
                }, 1000);
            }
        }

        // Dodanie nasłuchiwaczy zdarzeń do przycisków
        dealButton.addEventListener('click', startGame);
        hitButton.addEventListener('click', hit);
        standButton.addEventListener('click', stand);
        doubleButton.addEventListener('click', doubleDown);
        splitButton.addEventListener('click', split);

        // Inicjalizacja przycisków
        hitButton.disabled = true;
        standButton.disabled = true;
        doubleButton.disabled = true;
        splitButton.disabled = true;
    </script>
</body>
</html>
