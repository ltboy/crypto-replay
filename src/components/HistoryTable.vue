<script setup>
import { History } from 'lucide-vue-next';

defineProps({
  history: Array
});

function formatDate(timestamp) {
  return new Date(timestamp).toLocaleString();
}
</script>

<template>
  <div class="history-panel">
    <div class="panel-header">
      <History size="16" />
      <span>复盘交易记录</span>
    </div>
    
    <div class="history-list">
      <div v-if="history.length === 0" class="empty-state">
        暂无交易记录
      </div>
      <div v-for="(item, idx) in history" :key="idx" class="history-item">
        <div class="item-header">
          <span :class="['type-badge', item.type]">{{ item.type === 'long' ? '多' : '空' }}</span>
          <span class="symbol-text">{{ item.symbol }} {{ item.leverage }}x</span>
          <span :class="['pnl-text', item.pnl >= 0 ? 'plus' : 'minus']">
            {{ item.pnl >= 0 ? '+' : '' }}{{ item.pnl.toFixed(2) }}
          </span>
        </div>
        <div class="item-details">
          <div class="detail-line">
            <span>开: {{ item.entryPrice.toFixed(2) }}</span>
            <span>平: {{ item.exitPrice.toFixed(2) }}</span>
          </div>
          <div class="time-text">{{ formatDate(item.exitTimestamp) }}</div>
        </div>
      </div>
    </div>
  </div>
</template>

<style scoped>
.history-panel {
  flex: 1;
  display: flex;
  flex-direction: column;
  overflow: hidden;
}

.panel-header {
  padding: 15px;
  display: flex;
  align-items: center;
  gap: 8px;
  font-weight: bold;
  color: #2962ff;
  border-bottom: 1px solid #2a2e39;
}

.history-list {
  flex: 1;
  overflow-y: auto;
  padding: 10px;
}

.empty-state {
  text-align: center;
  color: #4a4e5a;
  margin-top: 40px;
  font-size: 0.9rem;
}

.history-item {
  background: #2a2e39;
  border-radius: 4px;
  padding: 10px;
  margin-bottom: 10px;
  font-size: 0.85rem;
}

.item-header {
  display: flex;
  align-items: center;
  gap: 8px;
  margin-bottom: 5px;
}

.type-badge {
  padding: 2px 6px;
  border-radius: 2px;
  font-size: 0.7rem;
  font-weight: bold;
  color: white;
}

.type-badge.long { background: #26a69a; }
.type-badge.short { background: #ef5350; }

.symbol-text {
  font-weight: 500;
  flex: 1;
}

.pnl-text {
  font-weight: bold;
}

.plus { color: #26a69a; }
.minus { color: #ef5350; }

.item-details {
  color: #8a8d97;
}

.detail-line {
  display: flex;
  justify-content: space-between;
  margin-bottom: 2px;
}

.time-text {
  font-size: 0.75rem;
  opacity: 0.7;
}
</style>
