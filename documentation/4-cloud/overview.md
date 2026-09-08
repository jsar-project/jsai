# Cloud 能力概览

AIUI Cloud 为第三方系统提供连接 Rokid 云服务和 Glasses 设备的服务端集成能力。业务服务、定时任务和外部 Agent 可以通过云端接口查询账号信息、暂存 AIUI 消息，或向用户的 Glasses 下发通知。

这些能力只能在可信服务端环境中使用，不属于 AIUI 页面通过内置 `open` 模块调用的 OpenAPI。请勿将账号 `access_token`、Rokid 账号 SK 或第三方 token 放入页面代码、客户端包、源码仓库或日志。

## 能力选择

| 需求 | 接口 | 文档 |
|---|---|---|
| 使用账号 token 获取账号信息 | `GET /account/v1/token` | [账号 Token 查询](./account-token.md) |
| 为账号和 AIUI 智能体暂存一条消息 | `POST /metis/openApi/v1/cacheAIUIMessage` | [AIUI 消息缓存](./message-cache.md) |
| 向指定用户的 Glasses 下发通知或跳转页面 | `CloudIntegration.sendNotification()` 或云端 HTTP 接口 | [通知下发](./notifications.md) |

`getToken` 仅供平台官方智能体使用。`cacheAIUIMessage` 写入的消息由 AIUI 页面侧 `api.agent.getAIUICacheMessage()` 原子获取并删除。通知下发支持 Node.js SDK 和直接 HTTP 两种方式。

## 安装 npm 包

通知下发推荐使用 `@yodaos-pkg/cloud-integration`。该包面向 Node.js 20 及以上环境，并使用运行时内置的 `fetch`：

```bash
npm install @yodaos-pkg/cloud-integration
```

```js
import { CloudIntegration } from '@yodaos-pkg/cloud-integration'

const cloud = new CloudIntegration({
  token: process.env.ROKID_SK,
})
```

SK 是智能体所属账号的敏感凭证。请将它保存在服务端环境变量或密钥管理服务中。

## 安装 Cloud Integration Skill

`aiui-cloud-integration` Skill 为 AI 编码助手提供账号查询、消息缓存、通知下发、页面跳转和错误处理的服务端集成上下文：

```bash
npx skills add https://github.com/jsar-project/AIUI/tree/main/skills/aiui-cloud-integration
```

Skill 用于辅助开发和生成代码，不会代替凭证，也不会自动调用云端接口。
