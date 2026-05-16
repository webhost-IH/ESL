# 英文樂園（English Paradise）— 專案現況總結

> 最後更新：2026-05-16

---

## 一、專案簡介

| 項目 | 內容 |
|------|------|
| App 名稱 | 英文樂園 / English Paradise |
| 類型 | 兒童英文閱讀練習 App |
| 框架 | Capacitor 6.x（單一 HTML 檔案） |
| 主要檔案 | `www/index.html`（約 3,500 行） |
| GitHub | https://github.com/webhost-IH/ESL |

---

## 二、文章內容

| Level | 文章數 | 難度說明 |
|-------|--------|----------|
| Level 1 | 20 篇 | 啟蒙 ~10 字 |
| Level 2 | 24 篇 | 啟蒙 ~20 字 |
| Level 3 | 28 篇 | 正式 ~40 字 |
| Level 4 | 36 篇 | 中年 ~60 字 |
| Level 5 | 36 篇 | 高年 ~80 字 |
| Level 6 | 28 篇 | 高年 ~100 字 |
| **合計** | **172 篇** | （去除 428 篇重複後） |

- 每篇文章包含：英文原文、中文翻譯（`zh`）、核心單字（`vocab`）
- 只保留上學期（Semester 1），下學期內容已移除

---

## 三、已完成功能

### 3.1 多語系（i18n）— 8 種語言
| 代碼 | 語言 |
|------|------|
| zh-TW | 繁體中文（預設） |
| zh-CN | 簡體中文 |
| en | English |
| ja | 日本語 |
| ko | 한국어 |
| vi | Tiếng Việt |
| es | Español |
| th | ภาษาไทย |

- 點擊 topbar 的 🌐 按鈕開啟語言選擇器
- App 標題、所有 UI 標籤、按鈕均跟隨語言切換
- 語言設定儲存於 `localStorage`，重開 App 自動恢復

### 3.2 即時翻譯（MyMemory API）
- **文章翻譯**：非中文語言時，自動呼叫 MyMemory 免費 API 翻譯整篇文章
  - 長文章（>450 字元）自動切句分批送出，避免超出 API 限制
  - 翻譯結果快取，同一篇不重複呼叫
  - 顯示 `⏳ ...` loading 狀態等待翻譯
- **單字 tooltip**：
  - 中文模式：使用內建靜態字典（離線，即時顯示）
  - 其他語言：hover / 點觸單字時懶翻譯，有快取

### 3.3 學習功能
- **完成這篇**：點按後記錄到學習歷史，自動跳到下一篇
- **連勝天數（Streak）**：每天完成至少一篇即累計
- **學習進度頁**：顯示總完成篇數、連勝天數、最長連勝、學習日曆
- **歷史紀錄**：記錄每篇完成的日期與 Level

### 3.4 朗讀功能
- 支援正常速度與慢速（0.55x）
- 使用 Capacitor TextToSpeech Plugin（iOS 裝置上）
- 瀏覽器環境 fallback 至 Web Speech API
- 音量滑桿控制
- 朗讀時單字逐字高亮

### 3.5 字型（本地化，無需網路）
| 字型 | 檔案 |
|------|------|
| Nunito 400/600/700/800/900 | `www/fonts/nunito-latin-*.woff2` |
| Fredoka One | `www/fonts/fredoka-one-latin.woff2` |

### 3.6 iOS 專案
- Xcode 專案位於 `ios/App/App.xcworkspace`
- 已安裝 CocoaPods（3 個 pods）
- 最後同步：`npx cap sync ios`

---

## 四、已修復的 Bug

| Bug | 原因 | 修復方式 |
|-----|------|----------|
| 「完成這篇」沒有跳下一篇 | `const t = todayStr()` 蓋掉了全域 `t()` 翻譯函式，導致後續程式中止 | 改名為 `const today = todayStr()` |
| 語言選擇器無法開啟 | Modal HTML 未正確插入 | 用 `sed` 補插 |
| STRINGS TDZ 錯誤 | Python 替換時誤把 STRINGS 內的字串也換成 `t()` 呼叫，造成初始化循環依賴 | 手動修正第 3083 行 |
| `_lang` 永遠是 zh-TW | `||` 優先順序高於 `?:`，localStorage 值被忽略 | 用括號修正優先順序 |
| 文章翻譯 API 超過 500 字元限制 | 直接把整篇送 API | `translateLong()`：按句切割成 ≤450 字元小塊分批翻譯 |
| 重複文章 | 每個 Level 都有大量重複（總計 428 篇） | Python 去重，600 → 172 篇 |
| articleIdx 超出範圍 | 去重後篇數縮短，舊存檔 index 超界 | `initLearnPage()` 加入邊界檢查 |

---

## 五、專案檔案結構

```
english-app/
├── www/
│   ├── index.html          # 主要 App（所有功能集中此檔）
│   ├── privacy.html        # 隱私權政策（中文）
│   └── fonts/
│       ├── fonts.css
│       ├── nunito-latin-400.woff2
│       ├── nunito-latin-600.woff2
│       ├── nunito-latin-700.woff2
│       ├── nunito-latin-800.woff2
│       ├── nunito-latin-900.woff2
│       └── fredoka-one-latin.woff2
├── ios/                    # Xcode iOS 專案（Capacitor）
├── android/                # Android 專案（Capacitor）
├── resources/icons/        # App 圖示（iOS + Android 所有尺寸）
├── generate-icons.py       # 圖示生成腳本（Python Pillow）
├── publish-privacy.sh      # 隱私權政策發布腳本
├── capacitor.config.json
└── package.json
```

---

## 六、待完成事項（需手動）

1. **申請 Apple 開發者帳號**（$99 USD/年）
   - https://developer.apple.com
2. **Xcode 設定 Code Signing**
   - 開啟 `ios/App/App.xcworkspace`
   - Signing & Capabilities → 選擇 Apple ID
3. **App Store Connect 建立 App 條目**
   - App 名稱、截圖（6.5吋 + 5.5吋）、描述
   - 年齡分級：4+
   - 隱私政策 URL：`https://www.inheart.tw/ESL/privacy.html`
4. **送審**
   - Xcode → Product → Archive → Distribute App → App Store Connect

---

## 七、技術備注

- **翻譯 API**：MyMemory（免費，無需 API Key，每 IP 每天約 1,000 字翻譯額度）
- **離線支援**：中文模式完全離線（內建靜態字典 + 本地字型）；其他語言翻譯需網路
- **資料儲存**：`localStorage`（學習進度、語言設定）
- **iOS 同步指令**：
  ```bash
  export LANG=en_US.UTF-8
  export DEVELOPER_DIR=/Applications/Xcode.app/Contents/Developer
  PATH="/tmp/node-v20.19.0-darwin-arm64/bin:$PATH" npx cap sync ios
  ```
