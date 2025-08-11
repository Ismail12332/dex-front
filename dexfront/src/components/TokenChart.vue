<template>
  <div class="token-chart-page">
    <!-- Header -->
    <div class="chart-header">
      <button class="back-btn" @click="goBack">
        <svg width="20" height="20" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2" stroke-linecap="round" stroke-linejoin="round">
          <path d="m12 19-7-7 7-7"/>
          <path d="m19 12H5"/>
        </svg>
        [go back]
      </button>
      
      <div class="token-info">
        <div v-if="isTokenLoading" class="loading-container">
          <div class="loading-spinner"></div>
          <span class="loading-text">Loading token data...</span>
        </div>
        
        <div v-else-if="tokenLoadError" class="error-container">
          <span class="error-text">❌ {{ tokenLoadError }}</span>
        </div>
        
        <template v-else>
          <div class="token-avatar">
            <img :src="tokenInfo.image" :alt="tokenInfo.name" />
          </div>
          <div class="token-details">
            <h1 class="token-name">{{ tokenInfo.name }} ({{ tokenInfo.ticker }})</h1>
            <div class="token-meta">
              <span class="created-info">🟢 {{ tokenInfo.created }}</span>
              <span class="market-cap">market cap: $33,692</span>
              <span class="replies">replies: 67</span>
            </div>
          </div>
        </template>
      </div>
    </div>

    <!-- Main Content Layout -->
    <div class="main-content">
      <!-- Chart Section -->
      <div class="chart-section">
        <div class="chart-container">
          <!-- Chart Controls -->
          <div class="chart-controls">
            <div class="timeframe-buttons">
              <button 
                v-for="timeframe in timeframes" 
                :key="timeframe"
                :class="['timeframe-btn', { active: selectedTimeframe === timeframe }]"
                @click="setTimeframe(timeframe)"
              >
                {{ timeframe }}
              </button>
            </div>
            
            <div class="chart-type-buttons">
              <button 
                :class="['chart-type-btn', { active: selectedChartType === 'price' }]"
                @click="setChartType('price')"
              >
                Price/MCap
              </button>
              <button 
                :class="['chart-type-btn', { active: selectedChartType === 'usd' }]"
                @click="setChartType('usd')"
              >
                USD/SOL
              </button>
            </div>
          </div>

          <!-- Price Info -->
          <div class="price-info">
            <div class="current-price">
              <div class="price-label-section">
                <span class="price-label">insider/SOL Market Cap (USD) • 1 • Pump</span>
                <span v-if="formattedHoveredTime" class="hovered-time">{{ formattedHoveredTime }}</span>
              </div>
              <div class="price-data">
                <span class="price-value">{{ currentPrice.toFixed(6) }}</span>
                <div class="price-changes">
                  <span :class="['price-change', priceChange >= 0 ? 'positive' : 'negative']">
                    {{ priceChange >= 0 ? '+' : '' }}{{ priceChange.toFixed(2) }}%
                  </span>
                  <span v-if="hoveredPriceChange !== null" :class="['price-change-vs-prev', hoveredPriceChange >= 0 ? 'positive' : 'negative']">
                    vs prev: {{ hoveredPriceChange >= 0 ? '+' : '' }}{{ hoveredPriceChange.toFixed(2) }}%
                  </span>
                </div>
              </div>
              <!-- OHLC Data Display -->
              <div v-if="hoveredCandle" class="ohlc-data">
                <div class="ohlc-row">
                  <div class="ohlc-item">
                    <span class="ohlc-label">O:</span>
                    <span class="ohlc-value">{{ hoveredCandle.open.toFixed(6) }}</span>
                  </div>
                  <div class="ohlc-item">
                    <span class="ohlc-label">H:</span>
                    <span class="ohlc-value high">{{ hoveredCandle.high.toFixed(6) }}</span>
                  </div>
                  <div class="ohlc-item">
                    <span class="ohlc-label">L:</span>
                    <span class="ohlc-value low">{{ hoveredCandle.low.toFixed(6) }}</span>
                  </div>
                  <div class="ohlc-item">
                    <span class="ohlc-label">C:</span>
                    <span :class="['ohlc-value', hoveredCandle.close > hoveredCandle.open ? 'positive' : 'negative']">
                      {{ hoveredCandle.close.toFixed(6) }}
                    </span>
                  </div>
                </div>
                
                <!-- Professional Trading Metrics -->
                <div class="trading-metrics">
                  <div class="metric-item">
                    <span class="metric-label">Body:</span>
                    <span :class="['metric-value', candleBodyChange >= 0 ? 'positive' : 'negative']">
                      {{ candleBodyChange >= 0 ? '+' : '' }}{{ candleBodyChange.toFixed(2) }}%
                    </span>
                  </div>
                  <div class="metric-item">
                    <span class="metric-label">Range:</span>
                    <span class="metric-value neutral">{{ priceRange ? priceRange.toFixed(2) : '0.00' }}%</span>
                  </div>
                  <div class="metric-item">
                    <span class="metric-label">Upper Wick:</span>
                    <span class="metric-value neutral">{{ wickHigh ? wickHigh.toFixed(2) : '0.00' }}%</span>
                  </div>
                  <div class="metric-item">
                    <span class="metric-label">Lower Wick:</span>
                    <span class="metric-value neutral">{{ wickLow ? wickLow.toFixed(2) : '0.00' }}%</span>
                  </div>
                  <div v-if="hoveredPriceChange !== null" class="metric-item">
                    <span class="metric-label">vs Prev:</span>
                    <span :class="['metric-value', hoveredPriceChange >= 0 ? 'positive' : 'negative']">
                      {{ hoveredPriceChange >= 0 ? '+' : '' }}{{ hoveredPriceChange.toFixed(2) }}%
                    </span>
                  </div>
                </div>
              </div>
            </div>
            <div class="volume-info">
              <span class="volume-label">Volume</span>
              <div class="volume-data">
                <span class="volume-value">{{ currentVolume.toFixed(2) }}</span>
                <span v-if="hoveredVolumeChange !== null" :class="['volume-change', hoveredVolumeChange >= 0 ? 'positive' : 'negative']">
                  {{ hoveredVolumeChange >= 0 ? '+' : '' }}{{ hoveredVolumeChange.toFixed(1) }}%
                </span>
              </div>
            </div>
          </div>

          <!-- Main Chart -->
          <div class="chart-wrapper">
            <div v-if="isChartLoading" class="chart-loading">
              <div class="loading-spinner"></div>
              <span class="loading-text">Loading chart...</span>
            </div>
            <div 
              ref="chartContainer"
              class="chart-canvas"
              :style="{ opacity: isChartLoading ? 0 : 1 }"
            ></div>
          </div>
        </div>
      </div>

      <!-- Right Sidebar -->
      <div class="right-sidebar">
        <!-- Buy/Sell Panel -->
        <div class="trading-panel">
          <div class="trading-tabs">
            <button :class="['trade-tab', { active: activeTradeTab === 'buy' }]" @click="setTradeTab('buy')">
              Buy
            </button>
            <button :class="['trade-tab', { active: activeTradeTab === 'sell' }]" @click="setTradeTab('sell')">
              Sell
            </button>
          </div>

          <div class="trading-header">
            <div class="balance-info">
              <span class="balance-label">balance:</span>
              <span class="balance-value">0.0459 SOL</span>
            </div>
            <button class="max-btn">Max</button>
          </div>

          <div class="trade-input-section">
            <div class="trade-amount">
              <input 
                v-model="tradeAmount" 
                type="number" 
                placeholder="0"
                class="amount-input"
              />
              <div class="token-selector">
                <span class="token-symbol">SOL</span>
                <svg width="12" height="12" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2">
                  <polyline points="6,9 12,15 18,9"/>
                </svg>
              </div>
            </div>
            
            <button :class="['place-trade-btn', activeTradeTab]" @click="buyToken">
              {{ activeTradeTab === 'buy' ? 'Place buy order' : 'Place sell order' }}
            </button>
          </div>

          <div class="trade-info">
            <div class="progress-section">
              <div class="progress-label">bonding curve progress: 87%</div>
              <div class="progress-bar">
                <div class="progress-fill" style="width: 87%"></div>
              </div>
              <div class="progress-description">
                graduate this coin to PumpSwap at $69,356 market cap.
                there is 53.98 SOL in the bonding curve.
              </div>
            </div>

            <div class="action-buttons">
              <button class="action-btn watchlist">
                <svg width="16" height="16" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2">
                  <path d="M19 14c1.49-1.46 3-3.21 3-5.5A5.5 5.5 0 0 0 16.5 3c-1.76 0-3 .5-4.5 2-1.5-1.5-2.74-2-4.5-2A5.5 5.5 0 0 0 2 8.5c0 2.29 1.51 4.04 3 5.5l7 7z"/>
                </svg>
                add to watchlist
              </button>
              
              <button class="action-btn twitter">
                <svg width="16" height="16" viewBox="0 0 24 24" fill="currentColor">
                  <path d="M18.244 2.25h3.308l-7.227 8.26 8.502 11.24H16.17l-5.214-6.817L4.99 21.75H1.68l7.73-8.835L1.254 2.25H8.08l4.713 6.231zm-1.161 17.52h1.833L7.084 4.126H5.117z"/>
                </svg>
                twitter
              </button>
            </div>

            <div class="contract-info">
              <div class="contract-row">
                <span class="contract-label">contract address:</span>
                <span class="contract-value">5TAYr...pump</span>
                <svg class="copy-icon" width="14" height="14" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2">
                  <rect x="9" y="9" width="13" height="13" rx="2" ry="2"/>
                  <path d="M5 15H4a2 2 0 0 1-2-2V4a2 2 0 0 1 2-2h9a2 2 0 0 1 2 2v1"/>
                </svg>
              </div>
              
              <div class="contract-row">
                <svg width="16" height="16" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2">
                  <path d="M10 13a5 5 0 0 0 7.54.54l3-3a5 5 0 0 0-7.07-7.07l-1.72 1.71"/>
                  <path d="M14 11a5 5 0 0 0-7.54-.54l-3 3a5 5 0 0 0 7.07 7.07l1.71-1.71"/>
                </svg>
                <span class="contract-link">trade on MEXC</span>
                <svg class="external-icon" width="12" height="12" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2">
                  <path d="M18 13v6a2 2 0 0 1-2 2H5a2 2 0 0 1-2-2V8a2 2 0 0 1 2-2h6"/>
                  <polyline points="15,3 21,3 21,9"/>
                  <line x1="10" y1="14" x2="21" y2="3"/>
                </svg>
              </div>
              
              <div class="contract-row">
                <svg width="16" height="16" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2">
                  <path d="M10 13a5 5 0 0 0 7.54.54l3-3a5 5 0 0 0-7.07-7.07l-1.72 1.71"/>
                  <path d="M14 11a5 5 0 0 0-7.54-.54l-3 3a5 5 0 0 0 7.07 7.07l1.71-1.71"/>
                </svg>
                <span class="contract-link">view coin in advanced</span>
                <svg class="external-icon" width="12" height="12" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2">
                  <path d="M18 13v6a2 2 0 0 1-2 2H5a2 2 0 0 1-2-2V8a2 2 0 0 1 2-2h6"/>
                  <polyline points="15,3 21,3 21,9"/>
                  <line x1="10" y1="14" x2="21" y2="3"/>
                </svg>
              </div>
            </div>
          </div>
        </div>

      </div>
    </div>

    <!-- Token Information and Top Holders Section -->
    <div class="token-info-section">
      <div class="token-info-container">
        <!-- Left Side: Token Description -->
        <div class="token-description-section">
          <!-- Token Image and Description Layout -->
          <div class="token-header-layout">
            <!-- Token Image -->
            <div class="token-image-section">
              <div class="token-large-image">
                <img :src="tokenInfo.image" :alt="tokenInfo.name" />
              </div>
            </div>

            <!-- Token Info -->
            <div class="token-info-text">
              <div class="token-description">
                <h3 class="description-title">{{ tokenInfo.name }} ({{ tokenInfo.ticker }})</h3>
                <p class="description-text">{{ tokenInfo.description }}</p>
              </div>
            </div>
          </div>

          <!-- Social Links -->
          <div class="social-links-section">
            <h4 class="social-title">Links & Social Media</h4>
            <div class="social-links-grid">
              <!-- Website -->
              <a 
                v-if="tokenInfo.website" 
                :href="tokenInfo.website" 
                target="_blank" 
                class="social-link website"
              >
                <svg width="20" height="20" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2" stroke-linecap="round" stroke-linejoin="round">
                  <circle cx="12" cy="12" r="10"/>
                  <line x1="2" y1="12" x2="22" y2="12"/>
                  <path d="M12 2a15.3 15.3 0 0 1 4 10 15.3 15.3 0 0 1-4 10 15.3 15.3 0 0 1-4-10 15.3 15.3 0 0 1 4-10z"/>
                </svg>
                <span>Website</span>
                <svg class="external-link-icon" width="14" height="14" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2">
                  <path d="M18 13v6a2 2 0 0 1-2 2H5a2 2 0 0 1-2-2V8a2 2 0 0 1 2-2h6"/>
                  <polyline points="15,3 21,3 21,9"/>
                  <line x1="10" y1="14" x2="21" y2="3"/>
                </svg>
              </a>

              <!-- Twitter/X -->
              <a 
                v-if="tokenInfo.twitter" 
                :href="tokenInfo.twitter" 
                target="_blank" 
                class="social-link twitter"
              >
                <svg width="20" height="20" viewBox="0 0 24 24" fill="currentColor">
                  <path d="M18.244 2.25h3.308l-7.227 8.26 8.502 11.24H16.17l-5.214-6.817L4.99 21.75H1.68l7.73-8.835L1.254 2.25H8.08l4.713 6.231zm-1.161 17.52h1.833L7.084 4.126H5.117z"/>
                </svg>
                <span>Twitter</span>
                <svg class="external-link-icon" width="14" height="14" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2">
                  <path d="M18 13v6a2 2 0 0 1-2 2H5a2 2 0 0 1-2-2V8a2 2 0 0 1 2-2h6"/>
                  <polyline points="15,3 21,3 21,9"/>
                  <line x1="10" y1="14" x2="21" y2="3"/>
                </svg>
              </a>

              <!-- Telegram -->
              <a 
                v-if="tokenInfo.telegram" 
                :href="tokenInfo.telegram" 
                target="_blank" 
                class="social-link telegram"
              >
                <svg width="20" height="20" viewBox="0 0 24 24" fill="currentColor">
                  <path d="M12 2C6.48 2 2 6.48 2 12s4.48 10 10 10 10-4.48 10-10S17.52 2 12 2zm4.64 6.8c-.15 1.58-.8 5.42-1.13 7.19-.14.75-.42 1-.68 1.03-.58.05-1.02-.38-1.58-.75-.88-.58-1.38-.94-2.23-1.5-.99-.65-.35-1.01.22-1.59.15-.15 2.71-2.48 2.76-2.69a.2.2 0 0 0-.05-.18c-.06-.05-.14-.03-.21-.02-.09.02-1.49.95-4.22 2.79-.4.27-.76.41-1.08.4-.36-.01-1.04-.2-1.55-.37-.63-.2-1.12-.31-1.08-.66.02-.18.27-.36.74-.55 2.92-1.27 4.86-2.11 5.83-2.51 2.78-1.16 3.35-1.36 3.73-1.36.08 0 .27.02.39.12.1.08.13.19.14.27-.01.06.01.24 0 .38z"/>
                </svg>
                <span>Telegram</span>
                <svg class="external-link-icon" width="14" height="14" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2">
                  <path d="M18 13v6a2 2 0 0 1-2 2H5a2 2 0 0 1-2-2V8a2 2 0 0 1 2-2h6"/>
                  <polyline points="15,3 21,3 21,9"/>
                  <line x1="10" y1="14" x2="21" y2="3"/>
                </svg>
              </a>

              <!-- Discord -->
              <a 
                v-if="tokenInfo.discord" 
                :href="tokenInfo.discord" 
                target="_blank" 
                class="social-link discord"
              >
                <svg width="20" height="20" viewBox="0 0 24 24" fill="currentColor">
                  <path d="M20.317 4.37a19.791 19.791 0 0 0-4.885-1.515.074.074 0 0 0-.079.037c-.21.375-.444.864-.608 1.25a18.27 18.27 0 0 0-5.487 0 12.64 12.64 0 0 0-.617-1.25.077.077 0 0 0-.079-.037A19.736 19.736 0 0 0 3.677 4.37a.07.07 0 0 0-.032.027C.533 9.046-.32 13.58.099 18.057a.082.082 0 0 0 .031.057 19.9 19.9 0 0 0 5.993 3.03.078.078 0 0 0 .084-.028 14.09 14.09 0 0 0 1.226-1.994.076.076 0 0 0-.041-.106 13.107 13.107 0 0 1-1.872-.892.077.077 0 0 1-.008-.128 10.2 10.2 0 0 0 .372-.292.074.074 0 0 1 .077-.01c3.928 1.793 8.18 1.793 12.062 0a.074.074 0 0 1 .078.01c.12.098.246.198.373.292a.077.077 0 0 1-.006.127 12.299 12.299 0 0 1-1.873.892.077.077 0 0 0-.041.107c.36.698.772 1.362 1.225 1.993a.076.076 0 0 0 .084.028 19.839 19.839 0 0 0 6.002-3.03.077.077 0 0 0 .032-.054c.5-5.177-.838-9.674-3.549-13.66a.061.061 0 0 0-.031-.03zM8.02 15.33c-1.183 0-2.157-1.085-2.157-2.419 0-1.333.956-2.419 2.157-2.419 1.21 0 2.176 1.096 2.157 2.42 0 1.333-.956 2.418-2.157 2.418zm7.975 0c-1.183 0-2.157-1.085-2.157-2.419 0-1.333.955-2.419 2.157-2.419 1.21 0 2.176 1.096 2.157 2.42 0 1.333-.946 2.418-2.157 2.418z"/>
                </svg>
                <span>Discord</span>
                <svg class="external-link-icon" width="14" height="14" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2">
                  <path d="M18 13v6a2 2 0 0 1-2 2H5a2 2 0 0 1-2-2V8a2 2 0 0 1 2-2h6"/>
                  <polyline points="15,3 21,3 21,9"/>
                  <line x1="10" y1="14" x2="21" y2="3"/>
                </svg>
              </a>

              <!-- GitHub -->
              <a 
                v-if="tokenInfo.github" 
                :href="tokenInfo.github" 
                target="_blank" 
                class="social-link github"
              >
                <svg width="20" height="20" viewBox="0 0 24 24" fill="currentColor">
                  <path d="M12 0c-6.626 0-12 5.373-12 12 0 5.302 3.438 9.8 8.207 11.387.599.111.793-.261.793-.577v-2.234c-3.338.726-4.033-1.416-4.033-1.416-.546-1.387-1.333-1.756-1.333-1.756-1.089-.745.083-.729.083-.729 1.205.084 1.839 1.237 1.839 1.237 1.07 1.834 2.807 1.304 3.492.997.107-.775.418-1.305.762-1.604-2.665-.305-5.467-1.334-5.467-5.931 0-1.311.469-2.381 1.236-3.221-.124-.303-.535-1.524.117-3.176 0 0 1.008-.322 3.301 1.23.957-.266 1.983-.399 3.003-.404 1.02.005 2.047.138 3.006.404 2.291-1.552 3.297-1.23 3.297-1.23.653 1.653.242 2.874.118 3.176.77.84 1.235 1.911 1.235 3.221 0 4.609-2.807 5.624-5.479 5.921.43.372.823 1.102.823 2.222v3.293c0 .319.192.694.801.576 4.765-1.589 8.199-6.086 8.199-11.386 0-6.627-5.373-12-12-12z"/>
                </svg>
                <span>GitHub</span>
                <svg class="external-link-icon" width="14" height="14" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2">
                  <path d="M18 13v6a2 2 0 0 1-2 2H5a2 2 0 0 1-2-2V8a2 2 0 0 1 2-2h6"/>
                  <polyline points="15,3 21,3 21,9"/>
                  <line x1="10" y1="14" x2="21" y2="3"/>
                </svg>
              </a>

              <!-- Medium -->
              <a 
                v-if="tokenInfo.medium" 
                :href="tokenInfo.medium" 
                target="_blank" 
                class="social-link medium"
              >
                <svg width="20" height="20" viewBox="0 0 24 24" fill="currentColor">
                  <path d="M13.54 12a6.8 6.8 0 01-6.77 6.82A6.8 6.8 0 010 12a6.8 6.8 0 016.77-6.82A6.8 6.8 0 0113.54 12zM20.96 12c0 3.54-1.51 6.42-3.38 6.42-1.87 0-3.39-2.88-3.39-6.42s1.52-6.42 3.39-6.42 3.38 2.88 3.38 6.42M24 12c0 3.17-.53 5.75-1.19 5.75-.66 0-1.19-2.58-1.19-5.75s.53-5.75 1.19-5.75C23.47 6.25 24 8.83 24 12z"/>
                </svg>
                <span>Medium</span>
                <svg class="external-link-icon" width="14" height="14" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2">
                  <path d="M18 13v6a2 2 0 0 1-2 2H5a2 2 0 0 1-2-2V8a2 2 0 0 1 2-2h6"/>
                  <polyline points="15,3 21,3 21,9"/>
                  <line x1="10" y1="14" x2="21" y2="3"/>
                </svg>
              </a>
            </div>
          </div>

          <!-- Token Stats -->
          <div class="token-stats-section">
            <h4 class="stats-title">Token Statistics</h4>
            <div class="stats-grid">
              <div class="stat-item">
                <span class="stat-label">Total Supply</span>
                <span class="stat-value">{{ tokenInfo.totalSupply }}</span>
              </div>
              <div class="stat-item">
                <span class="stat-label">Circulating Supply</span>
                <span class="stat-value">{{ tokenInfo.circulatingSupply }}</span>
              </div>
              <div class="stat-item">
                <span class="stat-label">Holders</span>
                <span class="stat-value">{{ tokenInfo.holders }}</span>
              </div>
              <div class="stat-item">
                <span class="stat-label">Created</span>
                <span class="stat-value">{{ tokenInfo.created }}</span>
              </div>
            </div>
          </div>
        </div>

        <!-- Right Side: Top Holders -->
        <div class="top-holders">
          <div class="holders-header">
            <h3 class="holders-title">Top holders</h3>
            <button class="bubble-map-btn">Generate bubble map</button>
          </div>
          
          <div class="holders-list">
            <div 
              v-for="(holder, index) in topHolders" 
              :key="index"
              class="holder-item"
            >
              <span class="holder-rank">{{ index + 1 }}.</span>
              <span class="holder-type">{{ holder.type }}</span>
              <span class="holder-percentage">{{ holder.percentage }}%</span>
            </div>
          </div>
        </div>
      </div>
    </div>

    <!-- Trades Section (Below Everything) -->
    <div class="trades-section">
      <div class="trades-header">
        <div class="trades-tabs">
          <button class="trades-tab active">thread</button>
          <button class="trades-tab">trades</button>
        </div>
        
        <div class="trades-filters">
          <div class="filter-group">
            <label class="filter-toggle">
              <input type="checkbox" v-model="filters.sizeFilter">
              <span class="toggle-slider"></span>
            </label>
            <span class="filter-label">filter by size</span>
            <input v-model="filters.minSize" type="number" step="0.01" placeholder="0.05" class="size-input">
            <span class="size-note">(1835 trades of size greater than 0.05 SOL)</span>
          </div>
          
          <div class="filter-group">
            <label class="filter-toggle">
              <input type="checkbox" v-model="filters.followingFilter">
              <span class="toggle-slider"></span>
            </label>
            <span class="filter-label">filter by following</span>
            <span class="following-note">(0 trades from people you follow)</span>
          </div>
          
          <div class="filter-group">
            <label class="filter-toggle">
              <input type="checkbox" v-model="filters.ownTradesFilter">
              <span class="toggle-slider"></span>
            </label>
            <span class="filter-label">filter by own trades</span>
            <span class="own-trades-note">(0 trades you made)</span>
          </div>
        </div>
      </div>

      <div class="trades-table">
        <div class="table-header">
          <div class="header-cell account">account</div>
          <div class="header-cell type">type</div>
          <div class="header-cell sol">SOL</div>
          <div class="header-cell insider">insider</div>
          <div class="header-cell date">date</div>
          <div class="header-cell transaction">transaction</div>
        </div>
        
        <div class="table-body">
          <div 
            v-for="(trade, index) in paginatedTrades" 
            :key="index"
            class="trade-row"
          >
            <div class="trade-cell account">
              <div class="account-info">
                <div class="account-avatar">
                  <img :src="trade.avatar" :alt="trade.account" />
                </div>
                <span class="account-name">{{ trade.account }}</span>
              </div>
            </div>
            <div class="trade-cell type">
              <span :class="['trade-type', trade.type]">{{ trade.type }}</span>
            </div>
            <div class="trade-cell sol">{{ trade.sol }}</div>
            <div class="trade-cell insider">
              {{ trade.insider }}
              <svg v-if="trade.warning" class="warning-icon" width="12" height="12" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2">
                <path d="M10.29 3.86L1.82 18a2 2 0 0 0 1.71 3h16.94a2 2 0 0 0 1.71-3L13.71 3.86a2 2 0 0 0-3.42 0z"/>
                <line x1="12" y1="9" x2="12" y2="13"/>
                <line x1="12" y1="17" x2="12.01" y2="17"/>
              </svg>
            </div>
            <div class="trade-cell date">{{ trade.date }}</div>
            <div class="trade-cell transaction">
              <a :href="trade.transactionUrl" class="transaction-link" target="_blank">
                {{ trade.transaction }}
              </a>
            </div>
          </div>
        </div>
        
        <!-- Pagination -->
        <div class="pagination">
          <button 
            class="pagination-btn" 
            :disabled="currentPage === 1"
            @click="previousPage"
          >
            <svg width="16" height="16" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2">
              <polyline points="15,18 9,12 15,6"/>
            </svg>
            Previous
          </button>
          
          <div class="pagination-info">
            <span class="page-info">Page {{ currentPage }} of {{ totalPages }}</span>
            <span class="trades-count">({{ startIndex }}-{{ endIndex }} of {{ totalTrades.length }} trades)</span>
          </div>
          
          <button 
            class="pagination-btn" 
            :disabled="currentPage === totalPages"
            @click="nextPage"
          >
            Next
            <svg width="16" height="16" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2">
              <polyline points="9,18 15,12 9,6"/>
            </svg>
          </button>
        </div>
      </div>
    </div>
  </div>
