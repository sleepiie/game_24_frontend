<template>
  <div class="host-room-container">
    <div class="room-id-section">
      <h1>Room ID: {{ roomId }}</h1>
    </div>
    <h2 class="leader">Leaderboard</h2>
    <div class="leaderboard-section">
      <div class="player-list" v-if="Object.keys(sortedPlayers).length">
        <div
          v-for="(player, name, index) in sortedPlayers"
          :key="name"
          class="player-card"
        >
          <span class="player-rank">{{ '#' + (index + 1) }}</span>
          <span class="player-name">{{ name }}</span>
          <span class="player-score">{{ player.score }}</span>
        </div>
      </div>
      <div class="no-players" v-else>No players yet...</div>
    </div>
    <div class="action-section">
      <button
        v-if="!isStarted && !isGameEnd"
        @click="startGame"
        class="start-button"
      >
        Start Game
      </button>
      <div v-if="isStarted && !isGameEnd" class="timer-bar">
        <div
          class="progress"
          :style="{ width: `${(timeLeft / maxTime) * 100}%` }"
        ></div>
        <p class="timer-text">Time Left: {{ timeLeft }} seconds</p>
      </div>
      <h4 v-if="!isStarted && isGameEnd" class="end-message">Game is Ended</h4>
    </div>
  </div>
</template>

<script setup>
import { ref, onMounted, onUnmounted } from 'vue';
import { useRoute } from 'vue-router';
import { io } from 'socket.io-client';

const route = useRoute();
const roomId = ref(route.params.roomId);
const players = ref({});
const socket = io('http://localhost:3001');
const isStarted = ref(false);
const timeLeft = ref(9999); // Adjust based on your game settings
const isGameEnd = ref(false);
const maxTime = 300; // Adjust to match the max time for your game

onMounted(() => {
  socket.emit('hostRoom', { roomId: roomId.value });

  socket.on('updatePlayers', (updatedPlayers) => {
    players.value = updatedPlayers;
  });

  socket.on('updatescorePlayers', (updatedPlayers) => {
    players.value = updatedPlayers;
  });

  socket.on('updateTimeLeft', (time) => {
    timeLeft.value = time;
  });

  socket.on('gameEnded', () => {
    isStarted.value = false;
    isGameEnd.value = true;
    alert('Time is up! Game Over.');
  });
});

onUnmounted(() => {
  socket.disconnect();
});

const startGame = () => {
  isStarted.value = true;
  socket.emit('startGame', { roomId: roomId.value });
};

const sortedPlayers = computed(() => {
  return Object.entries(players.value)
    .sort(([, a], [, b]) => b.score - a.score) // เรียงจากมากไปน้อย
    .reduce((acc, [name, player]) => {
      acc[name] = player;
      return acc;
    }, {});
});
</script>

<style scoped>
.host-room-container {
  display: flex;
  flex-direction: column;
  align-items: center;
  padding: 20px;
  background-color: black;
  color: white;
  min-height: 100vh;
  box-sizing: border-box;
}

.room-id-section {
  width: 100%;
  background-color: #444;
  padding: 15px 0;
  border-radius: 10px 10px 10px 10px;
  text-align: center;
  margin-bottom: 20px;
}

.room-id-section h1 {
  margin: 0;
  font-size: 24px;
  color: #fff;
}

.leader {
  width: 100%;
  text-align: left; /* จัดตำแหน่งซ้าย */
  font-size: 30px; /* ขยายขนาดตัวอักษร */
  margin: 10px 0; /* เพิ่มช่องว่าง */
}

.leaderboard-section {
  width: 100%;
  max-height: 300px;
  overflow-y: auto;
  background-color: #333;
  border-radius: 10px;
  padding: 15px;
  margin-bottom: 20px;
  box-sizing: border-box;
}

.player-list {
  display: flex;
  flex-direction: column;
  gap: 10px;
}

.player-card {
  display: flex;
  justify-content: space-between;
  align-items: center;
  padding: 10px;
  background-color: #444;
  border-radius: 8px;
  color: #fff;
}

.player-rank {
  font-size: 20px;
  font-weight: bold;
}

.player-name {
  flex: 1;
  font-size: 25px;
  text-align: left;
  margin-left: 10px;
}

.player-score {
  font-size: 25px;
  font-weight: bold;
}

.action-section {
  display: flex;
  flex-direction: column;
  align-items: center;
  margin-top: 20px;
  width: 100%;
}

.start-button {
  padding: 10px 20px;
  background-color: #28a745;
  color: white;
  border: none;
  border-radius: 10px;
  cursor: pointer;
  transition: background-color 0.3s;
  width: 80%;
  font-size: 18px;
}

.start-button:hover {
  background-color: #218838;
}

.timer-bar {
  width: 80%;
  height: 25px;
  background-color: #555;
  border-radius: 10px;
  position: relative;
  overflow: hidden;
  margin-bottom: 20px;
  box-sizing: border-box;
}

.progress {
  height: 100%;
  background-color: #28a745;
  transition: width 0.5s;
}

.timer-text {
  position: absolute;
  top: -12px;
  left: 50%;
  transform: translateX(-50%);
  font-size: 16px; /* ขยายขนาดตัวอักษร */
  color: #fff;
  pointer-events: none;
}

.end-message {
  font-size: 18px;
  font-weight: bold;
  color: #dc3545;
}
</style>
