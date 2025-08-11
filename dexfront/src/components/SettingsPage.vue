<template>
  <div class="settings-page">
    <div class="settings-container">
      <h1 class="settings-title">Settings</h1>
      <p class="settings-description">Configure your application preferences</p>
      
      <div class="settings-sections">
        <div class="settings-section">
          <h3>Theme</h3>
          <p>Customize the appearance of your application</p>
          <div class="setting-option">
            <label>Dark Mode</label>
            <input type="checkbox" checked disabled />
          </div>
        </div>

        <div class="settings-section">
          <h3>Notifications</h3>
          <p>Manage your notification preferences</p>
          <div class="setting-option">
            <label>Transaction Alerts</label>
            <input type="checkbox" checked />
          </div>
          <div class="setting-option">
            <label>Price Alerts</label>
            <input type="checkbox" />
          </div>
        </div>

        <div class="settings-section">
          <h3>Network</h3>
          <p>Configure network settings</p>
          <div class="setting-option">
            <label>Network</label>
            <select>
              <option value="devnet">Devnet</option>
              <option value="mainnet">Mainnet</option>
            </select>
          </div>
        </div>
      </div>
    </div>
  </div>
</template>

<script setup>
// Settings component logic can be added here
</script>

<style scoped>
.settings-page {
  padding: 20px 0;
}

.settings-container {
  max-width: 800px;
  margin: 0 auto;
  background: rgba(255, 255, 255, 0.05);
  backdrop-filter: blur(20px);
  border: 1px solid rgba(255, 255, 255, 0.1);
  border-radius: 20px;
  padding: 40px;
}

.settings-title {
  font-size: 32px;
  font-weight: 700;
  color: #ffffff;
  margin-bottom: 8px;
}

.settings-description {
  font-size: 16px;
  color: #9ca3af;
  margin-bottom: 40px;
}

.settings-sections {
  display: flex;
  flex-direction: column;
  gap: 32px;
}

.settings-section {
  padding: 24px;
  background: rgba(255, 255, 255, 0.03);
  border-radius: 16px;
  border: 1px solid rgba(255, 255, 255, 0.08);
}

.settings-section h3 {
  font-size: 20px;
  font-weight: 600;
  color: #ffffff;
  margin-bottom: 8px;
}

.settings-section p {
  font-size: 14px;
  color: #9ca3af;
  margin-bottom: 20px;
}

.setting-option {
  display: flex;
  justify-content: space-between;
  align-items: center;
  padding: 12px 0;
  border-bottom: 1px solid rgba(255, 255, 255, 0.05);
}

.setting-option:last-child {
  border-bottom: none;
}

.setting-option label {
  font-size: 14px;
  color: #ffffff;
  font-weight: 500;
}

.setting-option input[type="checkbox"] {
  width: 20px;
  height: 20px;
  accent-color: #42b883;
}

.setting-option select {
  padding: 8px 12px;
  background: rgba(255, 255, 255, 0.05);
  border: 1px solid rgba(255, 255, 255, 0.1);
  border-radius: 8px;
  color: #ffffff;
  font-size: 14px;
  outline: none;
}

.setting-option select:focus {
  border-color: #42b883;
}

/* Mobile Styles */
@media (max-width: 768px) {
  .settings-container {
    padding: 24px;
    margin: 0 16px;
  }
  
  .settings-title {
    font-size: 24px;
  }
  
  .settings-sections {
    gap: 24px;
  }
  
  .settings-section {
    padding: 20px;
  }
}
</style>
