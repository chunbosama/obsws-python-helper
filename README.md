# obsws-python-helper

对 [obsws-python](https://github.com/aatikturk/obsws-python) 库的完整封装 SDK，基于 OBS Studio WebSocket v5.0 协议，提供线程安全的 OBS 控制器封装类。

[![License: MIT](https://img.shields.io/badge/License-MIT-blue.svg)](LICENSE)
[![Python](https://img.shields.io/badge/Python-3.8+-green.svg)](https://www.python.org/)

---

## 功能特性

- **场景管理**：获取列表、切换、创建、删除场景
- **输入源管理**：获取列表、音量控制、静音控制、音频同步偏移
- **场景项管理**：启用/禁用、位置/缩放/旋转/裁剪、混合模式、层级调整
- **录制控制**：开始、停止、暂停、恢复
- **推流控制**：开始、停止、切换
- **虚拟摄像头**：启动、停止、切换
- **转场控制**：获取列表、切换转场、设置时长
- **淡出到黑场**：`fade_to_black()` / `fade_from_black()`，基于 Studio Mode 实现平滑黑场过渡
- **媒体源控制**：播放、暂停、停止、重启
- **滤镜管理**：列表、启用/禁用、参数设置、添加/删除，内置美颜/灰度/锐化等预设
- **截图**：获取指定源的 base64 编码截图
- **热键控制**：通过名称或按键序列触发热键
- **事件监听**：场景切换、录制状态、推流状态等事件回调
- **线程安全**：`_ThreadSafeReq` 代理确保多线程环境下请求/响应不乱序
- **自动重连**：连接断开后自动重连（可选）

---

## 安装步骤

### 前置要求

- Python 3.8 及以上版本
- OBS Studio 28.0 及以上（需启用 WebSocket 服务器）

### 启用 OBS WebSocket 服务器

1. 打开 OBS Studio
2. 进入 **工具 → WebSocket 服务器设置**
3. 勾选 **启用 WebSocket 服务器**
4. 记下端口号（默认 `4455`）
5. 如需密码认证，设置 **服务器密码**

### 安装依赖

```bash
pip install obsws-python
```

> `obsws-python-helper.py` 是单文件 SDK，无需额外安装本仓库，直接复制到项目中即可使用。

### 克隆仓库（可选）

```bash
git clone https://github.com/chunbosama/obsws-python-helper.git
cd obsws-python-helper
```

---

## 快速开始

```python
from obsws_python_helper import OBSController

# 创建控制器，连接到本地 OBS
obs = OBSController(
    host="localhost",
    port=4455,
    password="your_password",  # 无密码留空字符串
)

# 获取当前场景
current = obs.get_current_scene()
print(f"当前场景: {current}")

# 切换场景
obs.set_current_scene("场景2")

# 获取输入源列表
inputs = obs.get_input_list()
for inp in inputs:
    print(f"输入源: {inp['name']} ({inp['kind']})")

# 获取录制状态
status = obs.get_record_status()
print(f"录制中: {status['output_active']}")

obs.close()
```

---

## 使用示例

### 场景管理

```python
from obsws_python_helper import OBSController

obs = OBSController(password="your_password")

# 获取所有场景
scene_list = obs.get_scene_list()
print(f"当前场景: {scene_list['current_program_scene']}")
print(f"所有场景: {[s['name'] for s in scene_list['scenes']]}")

# 切换场景
obs.set_current_scene("摄像机")

# 创建新场景
obs.create_scene("新场景")

# 删除场景
obs.remove_scene("新场景")
```

### 音频控制

```python
# 获取输入源音量
vol = obs.get_input_volume("麦克风")
print(f"音量: {vol['volume_db']} dB")

# 设置音量（分贝）
obs.set_input_volume("麦克风", volume_db=-6.0)

# 设置音量（倍数）
obs.set_input_volume("麦克风", volume_mul=0.5)

# 切换静音
obs.toggle_input_mute("麦克风")

# 获取所有音频输入源
audio_inputs = obs.get_audio_inputs()
for inp in audio_inputs:
    print(f"音频源: {inp['name']}")
```

### 场景项变换控制

```python
# 获取场景中的所有场景项
items = obs.get_scene_items("主场景")
for item in items:
    print(f"场景项: {item['source_name']} (ID: {item['scene_item_id']})")

# 设置位置
obs.set_scene_item_position("主场景", item_id, x=960, y=540)

# 设置缩放
obs.set_scene_item_scale("主场景", item_id, scale_x=1.5, scale_y=1.5)

# 设置旋转
obs.set_scene_item_rotation("主场景", item_id, rotation=45.0)

# 设置裁剪
obs.set_scene_item_crop("主场景", item_id, left=100, right=100, top=50, bottom=50)

# 启用/禁用场景项
obs.set_scene_item_enabled("主场景", item_id, enabled=False)
```

### 录制与推流

```python
# 开始录制
obs.start_record()

# 暂停/恢复录制
obs.pause_record()
obs.resume_record()

# 停止录制（返回录制文件路径）
path = obs.stop_record()
print(f"录制文件保存在: {path}")

# 开始推流
obs.start_stream()

# 停止推流
obs.stop_stream()
```

### 淡出到黑场

```python
# 淡出到黑场（800ms 过渡）
obs.fade_to_black(duration_ms=800)

# 等待 3 秒后恢复
import time
time.sleep(3)

# 从黑场恢复
obs.fade_from_black(duration_ms=800)
```

### 滤镜管理

```python
# 获取源上的滤镜列表
filters = obs.get_filter_list("摄像机")
for f in filters:
    print(f"滤镜: {f['name']} ({f['kind']})")

# 添加美颜滤镜（使用内置预设）
from obsws_python_helper import FILTER_PRESETS

preset = FILTER_PRESETS["美颜-轻度"]
obs.add_filter(
    source_name="摄像机",
    filter_name=preset.name,
    filter_kind=preset.filter_kind,
    filter_settings=preset.settings
)

# 启用/禁用滤镜
obs.set_filter_enabled("摄像机", "美颜-轻度", enabled=True)

# 更新滤镜参数
obs.set_filter_settings("摄像机", "美颜-轻度", {"brightness": 0.1, "contrast": 0.1})
```

### 截图

```python
# 获取当前场景的截图（base64 编码）
image_data = obs.get_source_screenshot(
    source_name="主场景",
    img_format="jpg",
    width=1920,
    height=1080,
    quality=85
)

# 保存为文件
import base64
with open("screenshot.jpg", "wb") as f:
    f.write(base64.b64decode(image_data))
```

### 事件监听

```python
def on_scene_switch(data):
    print(f"场景切换到: {data.scene_name}")

def on_record_state(data):
    print(f"录制状态变化: {data.output_state}")

# 注册回调
obs.on_scene_changed(on_scene_switch)
obs.on_record_state_changed(on_record_state)

# 保持程序运行以接收事件
import time
while True:
    time.sleep(1)
```

### 上下文管理器

```python
with OBSController(password="your_password") as obs:
    obs.set_current_scene("直播间")
    obs.start_record()
    # 退出 with 块时自动调用 close()
```

---

## API 参考

### 初始化参数

| 参数 | 类型 | 默认值 | 说明 |
|------|------|--------|------|
| `host` | `str` | `"localhost"` | OBS WebSocket 服务器地址 |
| `port` | `int` | `4455` | OBS WebSocket 端口 |
| `password` | `str` | `""` | 认证密码 |
| `timeout` | `float \| None` | `None` | 请求超时（秒），None 表示无超时 |
| `auto_reconnect` | `bool` | `True` | 是否自动重连 |
| `log_level` | `int` | `logging.WARNING` | 日志级别 |

### 主要方法分类

| 分类 | 方法 |
|------|------|
| 通用 | `get_version()`, `get_stats()`, `get_summary()`, `broadcast_event()` |
| 场景 | `get_scene_list()`, `get_current_scene()`, `set_current_scene()`, `create_scene()`, `remove_scene()` |
| 输入源 | `get_input_list()`, `get_audio_inputs()`, `get_input_volume()`, `set_input_volume()`, `get_input_mute()`, `set_input_mute()`, `toggle_input_mute()` |
| 场景项 | `get_scene_items()`, `set_scene_item_enabled()`, `set_scene_item_transform()`, `set_scene_item_position()`, `set_scene_item_scale()`, `set_scene_item_rotation()`, `set_scene_item_crop()`, `set_scene_item_blend_mode()` |
| 录制 | `get_record_status()`, `start_record()`, `stop_record()`, `pause_record()`, `resume_record()` |
| 推流 | `get_stream_status()`, `start_stream()`, `stop_stream()`, `toggle_stream()` |
| 虚拟摄像头 | `get_virtualcam_status()`, `start_virtualcam()`, `stop_virtualcam()` |
| 转场 | `get_transition_list()`, `set_current_transition()`, `set_transition_duration()` |
| 黑场 | `fade_to_black()`, `fade_from_black()` |
| 媒体 | `get_media_input_status()`, `play_media()`, `pause_media()`, `stop_media()`, `restart_media()` |
| 滤镜 | `get_filter_list()`, `set_filter_enabled()`, `set_filter_settings()`, `add_filter()`, `remove_filter()` |
| 截图 | `get_source_screenshot()` |
| 热键 | `get_hotkey_list()`, `trigger_hotkey_by_name()`, `trigger_hotkey_by_sequence()` |
| Studio Mode | `get_studio_mode_enabled()`, `set_studio_mode_enabled()`, `trigger_studio_mode_transition()`, `set_current_preview_scene()` |

---

## 内置滤镜预设

`FILTER_PRESETS` 字典提供了常用滤镜预设，可直接使用：

| 预设名称 | 说明 |
|----------|------|
| `美颜-轻度` | 轻度美颜（亮度+0.05, 对比度+0.05, 饱和度+0.05）|
| `美颜-中度` | 中度美颜（亮度+0.1, 对比度+0.1, 饱和度+0.1）|
| `灰度` | 灰度效果（饱和度=0）|
| `反色` | 颜色反转 |
| `怀旧` | 复古暖色调 |
| `冷色调` | 冷色滤镜 |
| `暖色调` | 暖色滤镜 |
| `锐化` | 锐度+0.5 |
| `增益` | 亮度+0.2, 对比度+0.1 |
| `限幅` | 亮度-0.05, 对比度+0.2 |

---

## 贡献指南

贡献让这个项目变得更好！欢迎提交 Issue 和 Pull Request。

### 开发环境搭建

```bash
git clone https://github.com/chunbosama/obsws-python-helper.git
cd obsws-python-helper
pip install obsws-python
```



### 许可证

本项目采用 [MIT 许可证](LICENSE)，详情请参阅 LICENSE 文件。

---

### 相关链接

- [OBS Studio 官网](https://obsproject.com/)
- [OBS WebSocket 协议文档](https://github.com/obsproject/obs-websocket/blob/master/docs/generated/protocol.md)
- [obsws-python 库](https://github.com/aatikturk/obsws-python)
