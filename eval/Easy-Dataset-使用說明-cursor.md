# Easy-Dataset 使用說明

**文件版本**：v1.5.1  
**最後更新**：2025年11月
**專案網頁**：<https://github.com/ConardLi/easy-dataset>  
**參考網頁**：<https://deepwiki.com/ConardLi/easy-dataset>  
**目標讀者**：負責部署與維護的技術人員

---

## 1. 簡介

### 專案功能與架構

Easy Dataset 是一個專為建立大型語言模型（LLM）微調資料集而設計的應用程式。它提供完整的 workflow，從文件處理到資料集匯出，支援多種文件格式和 AI 模型。

**核心功能**：
- 智能文件處理（PDF、Markdown、DOCX、EPUB、TXT）
- 智能文本分割與視覺化調整
- 基於 LLM 的問題生成
- 基於 LLM 的答案生成（支援 Chain of Thought）
- 領域標籤樹狀結構管理
- 多格式資料集匯出（Alpaca、ShareGPT、multilingual-thinking）

**技術架構**：
- **前端**：Next.js 14 (App Router), React 18, Material-UI v5
- **後端**：Node.js, Next.js API Routes
- **資料庫**：SQLite (透過 Prisma ORM)
- **桌面應用**：Electron
- **容器化**：Docker, Docker Compose
- **AI 整合**：支援 OpenAI、Ollama、智譜AI、OpenRouter 等多種 LLM 提供商

**系統架構圖**：
```
┌─────────────────┐
│  Electron App   │ (桌面應用模式)
└────────┬────────┘
         │
┌────────▼────────┐
│  Next.js App    │ (Web UI + API)
└────────┬────────┘
         │
┌────────▼────────┐      ┌──────────────┐
│  SQLite DB      │      │  LLM APIs    │
│  (Prisma)       │      │  (OpenAI等)  │
└─────────────────┘      └──────────────┘
```

---

## 2. 環境變數詳解 (Environment Variables)

### 核心環境變數

Easy Dataset 主要透過專案設定（儲存在資料庫中）與環境變數進行配置。以下為主要環境變數：

| 變數名稱 | 預設值 | 必填 | 說明 |
|---------|--------|------|------|
| `DATABASE_URL` | `file:./local-db/db.sqlite` | 否 | SQLite 資料庫檔案路徑（Prisma 格式） |
| `NODE_ENV` | `development` | 否 | 執行環境（`development` / `production`） |
| `PORT` | `1717` | 否 | Next.js 服務監聽埠號 |

### LLM 整合設定

**重要說明**：Easy Dataset 的 LLM 設定主要透過 Web UI 的「設定」->「模型設定」進行配置，而非環境變數。每個專案可獨立設定 LLM 提供商、API Key、Endpoint 與模型參數。

#### Azure OpenAI 整合設定

雖然 Easy Dataset 未提供專用的 Azure OpenAI 環境變數，但可透過 UI 設定使用 Azure OpenAI：

**設定步驟**：
1. 在專案中進入「設定」->「模型設定」
2. 選擇「OpenAI」作為提供商
3. 設定以下參數：
   - **Endpoint**：`https://{your-resource-name}.openai.azure.com/`
   - **API Key**：Azure OpenAI 的 API Key
   - **Model**：Deployment Name（例如：`gpt-4`、`gpt-35-turbo`）

**Azure OpenAI 標準變數對照**（僅供參考，實際使用需透過 UI 設定）：

| Azure 標準變數 | Easy Dataset 對應設定 | 說明 |
|---------------|---------------------|------|
| `AZURE_OPENAI_API_KEY` | UI 中的「API Key」欄位 | Azure OpenAI 的 API Key |
| `AZURE_OPENAI_ENDPOINT` | UI 中的「Endpoint」欄位 | Azure OpenAI 的端點 URL |
| `AZURE_OPENAI_DEPLOYMENT_NAME` | UI 中的「Model」欄位 | 部署的模型名稱 |
| `OPENAI_API_VERSION` | 不支援（固定使用最新版本） | API 版本（目前不支援自訂） |