</template>

<script setup>
import { ref, onMounted, computed, inject, nextTick, onUnmounted } from 'vue'
import { createChart } from 'lightweight-charts'
import { useRouter, useRoute } from 'vue-router'

const router = useRouter()
const route = useRoute()

// Token loading state
const isTokenLoading = ref(true)
const tokenLoadError = ref(null)

// Chart data and state
const chartContainer = ref(null)
const chart = ref(null)
const candlestickSeries = ref(null)
const volumeSeries = ref(null)
const selectedTimeframe = ref('1m')
const timeframes = ['1m', '5m', '15m', '1h', '4h', '1d']
const selectedChartType = ref('price')
const chartTypes = ['price', 'usd']
const isChartLoading = ref(true)

// Trading state
const tradeAmount = ref('')
const activeTradeTab = ref('buy')

// Price data
const currentPrice = ref(0.13376)
const priceChange = ref(-10.03)
const currentVolume = ref(5)
const hoveredCandle = ref(null)
const hoveredTime = ref(null)
const hoveredPriceChange = ref(null)
const hoveredVolumeChange = ref(null)
const candleBodyChange = ref(null) // Изменение от open к close
const wickHigh = ref(null) // Насколько высокий фитиль от close/open к high
const wickLow = ref(null) // Насколько низкий фитиль от close/open к low
const priceRange = ref(null) // Диапазон цены (high - low) в %

