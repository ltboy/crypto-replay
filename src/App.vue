<script setup>
import { ref, reactive, onMounted, watch, computed } from 'vue';
import ReplayChart from './components/ReplayChart.vue';
import OrderPanel from './components/OrderPanel.vue';
import HistoryTable from './components/HistoryTable.vue';
import EquityChart from './components/EquityChart.vue';
import { Play, SkipForward, RotateCcw, Dice5, Pause, Pencil, Minus, Square, Ratio, TrendingUp as TrendingUpIcon, Trash2 } from 'lucide-vue-next';

// 状态管理
const state = reactive({
  symbol: 'BTCUSDT',
  interval: '1h',
  allData: [],
  currentIndex: 0,
  isPlaying: false,
  playSpeed: 500,
  currentPrice: 0,
  currentTime: 0,
  balance: 10000, // 初始资金
  initialBalance: 10000,
  balanceHistory: [{ time: Date.now(), balance: 10000 }],
  orders: [],
  history: [],
});

const totalPnl = computed(() => state.balance - state.initialBalance);

// 计算涨幅和波幅
const marketInfo = computed(() => {
  if (state.currentIndex <= 0 || state.allData.length === 0) return { change: 0, amplitude: 0 };
  const currentK = state.allData[state.currentIndex - 1];
  const prevK = state.currentIndex > 1 ? state.allData[state.currentIndex - 2] : currentK;
  
  const change = ((currentK.close - prevK.close) / prevK.close) * 100;
  const amplitude = ((currentK.high - currentK.low) / currentK.low) * 100;
  
  return {
    change,
    amplitude
  };
});

// 监听余额变化记录历史
watch(() => state.balance, (newVal) => {
  state.balanceHistory.push({
    time: Date.now(),
    balance: newVal
  });
  if (state.balanceHistory.length > 50) state.balanceHistory.shift();
});

const chartRef = ref(null);
const orderPanelRef = ref(null);
let playIntervalId = null;

// 监听开仓状态，绘制入场线
watch(() => orderPanelRef.value?.currentPosition, (newPos) => {
  if (newPos) {
    chartRef.value?.createOverlay({
      name: 'horizontalRayLine',
      id: 'entry-line',
      points: [
        { value: newPos.entryPrice, timestamp: newPos.timestamp },
        { value: newPos.entryPrice, timestamp: newPos.timestamp + 1000000000 } // 强制向右延伸
      ],
      styles: {
        line: {
          color: newPos.type === 'long' ? '#26a69a' : '#ef5350',
          size: 2,
          style: 'dashed'
        }
      },
      lock: true
    });
  } else {
    chartRef.value?.removeOverlay('entry-line');
  }
}, { deep: true });

// 获取币安数据
const intervalToMs = (interval) => {
  const unit = interval.slice(-1);
  const value = parseInt(interval.slice(0, -1));
  switch (unit) {
    case 'm': return value * 60 * 1000;
    case 'h': return value * 60 * 60 * 1000;
    case 'd': return value * 24 * 60 * 60 * 1000;
    case 'w': return value * 7 * 24 * 60 * 60 * 1000;
    default: return 60 * 1000;
  }
};

async function fetchBinanceData(symbol, interval, startTime = null, endTime = null, limit = 1000) {
  let url = `https://api.binance.com/api/v3/klines?symbol=${symbol.toUpperCase()}&interval=${interval}&limit=${limit}`;
  if (startTime) url += `&startTime=${startTime}`;
  if (endTime) url += `&endTime=${endTime}`;

  try {
    const response = await fetch(url);
    const rawData = await response.json();
    return rawData.map((item) => ({
      timestamp: item[0],
      open: parseFloat(item[1]),
      high: parseFloat(item[2]),
      low: parseFloat(item[3]),
      close: parseFloat(item[4]),
      volume: parseFloat(item[5])
    }));
  } catch (error) {
    console.error('Fetch error:', error);
    return [];
  }
}

// 随机加载数据
async function loadRandom() {
  stopPlay();
  const minTime = 1514764800000; // 2018
  const maxTime = Date.now() - (1000 * 60 * 60 * 24 * 30);
  const startTime = Math.floor(Math.random() * (maxTime - minTime) + minTime);
  
  const data = await fetchBinanceData(state.symbol, state.interval, startTime);
  if (data.length > 0) {
    state.allData = data;
    state.currentIndex = Math.floor(data.length * 0.5); // 从中间开始
    const visibleData = state.allData.slice(0, state.currentIndex);
    chartRef.value?.setNewData(visibleData);
    updateCurrentInfo();
  }
}

