<template>
  <div class="flash-card-wrapper">
    <div 
      class="flash-card" 
      :class="{ 'is-flipped': isFlipped }"
      @click="flipCard"
    >
      <div class="flash-card__face flash-card__face--front">
        <div class="word">{{ card.word }}</div>
        <div class="hint">点击翻转查看释义</div>
      </div>
      <div class="flash-card__face flash-card__face--back">
        <div class="meaning" v-if="card.meaning">
          <span class="label">释义：</span>
          <span class="content">{{ card.meaning }}</span>
        </div>
        <div class="example" v-if="card.example">
          <span class="label">例句：</span>
          <span class="content">{{ card.example }}</span>
        </div>
        <div class="hint">点击翻回</div>
      </div>
    </div>
    <button 
      class="edit-btn"
      @click.stop="$emit('edit-card', card)"
      title="编辑卡片"
    >
      ✏️
    </button>
    <button 
      class="delete-btn"
      @click.stop="$emit('delete-card', card.id)"
      title="删除卡片"
    >
      🗑️
    </button>
  </div>
</template>

<script setup>
import { ref } from 'vue'

defineProps({
  card: {
    type: Object,
    required: true
  }
})

defineEmits(['delete-card', 'edit-card'])

const isFlipped = ref(false)

const flipCard = () => {
  isFlipped.value = !isFlipped.value
}
</script>

<style scoped>
.flash-card-wrapper {
  position: relative;
  width: 100%;
  height: 280px;
  perspective: 1000px;
}

.flash-card {
  position: relative;
  width: 100%;
  height: 100%;
  cursor: pointer;
  transform-style: preserve-3d;
  transition: transform 0.6s;
}

.flash-card.is-flipped {
  transform: rotateY(180deg);
}

.flash-card__face {
  position: absolute;
  width: 100%;
  height: 100%;
  backface-visibility: hidden;
  border-radius: 16px;
  padding: 24px;
  display: flex;
  flex-direction: column;
  justify-content: center;
  align-items: center;
  text-align: center;
  box-shadow: 0 10px 30px rgba(0, 0, 0, 0.2);
}

.flash-card__face--front {
  background: linear-gradient(135deg, #667eea 0%, #764ba2 100%);
  color: white;
}

.flash-card__face--back {
  background: white;
  color: #333;
  transform: rotateY(180deg);
}

.word {
  font-size: 2.5rem;
  font-weight: bold;
  margin-bottom: 16px;
}

.meaning, .example {
  width: 100%;
  margin-bottom: 16px;
  text-align: left;
}

.meaning {
  margin-bottom: 20px;
}

.label {
  font-weight: bold;
  color: #667eea;
  display: block;
  margin-bottom: 4px;
}

.content {
  font-size: 1.1rem;
  line-height: 1.6;
}

.hint {
  font-size: 0.9rem;
  opacity: 0.7;
  margin-top: auto;
}

.flash-card__face--front .hint {
  color: rgba(255, 255, 255, 0.8);
}

.edit-btn {
  position: absolute;
  top: 10px;
  right: 56px;
  width: 36px;
  height: 36px;
  border: none;
  border-radius: 50%;
  background: rgba(102, 126, 234, 0.9);
  color: white;
  font-size: 1rem;
  cursor: pointer;
  z-index: 10;
  opacity: 0;
  transition: opacity 0.3s, transform 0.2s;
}

.delete-btn {
  position: absolute;
  top: 10px;
  right: 10px;
  width: 36px;
  height: 36px;
  border: none;
  border-radius: 50%;
  background: rgba(255, 100, 100, 0.9);
  color: white;
  font-size: 1rem;
  cursor: pointer;
  z-index: 10;
  opacity: 0;
  transition: opacity 0.3s, transform 0.2s;
}

.flash-card-wrapper:hover .edit-btn,
.flash-card-wrapper:hover .delete-btn {
  opacity: 1;
}

.edit-btn:hover {
  transform: scale(1.1);
  background: #5a6fd6;
}

.delete-btn:hover {
  transform: scale(1.1);
  background: #ff4444;
}
</style>
