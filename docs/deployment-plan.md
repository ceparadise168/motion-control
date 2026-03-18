# Motion Control 部署方案

## 專案架構特性

- 純前端靜態應用，單一 HTML 檔案（51 KB）
- 所有運算（手勢辨識、3D 粒子特效）在使用者裝置端執行
- 重量級資源（MediaPipe、Three.js、Google Fonts）透過第三方 CDN 載入
- 無後端伺服器、無資料庫、無 API 呼叫

## 方案 A：GitHub Pages（主方案）

| 項目 | 說明 |
|------|------|
| 狀態 | ✅ 已設定完成，push 即部署 |
| 網址 | https://ceparadise168.github.io/motion-control/ |
| 流量上限 | 100 GB / 月 |
| CDN | Fastly（全球節點） |
| 費用 | 免費 |
| 3000 人同時訪問 | 3000 × 51 KB = 150 MB，遠低於上限 |

### 為什麼足夠？

1. 伺服器只傳送 51 KB 的 HTML，其餘資源由 Google / jsDelivr CDN 負擔
2. GitHub Pages 背後是 Fastly CDN，具備全球快取能力
3. 無伺服器運算負載，不存在 CPU / 記憶體瓶頸

## 方案 B：Cloudflare Pages（備案）

| 項目 | 說明 |
|------|------|
| 切換時間 | 5 分鐘 |
| 流量上限 | **無限制**（免費方案） |
| CDN | Cloudflare（全球 300+ 節點） |
| 費用 | 免費 |
| 自訂網域 | 支援 |

### 啟用步驟

1. 註冊 Cloudflare 帳號（https://dash.cloudflare.com/sign-up）
2. 進入 Workers & Pages → Create → Pages → Connect to Git
3. 選擇 `ceparadise168/motion-control` repo
4. Build 設定留空（無需建置步驟），輸出目錄填 `/`
5. 部署完成，取得 `*.pages.dev` 網址

### 何時啟用備案？

- GitHub Pages 出現異常或回應緩慢
- 需要自訂網域（如 `demo.company.com`）
- 實際流量遠超預期（雖然機率極低）

## 風險評估

| 風險項目 | 可能性 | 影響 | 應對 |
|----------|--------|------|------|
| 伺服器流量過載 | 極低 | 頁面載入緩慢 | 切換至方案 B |
| 第三方 CDN 故障 | 極低 | 功能無法載入 | 與部署方案無關，兩方案皆受影響 |
| 使用者裝置效能不足 | 中等 | 體驗卡頓 | 屬客戶端問題，非部署問題 |
| GitHub Pages 服務中斷 | 極低 | 網站無法訪問 | 切換至方案 B（5 分鐘） |
