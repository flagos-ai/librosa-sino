# librosa CNPort

librosa CNPort 基于上游 [librosa](https://github.com/librosa/librosa)，面向国产化软硬件环境的音频与音乐分析能力验证。在保持上游 Python API、音频处理语义和测试基线的前提下，本仓库跟踪阿里云 Codeup 的 `patches/base` 分支，并为 GitHub 开源发布补充国产化背景、评估口径和跨平台测试记录。

上游原始说明保存在 [README_UPSTREAM.md](README_UPSTREAM.md)，许可证和引用信息以 [LICENSE.md](LICENSE.md) 及上游项目说明为准。

当前同步基线：`patches/base` / `origin/patches/base`，提交 `f146a3f9`。上游项目：[librosa/librosa](https://github.com/librosa/librosa)。

## 国产化背景

librosa 是 Python 原生音频和音乐信息检索库，覆盖音频加载、重采样、短时傅里叶变换、滤波器组、节拍/音高/起始点分析、和弦与谱特征、可视化等能力。其核心计算主要依赖 NumPy、SciPy、Numba、SoundFile、soxr、resampy 和 Matplotlib 等科学计算组件。

国产化工作的重点不是改变算法，而是验证同一份源码在不同处理器、操作系统、Python 解释器及科学计算依赖组合上的可安装性、功能正确性、数值稳定性和测试可复现性：

- 保持上游 API、函数签名、输入输出语义和测试数据不变；
- 优先验证原生 CPU/Python 路径，不以某一厂商的专用加速后端作为通过前提；
- 在 NVIDIA、华为昇腾、海光、寒武纪、摩尔线程和阿里平头哥等环境分别执行，结果独立记录；
- 通过 Codeup `patches/base` 作为同步基线，保留上游 README，方便后续差异审阅和版本同步。

## 安装与使用

基础安装方式与上游保持一致：

```bash
python -m pip install librosa
# 或从当前源码安装
python -m pip install -e .
```

各平台建议使用独立虚拟环境或容器，并按平台软件栈安装匹配的 Python、NumPy、SciPy、Numba、SoundFile、soxr、resampy、Matplotlib 和 ffmpeg。完整 API、示例及参数说明请参阅 [README_UPSTREAM.md](README_UPSTREAM.md) 和 [官方文档](https://librosa.org/doc/)。

## 国产化评估方案

验收以“固定源码提交 + 平台原生运行环境 + 同一测试数据”为基本单元，不将不同平台的耗时或浮点实现差异直接判定为功能失败。

| 层级 | 验证内容 | 通过标准 |
|---|---|---|
| 环境与安装 | Python、依赖导入、editable install、版本信息 | 依赖可导入，librosa 可安装和调用 |
| 原生全量测试 | 上游 `tests/` 的完整 pytest 套件 | 返回码为 0，失败数和错误数为 0 |
| 测试数据与网络边界 | 内置 `tests/data`、隔离测试、网络测试分别执行 | 明确记录跳过项、xfail/xpass 和网络依赖 |
| 跨平台一致性 | 在各国产化平台重复全量测试 | 各平台独立通过；失败需记录环境、用例和原因 |

## 使用 Docker 镜像执行原生全量测试

为保证测试环境稳定、隔离且可复现，原生全量测试应在固定版本的 Docker 镜像中执行。镜像预置 Python 3.10 及 librosa 测试所需的科学计算依赖；测试时挂载固定源码 worktree 和 `tests/data`，不在宿主机安装依赖，也不直接修改主 checkout。

当前标准镜像按宿主机架构选择：

```text
amd64: open-audio-native-registry.cn-beijing.cr.aliyuncs.com/hy_cnport/librosa-cpu-py310:v1.0.0-amd64
arm64: open-audio-native-registry.cn-beijing.cr.aliyuncs.com/hy_cnport/librosa-cpu-py310:v1.0.0-arm64
```

建议固定使用镜像 tag 对应的 digest，并在测试报告中记录完整镜像地址、digest、源码提交、宿主机架构和依赖版本。公网环境使用上述地址；已接入阿里云 VPC 的机器可将仓库域名替换为 `open-audio-native-registry-vpc.cn-beijing.cr.aliyuncs.com`。

### Docker 全量测试命令

```bash
# 拉取与宿主机架构匹配的固定镜像

docker pull --platform linux/amd64 \
  open-audio-native-registry.cn-beijing.cr.aliyuncs.com/hy_cnport/librosa-cpu-py310:v1.0.0-amd64

# 在独立源码 worktree 中执行全量测试

docker run --rm --platform linux/amd64 \
  -v /path/to/librosa-worktree:/workspace/librosa \
  -w /workspace/librosa \
  -e MPLBACKEND=Agg \
  open-audio-native-registry.cn-beijing.cr.aliyuncs.com/hy_cnport/librosa-cpu-py310:v1.0.0-amd64 \
  bash -lc 'python -m pytest tests/ -v -ra'
```

arm64 主机将命令中的平台和镜像替换为 `linux/arm64` 及 arm64 镜像。测试完成后应保存 pytest 日志和汇总结果，至少记录收集、通过、失败、跳过、xfailed、xpassed 数量、耗时及失败用例详情。网络依赖测试应单独标记；离线验收使用仓库内置测试数据，不将网络可用性混入功能结论。

## 平台测试结果

下表汇总当前已保存的原生测试记录。`通过/收集` 只表示 pytest 通过用例数与收集用例数；`xfailed`、`xpassed` 和 `skipped` 按 pytest 原始结果保留。未通过或未完成的平台不应被描述为已支持。

| 平台 | 最近记录 | 测试范围 | 结果 | 结论 |
|---|---|---|---|---|
| Codeup `patches/base` amd64 工作区 | 2026-10-06 | full，14,544 collected | ✅ 13,935 passed，0 failed，43 skipped，528 xfailed，38 xpassed | 当前基线全量复验通过 |
| NVIDIA A40 | 2026-10-13 | full，14,544 collected | ✅ 13,975 passed，0 failed，2 skipped，529 xfailed，38 xpassed | 原生全量通过 |
| 华为昇腾 CANN 8.5 | 2026-06-01 | unit+integration，收集阶段 | ❌ collection error：`No librosa.core attribute convert` | 需修复环境/源码导入问题后复测 |
| 海光 BW1000 | 2026-10-13 | full，14,544 collected | ✅ 13,975 passed，0 failed，2 skipped，529 xfailed，38 xpassed，191 warnings，456.07 s | 原生全量通过 |
| 寒武纪 MLU590 | 2026-10-13 | full，14,544 collected | ✅ 13,975 passed，0 failed，2 skipped，523 xfailed，44 xpassed | 原生全量通过 |
| 摩尔线程 S5000 | 2026-10-13 | full，14,544 collected | ✅ 13,975 passed，0 failed，2 skipped，529 xfailed，38 xpassed，191 warnings，418.33 s | 原生全量通过 |
| 阿里平头哥 | 2026-07-23 | full，14,544 collected | ✅ 13,935 passed，0 failed，43 skipped，528 xfailed，38 xpassed | 原生全量通过 |

本次 Codeup `patches/base` 行是在独立 worktree 中完成的全量源码复验；平台历史结果文件位于本地测试归档目录 `common/general/survey_result/librosa-survey/`。发布前如重新执行，应以新报告中的源码提交和镜像 digest 更新本表。不同平台的测试数量只有在相同测试基线下才可比较，不能将某个平台的结果推断到其他平台。

## 上游与许可证

- 上游项目：[librosa/librosa](https://github.com/librosa/librosa)
- 上游 README：[README_UPSTREAM.md](README_UPSTREAM.md)
- 许可证：[LICENSE.md](LICENSE.md)

本仓库的国产化适配与测试记录不改变上游代码许可证、音频数据许可证或第三方依赖许可证。使用前请同时阅读上游项目及相关依赖的许可条款。
