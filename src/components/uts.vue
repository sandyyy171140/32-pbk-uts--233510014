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
      <button type="submit" class="add-button">Add Game</button>
    </form>

    <div class="filter-container">
      <label class="filter-label">
        <input type="checkbox" v-model="showUnpurchased" class="filter-checkbox" />
        Show only unpurchased games
      </label>
    </div>

    <div class="game-list-container">
      <ul class="game-list">
        <li
          v-for="(game, index) in filteredGames"
          :key="game.id"
          class="game-item"
          :class="{ purchased: game.purchased }"
        >
          <div class="game-checkbox">
            <input
              type="checkbox"
              v-model="game.purchased"
              :id="'game-'+game.id"
              class="purchase-checkbox"
            />
            <label :for="'game-'+game.id"></label>
          </div>
          <div class="game-info">
            <span class="game-name" :class="{ completed: game.purchased }">{{ game.name }}</span>
            <span class="game-price">Rp {{ formatPrice(game.price) }}</span>
          </div>
          <button @click="removeGame(index)" class="remove-button">
            <svg xmlns="http://www.w3.org/2000/svg" width="16" height="16" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2" stroke-linecap="round" stroke-linejoin="round">
              <line x1="18" y1="6" x2="6" y2="18"></line>
              <line x1="6" y1="6" x2="18" y2="18"></line>
            </svg>
          </button>
        </li>
      </ul>
      <p v-if="filteredGames.length === 0" class="empty-message">No games added yet</p>
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
:root {
  --primary: #4361ee;
  --primary-dark: #3a0ca3;
  --secondary: #f72585;
  --dark: #1a1a2e;
  --light: #f8f9fa;
  --success: #4cc9f0;
  --warning: #f8961e;
  --danger: #ef233c;
  --gray: #6c757d;
  --light-gray: #e9ecef;
}

body {
  font-family: 'Segoe UI', Tahoma, Geneva, Verdana, sans-serif;
  background-color: #f8f9fa;
  margin: 0;
  padding: 20px;
  color: var(--dark);
}

.app-container {
  max-width: 600px;
  margin: 0 auto;
  background: white;
  padding: 2rem;
  border-radius: 12px;
  box-shadow: 0 4px 20px rgba(0, 0, 0, 0.1);
}

h1 {
  text-align: center;
  color: var(--primary);
  margin-bottom: 1.5rem;
  font-weight: 600;
}

/* Form Styles */
.game-form {
  display: flex;
  gap: 10px;
  margin-bottom: 1.5rem;
}

.game-form input[type="text"],
.game-form input[type="number"] {
  flex: 1;
  padding: 0.75rem 1rem;
  border: 1px solid var(--light-gray);
  border-radius: 8px;
  font-size: 1rem;
  transition: all 0.3s ease;
}

.game-form input[type="text"]:focus,
.game-form input[type="number"]:focus {
  outline: none;
  border-color: var(--primary);
  box-shadow: 0 0 0 3px rgba(67, 97, 238, 0.2);
}

/* Button Styles */
button {
  border: none;
  border-radius: 8px;
  padding: 0.75rem 1.25rem;
  font-weight: 600;
  cursor: pointer;
  transition: all 0.3s ease;
  display: flex;
  align-items: center;
  justify-content: center;
}

.add-button {
  background-color: var(--primary);
  color: white;
}

.add-button:hover {
  background-color: var(--primary-dark);
  transform: translateY(-2px);
}

.remove-button {
  background-color: transparent;
  color: var(--danger);
  padding: 0.5rem;
  border-radius: 50%;
  width: 32px;
  height: 32px;
}

.remove-button:hover {
  background-color: rgba(239, 35, 60, 0.1);
}

/* Filter Styles */
.filter-container {
  margin-bottom: 1.5rem;
  padding: 0.75rem;
  background-color: var(--light-gray);
  border-radius: 8px;
}

.filter-label {
  display: flex;
  align-items: center;
  gap: 8px;
  cursor: pointer;
}

.filter-checkbox {
  width: 18px;
  height: 18px;
  accent-color: var(--primary);
}

/* Game List Styles */
.game-list-container {
  max-height: 400px;
  overflow-y: auto;
  padding-right: 8px;
}

.game-list {
  list-style: none;
  padding: 0;
  margin: 0;
}

.game-item {
  display: flex;
  align-items: center;
  gap: 12px;
  padding: 1rem;
  background-color: white;
  border-radius: 8px;
  margin-bottom: 0.75rem;
  box-shadow: 0 2px 4px rgba(0, 0, 0, 0.05);
  transition: all 0.3s ease;
}

.game-item:hover {
  transform: translateY(-2px);
  box-shadow: 0 4px 8px rgba(0, 0, 0, 0.1);
}

.game-item.purchased {
  opacity: 0.7;
  background-color: var(--light-gray);
}

.game-checkbox {
  position: relative;
}

.purchase-checkbox {
  opacity: 0;
  position: absolute;
}

.purchase-checkbox + label {
  display: inline-block;
  width: 20px;
  height: 20px;
  border: 2px solid var(--primary);
  border-radius: 4px;
  cursor: pointer;
  position: relative;
}

.purchase-checkbox:checked + label {
  background-color: var(--primary);
}

.purchase-checkbox:checked + label::after {
  content: '';
  position: absolute;
  left: 50%;
  top: 50%;
  transform: translate(-50%, -50%);
  width: 12px;
  height: 12px;
  background-image: url("data:image/svg+xml,%3Csvg xmlns='http://www.w3.org/2000/svg' viewBox='0 0 24 24' fill='none' stroke='white' stroke-width='3' stroke-linecap='round' stroke-linejoin='round'%3E%3Cpolyline points='20 6 9 17 4 12'%3E%3C/polyline%3E%3C/svg%3E");
  background-repeat: no-repeat;
}

.game-info {
  flex: 1;
  display: flex;
  justify-content: space-between;
  align-items: center;
  gap: 1rem;
}

.game-name {
  font-weight: 500;
}

.game-name.completed {
  text-decoration: line-through;
  color: var(--gray);
}

.game-price {
  font-weight: 600;
  color: var(--success);
  white-space: nowrap;
}

.empty-message {
  text-align: center;
  color: var(--gray);
  padding: 1.5rem;
  font-style: italic;
}

/* Scrollbar Styles */
::-webkit-scrollbar {
  width: 8px;
}

::-webkit-scrollbar-track {
  background: var(--light-gray);
  border-radius: 4px;
}

::-webkit-scrollbar-thumb {
  background: var(--primary);
  border-radius: 4px;
}

::-webkit-scrollbar-thumb:hover {
  background: var(--primary-dark);
}
</style>