**注意事項**：
- Azure OpenAI 的 Endpoint 格式為：`https://{resource-name}.openai.azure.com/`
- Deployment Name 需與 Azure Portal 中建立的部署名稱一致
- 目前不支援自訂 API Version，使用 SDK 預設版本

---

## 3. 安裝與部署 (Installation & Deployment)

### 3.1 本機安裝（NPM）

#### 前置需求
- Node.js 18+
- npm 或 pnpm
- Git

#### 安裝步驟

1. **複製專案**：
```bash
git clone https://github.com/ConardLi/easy-dataset.git
cd easy-dataset
```

2. **安裝依賴**：
```bash
npm install
# 或使用 pnpm
pnpm install
```

3. **初始化資料庫**：
```bash
npm run db:push
```

4. **建置應用**：
```bash
npm run build
```

5. **啟動服務**：
```bash
npm run start
```

6. **存取應用**：
開啟瀏覽器訪問 `http://localhost:1717`

#### 開發模式

若需進行開發，可使用開發模式（自動重載）：
```bash
npm run dev
```

### 3.2 Docker 部署

#### 使用官方 Docker Image

1. **建立 `docker-compose.yml`**：
```yaml
services:
  easy-dataset:
    image: ghcr.io/conardli/easy-dataset
    container_name: easy-dataset
    ports:
      - "1717:1717"
    volumes:
      - ./local-db:/app/local-db
    restart: unless-stopped
```

2. **啟動服務**：
```bash
docker-compose up -d
```

3. **存取應用**：
開啟瀏覽器訪問 `http://localhost:1717`

#### 使用本地 Dockerfile 建置

1. **建置映像檔**：
```bash
docker build -t easy-dataset .
```

2. **執行容器**：
```bash
docker run -d \
  -p 1717:1717 \
  -v ./local-db:/app/local-db \
  --name easy-dataset \
  easy-dataset
```

### 3.3 Kubernetes Helm 部署

由於專案提供 `Dockerfile` 與 `docker-compose.yml`，具備容器化服務特徵，以下提供 Kubernetes Helm Chart 部署指引。

#### Helm Chart 結構

建立以下目錄結構：
```
easy-dataset/
├── Chart.yaml
├── values.yaml
└── templates/
    ├── deployment.yaml
    ├── service.yaml
    ├── pvc.yaml
    └── configmap.yaml
```

#### Chart.yaml
```yaml
apiVersion: v2
name: easy-dataset
description: A Helm chart for Easy Dataset
version: 1.5.1
appVersion: "1.5.1"
```

#### values.yaml
```yaml
replicaCount: 1

image:
  repository: ghcr.io/conardli/easy-dataset
  tag: "latest"
  pullPolicy: IfNotPresent

service:
  type: NodePort
  port: 1717
  nodePort: 31908

persistence:
  enabled: true
  storageClass: "nfs-client"
  accessMode: ReadWriteOnce
  size: 10Gi
  mountPath: /app/local-db

resources:
  requests:
    memory: "512Mi"
    cpu: "250m"
  limits:
    memory: "2Gi"
    cpu: "1000m"

nodeSelector: {}
tolerations: []
affinity: {}
```

#### templates/deployment.yaml
```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: {{ include "easy-dataset.fullname" . }}
  labels:
    app: {{ include "easy-dataset.name" . }}
spec:
  replicas: {{ .Values.replicaCount }}
  selector:
    matchLabels:
      app: {{ include "easy-dataset.name" . }}
  template:
    metadata:
      labels:
        app: {{ include "easy-dataset.name" . }}
    spec:
      containers:
      - name: easy-dataset
        image: "{{ .Values.image.repository }}:{{ .Values.image.tag }}"
        imagePullPolicy: {{ .Values.image.pullPolicy }}
        ports:
        - name: http
          containerPort: 1717
          protocol: TCP
        env:
        - name: NODE_ENV
          value: "production"
        - name: PORT
          value: "1717"
        volumeMounts:
        - name: data
          mountPath: {{ .Values.persistence.mountPath }}
        resources:
          {{- toYaml .Values.resources | nindent 10 }}
      volumes:
      - name: data
        {{- if .Values.persistence.enabled }}
        persistentVolumeClaim:
          claimName: {{ include "easy-dataset.fullname" . }}-pvc
        {{- end }}
      {{- with .Values.nodeSelector }}
      nodeSelector:
        {{- toYaml . | nindent 8 }}
      {{- end }}
      {{- with .Values.affinity }}
      affinity:
        {{- toYaml . | nindent 8 }}
      {{- end }}
      {{- with .Values.tolerations }}
      tolerations:
        {{- toYaml . | nindent 8 }}
      {{- end }}
```

