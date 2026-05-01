<template>
  <div class="review-mode">
    <div v-if="!isReviewing && !showResult" class="review-start">
      <h2>📝 复习模式</h2>
      <p class="review-description">随机打乱卡片顺序，逐张复习，标记认识或不认识</p>
      <button 
        v-if="cards.length > 0" 
        class="start-btn"
        @click="startReview"
      >
        开始复习 ({{ cards.length }} 张卡片)
      </button>
      <p v-else class="no-cards-text">没有卡片可供复习</p>
      <button class="back-btn" @click="exitReview">
        返回卡片列表
      </button>
    </div>

    <div v-if="isReviewing && !showResult" class="review-in-progress">
      <div class="review-progress">
        <div class="progress-info">
          <span class="current-card">第 {{ currentIndex + 1 }} 张</span>
          <span class="total-cards">/ {{ shuffledCards.length }}</span>
        </div>
        <div class="progress-bar">
          <div 
            class="progress-fill" 
            :style="{ width: ((currentIndex + 1) / shuffledCards.length * 100) + '%' }"
          ></div>
        </div>
        <div class="progress-stats">
          <span class="stat-known">✅ 认识: {{ knownCount }}</span>
          <span class="stat-unknown">❌ 不认识: {{ unknownCount }}</span>
        </div>
      </div>

      <div class="flash-card-wrapper">
        <div 
          class="flash-card" 
          :class="{ 'is-flipped': isFlipped }"
          @click="flipCard"
        >
          <div class="flash-card__face flash-card__face--front">
            <div class="word">{{ currentCard.word }}</div>
            <div class="hint">点击翻转查看释义</div>
          </div>
          <div class="flash-card__face flash-card__face--back">
            <div class="meaning" v-if="currentCard.meaning">
              <span class="label">释义：</span>
              <span class="content">{{ currentCard.meaning }}</span>
            </div>
            <div class="example" v-if="currentCard.example">
              <span class="label">例句：</span>
              <span class="content">{{ currentCard.example }}</span>
            </div>
            <div class="hint">点击翻回</div>
          </div>
        </div>
      </div>

      <div class="review-actions">
        <button 
          class="action-btn known-btn"
          :class="{ disabled: !isFlipped }"
          @click="markKnown"
          :disabled="!isFlipped"
        >
          ✅ 认识
        </button>
        <button 
          class="action-btn unknown-btn"
          :class="{ disabled: !isFlipped }"
          @click="markUnknown"
          :disabled="!isFlipped"
        >
          ❌ 不认识
        </button>
      </div>
      <p class="action-hint" v-if="!isFlipped">请先翻转卡片查看答案</p>
    </div>

    <div v-if="showResult" class="review-result">
      <h2>🎉 复习完成！</h2>
      <div class="result-summary">
        <div class="result-card">
          <div class="result-icon">📊</div>
          <div class="result-total">
            <span class="result-label">总共复习</span>
            <span class="result-value">{{ shuffledCards.length }}</span>
            <span class="result-unit">张卡片</span>
          </div>
        </div>
        <div class="result-details">
          <div class="result-item known">
            <div class="result-item-icon">✅</div>
            <div class="result-item-info">
              <span class="result-item-label">认识</span>
              <span class="result-item-value">{{ knownCount }}</span>
            </div>
          </div>
          <div class="result-item unknown">
            <div class="result-item-icon">❌</div>
            <div class="result-item-info">
              <span class="result-item-label">不认识</span>
              <span class="result-item-value">{{ unknownCount }}</span>
            </div>
          </div>
        </div>
        <div class="result-percentage">
          <div class="percentage-bar">
            <div 
              class="percentage-fill known" 
              :style="{ width: (knownCount / shuffledCards.length * 100) + '%' }"
            ></div>
            <div 
              class="percentage-fill unknown" 
              :style="{ width: (unknownCount / shuffledCards.length * 100) + '%' }"
            ></div>
          </div>
          <div class="percentage-text">
            <span class="known-percentage">{{ Math.round(knownCount / shuffledCards.length * 100) }}% 认识</span>
            <span class="unknown-percentage">{{ Math.round(unknownCount / shuffledCards.length * 100) }}% 不认识</span>
          </div>
        </div>
      </div>
      <div class="result-actions">
        <button class="result-btn restart-btn" @click="restartReview">
          🔄 再复习一次
        </button>
        <button class="result-btn back-btn" @click="exitReview">
          🏠 返回卡片列表
        </button>
      </div>
    </div>
  </div>
