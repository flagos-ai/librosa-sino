@../common/CLAUDE.md

# librosa — CNPort 移植项目

## 项目简介

librosa 是一个用于音乐和音频分析的 Python 软件包，版本 **0.11.0**。

主要功能包括：
- 音频加载与流式处理（`load`, `stream`, `resample`）
- 时域处理（`autocorrelate`, `lpc`, `zero_crossings`）
- 频谱表示（`stft`, `istft`, `reassigned_spectrogram`）
- 特征提取（MFCC, 梅尔频谱, 色度特征等）
- 节拍与节奏分析
- 音乐信号分解与分段

- upstream: https://github.com/librosa/librosa
- 公司仓库: code.iflytek.com/HY_CNPort/librosa.git
- 主跟踪分支: `main`
- 许可证: ISC

---

## 安装与构建命令

### 从源码安装（开发模式，推荐）

```bash
python -m pip install -e .
```

### 从源码构建后安装

```bash
python setup.py build
python setup.py install
```

### 安装运行时依赖

```bash
pip install audioread>=2.1.9 numba>=0.51.0 numpy>=1.22.3 scipy>=1.6.0 \
    scikit-learn>=1.1.0 joblib>=1.0 soundfile>=0.12.1 soxr>=0.3.2 \
    pooch>=1.1 lazy_loader>=0.1 msgpack>=1.0
```

### 安装测试依赖

```bash
pip install -e ".[tests]"
# 等价于:
pip install matplotlib>=3.5.0 packaging>=20.0 pytest-mpl pytest-cov pytest \
    samplerate resampy>=0.2.2 types-decorator
```

### 安装文档构建依赖

```bash
pip install -e ".[docs]"
```

### 系统音频解码依赖（Linux）

```bash
# apt-get 系统
apt-get install ffmpeg
# 或 GStreamer
apt-get install gstreamer1.0-plugins-base gstreamer1.0-plugins-ugly

# yum 系统
yum install ffmpeg
```

### 验证安装

```python
import librosa
librosa.show_versions()
```

---

## 测试命令

### 运行全部测试

```bash
pytest tests/
```

### 运行单个测试文件

```bash
pytest tests/test_core.py
pytest tests/test_features.py
pytest tests/test_beat.py
```

### 带覆盖率报告运行测试

```bash
pytest --cov=librosa --cov-report=term-missing tests/
```

### 常用测试选项（来自 setup.cfg）

```bash
# setup.cfg 中已配置:
# --cov-report term-missing --cov librosa --cov-report=xml
# --mpl --mpl-baseline-path=tests/baseline_images/test_display
pytest tests/
```

### 代码风格检查

```bash
pycodestyle librosa/   # 最大行长 119，忽略 E203,W503
flake8 librosa/        # 仅检查 E9, F63, F7, F82
mypy librosa/          # 类型检查（strict 模式）
```

---

## 重要目录结构

```
librosa/
├── librosa/                  # 核心 Python 包
│   ├── __init__.py           # 公开 API 入口，包含所有子模块导出
│   ├── core/                 # 核心 DSP 功能
│   │   ├── audio.py          # 音频加载、重采样（load, resample, stream）
│   │   ├── spectrum.py       # 频谱变换（stft, istft, cqt, vqt）
│   │   ├── fft.py            # FFT 后端管理（支持多种 FFT 库）
│   │   ├── constantq.py      # 常Q变换
│   │   ├── convert.py        # 单位转换（hz_to_mel, frames_to_time 等）
│   │   ├── pitch.py          # 音高估计
│   │   ├── harmonic.py       # 谐波/打击乐分离
│   │   ├── notation.py       # 音乐符号
│   │   └── intervals.py      # 音程处理
│   ├── feature/              # 特征提取
│   │   ├── spectral.py       # 频谱特征（mfcc, melspectrogram, chroma 等）
│   │   ├── rhythm.py         # 节奏特征（tempogram）
│   │   ├── inverse.py        # 特征逆变换
│   │   └── utils.py          # 特征工具函数
│   ├── util/                 # 工具函数
│   │   ├── utils.py          # 通用工具（normalize, pad_center 等）
│   │   ├── files.py          # 文件操作、示例数据
│   │   ├── matching.py       # 序列匹配
│   │   ├── _nnls.py          # 非负最小二乘
│   │   ├── decorators.py     # 函数装饰器
│   │   ├── deprecation.py    # 弃用警告
│   │   └── exceptions.py     # 自定义异常
│   ├── beat.py               # 节拍追踪
│   ├── decompose.py          # 信号分解（NMF, HPSS）
│   ├── display.py            # 可视化（需要 matplotlib）
│   ├── effects.py            # 音频效果（pitch_shift, time_stretch）
│   ├── filters.py            # 滤波器组（mel, chroma, constant-Q）
│   ├── onset.py              # 起始点检测
│   ├── segment.py            # 音频分段
│   ├── sequence.py           # 序列处理（Viterbi 等）
│   ├── version.py            # 版本号（0.11.0）
│   └── _cache.py             # 计算结果缓存
├── tests/                    # 测试套件
│   ├── test_core.py          # 核心功能测试
│   ├── test_features.py      # 特征提取测试
│   ├── test_beat.py          # 节拍测试
│   ├── test_display.py       # 可视化测试
│   ├── test_effects.py       # 音效测试
│   ├── test_filters.py       # 滤波器测试
│   ├── conftest.py           # pytest 配置与 fixtures
│   ├── baseline_images/      # 可视化测试基准图像
│   └── data/                 # 测试音频数据
├── docs/                     # 文档（Sphinx）
│   ├── conf.py               # Sphinx 配置
│   ├── BUILD.md              # 文档构建说明
│   └── examples/             # 高级示例
├── scripts/                  # 维护脚本
├── setup.py                  # 构建入口
├── setup.cfg                 # 包元数据、依赖、pytest/flake8/mypy 配置
└── pyproject.toml            # 构建系统配置（setuptools + wheel）
```