#### templates/service.yaml
```yaml
apiVersion: v1
kind: Service
metadata:
  name: {{ include "easy-dataset.fullname" . }}
  labels:
    app: {{ include "easy-dataset.name" . }}
spec:
  type: {{ .Values.service.type }}
  ports:
    - port: {{ .Values.service.port }}
      targetPort: http
      protocol: TCP
      name: http
      {{- if eq .Values.service.type "NodePort" }}
      nodePort: {{ .Values.service.nodePort }}
      {{- end }}
  selector:
    app: {{ include "easy-dataset.name" . }}
```

#### templates/pvc.yaml
```yaml
{{- if .Values.persistence.enabled }}
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: {{ include "easy-dataset.fullname" . }}-pvc
  labels:
    app: {{ include "easy-dataset.name" . }}
spec:
  accessModes:
    - {{ .Values.persistence.accessMode }}
  storageClassName: {{ .Values.persistence.storageClass }}
  resources:
    requests:
      storage: {{ .Values.persistence.size }}
{{- end }}
```

#### templates/_helpers.tpl
```yaml
{{- define "easy-dataset.name" -}}
{{- default .Chart.Name .Values.nameOverride | trunc 63 | trimSuffix "-" }}
{{- end }}

{{- define "easy-dataset.fullname" -}}
{{- if .Values.nameOverride }}
{{- .Values.nameOverride | trunc 63 | trimSuffix "-" }}
{{- else }}
{{- printf "%s-%s" .Chart.Name .Release.Name | trunc 63 | trimSuffix "-" }}
{{- end }}
{{- end }}
```

#### NFS Persistent Volume 範例

若需使用 NFS 作為儲存後端，可建立以下 PV：

```yaml
apiVersion: v1
kind: PersistentVolume
metadata:
  name: easy-dataset-nfs-pv
spec:
  capacity:
    storage: 10Gi
  accessModes:
    - ReadWriteOnce
  persistentVolumeReclaimPolicy: Retain
  storageClassName: nfs-client
  nfs:
    server: <NFS_SERVER_IP>
    path: /path/to/nfs/share
```

#### 部署指令

1. **安裝 Helm Chart**：
```bash
helm install easy-dataset ./easy-dataset
```

2. **升級部署**：
```bash
helm upgrade easy-dataset ./easy-dataset
```

3. **查看狀態**：
```bash
kubectl get pods -l app=easy-dataset
kubectl get svc -l app=easy-dataset
```

4. **存取應用**：
- NodePort 模式：`http://<NODE_IP>:31908`
- 或透過 Ingress 設定域名存取

---

## 4. 操作指南 (Operations)

### 4.1 基本操作

#### 啟動服務

**開發模式**：
```bash
npm run dev
```

**生產模式**：
```bash
npm run build
npm run start
```

**Docker 模式**：
```bash
docker-compose up -d
```

#### 建立專案

1. 開啟瀏覽器訪問 `http://localhost:1717`
2. 點擊「建立專案」
3. 輸入專案名稱與描述
4. 在「模型設定」中配置 LLM API

#### 處理文件

1. 進入專案後，點擊「Text Split」標籤
2. 上傳文件（支援 PDF、Markdown、DOCX、EPUB、TXT）
3. 檢視自動分割的文字塊，可手動調整
4. 檢視全域領域標籤樹

#### 生成問題與答案

1. 在「Distill」標籤中，批次生成問題
2. 檢視與編輯生成的問題
3. 在「Datasets」標籤中，批次生成答案
4. 檢視、編輯與優化生成的答案

#### 匯出資料集

