<template>
  <div class="flex p-10 gap-20 bg-gradient-to-br from-blue-50 to-purple-50 min-h-screen">
    <!-- Left: Blind Box Grid or Result -->
    <div class="flex-1">
      <h1 class="text-4xl font-extrabold mb-10 text-gray-900 tracking-tight drop-shadow-md">Labubu Blind Box</h1>
      <div v-if="!revealedPrize" class="grid grid-cols-3 gap-4 w-fit">
        <div
            v-for="(box, index) in boxes"
            :key="index"
            class="relative w-16 h-16 bg-gradient-to-br from-white to-gray-100 rounded-2xl cursor-pointer border transition-all duration-300 ease-in-out"
            :class="{
            'border-blue-600 ring-4 ring-blue-300/50 shadow-lg': selectedIndex === index && !isCycling,
            'border-gray-300 shadow-sm': selectedIndex !== index && !isCycling,
            'animate-cycle': isCycling,
            'opacity-70': isCycling && selectedIndex !== index,
          }"
            @click="selectBox(index)"
        >
          <img
              v-if="!box.isRevealed"
              :src="backImage"
              alt="Box"
              class="object-cover rounded-2xl w-full h-full"
              @error="handleImageError"
          />
        </div>
      </div>
      <!-- Display single prize image after reveal -->
      <div v-else class="mt-20 flex justify-center">
        <img
            :src="revealedPrize.image"
            :alt="revealedPrize.name"
            class="object-contain rounded-3xl max-w-lg h-auto shadow-2xl transition-transform duration-700 hover:scale-105 animate-reveal"
            @error="handleImageError"
        />
      </div>
      <div v-if="message" class="mt-8 text-xl font-semibold text-center text-gray-900 drop-shadow-sm">
        {{ message }}
      </div>
    </div>

    <!-- Right: Blind Box Machine Interface -->
    <div class="w-[28rem] bg-gradient-to-b from-gray-100 to-gray-200 p-10 rounded-3xl shadow-2xl ">
      <h2 class="text-3xl font-bold mb-8 text-center text-gray-900 tracking-tight">盲盒机器</h2>
      <div class="text-center mb-10 bg-gray-50 py-4 rounded-xl shadow-inner">
        <p v-if="selectedIndex === null && !revealedPrize" class="text-lg text-gray-600">请从左侧选择一个盲盒</p>
        <p v-else-if="!isCycling && !revealedPrize" class="text-lg text-gray-600">选择了 #{{ selectedIndex + 1 }}，准备开启！</p>
        <p v-else-if="isCycling" class="text-lg text-blue-600 font-semibold animate-pulse">盲盒开启中...</p>
        <p v-else class="text-lg text-green-600 font-semibold">已开启！点击重试继续</p>
      </div>
      <div class="flex justify-center gap-5">
        <button
            v-if="!isCycling && selectedIndex !== null && !revealedPrize"
            class="px-6 py-3 bg-gray-300 rounded-xl hover:bg-gray-400 text-lg text-gray-800 font-medium shadow-md transition-all duration-300 hover:scale-105 focus:ring-4 focus:ring-gray-300/50"
            @click="cancelSelection"
        >
          取消
        </button>
        <button
            v-if="!isCycling && !revealedPrize"
            class="px-6 py-3 bg-blue-600 text-white rounded-xl hover:bg-blue-700 text-lg font-medium shadow-md transition-all duration-300 hover:scale-105 focus:ring-4 focus:ring-blue-300/50"
            :disabled="selectedIndex === null"
            @click="startCycle"
        >
          开始
        </button>
        <button
            v-if="revealedPrize"
            class="px-6 py-3 bg-green-600 text-white rounded-xl hover:bg-green-700 text-lg font-medium shadow-md transition-all duration-300 hover:scale-105 focus:ring-4 focus:ring-green-300/50"
            @click="retryGame"
        >
          重试
        </button>
      </div>
    </div>
  </div>
</template>

<script setup>
import { ref } from 'vue';

