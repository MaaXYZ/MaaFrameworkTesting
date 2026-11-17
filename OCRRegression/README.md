# OCR 回归数据集

[English](README_en.md)

这个数据集保存 MaaFramework 历史 OCR 问题中的原始截图及识别参数，用于验证：在同一张截图中加入其他 OCR 候选，不应改变目标 ROI 单独识别的结果。

`cases.json` 包含 11 个用例，引用 5 张未修改的原始 PNG。所有图片都保留下载时的分辨率和字节内容；2560 × 1440 的图片也没有预先缩小。每个用例记录原帖、原附件 URL、原图尺寸和 SHA-256，方便核对来源。带有 OCR 调试框或文字叠加的 draw 图片未纳入。

| 来源 | 原图 | 用例 |
| --- | --- | --- |
| [MaaFramework #1448](https://github.com/MaaXYZ/MaaFramework/issues/1448) | `1448-hbr-2026-08.png`，1280 × 720 | 小 ROI「战斗」与全屏 OCR，以及两个未越过尺寸限制的派生场景 |
| [#1448 的 5.16.0 反馈](https://github.com/MaaXYZ/MaaFramework/issues/1448#issuecomment-6829542455) | `1448-hbr-2026-11.png`，2560 × 1440 | 新截图中的同一 ROI 和全屏 OCR |
| [MaaFramework #1148](https://github.com/MaaXYZ/MaaFramework/issues/1148) | `1148-bounty.png`，1280 × 720 | 狭长 ROI「悬赏」，分别搭配全屏和相同 ROI |
| [MaaFramework #1384](https://github.com/MaaXYZ/MaaFramework/issues/1384) | `1384-purchase.png`，1280 × 720 | 两个不同 ROI 的「12 / 16」，分别作为目标 |
| [MaaFramework #1137](https://github.com/MaaXYZ/MaaFramework/issues/1137) | `1137-delivery.png`，1280 × 720 | 三个地点名称的共享 ROI，搭配另一处任务 ROI |

8 月的 `1448-hbr-2026-08.png` 与 MaaFramework 早期 `BatchOcrRepro/Screenshot/hbr_battle.png` 完全相同，SHA-256 为 `f4f087898f843e2942587c9b09fd31f67d9e74afe2830d0696b95cbc5392d97f`。

## 清单格式

顶层格式为 `{ "version": 1, "cases": [...] }`。每个用例包含：

| 字段 | 含义 |
| --- | --- |
| `id` | 稳定、唯一的用例名称 |
| `image` | 相对于 `OCRRegression` 的原图路径 |
| `image_sha256` | 原图文件的 SHA-256，小写十六进制 |
| `original_size` | 原图宽、高 |
| `image_size` | 运行识别前的目标宽、高；当前均为 `[1280, 720]` |
| `source` | 提供图片或参数的 issue / comment URL |
| `source_image` | 原始图片附件 URL |
| `recognition` | 目标 OCR 参数对象，不包含 `type` |
| `peers` | 同一帧中其他 OCR 候选的参数对象数组；`{}` 表示默认全屏 OCR |
| `notes` | 来源中的默认值、用例裁剪和派生配置说明 |

`recognition` 和 `peers` 可以直接作为 Pipeline v2 的 OCR `param`。ROI 坐标以 `image_size` 为准，格式为 `[x, y, width, height]`。`expected` 保留来源中的正则表达式列表；多语名称不代表数据集中包含其他语言的截图。

来源未指定的 `threshold`、`model` 等参数保持省略。测试使用 `MaaCommonAssets/OCR/ppocr_v4/zh_cn` 作为默认中文模型。#1448 的 `threshold: 0.9` 沿用框架已有的回归断言，清单中已明确注明。#1137 仅保存固定 ROI 的识别参数，不声称重现其动态货物 ROI 和完整任务流程。

两个 #1448 八月截图派生用例保持目标 ROI、`expected` 和 `threshold` 不变，仅分别将 peer ROI 设为 `[400, 0, 880, 720]` 和 `[1000, 180, 280, 160]`。这些 peer 参数来自回归测试设计，并非原 issue 配置；它们与目标的并集最大边均小于 960，用于验证即使未触发检测模型的降采样限制，合并 ROI 仍会改变检测上下文、框和分数。

## 运行约定

测试程序先按 `image_size` 调整输入尺寸，再运行 OCR；缩小图片使用 OpenCV `INTER_AREA`。原始 PNG 不应被测试程序覆盖。

先确认目标单独识别能命中 `expected`，再比较它与其他候选同帧执行时的完整识别结果，包括文本、分数、框、排序和筛选结果。不能把双方均未命中当作通过，也不将旧错误结果记录为正确答案。`peers` 表示同帧候选关系，不要求每个 peer 在该截图中命中。

新增素材时，应保留原始附件、填写可核查的参数出处，并同步更新中英文说明。不要用调试 draw 替代原图，也不要根据图片猜测缺失的 Pipeline 参数。
