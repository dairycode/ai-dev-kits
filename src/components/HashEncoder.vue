<template>
  <div class="hash-encoder">
    <div class="input-section">
      <h2>Hash编码工具</h2>
      
      <div class="form-group">
        <label for="input-text">输入文本：</label>
        <textarea
          id="input-text"
          v-model="inputText"
          @input="autoResize"
          @keydown.ctrl.enter="calculateHash"
          @keydown.meta.enter="calculateHash"
          placeholder="请输入要计算Hash的文本..."
          class="large-textarea"
          ref="inputArea"
        ></textarea>
      </div>
      
      <div class="form-group">
        <label for="hash-type">选择Hash算法：</label>
        <select 
          id="hash-type" 
          v-model="selectedHashType" 
          class="hash-select"
        >
          <option value="md5">MD5</option>
          <option value="sha1">SHA-1</option>
          <option value="sha256">SHA-256</option>
          <option value="sha512">SHA-512</option>
        </select>
      </div>
      
      <div class="button-group">
        <button @click="calculateHash" class="btn btn-primary">
          计算Hash
        </button>
        <button @click="clearAll" class="btn btn-secondary">
          清空
        </button>
      </div>
    </div>
    
    <div class="result-section" v-if="hashResult">
      <div class="form-group">
        <label>Hash结果：</label>
        <div class="result-display">
          <div class="hash-result">{{ hashResult }}</div>
        </div>
        <button @click="copyResult" class="btn btn-copy">
          复制结果
        </button>
      </div>
    </div>
    
    <div class="message" v-if="message" :class="messageType">
      {{ message }}
    </div>
  </div>
</template>

<script>
import CryptoJS from 'crypto-js'