// 切换周期时保持时间不变
async function handleIntervalChange() {
  const currentTime = state.currentTime;
  const currentPrice = state.currentPrice;
  if (!currentTime) return;
  
  stopPlay();
  const startTime = currentTime - (intervalToMs(state.interval) * 800);
  const data = await fetchBinanceData(state.symbol, state.interval, startTime);
  
  if (data.length > 0) {
    state.allData = data;
    
    // 找到当前时间点所在的 K 线索引
    // 逻辑：找到第一个 timestamp >= currentTime 的索引
    let targetIndex = data.findIndex(item => item.timestamp >= currentTime);
    
    if (targetIndex === -1) targetIndex = data.length - 1;

    // 关键优化：如果当前处于大周期的中间（currentTime 还没到该 K 线的结束时间）
    // 我们需要保留当前的价格作为该 K 线的 Close
    const targetK = data[targetIndex];
    if (targetK.timestamp <= currentTime) {
       // 如果正在大周期内部，更新该 K 线的临时最高/最低/收盘
       targetK.close = currentPrice;
       if (currentPrice > targetK.high) targetK.high = currentPrice;
       if (currentPrice < targetK.low) targetK.low = currentPrice;
       state.currentIndex = targetIndex + 1;
    } else {
       state.currentIndex = targetIndex;
    }
    
    chartRef.value?.setNewData(state.allData.slice(0, state.currentIndex));
    updateCurrentInfo();
  }
}

async function handleLoadMore(timestamp) {
  const data = await fetchBinanceData(state.symbol, state.interval, null, timestamp - 1);
  if (data.length > 0) {
    // 将新数据插入到 allData 开头
    state.allData = [...data, ...state.allData];
    // 更新 currentIndex，因为前面增加了数据
    state.currentIndex += data.length;
    chartRef.value?.applyMoreData(data);
  }
}

function updateCurrentInfo() {
  if (state.currentIndex > 0) {
    const currentK = state.allData[state.currentIndex - 1];
    state.currentPrice = currentK.close;
    state.currentTime = currentK.timestamp;
    
    // 如果有开仓，更新未实现盈亏历史（用于收益曲线）
    if (orderPanelRef.value?.currentPosition) {
      // 可以在这里逻辑扩展，目前简化为只在余额变动时记录曲线
    }
  }
}

function nextK() {
  if (state.currentIndex < state.allData.length) {
    const k = state.allData[state.currentIndex];
    chartRef.value?.updateData(k);
    state.currentIndex++;
    updateCurrentInfo();
  } else {
    stopPlay();
  }
}

function togglePlay() {
  if (state.isPlaying) stopPlay();
  else startPlay();
}

function startPlay() {
  state.isPlaying = true;
  playIntervalId = setInterval(nextK, state.playSpeed);
}

function stopPlay() {
  state.isPlaying = false;
  if (playIntervalId) {
    clearInterval(playIntervalId);
    playIntervalId = null;
  }
}

function reset() {
  stopPlay();
  state.allData = [];
  state.currentIndex = 0;
  state.orders = [];
  state.history = [];
  state.balance = 10000;
  chartRef.value?.clear();
}

onMounted(() => {
  loadRandom();
});
</script>

