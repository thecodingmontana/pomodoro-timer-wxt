<script setup lang="ts">
import { ref, computed, watch, onMounted, onUnmounted } from "vue";
import { Icon } from "@iconify/vue";
import { motion } from "motion-v";

const focusLevel = ref(0);

// Timer duration in seconds (25 minutes)
const totalSeconds = ref(25 * 60);
const remainingSeconds = ref(totalSeconds.value);
const isRunning = ref(false);
const justCompleted = ref(false);
let timer: ReturnType<typeof setInterval> | null = null;

// Format time to mm:ss
const time = computed(() => {
  const mins = Math.floor(remainingSeconds.value / 60)
    .toString()
    .padStart(2, "0");
  const secs = (remainingSeconds.value % 60).toString().padStart(2, "0");
  return `${mins}:${secs}`;
});

// SVG progress
const radius = 90;
const circumference = 2 * Math.PI * radius;

const progress = computed(() => {
  return 1 - remainingSeconds.value / totalSeconds.value;
});
const progressOffset = computed(() => circumference * (1 - progress.value));

// Animation states
const pulseTimer = ref(false);
const shakeReset = ref(false);

// Watch for timer changes to trigger animations
watch(remainingSeconds, (newVal, oldVal) => {
  if (newVal < oldVal && isRunning.value) {
    // Pulse animation every second
    pulseTimer.value = true;
    setTimeout(() => {
      pulseTimer.value = false;
    }, 200);
  }
});

// Controls
const startPause = () => {
  if (isRunning.value) {
    pauseTimer();
  } else {
    startTimer();
  }
};

const startTimer = () => {
  if (timer) return;
  isRunning.value = true;
  justCompleted.value = false;
  timer = setInterval(() => {
    if (remainingSeconds.value > 0) {
      remainingSeconds.value--;
    } else {
      completeCycle();
    }
  }, 1000);
};

const pauseTimer = () => {
  isRunning.value = false;
  if (timer) {
    clearInterval(timer);
    timer = null;
  }
};

const resetTimer = () => {
  pauseTimer();
  remainingSeconds.value = totalSeconds.value;
  justCompleted.value = false;
  
  // Trigger shake animation
  shakeReset.value = true;
  setTimeout(() => {
    shakeReset.value = false;
  }, 500);
};

const completeCycle = () => {
  pauseTimer();
  focusLevel.value = (focusLevel.value % 4) + 1;
  remainingSeconds.value = totalSeconds.value;
  justCompleted.value = true;
  
  // Reset completion state after animation
  setTimeout(() => {
    justCompleted.value = false;
  }, 1000);
};

// Clean up
onUnmounted(() => {
  if (timer) clearInterval(timer);
});
</script>