// Store chart data for crosshair calculations
const chartCandleData = ref([])
const chartVolumeData = ref([])

// Token information
const tokenInfo = ref({
  name: 'insider stock',
  ticker: 'insider',
  image: 'https://images.pexels.com/photos/1640777/pexels-photo-1640777.jpeg?auto=compress&cs=tinysrgb&w=400&h=400&dpr=1',
  description: 'insider stock is a revolutionary meme token that brings transparency to the crypto market. Built on Solana blockchain, it aims to democratize access to insider trading information through community-driven insights and real-time market analysis. Join the movement of informed traders who believe in fair and transparent markets.',
  website: 'https://insiderstock.com',
  twitter: 'https://twitter.com/insiderstock',
  telegram: 'https://t.me/insiderstock',
  discord: 'https://discord.gg/insiderstock',
  github: 'https://github.com/insiderstock',
  medium: 'https://medium.com/@insiderstock',
  totalSupply: '1,000,000,000',
  circulatingSupply: '850,000,000',
  holders: '2,847',
  created: '2 hours ago'
})

// Filters state
const filters = ref({
  sizeFilter: true,
  minSize: 0.05,
  followingFilter: false,
  ownTradesFilter: false
})

// Pagination state
const currentPage = ref(1)
const tradesPerPage = 15

// Generate realistic OHLC candlestick data
const generateCandlestickData = () => {
  const candleData = []
  const volumeData = []
  const now = new Date()
  let basePrice = 0.13376 // Starting price
  
  for (let i = 99; i >= 0; i--) {
    const time = Math.floor((now.getTime() - i * 60 * 1000) / 1000) // Unix timestamp in seconds (1-minute candles)
    
    // Generate realistic OHLC data with trend and volatility
    const trend = (Math.random() - 0.5) * 0.002 // Small trend component
    const volatility = 0.001 + Math.random() * 0.005 // Variable volatility
    
    const open = basePrice
    
    // Generate intraday movements
    const upMove = Math.random() * volatility
    const downMove = Math.random() * volatility
    
    let high = open + upMove
    let low = open - downMove
    
    // Close price with trend bias
    const closeDirection = Math.random() > 0.5 ? 1 : -1
    const closeMagnitude = Math.random() * volatility * 0.7
    let close = open + (closeDirection * closeMagnitude) + trend
    
    // Ensure proper OHLC relationships
    high = Math.max(open, close, high)
    low = Math.min(open, close, low)
    
    // Ensure price doesn't go below a reasonable threshold
    if (low < 0.001) {
      low = 0.001
      if (close < low) close = low + 0.0001
      if (open < low) open = low + 0.0001
      if (high < close) high = close + 0.0001
    }
    
    // Volume correlated with price movement
    const priceMovement = Math.abs(close - open) / open
    const baseVolume = 2 + Math.random() * 8
    const volume = baseVolume * (1 + priceMovement * 10) // Higher volume with bigger moves
    
    candleData.push({
      time: time,
      open: parseFloat(open.toFixed(6)),
      high: parseFloat(high.toFixed(6)),
      low: parseFloat(low.toFixed(6)),
      close: parseFloat(close.toFixed(6))
    })
    
    volumeData.push({
      time: time,
      value: parseFloat(volume.toFixed(2)),
      color: close > open ? 'rgba(16, 185, 129, 0.4)' : 'rgba(239, 68, 68, 0.4)'
    })
    
    basePrice = close // Next candle starts where this one ended
  }
  
  return { candleData, volumeData }
}

