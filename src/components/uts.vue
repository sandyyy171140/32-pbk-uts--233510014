<template>
  <div class="app-container">
    <h1>Game Purchase List</h1>
    <form @submit.prevent="addGame" class="game-form">
      <input
        v-model.trim="newGameName"
        type="text"
        placeholder="Game Title"
        required
      />
      <input
        v-model.number="newGamePrice"
        type="number"
        placeholder="Price (Rp)"
        min="0"
        step="1000"
        required
      />
      <button type="submit">Add Game</button>
    </form>

    <div class="filter-container">
      <label>
        <input type="checkbox" v-model="showUnpurchased" />
        Show only unpurchased games
      </label>
    </div>

    <div>
      <ul class="game-list">
        <li
          v-for="(game, index) in filteredGames"
          :key="game.id"
          class="game-item"
        >
          <input
            type="checkbox"
            v-model="game.purchased"
          />
          <span :class="{ completed: game.purchased }">{{ game.name }}</span> - <span>Rp {{ formatPrice(game.price) }}</span>
          <button @click="removeGame(index)">Remove</button>
        </li>
      </ul>
      <p v-if="filteredGames.length === 0">No games added yet</p>
    </div>
  </div>
</template>

<script>
export default {
  name: 'GamePurchaseList',
  data() {
    return {
      newGameName: '',
      newGamePrice: '',
      games: [],
      idCounter: 0,
      showUnpurchased: false,
    };
  },
  mounted() {
    this.loadGames();
  },
  computed: {
    filteredGames() {
      return this.showUnpurchased
        ? this.games.filter(game => !game.purchased)
        : this.games;
    },
  },
  methods: {
    formatPrice(value) {
      if (!value) return "0";
      return value.toString().replace(/\B(?=(\d{3})+(?!\d))/g, ".");
    },
    loadGames() {
      try {
        const savedGames = localStorage.getItem('purchasedGames');
        if (savedGames) {
          this.games = JSON.parse(savedGames);
          if (this.games.length > 0) {
            this.idCounter = Math.max(...this.games.map(g => g.id));
          }
        }
      } catch (e) {
        console.error("Failed to load data:", e);
      }
    },
    addGame() {
      if (!this.newGameName.trim()) {
        alert("Game title cannot be empty");
        return;
      }

      const price = parseFloat(this.newGamePrice);
      if (isNaN(price) || price < 0) {
        alert("Price must be a positive number");
        return;
      }

      const newGame = {
        id: ++this.idCounter,
        name: this.newGameName.trim(),
        price: price,
        purchased: false,
      };

      this.games.push(newGame);
      this.saveGames();
      this.newGameName = '';
      this.newGamePrice = '';
    },
    removeGame(index) {
      if (confirm(`Are you sure you want to remove "${this.games[index].name}"?`)) {
        this.games.splice(index, 1);
        this.saveGames();
      }
    },
    saveGames() {
      localStorage.setItem('purchasedGames', JSON.stringify(this.games));
    },
  },
};
</script>

<style>
body {
  font-family: Arial, sans-serif;
  background-color: #f4f4f4;
  margin: 0;
  padding: 20px;
}

.app-container {
  max-width: 600px;
  margin: 0 auto;
  background: white;
  padding: 20px;
  border-radius: 8px;
  box-shadow: 0 2px 10px rgba(0, 0, 0, 0.1);
}

h1 {
  text-align: center;
  color: #333;
}

.game-form {
  display: flex;
  justify-content: space-between;
  margin-bottom: 20px;
}

.game-form input[type="text"],
.game-form input[type="number"] {
  flex: 1;
  padding: 10px;
  margin-right: 10px;
  border: 1px solid #ccc;
  border-radius: 4px;
}

.game-form button {
  padding: 10px 15px;
}
</style>