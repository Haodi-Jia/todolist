<template>
  <div class="app-container">
    <!-- 头部标题 -->
    <header class="app-header">
      <div class="header-content">
        <h1 class="main-title">
          <span class="title-icon">🌸</span>
          WHU Todo
          <span class="title-icon">🏛️</span>
        </h1>
        <p class="subtitle">在珞珈山下，留下属于你的青春印记</p>
        <div class="progress-bar">
          <div class="progress-fill" :style="{ width: `${completionPercentage}%` }"></div>
          <span class="progress-text">{{ completedCount }}/{{ totalCount }} 已完成</span>
        </div>
      </div>
    </header>

    <!-- 时间线容器 -->
    <div class="timeline-container">
      <div class="timeline-trunk"></div>
      
      <!-- Todo卡片列表 -->
      <div class="todo-list">
        <TodoCard 
          v-for="(todo, index) in sortedTodos" 
          :key="todo.id"
          :todo="todo"
          :position="index % 2 === 0 ? 'left' : 'right'"
          :style="{ animationDelay: `${index * 0.1}s` }"
        />
      </div>
    </div>

    <!-- 背景装饰 -->
    <div class="background-decoration">
      <div class="floating-sakura" v-for="n in 20" :key="n" :style="getSakuraStyle(n)">🌸</div>
    </div>
  </div>
</template>

<script setup lang="ts">
import { ref, computed, onMounted } from 'vue';
import TodoCard from './components/TodoCard.vue';
import type { Todo } from './types/todo';

const todos = ref<Todo[]>([]);
const loading = ref(true);

// 加载todo数据
const loadTodos = async () => {
  try {
    const response = await fetch('/todos.json');
    const data = await response.json();
    todos.value = data;
  } catch (error) {
    console.error('Failed to load todos:', error);
  } finally {
    loading.value = false;
  }
};

// 排序逻辑：根据date字段排序，有日期的在前且按日期排序，没有日期的保持原顺序在后
const sortedTodos = computed(() => {
  const todosWithDate = todos.value.filter(todo => todo.date && todo.date.trim() !== '');
  const todosWithoutDate = todos.value.filter(todo => !todo.date || todo.date.trim() === '');
  
  // 有日期的按日期排序（最早的在前面）
  todosWithDate.sort((a, b) => new Date(a.date).getTime() - new Date(b.date).getTime());
  
  // 没有日期的保持原来的id顺序
  todosWithoutDate.sort((a, b) => a.id - b.id);
  
  // 返回合并后的数组：有日期的在前，没有日期的在后
  return [...todosWithDate, ...todosWithoutDate];
});

// 计算完成进度
const completedCount = computed(() => todos.value.filter(todo => todo.complete).length);
const totalCount = computed(() => todos.value.length);
const completionPercentage = computed(() => 
  totalCount.value > 0 ? Math.round((completedCount.value / totalCount.value) * 100) : 0
);


// 生成樱花飘落样式
const getSakuraStyle = (n: number) => {
  const delay = Math.random() * 10;
  const duration = 15 + Math.random() * 10;
  const left = Math.random() * 100;
  const size = 0.8 + Math.random() * 0.4;
  
  return {
    left: `${left}%`,
    animationDelay: `${delay}s`,
    animationDuration: `${duration}s`,
    fontSize: `${size}rem`,
    opacity: 0.6 + Math.random() * 0.4
  };
};

onMounted(async () => {
  await loadTodos();
});
</script>

<style scoped>
.app-container {
  min-height: 100vh;
  background: linear-gradient(180deg, 
    #ffeef0 0%, 
    #fff5f7 25%, 
    #f8f9ff 50%, 
    #f0f8ff 75%, 
    #e8f5e8 100%
  );
  position: relative;
  overflow-x: hidden;
}

.app-header {
  text-align: center;
  padding: 3rem 2rem 2rem;
  position: relative;
  z-index: 10;
}

.header-content {
  max-width: 800px;
  margin: 0 auto;
}

