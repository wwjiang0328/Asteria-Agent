# HermesX

本地智能工作台 — 把 AI Agent 装进你的电脑。支持 Windows 和 macOS（Apple Silicon）。

> 下载地址：[hermesx.jackcloud.online](https://hermesx.jackcloud.online)

---

## 更新日志

### v1.0.0（2026-06-19）

- 正式版首发 — 从 0.x 跨越到 1.0.0
- 浏览器启动安全加固 — 白名单校验，拒绝非浏览器路径
- 更新下载 SHA256 校验 — 安装包完整性与防篡改
- SSRF 防御深化 — DNS 解析后 IP 校验，阻断重绑定攻击
- 缩减权限面 — 移除 fs:default 能力声明