1. 在「Datasets」標籤中，點擊「匯出」
2. 選擇格式（Alpaca、ShareGPT、multilingual-thinking）
3. 選擇檔案格式（JSON、JSONL）
4. 可新增自訂 System Prompt
5. 執行匯出

### 4.2 進階設定 (LLM Integration)

#### Azure OpenAI 整合設定

**步驟 1：取得 Azure OpenAI 資訊**
- 從 Azure Portal 取得：
  - Resource Name
  - API Key
  - Deployment Name
  - API Version（目前不支援自訂，使用 SDK 預設）

**步驟 2：在 Easy Dataset 中設定**

1. 進入專案「設定」->「模型設定」
2. 點擊「新增模型配置」
3. 選擇「OpenAI」作為提供商
4. 填入以下資訊：
   - **Endpoint**：`https://{resource-name}.openai.azure.com/`
   - **API Key**：Azure OpenAI 的 API Key
   - **Model**：Deployment Name（例如：`gpt-4`）
   - **Temperature**：建議 0.7
   - **Max Tokens**：建議 8192

**步驟 3：測試連線**

1. 在模型設定頁面，點擊「測試連線」
2. 確認可成功連線至 Azure OpenAI
3. 若失敗，檢查：
   - Endpoint 格式是否正確
   - API Key 是否有效
   - Deployment Name 是否正確

**範例設定**：
```
Provider: OpenAI
Endpoint: https://my-resource.openai.azure.com/
API Key: xxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx
Model: gpt-4
Temperature: 0.7
Max Tokens: 8192
```

#### 其他 LLM 提供商設定

Easy Dataset 支援多種 LLM 提供商，設定方式類似：

| 提供商 | Endpoint 範例 | 備註 |
|--------|-------------|------|
| OpenAI | `https://api.openai.com/v1/` | 標準 OpenAI API |
| Ollama | `http://127.0.0.1:11434/api` | 本地部署模型 |
| 智譜AI | `https://open.bigmodel.cn/api/paas/v4/` | 需申請 API Key |
| OpenRouter | `https://openrouter.ai/api/v1/` | 多模型聚合服務 |

### 4.3 故障排除 (Troubleshooting)

#### Azure 連線錯誤

**錯誤 401 (Unauthorized)**
- **原因**：API Key 無效或過期
- **解決**：檢查 Azure Portal 中的 API Key，確認是否正確複製

**錯誤 404 (Not Found)**
- **原因**：Endpoint 或 Deployment Name 錯誤
- **解決**：
  1. 確認 Endpoint 格式：`https://{resource-name}.openai.azure.com/`
  2. 確認 Deployment Name 與 Azure Portal 中的部署名稱一致
  3. 確認 Deployment 狀態為「已部署」

**錯誤 429 (Too Many Requests)**
- **原因**：API 請求頻率超過限制
- **解決**：
  1. 降低並行處理數量（在「任務設定」中調整 `concurrencyLimit`）
  2. 增加請求間隔時間
  3. 檢查 Azure OpenAI 的 Rate Limit 設定

#### NFS 掛載失敗（Kubernetes）

**錯誤：無法掛載 NFS Volume**
- **原因**：NFS Server 無法連線或路徑不存在
- **解決**：
  1. 確認 NFS Server 可從 Kubernetes 節點存取
  2. 確認 NFS 路徑存在且權限正確
  3. 檢查 StorageClass 設定
  4. 查看 Pod 事件：`kubectl describe pod <pod-name>`

#### 資料庫問題

**錯誤：資料庫初始化失敗**
- **原因**：資料庫檔案權限或路徑問題
- **解決**：
  1. 確認 `local-db` 目錄存在且可寫入
  2. 執行 `npm run db:push` 重新初始化
  3. 檢查 `DATABASE_URL` 環境變數（若設定）

**錯誤：Prisma Client 未生成**
- **原因**：Prisma Client 未正確生成
- **解決**：
```bash
npx prisma generate
npm run db:push
```

#### Node.js 套件相依性問題

**錯誤：模組找不到**
- **原因**：依賴未正確安裝
- **解決**：
```bash
rm -rf node_modules package-lock.json
npm install
# 或使用 pnpm
pnpm install
```

