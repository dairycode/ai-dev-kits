<template>
  <div class="base64-encoder">
    <div class="input-section">
      <div class="input-group">
        <label for="base64-input">输入文本</label>
        <textarea 
          id="base64-input"
          v-model="inputText"
          placeholder="请输入要编码或解码的文本..."
          @keydown.ctrl.enter="encodeBase64"
          @keydown.meta.enter="encodeBase64"
          class="large-textarea"
          ref="inputArea"
          @input="autoResize()"
        ></textarea>
      </div>
      
      <div class="button-group">
        <button @click="encodeBase64" class="btn btn-primary">
          <span>编码</span>
        </button>
        <button @click="decodeBase64" class="btn btn-secondary">
          <span>解码</span>
        </button>
        <button @click="clearAll" class="btn btn-clear">
          <span>清空</span>
        </button>
      </div>
    </div>
    
    <div v-if="message" :class="['message', messageType]">
      {{ message }}
    </div>
  </div>
</template>

<script>
export default {
  name: 'Base64Encoder',
  data() {
    return {
      inputText: '',
      message: '',
      messageType: 'success'
    }
  },
  watch: {
    inputText() {
      this.$nextTick(() => this.autoResize())
    }
  },
  mounted() {
    this.autoResize()
  },
  methods: {
    encodeBase64() {
      if (!this.inputText.trim()) {
        this.showMessage('请输入要编码的文本', 'error')
        return
      }
      
      try {
        const encoded = btoa(this.inputText)
        this.inputText = encoded
        this.showMessage('Base64编码成功！', 'success')
      } catch (error) {
        this.showMessage('编码失败：' + error.message, 'error')
      }
    },
    
    decodeBase64() {
      if (!this.inputText.trim()) {
        this.showMessage('请输入要解码的文本', 'error')
        return
      }
      
      try {
        // 检查输入是否为有效的Base64格式
        const base64Regex = /^[A-Za-z0-9+/]*={0,2}$/
        const cleanInput = this.inputText.trim()
        
        if (!base64Regex.test(cleanInput) || cleanInput.length % 4 !== 0) {
          this.showMessage('请输入有效的Base64编码文本', 'error')
          return
        }
        
        const decoded = atob(this.inputText)
        this.inputText = decoded
        this.showMessage('Base64解码成功！', 'success')
      } catch (error) {
        this.showMessage('解码失败：' + error.message, 'error')
      }
    },
    
    clearAll() {
      this.inputText = ''
      this.message = ''
      this.$nextTick(() => {
        this.autoResize()
      })
    },
    
    showMessage(text, type) {
      this.message = text
      this.messageType = type
      setTimeout(() => {
        this.message = ''
      }, 3000)
    },
    
    autoResize() {
      const area = this.$refs.inputArea
      if (area && area instanceof HTMLTextAreaElement) {
        area.style.height = 'auto'
        area.style.height = area.scrollHeight + 'px'
      }
    }
  }
}
</script>

<style scoped>
.base64-encoder {
  display: flex;
  flex-direction: column;
  gap: 32px;
  justify-content: flex-start;
  align-items: center;
  padding-top: 10vh;
  min-height: 100%;
  position: relative;
}

.input-section {
  display: flex;
  flex-direction: column;
  gap: 24px;
  width: 100%;
  max-width: 800px;
}

.large-textarea {
  min-height: 120px !important;
  font-family: 'Monaco', 'Menlo', 'Ubuntu Mono', monospace;
  font-size: 13px;
  line-height: 1.5;
  resize: none;
  overflow-y: hidden;
  transition: height 0.2s;
}

.message {
  position: fixed;
  top: 20px;
  right: 20px;
  z-index: 1000;
  max-width: 300px;
}

@media (max-width: 768px) {
  .base64-encoder {
    gap: 24px;
    padding-top: 5vh;
  }
  
  .large-textarea {
    min-height: 100px !important;
  }
  
  .message {
    right: 10px;
    left: 10px;
    max-width: none;
  }
}
</style>