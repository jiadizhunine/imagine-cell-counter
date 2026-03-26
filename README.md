<p align="center">
  <img src="logo.png" width="128" alt="Imagine Logo">
</p>

<h1 align="center">Imagine</h1>

<p align="center">
  <b>智能荧光显微镜细胞计数工具</b><br>
  <i>Intelligent Fluorescence Microscopy Cell Counter</i>
</p>

<p align="center">
  <img src="https://img.shields.io/badge/Platform-Windows%2011-blue?logo=windows" alt="Platform">
  <img src="https://img.shields.io/badge/Python-3.12-green?logo=python" alt="Python">
  <img src="https://img.shields.io/badge/License-MIT-orange" alt="License">
  <img src="https://img.shields.io/badge/GUI-PyQt5-purple" alt="GUI">
  <img src="https://img.shields.io/badge/语言-中文界面-red" alt="中文">
</p>

---

## 简介

**Imagine** 是一款专为生物学研究者设计的桌面应用程序，用于自动化分析荧光显微镜图像中的细胞计数与共定位分析。支持 Zeiss `.czi` 和 `.tif/.tiff` 格式。

**全中文界面、零代码操作、开箱即用** — 下载解压即可运行，无需安装 Python 或任何编程环境。

### 为什么选择 Imagine？

传统工具（如 Fiji/ImageJ）功能强大但上手门槛高，批量处理依赖宏脚本编写，对不熟悉编程的研究者并不友好。Imagine 通过以下设计理念解决这些痛点：

| 传统工具 (Fiji/ImageJ) | Imagine |
|:---|:---|
| 英文界面，需要查阅文档 | **全中文界面**，所有菜单、按钮、提示均为中文 |
| 批量处理需编写宏脚本 | **一键批量处理**，选择文件夹即可 |
| 参数调节需要命令行或对话框 | **滑块式参数调节**，拖动即可实时预览 |
| 多策略分析需多次手动操作 | **自动策略识别**，根据文件名智能选择分析方案 |
| 结果需手动记录到 Excel | **一键导出 CSV**，自动汇总所有结果和参数 |
| 安装配置复杂（Java 环境等） | **解压即用**，单文件夹部署，无任何依赖 |

---

## 界面设计

Imagine 采用**深色主题 + 三栏布局**设计，贴合研究者的日常使用习惯：

```
┌─────────────────────────────────────────────────────────────────┐
│  🖼️ Imagine                    ◀ 上一张  3/10  下一张 ▶  文件名 │
├──────────┬──────────────────────────────────┬───────────────────┤
│  左侧栏   │        中央图像查看区              │    右侧栏         │
│          │                                  │                   │
│ 检测到策略 │   ┌───────────────────────┐      │  分析结果          │
│ 通道分配   │   │                       │      │  ├ 细胞核总数       │
│ DAPI 阈值  │   │    显微镜图像           │      │  ├ AT2 数量        │
│ 红色阈值   │   │    滚轮缩放 / 拖拽平移   │      │  └ AT2 增殖率      │
│ 绿色阈值   │   │                       │      │                   │
│ 白色阈值   │   └───────────────────────┘      │  批量结果表格       │
│ 环宽度    │  通道: Blue Red Green White       │  (双击跳转/右键删除) │
│ 最小核面积 │  叠加层: 核轮廓 AT2数量 AT2增殖   │                   │
│          │  标注: 手动标注 清除 导出TIF        │  [导出 CSV]        │
│ [开始分析] │                                  │                   │
└──────────┴──────────────────────────────────┴───────────────────┘
```

### 交互设计亮点

- **所见即所得**：切换通道、调整叠加层时，图像实时更新且保持当前缩放位置不变
- **符合直觉的操作**：滚轮缩放、左键拖拽平移、右键标注 — 与常见看图软件一致
- **动态自适应**：界面控件根据图片内容自动显示/隐藏，无关参数不会干扰视线
- **Ctrl 多选**：打开文件时支持 Ctrl/Shift 多选，一次导入多张图片
- **键盘导航**：`←` `→` 快速切换上/下一张，无需频繁使用鼠标

---

## 功能特性

### 智能通道识别

从文件名自动解析抗体-荧光对应关系，无需手动配置：

| 文件名关键词 | 识别为 | 通道颜色 |
|:---|:---|:---|
| `Ki67-568` | Ki67 | 红色 (568nm) |
| `proSpc-647` / `proSPC-647` | ProSPC | 白色 (647nm) |
| `Hopx-488` / `HOPX-488` | HOPX | 绿色 (488nm) |
| `aSMA-568` | α-SMA | 红色 (568nm) |
| `GFP` | GFP | 绿色 (488nm) |
| DAPI | DAPI | 蓝色 (始终存在) |

### 自动分析策略

根据检测到的标记物组合，自动选择适用的分析方案。**如果文件无匹配策略，则不显示多余的策略界面，保持简洁。**

| 分析类型 | 所需标记 | 计算内容 |
|:---|:---|:---|
| **AT2 数量** | ProSPC + DAPI | ProSPC⁺ 细胞计数 |
| **AT2 增殖** | ProSPC + Ki67 + DAPI | Ki67⁺ProSPC⁺ / ProSPC⁺ |
| **AT1 数量** | HOPX + DAPI | HOPX⁺ 细胞计数 |
| **AT1 增殖** | HOPX + Ki67 + DAPI | Ki67⁺HOPX⁺ / HOPX⁺ |

