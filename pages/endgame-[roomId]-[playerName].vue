<template>
  <div class="player-end-container">
    <div class="room-id-section">
      <h1>Match Results</h1>
      <p>Room ID: {{ roomId }}</p>
    </div>

    <h2 class="leader">Leaderboard</h2>
    <div class="leaderboard-section">
      <div class="player-list" v-if="Object.keys(sortedPlayers).length">
        <div
          v-for="(player, name, index) in sortedPlayers"
          :key="name"
          class="player-card"
          :class="{ 'highlight': name === playerName }"
        >
          <span class="player-rank">
            {{ getRankEmoji(index) }} {{ '#' + (index + 1) }}
          </span>
          <span class="player-name">{{ name }}</span>
          <span class="player-score">{{ player.score }}</span>
        </div>
      </div>
      <div class="no-players" v-else>No players yet...</div>
    </div>

    <div v-if="isStarted && !isGameEnd" class="timer-bar">
        <div
          class="progress"
          :style="{ width: `${(timeLeft / 600) * 100}%` }"
        ></div>
        <p class="timer-text">Time Left: {{ timeLeft }} seconds</p>
    </div>
      <h4 v-if="isGameEnd && !isStarted" class="end-message">Game is Ended</h4>
  </div>
</template>

<script setup>
import { ref, onMounted, onUnmounted, computed } from 'vue';
import { useRoute , useRouter} from 'vue-router';
import { io } from 'socket.io-client';

const route = useRoute();
const router = useRouter();
const roomId = ref(route.params.roomId);
const players = ref({});
const playerName = ref(route.params.playerName);
const socket = io('https://game-24-backend.onrender.com');
const isStarted = ref(false);
const isGameEnd = ref(false);
const timeLeft = ref(99);

if (timeLeft.value > 0) {
  isStarted.value = true;
}
else {
  isStarted.value = false;
  isGameEnd.value = true;
}

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
  });

  socket.on('roomEnded', () => {
    router.push(`/`);
  });
});

onUnmounted(() => {
  socket.disconnect();
});

const sortedPlayers = computed(() => {
  return Object.entries(players.value)
    .sort(([, a], [, b]) => b.score - a.score)
    .reduce((acc, [name, player]) => {
      acc[name] = player;
      return acc;
    }, {});
});

const getRankEmoji = (index) => {
  if (index === 0) {
    return '🥇'; // ถ้วยทอง
  } else if (index === 1) {
    return '🥈'; // ถ้วยเงิน
  } else if (index === 2) {
    return '🥉'; // ถ้วยทองแดง
  }
  return ''; // ถ้าไม่ใช่อันดับ 1-3 จะไม่มีอีโมจิ
};

</script>

<style scoped>
.player-end-container {

  display: flex;
  flex-direction: column;
  align-items: center;
  margin: -10px;
  padding: 20px;
  background-color: black;
  color: white;
  height: 100vh;
}

.room-id-section {
  width: 100%;
  background-color: #444;
  padding: 0;
  padding-top: 15px;
  border-radius: 10px;
  text-align: center;
  margin-bottom: 15px;
}

.room-id-section h1 {
  margin: 0;
  font-size: 32px;
  color: #fff;
}

.leader {
  width: 100%;
  text-align: left;
  font-size: 30px;
  margin: 10px 0;
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

.game-status {
  margin-top: 20px;
}

.game-status p {
  font-size: 20px;
  font-weight: bold;
}
.player-card.highlight {
  background-color: #28a745;
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

