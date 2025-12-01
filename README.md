# ESPHome Solar Power Sink - 光伏余电空调智能控制系统

[![ESPHome](https://img.shields.io/badge/ESPHome-2025.9.1-blue)](https://esphome.io/)
[![Hardware](https://img.shields.io/badge/Hardware-M5Stack%20ATOM%20S3-orange)](https://shop.m5stack.com/products/atoms3-dev-kit-w-0-85-inch-screen)
[![License](https://img.shields.io/badge/License-MIT-green)](LICENSE)

基于 M5Stack ATOM S3 (ESP32-S3FN8) 的智能光伏余电消纳系统，通过红外遥控自动调节空调功率，最大化利用光伏发电，减少电网馈电。

## 🎯 项目目标

在家庭光伏发电系统中，当发电量超过家庭用电负载时，多余电力会馈入电网（通常收益较低）。本项目通过实时监测电表数据，智能控制空调运行状态，将光伏余电转化为空调制冷/制热，实现：

- ✅ **最大化光伏自用率** - 避免余电低价馈网
- ✅ **零成本舒适调温** - 利用免费光伏电力
- ✅ **电网功率平衡** - 减少对电网的波动影响
- ✅ **自动化控制** - 无需人工干预，智能决策

## 📐 系统架构

```
┌─────────────────┐      ┌──────────────────┐      ┌─────────────────┐
│  光伏逆变器      │─────▶│   智能电表        │─────▶│  ESP32-S3 (本项目) │
│  Solar Inverter │      │  Smart Meter     │      │  + IR Transmitter │
└─────────────────┘      └──────────────────┘      └─────────────────┘
                               │                            │
                               │ DL/T 645-2007             │ 红外38kHz
                               │ 电力协议                   │ IR Protocol
                               │                            ▼
                         测量实时功率               ┌─────────────────┐
                         (馈网/用电)                 │   空调 Air Conditioner  │
                                                    │   - 开/关机      │
                                                    │   - 温度调节     │
                                                    │   - 模式切换     │
                                                    └─────────────────┘
```

### 工作流程

1. **电表数据采集** - 通过 RS485/红外读取智能电表的实时功率数据（DL/T 645-2007 协议）
2. **光伏余电计算** - 判断当前是馈网（正值）还是用电（负值）
3. **空调控制策略**：
   - 馈网功率 > 500W → 开启空调制冷/制热（根据季节）
   - 馈网功率 > 1000W → 增加空调功率（降低温度/提高风速）
   - 馈网功率 < 300W → 降低空调功率或关机
4. **红外指令发送** - ESP32-S3 通过 GPIO2 红外发射器控制空调

## 🛠️ 硬件清单

| 组件 | 型号/规格 | 数量 | 说明 |
|------|----------|------|------|
| 主控板 | M5Stack ATOM S3 | 1 | ESP32-S3FN8, 8MB Flash, USB-C |
| 红外发射器 | 38kHz IR LED | 1 | 连接 GPIO2, 需要 NPN 三极管驱动 |
| 红外接收器 | VS1838B / TSOP4838 | 1 | 连接 GPIO1（用于学习遥控器） |
| 电表通信模块 | RS485/红外适配器 | 1 | 读取 DL/T 645-2007 协议数据 |
| 可选：OLED | 0.85" TFT | 1 | ATOM S3 自带屏幕，可显示状态 |

### 硬件连接

```
M5Stack ATOM S3
┌─────────────────┐
│   GPIO1 ────────┼──▶ IR Receiver (VS1838B)  ──▶ 学习空调遥控器信号
│   GPIO2 ────────┼──▶ IR LED (38kHz Emitter) ──▶ 控制空调
│   GPIO5/6 ──────┼──▶ UART/RS485 ──────────────▶ 连接智能电表
│   USB-C ────────┼──▶ 供电 + 调试日志
└─────────────────┘
```

## 📦 软件依赖

- **ESPHome** 2025.9.1 或更高版本
- **Python** 3.8+ (用于 ESPHome 环境)
- **IR 协议库** - ESPHome 内置红外协议支持
- **DL/T 645-2007** - 电力行业电表通信协议（需自定义组件）

### 安装 ESPHome

```bash
# 创建 Python 虚拟环境
python3 -m venv ~/venv/esphome

# 激活虚拟环境
source ~/venv/esphome/bin/activate

# 安装 ESPHome
pip install esphome

# 验证安装
esphome version
```

## 🚀 快速开始

### 1. 克隆项目

```bash
git clone git@github.com:chinawrj/esphome-soloar-power-sink.git
cd esphome-soloar-power-sink
```

### 2. 配置设备

创建 `secrets.yaml` 文件（已在 `.gitignore` 中排除）：

```yaml
# WiFi 配置
wifi_ssid: "YourWiFiSSID"
wifi_password: "YourWiFiPassword"

# Home Assistant API（可选）
api_encryption_key: "your-32-byte-base64-key"

# OTA 更新密码（可选）
ota_password: "your_ota_password"
```

### 3. 学习空调遥控器信号

使用 `ir_analyzer_esp32s3fn8.yaml` 学习你的空调遥控器信号：

```bash
# 激活 ESPHome 环境
source ~/venv/esphome/bin/activate

# 编译并上传 IR 分析器固件
esphome run ir_analyzer_esp32s3fn8.yaml

# 监控串口日志
esphome logs ir_analyzer_esp32s3fn8.yaml
```

**操作步骤**：
1. 将 IR 接收器（GPIO1）对准空调遥控器
2. 按下遥控器按键（如：开机、温度+、温度-、模式切换）
3. 记录日志中的原始红外时序数据
4. 将数据配置到控制程序中

### 4. 部署控制系统

```bash
# 编译主控制程序
esphome compile ac_protocol_analyzer.yaml

# 上传到设备
esphome upload ac_protocol_analyzer.yaml

# 实时监控运行状态
esphome logs ac_protocol_analyzer.yaml
```

## 📁 项目文件说明

```
esphome-soloar-power-sink/
├── README.md                           # 本文档
├── .gitignore                          # Git 忽略配置（包含 secrets.yaml）
├── secrets.yaml                        # 敏感信息配置（需手动创建）
├── ir_analyzer_esp32s3fn8.yaml        # 红外信号分析器（学习遥控器）
├── ac_protocol_analyzer.yaml          # 空调协议分析器（实验性）
├── ac_ir_analyzer_irremote.yaml       # IRremoteESP8266 集成（实验性）
└── .github/
    └── copilot-instructions.md         # GitHub Copilot 开发指南
```

### 配置文件详解

#### `ir_analyzer_esp32s3fn8.yaml` - 红外信号学习工具

**功能**：
- 接收并记录空调遥控器的红外信号
- 支持 NEC, Samsung, LG, Sony, RC5 等常见协议
- 自动分析未知协议的原始时序数据
- 防止波形退化的自发送信号过滤
- 5秒自动重发学习到的信号（用于测试）

**GPIO 配置**：
- GPIO1: 红外接收器（VS1838B）
- GPIO2: 红外发射器（38kHz IR LED）
- GPIO41: 用户按钮（手动触发发送）

**使用场景**：
1. 首次部署时学习空调遥控器信号
2. 调试红外硬件连接
3. 验证红外发射器工作正常

#### `ac_protocol_analyzer.yaml` - 空调协议分析器（实验性）

**目标**：尝试集成 IRremoteESP8266 库以支持更多空调品牌（50+ 协议）

**状态**：⚠️ **实验性** - 存在 Arduino ESP32 v3.x 兼容性问题

**支持的空调协议**（理论）：
- Gree, Midea, Haier, Daikin, Fujitsu, Hitachi
- LG, Samsung, Panasonic, Mitsubishi, Toshiba
- TCL, Coolix, Electra, Kelon, 等 50+ 品牌

## 🔧 高级配置

### 电表数据读取配置

本项目计划支持 **DL/T 645-2007** 中国电力行业标准协议，通过 RS485 或红外适配器读取智能电表数据。

**关键数据项**：
- **总有功功率** (Data ID: `0x02030000`) - 判断馈网/用电状态
  - 正值：馈网（光伏发电多于用电）
  - 负值：用电（光伏发电不足）
- **电网电压** (Data ID: `0x02010100`) - 监控电网状态
- **电网频率** (Data ID: `0x02800002`) - 电网质量指标

**配置示例**（待实现）：

```yaml
# UART 配置用于 DL/T 645-2007 通信
uart:
  - id: meter_uart
    tx_pin: GPIO5
    rx_pin: GPIO6
    baud_rate: 2400
    data_bits: 8
    parity: EVEN
    stop_bits: 1

# 自定义组件读取电表数据
sensor:
  - platform: custom
    lambda: |-
      // DL/T 645-2007 协议实现
      auto meter = new DLT645Meter(id(meter_uart));
      App.register_component(meter);
      return {meter->power_sensor, meter->voltage_sensor};
    sensors:
      - name: "Grid Power"
        unit_of_measurement: "W"
        accuracy_decimals: 2
      - name: "Grid Voltage"
        unit_of_measurement: "V"
        accuracy_decimals: 1
```

### 空调控制策略

**分级功率控制**：

```yaml
# 根据馈网功率自动调整空调状态
script:
  - id: adjust_ac_power
    then:
      - lambda: |-
          float grid_power = id(grid_power_sensor).state;
          
          if (grid_power > 1500) {
            // 馈网 > 1.5kW - 强力制冷
            send_ac_command(POWER_ON, COOL_MODE, TEMP_18C, FAN_HIGH);
          } else if (grid_power > 800) {
            // 馈网 > 800W - 标准制冷
            send_ac_command(POWER_ON, COOL_MODE, TEMP_22C, FAN_MED);
          } else if (grid_power > 300) {
            // 馈网 > 300W - 轻度制冷
            send_ac_command(POWER_ON, COOL_MODE, TEMP_26C, FAN_LOW);
          } else if (grid_power < 100) {
            // 馈网不足 - 关机
            send_ac_command(POWER_OFF);
          }
```

## 📊 监控与调试

### ESPHome 日志查看

```bash
# 实时查看设备日志
esphome logs ir_analyzer_esp32s3fn8.yaml

# 通过 WiFi OTA 查看日志（无需 USB 连接）
esphome logs ir_analyzer_esp32s3fn8.yaml --device 192.168.1.99
```

### Home Assistant 集成（可选）

本项目支持 Home Assistant 集成，可在 HA 中查看实时数据和控制空调：

```yaml
# 在 ESPHome 配置中启用 API
api:
  encryption:
    key: !secret api_encryption_key

# Home Assistant 会自动发现设备
# 实体示例：
# - sensor.grid_power (电网功率)
# - sensor.ac_temperature (空调目标温度)
# - switch.ac_power (空调开关)
# - climate.living_room_ac (空调控制实体)
```

## 🐛 故障排除

### 问题 1: 红外发射器无反应

**原因**：IR LED 需要 NPN 三极管驱动，GPIO2 直连电流不足

**解决方案**：
```
GPIO2 ──[1kΩ]──▶ NPN 基极 (如 2N2222)
                 ├─ 发射极 ──▶ GND
                 └─ 集电极 ──▶ IR LED 正极 ──[100Ω]──▶ 3.3V
```

### 问题 2: 红外信号学习失败

**检查清单**：
- ✅ VS1838B 供电 3.3V 正确
- ✅ 信号引脚连接 GPIO1
- ✅ 遥控器电池电量充足
- ✅ 遥控器对准接收器（距离 5-30cm）
- ✅ 日志级别设置为 DEBUG

### 问题 3: 编译失败

**IRremoteESP8266 兼容性问题**：

当前 IRremoteESP8266 v2.8.6 与 Arduino ESP32 v3.x 存在兼容性问题（timer API 变更）。

**临时解决方案**：
1. 使用 ESPHome 原生 IR 协议（已支持 15-20 种空调协议）
2. 等待 IRremoteESP8266 更新支持 Arduino v3.x
3. 考虑使用独立 Arduino 项目 + ESPHome UART 通信架构

## 🔮 未来计划

- [ ] 完整实现 DL/T 645-2007 电表通信协议
- [ ] 支持更多空调品牌和型号
- [ ] 增加天气预报集成（预测未来光伏发电量）
- [ ] 开发 Web 控制面板（实时监控和手动控制）
- [ ] 支持多台空调联动控制
- [ ] 机器学习优化控制策略
- [ ] 添加电费计算和节能报告

## 📄 许可证

本项目采用 MIT 许可证 - 详见 [LICENSE](LICENSE) 文件

## 🤝 贡献

欢迎提交 Issue 和 Pull Request！

**贡献指南**：
1. Fork 本项目
2. 创建特性分支 (`git checkout -b feature/AmazingFeature`)
3. 提交更改 (`git commit -m 'feat: add some amazing feature'`)
4. 推送到分支 (`git push origin feature/AmazingFeature`)
5. 开启 Pull Request

**提交规范**：遵循 [Conventional Commits](https://www.conventionalcommits.org/)

```
feat: 新功能
fix: 修复 Bug
docs: 文档更新
refactor: 代码重构
test: 测试相关
chore: 构建/工具链更新
```

## 📞 联系方式

- **GitHub Issues**: [提交问题](https://github.com/chinawrj/esphome-soloar-power-sink/issues)
- **项目地址**: https://github.com/chinawrj/esphome-soloar-power-sink

## 🙏 致谢

- [ESPHome](https://esphome.io/) - 优秀的 ESP32 开发框架
- [IRremoteESP8266](https://github.com/crankyoldgit/IRremoteESP8266) - 强大的红外协议库
- [M5Stack](https://m5stack.com/) - 出色的硬件平台
- 所有开源贡献者

---

⚡ **让光伏余电驱动空调，享受零成本的舒适生活！**