.main-title {
  font-size: 3rem;
  font-weight: 700;
  color: #2c3e50;
  margin: 0 0 1rem;
  text-shadow: 2px 2px 4px rgba(0, 0, 0, 0.1);
  background: linear-gradient(135deg, #d63384, #6f42c1, #0d6efd);
  -webkit-background-clip: text;
  -webkit-text-fill-color: transparent;
  background-clip: text;
}

.title-icon {
  display: inline-block;
  margin: 0 1rem;
  animation: bounce 2s infinite;
  font-family: 'Apple Color Emoji', 'Segoe UI Emoji', 'Noto Color Emoji', 'Segoe UI Symbol', 'Android Emoji', 'EmojiSymbols', sans-serif;
  font-variation-settings: 'EMOJIALT' 1;
  text-rendering: auto;
  color: initial;
  -webkit-text-fill-color: initial;
  filter: contrast(1.2) saturate(1.3);
}

.subtitle {
  font-size: 1.2rem;
  color: #5f6368;
  margin: 0 0 2rem;
  font-style: italic;
}

.progress-bar {
  position: relative;
  width: 100%;
  height: 20px;
  background: rgba(255, 255, 255, 0.3);
  border-radius: 10px;
  overflow: hidden;
  backdrop-filter: blur(10px);
  border: 1px solid rgba(255, 255, 255, 0.4);
}

.progress-fill {
  height: 100%;
  background: linear-gradient(90deg, #4CAF50, #81C784, #A5D6A7);
  border-radius: 10px;
  transition: width 0.8s ease;
  position: relative;
}

.progress-fill::after {
  content: '';
  position: absolute;
  top: 0;
  left: 0;
  right: 0;
  bottom: 0;
  background: linear-gradient(90deg, transparent, rgba(255, 255, 255, 0.3), transparent);
  animation: shimmer 2s infinite;
}

.progress-text {
  position: absolute;
  top: 50%;
  left: 50%;
  transform: translate(-50%, -50%);
  font-weight: 600;
  color: #2c3e50;
  font-size: 0.9rem;
  text-shadow: 1px 1px 2px rgba(255, 255, 255, 0.8);
}


.timeline-container {
  position: relative;
  max-width: 1200px;
  margin: 0 auto;
  padding: 2rem 1rem 4rem;
}

.timeline-trunk {
  position: absolute;
  left: 50%;
  top: 0;
  bottom: 0;
  width: 6px;
  background: linear-gradient(180deg, 
    #8B4513 0%, 
    #A0522D 20%, 
    #CD853F 40%, 
    #D2691E 60%, 
    #8B4513 80%, 
    #654321 100%
  );
  transform: translateX(-50%);
  border-radius: 3px;
  box-shadow: 0 0 20px rgba(139, 69, 19, 0.3);
}

.timeline-trunk::before {
  content: '';
  position: absolute;
  top: -20px;
  left: 50%;
  transform: translateX(-50%);
  width: 30px;
  height: 30px;
  background: #228B22;
  border-radius: 50%;
  box-shadow: 0 0 20px rgba(34, 139, 34, 0.5);
}

.timeline-trunk::after {
  content: '🌳';
  position: absolute;
  top: -35px;
  left: 50%;
  transform: translateX(-50%);
  font-size: 2rem;
  z-index: 5;
  font-family: 'Apple Color Emoji', 'Segoe UI Emoji', 'Noto Color Emoji', 'Segoe UI Symbol', 'Android Emoji', 'EmojiSymbols', sans-serif;
  font-variation-settings: 'EMOJIALT' 1;
  text-rendering: auto;
  color: initial;
  -webkit-text-fill-color: initial;
  filter: drop-shadow(0 2px 4px rgba(0, 0, 0, 0.3)) contrast(1.2) saturate(1.3);
}

.todo-list {
  position: relative;
  z-index: 5;
}

.background-decoration {
  position: fixed;
  top: 0;
  left: 0;
  width: 100%;
  height: 100%;
  pointer-events: none;
  z-index: 1;
}

.floating-sakura {
  position: absolute;
  animation: float linear infinite;
  pointer-events: none;
  font-family: 'Apple Color Emoji', 'Segoe UI Emoji', 'Noto Color Emoji', 'Segoe UI Symbol', 'Android Emoji', 'EmojiSymbols', sans-serif;
  font-variation-settings: 'EMOJIALT' 1;
  text-rendering: auto;
  color: initial;
  -webkit-text-fill-color: initial;
  filter: contrast(1.2) saturate(1.3);
}

@keyframes bounce {
  0%, 20%, 50%, 80%, 100% {
    transform: translateY(0);
  }
  40% {
    transform: translateY(-10px);
  }
  60% {
    transform: translateY(-5px);
  }
}

@keyframes shimmer {
  0% {
    transform: translateX(-100%);
  }
  100% {
    transform: translateX(100%);
  }
}

@keyframes float {
  from {
    transform: translateY(-100px) rotate(0deg);
  }
  to {
    transform: translateY(100vh) rotate(360deg);
  }
}

@media (max-width: 768px) {
  .main-title {
    font-size: 2rem;
  }
  
  .title-icon {
    margin: 0 0.5rem;
  }
  
  .subtitle {
    font-size: 1rem;
  }
  
  .timeline-trunk {
    display: none;
  }
  
  .app-header {
    padding: 2rem 1rem 1rem;
  }
}

/* 全局样式重置 */
:global(body) {
  margin: 0;
  padding: 0;
  font-family: 'PingFang SC', 'Hiragino Sans GB', 'Microsoft YaHei', sans-serif;
  line-height: 1.6;
}

:global(*) {
  box-sizing: border-box;
}
</style>