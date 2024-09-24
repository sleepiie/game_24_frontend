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
        <button @click="startGame" v-if="!isStarted">Start Game</button>
        <h4 v-if="isStarted">Game is Started</h4>
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
  
  
  onMounted(() => {
    socket.emit('hostRoom', { roomId: roomId.value });
  
    socket.on('updatePlayers', (updatedPlayers) => {
      players.value = updatedPlayers;
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