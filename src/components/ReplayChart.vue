<script setup>
import { onMounted, onUnmounted, ref } from 'vue';
import { init, dispose, registerOverlay } from 'klinecharts';

const chartContainer = ref(null);
let chart = null;

// 注册矩形叠加层
registerOverlay({
  name: 'rect',
  totalStep: 3,
  createPointFigures: ({ coordinates }) => {
    if (coordinates.length === 2) {
      return {
        type: 'rect',
        attrs: {
          x: Math.min(coordinates[0].x, coordinates[1].x),
          y: Math.min(coordinates[0].y, coordinates[1].y),
          width: Math.abs(coordinates[0].x - coordinates[1].x),
          height: Math.abs(coordinates[0].y - coordinates[1].y)
        },
        styles: {
          style: 'stroke_fill',
          color: 'rgba(41, 98, 255, 0.1)',
          borderColor: '#2962ff',
          borderSize: 1
        }
      };
    }
    return [];
  }
});

const setStyles = () => {
  chart?.setStyles({
    grid: {
      show: false, // 移除网格线
    },
    crosshair: {
      show: true,
      horizontal: {
        show: true,
        line: {
          show: true,
          color: '#333',
          style: 'dashed',
          size: 1
        }
      },
      vertical: {
        show: true,
        line: {
          color: '#555',
          style: 'dashed'
        }
      }
    },
    candle: {
      bar: {
        upColor: '#089981',
        downColor: '#f23645',
        noChangeColor: '#888888'
      },
      priceMark: {
        last: {
          show: true,
          upColor: '#089981',
          downColor: '#f23645',
          noChangeColor: '#888888',
          line: { show: true, style: 'dashed' }
        }
      },
      tooltip: {
        showRule: 'always',
        text: {
          color: '#d1d4dc',
          size: 12
        },
        custom: (data) => {
          const { prev, current } = data;
          const change = prev ? ((current.close - prev.close) / prev.close * 100).toFixed(2) : '0.00';
          const amplitude = ((current.high - current.low) / current.low * 100).toFixed(2);
          const color = current.close >= current.open ? '#089981' : '#f23645';
          
          return [
            { title: '时间', value: new Date(current.timestamp).toLocaleString() },
            { title: '开盘', value: current.open.toFixed(2) },
            { title: '最高', value: current.high.toFixed(2) },
            { title: '最低', value: current.low.toFixed(2) },
            { title: '收盘', value: current.close.toFixed(2) },
            { title: '涨跌', value: `${change}%`, color },
            { title: '波幅', value: `${amplitude}%` }
          ];
        }
      }
    },
    indicator: {
      lastValueMark: { show: false },
      tooltip: { showRule: 'always' }
    }
  });
};

onMounted(() => {
  if (chartContainer.value) {
    chart = init(chartContainer.value);
    setStyles();
    // 添加成交量指标
    chart.createIndicator('VOL', false);
    
    // 监听加载更多历史数据
    chart.loadMore(timestamp => {
      emit('loadMore', timestamp);
    });
  }
});

const emit = defineEmits(['loadMore']);

onUnmounted(() => {
  if (chart) {
    dispose(chartContainer.value);
  }
});

const setNewData = (data) => {
  chart?.applyNewData(data, true);
};

const applyMoreData = (data) => {
  chart?.applyMoreData(data, true);
};

const updateData = (data) => {
  chart?.updateData(data);
};

const clear = () => {
  chart?.applyNewData([]);
};

const createOverlay = (name) => {
  if (chart) {
    console.log(`Creating overlay: ${name}`);
    chart.createOverlay(name);
  }
};

const removeOverlay = (id) => {
  chart?.removeOverlay(id);
};

// 暴露方法给父组件
defineExpose({
  setNewData,
  applyMoreData,
  updateData,
  clear,
  createOverlay,
  removeOverlay
});
</script>

<template>
  <div ref="chartContainer" class="chart-container"></div>
</template>

<style scoped>
.chart-container {
  width: 100%;
  height: 100%;
}
</style>
