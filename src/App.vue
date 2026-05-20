<template>
  <div class="home">

    <!-- TOP -->
    <h1 class="main-title">
      Sandrine Puzzle Game
    </h1>

    <h2 class="sub-title">
      Select a Puzzle Challenge
    </h2>

    <!-- CARDS -->
    <div class="cards">

      <!-- EASY -->
      <div class="card">

        <img
          :src="require('./assets/easy.jpg')"
          class="card-image"
        />

        <h3>Easy</h3>

        <button @click="startLevel('easy', 3)">
          Play
        </button>

      </div>

      <!-- MEDIUM -->
      <div class="card">

        <img
          :src="require('./assets/medium.jpg')"
          class="card-image"
        />

        <h3>Medium</h3>

        <button @click="startLevel('medium', 4)">
          Play
        </button>

      </div>

      <!-- HARD -->
      <div class="card">

        <img
          :src="require('./assets/hard.jpg')"
          class="card-image"
        />

        <h3>Hard</h3>

        <button @click="startLevel('hard', 5)">
          Play
        </button>

      </div>

    </div>

    <!-- GAME -->
    <div v-if="gameStarted" class="game-section">

      <h1 class="game-title">
        Swap the Images to Win
      </h1>

      <!-- BUTTONS -->
      <div class="game-buttons">

        <button @click="shufflePuzzle">
          Start Game
        </button>

        <button @click="quitGame">
          Quit
        </button>

      </div>

      <!-- TIMER -->
      <p class="timer">
        Elapsed Time: {{ formattedTime }}
      </p>

      <!-- PUZZLE -->
      <div
        class="puzzle-container"
        :style="{
          gridTemplateColumns: `repeat(${size}, 1fr)`,
          width: boardSize + 'px',
          height: boardSize + 'px'
        }"
      >

        <div
          v-for="(piece, index) in pieces"
          :key="piece.id"
          class="piece"
          :class="{ empty: piece.empty }"
          :style="pieceStyle(piece)"
          @click="movePiece(index)"
        ></div>

      </div>

      <!-- RECORDS -->
      <h2 class="records">
        Records
      </h2>

    </div>

  </div>
</template>

<script>
export default {
  name: "App",

  data() {
    return {

      gameStarted: false,

      currentLevel: "easy",

      size: 3,

      boardSize: 650,

      image: require("./assets/easy.jpg"),

      pieces: [],

      seconds: 0,

      timer: null,

    };
  },

  computed: {

    formattedTime() {

      const hrs = String(
        Math.floor(this.seconds / 3600)
      ).padStart(2, "0");

      const mins = String(
        Math.floor((this.seconds % 3600) / 60)
      ).padStart(2, "0");

      const secs = String(
        this.seconds % 60
      ).padStart(2, "0");

      return `${hrs}:${mins}:${secs}`;
    },

    tileSize() {
      return this.boardSize / this.size;
    },
  },

  methods: {

    /* START LEVEL */
    startLevel(level, grid) {

      this.currentLevel = level;

      this.size = grid;

      this.image = require(`./assets/${level}.jpg`);

      this.gameStarted = true;

      this.createPuzzle();

      this.shufflePuzzle();

      this.startTimer();
    },

    /* CREATE PUZZLE */
    createPuzzle() {

      this.pieces = [];

      const total = this.size * this.size;

      for (let i = 0; i < total; i++) {

        this.pieces.push({
          id: i,
          correctIndex: i,
          empty: i === total - 1,
        });

      }
    },

    /* SHUFFLE */
    shufflePuzzle() {

      for (
        let i = this.pieces.length - 1;
        i > 0;
        i--
      ) {

        const j = Math.floor(
          Math.random() * (i + 1)
        );

        [
          this.pieces[i],
          this.pieces[j],
        ] = [
          this.pieces[j],
          this.pieces[i],
        ];
      }
    },

    /* MOVE PIECE */
    movePiece(index) {

      const emptyIndex = this.pieces.findIndex(
        (piece) => piece.empty
      );

      const validMoves = [
        emptyIndex - 1,
        emptyIndex + 1,
        emptyIndex - this.size,
        emptyIndex + this.size,
      ];

      if (!validMoves.includes(index)) return;

      const sameRow =
        Math.floor(index / this.size) ===
        Math.floor(emptyIndex / this.size);

      if (
        (
          index === emptyIndex - 1 ||
          index === emptyIndex + 1
        ) &&
        !sameRow
      ) {
        return;
      }

      [
        this.pieces[index],
        this.pieces[emptyIndex],
      ] = [
        this.pieces[emptyIndex],
        this.pieces[index],
      ];

      this.checkWin();
    },

    /* WIN */
    checkWin() {

      const won = this.pieces.every(
        (piece, index) =>
          piece.correctIndex === index
      );

      if (won) {

        clearInterval(this.timer);

        setTimeout(() => {

          alert(
            `You Won in ${this.formattedTime}`
          );

        }, 200);
      }
    },

    /* TIMER */
    startTimer() {

      clearInterval(this.timer);

      this.seconds = 0;

      this.timer = setInterval(() => {
        this.seconds++;
      }, 1000);
    },

    /* QUIT */
    quitGame() {

      clearInterval(this.timer);

      this.seconds = 0;

      this.gameStarted = false;
    },

    /* TILE STYLE */
    pieceStyle(piece) {

      if (piece.empty) {

        return {
          background: "transparent",
          border: "1px solid white",
        };
      }

      const row = Math.floor(
        piece.correctIndex / this.size
      );

      const col =
        piece.correctIndex % this.size;

      return {

        width: this.tileSize + "px",

        height: this.tileSize + "px",

        backgroundImage: `url(${this.image})`,

        backgroundSize: `
          ${this.boardSize}px
          ${this.boardSize}px
        `,

        backgroundPosition: `
          -${col * this.tileSize}px
          -${row * this.tileSize}px
        `,

        border: "1px solid white",
      };
    },
  },
};
</script>

