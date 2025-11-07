+++
title = "GitHub Copilot Coding Agent Playwright MCP 外部主機存取設定教學"
description = "在 GitHub Copilot Coding Agent 中設定 Playwright MCP 的 --allowed-hosts 參數，讓其能存取外部 CDN 與主機。"
date = "2025-11-07T06:53:38Z"
updated = "2025-11-07T07:19:33Z"
draft = false

[taxonomies]
tags = [ "GitHub", "MCP" ]
providers = [ "ChatGPT" ]

[extra]
withAI = "<https://chatgpt.com/share/690d9772-61ec-8005-b8c2-62183069c3a1>"
+++
{% chat(speaker="jim") %}
when using github coding agent, how to allow playwright MCP to get access to external hosts such as CDN? I don't mind allowing * although it may have security issue. Please guide me do the setup on GitHub.
{% end %}

{% chat(speaker="chatgpt") %}
可以的！

重點做法是：在你的 GitHub 儲存庫裡為 Copilot coding agent 加上一個自訂的 Playwright MCP 設定，並在參數裡加上 `--allowed-hosts *`（或列出你要允許的網域清單），讓瀏覽器可以連到外部主機／ CDN。

下面一步一步帶你在 GitHub 上完成設定。
{% end %}

## 為 Playwright MCP 開外網存取（含萬用字元）

1) **到儲存庫設定頁新增 MCP 設定**  
在 GitHub 儲存庫的 **Settings  Copilot  Coding agent** 頁面，找到 **MCP configuration** 區塊，貼上這段 JSON，然後儲存： [^2]

```json
{
  "mcpServers": {
    "playwright": {
      "type": "local",
      "command": "npx",
      "args": ["@playwright/mcp@latest", "--allowed-hosts", "*"],
      "tools": ["*"]
    }
  }
}
```

說明：  

- `--allowed-hosts *` 會解除伺服器的主機檢查，允許從任意主機提供內容（你也可以改成逗號分隔的網域清單，例如 `cdn.jsdelivr.net,unpkg.com`）。這個旗標是 Playwright MCP 內建的參數。 [^1]  
- Copilot coding agent 只支援 MCP 的「工具」；上面用 `tools: ["*"]` 全開，如果你想更安全，可改成只列你需要的工具名稱。 [^2]

> 為什麼要自己加這段？因為預設的 Playwright MCP 只看得到 `localhost / 127.0.0.1`，要連外部主機就要自行加上如上設定。 [^4]

2) **（選用）精準允許特定外站**  
如果你不想用萬用字元，而是只開特定 CDN，可以改成：

```json
{
  "mcpServers": {
    "playwright": {
      "type": "local",
      "command": "npx",
      "args": [
        "@playwright/mcp@latest",
        "--allowed-hosts", "cdn.jsdelivr.net,unpkg.com,static.cloudflareinsights.com",
        "--allowed-origins", "https://cdn.jsdelivr.net;https://unpkg.com;https://*.cloudflare.com"
      ],
      "tools": ["*"]
    }
  }
}
```

- `--allowed-origins` 是以分號分隔的清單，用來限制瀏覽器實際可發出的請求來源；預設是全允許，你也可以明確列出。 [^1]

3) **驗證是否生效**  
建立一個 Issue 指派給 Copilot，等它開好 PR 之後，點進 **View session** 檢查「Start MCP Servers」步驟的日誌，應能看到你的 Playwright MCP 工具已啟動。 [^2]

---

## 和防火牆、組織政策的關係（重要）

- **Agent 防火牆不會管 MCP 伺服器。** Copilot 的代理防火牆只會限制透過 Bash 工具啟動的行程，**不**套用到 MCP 伺服器；所以要讓 Playwright MCP 出網，不需要改 agent 防火牆，重點是上面那段 MCP 設定。 [^3]  
- **組織有 MCP 允許清單政策時。** 若你們的 Enterprise／Organization 啟用了 MCP registry 與「Registry only」政策，你也要確保 `@playwright/mcp` 在允許名單中，否則 IDE／代理可能擋住執行。管理員可在 **AI controls  MCP**（企業）或 **Organization Settings  Copilot  Policies**（組織）設定。 [^5]

---

## 常見加強與除錯

- 內網或自簽憑證的 CDN 需要放寬 TLS 驗證時，可追加 `--ignore-https-errors`（僅測試用途，務必審慎）。 [^1]  
- Playwright MCP 的更多參數（例如 `--host`、`--port`、`--blocked-origins` 等）都可用同樣方式放進 `args`。 [^1]

[^1]: [GitHub - microsoft/playwright-mcp: Playwright MCP server](https://github.com/microsoft/playwright-mcp)
[^2]: [Extending GitHub Copilot coding agent with the Model Context Protocol (MCP) - GitHub Docs](https://docs.github.com/copilot/how-tos/agents/copilot-coding-agent/extending-copilot-coding-agent-with-mcp)
[^3]: [Customizing or disabling the firewall for GitHub Copilot coding agent - GitHub Docs](https://docs.github.com/copilot/how-tos/agents/copilot-coding-agent/customizing-or-disabling-the-firewall-for-copilot-coding-agent)
[^4]: [Model Context Protocol (MCP) and GitHub Copilot coding agent - GitHub Docs](https://docs.github.com/en/copilot/concepts/agents/coding-agent/mcp-and-coding-agent)
[^5]: [Configure MCP server access for your organization or enterprise - GitHub Docs](https://docs.github.com/en/copilot/how-tos/administer-copilot/configure-mcp-server-access)
