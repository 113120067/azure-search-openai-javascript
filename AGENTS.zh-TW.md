# AGENTS.md（繁體中文翻譯）

## 專案概覽

這是一個使用 Azure OpenAI 與 Azure AI Search 建構的 ChatGPT + 企業資料 應用程式，實作了檢索增強生成（Retrieval Augmented Generation，RAG）模式。該應用允許使用者對企業資料進行聊天查詢，使用 Azure OpenAI 的 ChatGPT 模型（gpt-4o-mini）以及 Azure AI Search 做資料索引與檢索。

**架構**：此專案以 npm workspace 組織，包含三個主要套件：

- **webapp**：以 React + Vite 建立的前端應用，使用 Lit web components（部署為 Static Web App）
- **search**：基於 Fastify 的後端 API 服務，提供搜尋與聊天功能（部署為 Container App）
- **indexer**：文件索引服務，含 CLI 工具（部署為 Container App）

**關鍵技術**：

- TypeScript、React、Lit（web components）
- Fastify（後端 API 框架）
- Vite（前端建置工具）
- Azure OpenAI、Azure AI Search、Azure Container Apps、Azure Static Web Apps
- Playwright（端對端測試）、k6（負載測試）
- Azure Developer CLI（`azd`）用於基礎設施與部署

## 設定指令

### 前置需求

- Node.js 22 或以上（請檢查 `.nvmrc`）
- npm 10 或以上
- Azure Developer CLI（`azd`）— 用於部署
- 具備 OpenAI 服務存取的 Azure 訂閱

### 安裝

```bash
# 安裝所有相依套件（workspace 根目錄）
npm install

# 安裝 Playwright 瀏覽器（用於 e2e 測試）
npx playwright install --with-deps
```

### 第一次在 Azure 上部署

```bash
# 登入 Azure
azd auth login

# 部署基礎設施與應用程式
azd up

# 該命令將會：
# - 提示您選擇 Azure 區域
# - 佈建所有 Azure 資源（OpenAI、AI Search、Container Apps 等）
# - 建置並部署所有服務
# - 執行 post-provision hook 以建立範例資料索引
```

## 開發工作流程

### 在本機執行

**前提**：您必須先使用 `azd up` 將資源部署到 Azure，才能在本機執行。

```bash
# 1. 驗證 Azure 帳戶
azd auth login
az login

# 2. 從 Azure 佈建匯出環境變數到 .env
azd env get-values > .env

# 3. 建立資料索引
./scripts/index-data.sh    # Linux/macOS
./scripts/index-data.ps1   # Windows

# 4. 同時啟動所有服務
npm start

# 這會啟動三個服務：
# - webapp 在 http://localhost:5173
# - search API 在 http://localhost:3000
# - indexer API 在 http://localhost:3001
```

### 單獨啟動特定服務

```bash
# 僅啟動 webapp
npm run start:webapp

# 僅啟動 search API
npm run start:search

# 僅啟動 indexer
npm run start:indexer
```

### 開發模式（熱重載）

每個 workspace 套件均有自己的開發模式，支援熱重載：

```bash
# 在特定套件中建置並監看變更
npm run dev --workspace=webapp
npm run dev --workspace=search
npm run dev --workspace=indexer
```

### 環境變數

環境變數透過 Azure Developer CLI 管理：

```bash
# 顯示所有環境變數
azd env get-values

# 設定單一環境變數
azd env set VARIABLE_NAME value

# 匯出為 .env 檔供本機開發使用
azd env get-values > .env
```

## 測試說明

### 單元測試

```bash
# 執行 workspace 中的所有測試
npm test

# 執行特定套件的測試
npm test --workspace=search
npm test --workspace=indexer

# 測試使用 Node.js 內建測試執行器與 c8 做覆蓋率
```

### 端對端測試（Playwright）

```bash
# 執行 Playwright e2e 測試
npm run test:playwright

# 以有畫面模式執行以便除錯
npx playwright test --headed

# 執行特定測試檔
npx playwright test tests/e2e/webapp.spec.ts

# 檢視測試報告
npx playwright show-report
```

**測試設定**：請見 `playwright.config.ts`

- 測試對應目標為 `http://localhost:5173`
- 使用 chromium、firefox 與 webkit 瀏覽器
- 會自動啟動 webapp 的開發伺服器以供測試
- 測試檔位於 `tests/e2e/`

### 負載測試

```bash
# 使用 k6 執行負載測試
npm run test:load

# 測試腳本位於 tests/load/index.js
```

### 測試檔位置

- 單元測試：`packages/*/test/**/*.ts`
- E2E 測試：`tests/e2e/**/*.spec.ts`
- 負載測試：`tests/load/**/*.js`

## 程式碼風格

### Lint 與格式化

```bash
# 執行 ESLint
npm run lint

# 自動修正 ESLint 問題
npm run lint:fix

# 檢查 Prettier 格式
npm run format:check

# 使用 Prettier 格式化程式碼
npm run format
```

### 風格慣例

**Prettier 設定**（位於 `package.json`）：

- Tab 寬度：2 個空格
- 必要分號
- 單引號
- 換行寬度：120 字元
- 括號空白符號啟用

**ESLint**：使用 workspace 中的 `eslint-config-shared` 共同設定

**pre-commit Hooks**：

- 使用 `simple-git-hooks` 與 `lint-staged`
- 在 `git commit` 時自動執行
- 對 staged 檔案使用 Prettier 格式化