// 动态加载图片
const images = import.meta.glob('/src/assets/*.png', { eager: true });
const backImage = images['/src/assets/box2.png']?.default || 'https://via.placeholder.com/64?text=Box';
const prizes = [
  { name: 'Labubu 1', image: images['/src/assets/labubu1.png']?.default || 'https://via.placeholder.com/500?text=Labubu1' },
  { name: 'Labubu 2', image: images['/src/assets/labubu2.png']?.default || 'https://via.placeholder.com/500?text=Labubu2' },
  { name: 'Labubu 3', image: images['/src/assets/labubu3.png']?.default || 'https://via.placeholder.com/500?text=Labubu3' },
  { name: 'Labubu 4', image: images['/src/assets/labubu4.png']?.default || 'https://via.placeholder.com/500?text=Labubu4' },
  { name: 'Labubu 5', image: images['/src/assets/labubu5.png']?.default || 'https://via.placeholder.com/500?text=Labubu5' },
  { name: 'Labubu 6', image: images['/src/assets/labubu6.png']?.default || 'https://via.placeholder.com/500?text=Labubu6' },
];

// 调试图片加载
console.log('Loaded images:', Object.keys(images));

// 状态定义
const boxes = ref(
    Array(9)
        .fill()
        .map(() => ({
          isRevealed: false,
          image: '',
          prize: '',
        }))
);
const selectedIndex = ref(null);
const message = ref('');
const isCycling = ref(false);
const revealedPrize = ref(null);

// 方法
const selectBox = (index) => {
  if (!boxes.value[index].isRevealed && !isCycling.value && !revealedPrize.value) {
    console.log('Box clicked:', index);
    selectedIndex.value = index;
  }
};

const startCycle = () => {
  if (selectedIndex.value === null) return;
  isCycling.value = true;
  message.value = '';

  // 模拟循环动画，2.5秒后揭示盲盒
  setTimeout(() => {
    revealBox();
    isCycling.value = false;
  }, 2500);
};

const revealBox = () => {
  if (selectedIndex.value === null) return;
  console.log('Revealing box:', selectedIndex.value);
  const randomPrize = prizes[Math.floor(Math.random() * prizes.length)];
  boxes.value[selectedIndex.value] = {
    ...boxes.value[selectedIndex.value],
    isRevealed: true,
    image: randomPrize.image,
    prize: randomPrize.name,
  };
  revealedPrize.value = {
    name: randomPrize.name,
    image: randomPrize.image,
  };
  message.value = `哇！你抽到了 ${randomPrize.name}！`;
  selectedIndex.value = null;
};

const cancelSelection = () => {
  selectedIndex.value = null;
  message.value = '';
};

const retryGame = () => {
  revealedPrize.value = null;
  selectedIndex.value = null;
  message.value = '';
  boxes.value = Array(9)
      .fill()
      .map(() => ({
        isRevealed: false,
        image: '',
        prize: '',
      }));
};

const handleImageError = (event) => {
  console.error('Image failed to load:', event.target.src);
  event.target.src = 'https://via.placeholder.com/500?text=Error';
};
</script>

<style>
@keyframes cycle {
  0% {
    transform: scale(1) rotate(0deg);
    background: linear-gradient(145deg, #ffffff, #e5e7eb);
    box-shadow: 0 4px 10px rgba(0, 0, 0, 0.15);
  }
  50% {
    transform: scale(1.2) rotate(5deg);
    background: linear-gradient(145deg, #fef08a, #fde047);
    box-shadow: 0 8px 16px rgba(0, 0, 0, 0.25);
  }
  100% {
    transform: scale(1) rotate(0deg);
    background: linear-gradient(145deg, #ffffff, #e5e7eb);
    box-shadow: 0 4px 10px rgba(0, 0, 0, 0.15);
  }
}

@keyframes reveal {
  0% {
    opacity: 0;
    transform: scale(0.8) translateY(20px);
  }
  100% {
    opacity: 1;
    transform: scale(1) translateY(0);
  }
}

.animate-cycle {
  animation: cycle 0.4s infinite;
}

.animate-reveal {
  animation: reveal 0.7s ease-out forwards;
}
</style>