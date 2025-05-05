<template>
  <div class="app-container">
    <div class="container">
      <h1> Game Purchase List</h1>
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
        <button type="submit" class="add-button">Add Game</button>
      </form>

      <div class="game-list-container">
        <ul class="game-list">
          <li
            v-for="(game, index) in games"
            :key="game.id"
            :class="['game-item', { completed: game.purchased }]"
          >
            <div class="game-checkbox">
              <input
                type="checkbox"
                v-model="game.purchased"
                :id="'game-'+game.id"
              />
              <label :for="'game-'+game.id"></label>
            </div>
            <div class="game-info">
              <span class="name">{{ game.name }}</span>
              <span class="price">Rp {{ formatPrice(game.price) }}</span>
            </div>
            <button @click="removeGame(index)" class="remove-button" aria-label="Remove game">
              <svg xmlns="http://www.w3.org/2000/svg" width="16" height="16" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2" stroke-linecap="round" stroke-linejoin="round" aria-hidden="true">
                <line x1="18" y1="6" x2="6" y2="18"></line>
                <line x1="6" y1="6" x2="18" y2="18"></line>
              </svg>
            </button>
          </li>
        </ul>
        <p v-if="games.length === 0" class="no-games">No games added yet</p>
      </div>

      <button @click="savePurchased" class="save-button" :disabled="!hasPurchased">
        Save Purchased Games
      </button>
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
    };
  },
  computed: {
    hasPurchased() {
      return this.games.some(game => game.purchased);
    },
  },
  mounted() {
    this.loadGames();
  },
  methods: {
    formatPrice(value) {
      if (!value) return "0";
      return value.toString().replace(/\B(?=(\d{3})+(?!\d))/g, ".");
    },
    generateId() {
      this.idCounter++;
      return this.idCounter;
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

      this.games.push({
        id: this.generateId(),
        name: this.newGameName.trim(),
        price: price,
        purchased: false,
      });

      this.newGameName = '';
      this.newGamePrice = '';
    },
    removeGame(index) {
      if (confirm(`Delete ${this.games[index].name}?`)) {
        this.games.splice(index, 1);
      }
    },
    savePurchased() {
      const purchasedGames = this.games.filter(game => game.purchased);
      if (purchasedGames.length === 0) {
        alert("No games selected for saving");
        return;
      }

      try {
        localStorage.setItem('purchasedGames', JSON.stringify(purchasedGames));
        const listText = purchasedGames.map(g =>
          `${g.name} (Rp ${this.formatPrice(g.price)})`
        ).join('\\n');
        alert(`Saved ${purchasedGames.length} games:\n\n${listText}`);
      } catch (e) {
        console.error("Save failed:", e);
        alert("Failed to save data. Please try again.");
      }
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
  --darker: #16213e;
  --light: #f8f9fa;
  --success: #4cc9f0;
  --warning: #f8961e;
  --danger: #ef233c;
  --gray: #6c757d;
  --dark-gray: #343a40;
}

* {
  box-sizing: border-box;
  margin: 0;
  padding: 0;
}

body {
  font-family: 'Segoe UI', Tahoma, Geneva, Verdana, sans-serif;
  background: linear-gradient(135deg, #0f0c29, #302b63, #24243e);
  color: var(--light);
  min-height: 100vh;
  display: flex;
  justify-content: center;
  align-items: flex-start;
  padding: 2rem;
  line-height: 1.6;
}

.app-container {
  position: relative;
  width: 100%;
  max-width: 600px;
  min-height: 100vh;
  margin: 0 auto;
  padding: 2rem 1.5rem 3rem 1.5rem;
  overflow: visible;
  z-index: 0;
}

.app-container::before {
  content: "";
  position: fixed;
  top: 0;
  left: 0;
  width: 100vw;
  height: 100vh;
  background: url('https://wallpaperaccess.com/full/306739.jpg') no-repeat center center/cover;
  filter: brightness(0.7) blur(4px);
  z-index: -1;
  pointer-events: none;
}

/* Container */
.container {
  position: relative;
  background: url('https://images.akamai.steamusercontent.com/ugc/493519450183038433/7766E6F16031671F6F1049919D0A28AA8E6B9599/')
    no-repeat center center/cover;
  border-radius: 16px;
  padding: 2rem;
  box-shadow: 0 8px 32px rgba(0, 0, 0, 0.3);
  border: 1px solid rgba(255, 255, 255, 0.1);
  animation: fadeIn 0.5s ease-out;
  overflow: visible;
}

.container::before {
  content: "";
  position: absolute;
  inset: 0;
  background-color: rgba(26, 26, 46, 0.6);
  border-radius: 16px;
  z-index: 0;
}

.container > * {
  position: relative;
  z-index: 1;
}

@keyframes fadeIn {
  from { opacity: 0; transform: translateY(20px); }
  to { opacity: 1; transform: translateY(0); }
}

/* Header */
h1 {
  font-size: 2rem;
  font-weight: 700;
  color: var(--light);
  text-align: center;
  margin-bottom: 1.5rem;
  background: linear-gradient(to right, #4361ee, #4cc9f0);
  -webkit-background-clip: text;
  background-clip: text;
  color: transparent;
  position: relative;
  padding-bottom: 0.5rem;
}

h1::after {
  content: '';
  position: absolute;
  bottom: 0;
  left: 50%;
  transform: translateX(-50%);
  width: 80px;
  height: 3px;
  background: linear-gradient(to right, #4361ee, #4cc9f0);
  border-radius: 3px;
}

/* Form */
form {
  display: grid;
  grid-template-columns: 1fr 1fr auto;
  gap: 1rem;
  margin-bottom: 1.5rem;
}

input {
  padding: 0.75rem 1rem;
  border: none;
  border-radius: 8px;
  background: rgba(255, 255, 255, 0.1);
  color: var(--light);
  font-size: 1rem;
  transition: all 0.3s ease;
  min-width: 0;
}

input:focus {
  outline: none;
  background: rgba(255, 255, 255, 0.2);
  box-shadow: 0 0 0 2px var(--primary);
}

input::placeholder {
  color: rgba(255, 255, 255, 0.5);
}

/* Buttons */
button {
  border: none;
  border-radius: 8px;
  padding: 0.75rem 1rem;
  font-weight: 600;
  cursor: pointer;
  transition: all 0.3s ease;
  display: flex;
  align-items: center;
  justify-content: center;
  gap: 0.5rem;
}

.add-button {
  background: var(--primary);
  color: white;
}

.add-button:hover {
  background: var(--primary-dark);
  transform: translateY(-2px);
}

.save-button {
  width: 100%;
  background: var(--success);
  color: var(--dark);
  font-size: 1rem;
  margin-top: 1.5rem;
  padding: 1rem;
}

.save-button:hover:not(:disabled) {
  background: #3aa8d8;
  transform: translateY(-2px);
}

.save-button:disabled {
  background: var(--dark-gray);
  color: var(--gray);
  cursor: not-allowed;
}

.remove-button {
  background: transparent;
  color: var(--danger);
  padding: 0.5rem;
  border-radius: 50%;
  width: 32px;
  height: 32px;
}

.remove-button:hover {
  background: rgba(239, 35, 60, 0.2);
}

/* Game List */
.game-list-container {
  max-height: 400px;
  overflow-y: auto;
  padding-right: 0.5rem;
  margin-bottom: 1rem;
}

.game-list {
  list-style: none;
  margin: 0;
  padding: 0;
}

.game-item {
  display: flex;
  align-items: center;
  gap: 1rem;
  padding: 1rem;
  background: rgba(255, 255, 255, 0.05);
  border-radius: 8px;
  margin-bottom: 0.75rem;
  transition: all 0.3s ease;
}

.game-item:hover {
  background: rgba(255, 255, 255, 0.1);
  transform: translateX(5px);
}

.game-checkbox {
  position: relative;
}

.game-checkbox input[type="checkbox"] {
  opacity: 0;
  position: absolute;
}

.game-checkbox label {
  display: inline-block;
  width: 20px;
  height: 20px;
  border: 2px solid var(--primary);
  border-radius: 4px;
  cursor: pointer;
  position: relative;
  transition: all 0.3s ease;
}

.game-checkbox input[type="checkbox"]:checked + label {
  background: var(--primary);
  border-color: var(--primary);
}

.game-checkbox label::after {
  content: '';
  position: absolute;
  top: 50%;
  left: 50%;
  transform: translate(-50%, -50%) scale(0);
  width: 10px;
  height: 10px;
  background: url("data:image/svg+xml,%3Csvg xmlns='http://www.w3.org/2000/svg' viewBox='0 0 24 24' fill='none' stroke='white' stroke-width='3' stroke-linecap='round' stroke-linejoin='round'%3E%3Cpolyline points='20 6 9 17 4 12'%3E%3C/polyline%3E%3C/svg%3E") no-repeat center;
  transition: all 0.2s ease;
}

.game-checkbox input[type="checkbox"]:checked + label::after {
  transform: translate(-50%, -50%) scale(1);
}

.game-info {
  flex: 1;
  display: flex;
  justify-content: space-between;
  align-items: center;
  gap: 1rem;
}

.name {
  font-weight: 500;
  overflow: hidden;
  text-overflow: ellipsis;
  white-space: nowrap;
}

.price {
  font-weight: 600;
  color: var(--success);
  white-space: nowrap;
}

.completed .name {
  text-decoration: line-through;
  color: var(--gray);
  opacity: 0.7;
}

.no-games {
  text-align: center;
  color: var(--gray);
  padding: 1.5rem;
  font-style: italic;
}

/* Scrollbar */
::-webkit-scrollbar {
  width: 8px;
}

::-webkit-scrollbar-track {
  background: rgba(255, 255, 255, 0.05);
  border-radius: 4px;
}

::-webkit-scrollbar-thumb {
  background: var(--primary);
  border-radius: 4px;
}

::-webkit-scrollbar-thumb:hover {
  background: var(--primary-dark);
}

/* Responsive */
@media (max-width: 768px) {
  form {
    grid-template-columns: 1fr;
  }
  
  body {
    padding: 1rem;
  }
  
  .container {
    padding: 1.5rem;
  }
}

@media (max-width: 480px) {
  .game-item {
    flex-wrap: wrap;
  }
  
  .game-info {
    flex-basis: 100%;
    margin-left: 32px;
  }
}
</style>
