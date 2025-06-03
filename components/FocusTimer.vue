<script setup lang="ts">
import { ref, computed, watch, onMounted, onUnmounted } from "vue";
import { Icon } from "@iconify/vue";

const focusLevel = ref(0);

// Timer duration in seconds (25 minutes)
const totalSeconds = ref(25 * 60);
const remainingSeconds = ref(totalSeconds.value);
const isRunning = ref(false);
let timer: number | null = null;

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
};

const completeCycle = () => {
  pauseTimer();
  focusLevel.value = (focusLevel.value % 4) + 1;
  remainingSeconds.value = totalSeconds.value;
};

// Clean up
onUnmounted(() => {
  if (timer) clearInterval(timer);
});
</script>

<template>
  <div class="flex items-center justify-center text-white bg-[#010101]">
    <div
      class="w-80 h-80 flex flex-col items-center justify-between p-4 relative shadow-xl"
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
          <circle
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
          />
        </svg>
        <div class="absolute inset-0 flex flex-col items-center justify-center">
          <Icon icon="solar:eye-outline" class="w-5 h-5 mb-2 text-gray-300" />
          <span class="text-3xl font-bold">{{ time }}</span>
          <div class="mt-1 flex space-x-1">
            <span
              v-for="i in 4"
              :key="i"
              class="w-2 h-1 rounded-full"
              :class="i <= focusLevel ? 'bg-red-500' : 'bg-gray-600'"
            />
          </div>
          <span class="text-xs mt-1 tracking-widest text-gray-400">FOCUS</span>
        </div>
      </div>

      <!-- Controls -->
      <div class="w-full flex justify-between items-center px-4 mb-2">
        <button
          type="button"
          class="text-white text-xl cursor-pointer"
          @click="resetTimer"
        >
          <Icon icon="solar:refresh-outline" class="w-6 h-6" />
        </button>
        <button
          type="button"
          class="bg-gray-700 px-6 py-2 rounded-full text-white font-semibold tracking-wide"
          @click="startPause"
        >
          {{ isRunning ? "PAUSE" : "START" }}
        </button>
        <button type="button" class="text-white text-xl">
          <Icon icon="solar:settings-outline" class="w-6 h-6" />
        </button>
      </div>
    </div>
  </div>
</template>
