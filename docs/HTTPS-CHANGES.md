# HTTPS 修改记录

修改日期：2026-10-10。维护者：[haoyexueyin](https://github.com/haoyexueyin)。

## 原作者与版本来源

本修改版基于 [sganggs/Stronghold-Protocol](https://github.com/sganggs/Stronghold-Protocol) 的 [v0.2.2](https://github.com/sganggs/Stronghold-Protocol/releases/tag/v0.2.2)，通过 GitHub Fork 保留来源关系。游戏主体、玩法、战斗模拟、界面、游戏数据处理、原有 HTTP / WebSocket 联机系统及原文档均来自原作者和原项目贡献者。本次改动集中在 HTTPS 服务和启动配置，没有调整玩法与数值。

原项目的 [LICENSE](../LICENSE)、[NOTICE.md](../NOTICE.md)、[THIRD-PARTY-NOTICES.md](../THIRD-PARTY-NOTICES.md) 和原更新记录继续保留。新增代码和文档沿用原项目的 GPL-3.0-or-later 许可证；游戏素材仍归原权利人所有，并适用原项目的版权与非商业使用声明。

## 修改内容与文件

| 文件 | 改动 |
|---|---|
| `server/http/tls.js`（新增） | 统一选择 HTTPS / HTTP，读取本地或自有 PFX、PEM 证书；配置错误时停止启动 |
| `server/index.js` | 按证书配置创建 HTTP 或 HTTPS 服务器，共用静态文件、健康检查和游戏 WebSocket 路由 |
| `server/http/boot.js` | 启动横幅与 LAN 地址输出正确协议，HTTPS 模式提示樱花 TCP 映射配置 |
| `server/http/websocket.js` | 更新参数类型文档，明确现有 WebSocket 挂载支持 HTTPS 服务器 |
| `scripts/create-https-cert.ps1`（新增） | Windows 自动生成一年有效的本机自签名 PFX，复用未临近到期的证书，续签后提示重启；不安装受信任根证书 |
| `scripts/start-https-windows.bat`（新增） | Windows 双击准备证书并调用原启动脚本；自有证书优先 |
| `scripts/launch.mjs` | HTTPS 启动探测、浏览器打开和地址分享；识别旧服务协议不一致，提示更换证书后重启 |
| `tools/doctor.mjs` | 固定回环地址探测 HTTP / HTTPS 服务，正确输出 HTTPS 地址；自签名证书例外仅限本机探活 |
| `tools/setup.mjs` | 准备完成提示以实际启动窗口的 HTTP / HTTPS 地址为准 |
| `.gitignore` | 保留上游忽略规则，增加本机证书 `certs/` 与临时目录 `temp/` |
| `README.md`、`CHANGELOG.md`、`docs/DEPLOY.md`、本文件 | 原作者署名、本次修改范围、HTTPS 使用方法和樱花映射说明 |

客户端原本已按页面协议选择 `ws://` 或 `wss://`，本次沿用该能力，没有重写联机协议。

## 使用方法

从本 Fork 安装源码后，按原版说明安装 Node.js 和依赖、准备素材，再在 Windows 双击 `scripts\start-https-windows.bat`。本地地址为 `https://localhost:3000`。证书只在服务器启动时读取，切换协议或更换 / 续签证书前请停止旧服务。

已安装原作者 v0.2.2 完整包的用户，可按上表的路径覆盖本 Fork 对应文件，保留已有的素材和依赖；不要将本机 `certs/` 私钥目录复制到公开仓库，也不要覆盖游戏数据。原作者发布的完整包不包含这些 HTTPS 改动。

樱花采用 TCP 隧道：本地 IP `127.0.0.1`、端口 `3000`，关闭「自动 HTTPS」，朋友使用 `https://<远程地址>:<远程端口>`。访问时请明确输入 `https://`；输入 HTTP 会被部分樱花节点拦截。隧道配置依据见 [樱花官方说明](https://doc.natfrp.com/frpc/auto-https.html)。

自签名证书可加密连接，但浏览器会提示证书不受信任，公网地址与证书名称也可能不匹配。希望访问没有提示时，配置与公网域名匹配的受信任证书。环境变量、HTTP 回退及反向代理配置详见 [部署指南](DEPLOY.md#本地-https--樱花映射)。

## 已完成的验证

- 在 Windows 本地实际验证 HTTPS 健康接口和游戏首页。
- 通过 WSS 发送游戏 `hello` 并收到 `welcome`。
- 验证证书路径不被静态服务暴露、错误证书配置停止启动，以及显式 HTTP 回退。
- 验证 Windows HTTPS 启动入口、证书复用、重复启动和协议不一致提示。
- 续签提示改动通过 Node 语法检查及 PowerShell 解析检查，并完成独立安全审查。

上述结果来自本地验证；没有将樱花公网隧道连通性或全量游戏回归视为已验证结果。
