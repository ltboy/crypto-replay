<script setup>
import { ref, computed } from 'vue';
import { TrendingUp, TrendingDown, Wallet } from 'lucide-vue-next';

const props = defineProps({
  currentPrice: Number,
  balance: Number,
  history: Array,
  symbol: String
});

const emit = defineEmits(['update:balance', 'update:history']);

const leverage = ref(1);
const amount = ref(100); // 默认投入 100 USDT
const currentPosition = ref(null);

const maxLeverage = 10;
const leverageOptions = [1, 2, 3, 5, 10];

const canOpen = computed(() => props.currentPrice > 0 && props.balance >= amount.value && !currentPosition.value);

const pnl = computed(() => {
  if (!currentPosition.value) return 0;
  const diff = props.currentPrice - currentPosition.value.entryPrice;
  const rawPnl = currentPosition.value.type === 'long' ? diff : -diff;
  return (rawPnl / currentPosition.value.entryPrice) * currentPosition.value.amount * currentPosition.value.leverage;
});

defineExpose({
  currentPosition
});

function openPosition(type) {
  if (props.balance < amount.value) return;
  
  currentPosition.value = {
    type,
    entryPrice: props.currentPrice,
    leverage: leverage.value,
    amount: amount.value,
    timestamp: Date.now(),
    symbol: props.symbol
  };
  
  emit('update:balance', props.balance - amount.value);
}

function closePosition() {
  if (!currentPosition.value) return;
  
  const finalPnl = pnl.value;
  const finalAmount = currentPosition.value.amount + finalPnl;
  
  const record = {
    ...currentPosition.value,
    exitPrice: props.currentPrice,
    exitTimestamp: Date.now(),
    pnl: finalPnl,
    roi: (finalPnl / currentPosition.value.amount) * 100
  };
  
  emit('update:balance', props.balance + finalAmount);
  emit('update:history', [record, ...props.history]);
  
  currentPosition.value = null;
}
</script>

<template>
  <div class="order-panel">
    <div class="panel-header">
      <Wallet size="16" />
      <span>开仓系统</span>
    </div>
    
    <div class="panel-body">
      <div class="input-group">
        <label>杠杆 (1-10x)</label>
        <div class="leverage-selector">
          <button 
            v-for="opt in leverageOptions" 
            :key="opt"
            :class="{ active: leverage === opt }"
            @click="leverage = opt"
          >{{ opt }}x</button>
        </div>
      </div>

      <div class="input-group">
        <label>投入金额 (USDT)</label>
        <input v-model.number="amount" type="number" step="10" min="10" class="amount-input">
      </div>

      <div v-if="!currentPosition" class="action-buttons">
        <button class="btn-long" @click="openPosition('long')" :disabled="balance < amount">
          <TrendingUp size="16" /> 开多
        </button>
        <button class="btn-short" @click="openPosition('short')" :disabled="balance < amount">
          <TrendingDown size="16" /> 开空
        </button>
      </div>

      <div v-else class="position-info">
        <div class="pos-header" :class="currentPosition.type">
          {{ currentPosition.type === 'long' ? '多单' : '空单' }} {{ currentPosition.leverage }}x
        </div>
        <div class="pos-details">
          <div class="detail-row">
            <span>开仓价:</span>
            <span>{{ currentPosition.entryPrice.toFixed(2) }}</span>
          </div>
          <div class="detail-row">
            <span>当前价:</span>
            <span>{{ currentPrice.toFixed(2) }}</span>
          </div>
          <div class="detail-row pnl-row" :class="{ plus: pnl >= 0, minus: pnl < 0 }">
            <span>未实现盈亏:</span>
            <span>{{ pnl >= 0 ? '+' : '' }}{{ pnl.toFixed(2) }} USDT</span>
          </div>
        </div>
        <button class="btn-close" @click="closePosition">平仓</button>
      </div>
    </div>
  </div>
</template>

<style scoped>
.order-panel {
  padding: 15px;
  border-bottom: 1px solid #2a2e39;
}

.panel-header {
  display: flex;
  align-items: center;
  gap: 8px;
  font-weight: bold;
  margin-bottom: 15px;
  color: #2962ff;
}

.input-group {
  margin-bottom: 15px;
}

.input-group label {
  display: block;
  font-size: 0.8rem;
  color: #8a8d97;
  margin-bottom: 5px;
}

.leverage-selector {
  display: flex;
  gap: 5px;
}

.leverage-selector button {
  flex: 1;
  padding: 4px;
  font-size: 0.8rem;
  background: #2a2e39;
  border: 1px solid #363a45;
  color: #d1d4dc;
  border-radius: 4px;
}

.leverage-selector button.active {
  background: #2962ff;
  border-color: #2962ff;
  color: white;
}

.amount-input {
  width: 100%;
  background: #2a2e39;
  border: 1px solid #363a45;
  color: white;
  padding: 8px;
  border-radius: 4px;
}

.action-buttons {
  display: flex;
  gap: 10px;
}

.action-buttons button {
  flex: 1;
  padding: 10px;
  border-radius: 4px;
  border: none;
  font-weight: bold;
  cursor: pointer;
  display: flex;
  align-items: center;
  justify-content: center;
  gap: 5px;
  color: white;
}

.btn-long { background-color: #26a69a; }
.btn-short { background-color: #ef5350; }
.btn-long:disabled, .btn-short:disabled { opacity: 0.5; cursor: not-allowed; }

.position-info {
  background: #2a2e39;
  border-radius: 4px;
  overflow: hidden;
}

.pos-header {
  padding: 8px;
  text-align: center;
  font-weight: bold;
}

.pos-header.long { background: rgba(38, 166, 154, 0.2); color: #26a69a; }
.pos-header.short { background: rgba(239, 83, 80, 0.2); color: #ef5350; }

.pos-details {
  padding: 10px;
}

.detail-row {
  display: flex;
  justify-content: space-between;
  font-size: 0.9rem;
  margin-bottom: 5px;
}

.pnl-row {
  margin-top: 10px;
  font-weight: bold;
}

.plus { color: #26a69a; }
.minus { color: #ef5350; }

.btn-close {
  width: 100%;
  padding: 10px;
  background: #363a45;
  border: none;
  color: white;
  font-weight: bold;
  cursor: pointer;
}

.btn-close:hover { background: #454955; }
</style>
