<template>
  <div class="todo-card" :class="{ 'completed': todo.complete, 'left': position === 'left', 'right': position === 'right' }">
    <div class="card-content">
      <div class="card-header">
        <label class="checkbox-container">
          <input 
            type="checkbox" 
            :checked="todo.complete"
            @change="toggleComplete"
          />
          <span class="checkmark"></span>
        </label>
        <h3 class="card-title">{{ todo.title }}</h3>
      </div>
      
      <div class="card-body">
        <p class="card-description" v-if="todo.description">{{ todo.description }}</p>
        
        <div class="image-gallery" v-if="todo.image && todo.image.length > 0">
          <div class="image-carousel">
            <button 
              class="carousel-btn prev-btn" 
              @click="previousImage"
              v-if="todo.image.length > 1"
              :disabled="currentImageIndex === 0"
            >
              ‹
            </button>
            
            <div class="image-container">
              <img 
                :src="todo.image[currentImageIndex]" 
                :alt="`图片 ${currentImageIndex + 1}`"
                class="todo-image"
                @error="handleImageError"
              />
              <div class="image-indicator" v-if="todo.image.length > 1">
                {{ currentImageIndex + 1 }} / {{ todo.image.length }}
              </div>
            </div>
            
            <button 
              class="carousel-btn next-btn" 
              @click="nextImage"
              v-if="todo.image.length > 1"
              :disabled="currentImageIndex === todo.image.length - 1"
            >
              ›
            </button>
          </div>
        </div>
        
        <div class="date-info" v-if="todo.date">
          <span class="date-label">完成时间：</span>
          <span class="date-value">{{ formatDate(todo.date) }}</span>
        </div>
      </div>
    </div>
    
    <div class="connector-line"></div>
    <div class="tree-node"></div>
  </div>
</template>

<script setup lang="ts">
import { ref } from 'vue';
import type { Todo } from '../types/todo';

interface Props {
  todo: Todo;
  position: 'left' | 'right';
}

interface Emits {
  (e: 'update', todo: Todo): void;
}

const props = defineProps<Props>();
const emit = defineEmits<Emits>();

const currentImageIndex = ref(0);

const toggleComplete = () => {
  const updatedTodo = {
    ...props.todo,
    complete: !props.todo.complete,
    date: !props.todo.complete ? new Date().toISOString() : ''
  };
  emit('update', updatedTodo);
};

const formatDate = (dateString: string) => {
  if (!dateString) return '';
  const date = new Date(dateString);
  return date.toLocaleDateString('zh-CN', {
    year: 'numeric',
    month: 'long',
    day: 'numeric'
  });
};

const previousImage = () => {
  if (currentImageIndex.value > 0) {
    currentImageIndex.value--;
  }
};

const nextImage = () => {
  if (currentImageIndex.value < props.todo.image.length - 1) {
    currentImageIndex.value++;
  }
};

const handleImageError = (event: Event) => {
  const img = event.target as HTMLImageElement;
  img.style.display = 'none';
  console.warn('图片加载失败:', img.src);
};
</script>

<style scoped>
.todo-card {
  position: relative;
  margin: 2rem 0;
  max-width: 400px;
  opacity: 0;
  animation: fadeInUp 0.6s ease-out forwards;
}

.todo-card.left {
  margin-right: auto;
  margin-left: 2rem;
}

.todo-card.right {
  margin-left: auto;
  margin-right: 2rem;
}

