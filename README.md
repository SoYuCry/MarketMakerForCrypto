# 做市交易程序

针对加密货币做市交易的通用架构。该程序提供自动化做市功能，通过维持买卖价差赚取利润。

Twitter：[YuCry|(❤,👾,💵)](https://x.com/0xYuCry)

## 功能特点

- **多交易所架构**：支援 Backpack、Aster 未来可扩展至其他交易所
- **自动化做市策略**：智能价差管理和订单调整
- **永续合约做市**：仓位风险管理与风险中性机制
- **智能重平衡系统**：自动维持资产配置比例
- **增强日志系统**：详细的市场状态和策略追踪
- **WebSocket 实时连接**：即时市场数据和订单更新
- **命令行界面**：灵活的参数配置和策略执行
- **交互式面板**：用户友好的操作介面

*2025.09.24 更新：新增多 Aster 交易所支持。*

*2025.09.22 更新：新增多交易所架构和仓位管理优化，增强日志系统提供更清晰的市场状态追踪。*

## 刷量原理
每隔 10 秒钟（可以设置），上下挂 2 单（可以设置），如果这 10 秒钟，上下一哆嗦，就完成了一次超短的低买高卖。中间的价差减手续费 = 利润。未成交就取消，重新挂。

理论上，价差 > 进出手续费，你可以赚钱。

如果 10 秒钟只完成了买或者卖，那么你这部分敞口会暴露在市场 10 秒钟。可能造成亏损。

如果想成交的更频繁，就中间价差缩小（spread 参数，比如 0.03%）。

如果行情一直单边运行，敞口超过 0.5 SOL（max_postion 参数，可设置），会触发市价减仓，把超过的部分减掉。使得整体敞口维持中性（target_position 目标持仓，也可以设置维持多头/空头）。
![alt text](4ab077855373759f9706cc8814e39462.png)

## (第 0 步：如果么有账号的话）

Aster 自动全返链接：[fc51fF](https://www.asterdex.com/en/referral/fc51fF)

Backpack 自动全返链接：[9gq7rl2r](https://backpack.exchange/join/9gq7rl2r)

## 第一步：下载 viscode 和 anoconda 并安装

Viscode：https://code.visualstudio.com/download

Anoconda：https://www.anaconda.com/download/success


## 第二步：下载代码并配置环境变量

1. 克隆或下载此代码库（输入到 viscode 的黑窗中）:

```bash
git clone https://github.com/SoYuCry/MarketMakerForCrypto.git
cd MarketMakerForCrypto
```

2. 安装依赖:

```bash
pip install -r requirements.txt
```

3. 设置环境变数:

复制 `.env.example` 为 `.env` 并添加:

```
# Backpack Exchange
BACKPACK_KEY=your_backpack_api_key
BACKPACK_SECRET=your_backpack_secret_key
BACKPACK_PROXY_WEBSOCKET=
BASE_URL=https://api.backpack.work

# Websea Exchange
WEBSEA_TOKEN=your_websea_token
WEBSEA_SECRET=your_websea_secret
WEBSEA_BASE_URL=https://coapi.websea.com

# Aster Exchange
ASTER_API_KEY=your_aster_api_key
ASTER_SECRET_KEY=your_aster_secret_key
```

## 第三步：运行脚本

#### BackPack 永续做市
```bash
python run.py --exchange backpack --market-type perp --symbol SOL_USDC_PERP --spread 0.02 \
  --quantity 0.1 --max-orders 2 --target-position 0 --max-position 0.5 \
  --position-threshold 0.4 --inventory-skew 0 --duration 999999999 --interval 10
```

#### Aster 永续做市
```bash
python run.py --exchange aster --market-type perp --symbol SOLUSDT --spread 0.02 \
  --quantity 0.1 --max-orders 2 --target-position 0 --max-position 0.5 \
  --position-threshold 0.4 --inventory-skew 0 --duration 999999999 --interval 10
```

## 第四步：查看网页前端是否运行正常。
👉 正常表现：频繁挂单与撤单，偶尔成交。

⚠️ **遇到任何问题！先问 AI；搞不定再末尾扫码进群丢截图！！**

### 命令行参数

#### 基本参数
- `--exchange`：交易所选择（目前支持 backpack, aster, websea）
- `--symbol`：交易对（仅永续，例如：SOL_USDC_PERP）
- `--spread`：价差百分比（如：0.05,表示 0.05%，实操：0.01-0.05，追求量：0.02）
- `--quantity`：单笔订单数量（可选）
- `--max-orders`：每侧最大挂单数（默认：3）
- `--duration`：运行时间（秒，默认：3600，永久填 999999999999）
- `--interval`：挂单更新间隔（秒，5-15秒为宜，会自动快进/滞后0-2秒防止女巫）
- `--market-type`：市场类型（`spot` 或 `perp`）
- `--target-position`：永续合约目标净仓位（通常为 0）
- `--max-position`：永续合约最大允许净仓
- `--position-threshold`：永续仓位调整触发值
- `--inventory-skew`：永续做市报价偏移系数（暂时为 0，功能等待更新）
  - 持有多单时：报价下移，吸引卖单成交
  - 持有空单时：报价上移，吸引买单成交
  - 目标：持续将净仓位推向 `0`，降低方向性风险



#### 仓位管理逻辑优化

| 当前仓位 | 目标仓位 | 阈值 | 最大仓位 | 执行动作 |
|---------|---------|------|---------|---------|
| 0.1 SOL | 0 SOL | 0.2 SOL | 0.4 SOL | 无操作（在目标范围内） |
| 0.25 SOL | 0 SOL | 0.2 SOL | 0.4 SOL | 减仓 0.05 SOL（只平掉超出阈值线的部分） |
| 0.5 SOL | 0 SOL | 0.2 SOL | 0.4 SOL | 风控平仓 0.1 SOL（降到最大仓位限制内） |

#### 增强日志输出范例

```
=== 市场状态 ===
盘口: Bid 239.379 | Ask 239.447 | 价差 0.068 (0.028%)
中间价: 239.408
持仓: 空头 6.000 SOL | 目标: 1.0 | 上限: 1.0

=== 价格计算 ===
原始挂单: 买 238.800 | 卖 239.996
偏移计算: 净持仓 -6.000 | 偏移系数 0.00 | 偏移量 0.0000
调整后挂单: 买 238.800 | 卖 239.996

=== 本次执行总结 ===
成交: 买入 0.200 SOL | 卖出 0.150 SOL
本次盈亏: 2.4500 USDT (手续费: 0.1200)
累计盈亏: 15.2300 USDT | 未实现: -1.8900 USDT
活跃订单: 买 238.800 | 卖 239.996 | 价差 1.196 (0.50%)
```

## 注意事项

### 一般注意事项
⚠️ 交易有风险，请谨慎使用

✅ 建议先用小资金测试策略效果

📊 定期检查日志与统计，评估策略表现

<img src="fbd88aef3c0f049e8d3b57238e7565eb.jpg" alt="说明文字" width="300">