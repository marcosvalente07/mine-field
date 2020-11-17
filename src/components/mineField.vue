<template>
  <div id="mine-field">
    <div class="d-flex align-items-center justify-content-center p-3">
      <button
        type="button"
        class="btn btn-black mr-3"
        @click="setBeginnerDifficulty"
      >
        Iniciante
      </button>
      <button
        type="button"
        class="btn btn-black mr-3"
        @click="setIntermediateDifficulty"
      >
        Intermediário
      </button>
      <button
        type="button"
        class="btn btn-black mr-3"
        @click="setExpertDifficulty"
      >
        Avançado
      </button>
    </div>
    <div class="d-flex flex-row justify-content-center">
      <main :style="cssVars">
        <div class="d-flex align-items-center justify-content-between p-3">
          <div class="font-white"><span class="fa fa-bomb mr-1"></span> {{ addLeadingZerosBombsRemaining }}</div>
          <button class="btn btn-black" @click="resetGame">
            {{ gameStatus }}
          </button>
          <div class="font-white"><span class="fa fa-clock-o mr-1"></span>{{ addLeadingZerosSecondsPassed }}</div>
        </div>
        <div class="board">
          <div
            class="tile"
            v-for="(tile, i) in tiles"
            :key="i"
            :class="{ revealed: tile.revealed, 'wrong-pick': gameFailed &amp;&amp; ((tile.bomb &amp;&amp; tile.revealed) || (!tile.bomb &amp;&amp; tile.flagged))}"
            :data-surrounding-bombs="tile.surroundingBombs"
            @click="reveal(i)"
            @contextmenu.prevent="flag(i)"
          >
            <template v-if="tile.flagged">🔴</template>
            <template v-else-if="tile.revealed &amp;&amp; tile.bomb"
              >💣</template
            >
            <template
              v-else-if="tile.revealed &amp;&amp; tile.surroundingBombs"
              >{{ tile.surroundingBombs }}</template
            >
          </div>
        </div>
      </main>
    </div>
  </div>
</template>

<script>
const explosionAudio = new Audio(
  "https://www.myinstants.com/media/sounds/que-burro-andre-henning-zoa-ribery-apos-expulsao-256kbit.mp3"
);
const winAudio = new Audio(
  "https://www.myinstants.com/media/sounds/hallelujahshort.swf.mp3"
);

const getIndex = (row, column, config) => {
  if (row < 0) return;
  if (column < 0) return;
  if (row >= config.height) return;
  if (column >= config.width) return;

  return row * config.width + column;
};

const getTileCoordinates = (index, config) => ({
  row: Math.floor(index / config.width),
  column: index % config.width,
});

const generateTiles = (config) => {
  const bombs = Array.from({ length: config.width * config.height });

  let bombsPlanted = 0;
  while (bombsPlanted != config.totalNumberOfBombs) {
    const index = Math.floor(Math.random() * config.width * config.height);

    if (!bombs[index]) {
      bombs[index] = true;
      bombsPlanted++;
    }
  }

  return bombs.map((bomb, i, array) => {
    const { row, column } = getTileCoordinates(i, config);

    let surroundingBombs = 0;
    if (array[getIndex(row - 1, column - 1, config)]) surroundingBombs++;
    if (array[getIndex(row - 1, column - 0, config)]) surroundingBombs++;
    if (array[getIndex(row - 1, column + 1, config)]) surroundingBombs++;
    if (array[getIndex(row - 0, column - 1, config)]) surroundingBombs++;
    if (array[getIndex(row - 0, column + 1, config)]) surroundingBombs++;
    if (array[getIndex(row + 1, column - 1, config)]) surroundingBombs++;
    if (array[getIndex(row + 1, column - 0, config)]) surroundingBombs++;
    if (array[getIndex(row + 1, column + 1, config)]) surroundingBombs++;

    return {
      bomb,
      flagged: false,
      revealed: false,
      surroundingBombs,
    };
  });
};

