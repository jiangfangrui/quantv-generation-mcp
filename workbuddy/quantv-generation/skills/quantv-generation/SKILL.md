---
name: quantv-generation
display_name: QuantV 图片与语音
display_name_en: QuantV Image & Speech
description: 使用 QuantV 生成图片、参考图改图、文本转语音、选择音色，查询生成任务并获取作品链接。
description_zh: 使用 QuantV 账号积分生成图片和语音，查询任务与结果链接。
description_en: Generate images and speech with QuantV account credits, select voices, and retrieve task results.
version: 0.2.6
author: QuantV
examples_zh:
  - "生成一张 16:9 的咖啡新品海报，不要文字"
  - "先列出音色，再将文案生成语音"
examples_en:
  - "Generate a 16:9 coffee launch poster without text"
  - "List voices, then turn my text into speech"
---

# QuantV 图片与语音

需要用户已在连接器设置中填写自己的 Token。用户未明确要求生成时，查询模型、音色或任务即可，不发起扣积分的生成。不要要求用户在对话中发送 Token、密码，不读取其浏览器凭证。

## 调用顺序

- 图片：`quantv_generate_image` 提交一次，保存返回的 `id`；随后用 `quantv_get_task` 查询。
- 语音：先 `quantv_list_voices` 选择音色，使用条目的 `referenceId`（不是 `id`）调用 `quantv_generate_speech`，再查询任务。
- 本连接器生成工具的 `wait` 仅允许 `false`，缺省也是 `false`。不调用 `quantv_wait_task`，该工具在此版本不提供。
- 查询若为 `pending` 或 `processing`，说明任务仍在执行。可间隔约 5 秒查询，单次对话最多连续查询 6 次；仍未完成就告知任务 ID，用户继续查询时沿用它。
- `completed` 时提供 `resultUrls` 中的作品链接；`failed` 或 `refunded` 时说明状态和返回错误，不自动重新生成。
- 图片和语音均使用用户现有积分；当前价格以 QuantV 网站显示为准，不承诺固定费用。

## 工具与参数

### quantv_list_models
无参数，返回网站公开模型及动态参数配置。列出模型不表示此连接器支持所有媒体生成；当前生成工具仅覆盖图片和文本转语音。

### quantv_list_voices
可选 `modelId: string`、`officialOnly: boolean`。返回可用音色集合，生成语音必须使用音色的 `referenceId`。

### quantv_generate_image
必填 `prompt: string`。可选 `ratio` 默认 `auto`，支持 `1:1`、`16:9`、`9:16`、`3:2`、`2:3`、`5:4`、`4:5`、`4:3`、`3:4`、`21:9`、`9:21`、`2:1`、`1:2`；`resolution` 为 `1K`（默认）、`2K`、`4K`；`imageUrls: string[]` 最多 12 个可访问的图片 URL，默认空；`modelId` 可选，默认配置的图片模型。`wait: false`。

示例：`{"prompt":"一杯咖啡，暖色摄影，无文字","ratio":"16:9","resolution":"1K","wait":false}`。

### quantv_generate_speech
必填 `text: string`、`referenceId: string`。可选 `speed` 为 `0.5`、`1`（默认）、`2`；`volume` 为 `-1`、`0`（默认）、`1`；`modelId` 默认配置的语音模型。`wait: false`。

示例：`{"text":"欢迎来到米核 AI","referenceId":"从音色工具取得的真实值","speed":1,"volume":0,"wait":false}`。

### quantv_get_task
必填 `id: string`。生成工具返回任务对象，如 `{"id":"...","status":"processing"}`；本工具返回任务对象并附带 `resultUrls: string[]`，未完成时链接可能为空。

## 错误与重复扣费

生成返回任务 ID 后，只查询该 ID。网络错误、超时、响应未知均不能成为再次扣费的理由；没有 ID 时可按原参数再次调用以触发服务的查重恢复，但不得增加 `forceNew`。

生成工具的可选 `forceNew` 只接受 `CONFIRM_NEW_CHARGE`，仅当用户明确要求再生成一份全新结果并接受再次消耗积分时使用。

遇到授权失效、401 或刷新失败，引导用户在连接器设置中更新 Token 后重连；不要索取凭证文本。参数错误时根据返回信息修正，仍需保留已有任务 ID。任务尚未完成时不得编造作品或下载地址。
