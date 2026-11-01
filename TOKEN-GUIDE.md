# 获取 QuanTV Token

本连接器连接 https://aigc.quantv.com，生成内容会消耗该账号现有积分。不需要单独申请 API Key。

1. 在浏览器登录 https://aigc.quantv.com。
2. 打开开发者工具，Chrome/Edge 中选择 Application（应用）→ Local Storage（本地存储）→ `https://aigc.quantv.com`。
3. 复制 `token` 的完整值，在 WorkBuddy 的 QuanTV 连接器设置中填写 Access Token。
4. 建议同时复制 `refreshToken` 的完整值，填写 Refresh Token。没有此项时可以留空，但 Access Token 过期后需重新填写。
5. 保存并连接。先要求列出音色检查服务访问；明确要求生成图片或语音后才会消耗积分。

只在连接器设置表单中填写凭证，不要发送到聊天、截图、工单或 GitHub。不要填写账号密码，也不要使用其他网站或旧域名保存的 Token。

## 凭证存储与重连

WorkBuddy 将表单凭证保存在用户本机的 `~/.workbuddy` 目录。MCP 调用时将其发送到 QuanTV，不会发送给 GitHub；GitHub 仅提供程序下载。

如果填写了 Refresh Token，MCP 会在访问令牌过期时调用 QuanTV 的刷新接口，将刷新后的凭证保存在以下本机数据目录的独立子目录中，用于进程重启后恢复：

- macOS：`~/Library/Application Support/QuantVGenerationMCP/workbuddy/`
- Windows：`%LOCALAPPDATA%\QuantVGenerationMCP\workbuddy\`
- Linux：`~/.quantv-generation-mcp/workbuddy/`

子目录名是站点与初始凭证的 SHA-256 摘要，不包含原始凭证。`credentials.json` 保存刷新后的凭证，macOS/Linux 上目录权限为 700、文件权限为 600；Windows 使用当前用户数据目录继承权限。这些文件属于敏感数据。

授权失效时，在 QuanTV 重新登录并复制最新凭证，更新连接器表单后重新连接。切换到另一组凭证会使用独立缓存与任务记录。当前版本不跨多个同时运行的 MCP 进程协调刷新，建议同一组凭证只保持一个连接器实例。

断开 WorkBuddy 连接不会自动删除 MCP 数据目录。要彻底清理本机凭证，先断开连接并从 WorkBuddy 删除已保存的表单凭证，再删除上述目录内各子目录的 `credentials.json`。保留 `submission-journal.json` 可保留防重复扣费记录；删除整个目录则会同时删除该记录。清理本机文件不等于在 QuanTV 服务端撤销授权。

## 常见情况

- **401 / 登录凭证无效**：更新 Token 后重连；不要把真实 Token 贴到对话里。
- **pending / processing**：任务还在执行，继续查询原任务 ID；不要重新提交生成。
- **响应丢失 / 超时**：不能据此判断未扣积分。按原参数重试可触发查重恢复，不要启用 `forceNew`。
- **程序无法安装**：检查 GitHub Release 下载地址的网络可达性，以及 WorkBuddy 是否已升级至 5.0.0 或以上。

本连接器当前提供图片生成、已有音色的文本转语音、模型和音色查询、任务查询；不提供视频生成或声音克隆。
