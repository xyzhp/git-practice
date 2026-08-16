## P0 — 挡路问题

### 1. 前端构建失败:悬空导入 `@/ocr`

- **问题**
  `frontend/src/utils/reportAnalysis.ts:7` 存在 `import type { OcrIndicator } from '@/ocr';`,但项目中**不存在** `src/ocr` 模块(已用 Glob 确认无 `src/ocr*` 文件)。`vue-tsc -b`(即 `npm run build`)会因此报错,前端无法构建。该文件在健康计划关键路径上(`HealthPlanView.vue:28` 引用 `analyzeReportForPlan`)。

- **原因**
  最近一轮重构删除了 `api/ocr.ts` 等文件并将逻辑内联,但 `reportAnalysis.ts` 的类型导入没有同步更新,仍指向旧模块路径 `@/ocr`。

- **解决方案**
  `OcrIndicator` 的真实类型就是 `OcrItem`(`types/conversation.ts:143`,含 `name` / `value?` / `referenceRange?` / `unit?` / `confidence`),与 `reportAnalysis.ts` 里的用法(`ind.value`、`ind.referenceRange`、`ind.name`)完全吻合。修改如下:

  ```ts
  // reportAnalysis.ts:7
  import type { OcrItem } from '@/types/conversation';  // 原为: from '@/ocr'
  ```

  并将第 36 行函数签名中的 `OcrIndicator[]` 改为 `OcrItem[]`。

  **验证**:`cd frontend && npm run build` 通过。