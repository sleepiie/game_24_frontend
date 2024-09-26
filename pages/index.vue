<template>
  <div class="container">
    <h1 class="title">Game 24</h1>

    <div v-if="!showJoinForm" class="button-container">
      <button @click="showJoinForm = true" class="btn">Join</button>
      <button @click="createRoom" class="btn">Host</button>
    </div>

    <div v-if="showJoinForm" class="join-form">
      <input v-model="playerName" placeholder="Enter your name" class="input" />
      <input v-model="roomId" placeholder="Enter room ID" class="input" />
      <button @click="joinRoom" class="btn-join" >Join Game</button>
      <div v-if="errorMessage" class="error-message">
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

<style scoped>
html, body {
  margin: 0;
  padding: 0;
  height: 100%;
  width: 100%;
  background-color: black;
}
.container {
  display: flex;
  flex-direction: column;
  justify-content: center;
  align-items: center;
  width: 100vw;
  height: 100vh;
  background-color: black;
  margin: 0;
  padding: 0;
  color: white;
}

.title {
  font-size: 5rem;
  font-weight: bold;
  margin-bottom: 3rem;
}

.button-container {
  display: flex;
  flex-direction: column;
  gap: 1rem;
}

.btn {
  background-color: #4caf50;
  color: white;
  font-size: 2rem;
  padding: 1rem 3rem;
  border-radius: 1rem;
  border: none;
  cursor: pointer;
  transition: background-color 0.2s ease-in-out;
}

.btn:hover {
  background-color: #388e3c;
}

.join-form {
  display: flex;
  flex-direction: column;
  gap: 1.5rem;
  width: 100%;
  max-width: 400px;
  margin-top: 2rem;
}

.input {
  padding: 1rem;
  font-size: 1.2rem;
  border-radius: 0.5rem;
  border: 1px solid #ccc;
  background-color: white;
  color: black;
}

.input:focus {
  outline: none;
  box-shadow: 0 0 0 2px #10b981;
}

.btn-join {
  background-color: #4caf50;
  color: white;
  font-size: 1.5rem;
  padding: 1rem;
  border-radius: 1rem;
  border: none;
  cursor: pointer;
  transition: background-color 0.2s ease-in-out;
}

.btn-join:hover {
  background-color: #388e3c;
}

.error-message {
  color: red;
  text-align: center;
  font-size: 1.2rem;
}
</style>