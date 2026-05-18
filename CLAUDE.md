# CLAUDE.md — Project Notes

> 給未來的 Claude / 同學參考。內容包含本次作業執行記錄與待辦事項。

## 專案性質

- 課程：2026 NTU CI/CD Lab
- 作業頁：https://hackmd.io/@sychen6192/HJv9-CHCZe
- 繳交截止：2026-05-19 00:00
- 主程式：Fastify + TypeScript，搭配 vitest 測試、prettier 格式化、eslint lint

## 作業要求對照

| 項目                                             | 對應實作                                                                                 |
| ------------------------------------------------ | ---------------------------------------------------------------------------------------- |
| 1. `.github/workflows/ci_{學號}.yaml` 工作流程檔 | `.github/workflows/ci_b11705058.yaml`                                                    |
| 2. Push 自動觸發                                 | `on.push.branches: ['**']` + `pull_request` + `workflow_dispatch`                        |
| 3. 三項必要檢查                                  | `npm run typecheck` / `npm run format:check` / `npm test`                                |
| 4. 失敗即顯示 fail                               | 各 step 直接 fail（不加 `continue-on-error`），整個 job 會紅燈                           |
| 5. 測試結果可看                                  | vitest JUnit reporter → `actions/upload-artifact` + `$GITHUB_STEP_SUMMARY` markdown 表格 |

## 本次執行做了什麼（2026-05-18）

1. 讀作業說明、scan 整個 repo（src、test、snippets、Dockerfile、package.json、.prettierrc、tsconfig.json）。
2. `npm ci` 安裝相依。
3. 跑 `npm run typecheck` / `npm run format:check` / `npm test`，發現 prettier 對 3 個檔案有格式問題：
   - `docker-compose.yml`（雙引號 → 單引號）
   - `snippets/01_hello.yaml`、`snippets/02_run-test.yaml`（縮排）
   - 用 `npx prettier --write` 修正完畢，現在三項檢查全綠。
4. 建立 `.github/workflows/ci_b11705058.yaml`：
   - 觸發：`push` 任何 branch、`pull_request`、`workflow_dispatch`
   - 步驟：checkout → setup-node (22, npm cache) → `npm ci` → typecheck → prettier → vitest（產生 JUnit XML）→ upload artifact → 把測試統計寫進 step summary
   - 用 `if: always()` 確保即使 test fail 也會上傳 artifact 與寫 summary
5. 更新 README.md 加入「繳交說明」段落。
6. 建立這份 CLAUDE.md。

## 接下來的 TODO（學生要做）

- [ ] 到 GitHub Actions 頁面確認綠燈，截圖（要看到 typecheck / format check / test 三個 step 都成功）。
- [ ] 製造一個失敗 case（建議方法見 README 的「示範失敗 case」段落），push 後截圖，再 revert。
- [ ] 寫 PDF 報告：`學號_姓名_CICD_作業.pdf`，內容包含：
  - Pipeline 設計說明
  - 工作流程檔內容（貼 yaml）
  - 成功截圖
  - 失敗截圖 + 錯誤原因 + 修正方式

## 注意事項

- `reports/` 已在 `.gitignore` 裡，本地測試產生的 JUnit XML 不會被追蹤。
- prettier rule（單引號、2 空白縮排）對 yaml 也生效，新增 workflow 檔請遵守。
- 不要在 `package.json` 改 script 名稱，CI 是用 `npm run typecheck` / `format:check` / `test` 呼叫的。
