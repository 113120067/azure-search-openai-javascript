# MVP 執行計畫

**目的**：快速建立可運行的最小可行產品（MVP），展示 ChatGPT 與企業文件之間的 RAG 流程，並能在本機與 Azure 上驗證部署與除錯流程。

**範圍（MVP）**

- 前端：可在本機啟動的 `webapp`，基本聊天 UI。
- 後端：`search` API（本機或容器）能回應查詢並回傳來源引註。
- 索引：使用 `indexer` 將 `data/` 內的範例文件建立索引。
- DevOps：能以 `azd up` 進行最小配置的 Azure 部署（低資源、單副本）。

**成功指標**

- 本機啟動：`npm start` 後可在 `http://127.0.0.1:5173` 開啟前端並詢問問題。
- 索引建立：`./scripts/index-data.sh` 成功執行並產生可檢索的索引。
- Azure 部署（選項）：以最小配置完成 `azd up`，並在 Azure 上訪問應用。

**高階里程碑與建議時序（建議 3-5 天）**

- Day 0（準備）：環境檢查、取得 Azure 帳戶授權（若要部署）。
- Day 1（本機驗證）：安裝相依、建置、啟動所有服務，建立索引並跑基本測試。
- Day 2（修正與穩定）：根據錯誤日誌修正程式；加入基本監控與日誌輸出。
- Day 3（Azure 最小部署）：以最低資源執行 `azd up`，驗證端到端行為。
- Day 4（驗收與清理）：執行驗收測試，若為短期驗證則 `azd down --purge` 清除資源。

**執行任務清單（可複製執行）**

1. 環境檢查（在本機執行）

```bash
node -v
npm -v
docker --version || echo "Docker not required for pure local run"
azd version || echo "Install Azure Developer CLI"
git --version
```

2. 取得 Azure 環境（若要在 Azure 驗證）

```bash
azd auth login   # 互動式登入
az login         # 若需要 az 登入
azd env get-values > .env
```

3. 安裝相依與 Playwright（本機驗證/測試）

```bash
npm install
npx playwright install --with-deps
```

4. 建置並啟動（本機）

```bash
npm run build
npm start
# 或分別啟動
npm run start:webapp
npm run start:search
npm run start:indexer
```

5. 建立/更新索引（確保 `.env` 有正確的 BACKEND*URI/AZURE_SEARCH*\*）

```bash
./scripts/index-data.sh
```

6. 測試與除錯

```bash
npm test
npx playwright test
# 檢視服務日誌（在終端或 container logs）
```

7. Azure 最小部署（選項）

```bash
# 以最低資源配置執行 (先檢查 infra 參數)
azd up
# 部署完畢後若暫時驗證完成則清除資源
azd down --purge
```

**風險重點與緩解（精簡）**

- 成本：OpenAI 與 Search 使用會產生費用。緩解：使用最小配置或先在本機模擬；測試後立即刪除資源。
- 資料敏感性：索引可能含敏感資訊。緩解：使用匿名化或合成資料進行測試，限制訪問（Entra/ALLOWED_ORIGIN）。
- 權限與憑證：需互動登入或受限 Service Principal。緩解：避免在 repo 保存憑證；使用最小權限原則。
- 相依/環境差異：本機與 Azure 行為不同。緩解：使用 `.nvmrc`、Dev Container 或 Codespaces 保持一致性。

**必要前置條件**

- 若要部署到 Azure：具備有 Azure OpenAI 訪問權的訂閱與 `azd` 已安裝並能登入。`
- 本機測試：Node.js 22+、npm、可選 Docker（僅當你想用容器化啟動）。

**驗收測試（MVP 完成標準）**

- 在本機：前端能回應查詢且顯示來源引註（至少 3 個測試 query）。
- 在 Azure（若部署）：從公開 URL 發起查詢，得到有效回應，並能在 Application Insights 或日誌中看到請求追蹤。

**負責人與溝通**

- 若需我執行指令：請在終端完成 `azd auth login`（若進行 Azure 部署）並回覆「開始」，我將逐步執行並回報每步輸出。
- 若你想自行執行：請依上方命令逐步執行，遇到錯誤貼上錯誤輸出，我會協助分析與修正。

**備註**

- 建議先以「本機驗證」流程完成所有功能，再進行 Azure 部署以降低成本與風險。祝順利，若你要我開始執行請回覆「開始」。
