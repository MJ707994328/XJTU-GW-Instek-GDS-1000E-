# 西安交通大学示波器实验测评系统

一个面向教学场景的桌面应用，用于连接 GW Instek GDS-1000E 系列示波器，自动抓取实验过程画面，并结合实验目标、最终波形截图和实验用时生成智能评分结果。

项目包含两部分核心能力：

- 实验过程记录：自动识别示波器、实时预览当前屏幕、定时保存过程截图、生成实验记录目录。
- 智能评分：读取教师输入的实验目标，分析最终示波器截图，并按 `波形得分 70% + 时间得分 30%` 计算总分。

## 功能概览

- 自动检测 `/dev/ttyACM*` 和 `/dev/ttyUSB*` 下的 GW Instek GDS 示波器
- 通过 SCPI 命令抓取 GDS-1000E 屏幕图像
- 图形化实验界面，适合课堂或实验室现场使用
- 自动保存实验元数据、实验目标、最终截图和过程截图
- 支持 OpenAI 或 Moonshot/Kimi 兼容接口进行 AI 评分
- 明确区分波形扣分项与时间扣分项，输出结构化评分结果

## 项目结构

```text
.
├── teaching_eval_app.py      # 主界面程序
├── ai_scoring.py             # AI 评分逻辑
├── gds1000e.py               # GDS-1000E 串口通信与屏幕解码
├── test_screen.py            # 截图相关测试脚本
├── test_visa.py              # VISA 通信测试脚本
├── requirements.txt          # Python 依赖
├── run.sh                    # 一键启动脚本
├── assets/
│   └── xjtu_logo.png         # 界面资源
└── experiments/              # 运行后自动生成，默认不提交
```

## 运行环境

- Linux
- Python 3.10 及以上
- 可访问示波器的本机串口权限
- 适配 GW Instek GDS-1000E 系列示波器

说明：

- 当前设备发现逻辑依赖 `/dev/ttyACM*`、`/dev/ttyUSB*` 和 `termios`，因此默认面向 Linux 环境。
- 如果需要 AI 评分，还需要配置可用的模型 API Key。

## 安装依赖

### 方式一：使用启动脚本

项目自带 `run.sh`，会自动创建虚拟环境、安装依赖并启动主程序：

```bash
chmod +x run.sh
./run.sh
```

### 方式二：手动安装

```bash
python3 -m venv .venv
source .venv/bin/activate
pip install -r requirements.txt
python3 teaching_eval_app.py
```

当前依赖包括：

- `customtkinter`
- `Pillow`
- `pyvisa`
- `pyvisa-py`

## AI 评分配置

程序支持两类兼容接口：

- OpenAI
- Moonshot / Kimi

优先读取以下环境变量：

```bash
MOONSHOT_API_KEY
KIMI_API_KEY
OPENAI_API_KEY
```

也可以在项目根目录创建本地配置文件 `.local_secrets.json`。这个文件已被 `.gitignore` 忽略，不会被提交到 GitHub。

示例：

```json
{
  "provider": "openai",
  "openai_api_key": "YOUR_API_KEY",
  "openai_model": "gpt-5.4-mini"
}
```

常见可选配置项包括：

- `provider`
- `api_key`
- `openai_api_key`
- `moonshot_api_key`
- `kimi_api_key`
- `base_url`
- `openai_base_url`
- `moonshot_base_url`
- `openai_model`
- `moonshot_model`
- `kimi_model`

默认模型：

- OpenAI: `gpt-5.4-mini`
- Moonshot: `kimi-k2.5`

## 使用流程

1. 连接示波器并确保系统能识别对应串口设备。
2. 启动程序。
3. 点击“刷新设备”，确认已检测到示波器。
4. 输入教师给出的实验目标和预期时长。
5. 点击“开始实验”。
6. 程序将自动实时预览示波器画面，并定时保存过程截图。
7. 点击“结束实验”后，系统会保存最终截图并尝试自动生成 AI 评分。

如果当前没有配置 API Key，实验记录仍然会保存，但评分面板会提示需要先完成配置。

## 实验输出目录

每次实验结束后，程序会在 `experiments/` 下按时间创建一个独立目录，例如：

```text
experiments/20260415_143210/
├── meta.json
├── description.txt
├── final_screen.png
├── ai_score.json              # 若评分成功则生成
└── snapshots/
    ├── screen_001.png
    ├── screen_002.png
    └── ...
```

各文件含义如下：

- `meta.json`：实验元数据、设备信息、起止时间、实验时长等
- `description.txt`：教师输入的实验目标
- `final_screen.png`：实验结束时保存的最终示波器截图
- `ai_score.json`：智能评分结果，成功完成评分后生成
- `snapshots/`：实验过程中的定时截图

## 评分规则

总分公式：

```text
总分 = 波形得分 * 70% + 时间得分 * 30%
```

其中：

- 波形得分由 AI 基于实验目标和最终示波器截图生成
- 时间得分从 100 分开始，若实际用时超过预期用时，则按超时百分比扣分

例如：

- 预期 20 分钟，实际 24 分钟
- 超时 20%
- 时间部分扣 20 分

## 测试脚本

项目内包含一些辅助测试脚本：

```bash
python3 test_screen.py
python3 test_visa.py
```

它们适合在调试示波器连接、截图或 VISA 通信时单独使用。

## 常见问题

### 1. 检测不到示波器

- 确认示波器已经通过 USB 正常连接到电脑
- 确认系统中存在 `/dev/ttyACM*` 或 `/dev/ttyUSB*`
- 确认当前用户有访问串口设备的权限

### 2. 能记录实验，但没有自动评分

- 检查是否配置了 `MOONSHOT_API_KEY`、`KIMI_API_KEY` 或 `OPENAI_API_KEY`
- 检查 `.local_secrets.json` 是否填写正确
- 检查网络是否可访问对应模型接口

### 3. AI 返回 429 或连接被中断

项目已经对常见的 `429`、`500`、`502`、`503`、`504` 和连接中断做了重试处理，但如果上游服务繁忙，仍然可能需要稍后重新评分。

## Git 忽略说明

以下内容默认不会上传：

- `.local_secrets.json`
- `.venv/`
- `__pycache__/`
- `captures/`
- `experiments/`

## 说明

这个仓库目前更适合作为实验室内部工具或课程项目原型使用。

如果你准备继续完善，下一步比较值得补的是：

- 更完整的设备兼容性说明
- 截图和评分结果的回放功能
- 更详细的错误提示与日志记录
- 面向部署的安装说明
