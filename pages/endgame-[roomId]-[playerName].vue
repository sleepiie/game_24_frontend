<template>
    <div>
        <h1>Game is Ended</h1>
        <h2>Leaderboard</h2>
        <div>
            <p>Room ID: {{ roomId }}</p>
            <p>Player Name: {{ playerName }} </p>
        </div>
        <ul>
            <li v-for="(player, name) in players" :key="name">
                {{ name }}: {{ player.score }}
            </li>
        </ul>
        <div>
            <p v-if="isStarted">Time Left: {{ timeLeft }} seconds</p>
            <p v-if="!isStarted">Game is Ended</p>
        </div>
    </div>

</template>
<script setup>
  import { ref, onMounted, onUnmounted } from 'vue';
  import { useRoute} from 'vue-router';
  import { io } from 'socket.io-client';
  
  const route = useRoute();

  const roomId = ref(route.params.roomId);
  const players = ref({});
  const playerName =ref(route.params.playerName);
  const socket = io('http://localhost:3001');
  const isStarted = ref(false);
  const timeLeft = ref(999);
  
  if (timeLeft.value > 0) {
    isStarted.value = true;
  }
  else {
    isStarted.value = false;
  } 

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
    socket.on('gameEnded', () => {
      isStarted.value = false;
    });

  });
  
  onUnmounted(() => {
    socket.disconnect();
  });
</script>