</template>

<script setup>
import { ref, computed } from 'vue'

const props = defineProps({
  cards: {
    type: Array,
    default: () => []
  }
})

const emit = defineEmits(['exit-review'])

const isReviewing = ref(false)
const showResult = ref(false)
const shuffledCards = ref([])
const currentIndex = ref(0)
const isFlipped = ref(false)
const knownCount = ref(0)
const unknownCount = ref(0)

const currentCard = computed(() => {
  return shuffledCards.value[currentIndex.value] || {}
})

const shuffleArray = (array) => {
  const shuffled = [...array]
  for (let i = shuffled.length - 1; i > 0; i--) {
    const j = Math.floor(Math.random() * (i + 1))
    ;[shuffled[i], shuffled[j]] = [shuffled[j], shuffled[i]]
  }
  return shuffled
}

const startReview = () => {
  shuffledCards.value = shuffleArray(props.cards)
  currentIndex.value = 0
  isFlipped.value = false
  knownCount.value = 0
  unknownCount.value = 0
  isReviewing.value = true
  showResult.value = false
}

const flipCard = () => {
  isFlipped.value = !isFlipped.value
}

const markKnown = () => {
  if (!isFlipped.value) return
  
  knownCount.value++
  nextCard()
}

const markUnknown = () => {
  if (!isFlipped.value) return
  
  unknownCount.value++
  nextCard()
}

const nextCard = () => {
  if (currentIndex.value < shuffledCards.value.length - 1) {
    currentIndex.value++
    isFlipped.value = false
  } else {
    showResult.value = true
    isReviewing.value = false
  }
}

const restartReview = () => {
  startReview()
}

const exitReview = () => {
  isReviewing.value = false
  showResult.value = false
  emit('exit-review')
}
</script>

<style scoped>
.review-mode {
  max-width: 800px;
  margin: 0 auto;
  padding: 20px;
}

.review-start {
  text-align: center;
  padding: 40px 20px;
  background: rgba(255, 255, 255, 0.1);
  border-radius: 20px;
  backdrop-filter: blur(10px);
}

.review-start h2 {
  color: white;
  font-size: 2rem;
  margin-bottom: 15px;
  text-shadow: 2px 2px 4px rgba(0, 0, 0, 0.2);
}

.review-description {
  color: rgba(255, 255, 255, 0.9);
  font-size: 1.1rem;
  margin-bottom: 30px;
  line-height: 1.6;
}

