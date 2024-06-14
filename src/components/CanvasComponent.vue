<template>
    <div class="canvas-wrapper">
        <v-stage :config="stageConfig">
            <v-layer>
                <!-- Background -->
                <v-rect :config="backgroundRect"></v-rect>

                <!-- Timer -->
                <v-rect :config="timerRect"></v-rect>
                <v-text :config="timerTextConfig" />

                <!-- Cloud Image -->
                <v-image :config="cloudImageConfig" />

                <!-- Balance Button -->
                <v-rect :config="balanceRect" ></v-rect>
                <v-text :config="balanceLabelText"></v-text>
                <v-text :config="balanceValueText"></v-text>

                <!-- Bet Amount Button -->
                <v-rect :config="betRect" ></v-rect>
                <v-text :config="betLabelText"></v-text>
                <v-text :config="betValueText"></v-text>
                <v-rect :config="increaseButton" @mouseover="hoverButton" @mouseout="leaveButton" @click="increaseBet"></v-rect>
                <v-text :config="increaseButtonText" @mouseover="hoverText" @mouseout="leaveText" @click="increaseBet"></v-text>
                <v-rect :config="decreaseButton" @mouseover="hoverButton" @mouseout="leaveButton" @click="decreaseBet"></v-rect>
                <v-text :config="decreaseButtonText" @mouseover="hoverText" @mouseout="leaveText" @click="decreaseBet"></v-text>

                <!-- Place Bet Button -->
                <v-rect :config="placeBetRect" @mouseover="hoverPlaceBet" @mouseout="leavePlaceBet" @click="placeBet"></v-rect>
                <v-text :config="placeBetText" @mouseover="hoverText" @mouseout="leaveText" @click="placeBet" />
            </v-layer>
        </v-stage>
    </div>
</template>
  
<script setup>
import { reactive, ref, onMounted, watch } from 'vue';
import Konva from 'konva';
import clouds from '../assets/image/clouds.png';
import _ from 'lodash';

const countdown = ref(10);
const balance = ref(1350);
const betAmount = ref(10);

//结构
const stageConfig = {
    width: 400,
    height: 600,
};
const backgroundRect = {
    x: 0,
    y: 0,
    width: 400,
    height: 600,
    fill: '#f5d8cb',
};
const timerRect = {
    x: 150,
    y: 50,
    width: 100,
    height: 40,
    fill: '#fff',
    stroke: '#1d50ad',
    strokeWidth: 2,
    cornerRadius: 5,
    shadowColor: 'rgba(0,0,0,0.2)',
    shadowBlur: 6,
    shadowOffsetX: 1,
    shadowOffsetY: -2,
};
const timerTextConfig = reactive({
    x: 196,
    y: 62,
    text: 10,
    fontSize: 18,
    fontFamily: 'Arial',
    fill: '#000',
    align: 'center',
    verticalAlign: 'middle',
});
const cloudImageConfig = ref({
    x: 300,
    y: 50,
    image: null,
    width: 500,
    height: 500,
});
const balanceRect = reactive({
  x: 20,
  y: 520,
  width: 100,
  height: 40,
  fill: '#2e85d9',
  stroke: '#3fb5f8',
  strokeWidth: 5,
  cornerRadius: 10,
  shadowColor: 'rgba(0,0,0,0.7)',
  shadowBlur: 10,
  shadowOffsetX: 3,
  shadowOffsetY: 3,
});
const balanceLabelText = reactive({
  x: 44,
  y: 530,
  text: 'Balance SGD',
  fontSize: 9,
  fill: '#8ee6f8',
});
const balanceValueText = reactive({
  x: 48,
  y: 541,
  text: balance.value,
  fontSize: 12,
  fontStyle: 'bold',
  fill: '#fff',
});
const betRect = reactive({
  x: 130,
  y: 520,
  width: 140,
  height: 40,
  fill: '#2e85d9',
  stroke: '#3fb5f8',
  strokeWidth: 5,
  cornerRadius: 10,
  shadowColor: 'rgba(0,0,0,0.7)',
  shadowBlur: 10,
  shadowOffsetX: 3,
  shadowOffsetY: 3,
});
const betLabelText = reactive({
  x: 178,
  y: 530,
  text: 'BET SGD',
  fontSize: 9,
  fill: '#8ee6f8',
});
const betValueText = reactive({
  x: 191,
  y: 541,
  text: betAmount.value,
  fontSize: 12,
  fontStyle: 'bold',
  fill: '#fff',
});
const increaseButton = reactive({
  x: 230,
  y: 525,
  width: 28,
  height: 28,
  fill: '#fff',
  cornerRadius: 10,
  shadowColor: 'rgba(0,0,0,0.7)',
  shadowBlur: 5,
  shadowOffsetX: 2,
  shadowOffsetY: 2,
 
});
const increaseButtonText = reactive({
  x: 238,
  y: 530,
  text: '+',
  fontSize: 20,
  fill: 'blue',
 
});
const decreaseButton = reactive({
  x: 140,
  y: 525,
  width: 28,
  height: 28,
  fill: '#fff',
  cornerRadius: 10,
  shadowColor: 'rgba(0,0,0,0.7)',
  shadowBlur: 5,
  shadowOffsetX: 2,
  shadowOffsetY: 2,
 
});
const decreaseButtonText = reactive({
  x: 150,
  y: 530,
  text: '-',
  fontSize: 20,
  fill: 'red',
 
});
const placeBetRect = reactive({
  x: 280,
  y: 520,
  width: 100,
  height: 40,
  fill: '#61c046',
  cornerRadius: 10,
  shadowColor: 'rgba(0,0,0,0.7)',
  shadowBlur: 10,
  shadowOffsetX: 3,
  shadowOffsetY: 3,
  opacity: 1,
 
});
const placeBetText = reactive({
  x: 290,
  y: 535,
  text: 'PLACE BET',
  fontSize: 14,
  fontStyle: 'bold',
  fill: '#fff',
});
//styles
const hoverButton = (e) => {
  e.target.fill('#eee');
  e.target.draw();
  e.target.getStage().container().style.cursor = 'pointer'
};
const leaveButton = (e) => {
  e.target.fill('#fff');
  e.target.draw();
  e.target.getStage().container().style.cursor = 'default'
};
const hoverPlaceBet = (e) => {  
  e.target.fill('#3cc746');
  e.target.draw();
  if(balance.value < betAmount.value){
    e.target.getStage().container().style.cursor = 'not-allowed'
  }else{
    e.target.getStage().container().style.cursor = 'pointer'
  }
};
const leavePlaceBet = (e) => {
  e.target.fill('#61c046');
  e.target.draw();
  e.target.getStage().container().style.cursor = 'default'
};
const hoverText = (e) => {
    if(balance.value < betAmount.value || betAmount.value <= 10){
        e.target.getStage().container().style.cursor = 'not-allowed'
    }else{
        e.target.getStage().container().style.cursor = 'pointer'
    }
};
const leaveText = (e) => {
  e.target.getStage().container().style.cursor = 'default'
};

