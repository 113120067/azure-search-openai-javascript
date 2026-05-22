# 使用 Azure OpenAI 與 Azure AI Search 建構 ChatGPT + 企業資料 的範例

## 目錄

- [功能](#features)
- [快速開始](#getting-started)
- [Azure 帳戶需求](#azure-account-prerequisites)
- [在 Azure 上部署](#azure-deployment)
  - [成本估算](#cost-estimation)
  - [專案設定](#project-setup)
    - [GitHub Codespaces](#github-codespaces)
    - [VS Code Remote Containers](#vs-code-remote-containers)
    - [本機環境](#local-environment)
  - [從頭部署](#deploying-from-scratch)
  - [使用現有資源部署](#deploying-with-existing-resources)
  - [再次部署](#deploying-again)
- [分享環境](#sharing-environments)
- [清除資源](#clean-up)
- [啟用選用功能](#enabling-optional-features)
  - [啟用驗證](#enabling-authentication)
- [使用應用程式](#using-the-app)
- [指引](#guidance)
  - [在本機執行](#running-locally)
  - [使用不同的後端](#using-a-different-backend)
  - [投入生產環境時的注意事項](#productionizing)
  - [參考資源](#resources)
  - [常見問題](#faq)
  - [故障排除](#troubleshooting)
  - [降低部署成本](#reduce-deployment-costs)
- [備註](#note)

[![在 GitHub Codespaces 中開啟](https://img.shields.io/static/v1?style=for-the-badge&label=GitHub+Codespaces&message=Open&color=brightgreen&logo=github)](https://github.com/codespaces/new?hide_repo_select=true&ref=main&repo=684521881&machine=standardLinux32gb&devcontainer_path=.devcontainer%2Fdevcontainer.json&location=WestUs2)
[![加入 Azure AI Foundry Discord](https://img.shields.io/badge/Discord-Azure_AI_Community-blue?style=for-the-badge&logo=discord&color=5865f2&logoColor=fff)](https://aka.ms/foundry/discord)
[![在 Remote - Containers 中開啟](https://img.shields.io/static/v1?style=for-the-badge&label=Remote%20-%20Containers&message=Open&color=blue&logo=visualstudiocode)](https://vscode.dev/redirect?url=vscode://ms-vscode-remote.remote-containers/cloneInVolume?url=https://github.com/azure-samples/azure-search-openai-javascript)

此範例示範如何使用檢索增強生成（Retrieval Augmented Generation, RAG）模式，針對您自己的資料建立類 ChatGPT 的體驗。專案使用 Azure OpenAI Service 存取 ChatGPT 模型（gpt-4o-mini）以及 Azure AI Search 做資料索引與檢索。

![檢索增強生成架構](docs/rag-architecture.png)

此 repository 含有範例資料，可直接從頭試用。在本範例應用中，我們以虛構公司 Contoso Real Estate 為例，使用者可以詢問產品使用上的支援問題。範例資料包含服務條款、隱私政策與支援指南等文件。

應用由多個元件組成，包括：

- **Search service（搜尋服務）**：提供搜尋與檢索功能的後端服務。
- **Indexer service（索引服務）**：負責索引資料並建立搜尋索引的服務。
- **Web app（網站）**：提供使用者介面，並協調前端與後端之間互動的前端應用程式。

![應用程式架構](docs/app-architecture.drawio.png)

## 功能

- 聊天與問答介面
- 提供多種方式幫助使用者評估回應可信度（來源引註、來源內容追蹤等）
- 示範資料前處理、提示工程與模型（ChatGPT）與檢索器（Azure AI Search）之間互動的多種編排方式
- 在使用者介面中直接提供設定以調整行為並試驗選項
- 可選的效能追蹤與監控（Application Insights）

![聊天畫面](docs/chat-screenshot.png)

[📺 觀看應用程式概覽影片](https://youtu.be/uckVTuS36H0)

## 快速開始

## Azure 帳戶需求

**重要：** 若要部署並執行此範例，您需要：

- **Azure 帳戶**。若您是 Azure 新手，請[申請免費 Azure 帳戶](https://azure.microsoft.com/free)以取得入門的免費額度。
- **具備 Azure OpenAI 服務使用權限的 Azure 訂閱**。您可以透過[此表單](https://aka.ms/oaiapply)申請使用權。
- **Azure 帳戶權限**：
  - 您的 Azure 帳戶必須擁有 `Microsoft.Authorization/roleAssignments/write` 權限（例如 [Role Based Access Control Administrator](https://learn.microsoft.com/azure/role-based-access-control/built-in-roles#role-based-access-control-administrator-preview)、[User Access Administrator](https://learn.microsoft.com/azure/role-based-access-control/built-in-roles#user-access-administrator) 或 [Owner](https://learn.microsoft.com/azure/role-based-access-control/built-in-roles#owner)）。如果您沒有訂閱層級的權限，可以改為在現有資源群組中為您授予必要的 RBAC 權限，並針對該資源群組部署（請參考「使用現有資源部署」章節）。
  - 您的 Azure 帳戶還需要在訂閱層級擁有 `Microsoft.Resources/deployments/write` 權限。

## 在 Azure 上部署

### 成本估算

價格會依區域與使用量而異，無法精確估算。您可以使用 [Azure 定價計算機](https://azure.com/e/8468fee268374b6fbd32db323deec786) 計算下列資源的預估成本。

- Azure Container Apps：依 vCPU 與記憶體用量計費。請參閱 [定價](https://azure.microsoft.com/pricing/details/container-apps/)
- Azure Static Web Apps：免費等級。請參閱 [定價](https://azure.microsoft.com/pricing/details/app-service/static/)
- Azure OpenAI：標準等級（ChatGPT 與 Ada 模型），依每 1K tokens 計費，且每個問題至少會使用 1K tokens。請參閱 [定價](https://azure.microsoft.com/pricing/details/search/)
- Azure AI Search：標準等級，1 個副本，具有免費語意搜尋選項。依小時計費。請參閱 [定價](https://azure.microsoft.com/pricing/details/search/)（註：網頁資訊可能變動，請以連結頁面為準）
- Azure Blob Storage：標準等級並使用 ZRS（跨區域冗餘儲存）。依儲存與讀取操作計費。請參閱 [定價](https://azure.microsoft.com/pricing/details/storage/blobs/)
- Azure Monitor：依資料攝取量計費。請參閱 [定價](https://azure.microsoft.com/pricing/details/monitor/)

⚠️ 若要避免不必要的費用，使用完畢後請務必移除應用程式，或在入口網站刪除資源群組，或執行 `azd down --purge`。

### 專案設定

有多種方式可以成功設定本專案。

最簡單的方式是使用 GitHub Codespaces，Codespaces 已預先配置好所需工具，能協助您快速啟動。您也可以依下面的說明在本機環境設定。

#### GitHub Codespaces

您可以使用 GitHub Codespaces 在瀏覽器中啟動虛擬的 VS Code 環境：

[![在 GitHub Codespaces 中開啟](https://img.shields.io/static/v1?style=for-the-badge&label=GitHub+Codespaces&message=Open&color=brightgreen&logo=github)](https://github.com/codespaces/new?hide_repo_select=true&ref=main&repo=684521881&machine=standardLinux32gb&devcontainer_path=.devcontainer%2Fdevcontainer.json&location=WestUs2)

#### VS Code Remote Containers

另一個選項是使用 VS Code Remote Containers，透過 [Dev Containers 延伸功能](https://marketplace.visualstudio.com/items?itemName=ms-vscode-remote.remote-containers) 在本機 VS Code 中開啟專案：

[![在 Remote - Containers 中開啟](https://img.shields.io/static/v1?style=for-the-badge&label=Remote%20-%20Containers&message=Open&color=blue&logo=visualstudiocode)](https://vscode.dev/redirect?url=vscode://ms-vscode-remote.remote-containers/cloneInVolume?url=https://github.com/azure-samples/azure-search-openai-javascript)

#### 本機環境

- [Azure Developer CLI](https://aka.ms/azure-dev/install)
- [Node.js LTS](https://nodejs.org/en/download/)
- [Docker for Desktop](https://www.docker.com/products/docker-desktop/)
- [Git](https://git-scm.com/downloads)
- [PowerShell 7+ (pwsh)](https://github.com/powershell/powershell) — 僅限 Windows 使用者。
  - **重要**：請確認您能從 PowerShell 執行 `pwsh.exe`，若無法執行，可能需升級 PowerShell。

接著取得專案程式碼：

1. 在終端機中建立新資料夾並切換至該資料夾
1. 執行 `azd auth login`
1. 執行 `azd init -t azure-search-openai-javascript`
   - 注意：此命令會初始化一個 git repository，因此您不需要再另外 clone 本範例。

### 從頭部署

若您尚未擁有任何 Azure 服務，可執行下列步驟從全新部署開始：

1. 執行 `azd up` — 這會在 Azure 上佈建資源並部署此範例，包含根據 `./data` 資料夾中的檔案建立搜尋索引。
   - 執行時會提示您選擇大多數資源的地點（OpenAI 與 Static Web App 例外）。
   - 預設情況下，OpenAI 資源會部署在 `eastus2`。若要更改位置，可使用 `azd env set AZURE_OPENAI_RESOURCE_GROUP_LOCATION {location}`。可選的區域有限，並以 OpenAI 模型可用性表為準。
   - 預設情況下，Static Web App 資源會部署在 `eastus2`。可使用 `azd env set AZURE_WEBAPP_LOCATION {location}` 更改。注意 Static Web App 為全球性服務，此位置設定僅影響所管理的 Functions App（本範例未使用）。
1. 部署完成後，終端機會列出一個 URL，點選該 URL 即可在瀏覽器中存取應用程式。

畫面示意如下：

!['執行 azd up 的輸出畫面'](docs/deployment.png)

> 注意：完整部署可能需 15 分鐘或以上。

### 使用現有資源部署

如果您已有現成的 Azure 資源，也可以重用這些資源，方法是設定 `azd` 的環境變數。

#### 使用現有資源群組

1. 執行 `azd env set AZURE_RESOURCE_GROUP {Name of existing resource group}`
1. 執行 `azd env set AZURE_LOCATION {Location of existing resource group}`

#### 使用現有 OpenAI 資源

1. 執行 `azd env set AZURE_OPENAI_SERVICE {Name of existing OpenAI service}`
1. 執行 `azd env set AZURE_OPENAI_RESOURCE_GROUP {Name of existing resource group that OpenAI service is provisioned to}`
1. 執行 `azd env set AZURE_OPENAI_CHATGPT_DEPLOYMENT {Name of existing ChatGPT deployment}`（若您的 ChatGPT 部署非預設的 'chat' 才需要設定）
1. 執行 `azd env set AZURE_OPENAI_EMBEDDING_DEPLOYMENT {Name of existing GPT embedding deployment}`（若 embeddings 部署非預設 'embedding' 才需要設定）

#### 使用現有 Azure AI Search 資源

1. 執行 `azd env set AZURE_SEARCH_SERVICE {Name of existing Azure AI Search service}`
1. 執行 `azd env set AZURE_SEARCH_SERVICE_RESOURCE_GROUP {Name of existing resource group with ACS service}`
1. 若該資源群組位於與您打算在 `azd up` 選取的地點不同，請再執行 `azd env set AZURE_SEARCH_SERVICE_LOCATION {Location of existing service}`
1. 若搜尋服務的 SKU 非標準（standard），請執行 `azd env set AZURE_SEARCH_SERVICE_SKU {Name of SKU}`。免費層（free）不可用，因為不支援 Managed Identity。（參考 SKU 列表連結）

#### 其他現有 Azure 資源

您也可以重用現有的儲存帳戶。請參閱 `./infra/main.parameters.json`，該檔列出可以透過 `azd env set` 設定的變數名稱。

#### 佈建剩餘資源

設定好環境變數後，執行 `azd up`（請參考「從頭部署」章節）以同時佈建與部署程式碼。

### 再次部署

若您只變更了後端或前端的程式碼，無需再次佈建基礎設施，可直接執行：

`azd deploy`

但若變更了基礎設施檔案（`infra` 資料夾或 `azure.yaml`），則需重新佈建：

`azd up`

## 分享環境

若要讓其他人存取一個已部署並可運作的環境，請執行：

1. 安裝 [Azure Developer CLI](https://aka.ms/azure-dev/install)
1. 執行 `azd init -t azure-search-openai-javascript` 或直接 clone 本 repository。
1. 執行 `azd env refresh -e {environment name}`，提供的資訊會寫入 `.azure/{env name}/.env` 檔案。對方需知道 azd 環境名稱、訂閱 ID 與地點。
1. 在該 `.env` 檔或 shell 中設定環境變數 `AZURE_PRINCIPAL_ID` 為對方的 Azure ID（可用 `az ad signed-in-user show` 取得）。
1. 執行 `./scripts/roles.ps1` 或 `./scripts/roles.sh` 來為使用者分配必要角色。如果他們無法在訂閱中建立角色，您可能需要代為執行此腳本。腳本完成後，對方即可在本機執行應用程式。

## 清除資源

要移除此範例所建立的所有資源：

1. 執行 `azd down --purge`
2. 當提示是否繼續時，輸入 `y`
3. 當提示是否永久刪除資源時，輸入 `y`

系統會刪除資源群組與其下所有資源。

## 啟用選用功能

### 啟用驗證

預設情況下，部署後的 Azure 網站不會啟用驗證或存取限制，換言之，只要能連網就能存取並查詢已索引的資料。若要限制存取至 Azure Entra ID 使用者，可參考 [加入應用程式驗證](https://learn.microsoft.com/training/modules/publish-static-web-app-authentication/) 教學，將驗證設定套用於已部署的靜態網站。

接著若要限制特定使用者或群組的存取，可依 [限制 Azure Entra 應用存取至指定使用者/群組](https://learn.microsoft.com/entra/identity-platform/howto-restrict-your-app-to-a-set-of-users) 的步驟，在企業應用程式中將 "Assignment Required?" 選項打開，並指派使用者/群組存取權限。未被指派的人會看到錯誤訊息 AADSTS50105，顯示管理員已設定僅允許被指派的使用者存取。

### 其他安全性考量

我們建議額外啟用安全機制，例如在適用時設定 [VNet](https://learn.microsoft.com/azure/virtual-network/virtual-networks-overview) 或設定 [Proxy Policy](https://learn.microsoft.com/azure/api-management/proxy-policy)。

### 為替代前端啟用 CORS

預設情況下，部署後的搜尋 API 僅允許來自相同來源（origin）的請求。如果您想允許不同來源的前端呼叫 API，請執行：

1. 執行 `azd env set ALLOWED_ORIGIN https://<your-domain.com>`
2. 執行 `azd up`

## 在本機執行

必須先成功執行過 `azd up` 才能在本機執行。

1. 執行 `azd auth login`
2. 執行 `azd env get-values > .env` 以取得應用程式所需的環境變數。
3. 執行 `./scripts/index-data.sh` 或 `./scripts/index-data.ps1` 來建立索引資料。
4. 執行 `az login` 以登入 Azure（Managed Identity 本機運作需要）。
5. 執行 `npm start`，或使用 VS Code 的工作項目 `Start App` 來啟動專案。

## 使用應用程式

- 在 Azure 上：前往由 `azd` 部署的 Azure Static Web App，`azd` 佈署完成時終端機會列出一個「Endpoint」URL，可點選該連結，或在 Azure 入口網站中找到該網站。
- 本機執行：前往 `http://127.0.0.1:5173`

使用步驟：

- 在聊天或問答情境中嘗試各種主題；在聊天模式中試著追問、釐清、要求簡化或延伸回答等。
- 檢視回應的引註與來源。
- 點選「設定」以嘗試不同選項、調整提示等。

## 指引

### 使用不同的後端

Search API 服務實作了 [AI chat apps 的 HTTP protocol](https://aka.ms/chatprotocol)。您可以以任何實作相同協議的服務替換此後端，例如使用本 repository 中的 Python 後端範例（https://github.com/Azure-Samples/azure-search-openai-demo）來取代 Node.js 的實作。

步驟如下：

1. 先部署本 repository（依前述步驟）。
2. 取得前端 URL：

- 若使用已部署的靜態網站，執行 `azd env get-values | grep WEBAPP_URI` 以取得 URL。
- 若在本機使用，前端 URL 為 `http://localhost:5173`。
- 若在 Codespaces 使用本機模式，URL 形如 `https://<your_codespace_base_url>-5173.app.github.dev`。

3. 在要使用的替代後端 repository 中（例如 https://github.com/Azure-Samples/azure-search-openai-demo）進行部署。
4. 將前端 URL 設為允許的來源：`azd env set ALLOWED_ORIGIN <your_frontend_url>`。
5. 依替代後端的部署說明進行部署（例如 Python 後端的部署步驟）。
6. 取得後端 URL（`azd env get-values | grep BACKEND_URI`），並在本 repository 設定 `azd env set BACKEND_URI <your_backend_url>`。
7. 視情況執行：

- 若使用已部署的前端網站，執行 `azd up` 重新部署。
- 若在本機或 Codespaces 使用本機前端，則可輸出環境變數並啟動：

  ```sh
  export BACKEND_URI=<your_backend_url>
  npm start --workspace=webapp
  ```

### 啟用驗證（後端）

此範例由兩個應用程式組成：後端 API 部署於 [Azure Container Apps](https://learn.microsoft.com/azure/container-apps/overview)，前端部署於 [Azure Static Web Apps](https://azure.microsoft.com/products/app-service/static/)。預設情況下，部署的 Container App 不會啟用存取限制，任何有網路路徑的用戶都能呼叫 API。若要啟用 Entra ID 驗證，可參考 [Add container app authentication](https://learn.microsoft.com/azure/container-apps/authentication-azure-active-directory) 教學並套用於 Container App。

若要限制特定使用者或群組存取，也可參考 [Restrict your Azure Entra app to a set of users](https://learn.microsoft.com/entra/identity-platform/howto-restrict-your-app-to-a-set-of-users) 的步驟，設定「Assignment Required?」並指派使用者或群組。

### 投入生產環境的注意事項

此範例為您建立生產應用的起點，但在上線前請務必針對安全性與效能做完整審查。考量包括：

- **OpenAI 容量**：預設 TPM（每分鐘 tokens）為 30K，約等於每分鐘 30 次對話（假設每次訊息/回應約 1K tokens）。您可以在 `infra/main.bicep` 中調整 `chatGptDeploymentCapacity` 與 `embeddingDeploymentCapacity`，以符合帳戶允許的上限。也可在 [Azure OpenAI studio 的 Quotas 分頁](https://oai.azure.com/) 查看剩餘容量。
- **Azure 儲存**：預設儲存帳戶使用 `Standard_LRS`。若要提升韌性，建議在生產環境使用 `Standard_ZRS`，可透過 `infra/main.bicep` 中的 `storage` 模組設置 `sku` 屬性。
- **Azure AI Search**：預設使用 `Standard` SKU 並啟用免費語意搜尋選項（每月 1000 次免費查詢）。若預期詢問量超過 1000 次，請考慮改為標準語意搜尋或在請求中停用語意搜尋。若遇到搜尋服務容量不足的錯誤，可增副本數（`infra/core/search/search-services.bicep` 的 `replicaCount`），或從 Azure 入口網站手動擴充。
- **Azure Container Apps**：預設每個容器使用 1 vCPU 與 2 GB RAM，並啟用自動調整，最小 1 個副本、最大 10 個副本。您可在範本中調整 vCPU 與 RAM 設定（檔案位於 infra/main.bicep），並根據負載定義自動調整規則。更多資訊請參考 [設定 Container Apps 的縮放規則](https://learn.microsoft.com/azure/container-apps/scale-app?pivots=azure-resource-manager)。
- **驗證**：預設部署為公開存取，建議限制僅允許已驗證的使用者存取（參見「啟用驗證」章節）。
- **網路**：建議將服務部署於 Virtual Network（VNet）內。若應用僅供企業內部使用，請搭配私有 DNS 區域。也可考慮使用 Azure API Management（APIM）強化防火牆與保護機制。

欲了解更多，請參考 [Azure OpenAI Landing Zone 參考架構](https://techcommunity.microsoft.com/t5/azure-architecture-blog/azure-openai-landing-zone-reference-architecture/ba-p/3882102)。

## 參考資源

- [Generative AI For Beginners](https://github.com/microsoft/generative-ai-for-beginners)
- [Revolutionize your Enterprise Data with ChatGPT: Next-gen Apps w/ Azure OpenAI and AI Search](https://aka.ms/entgptsearchblog)
- [Azure AI Search](https://learn.microsoft.com/azure/search/search-what-is-azure-search)
- [Azure OpenAI Service](https://learn.microsoft.com/azure/ai-services/openai/overview)
- [Building ChatGPT-Like Experiences with Azure: A Guide to Retrieval Augmented Generation for JavaScript applications](https://devblogs.microsoft.com/azure-sdk/building-chatgpt-like-experiences-with-azure-a-guide-to-retrieval-augmented-generation-for-javascript-applications/)

### 常見問題（FAQ）

<details><a id="ingestion-why-chunk"></a>
<summary>為何需要把文件切成多個區塊（chunk），即便 Azure AI Search 支援搜尋大型文件？</summary>

切分可以限制傳送給 OpenAI 的資訊量以符合 token 限制。透過切分，我們能更精準地找出可插入到 OpenAI 提示中的資料片段。此範例採用滑動視窗（sliding window）方式切分，讓結尾句子會同時出現在相鄰的區塊中，降低失去上下文的風險。

</details>

<details><a id="ingestion-more-pdfs"></a>
<summary>如何在不重新部署整個系統的情況下上傳更多文件？</summary>

將檔案放到 `data/` 資料夾中，然後執行 `./scripts/index-data.sh` 或 `./scripts/index-data.ps1`。

</details>

<details><a id="compare-samples"></a>
<summary>此範例與其他「Chat with Your Data」範例有何差異？</summary>

另一個常見的範例 repository 是：https://github.com/Microsoft/sample-app-aoai-chatGPT/

該 repo 以 Azure OpenAI Studio 與 Azure Portal 為基礎進行設定，也提供 `azd` 支援供需要從頭部署的使用者使用。

主要差異：

- 本 repository 提供多種 RAG（檢索增強生成）做法，會將多次 API 呼叫的結果串接（例如同時呼叫 Azure OpenAI 與 ACS），而其他範例主要依賴 ChatCompletions API 的 data_sources 功能。
- 本 repository 比較實驗性，並非緊密綁定 Azure OpenAI Studio。

功能比較表：

| 功能                 | azure-search-openai-javascript | sample-app-aoai-chatGPT                  |
| -------------------- | ------------------------------ | ---------------------------------------- |
| RAG 做法             | 多種做法                       | 透過 ChatCompletion API 的 data_sources |
| 向量支援             | ✅ 支援                        | ✅ 支援                                  |
| 資料匯入             | ✅ 支援（MD）                  | ✅ 支援（PDF, TXT, MD, HTML）             |
| 持久化對話歷史       | ❌ 否（僅瀏覽器分頁）           | ✅ 是（CosmosDB）                         |

技術比較：

| 技術       | azure-search-openai-javascript | sample-app-aoai-chatGPT |
| ---------- | ------------------------------ | ----------------------- |
| 前端       | React/Lit                      | React                   |
| 後端       | Node.js (Fastify)              | Python (Flask)          |
| 向量資料庫 | Azure AI Search                | Azure AI Search         |
| 部署       | Azure Developer CLI (azd)      | Azure Portal, az, azd   |

</details>

<details><a id="switch-gpt4"></a>
<summary>如何在此範例中使用 GPT-4？</summary>

執行下列命令：

```bash
azd env set AZURE_OPENAI_CHATGPT_MODEL gpt-4
```

您可能也需根據帳戶允許的 TPM 調整 `infra/main.bicep` 中的容量參數。

</details>
<details><a id="chat-ask-diff"></a>
<summary>Chat 標籤與 Ask 標籤有何不同？</summary>

Chat 標籤採用 `packages/search/src/lib/approaches/chat-read-retrieve-read.ts` 中實作的方法；
Ask 標籤則採用 `packages/search/src/lib/approaches/ask-retrieve-then-read.ts` 中的實作方式。

</details>

<details><a id="azd-up-explanation"></a>
<summary>`azd up` 命令會做什麼？</summary>

`azd up` 為 Azure Developer CLI 的命令，它會負責佈建 Azure 資源並將程式碼部署到所選的主機上。

`azd up` 會讀取本專案的 `azure.yaml` 並結合 `infra/` 下的 Bicep 檔案進行佈建。本專案的 `azure.yaml` 定義了 prepackage 與 postprovision 的 hook。 `azd up` 會先執行 prepackage 步驟以安裝 Node 相依並建立 React 前端檔案，接著打包所有程式碼為 zip 檔以便後續上傳。

然後它會依據 `main.bicep` 與 `main.parameters.json` 佈建資源。由於 OpenAI 資源位置沒有預設值，系統會要求您從可選區域中選擇一個位置。佈建完成後，`azd up` 會執行 postprovision hook 來處理本機資料並將其加入 Azure AI Search 索引。

最後，`azd up` 會將程式碼上傳至 Azure 主機（如 Container Apps 與 Static Web Apps）。`azd up` 完成後應可使用，但首次部署後可能還要等幾分鐘才能完全可用。

相關命令：`azd provision`（僅佈建基礎設施）與 `azd deploy`（僅部署程式碼更新）。

</details>

### 故障排除

常見錯誤情境與解法：

1. 訂閱（`AZURE_SUBSCRIPTION_ID`）未開啟 Azure OpenAI 使用權。請確認 `AZURE_SUBSCRIPTION_ID` 與 [OpenAI 使用權申請流程](https://aka.ms/oai/access) 中的 ID 相符。

1. 您嘗試在不支援 Azure OpenAI 的區域建立資源（例如 East US 2 而非 East US），或該地區尚未啟用您要使用的模型。請參考模型可用性表。

1. 您超過配額上限（例如某區域資源數量）。請參考配額與限制說明（連結）。

1. 若出現 "same resource name not allowed" 衝突，通常是因為您重複執行範例並刪除資源，但未將資源從軟刪除中清除。Azure 會保留刪除的資源達 48 小時，若要移除殘留名稱，請參考資源清除文件。

1. 執行 `azd up` 並開啟網站時看到 404，請等待 10 分鐘再試，或執行 `azd deploy` 並再次等待。如果仍有問題，請參閱部署除錯提示或開 issue 尋求協助。

1. 在本機執行或部署時遇到 `401 Principal does not have access to API/Operation` 錯誤，通常是因為環境變數包含 `AZURE_TENANT_ID`、`AZURE_CLIENT_ID`、`AZURE_CLIENT_SECRET`。您應該為相關 Service Principal 授權，或從環境中移除這些變數以確保可正常操作。更多資訊請參考 [Azure identity SDK 的說明](https://github.com/Azure/azure-sdk-for-js/blob/main/sdk/identity/identity/README.md#defaultazurecredential)。

### 降低部署成本

請參考 ./docs/low-cost.md 以取得降低成本的建議。

### 尋求協助

若遇到問題或有開發上的疑問，歡迎加入：

[![Azure AI Foundry Discord](https://img.shields.io/badge/Discord-Azure_AI_Foundry_Community_Discord-blue?style=for-the-badge&logo=discord&color=5865f2&logoColor=fff)](https://aka.ms/foundry/discord)

若要回報產品意見或建置時遇到錯誤：

[![Azure AI Foundry Developer Forum](https://img.shields.io/badge/GitHub-Azure_AI_Foundry_Developer_Forum-blue?style=for-the-badge&logo=github&color=000000&logoColor=fff)](https://aka.ms/foundry/forum)

### 備註

> 注意：此示範中使用的文件內容包含透過語言模型（Azure OpenAI Service）產生的資訊。文件僅供演示用途，並不代表 Microsoft 的觀點或立場。Microsoft 對於文件中資訊的完整性、準確性、可靠性、適用性或可用性不作任何明示或默示之保證。版權所有 © Microsoft。
