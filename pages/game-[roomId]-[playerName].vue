<template>
  <div class="game-room-container">
    <div class="room-info-section">
      <h1 class="title">Game Room : {{ roomId }}</h1>
      <div class="player-name">Player Name: {{ playerName }}</div>
    </div>

    <div v-if="currentNumbers.length" class="game-set-info">
      <p>{{ currentSetIndex }} / 10</p>
    </div>

    <!-- ปุ่มตัวเลข -->
    <div class="number-grid" v-if="currentNumbers.length">
      <button 
          v-for="(number, index) in currentNumbers" 
          :key="index"
          class="number-button"  
          :class="{ 'selected': selectedNumber === index ,
          'empty-button': currentNumbers[index] === null,
          'correct-answer': number === 24 && resultIs24 ,
          'center': number === 24 && resultIs24}" 
          @click="currentNumbers[index] != null && !resultIs24 ? selectNumber(index) : null">
          {{ currentNumbers[index] != null ? number : '' }}
      </button>
    </div>

    <!-- ปุ่มเครื่องหมายบวก, ลบ, คูณ, หาร -->
    <div class="operator-grid">
      <button class="operator-button" @click="selectOperator('+')" :class="{ active: selectedOperator === '+' }">+</button>
      <button class="operator-button" @click="selectOperator('-')" :class="{ active: selectedOperator === '-' }">-</button>
      <button class="operator-button" @click="selectOperator('*')" :class="{ active: selectedOperator === '*' }">×</button>
      <button class="operator-button" @click="selectOperator('/')" :class="{ active: selectedOperator === '/' }">÷</button>
    </div>

    <button class="reset-button" @click="reset">Reset</button>

    <!-- หลอดนับถอยหลัง -->
    <div class="timer-bar">
      <div class="progress" :style="{ width: `${(timeLeft / 420) * 100}%` }"></div>
      <p class="timer-text">Time Left: {{ timeLeft }} seconds</p>
    </div>
  </div>
</template>
  
<script setup>
  import { ref, onMounted } from 'vue';
  import { useRoute, useRouter } from 'vue-router';
  import { io } from 'socket.io-client';
  
  const router = useRouter();
  const route = useRoute();
  const roomId = ref(route.params.roomId);
  const playerName = ref(route.params.playerName);
  const intitialNumbers = ref([]);
  const currentNumbers = ref([]);
  const currentSetIndex = ref(1);
  const firstNumber = ref(null); // เก็บค่าตัวเลขตัวแรก
  const secondNumber = ref(null); // เก็บค่าตัวเลขตัวที่สอง
  const operator = ref(null); // เก็บค่าของเครื่องหมาย
  const socket = io('https://game-24-backend.onrender.com');
  

  const selectedOperator = ref(null);
  const selectedNumber = ref(null);
  const resultIs24 =ref(false);
  const timeLeft = ref(99);
  
  onMounted(() => {
    socket.emit('joinGame', { roomId: roomId.value, playerName: playerName.value });
  
    socket.on('updateRoomData', (data) => {
      intitialNumbers.value = [...data.currentNumbers];
      currentNumbers.value = data.currentNumbers;
      currentSetIndex.value = data.currentSetIndex;
    });
  
    socket.on('updateCurrentSet', (data) => {
      intitialNumbers.value = [...data.currentNumbers];
      currentNumbers.value = data.currentNumbers;
      currentSetIndex.value = data.currentSetIndex;
    });

    socket.on('updateTimeLeft' , (time) => {
      timeLeft.value = time;
    });

    socket.on('gameEnded' , () => {
      router.push(`/endgame-${roomId.value}-${playerName.value}`);
    });

    socket.on('finished' ,(data) => {
        console.log(data.playerName);
        console.log(playerName.value);    
        if (data.playerName === playerName.value) {
            router.push(`/endgame-${roomId.value}-${playerName.value}`);
        }
    });

    socket.on('roomEnded', () => {
      router.push(`/`);
    });

  });

  const reset = () => {
    currentNumbers.value = [...intitialNumbers.value]; // คืนค่าเป็นตัวเลขเริ่มต้น
    firstNumber.value = null;
    secondNumber.value = null;
    operator.value = null;
    selectedNumber.value = null;
    selectedOperator.value = null;
    resultIs24.value = false;
  };
  
  const selectNumber = (index) => {
    if (selectedNumber.value === index) {
        // ยกเลิกการเลือก
        selectedNumber.value = null;
        firstNumber.value = null;
        secondNumber.value = null;
        operator.value = null;
    } else if (firstNumber.value === null) {
        // เลือกตัวเลขตัวแรก
        firstNumber.value = index;
        selectedNumber.value = index;
    } else if (operator.value !== null) {
        // เลือกตัวเลขตัวที่สองแล้วทำการคำนวณ
        secondNumber.value = index;
        calculateResult();
    }
  };
  
  const selectOperator = (op) => {
    if (selectedOperator.value === op) {
        // ถ้ากดซ้ำที่เครื่องหมายเดิม ให้ยกเลิกการเลือก
        selectedOperator.value = null;
        operator.value = null;
    } else {
        // เลือกเครื่องหมายใหม่
        selectedOperator.value = op;
        operator.value = op;
        
    }
  };

  const calculateResult = () => {
    const num1 = currentNumbers.value[firstNumber.value];
    const num2 = currentNumbers.value[secondNumber.value];
    let result = 0;
  
    switch (operator.value) {
      case '+':
        result = num1 + num2;
        break;
      case '-':
        result = num1 - num2;
        break;
      case '*':
        result = num1 * num2;
        break;
      case '/':
        result = num1 / num2;
        result = parseFloat(result.toFixed(2));
        break;
    }

    // อัปเดตตัวเลขในปุ่มที่สองด้วยผลลัพธ์
    currentNumbers.value[secondNumber.value] = result;
    currentNumbers.value[firstNumber.value] = null;

    console.log('Result:', result);

    selectedNumber.value = secondNumber.value;
    firstNumber.value = secondNumber.value;

    const remainingNumbers = currentNumbers.value.filter(number => number !== null);
    const isOnly24button = remainingNumbers.length === 1 && remainingNumbers[0] === 24;

    console.log('isOnly24button:', isOnly24button);

    if (result === 24) {
        if (isOnly24button) {
            // ถ้าเหลือเพียงปุ่มเดียว แสดงว่าคำตอบถูกต้อง
            selectedNumber.value = null;
            firstNumber.value = null;
            resultIs24.value = true;
            setTimeout(() => {
                nextNumberSet();
            }, 2000);
        }
    }
    // รีเซ็ตค่าหลังจากคำนวณเสร็จ
    
    secondNumber.value = null;
    operator.value = null;
    selectedOperator.value = null;
  };

  const nextNumberSet = () => {
    resultIs24.value = false;
    socket.emit('nextSet', { roomId: roomId.value, playerName: playerName.value });
  };


