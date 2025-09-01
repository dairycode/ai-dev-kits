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
            result = await this.calculateMD5(text)
            break
          case 'sha1':
            result = await this.calculateSHA1(text)
            break
          case 'sha256':
            result = await this.calculateSHA256(text)
            break
          case 'sha512':
            result = await this.calculateSHA512(text)
            break
          default:
            throw new Error('不支持的哈希算法')
        }

        this.hashResult = result
      } catch (error) {
        this.showMessage('Hash计算失败：' + error.message, 'error')
      }
    },

    async calculateMD5(text) {
      // 使用Web Crypto API计算MD5
      const encoder = new TextEncoder()
      const data = encoder.encode(text)
      
      // 使用标准MD5算法
      const md5Hash = await this.md5(data)
      return md5Hash
    },

    async calculateSHA1(text) {
      const encoder = new TextEncoder()
      const data = encoder.encode(text)
      const hashBuffer = await crypto.subtle.digest('SHA-1', data)
      return this.arrayBufferToHex(hashBuffer)
    },

    async calculateSHA256(text) {
      const encoder = new TextEncoder()
      const data = encoder.encode(text)
      const hashBuffer = await crypto.subtle.digest('SHA-256', data)
      return this.arrayBufferToHex(hashBuffer)
    },

    async calculateSHA512(text) {
      const encoder = new TextEncoder()
      const data = encoder.encode(text)
      const hashBuffer = await crypto.subtle.digest('SHA-512', data)
      return this.arrayBufferToHex(hashBuffer)
    },

    async md5(data) {
      // 使用第三方MD5库或标准实现
      // 这里使用基于Web Crypto API的简化实现
      const encoder = new TextEncoder()
      const text = typeof data === 'string' ? data : new TextDecoder().decode(data)
      
      // 使用标准MD5算法实现
      return this.calculateMD5Classic(text)
    },
    
    calculateMD5Classic(str) {
      // 标准MD5算法实现
      function md5cycle(x, k) {
        let a = x[0], b = x[1], c = x[2], d = x[3]
        
        const ff = (a, b, c, d, x, s, t) => {
          a = (a + ((b & c) | (~b & d)) + x + t) >>> 0
          return ((a << s) | (a >>> (32 - s))) + b >>> 0
        }
        
        const gg = (a, b, c, d, x, s, t) => {
          a = (a + ((b & d) | (c & ~d)) + x + t) >>> 0
          return ((a << s) | (a >>> (32 - s))) + b >>> 0
        }
        
        const hh = (a, b, c, d, x, s, t) => {
          a = (a + (b ^ c ^ d) + x + t) >>> 0
          return ((a << s) | (a >>> (32 - s))) + b >>> 0
        }
        
        const ii = (a, b, c, d, x, s, t) => {
          a = (a + (c ^ (b | ~d)) + x + t) >>> 0
          return ((a << s) | (a >>> (32 - s))) + b >>> 0
        }
        
        const T = [
          0xd76aa478, 0xe8c7b756, 0x242070db, 0xc1bdceee, 0xf57c0faf, 0x4787c62a, 0xa8304613, 0xfd469501,
          0x698098d8, 0x8b44f7af, 0xffff5bb1, 0x895cd7be, 0x6b901122, 0xfd987193, 0xa679438e, 0x49b40821,
          0xf61e2562, 0xc040b340, 0x265e5a51, 0xe9b6c7aa, 0xd62f105d, 0x02441453, 0xd8a1e681, 0xe7d3fbc8,
          0x21e1cde6, 0xc33707d6, 0xf4d50d87, 0x455a14ed, 0xa9e3e905, 0xfcefa3f8, 0x676f02d9, 0x8d2a4c8a,
          0xfffa3942, 0x8771f681, 0x6d9d6122, 0xfde5380c, 0xa4beea44, 0x4bdecfa9, 0xf6bb4b60, 0xbebfbc70,
          0x289b7ec6, 0xeaa127fa, 0xd4ef3085, 0x04881d05, 0xd9d4d039, 0xe6db99e5, 0x1fa27cf8, 0xc4ac5665,
          0xf4292244, 0x432aff97, 0xab9423a7, 0xfc93a039, 0x655b59c3, 0x8f0ccc92, 0xffeff47d, 0x85845dd1,
          0x6fa87e4f, 0xfe2ce6e0, 0xa3014314, 0x4e0811a1, 0xf7537e82, 0xbd3af235, 0x2ad7d2bb, 0xeb86d391
        ]
        
        a = ff(a, b, c, d, k[0], 7, T[0])
        d = ff(d, a, b, c, k[1], 12, T[1])
        c = ff(c, d, a, b, k[2], 17, T[2])
        b = ff(b, c, d, a, k[3], 22, T[3])
        a = ff(a, b, c, d, k[4], 7, T[4])
        d = ff(d, a, b, c, k[5], 12, T[5])
        c = ff(c, d, a, b, k[6], 17, T[6])
        b = ff(b, c, d, a, k[7], 22, T[7])
        a = ff(a, b, c, d, k[8], 7, T[8])
        d = ff(d, a, b, c, k[9], 12, T[9])
        c = ff(c, d, a, b, k[10], 17, T[10])
        b = ff(b, c, d, a, k[11], 22, T[11])
        a = ff(a, b, c, d, k[12], 7, T[12])
        d = ff(d, a, b, c, k[13], 12, T[13])
        c = ff(c, d, a, b, k[14], 17, T[14])
        b = ff(b, c, d, a, k[15], 22, T[15])
        
        a = gg(a, b, c, d, k[1], 5, T[16])
        d = gg(d, a, b, c, k[6], 9, T[17])
        c = gg(c, d, a, b, k[11], 14, T[18])
        b = gg(b, c, d, a, k[0], 20, T[19])
        a = gg(a, b, c, d, k[5], 5, T[20])
        d = gg(d, a, b, c, k[10], 9, T[21])
        c = gg(c, d, a, b, k[15], 14, T[22])
        b = gg(b, c, d, a, k[4], 20, T[23])
        a = gg(a, b, c, d, k[9], 5, T[24])
        d = gg(d, a, b, c, k[14], 9, T[25])
        c = gg(c, d, a, b, k[3], 14, T[26])
        b = gg(b, c, d, a, k[8], 20, T[27])
        a = gg(a, b, c, d, k[13], 5, T[28])
        d = gg(d, a, b, c, k[2], 9, T[29])
        c = gg(c, d, a, b, k[7], 14, T[30])
        b = gg(b, c, d, a, k[12], 20, T[31])
        
        a = hh(a, b, c, d, k[5], 4, T[32])
        d = hh(d, a, b, c, k[8], 11, T[33])
        c = hh(c, d, a, b, k[11], 16, T[34])
        b = hh(b, c, d, a, k[14], 23, T[35])
        a = hh(a, b, c, d, k[1], 4, T[36])
        d = hh(d, a, b, c, k[4], 11, T[37])
        c = hh(c, d, a, b, k[7], 16, T[38])
        b = hh(b, c, d, a, k[10], 23, T[39])
        a = hh(a, b, c, d, k[13], 4, T[40])
        d = hh(d, a, b, c, k[0], 11, T[41])
        c = hh(c, d, a, b, k[3], 16, T[42])
        b = hh(b, c, d, a, k[6], 23, T[43])
        a = hh(a, b, c, d, k[9], 4, T[44])
        d = hh(d, a, b, c, k[12], 11, T[45])
        c = hh(c, d, a, b, k[15], 16, T[46])
        b = hh(b, c, d, a, k[2], 23, T[47])
        
        a = ii(a, b, c, d, k[0], 6, T[48])
        d = ii(d, a, b, c, k[7], 10, T[49])
        c = ii(c, d, a, b, k[14], 15, T[50])
        b = ii(b, c, d, a, k[5], 21, T[51])
        a = ii(a, b, c, d, k[12], 6, T[52])
        d = ii(d, a, b, c, k[3], 10, T[53])
        c = ii(c, d, a, b, k[10], 15, T[54])
        b = ii(b, c, d, a, k[1], 21, T[55])
        a = ii(a, b, c, d, k[8], 6, T[56])
        d = ii(d, a, b, c, k[15], 10, T[57])
        c = ii(c, d, a, b, k[6], 15, T[58])
        b = ii(b, c, d, a, k[13], 21, T[59])
        a = ii(a, b, c, d, k[4], 6, T[60])
        d = ii(d, a, b, c, k[11], 10, T[61])
        c = ii(c, d, a, b, k[2], 15, T[62])
        b = ii(b, c, d, a, k[9], 21, T[63])
        
        x[0] = (x[0] + a) >>> 0
        x[1] = (x[1] + b) >>> 0
        x[2] = (x[2] + c) >>> 0
        x[3] = (x[3] + d) >>> 0
      }
      
      function str2binl(str) {
        const bin = []
        const mask = (1 << 8) - 1
        for (let i = 0; i < str.length * 8; i += 8) {
          bin[i >> 5] |= (str.charCodeAt(i / 8) & mask) << (i % 32)
        }
        return bin
      }
      
      function binl2hex(binarray) {
        const hex_tab = '0123456789abcdef'
        let str = ''
        for (let i = 0; i < binarray.length * 4; i++) {
          str += hex_tab.charAt((binarray[i >> 2] >> ((i % 4) * 8 + 4)) & 0xF) +
                 hex_tab.charAt((binarray[i >> 2] >> ((i % 4) * 8)) & 0xF)
        }
        return str
      }
      
      const x = str2binl(str)
      const len = str.length * 8
      
      x[len >> 5] |= 0x80 << ((len) % 32)
      x[(((len + 64) >>> 9) << 4) + 14] = len
      
      let a = 1732584193
      let b = -271733879
      let c = -1732584194
      let d = 271733878
      
      for (let i = 0; i < x.length; i += 16) {
        const olda = a
        const oldb = b
        const oldc = c
        const oldd = d
        
        const w = [a, b, c, d]
        md5cycle(w, x.slice(i, i + 16))
        a = w[0]
        b = w[1]
        c = w[2]
        d = w[3]
      }
      
      return binl2hex([a, b, c, d])
    },

    arrayBufferToHex(buffer) {
      const bytes = new Uint8Array(buffer)
      return Array.from(bytes)
        .map(b => b.toString(16).padStart(2, '0'))
        .join('')
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