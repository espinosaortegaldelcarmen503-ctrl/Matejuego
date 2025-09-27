# Matejuego
Juego de matematicas 
@@ -0,0 +1,216 @@
<!DOCTYPE html>
<html lang="es">
<head>
  <meta charset="UTF-8" />
  <meta name="viewport" content="width=device-width, initial-scale=1.0"/>
  <title>Matemáticas en Acción</title>
  <style>
    body {
      font-family: Arial, sans-serif;
      text-align: center;
      background-color: #f0f0f0;
    }

    h1 {
      color: #2d2d2d;
    }

    #board {
      display: grid;
      grid-template-columns: repeat(10, 50px);
      grid-template-rows: repeat(10, 50px);
      gap: 2px;
      margin: 20px auto;
      width: fit-content;
    }

    .cell {
      width: 50px;
      height: 50px;
      border: 1px solid #444;
      font-size: 10px;
      position: relative;
      display: flex;
      justify-content: center;
      align-items: center;
      color: #000;
    }

    .pink { background-color: pink; }
    .blue { background-color: lightblue; }
    .green { background-color: lightgreen; }
    .yellow { background-color: #fff69b; }
    .special { background-color: orange !important; font-weight: bold; }

    .player {
      width: 15px;
      height: 15px;
      border-radius: 50%;
      position: absolute;
    }

    .player1 { background-color: red; top: 5px; left: 5px; }
    .player2 { background-color: blue; top: 5px; right: 5px; }

    button {
      margin-top: 15px;
      padding: 10px 20px;
      font-size: 16px;
    }

    #status, #dice, #cardMessage {
      margin-top: 10px;
      font-weight: bold;
    }

    #cardMessage {
      font-style: italic;
      color: darkgreen;
    }
  </style>
</head>
<body>

  <h1>Matemáticas en Acción: Un Juego de Estrategia y Azar</h1>

  <div id="board"></div>

  <button onclick="rollDice()">🎲 Tirar Dados</button>

  <p id="dice"></p>
  <p id="status">Turno del Jugador 1</p>
  <p id="cardMessage"></p>

  <script>
    const board = document.getElementById("board");
    const status = document.getElementById("status");
    const diceDisplay = document.getElementById("dice");
    const cardMessage = document.getElementById("cardMessage");

    const operations = ['pink', 'blue', 'green', 'yellow'];

    const players = [
      { position: 1, element: document.createElement("div"), name: "Jugador 1" },
      { position: 1, element: document.createElement("div"), name: "Jugador 2" }
    ];
    let currentPlayer = 0;

    const cells = [];
    for (let row = 9; row >= 0; row--) {
      for (let col = 0; col < 10; col++) {
        const cellNum = row % 2 === 0
          ? row * 10 + col + 1
          : row * 10 + (9 - col) + 1;

        const cell = document.createElement("div");
        cell.classList.add("cell");

        if (cellNum === 1 || cellNum === 99) {
          cell.classList.add("special");
        } else if (![1, 99, 100].includes(cellNum)) {
          const color = operations[Math.floor(Math.random() * operations.length)];
          cell.classList.add(color);
        }

        cell.dataset.cellNum = cellNum;
        cell.textContent = cellNum;
        board.appendChild(cell);
        cells[cellNum] = cell;
      }
    }

    // Agregar jugadores
    players.forEach((player, index) => {
      player.element.classList.add("player", `player${index + 1}`);
      movePlayer(player, 1);
    });

    function rollDice() {
      const player = players[currentPlayer];
      const die1 = Math.ceil(Math.random() * 6);
      const die2 = Math.ceil(Math.random() * 6);
      diceDisplay.textContent = `${player.name} tiró 🎲 ${die1} y 🎲 ${die2}`;

      // Determinar operación según la casilla actual
      const currentCell = cells[player.position];
      let moveBy = die1 + die2;
      let operation = 'normal';

      if (currentCell.classList.contains('pink')) {
        moveBy = die1 + die2;
        operation = 'Suma';
      } else if (currentCell.classList.contains('blue')) {
        moveBy = Math.max(die1 - die2, 1);
        operation = 'Resta';
      } else if (currentCell.classList.contains('green')) {
        moveBy = die1 * die2;
        operation = 'Multiplicación';
      } else if (currentCell.classList.contains('yellow')) {
        moveBy = 1;
        operation = 'Potencia de cero';
      }

      let newPos = player.position + moveBy;
      if (newPos > 100) newPos = 100;

      movePlayer(player, newPos);

      // Cartas en casillas múltiplos de 10 (excepto 100)
      if (newPos % 10 === 0 && newPos !== 100) {
        const effects = [
          "Regresa al inicio",
          "Avanza 8 casillas",
          "Retrocede 7 casillas",
          "Recupera una carta"
        ];
        const effect = effects[Math.floor(Math.random() * effects.length)];
        cardMessage.textContent = `🃏 Carta obtenida: ${effect}`;

        if (effect === "Regresa al inicio") movePlayer(player, 1);
        else if (effect === "Avanza 8 casillas") movePlayer(player, Math.min(100, player.position + 8));
        else if (effect === "Retrocede 7 casillas") movePlayer(player, Math.max(1, player.position - 7));
        // "Recupera una carta" es decorativa por ahora
      } else {
        cardMessage.textContent = "";
      }

      // Casillas especiales
      if (player.position === 99) {
        movePlayer(player, 1);
        status.textContent = `${player.name} cayó en la casilla 99 y regresa al inicio.`;
      } else if (player.position === 1 && newPos !== 1) {
        movePlayer(player, Math.min(100, player.position + 20));
        status.textContent = `${player.name} cayó en la casilla 1 y avanza libremente.`;
      } else {
        status.textContent = `${player.name} usó ${operation} y llegó a la casilla ${player.position}.`;
      }

      if (player.position === 100) {
        alert(`${player.name} ha ganado el juego! 🎉`);
        resetGame();
        return;
      }

      currentPlayer = (currentPlayer + 1) % players.length;
      status.textContent += ` Turno del ${players[currentPlayer].name}.`;
    }

    function movePlayer(player, pos) {
      if (cells[player.position]?.contains(player.element)) {
        cells[player.position].removeChild(player.element);
      }
      cells[pos].appendChild(player.element);
      player.position = pos;
    }

    function resetGame() {
      players.forEach(p => movePlayer(p, 1));
      currentPlayer = 0;
      status.textContent = "Nuevo juego. Turno del Jugador 1.";
      diceDisplay.textContent = "";
      cardMessage.textContent = "";
    }
  </script>

</body>
</html>