</script>
  

<style>
  .title{
    font-size: 24px;
    color: white;
  }
  
  .number-grid {
    display: grid;
    grid-template-columns: repeat(2, 150px);
    grid-gap: 10px;
    justify-content: center;
    margin: 5px 5px;
  }
  
  .operator-grid {
    display: grid;
    grid-template-columns: repeat(4, 75px);
    grid-gap: 6px;
    justify-content: center;
    margin-top: 18px;
    margin-bottom: 25px;
    margin-left: 8px;
  }
  
  .number-button {
    background-color: #D9534F;
    color: white;
    font-size: 2rem;
    padding: 20px;
    border: none;
    border-radius: 5px;
    width: 150px;
    height: 150px;
    display: flex;
    align-items: center;
    justify-content: center;
    cursor: pointer;
    box-shadow: 0px 4px 6px rgba(0, 0, 0, 0.1);
  }
  .number-button.selected {
    background-color: #9e2521; /* สีที่เข้มขึ้นเมื่อเลือก */
  }
  
  
  .operator-button {
    background-color: #5CB85C;
    color: white;
    font-size: 1.8rem;
    padding: 15px;
    border: none;
    border-radius: 10%;
    width: 72px;
    height: 72px;
    display: flex;
    text-align: center;
    align-items: center;
    justify-content: center;
    cursor: pointer;
    box-shadow: 0px 4px 6px rgba(0, 0, 0, 0.1);
  }
  .operator-button.active {
  background-color: #378237; /* สีเข้มขึ้น */
 }

 .empty-button {
  background-color: transparent; /* พื้นหลังใส */
  border: none; /* ลบเส้นขอบ */
  box-shadow: none; /* ลบเงา */
  pointer-events: none; /* ไม่สามารถคลิกได้ */
}
.correct-answer {
  background-color: #5CB85C; /* สีเขียว */
  pointer-events: none; /* ไม่สามารถคลิกได้ */
}
  
.correct-answer.center {
  grid-column: 1 / span 8;
  grid-row: 1 / span 5;
  justify-self: center;
  align-self: center;
  width: 150px;
  height: 150px;
}
.game-room-container {
  display: flex;
  flex-direction: column;
  align-items: center;
  background: black;
  color: white;
  margin: -20px;
  padding: 30px;
  height: 100vh;
  box-shadow: 0 0 10px rgba(0, 0, 0, 0.5);
}

.room-info-section {
  background-color: #333;
  padding: 10px 10px;
  padding-top: 0;
  border-radius: 15px;
  text-align: center;
  margin: 0; /* ลบ margin ทั้งหมด */
  margin-bottom: 15px;
  width: 100%;
  box-shadow: 0 4px 10px rgba(0, 0, 0, 0.2);
}


.room-id, .player-name {
  font-size: 18px;
  color: white;
  margin: 5px 0;
}

.game-set-info {
  font-size: 24px;
  color: white;
}

.reset-button {
  padding: 10px 20px;
  background-color: #dc3545;
  color: white;
  border: none;
  border-radius: 20px;
  cursor: pointer;
  font-size: 18px;
  margin-bottom: 15px;
  transition: background-color 0.3s ease;
  box-shadow: 0 4px 8px rgba(0, 0, 0, 0.3);
}

.reset-button:hover {
  background-color: #c82333;
}

.timer-bar {
  width: 80%;
  height: 30px;
  background-color: #444;
  border-radius: 15px;
  position: relative;
  overflow: hidden;
  margin-top: 20px;
  box-shadow: inset 0 0 5px rgba(0, 0, 0, 0.5);
}

.progress {
  height: 100%;
  background-color: #28a745;
  transition: width 0.5s ease;
}

.timer-text {
  position: absolute;
  top: -6px;
  left: 50%;
  transform: translateX(-50%);
  font-size: 14px;
  color: #fff;
}
</style>
  