### 核心算法

- **DAPI 核分割**：高斯模糊 → Triangle/Otsu 双算法自动阈值 → 形态学开运算去噪 → 填洞 → 分水岭分离粘连核
- **核定位标记检测**（如 Ki67）：计算核区域内信号平均强度
- **胞质标记检测**（如 HOPX、ProSPC）：膨胀核区域生成环形 ROI，计算环内信号强度
- **向量化优化**：使用 `np.bincount` 和 bbox 局部处理，单张图像分析 < 1 秒

### 多色标注系统

支持 **6 种颜色标签**，每种颜色独立编号（1, 2, 3...），适合同时标记不同类型的细胞：

| 标签 | 颜色 | 用途示例 |
|:---|:---|:---|
| Tag A | 金色 | 主要目标细胞 |
| Tag B | 红色 | 阳性对照 |
| Tag C | 青色 | 可疑区域 |
| Tag D | 蓝色 | 参考标记 |
| Tag E | 绿色 | 补充标记 |
| Tag F | 紫色 | 其他 |

- **导入 Fiji 标注**：自动读取 ImageJ ROI 格式的点标注
- **手动标注**：右键点击添加/删除标注点（十字标记 + 序号）
- **撤回**：`Ctrl+Z` 撤回上一步标注
- **导出**：`Ctrl+S` 导出为 Fiji 兼容的 TIF 文件（含嵌入 ROI 数据）

### 批量处理与结果管理

- 选择文件夹一键处理所有 `.czi` / `.tif` 文件
- 结果表格支持**双击跳转**查看对应图像
- **右键菜单**：删除单条结果或清空全部，无需重启程序
- CSV 导出自动适配：仅导出有意义的列，并附带完整的分析参数

---

## 安装与使用

### 方法一：直接下载（推荐）

1. 从 [Releases](../../releases) 页面下载最新版本的压缩包
2. 解压到任意目录
3. 双击 `Imagine.exe` 运行

> **无需安装 Python 或任何依赖，解压即用。**

### 方法二：从源码运行

```bash
git clone https://github.com/jiadizhunine/imagine-cell-counter.git
cd imagine-cell-counter
pip install -r requirements.txt
python cell_counter.py
```

### 方法三：从源码打包

```bash
pip install pyinstaller
pyinstaller --onedir --windowed --name Imagine --icon=imagine.ico --add-data logo.png;. cell_counter.py
```

---

## 快捷键

| 快捷键 | 功能 |
|:---|:---|
| `Ctrl+O` | 打开文件（支持 Ctrl 多选） |
| `Ctrl+B` | 批量处理文件夹 |
| `Ctrl+R` | 开始分析 |
| `Ctrl+E` | 导出 CSV |
| `Ctrl+S` | 导出为 TIF（含标注） |
| `Ctrl+Z` | 撤回标注 |
| `←` `→` | 上/下一张图片 |
| 右键单击 | 添加/删除标注点 |
| 滚轮 | 缩放图像 |
| 左键拖拽 | 平移图像 |

## 参数说明

| 参数 | 默认值 | 说明 |
|:---|:---|:---|
| DAPI 阈值 | 自动 (Triangle/Otsu) | 核分割阈值，自动模式取两种算法的较优值 |
| 红色/绿色/白色阈值 | 15 / 10 / 15 | 对应通道的信号强度判定阈值（0-255） |
| 环宽度 | 15 px | 胞质信号检测的环形区域宽度 |
| 最小核面积 | 50 px | 过滤过小的噪声区域 |

## 支持的文件格式

| 格式 | 读取 | 写入 | 通道自动识别 | Fiji 标注 |
|:---|:---:|:---:|:---:|:---:|
| Zeiss CZI | ✅ | — | ✅ (元数据 + 文件名) | — |
| TIFF / TIF | ✅ | ✅ | ✅ (文件名) | ✅ (ImageJ ROI) |

## 技术栈

- **GUI**: PyQt5 (Fusion 风格 + 自定义深色主题)
- **图像处理**: scikit-image (分水岭分割, 形态学), scipy (距离变换, 标记)
- **数值计算**: NumPy (向量化运算)
- **文件格式**: czifile (Zeiss CZI), tifffile (TIFF/ImageJ)
- **打包**: PyInstaller (单文件夹部署)

## 系统要求

- **操作系统**: Windows 10 / 11 (64-bit)
- **内存**: 建议 4GB 以上
- **磁盘**: 约 90MB（压缩包）

## 开发背景

本项目源于肺部发育生物学研究中对 AT1/AT2 肺泡上皮细胞的定量分析需求。设计目标是让**不会编程的生物学研究者**也能独立完成从图像分析到数据导出的完整流程，无需依赖生物信息学支持。

## 许可证

本项目采用 [MIT License](LICENSE) 开源协议。

## 致谢

- [scikit-image](https://scikit-image.org/) — 图像分割算法
- [PyQt5](https://www.riverbankcomputing.com/software/pyqt/) — 图形界面框架
- [Fiji/ImageJ](https://fiji.sc/) — ROI 格式兼容性参考
