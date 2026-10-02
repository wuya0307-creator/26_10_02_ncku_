# 26_10_02_ncku_

> 以 **Revit MCP** 結合 AI 助理（Claude）操作 Autodesk Revit 的學習與實作專案。

## 專案簡介

本專案整理 Revit MCP 的安裝、設定與使用方式，目標是讓使用者能以自然語言（中文）下指令，由 AI 透過 MCP（Model Context Protocol）協定在 Revit 中查詢、建立與修改 BIM 模型元素。

## 檔案說明

| 檔案 | 說明 |
|------|------|
| [`Revit_MCP_使用手冊.md`](./Revit_MCP_使用手冊.md) | Revit MCP 完整使用手冊：架構、安裝、AI 用戶端設定、常用工具、提示詞範例、疑難排解 |
| `README.md` | 本檔案，專案總覽 |

## 快速開始

1. **安裝 Node.js 18+ 與 Revit**（建議 2023–2025）。
2. **安裝 MCP Server**：
   ```bash
   git clone https://github.com/revit-mcp/revit-mcp.git
   cd revit-mcp
   npm install
   npm run build
   ```
3. **安裝 Revit 外掛與指令集**：將 `revit-mcp-plugin` 與 `revit-mcp-commandset` 放入
   `C:\ProgramData\Autodesk\Revit\Addins\<版本>\`。
4. **連接 AI 用戶端**（以 Claude Code 為例）：
   ```bash
   claude mcp add revit-mcp -- node <路徑>/revit-mcp/build/index.js
   ```
5. **在 Revit 中啟動服務**：Add-Ins → Revit MCP Plugin → **Revit MCP Switch**。
6. 開始對 AI 下指令，例如：
   ```
   列出目前視圖中所有牆的數量與類型。
   ```

詳細步驟請參考 👉 [Revit MCP 使用手冊](./Revit_MCP_使用手冊.md)

## 系統架構

```
AI 用戶端 (Claude)  ⇄  revit-mcp (Node.js MCP Server)  ⇄  Revit 外掛 (C#)  ⇄  Revit API
```

## 使用注意事項

- 操作前請先**備份 Revit 專案檔**。
- 執行刪除或自訂程式碼（`send_code_to_revit`）前，請先確認 AI 產生的內容。
- MCP 服務僅在本機（localhost）運作，不使用時請關閉。

## 參考資源

- [Model Context Protocol](https://modelcontextprotocol.io)
- [revit-mcp](https://github.com/revit-mcp/revit-mcp)
- [revit-mcp-plugin](https://github.com/revit-mcp/revit-mcp-plugin)
- [revit-mcp-commandset](https://github.com/revit-mcp/revit-mcp-commandset)
- [Claude Code MCP 文件](https://docs.claude.com/en/docs/claude-code/mcp)
