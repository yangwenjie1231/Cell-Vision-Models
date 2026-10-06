# Cell-Vision-Models

[YWJCell / Cell-Vision-Pro](https://github.com/yangwenjie1231/Cell-Vision-Pro-v2) 的**用户端 GPU 推理模型权重**（ONNX）。

应用不把这些权重打进安装包（单个模型可达数十 MB，会让安装包显著变大，而且 asar 是只读的、模型没法替换），
改为**按需下载**：需要时由应用下载到本机 `<userData>/models/` 目录。

## 为什么单独开一个仓库

1. **安装包体积**：把模型留在应用仓库里会让每次 clone 都很重
2. **可替换**：用户可以自带权重，放在 `userData/models/` 下即可覆盖下载的版本
3. **便于镜像**：单仓库 + 固定文件名，方便同步到 Gitee 等国内镜像

## 模型清单

权重以 **Release 附件**形式提供（不进 git 历史）。

| 文件名 | 模型 | 大小 | 用途 |
|---|---|---|---|
| `realesrgan_x2_v1.onnx` | Real-ESRGAN x2（fp32，动态 H/W） | 64.0 MB | 图像增强（超分），**仅用于显示** |

### 校验值

```
realesrgan_x2_v1.onnx
  size    : 67073379
  sha256  : 8c585725403a52b50b2cc883042c29a3add0f8c3c35e4e98dc51f113d9a293d5
```

## 下载地址

应用通过 `ai.model.download_base` 设置项拼接下载地址，格式为：

```
<downloadBase>/<modelId>.onnx
```

该设置项**支持多个地址**（逗号或换行分隔），应用会**按顺序回退** ——
默认是 Gitee 优先、GitHub 兜底：

```
https://gitee.com/ywjgame/Cell-Vision-Models/releases/download/v1,
https://github.com/yangwenjie1231/Cell-Vision-Models/releases/download/v1
```

### 国内镜像（Gitee）

镜像仓库：<https://gitee.com/ywjgame/Cell-Vision-Models>

## ⚠️ 为什么模型走 Release 附件，而不是直接放进仓库

**Gitee 社区版有硬性配额限制**（[官方配额说明](https://help.gitee.com/account/usage-quota)）：

| 配额项 | 限制 | 本模型（64.0 MB） |
|---|---|---|
| 仓库**单文件**大小 | **50 MB** | ❌ 超限，放不进仓库 |
| 仓库单仓库容量 | 500 MB | |
| **附件**单文件大小 | **100 MB** | ✅ 可以 |
| 附件单仓库总容量 | 1 GB | |

所以：**模型放不进 Gitee 仓库**（64MB > 50MB），但可以走 **Release 附件**（上限 100MB）。

另外即使放得进，也不建议：git 历史会**永久保留每个版本的模型**，换一次模型仓库就只增不减
（Gitee 个人总容量才 5GB）。

### 代价：Gitee 不自动同步 Release

Gitee 的「仓库镜像」只同步 **git 内容**（代码/README），**不会同步 Release 及其附件**。
所以每次在 GitHub 发了新 release 后，需要在 Gitee **手动建一个同名 tag 的 release 并上传附件**：

1. 打开 <https://gitee.com/ywjgame/Cell-Vision-Models/releases/new>
2. Tag 填 `v1`（与 GitHub 保持一致，应用按这个路径拼地址）
3. 上传附件 `realesrgan_x2_v1.onnx`（文件名必须完全一致）
4. 发布

> 如果 Gitee release 还没建好，应用会自动回退到 GitHub 源（国内可能较慢或被墙）。


## ⚠️ 关于 `realesrgan_x2_v1` 的重要提醒

Real-ESRGAN 是 **GAN 生成式**模型 —— 它会"编造"输入中不存在的细节。

- ✅ 适合：给人看的展示图、报告配图
- ❌ **不要**用于定量分析（细胞计数、面积、融合度等）—— 会让统计结果失真

因此应用里它默认**关闭**，且只作为「显示增强」的可选项。

## 权重来源与许可

`realesrgan_x2_v1.onnx` 由 [ai-forever/Real-ESRGAN](https://huggingface.co/ai-forever/Real-ESRGAN)
的 `RealESRGAN_x2.pth` 导出（fp32 + 动态 H/W，IR 7 / opset 13），
导出脚本见应用仓库的 `scripts/export-realesrgan.py`。

Real-ESRGAN 采用 **BSD-3-Clause** 许可。

## 自己导出模型

如果官方 release 里没有你需要的模型，可以在本地导出后自行使用（不一定要放进本仓库）：

```bash
# 1. 下载官方权重
curl -L -o RealESRGAN_x2.pth \
  https://hf-mirror.com/ai-forever/Real-ESRGAN/resolve/main/RealESRGAN_x2.pth

# 2. 导出 ONNX（fp32 + 动态 H/W）
python scripts/export-realesrgan.py RealESRGAN_x2.pth --scale 2

# 3. 放到应用的模型目录（文件名要与 ai_models.path 里写的一致）
#    Windows: %APPDATA%/cell-vision-pro/models/realesrgan_x2_v1.onnx
```
