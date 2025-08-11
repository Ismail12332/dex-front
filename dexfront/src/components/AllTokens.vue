<template>
  <div class="all-tokens">
    <div class="section-header">
      <div class="tabs">
        <button 
          :class="['tab', { active: activeTab === 'explore' }]"
          @click="setActiveTab('explore')"
        >
          explore
        </button>
        <button 
          :class="['tab', { active: activeTab === 'watchlist' }]"
          @click="setActiveTab('watchlist')"
        >
          watchlist
        </button>
      </div>
      
      <div class="filters">
        <div class="filter-group">
          <label class="filter-label">sort: newest</label>
          <select class="filter-select" v-model="sortBy" @change="sortTokens">
            <option value="newest">newest</option>
            <option value="oldest">oldest</option>
            <option value="name">name</option>
          </select>
        </div>
        
        <div class="filter-group">
          <label class="filter-label">status:</label>
          <select class="filter-select" v-model="statusFilter" @change="filterTokens">
            <option value="all">all</option>
            <option value="created">created</option>
            <option value="pending">pending</option>
          </select>
        </div>
      </div>
    </div>
    
    <div class="tokens-grid">
      <div v-if="filteredTokens.length === 0" class="no-tokens">
        <p>No tokens available</p>
      </div>
      <div 
        v-for="token in filteredTokens" 
        :key="token.id"
        class="token-card"
        @click="selectToken(token)"
      >
        <div class="token-image">
          <img :src="token.image" :alt="token.name" />
          <!--<div v-if="token.isLive" class="live-badge">LIVE</div>
          <div v-if="token.status === 'pending'" class="pending-badge">PENDING</div>-->
        </div>
        
        <div class="token-content">
          <div class="token-meta">
            <span class="created-by">created by</span>
            <span class="creator">{{ token.creator }}</span>
            <span class="time-ago">{{ token.timeAgo }}</span>
          </div>
          
          <div class="token-stats-row">
            <span class="market-cap">market cap: {{ token.marketCap }}</span>
            <span class="replies">replies: {{ token.replies }}</span>
          </div>
          
          <h3 class="token-title">{{ token.name }} ({{ token.ticker }}):</h3>
          <p class="token-description">{{ token.description }}</p>
          
          <!--<div class="token-status">
            <span :class="['status-badge', token.status]">{{ token.status }}</span>
          </div>-->
        </div>
      </div>
    </div>
  </div>
</template>

<script setup>
import { ref, computed } from 'vue'

const props = defineProps({
  tokens: {
    type: Array,
    default: () => []
  }
})

const activeTab = ref('explore')
const sortBy = ref('newest')
const statusFilter = ref('all')

const emit = defineEmits(['token-selected'])

const filteredTokens = computed(() => {
  let filtered = [...props.tokens]
  
  // Filter by status
  if (statusFilter.value !== 'all') {
    filtered = filtered.filter(token => token.status === statusFilter.value)
  }
  
  // Sort tokens
  filtered.sort((a, b) => {
    switch (sortBy.value) {
      case 'newest':
        return new Date(b.createdAt || 0) - new Date(a.createdAt || 0)
      case 'oldest':
        return new Date(a.createdAt || 0) - new Date(b.createdAt || 0)
      case 'name':
        return a.name.localeCompare(b.name)
      default:
        return 0
    }
  })
  
  return filtered
})

const setActiveTab = (tab) => {
  activeTab.value = tab
}

const selectToken = (token) => {
  console.log('Selected token:', token)
  emit('token-selected', token)
}

const sortTokens = () => {
  // Sorting is handled by computed property
}

const filterTokens = () => {
  // Filtering is handled by computed property
}
</script>

<style scoped>
.all-tokens {
  margin-bottom: 48px;
}

.section-header {
  display: flex;
  align-items: center;
  justify-content: space-between;
  margin-bottom: 32px;
  flex-wrap: wrap;
  gap: 20px;
}

.tabs {
  display: flex;
  gap: 2px;
  background: rgba(255, 255, 255, 0.05);
  border-radius: 8px;
  padding: 4px;
}

.tab {
  padding: 8px 16px;
  background: transparent;
  border: none;
  border-radius: 6px;
  color: #9ca3af;
  font-size: 14px;
  font-weight: 500;
  cursor: pointer;
  transition: all 0.2s ease;
  text-transform: lowercase;
  white-space: nowrap;
}

.tab.active {
  background: #42b883;
  color: #ffffff;
}

.filters {
  display: flex;
  align-items: center;
  gap: 24px;
  flex-wrap: wrap;
}

.filter-group {
  display: flex;
  align-items: center;
  gap: 8px;
  flex-wrap: wrap;
}

.filter-label {
  font-size: 14px;
  color: #9ca3af;
  font-weight: 500;
  white-space: nowrap;
}

.filter-select {
  background: rgba(255, 255, 255, 0.05);
  border: 1px solid rgba(255, 255, 255, 0.1);
  border-radius: 6px;
  color: #ffffff;
  padding: 6px 12px;
  font-size: 14px;
  cursor: pointer;
}

.tokens-grid {
  display: grid;
  grid-template-columns: repeat(auto-fill, minmax(320px, 1fr));
  gap: 20px;
}

