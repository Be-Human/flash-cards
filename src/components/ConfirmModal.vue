<template>
  <div v-if="visible" class="modal-overlay" @click.self="handleCancel">
    <div class="modal-container">
      <div class="modal-header">
        <span class="modal-icon">{{ displayIcon }}</span>
        <h3>{{ title }}</h3>
      </div>
      <div class="modal-body">
        <p v-for="(line, index) in messageLines" :key="index">{{ line }}</p>
      </div>
      <div class="modal-footer">
        <button class="btn btn-cancel" @click="handleCancel">
          {{ cancelText }}
        </button>
        <button 
          class="btn btn-confirm" 
          :class="{ danger: isDanger }" 
          @click="handleConfirm"
        >
          {{ confirmText }}
        </button>
      </div>
    </div>
  </div>
</template>

<script setup>
import { computed } from 'vue'

const props = defineProps({
  visible: {
    type: Boolean,
    default: false
  },
  title: {
    type: String,
    default: '确认'
  },
  message: {
    type: String,
    default: ''
  },
  icon: {
    type: String,
    default: ''
  },
  confirmText: {
    type: String,
    default: '确认'
  },
  cancelText: {
    type: String,
    default: '取消'
  },
  isDanger: {
    type: Boolean,
    default: false
  }
})

const emit = defineEmits(['confirm', 'cancel', 'update:visible'])

const displayIcon = computed(() => {
  return props.icon || '?'
})

const messageLines = computed(() => {
  return props.message.split('\n')
})

const handleConfirm = () => {
  emit('confirm')
  emit('update:visible', false)
}

const handleCancel = () => {
  emit('cancel')
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
  max-width: 450px;
  width: 100%;
  box-shadow: 0 20px 60px rgba(0, 0, 0, 0.3);
}

.modal-header {
  display: flex;
  flex-direction: column;
  align-items: center;
  margin-bottom: 20px;
}

.modal-icon {
  font-size: 3rem;
  margin-bottom: 12px;
}

.modal-header h3 {
  font-size: 1.4rem;
  color: #333;
  margin: 0;
}

.modal-body {
  text-align: center;
  margin-bottom: 28px;
}

.modal-body p {
  font-size: 1.05rem;
  color: #555;
  line-height: 1.6;
  margin: 8px 0;
}

.modal-footer {
  display: flex;
  gap: 12px;
  justify-content: center;
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

.btn-confirm.danger {
  background: linear-gradient(135deg, #ff6b6b 0%, #ee5a5a 100%);
}

.btn-confirm.danger:hover {
  box-shadow: 0 4px 16px rgba(255, 107, 107, 0.4);
}
</style>
