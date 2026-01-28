# CryptoReplay Pro

一个基于 Vue 3 和 KLineCharts 开发的高性能加密货币行情复盘系统。旨在帮助交易者通过历史行情回放，磨炼交易直觉，验证交易策略。

![License](https://img.shields.io/badge/license-MIT-blue.svg)
![Vue](https://img.shields.io/badge/vue-3.5.x-brightgreen.svg)
![KLineCharts](https://img.shields.io/badge/klinecharts-9.4.0-orange.svg)

## ✨ 功能特性

- **🚀 极速复盘**: 支持从 2018 年至今的任意历史时刻随机开始复盘。
- **📈 深度集成 KLineCharts**:
    - 支持多周期切换（1m, 5m, 15m, 1h, 4h, 1d）。
    - 周期切换时时间/价格精准对齐，模拟真实市场收盘逻辑。
    - 无限历史行情加载：向左拖动图表自动拉取更早的数据。
- **🛠️ 专业交易工具**:
    - **画线工具**: 趋势线、水平射线（自动向右延伸）、矩形区域、斐波那契回撤。
    - **入场可视化**: 开仓后自动在图表绘制入场价格线。
    - **指标支持**: 内置成交量 (VOL) 指标，支持扩展更多技术指标。
- **💰 模拟交易系统**:
    - 支持多/空双向开仓。
    - 可自定义杠杆（最高 10x）与投入金额。
    - 实时计算未实现盈亏与 ROI。
- **📊 数据统计**:
    - 详细的交易历史记录表。
    - 实时生成的**收益曲线 (Equity Chart)**，直观展现资金变动。

## 🛠️ 技术栈

- **前端框架**: Vue 3 (Composition API)
- **图表库**: [KLineCharts v9.4.0](https://klinecharts.com/)
- **图标库**: Lucide Vue Next
- **构建工具**: Vite
- **数据源**: Binance Public API

## 📦 快速开始

### 环境准备

确保您的电脑已安装 [Node.js](https://nodejs.org/) (建议 v18+) 和 [Bun](https://bun.sh/) (可选，推荐)。

### 安装步骤

1. 克隆仓库:
   ```bash
   git clone https://github.com/your-username/crypto-replay-vue.git
   cd crypto-replay-vue
   ```

2. 安装依赖:
   ```bash
   bun install
   # 或者使用 npm
   npm install
   ```

3. 启动开发服务器:
   ```bash
   bun run dev
   # 或者使用 npm
   npm run dev
   ```

4. 访问地址:
   在浏览器打开 `http://localhost:5173/`。

## 📖 使用指南

1. **选择币种**: 在顶部输入框输入币安支持的交易对（如 `ETHUSDT`）。
2. **随机跳转**: 点击 🎲 按钮，系统将带你回到历史中一个随机的时刻。
3. **播放控制**: 使用 ⏯️ 按钮开始自动播放 K 线，或使用 ⏭️ 按钮手动逐根查看。
4. **进行交易**: 在右侧面板设置杠杆和金额，点击“开多”或“开空”入场。
5. **策略分析**: 使用顶部的画线工具标注支撑阻力位或趋势。

## 🤝 贡献

欢迎提交 Issue 或 Pull Request 来完善这个项目！

## 📄 开源协议

本项目采用 [MIT License](LICENSE) 协议。