const { candleData, volumeData } = generateCandlestickData()

// Store data globally for crosshair calculations
chartCandleData.value = candleData
chartVolumeData.value = volumeData

// Top holders data
const topHolders = ref([
  { type: 'bonding curve', percentage: 31.03 },
  { type: 'SvTim3', percentage: 3.39 },
  { type: '3REFEM', percentage: 3.24 },
  { type: '7DeBwp', percentage: 2.92 },
  { type: 'AFHDSs', percentage: 2.81 },
  { type: '7dqYYS', percentage: 2.77 },
  { type: '7D94jp', percentage: 2.61 },
  { type: 'GTYwG', percentage: 2.50 },
  { type: 'GB4hYX', percentage: 2.50 },
  { type: 'CmBY2Q', percentage: 2.12 },
  { type: '3gahKg', percentage: 1.84 }
])

// Generate trades data
const generateTradesData = () => {
  const trades = []
  const accounts = ['zWxte', 'zhb47', 'sNT3Um', 'shu1pu', 'JDEMG']
  const avatars = [
    'https://images.pexels.com/photos/1640777/pexels-photo-1640777.jpeg?auto=compress&cs=tinysrgb&w=40&h=40&dpr=1',
    'https://images.pexels.com/photos/2379004/pexels-photo-2379004.jpeg?auto=compress&cs=tinysrgb&w=40&h=40&dpr=1',
    'https://images.pexels.com/photos/33535/monkey-ape-thinking-mimic.jpg?auto=compress&cs=tinysrgb&w=40&h=40&dpr=1',
    'https://images.pexels.com/photos/730547/pexels-photo-730547.jpeg?auto=compress&cs=tinysrgb&w=40&h=40&dpr=1',
    'https://images.pexels.com/photos/1571460/pexels-photo-1571460.jpeg?auto=compress&cs=tinysrgb&w=40&h=40&dpr=1'
  ]
  
  for (let i = 0; i < 50; i++) {
    const account = accounts[Math.floor(Math.random() * accounts.length)]
    const type = Math.random() > 0.5 ? 'buy' : 'sell'
    const sol = (Math.random() * 2).toFixed(3)
    const insider = (Math.random() * 5).toFixed(2) + 'm'
    const hoursAgo = Math.floor(Math.random() * 36) + 1
    const date = `${hoursAgo}h ago`
    const transactionId = Math.random().toString(36).substring(2, 8)
    
    trades.push({
      account,
      avatar: avatars[Math.floor(Math.random() * avatars.length)],
      type,
      sol,
      insider,
      date,
      transaction: transactionId,
      transactionUrl: `https://solscan.io/tx/${transactionId}`,
      warning: Math.random() > 0.8
    })
  }
  
  return trades
}

const totalTrades = ref(generateTradesData())

// Computed properties for pagination
const totalPages = computed(() => Math.ceil(totalTrades.value.length / tradesPerPage))
const startIndex = computed(() => (currentPage.value - 1) * tradesPerPage + 1)
const endIndex = computed(() => Math.min(currentPage.value * tradesPerPage, totalTrades.value.length))

const paginatedTrades = computed(() => {
  const start = (currentPage.value - 1) * tradesPerPage
  const end = start + tradesPerPage
  return totalTrades.value.slice(start, end)
})

const formattedHoveredTime = computed(() => {
  if (!hoveredTime.value) return null
  
  const date = new Date(hoveredTime.value * 1000)
  return date.toLocaleString('en-US', {
    month: 'short',
    day: 'numeric',
    hour: '2-digit',
    minute: '2-digit',
    hour12: false
  })
})

// Initialize TradingView Chart
const initializeChart = () => {
  if (!chartContainer.value) return

  // Create the chart
  chart.value = createChart(chartContainer.value, {
    width: chartContainer.value.clientWidth,
    height: 500,
    layout: {
      background: { color: 'transparent' },
      textColor: '#d1d5db',
      fontSize: 12,
      fontFamily: 'Monaco, Menlo, monospace',
    },
    grid: {
      vertLines: { 
        color: 'rgba(255, 255, 255, 0.06)',
        style: 1,
        visible: true,
      },
      horzLines: { 
        color: 'rgba(255, 255, 255, 0.06)',
        style: 1,
        visible: true,
      },
    },
    crosshair: {
      mode: 1, // Normal crosshair mode
      vertLine: {
        color: 'rgba(255, 255, 255, 0.8)',
        width: 1,
        style: 2, // Dashed line
        labelVisible: true,
        labelBackgroundColor: '#42b883',
      },
      horzLine: {
        color: 'rgba(255, 255, 255, 0.8)',
        width: 1,
        style: 2, // Dashed line
        labelVisible: true,
        labelBackgroundColor: '#42b883',
      },
    },
    priceScale: {
      borderColor: 'rgba(255, 255, 255, 0.1)',
      textColor: '#d1d5db',
      fontSize: 11,
      entireTextOnly: false,
      scaleMargins: {
        top: 0.1,
        bottom: 0.2,
      },
    },
    timeScale: {
      borderColor: 'rgba(255, 255, 255, 0.1)',
      textColor: '#d1d5db',
      fontSize: 11,
      timeVisible: true,
      secondsVisible: false,
      tickMarkFormatter: (time) => {
        const date = new Date(time * 1000)
        const now = new Date()
        const diffHours = (now - date) / (1000 * 60 * 60)
        
        if (diffHours < 1) {
          return date.toLocaleTimeString('en-US', {
            hour: '2-digit',
            minute: '2-digit',
            hour12: false
          })
        } else if (diffHours < 24) {
          return date.toLocaleTimeString('en-US', {
            hour: '2-digit',
            minute: '2-digit',
            hour12: false
          })
        } else {
          return date.toLocaleDateString('en-US', {
            month: 'short',
            day: 'numeric'
          })
        }
      },
    },
    rightPriceScale: {
      visible: true,
      borderColor: 'rgba(255, 255, 255, 0.1)',
      textColor: '#d1d5db',
      fontSize: 11,
      scaleMargins: {
        top: 0.1,
        bottom: 0.2,
      },
    },
    leftPriceScale: {
      visible: false,
    },
    handleScroll: {
      mouseWheel: true,
      pressedMouseMove: true,
      horzTouchDrag: true,
      vertTouchDrag: true,
    },
    handleScale: {
      axisPressedMouseMove: true,
      mouseWheel: true,
      pinch: true,
    },
    kineticScroll: {
      touch: true,
      mouse: false,
    },
  })

  // Add candlestick series
  candlestickSeries.value = chart.value.addCandlestickSeries({
    upColor: '#10b981',
    downColor: '#ef4444',
    borderUpColor: '#10b981',
    borderDownColor: '#ef4444',
    wickUpColor: '#10b981',
    wickDownColor: '#ef4444',
    borderVisible: true,
    wickVisible: true,
    priceFormat: {
      type: 'price',
      precision: 6,
      minMove: 0.000001,
    },
  })

  // Add volume series (histogram)
  volumeSeries.value = chart.value.addHistogramSeries({
    color: 'rgba(66, 184, 131, 0.4)',
    priceFormat: {
      type: 'volume',
      precision: 2,
    },
    priceScaleId: '', // Set as an overlay by setting a blank priceScaleId
    scaleMargins: {
      top: 0.75, // highest point of the series will be 75% away from the top
      bottom: 0,
    },
    lastValueVisible: false,
    priceLineVisible: false,
  })

  // Set the data
  candlestickSeries.value.setData(chartCandleData.value)
  volumeSeries.value.setData(chartVolumeData.value)

  // Subscribe to crosshair move to show tooltip
  chart.value.subscribeCrosshairMove((param) => {
    if (!param.time || !param.point || !param.seriesData.size) {
      // Reset to latest candle data when no hover
      hoveredCandle.value = null
      hoveredTime.value = null
      hoveredPriceChange.value = null
      hoveredVolumeChange.value = null
      candleBodyChange.value = null
      wickHigh.value = null
      wickLow.value = null
      priceRange.value = null
      
      if (chartCandleData.value.length > 0) {
        const latestCandle = chartCandleData.value[chartCandleData.value.length - 1]
        currentPrice.value = latestCandle.close
        
        // Calculate price change from previous candle
        if (chartCandleData.value.length > 1) {
          const previousCandle = chartCandleData.value[chartCandleData.value.length - 2]
          const change = ((latestCandle.close - previousCandle.close) / previousCandle.close) * 100
          priceChange.value = parseFloat(change.toFixed(2))
        }
      }
      
      if (chartVolumeData.value.length > 0) {
        const latestVolume = chartVolumeData.value[chartVolumeData.value.length - 1]
        currentVolume.value = latestVolume.value
      }
      return
    }

    // Get candlestick data for current position
    const candleDataPoint = param.seriesData.get(candlestickSeries.value)
    const volumeDataPoint = param.seriesData.get(volumeSeries.value)
    
    if (candleDataPoint) {
      // Update hovered candle data
      hoveredCandle.value = candleDataPoint
      hoveredTime.value = param.time
      
      // Update current price display with hovered candle data
      currentPrice.value = candleDataPoint.close
      
      // Calculate candle body change (from open to close)
      const bodyChange = ((candleDataPoint.close - candleDataPoint.open) / candleDataPoint.open) * 100
      candleBodyChange.value = parseFloat(bodyChange.toFixed(2))
      priceChange.value = parseFloat(bodyChange.toFixed(2))
      
      // Calculate price range (high - low) as percentage of open
      const range = ((candleDataPoint.high - candleDataPoint.low) / candleDataPoint.open) * 100
      priceRange.value = parseFloat(range.toFixed(2))
      
      // Calculate wick sizes
      const bodyHigh = Math.max(candleDataPoint.open, candleDataPoint.close)
      const bodyLow = Math.min(candleDataPoint.open, candleDataPoint.close)
      
      const upperWick = ((candleDataPoint.high - bodyHigh) / candleDataPoint.open) * 100
      const lowerWick = ((bodyLow - candleDataPoint.low) / candleDataPoint.open) * 100
      
      wickHigh.value = parseFloat(upperWick.toFixed(2))
      wickLow.value = parseFloat(lowerWick.toFixed(2))
      
      // Find the previous candle to calculate change vs previous close
      const currentIndex = chartCandleData.value.findIndex(candle => candle.time === param.time)
      if (currentIndex > 0) {
        const previousCandle = chartCandleData.value[currentIndex - 1]
        const changeVsPrev = ((candleDataPoint.close - previousCandle.close) / previousCandle.close) * 100
        hoveredPriceChange.value = parseFloat(changeVsPrev.toFixed(2))
      } else {
        hoveredPriceChange.value = null
      }
    }
    
    if (volumeDataPoint) {
      currentVolume.value = volumeDataPoint.value
      
      // Calculate volume change vs previous if available
      const currentIndex = chartVolumeData.value.findIndex(vol => vol.time === param.time)
      if (currentIndex > 0) {
        const previousVolume = chartVolumeData.value[currentIndex - 1]
        const volumeChangePct = ((volumeDataPoint.value - previousVolume.value) / previousVolume.value) * 100
        hoveredVolumeChange.value = parseFloat(volumeChangePct.toFixed(2))
      } else {
        hoveredVolumeChange.value = null
      }
    }
  })

  // Auto-resize chart
  const resizeObserver = new ResizeObserver(entries => {
    if (chart.value && chartContainer.value) {
      chart.value.applyOptions({
        width: chartContainer.value.clientWidth,
        height: 500,
      })
    }
  })
  
  resizeObserver.observe(chartContainer.value)

  // Store the observer for cleanup
  chart.value._resizeObserver = resizeObserver
}

