<template>
  <div class="lobby-container">
    <h1 class="room-id">Lobby : {{ roomId }}</h1>

    <div class="players-section">
      <h2 class="players-title">Players</h2>
        <div class="players-list">
            <div
              v-for="(player, name) in players"
              :key="name"
              class="player-card"
              :class="{ 'highlight': name === playerName }"
            >
              <span class="player-name">{{ name }}</span>
            </div>
        </div>
    </div>

    <div class="waiting-section">
      <span class="waiting-text">Waiting for host to start the game</span>
      <svg class="spinner" viewBox="0 0 50 50">
        <circle
          class="spinner-path"
          cx="25"
          cy="25"
          r="20"
          fill="none"
          stroke-width="4"
        ></circle>
      </svg>
    </div>
  </div>
</template>

<script setup>
import { ref, onMounted, onUnmounted } from 'vue';
import { useRoute, useRouter} from 'vue-router';
import { io } from 'socket.io-client';

const route = useRoute();
const router = useRouter();
const roomId = ref(route.params.roomId);
const playerName = ref(route.params.playerName);
const players = ref({});
const socket = io('https://game-24-backend.onrender.com');
let isJoiningGame = false;


onMounted(() => {
  socket.emit('hostRoom', { roomId: roomId.value, playerName: playerName.value });

  socket.on('updatePlayers', (updatedPlayers) => {
    players.value = updatedPlayers;
  });

  socket.on('gameStarted', () => {
    isJoiningGame = true;
    router.push(`/game-${roomId.value}-${playerName.value}`);
  });
  socket.on('roomEnded', () => {
    router.push(`/`);
  });

  socket.on('leftRoom', () => {
    router.push('/');
  });

});

onBeforeUnmount(() => {
  if (!isJoiningGame) {
    leaveRoom();
  }
});


onUnmounted(() => {
  socket.off('updatePlayers');
  socket.off('gameStarted');
  socket.off('roomEnded');
  socket.off('leftRoom');
});

const leaveRoom = () => {
  socket.emit('leaveRoom', { roomId: roomId.value, playerName: playerName.value });
};

if (typeof window !== 'undefined') {
  window.addEventListener('beforeunload', (event) => {
    if (!isJoiningGame) {
      leaveRoom();
      event.preventDefault();
      event.returnValue = '';
    }
  });
}

</script>


<style scoped>
.lobby-container {
  display: flex;
  flex-direction: column;
  align-items: center;
  margin: -20px;
  padding: 30px;
  background-color: #121212;
  color: #ffffff;
  min-height: 100vh;
}

.room-id {
  font-size: 24px;
  font-weight: bold;
  margin-bottom: 20px;
}

.players-section {
  width: 100%;
  max-width: 400px;
  margin-bottom: 20px;
}

.players-title {
  font-size: 20px;
  font-weight: bold;
  margin: 0;
  text-align: center;
  padding-top: 5px;
  margin-bottom: 20px;
}

.players-list {
  display: flex;
  flex-direction: column;
  gap: 10px;
  max-height: 500px;
  overflow-y: auto;
  border-radius: 10px;
  margin-bottom: 10px;
}

.scrollable-list {
  display: flex;
  flex-direction: column;
  gap: 8px;
}
.waiting-section {
  display: flex;
  align-items: center;
  justify-content: center;
  margin-top: 30px;
}

.waiting-text {
  font-size: 16px;
  font-weight: 500;
  margin-right: 10px;
}

.spinner {
  height: 24px;
  width: 24px;
  animation: spin 1s linear infinite;
}

.spinner-path {
  stroke: #ffffff;
  stroke-linecap: round;
  stroke-dasharray: 10;
  stroke-dashoffset: 0;
  animation: dash 3s ease-in-out infinite, spin 1s linear infinite;
}

@keyframes spin {
  0% {
    transform: rotate(0deg);
  }
  100% {
    transform: rotate(0deg);
  }

}
@keyframes dash {
  0% {
    stroke-dashoffset:0;
    transform: rotate(0deg);
  }
  50% {
    stroke-dashoffset: 90;
    transform: rotate(180deg);
  }
  100% {
    stroke-dashoffset: 180;
    transform: rotate(360deg);
  }
}

.players-section{
  width: 100%;
  max-height: 300px;
  overflow-y: auto;
  background-color: #333;
  border-radius: 10px;
  padding: 15px;
  margin-bottom: 20px;
  box-sizing: border-box;
}
.player-card{
  display: flex;
  justify-content: space-between;
  align-items: center;
  padding: 10px;
  background-color: #444;
  border-radius: 8px;
  color: #fff;
}
.player-card.highlight {
  background-color: #28a745;
}
</style>
