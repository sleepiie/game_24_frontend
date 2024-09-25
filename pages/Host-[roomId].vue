<template>
    <div>
      <h1>Host Room</h1>
      <div>Room ID: {{ roomId }}</div>
      <div >
        <h2>Leaderboard</h2>
        <ul>
          <li v-for="(player, name) in players" :key="name">
            {{ name }}: {{ player.score }}
          </li>
        </ul>
        <button @click="startGame" v-if="!isStarted && !isGameEnd">Start Game</button>
        <h4 v-if="isStarted && !isGameEnd">Game is Started</h4>
        <h4 v-if="isStarted && !isGameEnd">Time Left: {{ timeLeft }} seconds</h4>
        <h4 v-if="!isStarted && isGameEnd">Game is Ended</h4>
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
  const timeLeft = ref(0);
  const isGameEnd = ref(false);
  
  
  onMounted(() => {
    socket.emit('hostRoom', { roomId: roomId.value });
  
    socket.on('updatePlayers', (updatedPlayers) => {
      players.value = updatedPlayers;
    });
    
    socket.on('updatescorePlayers', (updatedPlayers) => {
      players.value = updatedPlayers;
    });


    socket.on('updateTimeLeft' , (time) => {
      timeLeft.value = time;
    });

    socket.on('gameEnded' , () => {
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
</script>