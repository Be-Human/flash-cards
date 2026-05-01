<template>
  <div class="app">
    <header class="header">
      <h1>📚 单词闪卡应用</h1>
      <p class="subtitle">轻松记忆单词，高效学习</p>
    </header>

    <AddCard @add-card="addCard" />

    <CardList 
      :cards="cards" 
      @delete-card="deleteCard" 
    />

    <div v-if="cards.length === 0" class="empty-state">
      <div class="empty-icon">📖</div>
      <p>还没有单词卡片，快来添加第一张吧！</p>
    </div>
  </div>
</template>

<script setup>
import { ref, onMounted, watch } from 'vue'
import AddCard from './components/AddCard.vue'
import CardList from './components/CardList.vue'

const cards = ref([])

const loadCards = () => {
  const savedCards = localStorage.getItem('flashCards')
  if (savedCards) {
    try {
      cards.value = JSON.parse(savedCards)
    } catch (e) {
      console.error('Failed to load cards:', e)
      cards.value = []
    }
  }
}

const saveCards = () => {
  localStorage.setItem('flashCards', JSON.stringify(cards.value))
}

const addCard = (cardData) => {
  const newCard = {
    id: Date.now().toString(),
    ...cardData,
    createdAt: new Date().toISOString()
  }
  cards.value.unshift(newCard)
}

const deleteCard = (id) => {
  if (confirm('确定要删除这张卡片吗？')) {
    cards.value = cards.value.filter(card => card.id !== id)
  }
}

onMounted(() => {
  loadCards()
})

watch(cards, () => {
  saveCards()
}, { deep: true })
</script>

<style scoped>
.app {
  min-height: 100vh;
}

.header {
  text-align: center;
  color: white;
  margin-bottom: 40px;
}

.header h1 {
  font-size: 2.5rem;
  margin-bottom: 10px;
  text-shadow: 2px 2px 4px rgba(0, 0, 0, 0.2);
}

.subtitle {
  font-size: 1.1rem;
  opacity: 0.9;
}

.empty-state {
  text-align: center;
  color: white;
  padding: 60px 20px;
}

.empty-icon {
  font-size: 5rem;
  margin-bottom: 20px;
  animation: bounce 2s infinite;
}

@keyframes bounce {
  0%, 20%, 50%, 80%, 100% {
    transform: translateY(0);
  }
  40% {
    transform: translateY(-20px);
  }
  60% {
    transform: translateY(-10px);
  }
}
</style>
