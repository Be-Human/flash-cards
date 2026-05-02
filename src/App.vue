<template>
  <div class="app">
    <header class="header">
      <h1>📚 单词闪卡应用</h1>
      <p class="subtitle">轻松记忆单词，高效学习</p>
    </header>

    <ReviewMode 
      v-if="isReviewMode" 
      :cards="cards" 
      @exit-review="exitReviewMode" 
    />

    <template v-else>
      <div class="app-actions">
        <button 
          v-if="cards.length > 0" 
          class="review-mode-btn"
          @click="enterReviewMode"
        >
          📝 开始复习
        </button>
      </div>

      <AddCard @add-card="handleAddCard" />

      <CardList 
        :cards="cards" 
        @delete-card="showDeleteConfirm" 
        @edit-card="handleEditCard"
      />

      <div v-if="cards.length === 0" class="empty-state">
        <div class="empty-icon">📖</div>
        <p>还没有单词卡片，快来添加第一张吧！</p>
      </div>
    </template>

    <ConfirmModal
      :visible="showDuplicateModal"
      title="单词已存在"
      :message="duplicateMessage"
      icon="⚠️"
      :confirm-text="isEditDuplicate ? '仍要修改' : '仍要添加'"
      cancel-text="取消"
      @update:visible="handleDuplicateModalClose"
      @confirm="handleDuplicateConfirm"
    />

    <ConfirmModal
      :visible="showDeleteModal"
      title="确认删除"
      :message="deleteMessage"
      icon="🗑️"
      confirm-text="删除"
      cancel-text="取消"
      :is-danger="true"
      @update:visible="(val) => showDeleteModal = val"
      @confirm="confirmDelete"
    />

    <EditCard
      :visible="showEditModal"
      :card="editingCard"
      @update:visible="(val) => showEditModal = val"
      @save-card="handleSaveCard"
    />
  </div>
</template>

<script setup>
import { ref, onMounted, watch, computed } from 'vue'
import AddCard from './components/AddCard.vue'
import CardList from './components/CardList.vue'
import ConfirmModal from './components/ConfirmModal.vue'
import ReviewMode from './components/ReviewMode.vue'
import EditCard from './components/EditCard.vue'

const cards = ref([])
const showDuplicateModal = ref(false)
const showDeleteModal = ref(false)
const showEditModal = ref(false)
const pendingCard = ref(null)
const pendingDeleteId = ref(null)
const editingCard = ref(null)
const pendingEditCard = ref(null)
const isEditDuplicate = ref(false)
const isReviewMode = ref(false)

const deleteMessage = computed(() => {
  return '确定要删除这张卡片吗？\n删除后无法恢复。'
})

const duplicateMessage = computed(() => {
  if (isEditDuplicate.value && pendingEditCard.value) {
    const word = pendingEditCard.value.word
    const existingCard = cards.value.find(
      c => c.word.toLowerCase() === word.toLowerCase() && c.id !== pendingEditCard.value.id
    )
    if (existingCard) {
      return '单词 "' + word + '" 已存在：\n释义：' + existingCard.meaning + '\n\n是否仍要修改此卡片？'
    }
    return '单词 "' + word + '" 已存在。\n是否仍要修改？'
  }
  
  if (!pendingCard.value) return ''
  const word = pendingCard.value.word
  const existingCard = cards.value.find(
    c => c.word.toLowerCase() === word.toLowerCase()
  )
  if (existingCard) {
    return '单词 "' + word + '" 已存在：\n释义：' + existingCard.meaning + '\n\n是否仍要添加新的卡片？'
  }
  return '单词 "' + word + '" 已存在。\n是否仍要添加？'
})

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

const isWordDuplicate = (word, excludeId = null) => {
  return cards.value.some(
    card => card.word.toLowerCase() === word.toLowerCase() && card.id !== excludeId
  )
}

const addCardToData = (cardData) => {
  const newCard = {
    id: Date.now().toString(),
    ...cardData,
    createdAt: new Date().toISOString()
  }
  cards.value.unshift(newCard)
}

const handleAddCard = (cardData) => {
  if (isWordDuplicate(cardData.word)) {
    pendingCard.value = cardData
    showDuplicateModal.value = true
  } else {
    addCardToData(cardData)
  }
}

const confirmAddDuplicate = () => {
  if (pendingCard.value) {
    addCardToData(pendingCard.value)
    pendingCard.value = null
  }
}

const handleDuplicateModalClose = (val) => {
  showDuplicateModal.value = val
  if (!val) {
    isEditDuplicate.value = false
    pendingEditCard.value = null
    pendingCard.value = null
  }
}

const handleDuplicateConfirm = () => {
  if (isEditDuplicate.value) {
    confirmEditDuplicate()
  } else {
    confirmAddDuplicate()
  }
}

const showDeleteConfirm = (id) => {
  pendingDeleteId.value = id
  showDeleteModal.value = true
}

const confirmDelete = () => {
  if (pendingDeleteId.value) {
    cards.value = cards.value.filter(card => card.id !== pendingDeleteId.value)
    pendingDeleteId.value = null
  }
}

const handleEditCard = (card) => {
  editingCard.value = { ...card }
  showEditModal.value = true
}

const handleSaveCard = (updatedCard) => {
  if (isWordDuplicate(updatedCard.word, updatedCard.id)) {
    pendingEditCard.value = updatedCard
    isEditDuplicate.value = true
    showDuplicateModal.value = true
    return
  }
  
  saveCardToData(updatedCard)
}

const saveCardToData = (updatedCard) => {
  const index = cards.value.findIndex(card => card.id === updatedCard.id)
  if (index !== -1) {
    cards.value[index] = {
      ...cards.value[index],
      ...updatedCard
    }
  }
  editingCard.value = null
  pendingEditCard.value = null
  isEditDuplicate.value = false
}

const confirmEditDuplicate = () => {
  if (pendingEditCard.value) {
    saveCardToData(pendingEditCard.value)
  }
}

const enterReviewMode = () => {
  isReviewMode.value = true
}

const exitReviewMode = () => {
  isReviewMode.value = false
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

.app-actions {
  display: flex;
  justify-content: center;
  margin-bottom: 30px;
}

.review-mode-btn {
  background: linear-gradient(135deg, #f093fb 0%, #f5576c 100%);
  color: white;
  border: none;
  padding: 12px 30px;
  font-size: 1.1rem;
  font-weight: bold;
  border-radius: 12px;
  cursor: pointer;
  transition: transform 0.2s, box-shadow 0.2s;
  box-shadow: 0 5px 15px rgba(240, 147, 251, 0.4);
}

.review-mode-btn:hover {
  transform: translateY(-3px);
  box-shadow: 0 10px 25px rgba(240, 147, 251, 0.5);
}

.review-mode-btn:active {
  transform: translateY(0);
}
</style>