---

## 移植相关注意事项

### 分支结构

| 分支 | 说明 |
|------|------|
| `main` | 跟踪 upstream (github.com/librosa/librosa) 的主分支 |
| `platform/cuda` | NVIDIA GPU (CUDA) 移植分支 |
| `platform/amd` | AMD GPU (ROCm/HIP) 移植分支 |
| `platform/apple` | Apple Silicon (Metal/MPS) 移植分支 |

### 关键移植关注点

1. **numba JIT 加速**: librosa 核心计算（尤其是 CQT、动态规划等）大量使用 `numba` JIT 编译。不同平台需确认 numba 版本与硬件加速后端的兼容性。
   - CUDA 平台：确认 `numba.cuda` 可用且版本匹配
   - AMD 平台：numba 对 ROCm 支持有限，可能需要 CPU fallback 或替换实现
   - Apple 平台：numba 在 Apple Silicon 上支持有限，需测试 `@jit` 函数行为

2. **FFT 后端** (`librosa/core/fft.py`): librosa 支持多种 FFT 实现，可通过 `librosa.set_fftlib()` 切换（注：0.11.0 已弃用）。平台移植时可针对性优化 FFT 后端：
   - CUDA：可替换为 `cuFFT`（通过 cupy 或 torch.fft）
   - AMD：可使用 `rocFFT`
   - Apple：可使用 `vDSP`（Accelerate 框架）

3. **numpy/scipy 依赖**: librosa 大量依赖 numpy 和 scipy，各平台需确认加速库（如 OpenBLAS、MKL、Apple Accelerate）正确链接。

4. **音频后端** (`librosa/core/audio.py`): 依赖 `soundfile` 和 `audioread`，在非 x86 平台上需确认 `libsndfile` 的编译支持。

5. **soxr 重采样**: 依赖 `soxr`（Sox Resampler），需在各平台上验证编译可用性。

### 移植开发工作流

```bash
# 切换到目标平台分支
git checkout platform/cuda   # 或 platform/amd / platform/apple

# 只提交平台特有的差异（构建脚本、硬件加速代码）
# 原则：平台分支的 diff 越小，rebase 越轻松

# 每周同步 upstream
cd /data/xuchen2/git/CNPort
./sync-upstream.sh librosa
# 或同步全部仓库
./sync-upstream.sh
```

### 推送到公司仓库（必须绕过代理）

```bash
no_proxy="code.iflytek.com" NO_PROXY="code.iflytek.com" git push origin main
no_proxy="code.iflytek.com" NO_PROXY="code.iflytek.com" git push origin platform/cuda
no_proxy="code.iflytek.com" NO_PROXY="code.iflytek.com" git push origin platform/amd
no_proxy="code.iflytek.com" NO_PROXY="code.iflytek.com" git push origin platform/apple
```

### Remote 配置

```bash
git remote -v
# origin   code.iflytek.com/HY_CNPort/librosa.git  (公司仓库)
# upstream https://github.com/librosa/librosa       (开源上游)
```

### 验证移植是否正常

```bash
# 基础功能验证
python -c "import librosa; librosa.show_versions()"

# 运行核心测试（跳过需要网络的测试）
pytest tests/test_core.py tests/test_features.py -x -v

# 运行完整测试套件
pytest tests/ -v --tb=short
```
