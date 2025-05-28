<!DOCTYPE html>
<html lang="ko">
<head>
  <meta charset="UTF-8">
  <title>테트리스</title>
  <style>
    body {
      background: #111;
      color: #fff;
      font-family: sans-serif;
      display: flex;
      justify-content: center;
      padding: 30px;
    }

    .container {
      display: flex;
      gap: 30px;
    }

    .grid {
      display: grid;
      grid-template-columns: repeat(10, 30px);
      grid-template-rows: repeat(20, 30px);
      background: #000;
      border: 2px solid #444;
    }

    .grid div {
      width: 30px;
      height: 30px;
      box-sizing: border-box;
      border: 1px solid #222;
    }

    .key-log {
      background: #222;
      padding: 10px;
      font-size: 16px;
      border: 1px solid #444;
      min-width: 150px;
      min-height: 100px;
    }

    .block {
      background-color: gray;
    }
  </style>
</head>
<body>
  <div class="container">
    <div>
      <h1>테트리스</h1>
      <div class="grid" id="grid"></div>
    </div>
    <div>
      <h2>조작 상태</h2>
      <div class="key-log" id="key-log">Enter 키로 시작</div>
    </div>
  </div>

  <audio id="bgm" src="bgm.mp3" loop></audio>

  <script>
    const grid = document.getElementById('grid');
    const keyLog = document.getElementById('key-log');
    const bgm = document.getElementById('bgm');
    const width = 10;
    let squares = [];

    for (let i = 0; i < 200; i++) {
      const div = document.createElement('div');
      grid.appendChild(div);
      squares.push(div);
    }

    const colors = ["#f00", "#0f0", "#00f", "#ff0", "#0ff", "#f0f", "#ffa500"];

    const tetrominoes = [
      {
        shape: [[1, width+1, width*2+1, 2], [width, width+1, width+2, width*2+2], [1, width+1, width*2+1, width*2], [width, width*2, width*2+1, width*2+2]],
        color: colors[0]
      },
      {
        shape: [[1, width, width+1, width+2], [1, width+1, width+2, width*2+1], [width, width+1, width+2, width*2+1], [1, width, width+1, width*2+1]],
        color: colors[1]
      },
      {
        shape: [[0, 1, width+1, width+2], [1, width+1, width+2, width*2+1]],
        color: colors[2]
      },
      {
        shape: [[0, width, width+1, width*2+1]],
        color: colors[3]
      },
      {
        shape: [[1, width, width+1, width+2]],
        color: colors[4]
      }
    ];

    let currentTetromino = null;
    let current = null;
    let currentColor = "#fff";
    let currentPosition = 4;
    let currentRotation = 0;
    let interval = null;
    let previousIndex = -1;
    let gameOver = false;

    function getRandomTetromino() {
      let index;
      do {
        index = Math.floor(Math.random() * tetrominoes.length);
      } while (index === previousIndex);
      previousIndex = index;
      return tetrominoes[index];
    }

    function draw() {
      current.forEach(index => {
        const pos = currentPosition + index;
        if (pos < squares.length) {
          squares[pos].classList.add("block");
          squares[pos].style.backgroundColor = currentColor;
        }
      });
    }

    function undraw() {
      current.forEach(index => {
        const pos = currentPosition + index;
        if (pos < squares.length) {
          squares[pos].classList.remove("block");
          squares[pos].style.backgroundColor = "";
        }
      });
    }

    function freeze() {
      if (current.some(index => {
        const next = currentPosition + index + width;
        return next >= 200 || squares[next].classList.contains("block");
      })) {
        current.forEach(index => {
          squares[currentPosition + index].classList.add("block");
          squares[currentPosition + index].style.backgroundColor = currentColor;
        });
        spawnNewTetromino();
      }
    }

    function spawnNewTetromino() {
      currentTetromino = getRandomTetromino();
      currentRotation = 0;
      current = currentTetromino.shape[currentRotation];
      currentColor = currentTetromino.color;
      currentPosition = 4;

      // 게임 오버 판별
      if (current.some(index => squares[currentPosition + index].classList.contains("block"))) {
        clearInterval(interval);
        keyLog.textContent = "게임 오버";
        gameOver = true;
        return;
      }

      draw();
    }

    function moveDown() {
      if (gameOver || !current) return;
      undraw();
      currentPosition += width;

      if (current.some(index => {
        const next = currentPosition + index;
        return next >= 200 || squares[next].classList.contains("block");
      })) {
        currentPosition -= width;
        draw();
        freeze();
        return;
      }

      draw();
    }

    function moveLeft() {
      if (!current) return;
      undraw();
      if (!current.some(index => (currentPosition + index) % width === 0 &&
        squares[currentPosition + index - 1]?.classList.contains("block"))) {
        currentPosition--;
        draw();
      } else {
        draw();
      }
    }

    function moveRight() {
      if (!current) return;
      undraw();
      if (!current.some(index => (currentPosition + index) % width === width - 1 &&
        squares[currentPosition + index + 1]?.classList.contains("block"))) {
        currentPosition++;
        draw();
      } else {
        draw();
      }
    }

    function rotate() {
      if (!currentTetromino) return;
      const nextRotation = (currentRotation + 1) % currentTetromino.shape.length;
      const next = currentTetromino.shape[nextRotation];
      if (next.some(index => {
        const pos = currentPosition + index;
        return pos >= 200 || squares[pos].classList.contains("block");
      })) return;

      undraw();
      currentRotation = nextRotation;
      current = next;
      draw();
    }

    function startGame() {
      clearInterval(interval);
      gameOver = false;
      squares.forEach(s => {
        s.classList.remove("block");
        s.style.backgroundColor = "";
      });
      current = null;
      spawnNewTetromino();
      interval = setInterval(moveDown, 600);
      keyLog.textContent = "게임 시작됨";
      bgm.volume = 0.4;
      bgm.play();
    }

    document.addEventListener("keydown", (e) => {
      if (e.key === "Enter") startGame();
      if (!current) return;

      switch (e.key) {
        case "ArrowLeft":
          moveLeft();
          keyLog.textContent = "← 왼쪽 이동";
          break;
        case "ArrowRight":
          moveRight();
          keyLog.textContent = "→ 오른쪽 이동";
          break;
        case "ArrowDown":
          moveDown();
          keyLog.textContent = "↓ 아래 이동";
          break;
        case "ArrowUp":
          rotate();
          keyLog.textContent = "↑ 회전";
          break;
      }
    });
  </script>
</body>
</html>