<style scoped>

/* PAGE */
.home {
  min-height: 100vh;
  background: #efefef;
  padding: 20px;
  text-align: center;
  font-family: Arial, Helvetica, sans-serif;
}

/* TOP TITLES */
.main-title {
  font-size: 70px;
  color: #173f6c;
  font-weight: 900;
  margin-bottom: 10px;
}

.sub-title {
  font-size: 34px;
  color: #173f6c;
  margin-bottom: 40px;
  font-weight: 700;
}

/* CARDS */
.cards {
  display: flex;
  justify-content: center;
  gap: 30px;
  flex-wrap: wrap;
  margin-bottom: 70px;
}

.card {
  width: 460px;
  background: #ecd7d8;
  border-radius: 18px;
  padding: 22px;
}

.card-image {
  width: 100%;
  height: 250px;
  object-fit: cover;
  border-radius: 10px;
}

.card h3 {
  font-size: 60px;
  color: #173f6c;
  margin: 25px 0;
  font-weight: 900;
}

/* BUTTONS */
button {
  width: 220px;
  height: 70px;
  background: #2d5b2d;
  color: white;
  border: none;
  border-radius: 10px;
  font-size: 28px;
  font-weight: bold;
  cursor: pointer;
  transition: 0.3s;
}

button:hover {
  background: #234723;
}

/* GAME */
.game-section {
  margin-top: 60px;
}

.game-title {
  font-size: 70px;
  color: #173f6c;
  font-weight: 900;
  margin-bottom: 40px;
}

.game-buttons {
  display: flex;
  justify-content: center;
  gap: 20px;
  margin-bottom: 30px;
}

.timer {
  font-size: 34px;
  color: #173f6c;
  margin-bottom: 40px;
}

/* PUZZLE */
.puzzle-container {
  display: grid;
  margin: auto;
  border: 3px solid white;
}

.piece {
  cursor: pointer;
  transition: 0.2s;
}

.piece:hover {
  opacity: 0.9;
}

.empty {
  background: transparent !important;
}

/* RECORDS */
.records {
  margin-top: 60px;
  font-size: 60px;
  color: #173f6c;
  font-weight: 900;
}

/* MOBILE */
@media (max-width: 768px) {

  .main-title,
  .game-title {
    font-size: 40px;
  }

  .sub-title {
    font-size: 24px;
  }

  .card {
    width: 100%;
  }

  .card h3 {
    font-size: 42px;
  }

  button {
    width: 170px;
    height: 58px;
    font-size: 20px;
  }

  .puzzle-container {
    width: 95vw !important;
    height: 95vw !important;
  }
}
</style>