<script setup>
import { ref, computed } from 'vue';
import { TrendingUp, TrendingDown, Wallet } from 'lucide-vue-next';

const props = defineProps({
  currentPrice: Number,
  balance: Number,
  history: Array,
  symbol: String,
  currentTime: Number
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
    timestamp: props.currentTime,
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
    exitTimestamp: props.currentTime,
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
      <div class="header-title">
        <Wallet size="14" class="icon-wallet" />
        <span>交易终端</span>
      </div>
      <div class="symbol-badge">{{ symbol }}</div>
    </div>
    
    <div class="panel-body">
      <div class="input-section">
        <div class="input-row">
          <label>杠杆倍数</label>
          <div class="leverage-grid">
            <button 
              v-for="opt in leverageOptions" 
              :key="opt"
              :class="{ active: leverage === opt }"
              @click="leverage = opt"
            >{{ opt }}x</button>
          </div>
        </div>

        <div class="input-row">
          <label>下单金额 (USDT)</label>
          <div class="amount-input-wrapper">
            <input v-model.number="amount" type="number" step="10" min="10" class="amount-input">
            <div class="input-suffix">USDT</div>
          </div>
        </div>
      </div>

      <div v-if="!currentPosition" class="action-buttons">
        <button class="btn-trade btn-long" @click="openPosition('long')" :disabled="balance < amount">
          <TrendingUp size="16" /> <span>买入 / 做多</span>
        </button>
        <button class="btn-trade btn-short" @click="openPosition('short')" :disabled="balance < amount">
          <TrendingDown size="16" /> <span>卖出 / 做空</span>
        </button>
      </div>

      <div v-else class="position-info">
        <div class="pos-status" :class="currentPosition.type">
          <div class="pos-type">
            {{ currentPosition.type === 'long' ? '多单持仓' : '空单持仓' }} 
            <span class="lev-text">{{ currentPosition.leverage }}x</span>
          </div>
          <div class="pos-pnl-pct" :class="{ plus: pnl >= 0, minus: pnl < 0 }">
            {{ pnl >= 0 ? '+' : '' }}{{ ((pnl / currentPosition.amount) * 100).toFixed(2) }}%
          </div>
        </div>
        <div class="pos-details">
          <div class="detail-item">
            <span class="label">开仓价格</span>
            <span class="value">{{ currentPosition.entryPrice.toFixed(2) }}</span>
          </div>
          <div class="detail-item">
            <span class="label">当前价格</span>
            <span class="value highlight">{{ currentPrice.toFixed(2) }}</span>
          </div>
          <div class="detail-item pnl-main">
            <span class="label">未实现盈亏</span>
            <span class="value" :class="{ plus: pnl >= 0, minus: pnl < 0 }">
              {{ pnl >= 0 ? '+' : '' }}{{ pnl.toFixed(2) }} <small>USDT</small>
            </span>
          </div>
        </div>
        <button class="btn-close" @click="closePosition">立即平仓</button>
      </div>
    </div>
  </div>
</template>

<style scoped>
.order-panel {
  display: flex;
  flex-direction: column;
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

.icon-wallet { color: #2962ff; }

.symbol-badge {
  background: rgba(41, 98, 255, 0.1);
  color: #2962ff;
  padding: 2px 6px;
  border-radius: 3px;
  font-size: 10px;
  font-weight: 600;
}

.panel-body {
  padding: 16px;
}

.input-section {
  display: flex;
  flex-direction: column;
  gap: 16px;
  margin-bottom: 20px;
}

.input-row label {
  display: block;
  font-size: 10px;
  font-weight: 700;
  color: #8a8d97;
  margin-bottom: 8px;
  letter-spacing: 0.5px;
}

.leverage-grid {
  display: grid;
  grid-template-columns: repeat(5, 1fr);
  gap: 4px;
}

.leverage-grid button {
  padding: 6px 0;
  font-size: 11px;
  background: #2a2e39;
  border: 1px solid #363a45;
  color: #d1d4dc;
  border-radius: 4px;
  cursor: pointer;
  transition: all 0.2s;
}

.leverage-grid button:hover { border-color: #4a4e5a; }
.leverage-grid button.active {
  background: #2962ff;
  border-color: #2962ff;
  color: #fff;
  font-weight: 600;
}

.amount-input-wrapper {
  position: relative;
  display: flex;
  align-items: center;
}

.amount-input {
  width: 100%;
  background: #2a2e39;
  border: 1px solid #363a45;
  color: #fff;
  padding: 10px 12px;
  border-radius: 4px;
  font-size: 13px;
  outline: none;
  font-family: 'JetBrains Mono', monospace;
}

.amount-input:focus { border-color: #2962ff; }

.input-suffix {
  position: absolute;
  right: 12px;
  font-size: 10px;
  color: #8a8d97;
  font-weight: 600;
}

.action-buttons {
  display: grid;
  grid-template-columns: 1fr 1fr;
  gap: 12px;
}

.btn-trade {
  height: 42px;
  border-radius: 6px;
  border: none;
  font-weight: 700;
  cursor: pointer;
  display: flex;
  flex-direction: column;
  align-items: center;
  justify-content: center;
  gap: 2px;
  color: #fff;
  transition: opacity 0.2s;
}

.btn-trade span { font-size: 10px; }
.btn-trade:hover { opacity: 0.9; }
.btn-trade:disabled { opacity: 0.3; cursor: not-allowed; }

.btn-long { background-color: #089981; }
.btn-short { background-color: #f23645; }

.position-info {
  background: #131722;
  border-radius: 8px;
  overflow: hidden;
  border: 1px solid #2a2e39;
}

.pos-status {
  padding: 10px 16px;
  display: flex;
  justify-content: space-between;
  align-items: center;
  font-weight: 700;
  font-size: 12px;
}

.pos-status.long { background: rgba(8, 153, 129, 0.1); color: #089981; }
.pos-status.short { background: rgba(242, 54, 69, 0.1); color: #f23645; }

.lev-text { opacity: 0.7; margin-left: 4px; font-weight: normal; }

.pos-details {
  padding: 12px 16px;
  display: flex;
  flex-direction: column;
  gap: 8px;
}

.detail-item {
  display: flex;
  justify-content: space-between;
  font-size: 12px;
}

.detail-item .label { color: #8a8d97; }
.detail-item .value { color: #d1d4dc; font-family: 'JetBrains Mono', monospace; }
.detail-item .value.highlight { color: #fff; font-weight: 600; }

.pnl-main {
  margin-top: 4px;
  padding-top: 8px;
  border-top: 1px dashed #2a2e39;
}

.pnl-main .value { font-size: 14px; font-weight: 700; }
.pnl-main .value small { font-size: 10px; opacity: 0.7; }

.value.plus { color: #089981; }
.value.minus { color: #f23645; }

.btn-close {
  width: 100%;
  padding: 12px;
  background: #2a2e39;
  border: none;
  color: #fff;
  font-size: 11px;
  font-weight: 700;
  cursor: pointer;
  letter-spacing: 1px;
}

.btn-close:hover { background: #363a45; }
</style>
