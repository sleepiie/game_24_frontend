<template>
    <div>
      <h1>Game Room</h1>
      <div>Room ID: {{ roomId }}</div>
      <div>Player Name: {{ playerName }}</div>
      <div v-if="currentNumbers.length">
        <p>Set Index: {{ currentSetIndex }}</p>
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
        <button class="operator-button" @click="selectOperator('*')" :class="{ active: selectedOperator === '*' }">*</button>
        <button class="operator-button" @click="selectOperator('/')" :class="{ active: selectedOperator === '/' }">/</button>
      </div>
  
      <button @click="GotoEnd">Go to End game</button>
      <div>
        <p>Time Left: {{ timeLeft }} seconds</p>
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
  const currentNumbers = ref([]);
  const currentSetIndex = ref(1);
  const firstNumber = ref(null); // เก็บค่าตัวเลขตัวแรก
  const secondNumber = ref(null); // เก็บค่าตัวเลขตัวที่สอง
  const operator = ref(null); // เก็บค่าของเครื่องหมาย
  const socket = io('http://localhost:3001');

  const selectedOperator = ref(null);
  const selectedNumber = ref(null);
  const resultIs24 =ref(false);
  const timeLeft = ref(0);
  
  onMounted(() => {
    socket.emit('joinGame', { roomId: roomId.value, playerName: playerName.value });
  
    socket.on('updateRoomData', (data) => {
      currentNumbers.value = data.currentNumbers;
      currentSetIndex.value = data.currentSetIndex;
    });
  
    socket.on('updateCurrentSet', (data) => {
      currentNumbers.value = data.currentNumbers;
      currentSetIndex.value = data.currentSetIndex;
    });

    socket.on('updateTimeLeft' , (time) => {
      timeLeft.value = time;
    });

    socket.on('gameEnded' , () => {
      router.push(`/endgame-${roomId.value}-${playerName.value}`);

    });

  });
  
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
        break;
    }

    // อัปเดตตัวเลขในปุ่มที่สองด้วยผลลัพธ์
    currentNumbers.value[secondNumber.value] = result;
    currentNumbers.value[firstNumber.value] = null;

    console.log('Result:', result);

    const remainingNumbers = currentNumbers.value.filter(number => number !== null);
    const isOnly24button = remainingNumbers.length === 1 && remainingNumbers[0] === 24;

    console.log('isOnly24button:', isOnly24button);

    if (result === 24) {
        if (isOnly24button) {
            // ถ้าเหลือเพียงปุ่มเดียว แสดงว่าคำตอบถูกต้อง
            resultIs24.value = true;
            setTimeout(() => {
                nextNumberSet();
            }, 2000);
        }
    }
    // รีเซ็ตค่าหลังจากคำนวณเสร็จ
    
    firstNumber.value = null;
    secondNumber.value = null;
    operator.value = null;

    selectedNumber.value = null;
    selectedOperator.value = null;
  };
  
  const GotoEnd = () => {
    router.push(`/endgame-${roomId.value}-${playerName.value}`);
  };

  const nextNumberSet = () => {
    resultIs24.value = false;
    socket.emit('nextSet', { roomId: roomId.value, playerName: playerName.value });
  };

</script>
  

<style>
  .number-grid {
    display: grid;
    grid-template-columns: repeat(2, 100px);
    grid-gap: 10px;
    justify-content: center;
    margin: 20px 0;
  }
  
  .operator-grid {
    display: grid;
    grid-template-columns: repeat(4, 52px);
    grid-gap: 1px;
    justify-content: center;
    margin: 20px 20px;
  }
  
  .number-button {
    background-color: #D9534F;
    color: white;
    font-size: 2rem;
    padding: 20px;
    border: none;
    border-radius: 5px;
    width: 100px;
    height: 100px;
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
    width: 45px;
    height: 45px;
    display: flex;
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
  width: 100px;
  height: 100px;
}
</style>
  