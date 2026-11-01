# QuantV 图片与语音 MCP

用于 WorkBuddy 的本地 MCP 连接器，连接 https://aigc.quantv.com，支持图片生成、参考图、KFish 文本转语音、音色查询和任务结果链接。

使用自己的 QuantV 账号及积分。无需专用 API Key，也不使用 OAuth。Token 只在本机配置，业务请求直接发送到 QuantV。

## WorkBuddy 使用

从本仓库 Release 下载 `QuantV-WorkBuddy-Connector-0.2.6.zip`，供 WorkBuddy 开放平台连接器审核提交使用。市场用户安装连接器后，在配置表单中填写自己的凭证。详见 [获取 Token 与凭证清理](./TOKEN-GUIDE.md)。

连接器要求 WorkBuddy 5.0.0 或以上，使用其托管的 Node.js 20，无需用户自行安装 Node.js。安装时从固定版本 GitHub Release 下载无外部依赖的程序包。

WorkBuddy 模式仅提供异步生成和任务查询：提交一次后返回任务 ID，再查询结果。暂时性网络故障会触发查重恢复，避免盲目重发生成请求。实际费用、支持模型和积分规则以 QuantV 网站为准。

## 其他 MCP 客户端

需要 Node.js 20 或以上，设置 `QUANTV_ACCESS_TOKEN` 和可选的 `QUANTV_REFRESH_TOKEN`，运行本仓库的 `server.mjs`。设置 `QUANTV_CLIENT_PROFILE=workbuddy` 可启用异步调用与刷新凭证本地缓存。

## 发布内容与许可

`server.mjs` 为打包后的服务，运行时不需要额外下载 npm 依赖。程序包不包含任何真实用户凭证。第三方依赖许可见 `THIRD_PARTY_NOTICES.txt`。QuantV 自有代码未授予开源再分发许可（UNLICENSED）。
