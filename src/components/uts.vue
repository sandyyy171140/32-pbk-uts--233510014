<script>
export default {
  name: 'GamePurchaseList',
  data() {
    return {
      games: [],
    };
  },
  mounted() {
    this.loadGames();
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
        }
      } catch (e) {
        console.error("Failed to load data:", e);
      }
    },
  },
};
</script>

<template>
    <div>
      <h1>Game Purchase List</h1>
      <div>
        <ul>
          <li
            v-for="(game, index) in games"
            :key="game.id"
          >
            <span>{{ game.name }}</span> - <span>Rp {{ formatPrice(game.price) }}</span>
          </li>
        </ul>
        <p v-if="games.length === 0">No games added yet</p>
      </div>
    </div>
  </template>
  

  