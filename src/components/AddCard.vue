<template>
  <div class="add-card">
    <div class="form-container">
      <h2>➕ 添加新单词卡片</h2>
      <form @submit.prevent="handleSubmit">
        <div class="form-group">
          <label for="word">单词 *</label>
          <input 
            type="text" 
            id="word" 
            v-model="formData.word"
            placeholder="请输入单词"
            required
          />
        </div>
        
        <div class="form-group">
          <label for="meaning">释义 *</label>
          <input 
            type="text" 
            id="meaning" 
            v-model="formData.meaning"
            placeholder="请输入单词释义"
            required
          />
        </div>
        
        <div class="form-group">
          <label for="example">例句</label>
          <textarea 
            id="example" 
            v-model="formData.example"
            placeholder="请输入例句（可选）"
            rows="2"
          ></textarea>
        </div>
        
        <button type="submit" class="submit-btn">
          添加卡片
        </button>
      </form>
    </div>
  </div>
</template>

<script setup>
import { reactive, ref } from 'vue'

const emit = defineEmits(['add-card'])

const formData = reactive({
  word: '',
  meaning: '',
  example: ''
})

const handleSubmit = () => {
  if (!formData.word.trim() || !formData.meaning.trim()) {
    alert('单词和释义为必填项！')
    return
  }
  
  emit('add-card', {
    word: formData.word.trim(),
    meaning: formData.meaning.trim(),
    example: formData.example.trim()
  })
  
  formData.word = ''
  formData.meaning = ''
  formData.example = ''
}
</script>

<style scoped>
.add-card {
  display: flex;
  justify-content: center;
}

.form-container {
  background: white;
  border-radius: 20px;
  padding: 30px;
  width: 100%;
  max-width: 500px;
  box-shadow: 0 15px 40px rgba(0, 0, 0, 0.2);
}

.form-container h2 {
  text-align: center;
  color: #333;
  margin-bottom: 24px;
  font-size: 1.4rem;
}

form {
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

.submit-btn {
  margin-top: 10px;
  padding: 14px 24px;
  background: linear-gradient(135deg, #667eea 0%, #764ba2 100%);
  color: white;
  border: none;
  border-radius: 10px;
  font-size: 1.1rem;
  font-weight: 600;
  cursor: pointer;
  transition: transform 0.2s, box-shadow 0.2s;
}

.submit-btn:hover {
  transform: translateY(-2px);
  box-shadow: 0 5px 20px rgba(102, 126, 234, 0.4);
}

.submit-btn:active {
  transform: translateY(0);
}

@media (max-width: 640px) {
  .form-container {
    padding: 20px;
    margin: 0 10px;
  }
}
</style>
