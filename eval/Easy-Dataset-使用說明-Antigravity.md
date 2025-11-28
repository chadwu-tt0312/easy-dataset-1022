# Easy-Dataset-使用說明

## 1. 簡介

Easy Dataset 是一個專為大型語言模型（LLM）微調設計的資料集製作工具。它能將 PDF, Markdown, DOCX 等非結構化文檔，透過智慧分詞與 LLM 輔助，轉化為高質量的 Q&A 訓練數據。

- **專案網頁**：<https://github.com/ConardLi/easy-dataset>
- **參考網頁**：<https://deepwiki.com/ConardLi/easy-dataset>
- **目標讀者**：負責部署與維護 Easy Dataset 的 DevOps 工程師與系統管理員。

## 2. 環境變數詳解 (Environment Variables)

本專案主要依賴 Web UI 進行配置，系統層級的環境變數較少。

| 變數名稱 | 預設值 | 必填 | 說明 |
| :--- | :--- | :--- | :--- |
| `NODE_ENV` | `production` | 否 | Node.js 執行環境，開發時設為 `development`，部署時設為 `production`。 |
| `PORT` | `3000` (或 `1717`) | 否 | 應用程式監聽的 Port (Next.js 預設 3000，但在 Docker 中常配置為 1717)。 |

> **特別注意 (LLM Integration)**：
> 針對 **Azure OpenAI** 或其他 LLM 的整合（如 `AZURE_OPENAI_API_KEY`, `AZURE_OPENAI_ENDPOINT` 等），本專案**不使用環境變數**進行配置。
> 請在服務啟動後，進入 Web UI 的「專案設定」或「建立專案」流程中，直接於介面上輸入相關憑證與參數，這些資訊將被加密存儲於本地 SQLite 資料庫中。

## 3. 安裝與部署 (Installation & Deployment)

### 3.1 Docker Compose 部署 (推薦)

專案根目錄已包含 `docker-compose.yml`，這是最快速的部署方式。

1.  **下載專案**：
    ```bash
    git clone https://github.com/ConardLi/easy-dataset.git
    cd easy-dataset
    ```

2.  **啟動服務**：
    ```bash
    docker-compose up -d
    ```

3.  **驗證**：
    瀏覽器訪問 `http://localhost:1717`。

### 3.2 Kubernetes Helm 部署

由於本專案具備容器化特徵 (`Dockerfile` 與 `docker-compose.yml`)，以下提供適用於 K8s 的 Helm 部署指引。

#### 3.2.1 部署架構說明
- **Service Type**: 使用 `NodePort` 以便於測試與存取。
- **Storage**: 需掛載 `/app/local-db` 以持久化 SQLite 資料庫，建議使用 NFS Persistent Volume。

#### 3.2.2 資源清單範例 (YAML)

**PersistentVolumeClaim (PVC)**
```yaml
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: easy-dataset-pvc
spec:
  accessModes:
    - ReadWriteMany
  resources:
    requests:
      storage: 5Gi
  storageClassName: nfs-client # 請根據實際環境調整
```

**Deployment**
```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: easy-dataset
spec:
  replicas: 1
  selector:
    matchLabels:
      app: easy-dataset
  template:
    metadata:
      labels:
        app: easy-dataset
    spec:
      containers:
      - name: easy-dataset
        image: ghcr.io/conardli/easy-dataset:latest
        ports:
        - containerPort: 1717
        volumeMounts:
        - mountPath: "/app/local-db"
          name: data-volume
      volumes:
      - name: data-volume
        persistentVolumeClaim:
          claimName: easy-dataset-pvc
```

**Service**
```yaml
apiVersion: v1
kind: Service
metadata:
  name: easy-dataset-service
spec:
  type: NodePort
  selector:
    app: easy-dataset
  ports:
    - protocol: TCP
      port: 1717
      targetPort: 1717
      nodePort: 31717 # 可自行指定範圍內的 Port
```

### 3.3 NPM 本機安裝

若需進行二次開發或除錯，可使用 NPM 安裝。

1.  **安裝依賴**：
    ```bash
    npm install
    ```
2.  **建置與啟動**：
    ```bash
    npm run build
    npm run start
    ```

## 4. 操作指南 (Operations)

### 4.1 基本操作
- **啟動服務**：參見上述部署章節。
- **存取介面**：預設為 `http://<Server-IP>:1717`。

### 4.2 進階設定 (LLM Integration - Azure OpenAI)
若要整合 Azure OpenAI，請在 Web UI 進行以下設定：

1.  進入專案設定頁面。
2.  選擇模型供應商為 **Azure OpenAI**。
3.  填寫以下欄位：
    - **API Key**: 您的 Azure OpenAI 金鑰。
    - **Endpoint**: 資源端點 (例如 `https://my-resource.openai.azure.com/`)。
    - **API Version**: 例如 `2023-05-15`。
    - **Deployment Name**: 您在 Azure Portal 部署的模型名稱 (例如 `gpt-4-32k`)。

> [圖片說明：此處應顯示 Web UI 中 Azure OpenAI 的設定畫面截圖]

### 4.3 故障排除 (Troubleshooting)

- **Azure 連線錯誤 (401/404)**：
  - 檢查 **Deployment Name** 是否與 Azure Portal 完全一致（注意大小寫）。
  - 確認 **API Version** 是否支援您所選用的模型功能（如 Chat Completion）。
  
- **資料庫掛載失敗 (Docker/K8s)**：
  - 若掛載主機目錄至 `/app/local-db`，請確保主機目錄具有讀寫權限 (chmod 777 或 chown)。
  - 若手動掛載 `/app/prisma`，需先執行 `npm run db:push` 初始化 schema。

- **NFS 掛載問題**：
  - 若 Pod 處於 ContainerCreating 狀態過久，請檢查 NFS Server 連線及 PVC 綁定狀態。

## 5. 範例與截圖 (Examples)

### 範例：Docker Compose 啟動日誌
```text
easy-dataset  | ready - started server on 0.0.0.0:1717, url: http://localhost:1717
easy-dataset  | info  - Loaded env from /app/.env
```

> [圖片說明：此處應顯示 Easy Dataset 首頁 Dashboard 截圖]
