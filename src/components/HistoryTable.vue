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
      <div class="header-title">
        <History size="14" class="icon-history" />
        <span>交易历史</span>
      </div>
      <div class="trade-count" v-if="history.length > 0">{{ history.length }} 笔交易</div>
    </div>
    
    <div class="history-list">
      <div v-if="history.length === 0" class="empty-state">
        <div class="empty-icon">📂</div>
        <p>暂无交易记录</p>
      </div>
      <div v-for="(item, idx) in history" :key="idx" class="history-item">
        <div class="item-main">
          <div class="item-left">
            <span :class="['type-badge', item.type]">{{ item.type === 'long' ? '多' : '空' }}</span>
            <div class="item-info">
              <div class="item-symbol">{{ item.symbol }} <span class="lev">{{ item.leverage }}x</span></div>
              <div class="item-time">{{ formatDate(item.exitTimestamp) }}</div>
            </div>
          </div>
          <div class="item-right">
            <div :class="['pnl-value', item.pnl >= 0 ? 'plus' : 'minus']">
              {{ item.pnl >= 0 ? '+' : '' }}{{ item.pnl.toFixed(2) }}
            </div>
            <div class="roi-value" :class="item.pnl >= 0 ? 'plus' : 'minus'">
              {{ item.roi.toFixed(2) }}%
            </div>
          </div>
        </div>
        <div class="item-footer">
          <div class="price-row">
            <span class="price-tag">进场</span>
            <span class="price-val">{{ item.entryPrice.toFixed(2) }}</span>
            <span class="price-sep">→</span>
            <span class="price-tag">离场</span>
            <span class="price-val">{{ item.exitPrice.toFixed(2) }}</span>
          </div>
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
  background-color: #1e222d;
}

.panel-header {
  padding: 12px 16px;
  display: flex;
  align-items: center;
  justify-content: space-between;
  border-bottom: 1px solid #2a2e39;
}

.header-title {
  display: flex;
  align-items: center;
  gap: 8px;
  font-size: 11px;
  font-weight: 700;
  color: #8a8d97;
  letter-spacing: 0.5px;
}

.icon-history { color: #2962ff; }

.trade-count {
  font-size: 9px;
  font-weight: 600;
  color: #8a8d97;
  background: rgba(255, 255, 255, 0.05);
  padding: 2px 6px;
  border-radius: 10px;
}

.history-list {
  flex: 1;
  overflow-y: auto;
  padding: 8px;
}

/* Custom scrollbar */
.history-list::-webkit-scrollbar { width: 4px; }
.history-list::-webkit-scrollbar-track { background: transparent; }
.history-list::-webkit-scrollbar-thumb { background: #363a45; border-radius: 2px; }

.empty-state {
  display: flex;
  flex-direction: column;
  align-items: center;
  justify-content: center;
  height: 100%;
  color: #4a4e5a;
  opacity: 0.6;
}

.empty-icon { font-size: 24px; margin-bottom: 8px; }
.empty-state p { font-size: 12px; font-weight: 500; }

.history-item {
  background: #131722;
  border: 1px solid #2a2e39;
  border-radius: 6px;
  padding: 10px;
  margin-bottom: 8px;
  transition: border-color 0.2s;
}

.history-item:hover { border-color: #363a45; }

.item-main {
  display: flex;
  justify-content: space-between;
  align-items: flex-start;
  margin-bottom: 8px;
}

.item-left {
  display: flex;
  gap: 10px;
}

.type-badge {
  width: 18px;
  height: 18px;
  display: flex;
  align-items: center;
  justify-content: center;
  border-radius: 3px;
  font-size: 10px;
  font-weight: 800;
  color: #fff;
}

.type-badge.long { background: #089981; }
.type-badge.short { background: #f23645; }

.item-info {
  display: flex;
  flex-direction: column;
}

.item-symbol {
  font-size: 13px;
  font-weight: 600;
  color: #d1d4dc;
}

.item-symbol .lev {
  font-size: 10px;
  color: #8a8d97;
  margin-left: 4px;
  font-weight: normal;
}

.item-time {
  font-size: 10px;
  color: #8a8d97;
  margin-top: 2px;
}

.item-right {
  text-align: right;
}

.pnl-value {
  font-size: 13px;
  font-weight: 700;
  font-family: 'JetBrains Mono', monospace;
}

.roi-value {
  font-size: 10px;
  font-weight: 600;
  margin-top: 2px;
}

.plus { color: #089981; }
.minus { color: #f23645; }

.item-footer {
  border-top: 1px solid #2a2e39;
  padding-top: 8px;
}

.price-row {
  display: flex;
  align-items: center;
  gap: 6px;
  font-size: 11px;
}

.price-tag { color: #8a8d97; font-size: 9px; text-transform: uppercase; }
.price-val { color: #d1d4dc; font-family: 'JetBrains Mono', monospace; }
.price-sep { color: #363a45; }
</style>
