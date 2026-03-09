# Agentic Workflow Guide for Crawlstack

This guide is intended for AI agents (like Gemini, Claude, or GPT) who are operating the Crawlstack infrastructure via MCP.

## 1. Discovery Phase

Before acting, always map the current environment:
1. Call `list_nodes` to identify active browser instances.
2. Call `extension_list_crawlers` on a node to see what's already built.
3. If you are fixing a bug, call `extension_get_run_logs` or `extension_get_run_items` to understand the failure state.

## 2. Development Cycle (The "Preview" First Pattern)

Never create a permanent crawler without testing the script first.

1. **Test Selectors**: Use `extension_preview_script` with `keep_alive: true`.
2. **Visual Feedback**: If items aren't being published, use `extension_take_tab_screenshot` to see if a popup or "Accept Cookies" banner is blocking the view.
3. **Handle Interstitials**: If a banner is found, call `extension_preview_script` on the *same tab* with code to click the "Accept" button.
4. **Finalize**: Once the preview returns the expected items, use `extension_upsert_crawler` to save the script.

## 3. Dealing with Complex Data

- **Downloads**: If the goal is a file, use `await runner.enableDownloads()` and then click the target. Wait for the file to appear in `await runner.getDownloads()`.
- **Parsing**: If the file is binary (XLSX), inject a parser:
  ```javascript
  const script = document.createElement('script');
  script.src = 'https://cdnjs.cloudflare.com/ajax/libs/xlsx/0.18.5/xlsx.full.min.js';
  document.head.appendChild(script);
  await new Promise(r => script.onload = r);
  const XLSX = window.XLSX;
  ```
- **Local Reading**: Use `await runner.readLocalFile(download.url)` to get the content into your script scope.

## 4. Troubleshooting Checklist

- **"Internal Server Error"**: Check the Relay Server logs. Usually means a malformed script or a timeout.
- **"No tab with given id"**: Happens if the tab was closed or the extension reloaded. Restart the preview.
- **Empty Items**: Check if the site uses a Shadow DOM. If so, use `runner.getByTextDeep` or `element.shadowRoot`.
- **Timeouts**: For heavy pages (like Amundi or Amazon), increase the `timeout` in the crawler config or add long `setTimeout` calls in the script.
