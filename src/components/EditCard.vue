<template>
  <div v-if="visible" class="modal-overlay" @click.self="handleCancel">
    <div class="modal-container">
      <div class="modal-header">
        <span class="modal-icon">✏️</span>
        <h3>编辑卡片</h3>
      </div>
      <form @submit.prevent="handleSubmit" class="modal-form">
        <div class="form-group" :class="{ 'has-error': wordError }">
          <label for="edit-word">单词 *</label>
          <input 
            type="text" 
            id="edit-word" 
            v-model="formData.word"
            placeholder="请输入单词"
            @input="clearWordError"
          />
          <span v-if="wordError" class="error-message">{{ wordError }}</span>
        </div>
        
        <div class="form-group" :class="{ 'has-error': meaningError }">
          <label for="edit-meaning">释义 *</label>
          <input 
            type="text" 
            id="edit-meaning" 
            v-model="formData.meaning"
            placeholder="请输入单词释义"
            @input="clearMeaningError"
          />
          <span v-if="meaningError" class="error-message">{{ meaningError }}</span>
        </div>
        
        <div class="form-group">
          <label for="edit-example">例句</label>
          <textarea 
            id="edit-example" 
            v-model="formData.example"
            placeholder="请输入例句（可选）"
            rows="2"
          ></textarea>
        </div>
        
        <div class="modal-footer">
          <button type="button" class="btn btn-cancel" @click="handleCancel">
            取消
          </button>
          <button type="submit" class="btn btn-confirm">
            保存
          </button>
        </div>
      </form>
    </div>
  </div>
</template>

<script setup>
import { ref, watch, reactive } from 'vue'

const props = defineProps({
  visible: {
    type: Boolean,
    default: false
  },
  card: {
    type: Object,
    default: null
  }
})

const emit = defineEmits(['update:visible', 'save-card'])

const formData = reactive({
  word: '',
  meaning: '',
  example: ''
})

const wordError = ref('')
const meaningError = ref('')

watch(() => props.card, (newCard) => {
  if (newCard) {
    formData.word = newCard.word || ''
    formData.meaning = newCard.meaning || ''
    formData.example = newCard.example || ''
    wordError.value = ''
    meaningError.value = ''
  }
}, { immediate: true })

const clearWordError = () => {
  wordError.value = ''
}

const clearMeaningError = () => {
  meaningError.value = ''
}

const handleSubmit = () => {
  wordError.value = ''
  meaningError.value = ''
  
  let hasError = false
  
  if (!formData.word.trim()) {
    wordError.value = '请输入单词'
    hasError = true
  }
  
  if (!formData.meaning.trim()) {
    meaningError.value = '请输入释义'
    hasError = true
  }
  
  if (hasError) return
  
  emit('save-card', {
    id: props.card.id,
    word: formData.word.trim(),
    meaning: formData.meaning.trim(),
    example: formData.example.trim()
  })
  
  emit('update:visible', false)
}

const handleCancel = () => {
  emit('update:visible', false)
}
</script>

<style scoped>
.modal-overlay {
  position: fixed;
  top: 0;
  left: 0;
  right: 0;
  bottom: 0;
  background: rgba(0, 0, 0, 0.6);
  display: flex;
  justify-content: center;
  align-items: center;
  z-index: 1000;
  padding: 20px;
}

.modal-container {
  background: white;
  border-radius: 20px;
  padding: 30px;
  max-width: 500px;
  width: 100%;
  box-shadow: 0 20px 60px rgba(0, 0, 0, 0.3);
}

.modal-header {
  display: flex;
  flex-direction: column;
  align-items: center;
  margin-bottom: 24px;
}

.modal-icon {
  font-size: 2.5rem;
  margin-bottom: 12px;
}

.modal-header h3 {
  font-size: 1.4rem;
  color: #333;
  margin: 0;
}

.modal-form {
  display: flex;
  flex-direction: column;
  gap: 16px;
}

.form-group {
  display: flex;
  flex-direction: column;
  gap: 6px;
}

.form-group label {
  font-weight: 600;
  color: #555;
  font-size: 0.95rem;
}

.form-group input,
.form-group textarea {
  padding: 12px 16px;
  border: 2px solid #e0e0e0;
  border-radius: 10px;
  font-size: 1rem;
  transition: border-color 0.3s, box-shadow 0.3s;
  font-family: inherit;
}

.form-group.has-error input,
.form-group.has-error textarea {
  border-color: #ff6b6b;
}

.form-group.has-error input:focus,
.form-group.has-error textarea:focus {
  border-color: #ff6b6b;
  box-shadow: 0 0 0 3px rgba(255, 107, 107, 0.15);
}

.error-message {
  color: #ff6b6b;
  font-size: 0.85rem;
  margin-top: 4px;
}

.form-group input:focus,
.form-group textarea:focus {
  outline: none;
  border-color: #667eea;
  box-shadow: 0 0 0 3px rgba(102, 126, 234, 0.15);
}

.form-group input::placeholder,
.form-group textarea::placeholder {
  color: #aaa;
}

.form-group textarea {
  resize: vertical;
  min-height: 60px;
}

.modal-footer {
  display: flex;
  gap: 12px;
  justify-content: center;
  margin-top: 10px;
}

.btn {
  padding: 12px 28px;
  border: none;
  border-radius: 10px;
  font-size: 1rem;
  font-weight: 600;
  cursor: pointer;
  transition: transform 0.2s, box-shadow 0.2s;
}

.btn:hover {
  transform: translateY(-2px);
}

.btn:active {
  transform: translateY(0);
}

.btn-cancel {
  background: #f0f0f0;
  color: #666;
}

.btn-cancel:hover {
  background: #e0e0e0;
  box-shadow: 0 4px 12px rgba(0, 0, 0, 0.15);
}

.btn-confirm {
  background: linear-gradient(135deg, #667eea 0%, #764ba2 100%);
  color: white;
}

.btn-confirm:hover {
  box-shadow: 0 4px 16px rgba(102, 126, 234, 0.4);
}

@media (max-width: 640px) {
  .modal-container {
    padding: 20px;
  }
}
</style>