// Methods
const goBack = () => {
  router.back()
}

const setTimeframe = (timeframe) => {
  selectedTimeframe.value = timeframe
  
  // Generate new data based on timeframe
  const timeframeMinutes = {
    '1m': 1,
    '5m': 5,
    '15m': 15,
    '1h': 60,
    '4h': 240,
    '1d': 1440
  }
  
  const minutes = timeframeMinutes[timeframe] || 1
  const candleData = []
  const volumeData = []
  const now = new Date()
  let basePrice = currentPrice.value || 0.13376
  
  for (let i = 99; i >= 0; i--) {
    const time = Math.floor((now.getTime() - i * minutes * 60 * 1000) / 1000)
    
    const trend = (Math.random() - 0.5) * 0.002 * minutes // Longer timeframes have bigger trends
    const volatility = (0.001 + Math.random() * 0.005) * Math.sqrt(minutes) // Volatility scales with time
    
    const open = basePrice
    const upMove = Math.random() * volatility
    const downMove = Math.random() * volatility
    
    let high = open + upMove
    let low = open - downMove
    
    const closeDirection = Math.random() > 0.5 ? 1 : -1
    const closeMagnitude = Math.random() * volatility * 0.7
    let close = open + (closeDirection * closeMagnitude) + trend
    
    high = Math.max(open, close, high)
    low = Math.min(open, close, low)
    
    if (low < 0.001) {
      low = 0.001
      if (close < low) close = low + 0.0001
      if (open < low) open = low + 0.0001
      if (high < close) high = close + 0.0001
    }
    
    const priceMovement = Math.abs(close - open) / open
    const baseVolume = (2 + Math.random() * 8) * minutes // Volume scales with timeframe
    const volume = baseVolume * (1 + priceMovement * 10)
    
    candleData.push({
      time: time,
      open: parseFloat(open.toFixed(6)),
      high: parseFloat(high.toFixed(6)),
      low: parseFloat(low.toFixed(6)),
      close: parseFloat(close.toFixed(6))
    })
    
    volumeData.push({
      time: time,
      value: parseFloat(volume.toFixed(2)),
      color: close > open ? 'rgba(16, 185, 129, 0.4)' : 'rgba(239, 68, 68, 0.4)'
    })
    
    basePrice = close
  }
  
  if (candlestickSeries.value && volumeSeries.value) {
    candlestickSeries.value.setData(candleData)
    volumeSeries.value.setData(volumeData)
    
    // Update global chart data for crosshair calculations
    chartCandleData.value = candleData
    chartVolumeData.value = volumeData
    
    // Update current price to latest candle
    if (candleData.length > 0) {
      const latestCandle = candleData[candleData.length - 1]
      currentPrice.value = latestCandle.close
      
      if (candleData.length > 1) {
        const previousCandle = candleData[candleData.length - 2]
        const change = ((latestCandle.close - previousCandle.close) / previousCandle.close) * 100
        priceChange.value = parseFloat(change.toFixed(2))
      }
    }
  }
}

const setChartType = (type) => {
  selectedChartType.value = type
  // In a real implementation, this would change the data source
  // For demo purposes, we'll just change the price multiplier
  const multiplier = type === 'usd' ? 0.000045 : 1 // Simulating USD/SOL rate
  
  // Regenerate data with new multiplier
  setTimeframe(selectedTimeframe.value)
}

const setTradeTab = (tab) => {
  activeTradeTab.value = tab
}

const previousPage = () => {
  if (currentPage.value > 1) {
    currentPage.value--
  }
}

const nextPage = () => {
  if (currentPage.value < totalPages.value) {
    currentPage.value++
  }
}

// Load token data from API
const loadTokenData = async (mintAddress) => {
  if (!mintAddress) {
    tokenLoadError.value = 'No mint address provided'
    isTokenLoading.value = false
    return
  }

  try {
    isTokenLoading.value = true
    tokenLoadError.value = null

    const response = await fetch(`http://localhost:3000/api/coin/${mintAddress}`)
    
    if (!response.ok) {
      throw new Error(`Failed to load token data: ${response.status}`)
    }

    const responseData = await response.json()
    
    // Проверяем структуру ответа и извлекаем токен
    const tokenData = responseData.success ? responseData.token : responseData
    
    // Update token info with API data
    tokenInfo.value = {
      ...tokenInfo.value,
      name: tokenData.name || tokenInfo.value.name,
      ticker: tokenData.symbol || tokenInfo.value.ticker,
      description: tokenData.description || tokenInfo.value.description,
      image: tokenData.image_url && tokenData.image_url !== 'placeholder' 
        ? tokenData.image_url 
        : tokenInfo.value.image,
      website: tokenData.website || tokenInfo.value.website,
      twitter: tokenData.twitter || tokenInfo.value.twitter,
      telegram: tokenData.telegram || tokenInfo.value.telegram,
      totalSupply: tokenData.supply ? tokenData.supply.toLocaleString() : tokenInfo.value.totalSupply,
      created: tokenData.createdAt ? new Date(tokenData.createdAt).toLocaleString() : tokenInfo.value.created,
      mintAddress: tokenData.mint_address
    }

    console.log('Token data loaded:', tokenData)
    
  } catch (error) {
    console.error('Failed to load token data:', error)
    tokenLoadError.value = error.message
  } finally {
    isTokenLoading.value = false
  }
}

// Добавьте этот метод в <script setup> или methods вашего компонента

async function buyToken() {
  if (!window.solana || !window.solana.isPhantom) {
    alert('Phantom wallet не найден!');
    return;
  }
  await window.solana.connect();

  const mint = tokenInfo.value.mintAddress || route.params.mint_address;
  const userPubkey = window.solana.publicKey.toString();
  const solAmount = Math.floor(Number(tradeAmount.value) * 1e9);
  const minTokensOut = 1;

  console.log('Покупка токенов:', {
    mint,
    userPubkey,
    solAmount,
    minTokensOut
  });
  // 1. Запрос на backend
  const response = await fetch('http://localhost:3001/api/buy-tokens', {
    method: 'POST',
    headers: { 'Content-Type': 'application/json' },
    body: JSON.stringify({
      token_mint: mint,
      buyer_pubkey: userPubkey,
      sol_amount: solAmount,
      min_tokens_out: minTokensOut
    })
  });

  const data = await response.json();
  if (!data.success) {
    alert('Ошибка: ' + (data.message || 'Не удалось получить инструкцию'));
    return;
  }

  console.log('Полученная инструкция:', data);

  // 2. Собираем инструкцию
  const { Transaction, TransactionInstruction, PublicKey, Connection, clusterApiUrl } = await import('@solana/web3.js');
  const connection = new Connection(clusterApiUrl('devnet'));

  const instr = data.instruction;
  const instruction = new TransactionInstruction({
    programId: new PublicKey(instr.program_id),
    keys: instr.accounts.map(acc => ({
      pubkey: new PublicKey(acc.pubkey),
      isSigner: acc.is_signer,
      isWritable: acc.is_writable
    })),
    data: Uint8Array.from(atob(instr.data), c => c.charCodeAt(0))
  });

  // 3. Получаем свежий блокхеш и собираем транзакцию
  const { blockhash } = await connection.getLatestBlockhash();
  const tx = new Transaction().add(instruction);
  tx.recentBlockhash = blockhash;
  tx.feePayer = window.solana.publicKey;

  // 4. Подписываем через Phantom
  const signedTx = await window.solana.signTransaction(tx);

  // 5. Отправляем в сеть
  const signature = await connection.sendRawTransaction(signedTx.serialize());
  await connection.confirmTransaction(signature);

  alert('Покупка успешно отправлена! Транзакция: ' + signature);
}

// Lifecycle
onMounted(async () => {
  await nextTick()
  
  // Load token data from API if mint_address is provided in route params
  const mintAddress = route.params.mint_address
  if (mintAddress) {
    await loadTokenData(mintAddress)
  }
  
  // Initialize TradingView chart
  initializeChart()
  
  // Update current price from latest candle
  if (chartCandleData.value.length > 0) {
    const latestCandle = chartCandleData.value[chartCandleData.value.length - 1]
    currentPrice.value = latestCandle.close
  }
  
  if (chartVolumeData.value.length > 0) {
    const latestVolume = chartVolumeData.value[chartVolumeData.value.length - 1]
    currentVolume.value = latestVolume.value
  }
  
  // Simulate loading delay for smooth UX
  setTimeout(() => {
    isChartLoading.value = false
  }, 1000)
})

