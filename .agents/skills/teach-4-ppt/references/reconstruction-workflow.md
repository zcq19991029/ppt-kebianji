# 复现执行表：原图、OCR、资产、对象和对照修复

## A. 任务边界与输入核对

- 最终确定版原图决定布局、颜色、对象身份和插图位置；复现清单校对准确文字、数字、公式和关系；模板只决定主体放置区域。
- 记录源图绝对路径、SHA-256、宽高、比例、任务名、输出选择、模板版本、原图来源页和输出页号。
- 没有清单但原图可读时可以按原图建立清单；不得凭空补内容。多个候选任务或冲突输入必须先问。

## B. OCR 与视觉盘点

1. `editppt prepare` 生成 `text_hints.json`/`text_hints.png`；若 PaddleOCR 可用，保存后端状态。OCR 文字永远是候选，不是最终文本。
2. 对 source.png 做整图和局部查看。逐条将 OCR 框匹配到原图文字，纠正中文字符、英文大小写、数字、标点、括号、箭头文字、换行和字号组。漏检文字必须手工加入，误检装饰线必须删除。
3. 为每个前景视觉对象建立 `id`、对象类型、原图框、方向、颜色/纹理、是否含品牌文字、分离资产路径和预期放置框；为每条连线建立起点、终点、方向、箭头和语义关系。
4. 明确哪些是背景、结构和独立位图。普通文字不能藏在图片里；品牌内部文字可以作为品牌位图身份，但旁边的标题和标签仍必须原生化。

## C. 图像资产分离

- 先 `view_image` 检查 source.png，再向 `image_gen.imagegen` 提交只包含当前任务 source 和提示词的资产表请求。
- 资产表使用不与主体冲突的纯色键背景，要求保持原图对象身份、比例、笔画、颜色、方向、纹理和内部品牌文字；禁止重绘、换图标、简化或添加文字。
- `editppt image import` 记录实际后端和哈希；`process-sheet` 后查看 alpha、分割报告和 contact sheet。区域不能重叠或截断对象；失败时先调整区域/键色/提示词，再重试。
- 分离后的资产必须逐一与 `visual_inventory` 对账。对象缺失、身份错误、方向错误、背景残留、严重变形或错位都属于必须返工，不是 warning。

## D. Manifest 重建约束

- 复制 `page_request.json` 的 `slide`、`content_box` 和源尺寸，不重新计算画布，不裁切、拉伸或补白。
- `text_boxes`、`images`、`tables` 和非线条 shapes 必须有 `box_px`；线条/path 必须有 `points_px` 或 `path_px`。`text_inventory` 只是覆盖清单，不能替代定位对象。
- 同一层级文字统一字号，使用 `font_size_source: measured`；文字内容按原图最终核对。卡片、流程节点和规则箭头用 native shapes；连续曲线保持一个 path。
- `quality_checks` 至少包含 `font_size_calibrated`、`visual_inventory_matched`、`background_strategy_checked`、`shape_corner_geometry_checked`，并增加 `ocr_content_checked`、`source_render_compared`、`repair_rounds`。

## E. 三层对照验收

每次 build 后必须生成预览并执行：

1. **文本层**：逐项比对 OCR、清单和原图；检查漏字、错字、数字、标点、断行、字号、位置和中英文混排。
2. **结构层**：检查卡片、分区、路径、箭头、端点、分支、循环和 z-index；确认图片没有覆盖文字，文字没有重复出现在图片和 native 对象中。
3. **视觉层**：source 与 preview 并排或 contact sheet 对照；检查对象身份、方向、比例、边缘、颜色意义、遮挡、裁切、空白、对齐和相邻关系。局部批注必须逐条复核。

确定问题后：定位所属 manifest/资产/文字框 → 只修相关对象 → 重新 build/render/validate → 再做受影响区域和整页对照。最多连续三轮未改善时停止并报告根因，不把失败结果交付。

## F. 必须保存的验收证据

- `validation.json`：顶层 `passed: true`、结构统计、缺失文本/资产/关系和 warnings。
- `page_result.json`、`split_assets_contact.png`、最终预览、最终 deck validation。
- `验证报告.md`：源图哈希、OCR 后端、文字/视觉对照结论、修复轮次、编辑范围、位图清单、已知差异、Office/WPS/OfficeCLI 实际验证边界。
- 批注返工时保留批注对象 ID/坐标、原图证据、修复前后结论；不删除原始输入。
