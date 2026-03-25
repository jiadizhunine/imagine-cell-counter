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
</p>

---

## 简介

**Imagine** 是一款专为生物学研究者设计的桌面应用程序，用于自动化分析荧光显微镜图像中的细胞计数与共定位分析。支持 Zeiss `.czi` 和 `.tif/.tiff` 格式，能够从文件名自动识别抗体标记和荧光通道，并根据实验设计智能选择分析策略。

传统工具（如 Fiji/ImageJ）在处理不规则边缘的荧光信号时效果有限，且批量处理需要大量手动操作。Imagine 通过自适应阈值、分水岭分割和局部区域分析算法，显著提升了不规则胞质信号（如 GFP、HOPX）的检测准确性，并支持一键批量处理数百张图像。

## 功能特性

### 🔬 智能通道识别

从文件名自动解析抗体-荧光对应关系：

| 文件名关键词 | 识别为 | 通道颜色 |
|:---|:---|:---|
| `Ki67-568` | Ki67 | 红色 (568nm) |
| `proSpc-647` / `proSPC-647` | ProSPC | 白色 (647nm) |
| `Hopx-488` / `HOPX-488` | HOPX | 绿色 (488nm) |
| `aSMA-568` | α-SMA | 红色 (568nm) |
| `GFP` | GFP | 绿色 (488nm) |
| DAPI | DAPI | 蓝色 (始终存在) |

### 📊 自动分析策略

根据检测到的标记物组合，自动选择适用的分析方案：

| 分析类型 | 所需标记 | 计算内容 |
|:---|:---|:---|
| **AT2 数量** | ProSPC + DAPI | ProSPC⁺ 细胞计数 |
| **AT2 增殖** | ProSPC + Ki67 + DAPI | Ki67⁺ProSPC⁺ / ProSPC⁺ |
| **AT1 数量** | HOPX + DAPI | HOPX⁺ 细胞计数 |
| **AT1 增殖** | HOPX + Ki67 + DAPI | Ki67⁺HOPX⁺ / HOPX⁺ |

### 🧮 核心算法

- **DAPI 核分割**：高斯模糊 → Otsu 自动阈值 → 填洞 → 距离变换 → 分水岭分离粘连核
- **核定位标记检测**（如 Ki67）：计算核区域内信号平均强度
- **胞质标记检测**（如 HOPX、ProSPC）：膨胀核区域生成环形 ROI，计算环内信号强度
- **向量化优化**：使用 `np.bincount` 和 bbox 局部处理，单张图像分析 < 1 秒

### 🖊️ 标注功能

- **导入 Fiji 标注**：自动读取 ImageJ ROI 格式的点标注
- **手动标注**：右键点击添加/删除标注点（黄色十字 + 序号）
- **撤回**：`Ctrl+Z` 撤回上一步标注
- **导出**：`Ctrl+S` 导出为 Fiji 兼容的 TIF 文件（含嵌入 ROI 数据）

### 📁 批量处理

- 选择文件夹一键处理所有 `.czi` / `.tif` 文件
- 实时进度显示
- 结果表格支持**双击跳转**查看对应图像
- 导出 CSV 汇总表（含所有策略结果）

### 🎨 现代化界面

- 深色主题设计
- 通道彩色切换按钮（蓝/红/绿/白）
- 鼠标滚轮缩放 + 拖拽平移
- 动态适配：左侧参数面板、叠加层按钮、右侧结果均根据检测到的标记自动调整
- 文件导航：`←` `→` 快速切换上/下一张

## 截图

> *在此添加应用程序截图*

## 安装与使用

### 方法一：直接下载（推荐）

1. 从 [Releases](../../releases) 页面下载最新版本的压缩包
2. 解压到任意目录
3. 双击 `Imagine.exe` 运行

> 无需安装 Python 或任何依赖，开箱即用。

### 方法二：从源码运行

```bash
# 克隆仓库
git clone https://github.com/jiadizhunine/Imagine.git
cd Imagine

# 安装依赖
pip install numpy opencv-python scikit-image scipy czifile PyQt5

# 运行
python cell_counter.py
```

### 方法三：从源码打包

```bash
pip install pyinstaller
pyinstaller --onedir --windowed --name Imagine --icon=imagine.ico cell_counter.py
```

## 使用指南

### 基本流程

1. **打开文件**：`Ctrl+O` 或菜单 → 文件 → 打开文件
2. **查看预览**：自动显示四通道合成图像，可切换各通道显示
3. **调整参数**：左侧面板可调节各通道阈值、核面积等参数
4. **开始分析**：点击"开始分析"或 `Ctrl+R`
5. **查看结果**：右侧面板显示各策略的计数与百分比
6. **批量处理**：`Ctrl+B` 选择文件夹批量分析
7. **导出结果**：`Ctrl+E` 导出 CSV 汇总表

### 快捷键

| 快捷键 | 功能 |
|:---|:---|
| `Ctrl+O` | 打开文件 |
| `Ctrl+B` | 批量处理 |
| `Ctrl+R` | 开始分析 |
| `Ctrl+E` | 导出 CSV |
| `Ctrl+S` | 导出为 TIF（含标注） |
| `Ctrl+Z` | 撤回标注 |
| `←` `→` | 上/下一张图片 |
| 右键单击 | 添加/删除标注点 |
| 滚轮 | 缩放图像 |
| 左键拖拽 | 平移图像 |

### 参数说明

| 参数 | 默认值 | 说明 |
|:---|:---|:---|
| DAPI 阈值 | 自动 (Otsu) | 核分割的灰度阈值，勾选自动时使用 Otsu 算法 |
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
- **打包**: PyInstaller

## 系统要求

- **操作系统**: Windows 10 / 11 (64-bit)
- **内存**: 建议 4GB 以上
- **磁盘**: 约 160MB（不含数据文件）

## 开发背景

本项目源于肺部发育生物学研究中对 AT1/AT2 肺泡上皮细胞的定量分析需求。传统的 Fiji/ImageJ 工作流在以下场景表现不佳：

- **不规则胞质信号**（如 HOPX、GFP）的边缘检测不准确
- **批量处理**需要编写宏脚本，学习成本高
- **多策略分析**（数量 + 增殖率）需要多次手动操作

Imagine 针对这些痛点，提供了开箱即用的自动化解决方案。

## 许可证

本项目采用 [MIT License](LICENSE) 开源协议。

## 致谢

- [scikit-image](https://scikit-image.org/) — 图像分割算法
- [PyQt5](https://www.riverbankcomputing.com/software/pyqt/) — 图形界面框架
- [Fiji/ImageJ](https://fiji.sc/) — ROI 格式兼容性参考
