# tgweb-deno

Telegram 官方 WEB 代理（td web-proxy）后端，跑在 **Deno Deploy** 上。
把 Telegram 客户端（Desktop / Android）内嵌 webview 发起的 WebSocket lanes，
直接转接到 Telegram Web K 自己的 KWS 通道（`kws{dc}.web.telegram.org/apiws`）。

改编自 ToiCF/CF-Workers-TGProxy（Cloudflare Workers 版），把
`WebSocketPair` 换成 Deno 的 `Deno.upgradeWebSocket`，其余逻辑不变：
bridge 鉴权、bootstrap/session 令牌、多 lane WS 复用、AES-CTR
obfuscation 重签、自定义帧、流控。

## 部署（Deno Deploy，免费，不用绑卡）

1. 打开 https://dash.deno.com/new ，用 GitHub 登录，选这个仓库建项目。
2. 入口文件填 `main.js`。
3. 加两个环境变量（Environment Variables）：
   - `SECRET`：代理密钥，32 位 hex（`openssl rand -hex 16` 生成）
   - `BOOT`：bootstrap 密钥，任意 ≥16 字符随机串（`openssl rand -hex 24` 生成）
4. 点 Deploy。部署完会得到一个地址，例如 `https://tgweb-deno.deno.dev`。

## 使用

在 Telegram 里添加 WEB 类型代理：

- 服务器：`tgweb-deno.deno.dev`（换成你自己的部署地址，去掉 `https://`）
- 密钥：`SECRET` 的值

一键链接：`tg://webproxy?server=tgweb-deno.deno.dev&secret=<SECRET>`

手机：设置 → 数据和存储 → 代理 → 添加代理 → WEB。
电脑：设置 → 高级 → 连接类型 → 添加代理 → WEB。

## 自测

```sh
SECRET=... BOOT=... deno run --allow-net --allow-env main.js
curl http://127.0.0.1:8000/health   # 应返回 {"ok":true}
```

## 注意

- Deno Deploy 免费档不用绑卡，但未验证组织的额度是受限版（请求数/流量比完整版低），一个人用 Telegram 够了；如果提示额度用完，再绑卡解锁完整免费额度（只验证，不扣费）。
- 应用闲置 20–30 秒后会缩到零，下次连接有 1–2 秒冷启动，Telegram 会自动重连。
- `SECRET` 就是代理的唯一凭证，谁拿到链接谁就能用，不要公开。