// timer
const updateTimer = () => {
    timerTextConfig.text = countdown.value;
};
const countdownTimer = () => {
    setInterval(() => {
        if (countdown.value > 0) {
            countdown.value -= 1;
            updateTimer();
        } else {
            countdown.value = 10;
            updateTimer();
        }
    }, 1000);
};
  
//image in for the middle section
const loadImage = () => {
    const imageObj = new Image();
        imageObj.src = clouds;
        imageObj.onload = () => {
            cloudImageConfig.value.image = imageObj;
            const cloudLayer = new Konva.Layer();
            const cloud = new Konva.Image(cloudImageConfig.value);
            cloudLayer.add(cloud);

            cloudAnimation(cloudLayer,cloud)    
        };
};
const cloudAnimation = (cloudLayer,cloud) => {
    const anim = new Konva.Animation((frame) => {
        const velocity = 75; // pixels per second
        const moveAmount = (velocity * frame.timeDiff) / 1000;
        cloudImageConfig.value.x -= moveAmount;
        if (cloudImageConfig.value.x + cloudImageConfig.value.width < 0) {
            cloudImageConfig.value.x = stageConfig.width;
        }
        cloud.x(cloudImageConfig.value.x);
    }, cloudLayer);
    anim.start(); 
}

onMounted(() => {
    loadImage();
    countdownTimer();
});

  // + / - amount
const increaseBet = () => {
  if (betAmount.value < 100) {
    betAmount.value += 10;
  } else if (betAmount.value >= 100 && betAmount.value < 1000) {
    betAmount.value += 100;
  }
  betAmount.value = Math.min(betAmount.value, balance.value, 1000);
  betValueText.text = betAmount.value;
};
const decreaseBet = () => {
  if (betAmount.value > 100) {
    betAmount.value -= 100;
  } else if (betAmount.value > 10 && betAmount.value <= 100) {
    betAmount.value -= 10;
  }
  betValueText.text = betAmount.value;
};

// place bet
const updateButtonStyle = () => {
  if (balance.value < betAmount.value) {
    placeBetRect.opacity = 0.5; 
  } else {
    placeBetRect.opacity = 1;
  }
};
watch([balance, betAmount], updateButtonStyle);
const placeBet = _.debounce(() => {
  if (balance.value >= betAmount.value) {
    balance.value -= betAmount.value;
    balanceValueText.text = balance.value;
  }
}, 300);

  </script>
  
  <style scoped>
  .canvas-wrapper {
    height:100%;
    display: flex;
    justify-content: center;
    
    height: 100vh;
    background-color: #fff;
  }
  </style>
  