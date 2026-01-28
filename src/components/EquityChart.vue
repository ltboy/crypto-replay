<script setup>
import { onMounted, onUnmounted, ref, watch } from 'vue';
import { init, dispose } from 'klinecharts';

const props = defineProps({
  data: Array
});

const chartContainer = ref(null);
let chart = null;

onMounted(() => {
  if (chartContainer.value) {
    chart = init(chartContainer.value);
    chart.setStyles({
      grid: { show: false },
      yAxis: { show: false }, // 移除纵坐标
      xAxis: {
        axisLine: { show: false },
        tickLine: { show: false },
        tickText: { show: false }
      },
      candle: {
        show: true,
        type: 'area',
        area: {
          lineColor: '#2962ff',
          fillColor: [{ offset: 0, color: 'rgba(41, 98, 255, 0.2)' }, { offset: 1, color: 'rgba(41, 98, 255, 0)' }]
        },
        tooltip: {
          showRule: 'follow_cross',
          showType: 'rect',
          text: {
            size: 12,
            color: '#2962ff'
          },
          custom: (data) => {
            return [
              { title: '余额', value: `$${data.current.close.toFixed(2)}` }
            ];
          }
        }
      },
      indicator: {
        lastValueMark: { show: false },
        tooltip: { showRule: 'none' }
      },
      crosshair: {
        show: true,
        horizontal: { show: false },
        vertical: { line: { color: '#555', style: 'dashed' } }
      }
    });
    updateChart();
  }
});

function updateChart() {
  if (!chart) return;
  const chartData = props.data.map(d => ({
    timestamp: d.time,
    open: d.balance,
    high: d.balance,
    low: d.balance,
    close: d.balance,
    volume: 0
  }));
  chart.applyNewData(chartData);
}

watch(() => props.data, () => {
  updateChart();
}, { deep: true });

onUnmounted(() => {
  if (chart) dispose(chartContainer.value);
});
</script>

<template>
  <div class="equity-container">
    <div class="title">收益曲线</div>
    <div ref="chartContainer" class="chart"></div>
  </div>
</template>

<style scoped>
.equity-container {
  height: 120px;
  border-top: 1px solid #2a2e39;
  padding: 0; /* 彻底移除内边距 */
  display: flex;
  flex-direction: column;
}
.title {
  font-size: 0.7rem;
  color: #8a8d97;
  margin-top: 5px;
  margin-bottom: 2px;
  padding-left: 10px;
}
.chart {
  flex: 1;
}
</style>
