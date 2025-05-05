<template>
  <div>
    <h1>Game Purchase List</h1>
    <form @submit.prevent="addGame">
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

    <div>
      <label>
        <input type="checkbox" v-model="showUnpurchased" />
        Show only unpurchased games
      </label>
    </div>

    <div>
      <ul>
        <li
          v-for="(game, index) in filteredGames"
          :key="game.id"
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
      showUnpurchased: false, // Menambahkan properti untuk filter
    };
  },
  mounted() {
    this.loadGames();
  },
  computed: {
    filteredGames() {
      // Mengembalikan daftar game yang difilter berdasarkan status purchased
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
        purchased: false, // Menambahkan properti purchased
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
.completed {
  text-decoration: line-through;
  color: gray;
}
</style>