onUnmounted(() => {
  // Clean up chart and resize observer
  if (chart.value) {
    if (chart.value._resizeObserver) {
      chart.value._resizeObserver.disconnect()
    }
    chart.value.remove()
  }
})
</script>

<style scoped>
.token-chart-page {
  padding: 20px;
  max-width: 1600px;
  margin: 0 auto;
  background: #0f0f23;
  min-height: 100vh;
}

.chart-header {
  display: flex;
  align-items: center;
  gap: 20px;
  margin-bottom: 24px;
  padding-bottom: 16px;
  border-bottom: 1px solid rgba(255, 255, 255, 0.1);
}

.back-btn {
  background: transparent;
  border: none;
  color: #9ca3af;
  font-size: 14px;
  cursor: pointer;
  display: flex;
  align-items: center;
  gap: 8px;
  transition: color 0.2s ease;
}

.back-btn:hover {
  color: #ffffff;
}

.token-info {
  display: flex;
  align-items: center;
  gap: 16px;
  flex: 1;
}

.token-avatar {
  width: 60px;
  height: 60px;
  border-radius: 12px;
  overflow: hidden;
  flex-shrink: 0;
}

.token-avatar img {
  width: 100%;
  height: 100%;
  object-fit: cover;
}

.token-details {
  flex: 1;
}

.token-name {
  font-size: 24px;
  font-weight: 700;
  color: #ffffff;
  margin-bottom: 8px;
}

.token-meta {
  display: flex;
  align-items: center;
  gap: 16px;
  font-size: 14px;
  color: #9ca3af;
}

.created-info {
  color: #10b981;
}

/* Loading and Error States */
.loading-container {
  display: flex;
  align-items: center;
  gap: 12px;
  padding: 20px;
}

.loading-spinner {
  width: 20px;
  height: 20px;
  border: 2px solid rgba(255, 255, 255, 0.1);
  border-left-color: #42b883;
  border-radius: 50%;
  animation: spin 1s linear infinite;
}

.loading-text {
  color: #9ca3af;
  font-size: 14px;
}

.error-container {
  padding: 20px;
}

.error-text {
  color: #ef4444;
  font-size: 14px;
  font-weight: 500;
}

@keyframes spin {
  to {
    transform: rotate(360deg);
  }
}

/* Main Content Layout - Enlarged Chart */
.main-content {
  display: grid;
  grid-template-columns: 1fr 360px;
  gap: 24px;
  margin-bottom: 32px;
}

.chart-section {
  min-width: 0; /* Prevents grid overflow */
}

.chart-container {
  background: rgba(255, 255, 255, 0.05);
  backdrop-filter: blur(10px);
  border: 1px solid rgba(255, 255, 255, 0.1);
  border-radius: 16px;
  padding: 20px;
}

.chart-controls {
  display: flex;
  justify-content: space-between;
  align-items: center;
  margin-bottom: 16px;
}

.timeframe-buttons {
  display: flex;
  gap: 4px;
}

.timeframe-btn {
  padding: 6px 12px;
  background: rgba(255, 255, 255, 0.05);
  border: 1px solid rgba(255, 255, 255, 0.1);
  border-radius: 6px;
  color: #9ca3af;
  font-size: 12px;
  font-weight: 500;
  cursor: pointer;
  transition: all 0.2s ease;
}

.timeframe-btn.active {
  background: #42b883;
  color: #ffffff;
  border-color: #42b883;
}

.chart-type-buttons {
  display: flex;
  gap: 4px;
}

.chart-type-btn {
  padding: 6px 12px;
  background: rgba(255, 255, 255, 0.05);
  border: 1px solid rgba(255, 255, 255, 0.1);
  border-radius: 6px;
  color: #9ca3af;
  font-size: 12px;
  font-weight: 500;
  cursor: pointer;
  transition: all 0.2s ease;
}

.chart-type-btn.active {
  background: #42b883;
  color: #ffffff;
  border-color: #42b883;
}

.price-info {
  display: flex;
  justify-content: space-between;
  align-items: center;
  margin-bottom: 16px;
  padding-bottom: 12px;
  border-bottom: 1px solid rgba(255, 255, 255, 0.1);
}

.current-price {
  display: flex;
  flex-direction: column;
  gap: 4px;
}

.price-label-section {
  display: flex;
  justify-content: space-between;
  align-items: center;
}

.price-label {
  font-size: 12px;
  color: #9ca3af;
}

.hovered-time {
  font-size: 11px;
  color: #42b883;
  font-weight: 600;
  background: rgba(66, 184, 131, 0.1);
  padding: 2px 8px;
  border-radius: 4px;
}

.price-data {
  display: flex;
  flex-direction: column;
  gap: 8px;
}

.price-changes {
  display: flex;
  gap: 12px;
  align-items: center;
}

.price-value {
  font-size: 20px;
  font-weight: 700;
  color: #ffffff;
  font-family: 'Monaco', 'Menlo', monospace;
}

.price-change {
  font-size: 14px;
  font-weight: 600;
}

.price-change-vs-prev {
  font-size: 12px;
  font-weight: 500;
  padding: 2px 6px;
  border-radius: 4px;
  background: rgba(255, 255, 255, 0.05);
}

.price-change.positive,
.price-change-vs-prev.positive {
  color: #10b981;
}

.price-change.negative,
.price-change-vs-prev.negative {
  color: #ef4444;
}

.volume-info {
  display: flex;
  flex-direction: column;
  align-items: flex-end;
  gap: 4px;
}

.volume-label {
  font-size: 12px;
  color: #9ca3af;
}

.volume-data {
  display: flex;
  flex-direction: column;
  align-items: flex-end;
  gap: 4px;
}

.volume-value {
  font-size: 16px;
  font-weight: 600;
  color: #ffffff;
}

.volume-change {
  font-size: 11px;
  font-weight: 500;
  padding: 1px 4px;
  border-radius: 3px;
  background: rgba(255, 255, 255, 0.05);
}

.volume-change.positive {
  color: #10b981;
}

.volume-change.negative {
  color: #ef4444;
}

.ohlc-data {
  display: flex;
  flex-direction: column;
  gap: 12px;
  margin-top: 8px;
  padding: 12px 0;
  border-top: 1px solid rgba(255, 255, 255, 0.1);
}

.ohlc-row {
  display: flex;
  gap: 16px;
}

.ohlc-item {
  display: flex;
  gap: 4px;
  align-items: center;
}

.ohlc-label {
  font-size: 11px;
  color: #9ca3af;
  font-weight: 600;
  min-width: 15px;
}

.ohlc-value {
  font-size: 11px;
  font-weight: 600;
  color: #ffffff;
  font-family: 'Monaco', 'Menlo', monospace;
}

.ohlc-value.high {
  color: #10b981;
}

.ohlc-value.low {
  color: #ef4444;
}

.ohlc-value.positive {
  color: #10b981;
}

.ohlc-value.negative {
  color: #ef4444;
}

/* Professional Trading Metrics */
.trading-metrics {
  display: grid;
  grid-template-columns: repeat(auto-fit, minmax(100px, 1fr));
  gap: 8px;
  padding: 8px 12px;
  background: rgba(255, 255, 255, 0.02);
  border-radius:  6px;
  border: 1px solid rgba(255, 255, 255, 0.05);
}

.metric-item {
  display: flex;
  justify-content: space-between;
  align-items: center;
  padding: 2px 0;
}

.metric-label {
  font-size: 10px;
  color: #9ca3af;
  font-weight: 500;
  text-transform: uppercase;
  letter-spacing: 0.5px;
}

.metric-value {
  font-size: 10px;
  font-weight: 700;
  font-family: 'Monaco', 'Menlo', monospace;
}

.metric-value.positive {
  color: #10b981;
}

.metric-value.negative {
  color: #ef4444;
}

.metric-value.neutral {
  color: #60a5fa;
}

