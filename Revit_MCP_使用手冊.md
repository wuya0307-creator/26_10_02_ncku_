# Revit MCP 使用手冊

> 本手冊說明如何安裝、設定並使用 **Revit MCP**，讓 AI 助理（如 Claude Desktop、Claude Code）透過自然語言操作 Autodesk Revit。

---

## 目錄

1. [什麼是 Revit MCP](#1-什麼是-revit-mcp)
2. [系統架構](#2-系統架構)
3. [環境需求](#3-環境需求)
4. [安裝步驟](#4-安裝步驟)
5. [連接 AI 用戶端](#5-連接-ai-用戶端)
6. [啟動與使用流程](#6-啟動與使用流程)
7. [常用工具（指令）一覽](#7-常用工具指令一覽)
8. [提示詞範例](#8-提示詞範例)
9. [疑難排解](#9-疑難排解)
10. [安全與使用建議](#10-安全與使用建議)
11. [參考資源](#11-參考資源)

---

## 1. 什麼是 Revit MCP

**MCP（Model Context Protocol）** 是一種開放協定，讓大型語言模型（LLM）能以標準化方式呼叫外部工具與資料來源。

**Revit MCP** 則是將 Revit 的 API 包裝成 MCP 工具，使 AI 能夠：

- 讀取目前視圖、選取元素、族群類型等模型資訊
- 建立牆、門窗、樓板、柱等元素
- 修改、刪除、上色（Color Splash）與標記元素
- 依條件篩選元素並輸出統計資料
- 在 Revit 中執行自訂的 C# 程式碼（進階）

換句話說：**你用中文對 AI 下指令，AI 幫你在 Revit 裡操作模型。**

---

## 2. 系統架構

```
┌──────────────────┐    MCP (stdio)    ┌──────────────────┐   Socket (localhost)   ┌─────────────────────┐
│  AI 用戶端         │ ◄──────────────► │  revit-mcp        │ ◄────────────────────► │  Revit 外掛          │
│ Claude Desktop /  │                   │  (Node.js 伺服器)  │                        │ revit-mcp-plugin    │
│ Claude Code 等     │                   │                   │                        │ + commandset        │
└──────────────────┘                   └──────────────────┘                        └─────────────────────┘
                                                                                             │
                                                                                             ▼
                                                                                      Autodesk Revit API
```

| 元件 | 說明 |
|------|------|
| **revit-mcp**（MCP Server） | 以 TypeScript / Node.js 撰寫，負責向 AI 宣告可用工具，並把請求轉送給 Revit |
| **revit-mcp-plugin**（Revit 外掛） | 以 C# 撰寫的 Revit Add-in，在 Revit 內開啟本機 Socket 服務接收指令 |
| **revit-mcp-commandset**（指令集） | 實際執行 Revit API 操作的指令實作，可自行擴充 |

---

## 3. 環境需求

| 項目 | 需求 |
|------|------|
| 作業系統 | Windows 10 / 11 |
| Revit | 2019 以上（建議 2023–2025，依外掛發行版本而定） |
| Node.js | 18 以上（安裝後於終端機輸入 `node -v` 確認） |
| Git | 用於下載原始碼（選用） |
| AI 用戶端 | Claude Desktop、Claude Code，或其他支援 MCP 的用戶端 |
| .NET / Visual Studio | 僅在需要自行編譯外掛時才需要 |

---

## 4. 安裝步驟

### 4.1 安裝 MCP Server（revit-mcp）

```bash
# 1. 下載原始碼
git clone https://github.com/revit-mcp/revit-mcp.git
cd revit-mcp

# 2. 安裝相依套件
npm install

# 3. 編譯
npm run build
```

完成後會產生 `build/index.js`，請記下其**完整路徑**，例如：

```
C:\Tools\revit-mcp\build\index.js
```

### 4.2 安裝 Revit 外掛（revit-mcp-plugin）

1. 至 [revit-mcp-plugin Releases](https://github.com/revit-mcp/revit-mcp-plugin/releases) 下載對應 Revit 版本的壓縮檔（或自行用 Visual Studio 編譯）。
2. 將 `.addin` 檔與外掛資料夾複製到 Revit Add-ins 目錄：

   ```
   C:\ProgramData\Autodesk\Revit\Addins\<Revit 版本>\
   ```

   例如 Revit 2024：`C:\ProgramData\Autodesk\Revit\Addins\2024\`

3. 確認 `.addin` 檔中的 `<Assembly>` 路徑指向正確的 `.dll`。

### 4.3 安裝指令集（revit-mcp-commandset）

1. 至 [revit-mcp-commandset Releases](https://github.com/revit-mcp/revit-mcp-commandset/releases) 下載指令集。
2. 依外掛說明將指令集放入外掛資料夾下的 `Commands` 目錄（實際路徑請依版本 README 為準）。
3. 啟動 Revit 時若跳出「載入外掛」的安全提示，請選擇 **「總是載入」**。

---

## 5. 連接 AI 用戶端

### 5.1 Claude Desktop

開啟設定檔（Claude Desktop → Settings → Developer → Edit Config），路徑通常為：

```
%APPDATA%\Claude\claude_desktop_config.json
```

加入以下內容（路徑請改成你自己的，Windows 路徑中的 `\` 需寫成 `\\` 或 `/`）：

```json
{
  "mcpServers": {
    "revit-mcp": {
      "command": "node",
      "args": ["C:/Tools/revit-mcp/build/index.js"]
    }
  }
}
```

儲存後**完全關閉並重新開啟** Claude Desktop，在對話框的工具圖示中應可看到 `revit-mcp` 的工具。

### 5.2 Claude Code

在終端機執行：

```bash
claude mcp add revit-mcp -- node C:/Tools/revit-mcp/build/index.js
```

確認是否加入成功：

```bash
claude mcp list
```

在 Claude Code 對話中輸入 `/mcp` 可檢查連線狀態。

---

## 6. 啟動與使用流程

> ⚠️ **順序很重要**：Revit 端的服務必須先啟動，AI 才能成功呼叫工具。

1. **開啟 Revit**，並開啟一個專案檔（`.rvt`）。
2. 切換到 **Add-Ins（增益集）** 頁籤，找到 **Revit MCP Plugin** 面板。
3. 點選 **Settings**，勾選要啟用的指令（第一次使用時建議全部勾選後儲存）。
4. 點選 **Revit MCP Switch** 開啟服務（預設監聽本機連接埠，例如 `localhost:8080`）。
5. 開啟 Claude Desktop / Claude Code，開始用自然語言下指令。
6. 使用完畢後，再次點選 **Revit MCP Switch** 關閉服務。

---

## 7. 常用工具（指令）一覽

> 實際可用工具依 commandset 版本而異，請以 AI 用戶端中顯示的工具清單為準。

### 查詢類

| 工具名稱 | 功能 |
|----------|------|
| `get_current_view_info` | 取得目前視圖資訊（名稱、類型、比例等） |
| `get_current_view_elements` | 取得目前視圖中的元素 |
| `get_selected_elements` | 取得使用者目前選取的元素 |
| `get_available_family_types` | 列出專案中可用的族群與類型 |
| `ai_element_filter` | 依條件（類別、參數、範圍等）篩選元素 |

### 建立類

| 工具名稱 | 功能 |
|----------|------|
| `create_point_based_element` | 建立點式元素（門、窗、家具等） |
| `create_line_based_element` | 建立線式元素（牆、梁等） |
| `create_surface_based_element` | 建立面式元素（樓板、天花板等） |

### 修改 / 視覺化類

| 工具名稱 | 功能 |
|----------|------|
| `operate_element` | 選取、隱藏、隔離、上色等元素操作 |
| `color_elements` / `color_splash` | 依參數值為元素上色 |
| `tag_all_walls` / `tag_walls` | 為牆加上標籤 |
| `delete_element` | 刪除指定元素 |

### 進階

| 工具名稱 | 功能 |
|----------|------|
| `send_code_to_revit` | 將 C# 程式碼送到 Revit 中執行 |

---

## 8. 提示詞範例

**查詢模型資訊**
```
請告訴我目前視圖的名稱與類型，並列出視圖中所有牆的數量與類型。
```

**建立元素**
```
在 Level 1 從 (0,0) 到 (10000,0) 建立一道 200mm 厚的基本牆，高度 3000mm。
```

**放置門窗**
```
在剛剛建立的牆中點放一扇 900 x 2100 的單扇門。
```

**視覺化分析**
```
請依照「防火等級」參數為所有牆上色，不同數值使用不同顏色。
```

**統計輸出**
```
統計專案中所有房間的名稱與面積，用表格列出並計算總面積。
```

**批次標記**
```
幫目前視圖中所有的牆加上類型標籤。
```

### 撰寫提示詞的技巧

- **明確指定單位**：Revit 內部單位為英尺，建議在提示中寫清楚「mm」或「m」。
- **指定樓層與視圖**：例如「在 Level 1」、「在目前平面視圖」。
- **先查詢再操作**：先讓 AI 列出可用族群類型，再指定要用哪一種。
- **分步驟進行**：複雜任務拆成多個小步驟，每步確認結果後再繼續。

---

## 9. 疑難排解

| 問題 | 可能原因與解法 |
|------|----------------|
| AI 用戶端看不到 revit-mcp 工具 | 檢查設定檔 JSON 格式與 `index.js` 路徑；重新啟動用戶端；確認 `node -v` 可正常執行 |
| 呼叫工具時顯示連線失敗 / 逾時 | 確認 Revit 已開啟專案且已按下 **Revit MCP Switch**；檢查防火牆是否阻擋本機連接埠；確認連接埠未被其他程式占用 |
| Revit 中找不到外掛面板 | 確認 `.addin` 放在正確的版本資料夾；`.addin` 中的 dll 路徑正確；外掛版本與 Revit 版本相符 |
| 工具可呼叫但回傳「指令未註冊」 | 至 **Settings** 勾選該指令並儲存；確認 commandset 已放入正確目錄 |
| 建立的元素位置 / 尺寸不對 | 檢查單位（mm vs. ft）；明確指定樓層與座標 |
| Revit 跳出對話框後指令卡住 | 先手動關閉 Revit 中的對話框，再重新下指令 |

---

## 10. 安全與使用建議

- ✅ **操作前先備份**專案檔，或在測試用的副本上練習。
- ✅ 善用 Revit 的 **復原（Ctrl + Z）**，AI 的每次操作通常可單獨復原。
- ⚠️ `send_code_to_revit` 會直接執行程式碼，請先檢視 AI 產生的程式內容再允許執行。
- ⚠️ 刪除類指令請先讓 AI 列出要刪除的元素清單，確認無誤再執行。
- 🔒 服務僅應監聽本機（localhost），不要對外開放連接埠。
- 💡 不使用時記得關閉 **Revit MCP Switch**。

---

## 11. 參考資源

- Model Context Protocol 官方文件：<https://modelcontextprotocol.io>
- revit-mcp（MCP Server）：<https://github.com/revit-mcp/revit-mcp>
- revit-mcp-plugin（Revit 外掛）：<https://github.com/revit-mcp/revit-mcp-plugin>
- revit-mcp-commandset（指令集）：<https://github.com/revit-mcp/revit-mcp-commandset>
- Claude Code MCP 設定說明：<https://docs.claude.com/en/docs/claude-code/mcp>
- Revit API 文件：<https://www.revitapidocs.com>

---

*最後更新：2026-10-02*