.start-btn {
  background: linear-gradient(135deg, #667eea 0%, #764ba2 100%);
  color: white;
  border: none;
  padding: 15px 40px;
  font-size: 1.2rem;
  font-weight: bold;
  border-radius: 12px;
  cursor: pointer;
  transition: transform 0.2s, box-shadow 0.2s;
  margin-bottom: 20px;
}

.start-btn:hover {
  transform: translateY(-3px);
  box-shadow: 0 10px 20px rgba(102, 126, 234, 0.4);
}

.start-btn:active {
  transform: translateY(0);
}

.no-cards-text {
  color: rgba(255, 255, 255, 0.7);
  font-size: 1.1rem;
  margin-bottom: 20px;
}

.back-btn {
  background: rgba(255, 255, 255, 0.2);
  color: white;
  border: 2px solid rgba(255, 255, 255, 0.3);
  padding: 12px 30px;
  font-size: 1rem;
  font-weight: 600;
  border-radius: 10px;
  cursor: pointer;
  transition: background 0.2s, transform 0.2s;
}

.back-btn:hover {
  background: rgba(255, 255, 255, 0.3);
  transform: translateY(-2px);
}

.review-in-progress {
  display: flex;
  flex-direction: column;
  align-items: center;
}

.review-progress {
  width: 100%;
  max-width: 500px;
  margin-bottom: 30px;
  background: rgba(255, 255, 255, 0.1);
  padding: 20px;
  border-radius: 15px;
  backdrop-filter: blur(10px);
}

.progress-info {
  display: flex;
  justify-content: center;
  margin-bottom: 15px;
  color: white;
  font-size: 1.1rem;
  font-weight: 600;
}

.current-card {
  color: #fff;
}

.total-cards {
  color: rgba(255, 255, 255, 0.7);
}

.progress-bar {
  width: 100%;
  height: 10px;
  background: rgba(255, 255, 255, 0.2);
  border-radius: 5px;
  overflow: hidden;
  margin-bottom: 15px;
}

.progress-fill {
  height: 100%;
  background: linear-gradient(90deg, #667eea 0%, #764ba2 100%);
  border-radius: 5px;
  transition: width 0.3s ease;
}

.progress-stats {
  display: flex;
  justify-content: space-around;
  color: white;
  font-size: 0.95rem;
}

.stat-known {
  color: #4ade80;
}

.stat-unknown {
  color: #f87171;
}

.flash-card-wrapper {
  width: 100%;
  max-width: 500px;
  height: 320px;
  perspective: 1000px;
  margin-bottom: 30px;
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
  border-radius: 20px;
  padding: 30px;
  display: flex;
  flex-direction: column;
  justify-content: center;
  align-items: center;
  text-align: center;
  box-shadow: 0 15px 40px rgba(0, 0, 0, 0.3);
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
  font-size: 3rem;
  font-weight: bold;
  margin-bottom: 20px;
}

.meaning, .example {
  width: 100%;
  margin-bottom: 20px;
  text-align: left;
}

.meaning {
  margin-bottom: 25px;
}

.label {
  font-weight: bold;
  color: #667eea;
  display: block;
  margin-bottom: 6px;
  font-size: 1.1rem;
}

.content {
  font-size: 1.2rem;
  line-height: 1.7;
}

.hint {
  font-size: 1rem;
  opacity: 0.7;
  margin-top: auto;
}

.flash-card__face--front .hint {
  color: rgba(255, 255, 255, 0.8);
}

.review-actions {
  display: flex;
  gap: 20px;
  margin-bottom: 15px;
}

.action-btn {
  padding: 15px 40px;
  font-size: 1.1rem;
  font-weight: bold;
  border: none;
  border-radius: 12px;
  cursor: pointer;
  transition: transform 0.2s, box-shadow 0.2s, opacity 0.2s;
}

.action-btn:hover:not(.disabled) {
  transform: translateY(-3px);
}

.action-btn:active:not(.disabled) {
  transform: translateY(0);
}

.known-btn {
  background: linear-gradient(135deg, #4ade80 0%, #22c55e 100%);
  color: white;
  box-shadow: 0 5px 15px rgba(74, 222, 128, 0.4);
}

.known-btn:hover:not(.disabled) {
  box-shadow: 0 10px 25px rgba(74, 222, 128, 0.5);
}

.unknown-btn {
  background: linear-gradient(135deg, #f87171 0%, #ef4444 100%);
  color: white;
  box-shadow: 0 5px 15px rgba(248, 113, 113, 0.4);
}

.unknown-btn:hover:not(.disabled) {
  box-shadow: 0 10px 25px rgba(248, 113, 113, 0.5);
}

.action-btn.disabled {
  opacity: 0.5;
  cursor: not-allowed;
  transform: none;
}

.action-hint {
  color: rgba(255, 255, 255, 0.8);
  font-size: 0.95rem;
  font-style: italic;
}

.review-result {
  text-align: center;
  padding: 40px 20px;
  background: rgba(255, 255, 255, 0.1);
  border-radius: 20px;
  backdrop-filter: blur(10px);
}

.review-result h2 {
  color: white;
  font-size: 2rem;
  margin-bottom: 30px;
  text-shadow: 2px 2px 4px rgba(0, 0, 0, 0.2);
}

.result-summary {
  max-width: 500px;
  margin: 0 auto 30px;
}

.result-card {
  background: rgba(255, 255, 255, 0.15);
  padding: 25px;
  border-radius: 15px;
  margin-bottom: 25px;
}

.result-icon {
  font-size: 3rem;
  margin-bottom: 15px;
}

.result-total {
  display: flex;
  align-items: baseline;
  justify-content: center;
  gap: 10px;
  color: white;
}

.result-label {
  font-size: 1.1rem;
  opacity: 0.9;
}

.result-value {
  font-size: 2.5rem;
  font-weight: bold;
}

.result-unit {
  font-size: 1.1rem;
  opacity: 0.9;
}

.result-details {
  display: flex;
  justify-content: space-around;
  margin-bottom: 25px;
}

.result-item {
  display: flex;
  flex-direction: column;
  align-items: center;
  padding: 20px;
  border-radius: 12px;
  min-width: 140px;
}

.result-item.known {
  background: rgba(74, 222, 128, 0.2);
  border: 2px solid rgba(74, 222, 128, 0.4);
}

.result-item.unknown {
  background: rgba(248, 113, 113, 0.2);
  border: 2px solid rgba(248, 113, 113, 0.4);
}

.result-item-icon {
  font-size: 2rem;
  margin-bottom: 10px;
}

.result-item-info {
  display: flex;
  flex-direction: column;
  align-items: center;
  gap: 5px;
}

.result-item-label {
  color: rgba(255, 255, 255, 0.8);
  font-size: 1rem;
}

.result-item-value {
  color: white;
  font-size: 2rem;
  font-weight: bold;
}

.result-percentage {
  margin-bottom: 20px;
}

.percentage-bar {
  height: 12px;
  background: rgba(255, 255, 255, 0.2);
  border-radius: 6px;
  overflow: hidden;
  margin-bottom: 10px;
  display: flex;
}

.percentage-fill {
  height: 100%;
  transition: width 0.5s ease;
}

.percentage-fill.known {
  background: linear-gradient(90deg, #4ade80 0%, #22c55e 100%);
}

.percentage-fill.unknown {
  background: linear-gradient(90deg, #f87171 0%, #ef4444 100%);
}

.percentage-text {
  display: flex;
  justify-content: space-between;
  color: white;
  font-size: 0.95rem;
  font-weight: 600;
}

.known-percentage {
  color: #4ade80;
}

.unknown-percentage {
  color: #f87171;
}

.result-actions {
  display: flex;
  justify-content: center;
  gap: 20px;
  flex-wrap: wrap;
}

.result-btn {
  padding: 15px 35px;
  font-size: 1.1rem;
  font-weight: bold;
  border: none;
  border-radius: 12px;
  cursor: pointer;
  transition: transform 0.2s, box-shadow 0.2s;
}

.result-btn:hover {
  transform: translateY(-3px);
}

.result-btn:active {
  transform: translateY(0);
}

.restart-btn {
  background: linear-gradient(135deg, #667eea 0%, #764ba2 100%);
  color: white;
  box-shadow: 0 5px 15px rgba(102, 126, 234, 0.4);
}

.restart-btn:hover {
  box-shadow: 0 10px 25px rgba(102, 126, 234, 0.5);
}

.result-btn.back-btn {
  background: rgba(255, 255, 255, 0.2);
  color: white;
  border: 2px solid rgba(255, 255, 255, 0.3);
}

.result-btn.back-btn:hover {
  background: rgba(255, 255, 255, 0.3);
}

@media (max-width: 640px) {
  .review-actions {
    flex-direction: column;
    width: 100%;
    max-width: 300px;
  }
  
  .action-btn {
    width: 100%;
  }
  
  .result-details {
    flex-direction: column;
    gap: 15px;
  }
  
  .result-item {
    width: 100%;
  }
  
  .result-actions {
    flex-direction: column;
    width: 100%;
    max-width: 300px;
    margin: 0 auto;
  }
  
  .result-btn {
    width: 100%;
  }
}
</style>