.chart-wrapper {
  position: relative;
  height: 500px;
  background: linear-gradient(135deg, #0a0a1a 0%, #0f0f23 100%);
  border-radius: 12px;
  overflow: hidden;
  border: 1px solid rgba(255, 255, 255, 0.08);
  box-shadow: 0 8px 32px rgba(0, 0, 0, 0.3);
}

.chart-canvas {
  width: 100%;
  height: 100%;
  cursor: crosshair;
  transition: all 0.3s ease;
}

.chart-canvas:hover {
  transform: scale(1.001);
}

.chart-canvas:active {
  cursor: grabbing;
}

.chart-loading {
  position: absolute;
  top: 50%;
  left: 50%;
  transform: translate(-50%, -50%);
  display: flex;
  flex-direction: column;
  align-items: center;
  gap: 16px;
  z-index: 10;
}

.loading-spinner {
  width: 32px;
  height: 32px;
  border: 3px solid rgba(66, 184, 131, 0.2);
  border-top: 3px solid #42b883;
  border-radius: 50%;
  animation: spin 1s linear infinite;
}

.loading-text {
  color: #9ca3af;
  font-size: 14px;
  font-weight: 500;
}

@keyframes spin {
  0% { transform: rotate(0deg); }
  100% { transform: rotate(360deg); }
}

.chart-tooltip {
  position: absolute;
  background: rgba(26, 27, 35, 0.95);
  backdrop-filter: blur(10px);
  border: 1px solid rgba(255, 255, 255, 0.1);
  border-radius: 8px;
  padding: 12px;
  pointer-events: none;
  z-index: 10;
  min-width: 140px;
}

.tooltip-content {
  display: flex;
  flex-direction: column;
  gap: 8px;
}

.tooltip-time {
  font-size: 12px;
  color: #42b883;
  font-weight: 600;
  text-align: center;
  border-bottom: 1px solid rgba(255, 255, 255, 0.1);
  padding-bottom: 4px;
}

.tooltip-ohlc {
  display: grid;
  grid-template-columns: 1fr 1fr;
  gap: 4px;
}

.ohlc-row {
  display: flex;
  justify-content: space-between;
  align-items: center;
}

.ohlc-label {
  font-size: 11px;
  color: #9ca3af;
  font-weight: 500;
}

.ohlc-value {
  font-size: 11px;
  color: #ffffff;
  font-family: 'Monaco', 'Menlo', monospace;
}

.tooltip-volume {
  display: flex;
  justify-content: space-between;
  align-items: center;
  border-top: 1px solid rgba(255, 255, 255, 0.1);
  padding-top: 4px;
}

/* Right Sidebar - Same Height as Chart */
.right-sidebar {
  display: flex;
  flex-direction: column;
  gap: 20px;
  height: fit-content;
}

.trading-panel {
  background: rgba(255, 255, 255, 0.05);
  backdrop-filter: blur(10px);
  border: 1px solid rgba(255, 255, 255, 0.1);
  border-radius: 16px;
  padding: 20px;
}

.trading-tabs {
  display: flex;
  gap: 2px;
  background: rgba(255, 255, 255, 0.05);
  border-radius: 8px;
  padding: 4px;
  margin-bottom: 16px;
}

.trade-tab {
  flex: 1;
  padding: 8px 16px;
  background: transparent;
  border: none;
  border-radius: 6px;
  color: #9ca3af;
  font-size: 14px;
  font-weight: 500;
  cursor: pointer;
  transition: all 0.2s ease;
  text-align: center;
}

.trade-tab.active {
  background: #42b883;
  color: #ffffff;
}

.trading-header {
  display: flex;
  justify-content: space-between;
  align-items: center;
  margin-bottom: 16px;
}

.balance-info {
  display: flex;
  align-items: center;
  gap: 8px;
  font-size: 14px;
}

.balance-label {
  color: #9ca3af;
}

.balance-value {
  color: #ffffff;
  font-weight: 600;
  font-family: 'Monaco', 'Menlo', monospace;
}

.max-btn {
  padding: 4px 8px;
  background: rgba(66, 184, 131, 0.1);
  border: 1px solid rgba(66, 184, 131, 0.3);
  border-radius: 4px;
  color: #42b883;
  font-size: 12px;
  font-weight: 500;
  cursor: pointer;
  transition: all 0.2s ease;
}

.max-btn:hover {
  background: rgba(66, 184, 131, 0.2);
}

.trade-input-section {
  margin-bottom: 20px;
}

.trade-amount {
  display: flex;
  align-items: center;
  background: rgba(255, 255, 255, 0.05);
  border: 1px solid rgba(255, 255, 255, 0.1);
  border-radius: 8px;
  padding: 12px;
  margin-bottom: 12px;
}

.amount-input {
  flex: 1;
  background: transparent;
  border: none;
  color: #ffffff;
  font-size: 16px;
  font-weight: 600;
  outline: none;
}

.amount-input::placeholder {
  color: #6b7280;
}

.token-selector {
  display: flex;
  align-items: center;
  gap: 4px;
  color: #ffffff;
  font-weight: 600;
}

.place-trade-btn {
  width: 100%;
  padding: 12px;
  border: none;
  border-radius: 8px;
  color: #ffffff;
  font-size: 16px;
  font-weight: 600;
  cursor: pointer;
  transition: all 0.2s ease;
}

.place-trade-btn.buy {
  background: #10b981;
}

.place-trade-btn.buy:hover {
  background: #059669;
  transform: translateY(-1px);
}

.place-trade-btn.sell {
  background: #ef4444;
}

.place-trade-btn.sell:hover {
  background: #dc2626;
  transform: translateY(-1px);
}

.trade-info {
  display: flex;
  flex-direction: column;
  gap: 20px;
}

.progress-section {
  display: flex;
  flex-direction: column;
  gap: 8px;
}

.progress-label {
  font-size: 14px;
  color: #ffffff;
  font-weight: 500;
}

.progress-bar {
  width: 100%;
  height: 8px;
  background: rgba(255, 255, 255, 0.1);
  border-radius: 4px;
  overflow: hidden;
}

.progress-fill {
  height: 100%;
  background: linear-gradient(90deg, #42b883 0%, #10b981 100%);
  border-radius: 4px;
  transition: width 0.3s ease;
}

.progress-description {
  font-size: 12px;
  color: #9ca3af;
  line-height: 1.4;
}

.action-buttons {
  display: flex;
  flex-direction: column;
  gap: 8px;
}

.action-btn {
  display: flex;
  align-items: center;
  justify-content: center;
  gap: 8px;
  padding: 10px 16px;
  background: rgba(255, 255, 255, 0.05);
  border: 1px solid rgba(255, 255, 255, 0.1);
  border-radius: 8px;
  color: #9ca3af;
  font-size: 14px;
  font-weight: 500;
  cursor: pointer;
  transition: all 0.2s ease;
}

.action-btn:hover {
  background: rgba(255, 255, 255, 0.1);
  color: #ffffff;
}

.action-btn.watchlist:hover {
  border-color: rgba(239, 68, 68, 0.3);
  color: #ef4444;
}

.action-btn.twitter:hover {
  border-color: rgba(29, 161, 242, 0.3);
  color: #1da1f2;
}

.contract-info {
  display: flex;
  flex-direction: column;
  gap: 8px;
}

.contract-row {
  display: flex;
  align-items: center;
  gap: 8px;
  font-size: 12px;
  color: #9ca3af;
}

.contract-label {
  color: #6b7280;
}

.contract-value {
  color: #ffffff;
  font-family: 'Monaco', 'Menlo', monospace;
}

.contract-link {
  color: #42b883;
  cursor: pointer;
  text-decoration: none;
}

.contract-link:hover {
  text-decoration: underline;
}

.copy-icon,
.external-icon {
  cursor: pointer;
  transition: color 0.2s ease;
}

.copy-icon:hover,
.external-icon:hover {
  color: #ffffff;
}

.top-holders {
  background: rgba(255, 255, 255, 0.05);
  backdrop-filter: blur(10px);
  border: 1px solid rgba(255, 255, 255, 0.1);
  border-radius: 16px;
  padding: 20px;
}

.holders-header {
  display: flex;
  justify-content: space-between;
  align-items: center;
  margin-bottom: 16px;
}

.holders-title {
  font-size: 16px;
  font-weight: 600;
  color: #ffffff;
}

.bubble-map-btn {
  padding: 6px 12px;
  background: rgba(100, 108, 255, 0.1);
  border: 1px solid rgba(100, 108, 255, 0.3);
  border-radius: 6px;
  color: #646cff;
  font-size: 12px;
  font-weight: 500;
  cursor: pointer;
  transition: all 0.2s ease;
}

.bubble-map-btn:hover {
  background: rgba(100, 108, 255, 0.2);
}

.holders-list {
  display: flex;
  flex-direction: column;
  gap: 8px;
}

.holder-item {
  display: flex;
  align-items: center;
  gap: 8px;
  padding: 8px 0;
  font-size: 14px;
}

.holder-rank {
  color: #6b7280;
  font-weight: 500;
  min-width: 20px;
}

.holder-type {
  color: #ffffff;
  flex: 1;
}

.holder-percentage {
  color: #42b883;
  font-weight: 600;
  font-family: 'Monaco', 'Menlo', monospace;
}

/* Token Information Section */
.token-info-section {
  background: rgba(255, 255, 255, 0.05);
  backdrop-filter: blur(10px);
  border: 1px solid rgba(255, 255, 255, 0.1);
  border-radius: 16px;
  padding: 32px;
  margin-bottom: 32px;
}

.token-info-container {
  display: grid;
  grid-template-columns: 1fr 300px;
  gap: 40px;
  align-items: start;
}

.token-large-image img {
  width: 100%;
  height: 100%;
  object-fit: cover;
}

.token-description-section {
  display: flex;
  flex-direction: column;
  gap: 32px;
}

.token-header-layout {
  display: flex;
  gap: 24px;
  align-items: flex-start;
}

.token-image-section {
  flex-shrink: 0;
}

.token-large-image {
  width: 120px;
  height: 120px;
  border-radius: 16px;
  overflow: hidden;
  box-shadow: 0 10px 25px rgba(0, 0, 0, 0.3);
  border: 2px solid rgba(255, 255, 255, 0.1);
}

.token-info-text {
  flex: 1;
  min-width: 0;
}

.token-description {
  margin-bottom: 0;
}

.token-description {
  margin-bottom: 8px;
}

.description-title {
  font-size: 28px;
  font-weight: 700;
  color: #ffffff;
  margin-bottom: 12px;
  line-height: 1.2;
}

.description-text {
  font-size: 16px;
  color: #d1d5db;
  line-height: 1.7;
  text-align: justify;
}

.social-links-section {
  margin-bottom: 8px;
}

.social-title {
  font-size: 18px;
  font-weight: 600;
  color: #ffffff;
  margin-bottom: 16px;
}

.social-links-grid {
  display: grid;
  grid-template-columns: repeat(auto-fit, minmax(200px, 1fr));
  gap: 12px;
}

.social-link {
  display: flex;
  align-items: center;
  gap: 12px;
  padding: 12px 16px;
  background: rgba(255, 255, 255, 0.05);
  border: 1px solid rgba(255, 255, 255, 0.1);
  border-radius: 12px;
  color: #ffffff;
  text-decoration: none;
  font-size: 14px;
  font-weight: 500;
  transition: all 0.2s ease;
  position: relative;
}

.social-link:hover {
  background: rgba(255, 255, 255, 0.1);
  transform: translateY(-2px);
  box-shadow: 0 8px 16px rgba(0, 0, 0, 0.2);
}

.social-link.website:hover {
  border-color: rgba(100, 108, 255, 0.5);
  box-shadow: 0 8px 16px rgba(100, 108, 255, 0.2);
}

.social-link.twitter:hover {
  border-color: rgba(29, 161, 242, 0.5);
  box-shadow: 0 8px 16px rgba(29, 161, 242, 0.2);
}

.social-link.telegram:hover {
  border-color: rgba(0, 136, 204, 0.5);
  box-shadow: 0 8px 16px rgba(0, 136, 204, 0.2);
}

.social-link.discord:hover {
  border-color: rgba(114, 137, 218, 0.5);
  box-shadow: 0 8px 16px rgba(114, 137, 218, 0.2);
}

.social-link.github:hover {
  border-color: rgba(255, 255, 255, 0.5);
  box-shadow: 0 8px 16px rgba(255, 255, 255, 0.1);
}

.social-link.medium:hover {
  border-color: rgba(0, 171, 107, 0.5);
  box-shadow: 0 8px 16px rgba(0, 171, 107, 0.2);
}

.external-link-icon {
  margin-left: auto;
  opacity: 0.7;
}

.token-stats-section {
  margin-bottom: 0;
}

.stats-title {
  font-size: 18px;
  font-weight: 600;
  color: #ffffff;
  margin-bottom: 16px;
}

.stats-grid {
  display: grid;
  grid-template-columns: repeat(auto-fit, minmax(200px, 1fr));
  gap: 16px;
}

.stat-item {
  display: flex;
  flex-direction: column;
  gap: 8px;
  padding: 16px;
  background: rgba(255, 255, 255, 0.03);
  border: 1px solid rgba(255, 255, 255, 0.08);
  border-radius: 12px;
}

.stat-label {
  font-size: 14px;
  color: #9ca3af;
  font-weight: 500;
}

.stat-value {
  font-size: 18px;
  color: #ffffff;
  font-weight: 700;
  font-family: 'Monaco', 'Menlo', monospace;
}

/* Trades Section - Full Width Below */
.trades-section {
  background: rgba(255, 255, 255, 0.05);
  backdrop-filter: blur(10px);
  border: 1px solid rgba(255, 255, 255, 0.1);
  border-radius: 16px;
  padding: 20px;
  margin-top: 24px;
}

.trades-header {
  margin-bottom: 20px;
}

.trades-tabs {
  display: flex;
  gap: 2px;
  background: rgba(255, 255, 255, 0.05);
  border-radius: 8px;
  padding: 4px;
  margin-bottom: 16px;
  width: fit-content;
}

.trades-tab {
  padding: 8px 16px;
  background: transparent;
  border: none;
  border-radius: 6px;
  color: #9ca3af;
  font-size: 14px;
  font-weight: 500;
  cursor: pointer;
  transition: all 0.2s ease;
}

.trades-tab.active {
  background: #42b883;
  color: #ffffff;
}

.trades-filters {
  display: flex;
  flex-direction: column;
  gap: 12px;
}

.filter-group {
  display: flex;
  align-items: center;
  gap: 8px;
  font-size: 12px;
  color: #9ca3af;
}

.filter-toggle {
  position: relative;
  display: inline-block;
  width: 32px;
  height: 18px;
}

.filter-toggle input {
  opacity: 0;
  width: 0;
  height: 0;
}

.toggle-slider {
  position: absolute;
  cursor: pointer;
  top: 0;
  left: 0;
  right: 0;
  bottom: 0;
  background-color: rgba(255, 255, 255, 0.1);
  transition: 0.2s;
  border-radius: 18px;
}

.toggle-slider:before {
  position: absolute;
  content: "";
  height: 14px;
  width: 14px;
  left: 2px;
  bottom: 2px;
  background-color: #6b7280;
  transition: 0.2s;
  border-radius: 50%;
}

input:checked + .toggle-slider {
  background-color: #42b883;
}

input:checked + .toggle-slider:before {
  transform: translateX(14px);
  background-color: #ffffff;
}

.filter-label {
  color: #ffffff;
  font-weight: 500;
}

.size-input {
  width: 60px;
  padding: 2px 6px;
  background: rgba(255, 255, 255, 0.05);
  border: 1px solid rgba(255, 255, 255, 0.1);
  border-radius: 4px;
  color: #ffffff;
  font-size: 11px;
  outline: none;
}

.size-note,
.following-note,
.own-trades-note {
  color: #6b7280;
  font-style: italic;
}

.trades-table {
  width: 100%;
}

.table-header {
  display: grid;
  grid-template-columns: 2fr 1fr 1fr 1fr 1fr 1.5fr;
  gap: 16px;
  padding: 12px 0;
  border-bottom: 1px solid rgba(255, 255, 255, 0.1);
  margin-bottom: 8px;
}

.header-cell {
  font-size: 12px;
  color: #6b7280;
  font-weight: 500;
  text-transform: lowercase;
}

.table-body {
  display: flex;
  flex-direction: column;
  gap: 8px;
  margin-bottom: 20px;
}

.trade-row {
  display: grid;
  grid-template-columns: 2fr 1fr 1fr 1fr 1fr 1.5fr;
  gap: 16px;
  align-items: center;
  padding: 12px 0;
  border-bottom: 1px solid rgba(255, 255, 255, 0.05);
}

.trade-cell {
  font-size: 14px;
  color: #ffffff;
}

.account-info {
  display: flex;
  align-items: center;
  gap: 8px;
}

.account-avatar {
  width: 24px;
  height: 24px;
  border-radius: 4px;
  overflow: hidden;
  flex-shrink: 0;
}

.account-avatar img {
  width: 100%;
  height: 100%;
  object-fit: cover;
}

.account-name {
  color: #42b883;
  font-weight: 500;
  font-family: 'Monaco', 'Menlo', monospace;
}

.trade-type {
  font-weight: 600;
  text-transform: lowercase;
}

.trade-type.buy {
  color: #10b981;
}

.trade-type.sell {
  color: #ef4444;
}

.warning-icon {
  color: #fbbf24;
  margin-left: 4px;
}

.transaction-link {
  color: #646cff;
  text-decoration: none;
  font-family: 'Monaco', 'Menlo', monospace;
}

.transaction-link:hover {
  text-decoration: underline;
}

.pagination {
  display: flex;
  justify-content: space-between;
  align-items: center;
  padding-top: 16px;
  border-top: 1px solid rgba(255, 255, 255, 0.1);
}

.pagination-btn {
  display: flex;
  align-items: center;
  gap: 8px;
  padding: 8px 16px;
  background: rgba(255, 255, 255, 0.05);
  border: 1px solid rgba(255, 255, 255, 0.1);
  border-radius: 8px;
  color: #9ca3af;
  font-size: 14px;
  font-weight: 500;
  cursor: pointer;
  transition: all 0.2s ease;
}

.pagination-btn:hover:not(:disabled) {
  background: rgba(255, 255, 255, 0.1);
  color: #ffffff;
}

.pagination-btn:disabled {
  opacity: 0.5;
  cursor: not-allowed;
}

.pagination-info {
  display: flex;
  flex-direction: column;
  align-items: center;
  gap: 4px;
}

.page-info {
  font-size: 14px;
  color: #ffffff;
  font-weight: 500;
}

.trades-count {
  font-size: 12px;
  color: #6b7280;
}

/* Mobile Styles */
@media (max-width: 1200px) {
  .main-content {
    grid-template-columns: 1fr 320px;
  }
  
  .token-info-container {
    grid-template-columns: 1fr 280px;
    gap: 30px;
  }
  
  .token-header-layout {
    gap: 20px;
  }
  
  .token-large-image {
    width: 100px;
    height: 100px;
  }
  
  .description-title {
    font-size: 24px;
  }
}

@media (max-width: 1024px) {
  .main-content {
    grid-template-columns: 1fr;
    gap: 20px;
  }
  
  .right-sidebar {
    display: grid;
    grid-template-columns: 1fr 1fr;
    gap: 20px;
  }
  
  .chart-wrapper {
    height: 400px;
  }
  
  .token-info-container {
    grid-template-columns: 1fr;
    gap: 24px;
  }
  
  .token-header-layout {
    flex-direction: column;
    align-items: center;
    text-align: center;
    gap: 20px;
  }
  
  .token-large-image {
    width: 150px;
    height: 150px;
  }
  
  .description-title {
    font-size: 26px;
  }
  
  .social-links-grid {
    grid-template-columns: repeat(auto-fit, minmax(180px, 1fr));
  }
  
  .stats-grid {
    grid-template-columns: repeat(auto-fit, minmax(150px, 1fr));
  }
}

@media (max-width: 768px) {
  .token-chart-page {
    padding: 16px;
  }
  
  .chart-header {
    flex-direction: column;
    align-items: flex-start;
    gap: 12px;
  }
  
  .token-info {
    width: 100%;
  }
  
  .token-name {
    font-size: 20px;
  }
  
  .token-meta {
    flex-direction: column;
    align-items: flex-start;
    gap: 8px;
  }
  
  .chart-controls {
    flex-direction: column;
    gap: 12px;
    align-items: flex-start;
  }
  
  .chart-wrapper {
    height: 300px;
  }
  
  .right-sidebar {
    grid-template-columns: 1fr;
    gap: 16px;
  }
  
  .token-info-section {
    padding: 24px;
  }
  
  .token-header-layout {
    flex-direction: column;
    align-items: center;
    text-align: center;
    gap: 16px;
  }
  
  .token-large-image {
    width: 120px;
    height: 120px;
  }
  
  .description-title {
    font-size: 22px;
  }
  
  .description-text {
    font-size: 15px;
  }
  
  .social-title,
  .stats-title {
    font-size: 16px;
  }
  
  .social-links-grid {
    grid-template-columns: 1fr;
  }
  
  .stats-grid {
    grid-template-columns: repeat(auto-fit, minmax(120px, 1fr));
    gap: 12px;
  }
  
  .stat-item {
    padding: 12px;
  }
  
  .stat-value {
    font-size: 16px;
  }
  
  .table-header,
  .trade-row {
    grid-template-columns: 1fr;
    gap: 8px;
  }
  
  .trade-cell {
    display: flex;
    justify-content: space-between;
    align-items: center;
    padding: 4px 0;
  }
  
  .trade-cell:before {
    content: attr(data-label);
    font-size: 12px;
    color: #6b7280;
    font-weight: 500;
  }
  
  .pagination {
    flex-direction: column;
    gap: 12px;
  }
  
  .pagination-info {
    order: -1;
  }
}

@media (max-width: 480px) {
  .token-info-section {
    padding: 20px;
  }
  
  .token-large-image {
    width: 140px;
    height: 140px;
  }
  
  .description-title {
    font-size: 18px;
  }
  
  .description-text {
    font-size: 14px;
  }
  
  .social-title,
  .stats-title {
    font-size: 15px;
  }
  
  .social-link {
    padding: 10px 12px;
    font-size: 13px;
  }
  
  .stats-grid {
    grid-template-columns: 1fr;
  }
  
  .stat-item {
    padding: 10px;
  }
  
  .stat-value {
    font-size: 14px;
  }
}
</style>