<template>
  <div class="flex items-center justify-center text-white bg-[#010101] min-h-screen">
    <motion.div
      class="w-80 h-80 flex flex-col items-center justify-between p-4 relative shadow-xl"
      :animate="{
        scale: justCompleted ? [1, 1.05, 1] : 1,
        rotate: shakeReset ? [-2, 2, -2, 2, 0] : 0
      }"
      :transition="{ 
        duration: justCompleted ? 0.6 : 0.5,
        ease: 'easeInOut'
      }"
    >
      <!-- Progress Circle -->
      <div class="relative w-48 h-48 my-4">
        <svg class="w-full h-full transform -rotate-90">
          <circle
            class="text-gray-700"
            stroke-width="10"
            stroke="currentColor"
            fill="transparent"
            r="90"
            cx="96"
            cy="96"
          />
          <motion.circle
            class="text-red-500 transition-all duration-300 rounded-full"
            :stroke-dasharray="circumference"
            :stroke-dashoffset="progressOffset"
            stroke-width="10"
            stroke="currentColor"
            fill="transparent"
            r="90"
            cx="96"
            cy="96"
            stroke-linecap="round"
            :animate="{
              strokeWidth: justCompleted ? [10, 15, 10] : 10,
              opacity: justCompleted ? [1, 0.7, 1] : 1
            }"
            :transition="{ duration: 0.6, ease: 'easeInOut' }"
          />
        </svg>
        
        <!-- Timer Display -->
        <motion.div 
          class="absolute inset-0 flex flex-col items-center justify-center"
          :animate="{
            scale: pulseTimer ? [1, 0.95, 1] : 1,
            y: justCompleted ? [0, -5, 0] : 0
          }"
          :transition="{ 
            duration: pulseTimer ? 0.2 : 0.6,
            ease: 'easeInOut'
          }"
        >
          <motion.div
            :animate="{
              scale: isRunning ? [1, 1.1, 1] : 1,
              rotate: justCompleted ? [0, 360] : 0
            }"
            :transition="{ 
              duration: isRunning ? 2 : 0.8,
              repeat: isRunning ? Infinity : 0,
              ease: 'easeInOut'
            }"
          >
            <Icon icon="solar:eye-outline" class="w-5 h-5 mb-2 text-gray-300" />
          </motion.div>
          
          <motion.span 
            class="text-3xl font-bold"
            :animate="{
              color: justCompleted ? ['#ffffff', '#ef4444', '#ffffff'] : '#ffffff',
              scale: remainingSeconds <= 10 && isRunning ? [1, 1.1, 1] : 1
            }"
            :transition="{ 
              duration: remainingSeconds <= 10 && isRunning ? 0.5 : 0.6,
              repeat: remainingSeconds <= 10 && isRunning ? Infinity : 0,
              ease: 'easeInOut'
            }"
          >
            {{ time }}
          </motion.span>
          
          <!-- Focus Level Dots -->
          <div class="mt-1 flex space-x-1">
            <motion.span
              v-for="i in 4"
              :key="i"
              class="w-2 h-1 rounded-full"
              :class="i <= focusLevel ? 'bg-red-500' : 'bg-gray-600'"
              :animate="{
                scale: i === focusLevel && justCompleted ? [1, 1.5, 1] : 1,
                opacity: i === focusLevel && justCompleted ? [1, 0.5, 1] : 1
              }"
              :transition="{ 
                duration: 0.6,
                delay: i * 0.1,
                ease: 'easeInOut'
              }"
            />
          </div>
          
          <motion.span 
            class="text-xs mt-1 tracking-widest text-gray-400"
            :animate="{
              opacity: justCompleted ? [1, 0.5, 1] : 1,
              letterSpacing: justCompleted ? ['0.1em', '0.2em', '0.1em'] : '0.1em'
            }"
            :transition="{ duration: 0.6, ease: 'easeInOut' }"
          >
            FOCUS
          </motion.span>
        </motion.div>
      </div>

      <!-- Controls -->
      <div class="w-full flex justify-between items-center px-4 mb-2">
        <motion.button
          type="button"
          class="text-white text-xl cursor-pointer"
          @click="resetTimer"
          :whileHover="{ scale: 1.1, rotate: 180 }"
          :whileTap="{ scale: 0.9 }"
          :transition="{ duration: 0.2 }"
        >
          <Icon icon="solar:refresh-outline" class="w-6 h-6" />
        </motion.button>
        
        <motion.button
          type="button"
          class="bg-gray-700 px-6 py-2 rounded-full text-white font-semibold tracking-wide"
          @click="startPause"
          :whileHover="{ 
            scale: 1.05,
            backgroundColor: isRunning ? '#374151' : '#1f2937'
          }"
          :whileTap="{ scale: 0.95 }"
          :animate="{
            backgroundColor: isRunning ? '#ef4444' : '#374151'
          }"
          :transition="{ duration: 0.3 }"
        >
          {{ isRunning ? "PAUSE" : "START" }}
        </motion.button>
        
        <motion.button 
          type="button" 
          class="text-white text-xl"
          :whileHover="{ scale: 1.1, rotate: 90 }"
          :whileTap="{ scale: 0.9 }"
          :transition="{ duration: 0.2 }"
        >
          <Icon icon="solar:settings-outline" class="w-6 h-6" />
        </motion.button>
      </div>
    </motion.div>
  </div>
</template>