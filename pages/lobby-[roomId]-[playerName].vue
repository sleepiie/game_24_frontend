<template>
    <div class="container mx-auto p-4">
      <h1 class="text-3xl font-bold mb-4">Game Lobby</h1>
      <div class="text-xl mb-4">Room ID: {{ roomId }}</div>
      
      <div class="mb-4">
        <h2 class="text-2xl font-bold">Players</h2>
        <ul>
          <li v-for="(player, name) in players" :key="name" :class="{ 'text-yellow-500': name === playerName, 'text-gray-700': name !== playerName }">
            {{ name }}
          </li>
        </ul>
        <button @click="GotoGame">Go to game page</button>
      </div>
      
    </div>
  </template>
  
  <script setup>
  import { ref, onMounted, onUnmounted } from 'vue';
  import { useRoute, useRouter } from 'vue-router';
  import { io } from 'socket.io-client';
  
  const route = useRoute();
  const router = useRouter();
  const roomId = ref(route.params.roomId);
  const playerName = ref(route.params.playerName);
  const players = ref({});
  const socket = io('http://localhost:3001');
  
  onMounted(() => {
    socket.emit('joinRoom', { roomId: roomId.value, playerName: playerName.value });
  
    socket.on('updatePlayers', (updatedPlayers) => {
      players.value = updatedPlayers;
    });

    socket.on('gameStarted',() => {
      router.push(`/game-${roomId.value}-${playerName.value}`);
    });
  });
  
  onUnmounted(() => {
    socket.emit('leaveRoom', { roomId: roomId.value, playerName: playerName.value });
    socket.disconnect();
  });

  </script>