<template>
  <div class="app">
    <div class="sidebar">
      <div class="sidebar-header">
        <h1 class="logo">🔧 开发工具包</h1>
        <p class="subtitle">实用的在线开发工具</p>
      </div>
      
      <nav class="sidebar-nav">
        <button 
          v-for="tab in tabs" 
          :key="tab.key"
          :class="['nav-item', { active: activeTab === tab.key }]"
          @click="activeTab = tab.key"
        >
          <span class="nav-icon">{{ tab.icon }}</span>
          <span class="nav-title">{{ tab.title }}</span>
        </button>
      </nav>
    </div>

    <div class="main-content">
      <div class="content-body">
        <component :is="currentTabComponent" />
      </div>
    </div>
  </div>
</template>

<script>
import UrlEncoder from './components/UrlEncoder.vue'
import TimestampConverter from './components/TimestampConverter.vue'
import QrCodeGenerator from './components/QrCodeGenerator.vue'
import JsonFormatter from './components/JsonFormatter.vue'
import Base64Encoder from './components/Base64Encoder.vue'
import HashEncoder from './components/HashEncoder.vue'

export default {
  name: 'App',
  components: {
    UrlEncoder,
    TimestampConverter,
    QrCodeGenerator,
    JsonFormatter,
    Base64Encoder,
    HashEncoder
  },
  data() {
    return {
      activeTab: 'url',
      tabs: [
        { 
          key: 'url', 
          title: 'URL编码/解码', 
          icon: '🔗', 
          component: 'UrlEncoder',
          description: 'URL编码和解码工具，支持特殊字符转换'
        },
        { 
          key: 'base64', 
          title: 'Base64编码/解码', 
          icon: '🔐', 
          component: 'Base64Encoder',
          description: 'Base64编码和解码工具'
        },
        {
          key: 'hash',
          title: 'Hash编码',
          icon: '🔒',
          component: 'HashEncoder',
          description: 'MD5、SHA等Hash计算工具'
        },
        { 
          key: 'timestamp', 
          title: '时间戳转换', 
          icon: '⏰', 
          component: 'TimestampConverter',
          description: 'Unix时间戳与日期时间互转工具'
        },
        { 
          key: 'qrcode', 
          title: '二维码生成', 
          icon: '📱', 
          component: 'QrCodeGenerator',
          description: '生成文本、URL的二维码图片'
        },
        { 
          key: 'json', 
          title: 'JSON格式化', 
          icon: '📄', 
          component: 'JsonFormatter',
          description: 'JSON格式化和美化工具'
        }
      ]
    }
  },
  computed: {
    currentTabComponent() {
      const tab = this.tabs.find(t => t.key === this.activeTab)
      return tab ? tab.component : 'UrlEncoder'
    },
    currentTabTitle() {
      const tab = this.tabs.find(t => t.key === this.activeTab)
      return tab ? tab.title : ''
    },
    currentTabDescription() {
      const tab = this.tabs.find(t => t.key === this.activeTab)
      return tab ? tab.description : ''
    }
  }
}
</script>

<style scoped>
.app {
  display: flex;
  height: 100vh;
  background: white;
}

.sidebar {
  width: 280px;
  background: #f8f9fa;
  border-right: 1px solid #e9ecef;
  display: flex;
  flex-direction: column;
}

.sidebar-header {
  padding: 24px 20px;
  border-bottom: 1px solid #e9ecef;
}

.logo {
  font-size: 20px;
  font-weight: 600;
  color: #1a1a1a;
  margin: 0 0 4px 0;
}

.subtitle {
  font-size: 14px;
  color: #6c757d;
  margin: 0;
}

.sidebar-nav {
  flex: 1;
  padding: 16px 0;
}

.nav-item {
  width: 100%;
  padding: 12px 20px;
  background: none;
  border: none;
  text-align: left;
  cursor: pointer;
  display: flex;
  align-items: center;
  gap: 12px;
  transition: all 0.2s ease;
  color: #6c757d;
  font-size: 14px;
  font-weight: 500;
}

.nav-item:hover {
  background: #f8f9fa;
  color: #495057;
}

.nav-item.active {
  background: #e3f2fd;
  color: #1976d2;
  border-right: 3px solid #1976d2;
}

.nav-icon {
  font-size: 18px;
  width: 24px;
  text-align: center;
}

.nav-title {
  flex: 1;
}

.main-content {
  flex: 1;
  display: flex;
  flex-direction: column;
  background: white;
}

.content-body {
  flex: 1;
  padding: 32px;
  overflow-y: auto;
}

@media (max-width: 768px) {
  .app {
    flex-direction: column;
    height: auto;
  }
  
  .sidebar {
    width: 100%;
    border-right: none;
    border-bottom: 1px solid #e9ecef;
  }
  
  .sidebar-nav {
    display: flex;
    overflow-x: auto;
    padding: 12px 16px;
  }
  
  .nav-item {
    flex-shrink: 0;
    padding: 8px 16px;
    border-radius: 8px;
    white-space: nowrap;
  }
  
  .nav-item.active {
    border-right: none;
    background: #e3f2fd;
  }
  
  .main-content {
    margin: 0;
    border-radius: 0;
  }
  
  .content-body {
    padding: 24px;
  }
}
</style>