.no-tokens {
  grid-column: 1 / -1;
  display: flex;
  align-items: center;
  justify-content: center;
  color: #9ca3af;
  font-size: 16px;
  padding: 80px 20px;
  background: rgba(255, 255, 255, 0.05);
  border: 1px solid rgba(255, 255, 255, 0.1);
  border-radius: 16px;
}

.token-card {
  background: rgba(255, 255, 255, 0.05);
  backdrop-filter: blur(10px);
  border: 1px solid rgba(255, 255, 255, 0.1);
  border-radius: 12px;
  overflow: hidden;
  cursor: pointer;
  transition: all 0.3s ease;
}

.token-card:hover {
  transform: translateY(-4px);
  background: rgba(255, 255, 255, 0.08);
  border-color: rgba(100, 108, 255, 0.3);
  box-shadow: 0 12px 24px rgba(100, 108, 255, 0.1);
}

.token-image {
  position: relative;
  width: 100%;
  height: 200px;
  overflow: hidden;
}

.token-image img {
  width: 100%;
  height: 100%;
  object-fit: cover;
}

.live-badge {
  position: absolute;
  top: 12px;
  right: 12px;
  background: #ef4444;
  color: #ffffff;
  padding: 4px 8px;
  border-radius: 4px;
  font-size: 12px;
  font-weight: 600;
}

.pending-badge {
  position: absolute;
  top: 12px;
  right: 12px;
  background: #f59e0b;
  color: #ffffff;
  padding: 4px 8px;
  border-radius: 4px;
  font-size: 12px;
  font-weight: 600;
}

.token-content {
  padding: 16px;
}

.token-meta {
  display: flex;
  align-items: center;
  gap: 6px;
  margin-bottom: 8px;
  font-size: 12px;
  flex-wrap: wrap;
}

.created-by {
  color: #6b7280;
}

.creator {
  color: #42b883;
  font-weight: 600;
}

.time-ago {
  color: #6b7280;
}

.token-stats-row {
  display: flex;
  gap: 16px;
  margin-bottom: 12px;
  font-size: 12px;
  flex-wrap: wrap;
}

.market-cap,
.replies {
  color: #9ca3af;
}

.token-title {
  font-size: 16px;
  font-weight: 600;
  color: #ffffff;
  margin-bottom: 8px;
  line-height: 1.3;
}

.token-description {
  font-size: 14px;
  color: #9ca3af;
  line-height: 1.5;
  display: -webkit-box;
  -webkit-line-clamp: 3;
  -webkit-box-orient: vertical;
  overflow: hidden;
  margin-bottom: 12px;
}

.token-status {
  display: flex;
  justify-content: flex-end;
}

.status-badge {
  padding: 4px 8px;
  border-radius: 4px;
  font-size: 12px;
  font-weight: 600;
  text-transform: uppercase;
}

.status-badge.created {
  background: #10b981;
  color: #ffffff;
}

.status-badge.pending {
  background: #f59e0b;
  color: #ffffff;
}

/* Tablet Styles */
@media (max-width: 1024px) {
  .all-tokens {
    margin-bottom: 40px;
  }
  
  .section-header {
    margin-bottom: 28px;
    gap: 16px;
  }
  
  .filters {
    gap: 20px;
  }
  
  .tokens-grid {
    grid-template-columns: repeat(auto-fill, minmax(300px, 1fr));
    gap: 18px;
  }
  
  .token-image {
    height: 180px;
  }
}

/* Mobile Styles */
@media (max-width: 768px) {
  .all-tokens {
    margin-bottom: 32px;
  }
  
  .section-header {
    flex-direction: column;
    align-items: flex-start;
    margin-bottom: 24px;
    gap: 16px;
    padding: 0 4px;
  }
  
  .filters {
    flex-direction: column;
    align-items: flex-start;
    gap: 12px;
    width: 100%;
  }
  
  .filter-group {
    gap: 6px;
  }
  
  .tokens-grid {
    grid-template-columns: 1fr;
    gap: 16px;
    padding: 0 4px;
  }
  
  .token-image {
    height: 160px;
  }
  
  .token-content {
    padding: 14px;
  }
  
  .token-meta {
    gap: 4px;
    font-size: 11px;
  }
  
  .token-stats-row {
    gap: 12px;
    font-size: 11px;
  }
  
  .token-title {
    font-size: 15px;
  }
  
  .token-description {
    font-size: 13px;
  }
}

/* Small Mobile Styles */
@media (max-width: 480px) {
  .section-header {
    padding: 0 2px;
    gap: 12px;
  }
  
  .tab {
    padding: 6px 12px;
    font-size: 13px;
  }
  
  .filter-label {
    font-size: 13px;
  }
  
  .filter-select {
    padding: 5px 10px;
    font-size: 13px;
  }
  
  .toggle-btn {
    padding: 3px 6px;
    font-size: 11px;
  }
  
  .tokens-grid {
    padding: 0 2px;
    gap: 14px;
  }
  
  .token-image {
    height: 140px;
  }
  
  .token-content {
    padding: 12px;
  }
  
  .token-title {
    font-size: 14px;
  }
  
  .token-description {
    font-size: 12px;
  }
}

/* Very Small Mobile */
@media (max-width: 360px) {
  .token-image {
    height: 120px;
  }
  
  .token-content {
    padding: 10px;
  }
  
  .token-meta,
  .token-stats-row {
    font-size: 10px;
  }
  
  .token-title {
    font-size: 13px;
  }
  
  .token-description {
    font-size: 11px;
  }
}
</style>