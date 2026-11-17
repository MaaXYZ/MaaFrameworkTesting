# OCR regression corpus

[简体中文](README.md)

This corpus preserves original screenshots and recognition parameters from historical MaaFramework OCR reports. It verifies that adding other OCR candidates on the same screenshot does not change the result obtained by recognizing the target ROI alone.

`cases.json` contains 11 cases referencing 5 unchanged original PNGs. All images retain their downloaded resolution and bytes, including the 2560 × 1440 image. Each case records the source post, attachment URL, original dimensions, and SHA-256 so its provenance can be checked. Debug draw images containing OCR boxes or text overlays are excluded.

| Source | Original image | Cases |
| --- | --- | --- |
| [MaaFramework #1448](https://github.com/MaaXYZ/MaaFramework/issues/1448) | `1448-hbr-2026-08.png`, 1280 × 720 | Small ROI containing “战斗” alongside full-screen OCR, plus two derived scenarios below the size limit |
| [The #1448 v5.16.0 follow-up](https://github.com/MaaXYZ/MaaFramework/issues/1448#issuecomment-6829542455) | `1448-hbr-2026-11.png`, 2560 × 1440 | The same ROI and full-screen OCR on a newer screenshot |
| [MaaFramework #1148](https://github.com/MaaXYZ/MaaFramework/issues/1148) | `1148-bounty.png`, 1280 × 720 | Narrow ROI containing “悬赏”, with full-screen and identical-ROI peers |
| [MaaFramework #1384](https://github.com/MaaXYZ/MaaFramework/issues/1384) | `1384-purchase.png`, 1280 × 720 | Two different ROIs searching for “12 / 16”, each used as the target |
| [MaaFramework #1137](https://github.com/MaaXYZ/MaaFramework/issues/1137) | `1137-delivery.png`, 1280 × 720 | Three location names sharing an ROI, alongside a separate mission ROI |

The August `1448-hbr-2026-08.png` is byte-for-byte identical to MaaFramework's earlier `BatchOcrRepro/Screenshot/hbr_battle.png`. Its SHA-256 is `f4f087898f843e2942587c9b09fd31f67d9e74afe2830d0696b95cbc5392d97f`.

## Manifest format

The top-level format is `{ "version": 1, "cases": [...] }`. Each case contains:

| Field | Meaning |
| --- | --- |
| `id` | Stable, unique case identifier |
| `image` | Original image path relative to `OCRRegression` |
| `image_sha256` | SHA-256 of the original file, in lowercase hexadecimal |
| `original_size` | Original width and height |
| `image_size` | Target width and height before recognition; currently `[1280, 720]` for all cases |
| `source` | Issue or comment URL providing the image or parameters |
| `source_image` | Original image attachment URL |
| `recognition` | Target OCR parameter object, without `type` |
| `peers` | Array of OCR parameter objects for other candidates on the same frame; `{}` means default full-screen OCR |
| `notes` | Notes about source defaults, reduced scenarios, and derived configuration |

`recognition` and `peers` can be used directly as Pipeline v2 OCR `param` objects. ROI coordinates refer to `image_size` and use `[x, y, width, height]`. `expected` preserves the source's regular-expression lists; multilingual names do not imply that screenshots in all those languages are included.

Parameters such as `threshold` and `model` remain omitted when the source did not specify them. Tests use `MaaCommonAssets/OCR/ppocr_v4/zh_cn` as the default Chinese model. The `threshold: 0.9` for #1448 follows the framework's existing regression assertion and is explicitly identified in the manifest. The #1137 cases preserve only the fixed-ROI recognition parameters; they do not claim to reproduce its dynamic cargo ROIs or complete task flow.

Two derived cases use the August #1448 screenshot without changing the target ROI, `expected`, or `threshold`. Their peer ROIs are `[400, 0, 880, 720]` and `[1000, 180, 280, 160]`. These peer parameters were designed for regression testing and were not reported in the original issue. Each union with the target has a maximum side below 960, covering changes to detection context, boxes, and scores even when merging does not cross the detector's downsampling limit.

## Execution contract

The test runner first adjusts the input to `image_size`, then runs OCR. Downscaling uses OpenCV `INTER_AREA`. Tests must not overwrite the original PNGs.

First verify that standalone recognition matches the target's `expected`, then compare its complete recognition results when other candidates execute on the same frame, including text, score, boxes, ordering, and filtering. Two misses do not constitute a passing test, and historical incorrect output is not a golden answer. `peers` describes the candidate relationship; each peer need not match this screenshot.

When adding material, retain the original attachment, provide verifiable parameter sources, and update both language versions of this documentation. Do not substitute debug draw images for originals or infer missing Pipeline parameters from an image.