export default {
  data: () => ({
    config: {},
    tiles: [],

    secondsPassed: 0,
    timerIntervalID: undefined,
  }),
  created() {
    this.setBeginnerDifficulty();
  },
  computed: {
    addLeadingZerosBombsRemaining() {
      return ("00" + this.bombsRemaining).slice(-3);
    },
    addLeadingZerosSecondsPassed() {
      return ("00" + this.secondsPassed).slice(-3);
    },
    cssVars() {
      return {
        "--width": this.config.width,
        "--height": this.config.height,
        "--size": `${this.config.size}px`,
      };
    },
    bombsRemaining() {
      const numberOfFlaggedTiles = this.tiles.filter((t) => t.flagged).length;
      return this.config.totalNumberOfBombs - numberOfFlaggedTiles;
    },
    gameIsProgress() {
      if (this.gameStatus == "😞" || this.gameStatus == "😎") return false;
      if (!this.tiles.find((tile) => tile.revealed)) return false;
      return true;
    },
    gameWon() {
      const numberOfRevealedTiles = this.tiles.filter((t) => t.revealed).length;

      const numberOfTilesThatNeedsToBeRevealed =
        this.config.width * this.config.height - this.config.totalNumberOfBombs;

      return numberOfRevealedTiles == numberOfTilesThatNeedsToBeRevealed;
    },
    gameFailed() {
      return this.tiles.find((tile) => tile.bomb && tile.revealed);
    },
    gameStatus() {
      if (this.gameFailed) return "😞";
      if (this.gameWon) return "😎";
      return "🙂";
    },
  },
  watch: {
    gameIsProgress(value) {
      if (value) {
        this.timerIntervalID = setInterval(() => {
          this.secondsPassed++;
        }, 1000);
      } else {
        clearInterval(this.timerIntervalID);
      }
    },
    gameStatus(value) {
      if (value == "😎") {
        winAudio.play();
      }
      if (value == "😞") {
        explosionAudio.play();
      }
    },
  },
  methods: {
    setCoffeeMilkDifficulty() {
      this.config = {
        width: 4,
        height: 4,
        totalNumberOfBombs: 1,
        size: 40,
      };
      this.resetGame();
    },
    setBeginnerDifficulty() {
      this.config = {
        width: 8,
        height: 8,
        totalNumberOfBombs: 10,
        size: 40,
      };
      this.resetGame();
    },
    setIntermediateDifficulty() {
      this.config = {
        width: 18,
        height: 12,
        totalNumberOfBombs: 40,
        size: 35,
      };
      this.resetGame();
    },
    setExpertDifficulty() {
      this.config = {
        width: 30,
        height: 16,
        totalNumberOfBombs: 99,
        size: 25,
      };
      this.resetGame();
    },
    resetGame() {
      this.tiles = generateTiles(this.config);

      this.secondsPassed = 0;
    },
    reveal(i) {
      if (this.gameFailed) return;

      if (i == undefined) return;

      const tile = this.tiles[i];

      if (tile.flagged) return;

      if (!tile.revealed) {
        tile.revealed = true;

        if (!tile.bomb && tile.surroundingBombs == 0) {
          const { row, column } = getTileCoordinates(i, this.config);

          this.reveal(getIndex(row - 1, column - 1, this.config));
          this.reveal(getIndex(row - 1, column - 0, this.config));
          this.reveal(getIndex(row - 1, column + 1, this.config));
          this.reveal(getIndex(row - 0, column - 1, this.config));
          this.reveal(getIndex(row - 0, column + 1, this.config));
          this.reveal(getIndex(row + 1, column - 1, this.config));
          this.reveal(getIndex(row + 1, column - 0, this.config));
          this.reveal(getIndex(row + 1, column + 1, this.config));
        }
      }
    },
    flag(i) {
      if (this.gameFailed) return;

      if (this.tiles[i].revealed) return;

      this.tiles[i].flagged = !this.tiles[i].flagged;
    },
  },
};
</script>

<style scoped>
#mine-field {
  overflow: auto;
}
.board {
  display: grid;
  grid-template-columns: repeat(var(--width), auto);
  grid-template-rows: repeat(var(--height), auto);
  user-select: none;
  background-color: #ffe53b;
  background-image: linear-gradient(147deg, #ffe53b 0%, #ff2525 74%);
  background-size: 100%, 400%;
  position: relative;
}
.tile {
  width: var(--size);
  height: var(--size);
  line-height: var(--size);
}
.tile:not(.revealed) {
  box-shadow: inset calc(var(--size) / 15) calc(var(--size) / 15) 0px 0px
      rgba(255, 255, 255, 0.45),
    inset calc(calc(var(--size) / 12.5) * -1)
      calc(calc(var(--size) / 12.5) * -1) 0px 0px rgba(0, 0, 0, 0.25);
  cursor: pointer;
}
.tile.revealed {
  border: 0.5px solid #bdbdbd;
  box-sizing: border-box;
}
.tile.wrong-pick {
  background-color: lightcoral;
}
.tile[data-surrounding-bombs="1"] {
  color: white;
}
.tile[data-surrounding-bombs="2"] {
  color: white;
}
.tile[data-surrounding-bombs="3"] {
  color: white;
}
.tile[data-surrounding-bombs="4"] {
  color: white;
}
.tile[data-surrounding-bombs="5"] {
  color: white;
}
.tile[data-surrounding-bombs="6"] {
  color: white;
}
.tile[data-surrounding-bombs="7"] {
  color: white;
}
.tile[data-surrounding-bombs="8"] {
  color: white;
}
.btn-black {
  color: white;
  background-color: #ffe53b;
  background-image: linear-gradient(147deg, #ffe53b 0%, #ff2525 74%);
  background-size: 100%, 400%;
  position: relative;
}
.btn-black:hover {
  color: white;
}
</style>
