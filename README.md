# agent-engine-release — AgentDev 商业版发行仓库

本仓库是 AgentDev 商业发行包（混淆构建 + Ed25519 签名 manifest）的 **git 分发面**，
结构与 R2 发布面同构（`rpo/publish-r2.mjs` 语义）：

```
releases/index.json                    版本索引（客户端 update 的版本发现读它）
releases/latest-<版本>.json            该版本的签名 manifest（pack 产物，不可变）
releases/<版本>/agentdev-<版本>.tgz    交付包（不可变）
releases/<版本>/agentdev-<版本>.tgz.sha256
```

当前版本：**2.0.0**

## 下载（三步之一）

- **网页**：在 GitHub 网页进入 `releases/2.0.0/` 下载 `agentdev-2.0.0.tgz`；
- **命令行（raw URL）**：
  ```
  curl -L -o agentdev-2.0.0.tgz https://raw.githubusercontent.com/arch-engine-new/agent-engine-release/master/releases/2.0.0/agentdev-2.0.0.tgz
  curl -L -o agentdev-2.0.0.tgz.sha256 https://raw.githubusercontent.com/arch-engine-new/agent-engine-release/master/releases/2.0.0/agentdev-2.0.0.tgz.sha256
  ```
  本仓库当前为 **Public**（raw 匿名可读，2026-10-02 实测 200）；网页下载需登录账号。
  若仓库转为 Private：raw 匿名访问会 404，命令行下载需认证
  （`curl -H "Authorization: Bearer <GitHub token>"`，token 需对该仓库有读权限）。
- **已装客户机**：直接用包内更新命令（见下），自动完成版本发现/验签/下载/安装。

## 安装（三步之二）

要求 Node.js >= 22（`npm ci` 需联网）。

1. 解包到客户项目根下一级目录（推荐 `<项目根>/agentdev/`）：
   ```
   mkdir -p <项目根>/agentdev && tar -xzf agentdev-2.0.0.tgz -C <项目根>/agentdev
   ```
2. 校验完整性（可选但推荐）：
   ```
   cd <项目根>/agentdev && sha256sum -c <(cat /path/to/agentdev-2.0.0.tgz.sha256)
   ```
3. 安装运行时依赖（包根执行）：`npm ci`

## 激活（三步之三）

1. 采集机器指纹，发给发布方签发 license：
   ```
   node agentdev-fingerprint.mjs        # stdout = 32-hex 指纹
   ```
2. 发布方按指纹签发 license（发证侧 `rpo/issue-licenses.mjs`），回传 license 文件；
3. 激活（双条件判据：激活成功 ∧ 复查 activated）：
   ```
   AGENTDEV_PROJECT_ROOT=<项目根> node agentdev-activate.mjs <license.json>
   ```

## 注册 MCP 服务

入口 `node <安装根>/mcp/src/index.js`，env `AGENTDEV_PROJECT_ROOT=<项目根>`；
模板见包内 `distribute/mcp-configs/commercial-node.mcp.json`。

## 在线更新（已装客户机）

```
node update.mjs --base-url=https://raw.githubusercontent.com/arch-engine-new/agent-engine-release/master/
```

流程：`releases/index.json` 版本发现 → manifest 验签（内嵌信任锚）→ 单调性校验 →
下载 + sha256 校验 → 备份 → 解包 → skills 重分发 → `npm ci` → 版本回写。
Private 仓库下 raw 下载需认证：设 env `AGENTDEV_RELEASE_AUTH_TOKEN=<GitHub token>`
（update 会作为 `Authorization: Bearer` 附在请求上）。
注意：push 后 raw CDN 有分钟级收敛延迟，若 sha256 校验失败请稍后重试
（fail-closed：校验不符拒装，不会装坏）。

详细说明见包内 `INSTALL.md`。