<template>
  <div class="app-container">
    <header class="navbar">
      <div class="logo">CryptoReplay Pro</div>
      <div class="market-status" v-if="state.currentIndex > 0">
        <span class="symbol">{{ state.symbol }}</span>
        <span class="price">{{ state.currentPrice.toFixed(2) }}</span>
        <span :class="['change', marketInfo.change >= 0 ? 'plus' : 'minus']">
          {{ marketInfo.change >= 0 ? '+' : '' }}{{ marketInfo.change.toFixed(2) }}%
        </span>
        <span class="amplitude">波幅: {{ marketInfo.amplitude.toFixed(2) }}%</span>
      </div>
      <div class="controls">
        <input v-model="state.symbol" placeholder="BTCUSDT" class="input-symbol" @keyup.enter="loadRandom">
        <select v-model="state.interval" @change="handleIntervalChange" class="select-interval">
          <option value="1m">1m</option>
          <option value="5m">5m</option>
          <option value="15m">15m</option>
          <option value="1h">1h</option>
          <option value="4h">4h</option>
          <option value="1d">1d</option>
        </select>
        <button @click="loadRandom" title="随机历史时刻"><Dice5 size="18" /></button>
        <div class="divider"></div>
        <button @click="chartRef?.createOverlay('segment')" title="趋势线"><Pencil size="18" /></button>
        <button @click="chartRef?.createOverlay('horizontalRayLine')" title="水平向右射线"><Minus size="18" /></button>
        <button @click="chartRef?.createOverlay('rect')" title="矩形"><Square size="18" /></button>
        <button @click="chartRef?.createOverlay('fibonacciLine')" title="斐波那契回调"><Ratio size="18" /></button>
        <button @click="chartRef?.removeOverlay()" title="清除所有画线"><Trash2 size="18" /></button>
        <div class="divider"></div>
        <button @click="nextK" title="下一根K线"><SkipForward size="18" /></button>
        <button @click="togglePlay" :class="{ active: state.isPlaying }">
          <Pause v-if="state.isPlaying" size="18" />
          <Play v-else size="18" />
        </button>
        <button @click="reset" title="重置"><RotateCcw size="18" /></button>
      </div>
      <div class="account-info">
        <span>余额: ${{ state.balance.toFixed(2) }}</span>
        <span :class="['total-pnl', totalPnl >= 0 ? 'plus' : 'minus']">
          ({{ totalPnl >= 0 ? '+' : '' }}{{ totalPnl.toFixed(2) }})
        </span>
      </div>
    </header>

    <main class="main-content">
      <div class="chart-section">
        <ReplayChart ref="chartRef" @loadMore="handleLoadMore" />
      </div>
      <aside class="side-panel">
        <OrderPanel 
          ref="orderPanelRef"
          :currentPrice="state.currentPrice" 
          v-model:balance="state.balance"
          v-model:history="state.history"
          :symbol="state.symbol"
        />
        <HistoryTable :history="state.history" />
        <EquityChart :data="state.balanceHistory" />
      </aside>
    </main>
  </div>
</template>

<style scoped>
.app-container {
  display: flex;
  flex-direction: column;
  height: 100vh;
  background-color: #131722;
  color: #d1d4dc;
}

.navbar {
  height: 50px;
  background-color: #1e222d;
  display: flex;
  align-items: center;
  padding: 0 20px;
  gap: 20px;
  border-bottom: 1px solid #2a2e39;
}

.logo {
  font-weight: bold;
  font-size: 1.2rem;
  color: #2962ff;
  white-space: nowrap;
}

.market-status {
  display: flex;
  align-items: center;
  gap: 15px;
  font-size: 0.9rem;
  background: rgba(255, 255, 255, 0.05);
  padding: 4px 12px;
  border-radius: 4px;
}

.market-status .symbol { font-weight: bold; color: #fff; }
.market-status .price { color: #fff; font-family: monospace; }
.market-status .change.plus { color: #26a69a; }
.market-status .change.minus { color: #ef5350; }
.market-status .amplitude { color: #8a8d97; font-size: 0.8rem; }

.controls {
  display: flex;
  align-items: center;
  gap: 10px;
  flex: 1;
}

.input-symbol {
  background: #2a2e39;
  border: 1px solid #363a45;
  color: white;
  padding: 4px 8px;
  border-radius: 4px;
  width: 100px;
}

.select-interval {
  background: #2a2e39;
  border: 1px solid #363a45;
  color: white;
  padding: 4px 8px;
  border-radius: 4px;
}

button {
  background: #2a2e39;
  border: 1px solid #363a45;
  color: #d1d4dc;
  padding: 6px;
  border-radius: 4px;
  cursor: pointer;
  display: flex;
  align-items: center;
  justify-content: center;
}

button:hover {
  background: #363a45;
}

button.active {
  background: #2962ff;
  color: white;
}

.divider {
  width: 1px;
  height: 20px;
  background: #363a45;
  margin: 0 5px;
}

.main-content {
  flex: 1;
  display: flex;
  overflow: hidden;
}

.chart-section {
  flex: 1;
  position: relative;
}

.side-panel {
  width: 350px;
  border-left: 1px solid #2a2e39;
  display: flex;
  flex-direction: column;
  background-color: #1e222d;
}

.account-info {
  font-weight: 500;
  color: #00c853;
  display: flex;
  flex-direction: column;
  align-items: flex-end;
  line-height: 1.2;
}

.total-pnl {
  font-size: 0.8rem;
}

.total-pnl.plus { color: #00c853; }
.total-pnl.minus { color: #ef5350; }
</style>
