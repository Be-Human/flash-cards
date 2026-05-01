<template>
  <div class="card-list">
    <div v-if="cards.length > 0" class="list-header">
      <div class="header-actions">
        <h2>📋 我的单词卡片 ({{ filteredCards.length }})</h2>
        <div class="search-sort-container">
          <div class="search-box">
            <input 
              type="text" 
              v-model="searchQuery"
              placeholder="搜索单词..."
              class="search-input"
            />
            <span v-if="searchQuery" class="search-clear" @click="clearSearch">✕</span>
          </div>
          <div class="sort-box">
            <select v-model="sortBy" class="sort-select">
              <option value="newest">最新添加</option>
              <option value="oldest">最早添加</option>
              <option value="alphabetical">字母顺序 (A-Z)</option>
              <option value="alphabetical-reverse">字母顺序 (Z-A)</option>
            </select>
          </div>
        </div>
      </div>
    </div>
    <div v-if="searchQuery && filteredCards.length === 0" class="no-results">
      <p>没有找到匹配的卡片</p>
    </div>
    <div class="cards-grid">
      <FlashCard 
        v-for="card in filteredCards" 
        :key="card.id" 
        :card="card"
        @delete-card="$emit('delete-card', $event)"
        @edit-card="$emit('edit-card', $event)"
      />
    </div>
  </div>
</template>

<script setup>
import { ref, computed } from 'vue'
import FlashCard from './FlashCard.vue'

const props = defineProps({
  cards: {
    type: Array,
    default: () => []
  }
})

const emit = defineEmits(['delete-card', 'edit-card'])

const searchQuery = ref('')
const sortBy = ref('newest')

const filteredCards = computed(() => {
  let result = [...props.cards]
  
  if (searchQuery.value.trim()) {
    const query = searchQuery.value.toLowerCase().trim()
    result = result.filter(card => 
      card.word.toLowerCase().includes(query) ||
      (card.meaning && card.meaning.toLowerCase().includes(query)) ||
      (card.example && card.example.toLowerCase().includes(query))
    )
  }
  
  switch (sortBy.value) {
    case 'newest':
      result.sort((a, b) => new Date(b.createdAt) - new Date(a.createdAt))
      break
    case 'oldest':
      result.sort((a, b) => new Date(a.createdAt) - new Date(b.createdAt))
      break
    case 'alphabetical':
      result.sort((a, b) => a.word.localeCompare(b.word, 'en', { sensitivity: 'base' }))
      break
    case 'alphabetical-reverse':
      result.sort((a, b) => b.word.localeCompare(a.word, 'en', { sensitivity: 'base' }))
      break
  }
  
  return result
})

const clearSearch = () => {
  searchQuery.value = ''
}
</script>

<style scoped>
.card-list {
  margin-top: 40px;
}

.list-header {
  text-align: center;
  margin-bottom: 30px;
}

.header-actions {
  display: flex;
  flex-direction: column;
  align-items: center;
  gap: 20px;
}

.list-header h2 {
  color: white;
  font-size: 1.5rem;
  text-shadow: 1px 1px 2px rgba(0, 0, 0, 0.2);
  margin: 0;
}

.search-sort-container {
  display: flex;
  gap: 15px;
  align-items: center;
  flex-wrap: wrap;
  justify-content: center;
}

.search-box {
  position: relative;
  display: flex;
  align-items: center;
}

.search-input {
  padding: 10px 40px 10px 15px;
  border: 2px solid rgba(255, 255, 255, 0.3);
  border-radius: 10px;
  background: rgba(255, 255, 255, 0.1);
  color: white;
  font-size: 0.95rem;
  width: 200px;
  transition: border-color 0.3s, background 0.3s;
}

.search-input::placeholder {
  color: rgba(255, 255, 255, 0.6);
}

.search-input:focus {
  outline: none;
  border-color: rgba(102, 126, 234, 0.8);
  background: rgba(255, 255, 255, 0.15);
}

.search-clear {
  position: absolute;
  right: 12px;
  color: rgba(255, 255, 255, 0.6);
  cursor: pointer;
  font-size: 1rem;
  transition: color 0.2s;
}

.search-clear:hover {
  color: white;
}

.sort-box {
  display: flex;
  align-items: center;
}

.sort-select {
  padding: 10px 35px 10px 15px;
  border: 2px solid rgba(255, 255, 255, 0.3);
  border-radius: 10px;
  background: rgba(255, 255, 255, 0.1);
  color: white;
  font-size: 0.95rem;
  cursor: pointer;
  appearance: none;
  background-image: url("data:image/svg+xml,%3Csvg xmlns='http://www.w3.org/2000/svg' fill='none' viewBox='0 0 24 24' stroke='rgba(255,255,255,0.8)'%3E%3Cpath stroke-linecap='round' stroke-linejoin='round' stroke-width='2' d='M19 9l-7 7-7-7'%3E%3C/path%3E%3C/svg%3E");
  background-repeat: no-repeat;
  background-position: right 12px center;
  background-size: 16px;
  transition: border-color 0.3s, background 0.3s;
}

.sort-select:focus {
  outline: none;
  border-color: rgba(102, 126, 234, 0.8);
  background-color: rgba(255, 255, 255, 0.15);
}

.sort-select option {
  background: #667eea;
  color: white;
}

.no-results {
  text-align: center;
  color: rgba(255, 255, 255, 0.7);
  padding: 40px 20px;
  font-size: 1.1rem;
}

.cards-grid {
  display: grid;
  grid-template-columns: repeat(auto-fill, minmax(300px, 1fr));
  gap: 24px;
  padding: 0 10px;
}

@media (max-width: 640px) {
  .search-sort-container {
    flex-direction: column;
    width: 100%;
  }
  
  .search-box,
  .sort-box {
    width: 100%;
    max-width: 300px;
  }
  
  .search-input,
  .sort-select {
    width: 100%;
  }
  
  .cards-grid {
    grid-template-columns: 1fr;
    gap: 20px;
  }
}
</style>
