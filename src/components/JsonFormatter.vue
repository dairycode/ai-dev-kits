<template>
  <div class="json-formatter">
    <div class="input-section">
      <div class="input-group">
        <label for="json-input">输入JSON</label>
        <textarea 
          id="json-input"
          v-model="inputText"
          placeholder="请输入要格式化的JSON数据..."
          @keydown.ctrl.enter="formatJson"
          @keydown.meta.enter="formatJson"
          class="json-textarea"
          ref="inputArea"
          @input="autoResize()"
        ></textarea>
      </div>
      
      <div class="button-group">
        <button @click="formatJson" class="btn btn-primary">
          <span>格式化</span>
        </button>
        <button @click="compressJson" class="btn btn-secondary">
          <span>压缩</span>
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
  name: 'JsonFormatter',
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
    formatJson() {
      if (!this.inputText.trim()) {
        this.showMessage('请输入要格式化的JSON数据', 'error')
        return
      }
      
      try {
        // 先解析JSON确保格式正确
        const parsed = JSON.parse(this.inputText)
        // 格式化输出，使用2个空格缩进，直接覆盖输入框
        this.inputText = JSON.stringify(parsed, null, 2)
        this.showMessage('JSON格式化成功！', 'success')
      } catch (error) {
        this.showMessage('JSON格式错误：' + error.message, 'error')
      }
    },
    
    compressJson() {
      if (!this.inputText.trim()) {
        this.showMessage('请输入要压缩的JSON数据', 'error')
        return
      }
      
      try {
        // 先解析JSON确保格式正确
        const parsed = JSON.parse(this.inputText)
        // 压缩输出，无缩进，直接覆盖输入框
        this.inputText = JSON.stringify(parsed)
        this.showMessage('JSON压缩成功！', 'success')
      } catch (error) {
        this.showMessage('JSON格式错误：' + error.message, 'error')
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
.json-formatter {
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

.json-textarea {
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
  .json-formatter {
    gap: 24px;
    padding-top: 5vh;
  }
  
  .json-textarea {
    min-height: 100px !important;
  }
  
  .message {
    right: 10px;
    left: 10px;
    max-width: none;
  }
}
</style> 