### 檔案組織

```
/
├── packages/           # Workspace 套件
│   ├── webapp/        # 前端應用
│   ├── search/        # 搜尋 API 服務
│   ├── indexer/       # 索引服務
│   ├── chat-component/# 共用的聊天元件
│   └── eslint-config/ # 共用 ESLint 設定
├── infra/             # Bicep 基礎設施代碼
├── scripts/           # 建置與部署腳本
├── tests/             # E2E 與負載測試
├── data/              # 範例文件（用於索引）
└── docs/              # 文件
```

### TypeScript

- 所有套件皆使用 TypeScript
- 以 `tsc`（TypeScript 編譯器）建置
- 各套件的 `tsconfig.json` 含有設定
- 使用 `@types/*` 提供型別定義

## 建置與部署

### 建置所有套件

```bash
# 建置所有 workspace 套件
npm run build

# 這會編譯 TypeScript 並建立 Vite 套件
```

### 建置特定套件

```bash
# 建置特定套件
npm run build --workspace=webapp
npm run build --workspace=search
npm run build --workspace=indexer
```

### 建置 Docker 映像檔

```bash
# 建置所有 Docker 映像（僅 Linux）
npm run docker:build

# 建置特定服務映像
npm run docker:build --workspace=search
npm run docker:build --workspace=indexer
```

### 使用 Azure Developer CLI 部署

```bash
# 部署基礎設施與所有服務
azd up

# 僅部署程式碼變更（略過基礎設施）
azd deploy

# 僅佈建基礎設施
azd provision

# 修改後重部署
azd deploy
```

### 部署架構

`azure.yaml` 檔定義三個服務：

- **webapp**：Static Web App（前端）
  - predeploy hook 會建置 webapp
  - 部署 `dist` 資料夾
- **search**：Container App（後端 API）
  - 在 Azure 上進行遠端 Docker 建置
- **indexer**：Container App（索引服務）
  - 在 Azure 上進行遠端 Docker 建置

**Post-deployment Hook**：佈建完成後，`postup` hook 執行 `index-data` 腳本以將範例資料加入搜尋索引。

### 環境專屬設定

```bash
# 若使用現有 Azure 資源，可設定下列變數：
azd env set AZURE_RESOURCE_GROUP <existing-group>
azd env set AZURE_OPENAI_RESOURCE_GROUP <existing-openai-group>
azd env set AZURE_OPENAI_RESOURCE <existing-openai-resource>
azd env set AZURE_SEARCH_SERVICE <existing-search-service>

# 然後執行 azd up 以使用該資源進行部署
azd up
```

## Pull Request 指引

### 提交前步驟

1. **執行 lint 與格式化**：

   ```bash
   npm run lint:fix
   npm run format
   ```

2. **建置程式碼**：

   ```bash
   npm run build
   ```

3. **執行測試**：

   ```bash
   npm test
   ```

4. **執行 E2E 測試（若有 UI 變更）**：
   ```bash
   npm run test:playwright
   ```

### PR 要求

- 標題需清楚且具描述性
- 所有 lint 檢查需通過
- 所有測試需通過
- 程式碼需經 Prettier 格式化
- 遵循 TypeScript 最佳實作
- 新功能或修正需包含測試

### CI/CD 工作流程

GitHub Actions workflow（`.github/workflows/build-test.yaml`）會在每次 PR 時執行：

- 使用 `npm ci` 安裝相依
- 建置所有套件
- 建置 Docker 映像（僅 Ubuntu）
- 執行 lint
- 執行單元測試
- 在 Ubuntu、macOS 與 Windows 上以 Node.js 22 執行測試

## 補充說明

### Workspace 指令

此專案使用 npm workspaces。常用範例：

```bash
# 在所有 workspace 執行指令
npm run <script> -ws

# 在含有該 script 的所有 workspace 執行
npm run <script> -ws --if-present

# 在特定 workspace 執行指令
npm run <script> --workspace=<package-name>
```

### 常見問題

1. **必須先部署到 Azure**：本專案的本機開發依賴 Azure 資源，必須先執行 `azd up`。

2. **環境變數**：`.env` 檔由 Azure 佈建產生，請使用 `azd env get-values > .env`，不要手動建立。

3. **資料索引**：佈署後請確認索引資料，`postup` hook 會自動建立，若變更資料則需手動執行索引腳本。

4. **OpenAI 容量**：預設為 30K TPM（每分鐘 tokens），約 30 次對話/分鐘。若需要更多，請調整 `infra/main.bicep` 中的 `chatGptDeploymentCapacity`。

5. **Playwright 測試**：需安裝瀏覽器，若缺少請執行 `npm run install:playwright`。

6. **Node.js 版本**：本專案需要 Node.js 22+，請檢查 `.nvmrc`。

### 故障排除

**"Resource name not allowed" 衝突**：Azure 會保留刪除的資源 48 小時，請釋放名稱或使用其他名稱。

```bash
# 清除並 purge 所有資源
azd down --purge
```

**埠號衝突**：本機執行需確保 3000、3001、5173 等埠未被佔用。

**驗證錯誤**：請確保已執行 `azd auth login` 與 `az login`。

### 重新建立索引

新增或更新文件時：

```bash
# 1. 將檔案加入 data/ 資料夾
# 2. 執行索引腳本
./scripts/index-data.sh    # Linux/macOS
./scripts/index-data.ps1   # Windows
```
