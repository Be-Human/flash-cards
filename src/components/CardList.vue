<template>
  <div class="card-list">
    <div v-if="cards.length > 0" class="list-header">
      <h2>📋 我的单词卡片 ({{ cards.length }})</h2>
    </div>
    <div class="cards-grid">
      <FlashCard 
        v-for="card in cards" 
        :key="card.id" 
        :card="card"
        @delete-card="$emit('delete-card', $event)"
      />
    </div>
  </div>
</template>

<script setup>
import FlashCard from './FlashCard.vue'

defineProps({
  cards: {
    type: Array,
    default: () => []
  }
})

defineEmits(['delete-card'])
</script>

<style scoped>
.card-list {
  margin-top: 40px;
}

.list-header {
  text-align: center;
  margin-bottom: 30px;
}

.list-header h2 {
  color: white;
  font-size: 1.5rem;
  text-shadow: 1px 1px 2px rgba(0, 0, 0, 0.2);
}

.cards-grid {
  display: grid;
  grid-template-columns: repeat(auto-fill, minmax(300px, 1fr));
  gap: 24px;
  padding: 0 10px;
}

@media (max-width: 640px) {
  .cards-grid {
    grid-template-columns: 1fr;
    gap: 20px;
  }
}
</style>
