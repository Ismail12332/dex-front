<template>
  <div class="dashboard">
    <!-- Token Search -->
    <TokenSearch @token-selected="navigateToTokenChart" :tokens="tokens" />
    
    <!-- Now Trending Section -->
    <TrendingTokens @token-selected="navigateToTokenChart" :tokens="trendingTokens" />
    
    <!-- All Tokens Section -->
    <AllTokens @token-selected="navigateToTokenChart" :tokens="allTokens" />
  </div>
</template>

<script setup>
import { ref, onMounted } from 'vue'
import { useRouter } from 'vue-router'
import TokenSearch from './TokenSearch.vue'
import TrendingTokens from './TrendingTokens.vue'
import AllTokens from './AllTokens.vue'

const router = useRouter()

// Loading states
const isLoading = ref(true)
const error = ref(null)

// Token data
const tokens = ref([])
const trendingTokens = ref([])
const allTokens = ref([])

// Load tokens from API
const loadTokens = async () => {
  try {
    isLoading.value = true
    error.value = null

    const response = await fetch('http://localhost:3000/api/tokens')
    
    if (!response.ok) {
      throw new Error(`Failed to load tokens: ${response.status}`)
    }

    const data = await response.json()
    
    if (data.success && data.tokens) {
      const apiTokens = data.tokens.map(token => ({
        id: token.id,
        name: token.name,
        ticker: token.symbol,
        symbol: token.symbol,
        description: token.description,
        image: token.image_url && token.image_url !== 'placeholder' 
          ? token.image_url 
          : 'https://images.pexels.com/photos/730547/pexels-photo-730547.jpeg?auto=compress&cs=tinysrgb&w=100&h=100&dpr=1',
        marketCap: generateMarketCap(),
        replies: Math.floor(Math.random() * 500),
        creator: token.user_pubkey ? token.user_pubkey.slice(0, 6) : 'Unknown',
        timeAgo: formatTimeAgo(token.createdAt),
        status: token.status,
        mintAddress: token.mint_address,
        isLive: token.status === 'created' && token.mint_address
      }))

      tokens.value = apiTokens
      
      // Разделяем токены: trending - это токены со статусом 'created' и mint_address
      trendingTokens.value = apiTokens
        .filter(token => token.status === 'created' && token.mintAddress)
        .slice(0, 6)
      
      // Все остальные токены
      allTokens.value = apiTokens
    }
  } catch (err) {
    console.error('Failed to load tokens:', err)
    error.value = err.message
  } finally {
    isLoading.value = false
  }
}

// Helper functions
const generateMarketCap = () => {
  const value = Math.random() * 50000000 // До 50M
  if (value > 1000000) {
    return `$${(value / 1000000).toFixed(1)}M`
  } else if (value > 1000) {
    return `$${(value / 1000).toFixed(1)}K`
  } else {
    return `$${value.toFixed(0)}`
  }
}

const formatTimeAgo = (dateString) => {
  const date = new Date(dateString)
  const now = new Date()
  const diffInMinutes = Math.floor((now - date) / (1000 * 60))
  
  if (diffInMinutes < 60) {
    return `${diffInMinutes}m ago`
  } else if (diffInMinutes < 1440) {
    return `${Math.floor(diffInMinutes / 60)}h ago`
  } else {
    return `${Math.floor(diffInMinutes / 1440)}d ago`
  }
}

const navigateToTokenChart = (token) => {
  if (token.mintAddress) {
    router.push(`/coin/${token.mintAddress}`)
  } else {
    console.warn('Token has no mint address:', token)
  }
}

// Load tokens on component mount
onMounted(() => {
  loadTokens()
})
</script>

<style scoped>
.dashboard {
  display: flex;
  flex-direction: column;
  gap: 32px;
}

/* Mobile Styles */
@media (max-width: 768px) {
  .dashboard {
    gap: 24px;
  }
}
</style>
