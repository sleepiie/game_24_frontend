<template>
  <div class="container mx-auto p-4">
    <h1 class="text-3xl font-bold mb-4">Welcome to Game 24</h1>
    <div v-if="!showJoinForm">
      <button @click="createRoom">Host Game</button>
      <button @click="showJoinForm = true">Join Game</button>
    </div>
    <div v-if="showJoinForm" class="mt-4">
      <input v-model="playerName" placeholder="Enter your name" />
      <input v-model="roomId" placeholder="Enter room ID" />
      <button @click="joinRoom">Join Room</button>
      <div v-if="errorMessage" class="mt-4 text-red-500">
        {{ errorMessage }}
      </div>
    </div>
    
  </div>
</template>

<script setup>
import { ref } from 'vue';
import { useRouter } from 'vue-router';
import { io } from 'socket.io-client';

const router = useRouter();
const socket = io('http://localhost:3001');
const showJoinForm = ref(false);
const playerName = ref('');
const roomId = ref('');
const errorMessage = ref('');

socket.on('joinRoomError', (error) => {
      errorMessage.value = error;
});

const createRoom = () => {
  socket.emit('createRoom');
  socket.on('roomCreated', (roomId) => {
    router.push(`/Host-${roomId}`);
  });
};

const joinRoom = () => {
  if (playerName.value && roomId.value) {
    socket.emit('joinRoom', { roomId: roomId.value, playerName: playerName.value });
    
    socket.on('updatePlayers', () => {
      router.push(`/lobby-${roomId.value}-${playerName.value}`);
    });
  }
};

</script>