export default {
  name: 'HashEncoder',
  data() {
    return {
      inputText: '',
      hashResult: '',
      selectedHashType: 'md5',
      message: '',
      messageType: ''
    }
  },
  watch: {
    inputText(newVal) {
      if (newVal.trim()) {
        this.calculateHash()
      } else {
        this.hashResult = ''
      }
    },
    selectedHashType() {
      if (this.inputText.trim()) {
        this.calculateHash()
      }
    }
  },
  methods: {
    async calculateHash() {
      if (!this.inputText.trim()) {
        this.hashResult = ''
        return
      }

      try {
        const text = this.inputText.trim()
        let result = ''

        switch (this.selectedHashType) {
          case 'md5':
            result = CryptoJS.MD5(text).toString()
            break
          case 'sha1':
            result = CryptoJS.SHA1(text).toString()
            break
          case 'sha256':
            result = CryptoJS.SHA256(text).toString()
            break
          case 'sha512':
            result = CryptoJS.SHA512(text).toString()
            break
          default:
            throw new Error('不支持的哈希算法')
        }

        this.hashResult = result
      } catch (error) {
        this.showMessage('Hash计算失败：' + error.message, 'error')
      }
    },



    async copyResult() {
      if (!this.hashResult) return
      
      try {
        await navigator.clipboard.writeText(this.hashResult)
        this.showMessage('Hash值已复制到剪贴板！', 'success')
      } catch (error) {
        const textArea = document.createElement('textarea')
        textArea.value = this.hashResult
        document.body.appendChild(textArea)
        textArea.select()
        document.execCommand('copy')
        document.body.removeChild(textArea)
        this.showMessage('Hash值已复制到剪贴板！', 'success')
      }
    },

    clearAll() {
      this.inputText = ''
      this.hashResult = ''
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
.hash-encoder {
  display: flex;
  flex-direction: column;
  gap: 32px;
  justify-content: flex-start;
  align-items: center;
  padding-top: 10vh;
  min-height: 100%;
  position: relative;
  width: 100%;
  max-width: 800px;
  margin: 0 auto;
}

.hash-select {
  background-image: url("data:image/svg+xml,%3Csvg xmlns='http://www.w3.org/2000/svg' width='14' height='14' viewBox='0 0 24 24' fill='none' stroke='%23495057' stroke-width='2' stroke-linecap='round' stroke-linejoin='round'%3E%3Cpolyline points='6 9 12 15 18 9'%3E%3C/polyline%3E%3C/svg%3E");
  background-repeat: no-repeat;
  background-position: right 12px center;
  background-size: 14px;
  cursor: pointer;
  transition: all 0.3s ease;
  border: 1.5px solid #e9ecef;
  border-radius: 8px;
  padding: 10px 36px 10px 12px;
  font-size: 14px;
  font-weight: 500;
  color: #495057;
  background-color: #ffffff;
  box-shadow: 0 1px 3px rgba(0, 0, 0, 0.08);
  appearance: none;
  -webkit-appearance: none;
  -moz-appearance: none;
  min-width: 180px;
}

.hash-select:hover {
  border-color: #007bff;
  box-shadow: 0 2px 8px rgba(0, 123, 255, 0.12);
  transform: translateY(-0.5px);
}

.hash-select:focus {
  outline: none;
  border-color: #007bff;
  box-shadow: 0 0 0 2px rgba(0, 123, 255, 0.2), 0 2px 8px rgba(0, 123, 255, 0.12);
  transform: translateY(-0.5px);
}

.input-section, .result-section {
  width: 100%;
  display: flex;
  flex-direction: column;
  gap: 24px;
}

.form-group {
  display: flex;
  flex-direction: column;
  gap: 8px;
}

.form-group label {
  font-weight: 600;
  color: #495057;
  font-size: 14px;
}

.large-textarea {
  min-height: 120px !important;
  font-family: 'Monaco', 'Menlo', 'Ubuntu Mono', monospace;
  font-size: 13px;
  line-height: 1.5;
  resize: none;
  overflow-y: hidden;
  transition: height 0.2s;
  border: 1.5px solid #e9ecef;
  border-radius: 8px;
  padding: 12px;
  background-color: #ffffff;
  color: #495057;
}

.large-textarea:focus {
  outline: none;
  border-color: #007bff;
  box-shadow: 0 0 0 2px rgba(0, 123, 255, 0.2);
}

.button-group {
  display: flex;
  gap: 12px;
  flex-wrap: wrap;
}

.btn {
  padding: 10px 20px;
  border: none;
  border-radius: 8px;
  font-size: 14px;
  font-weight: 500;
  cursor: pointer;
  transition: all 0.3s ease;
  min-width: 100px;
}

.btn-primary {
  background-color: #007bff;
  color: white;
}

.btn-primary:hover {
  background-color: #0056b3;
  transform: translateY(-1px);
}

.btn-secondary {
  background-color: #6c757d;
  color: white;
}

.btn-secondary:hover {
  background-color: #545b62;
  transform: translateY(-1px);
}

.btn-copy {
  background-color: #28a745;
  color: white;
}

.btn-copy:hover {
  background-color: #1e7e34;
  transform: translateY(-1px);
}

.result-display {
  background: #f8f9fa;
  padding: 16px;
  border-radius: 8px;
  border: 1px solid #dee2e6;
}

.hash-result {
  font-family: 'Monaco', 'Menlo', 'Ubuntu Mono', monospace;
  font-size: 14px;
  word-break: break-all;
  color: #495057;
  background: transparent;
  padding: 0;
  border: none;
  line-height: 1.5;
}

.message {
  position: fixed;
  top: 20px;
  right: 20px;
  z-index: 1000;
  max-width: 300px;
  padding: 12px 16px;
  border-radius: 8px;
  font-size: 14px;
  font-weight: 500;
  box-shadow: 0 4px 12px rgba(0, 0, 0, 0.15);
  animation: slideIn 0.3s ease;
}

.message.success {
  background-color: #d4edda;
  color: #155724;
  border: 1px solid #c3e6cb;
}

.message.error {
  background-color: #f8d7da;
  color: #721c24;
  border: 1px solid #f5c6cb;
}

@keyframes slideIn {
  from {
    transform: translateX(100%);
    opacity: 0;
  }
  to {
    transform: translateX(0);
    opacity: 1;
  }
}

@media (max-width: 768px) {
  .hash-encoder {
    gap: 24px;
    padding-top: 5vh;
    padding-left: 16px;
    padding-right: 16px;
  }
  
  .large-textarea {
    min-height: 100px !important;
  }
  
  .button-group {
    flex-direction: column;
  }
  
  .btn {
    width: 100%;
  }
  
  .message {
    right: 10px;
    left: 10px;
    max-width: none;
  }
}
</style>