**錯誤：Sharp 模組錯誤（圖片處理）**
- **原因**：Sharp 的 native 模組編譯失敗
- **解決**：
```bash
npm rebuild sharp
# 或重新安裝
npm uninstall sharp
npm install sharp
```

#### 記憶體不足

**錯誤：JavaScript heap out of memory**
- **原因**：處理大檔案時記憶體不足
- **解決**：
  1. 增加 Node.js 記憶體限制：
```bash
NODE_OPTIONS="--max-old-space-size=4096" npm run start
```
  2. 在 Docker 中增加記憶體限制
  3. 分批處理大檔案

---

## 5. 範例與截圖 (Examples)

### 5.1 基本使用流程

> [圖片說明：此處應顯示 Easy Dataset 首頁，包含「建立專案」按鈕與專案列表]

**建立專案範例**：
1. 點擊「建立專案」
2. 輸入專案名稱：「技術文件微調資料集」
3. 輸入描述：「用於建立技術文件的問答資料集」
4. 配置 LLM 模型（例如：GPT-4）

> [圖片說明：此處應顯示專案建立對話框，包含名稱、描述與模型設定選項]

### 5.2 文件處理範例

**上傳文件**：
- 支援格式：PDF、Markdown、DOCX、EPUB、TXT
- 檔案大小限制：建議單檔不超過 50MB

> [圖片說明：此處應顯示文件上傳介面，包含拖放區域與檔案列表]

**文字分割結果**：
- 自動分割為多個文字塊
- 可手動調整分割點
- 顯示每個文字塊的摘要

> [圖片說明：此處應顯示文字分割結果，包含文字塊列表與預覽]

### 5.3 問題生成範例

**批次生成問題**：
- 基於文字塊自動生成相關問題
- 支援自訂提示詞
- 可編輯與篩選問題

> [圖片說明：此處應顯示問題生成介面，包含問題列表與編輯功能]

### 5.4 答案生成範例

**批次生成答案**：
- 使用設定的 LLM 模型生成答案
- 支援 Chain of Thought (COT)
- 可編輯與評分答案

> [圖片說明：此處應顯示答案生成介面，包含答案列表與編輯功能]

### 5.5 資料集匯出範例

**匯出格式選項**：
- Alpaca 格式
- ShareGPT 格式
- multilingual-thinking 格式

**匯出設定**：
```json
{
  "format": "alpaca",
  "fileType": "jsonl",
  "systemPrompt": "你是一個專業的技術文件助手。"
}
```

> [圖片說明：此處應顯示資料集匯出對話框，包含格式選項與設定]

### 5.6 匯出檔案範例

**Alpaca 格式範例**：
```json
{
  "instruction": "什麼是資料湖？",
  "input": "",
  "output": "資料湖是一個集中式儲存庫，允許以原始格式儲存大量資料..."
}
```

**ShareGPT 格式範例**：
```json
{
  "conversations": [
    {
      "from": "human",
      "value": "什麼是資料湖？"
    },
    {
      "from": "gpt",
      "value": "資料湖是一個集中式儲存庫，允許以原始格式儲存大量資料..."
    }
  ]
}
```

---

## 附錄

### A. 常用指令參考

```bash
# 開發模式
npm run dev

# 建置
npm run build

# 啟動生產服務
npm run start

# 資料庫操作
npm run db:push        # 推送資料庫 schema
npm run db:studio      # 開啟 Prisma Studio

# Docker 操作
docker-compose up -d    # 啟動服務
docker-compose down     # 停止服務
docker-compose logs -f  # 查看日誌

# Electron 桌面應用
npm run electron-dev   # 開發模式
npm run electron-build # 建置桌面應用
```

### B. 相關資源

- **官方文件**：<https://docs.easy-dataset.com/ed/en>
- **GitHub 專案**：<https://github.com/ConardLi/easy-dataset>
- **論文**：<https://arxiv.org/abs/2507.04009v1>
- **社群討論**：<https://docs.easy-dataset.com/geng-duo/lian-xi-wo-men>