.card-content {
  background: linear-gradient(135deg, #ffffff 0%, #f8f9fa 100%);
  border-radius: 15px;
  padding: 1.5rem;
  box-shadow: 0 8px 32px rgba(0, 0, 0, 0.1);
  border: 1px solid rgba(255, 255, 255, 0.3);
  backdrop-filter: blur(10px);
  transition: all 0.3s ease;
  position: relative;
  overflow: hidden;
}

.card-content::before {
  content: '';
  position: absolute;
  top: 0;
  left: 0;
  right: 0;
  height: 4px;
  background: linear-gradient(90deg, #4CAF50, #81C784);
  border-radius: 15px 15px 0 0;
}

.completed .card-content {
  background: linear-gradient(135deg, #e8f5e8 0%, #f1f8e9 100%);
  border-color: #4CAF50;
}

.completed .card-content::before {
  background: linear-gradient(90deg, #4CAF50, #66BB6A);
}

.card-content:hover {
  transform: translateY(-5px);
  box-shadow: 0 12px 40px rgba(0, 0, 0, 0.15);
}

.card-header {
  display: flex;
  align-items: flex-start;
  gap: 1rem;
  margin-bottom: 1rem;
}

.checkbox-container {
  position: relative;
  cursor: pointer;
  display: block;
  margin-top: 0.2rem;
}

.checkbox-container input {
  position: absolute;
  opacity: 0;
  cursor: pointer;
  height: 0;
  width: 0;
}

.checkmark {
  display: block;
  height: 24px;
  width: 24px;
  background-color: #ffffff;
  border: 2px solid #ddd;
  border-radius: 50%;
  transition: all 0.3s ease;
}

.checkbox-container:hover .checkmark {
  border-color: #4CAF50;
  box-shadow: 0 0 10px rgba(76, 175, 80, 0.3);
}

.checkbox-container input:checked ~ .checkmark {
  background-color: #4CAF50;
  border-color: #4CAF50;
}

.checkmark:after {
  content: "";
  position: absolute;
  display: none;
}

.checkbox-container input:checked ~ .checkmark:after {
  display: block;
}

.checkbox-container .checkmark:after {
  left: 8px;
  top: 4px;
  width: 6px;
  height: 12px;
  border: solid white;
  border-width: 0 2px 2px 0;
  transform: rotate(45deg);
}

.card-title {
  font-size: 1.1rem;
  font-weight: 600;
  color: #2c3e50;
  margin: 0;
  line-height: 1.4;
  flex: 1;
}

.completed .card-title {
  color: #4CAF50;
  text-decoration: line-through;
}

.card-body {
  margin-left: 2rem;
}

.card-description {
  color: #5f6368;
  line-height: 1.6;
  margin: 0.5rem 0;
  font-size: 0.9rem;
}

.image-gallery {
  margin: 1rem 0;
  border-radius: 12px;
  overflow: hidden;
}

.image-carousel {
  position: relative;
  display: flex;
  align-items: center;
  background: #f8f9fa;
  border-radius: 12px;
  overflow: hidden;
}

.image-container {
  flex: 1;
  position: relative;
  min-height: 150px;
  max-height: 300px;
  overflow: hidden;
  display: flex;
  align-items: center;
  justify-content: center;
  background: #f8f9fa;
}

.todo-image {
  max-width: 100%;
  max-height: 300px;
  width: auto;
  height: auto;
  object-fit: contain;
  transition: transform 0.3s ease;
}

.image-indicator {
  position: absolute;
  bottom: 8px;
  right: 8px;
  background: rgba(0, 0, 0, 0.7);
  color: white;
  padding: 4px 8px;
  border-radius: 12px;
  font-size: 0.75rem;
  font-weight: 500;
}

.carousel-btn {
  position: absolute;
  top: 50%;
  transform: translateY(-50%);
  background: rgba(255, 255, 255, 0.9);
  border: none;
  width: 40px;
  height: 40px;
  border-radius: 50%;
  display: flex;
  align-items: center;
  justify-content: center;
  font-size: 20px;
  font-weight: bold;
  color: #333;
  cursor: pointer;
  transition: all 0.3s ease;
  z-index: 2;
  backdrop-filter: blur(10px);
  box-shadow: 0 2px 8px rgba(0, 0, 0, 0.15);
}

.carousel-btn:hover:not(:disabled) {
  background: rgba(255, 255, 255, 1);
  transform: translateY(-50%) scale(1.1);
  box-shadow: 0 4px 12px rgba(0, 0, 0, 0.2);
}

.carousel-btn:disabled {
  opacity: 0.3;
  cursor: not-allowed;
}

.prev-btn {
  left: 8px;
}

.next-btn {
  right: 8px;
}

.carousel-btn:active {
  transform: translateY(-50%) scale(0.95);
}

.date-info {
  margin-top: 1rem;
  padding: 0.5rem;
  background: rgba(76, 175, 80, 0.1);
  border-radius: 8px;
  font-size: 0.85rem;
}

.date-label {
  color: #4CAF50;
  font-weight: 600;
}

.date-value {
  color: #2c3e50;
  margin-left: 0.5rem;
}

.connector-line {
  position: absolute;
  width: 3px;
  height: 60px;
  background: linear-gradient(180deg, #4CAF50, #81C784);
  top: 50%;
  transform: translateY(-50%);
  border-radius: 2px;
}

.left .connector-line {
  right: -50px;
}

.right .connector-line {
  left: -50px;
}

.tree-node {
  position: absolute;
  width: 12px;
  height: 12px;
  background: #4CAF50;
  border: 3px solid #ffffff;
  border-radius: 50%;
  top: 50%;
  transform: translateY(-50%);
  box-shadow: 0 2px 8px rgba(76, 175, 80, 0.4);
}

.left .tree-node {
  right: -56px;
}

.right .tree-node {
  left: -56px;
}

.completed .tree-node {
  background: #66BB6A;
  box-shadow: 0 2px 12px rgba(102, 187, 106, 0.6);
}

@keyframes fadeInUp {
  from {
    opacity: 0;
    transform: translateY(30px);
  }
  to {
    opacity: 1;
    transform: translateY(0);
  }
}

@media (max-width: 768px) {
  .todo-card {
    max-width: 90%;
    margin-left: 1rem !important;
    margin-right: 1rem !important;
  }
  
  .card-content {
    padding: 1rem;
  }
  
  .card-body {
    margin-left: 1rem;
  }
  
  .connector-line,
  .tree-node {
    display: none;
  }
}
</style>
