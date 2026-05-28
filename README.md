# 🛡️ TrapDoor 安全自查脚本

> 基于 [Socket Security《TrapDoor Supply Chain Attack》报告（2026.05.25）](https://socket.dev) 编写的开发环境安全自查工具。
> 帮助 Web3 / AI 开发者快速检测本机和 Docker 容器是否受到 TrapDoor 供应链攻击影响。

---

## ⚡ 快速开始

```bash
# 宿主机检查
chmod +x trapdoor_check && ./trapdoor_check

# Docker 容器检查（需要 docker 权限）
chmod +x trapdoor_docker_check && ./trapdoor_docker_check
# 如提示权限拒绝：
sudo ./trapdoor_docker_check
```

---

## 📋 脚本说明

### `trapdoor_check` — 宿主机安全自查

在开发者本机（Linux）运行，依次扫描 9 个维度：

| # | 检查项 | 说明 |
|---|--------|------|
| 1 | 恶意 npm 包 | 检查 `wallet-security-checker` 等 9 个已知 TrapDoor npm 包 |
| 2 | 恶意 PyPI 包 | 检查 `eth-security-auditor` 等 4 个已知恶意 Python 包 |
| 3 | 恶意 Cargo 包 | 检查 `sui-move-build-helper` 等针对 Sui/Move 开发者的恶意包 |
| 4 | AI 配置文件完整性 | 检测 `.cursorrules`、`CLAUDE.md`、`AGENTS.md` 中是否含隐藏控制字符 |
| 5 | SSH 安全 | 检查 `authorized_keys` 是否存在陌生公钥，私钥是否有密码保护 |
| 6 | 云端凭证 | 检查 AWS / GCP / Azure / Docker Registry 凭证文件是否存在 |
| 7 | 浏览器钱包扩展 | 检查 MetaMask / Rabby / OKX / Binance Web3 / Phantom / TronLink 数据目录 |
| 8 | CLI 开发者钱包 | 检查 Geth keystore、Foundry、Hardhat 明文私钥、Solana、Sui、Aptos、TRON |
| 9 | 环境变量与配置 | 检查当前环境变量、`.env` 文件、shell 配置中的明文凭证及可疑 systemd 服务 |

### `trapdoor_docker_check` — Docker 容器安全自查

在宿主机运行，通过 Docker API 自动遍历所有运行中的容器：

| 检查方式 | 检查内容 |
|----------|----------|
| 容器内扫描 | 在容器内执行包管理器命令，检查 PyPI / npm / Cargo 恶意包 |
| 容器内扫描 | 查找容器内被篡改的 AI 配置文件（含隐藏字符检测） |
| 挂载目录扫描 | 检查宿主机挂载目录中的 `.env` 明文凭证 |
| 挂载目录扫描 | 检测挂载的浏览器钱包扩展数据（MetaMask / OKX / Phantom 等 6 种） |
| 挂载目录扫描 | 检测挂载的 EVM keystore / Foundry / Hardhat / Solana / Sui / TRON 私钥文件 |

---

## ✅ 安全性说明

**这两个脚本不会对你的系统做任何修改。**

| 行为 | 状态 |
|------|------|
| 读取文件内容（私钥、助记词） | ❌ 不会 |
| 修改任何文件或配置 | ❌ 不会 |
| 发起网络请求或上传数据 | ❌ 不会 |
| 安装任何软件包或依赖 | ❌ 不会 |
| 打印敏感值（会替换为 `***已隐藏***`） | ❌ 不会 |
| 仅使用系统标准命令（find/grep/awk/du） | ✅ 是 |
| 源码无混淆，可直接阅读 | ✅ 是 |

> 如有疑虑，执行前请用 `cat trapdoor_check` 或 `less trapdoor_check` 审阅完整源码。

---

## 🔍 如何解读结果

```
✅ PASS  — 正常，未发现风险
⚠️  WARN  — 需人工确认（发现相关文件，但不一定是威胁）
❌ FAIL  — 发现明确风险，需立即处理
```

**关于 WARN 项**：WARN 不等于感染。例如，脚本发现 `~/.ethereum/keystore` 存在时会标记 WARN，含义是"这里有钱包文件，属于 TrapDoor 的攻击目标，请确认它的访问权限和完整性"，而非"你已被攻击"。**只有 FAIL 才需要立即响应。**

---

## 🖥️ 环境要求

| 项目 | 要求 |
|------|------|
| 操作系统 | Linux（bash 4+） |
| 必要命令 | `find` `grep` `awk` `du` `systemctl` |
| Docker 脚本额外需要 | `docker`（daemon 正在运行） |
| 网络访问 | 不需要 |
| root 权限 | 不需要（Docker 脚本在无 docker 组权限时需要 `sudo`） |
| macOS | 暂不支持 |

---

## 📦 TrapDoor 检测的恶意包列表

### npm
```
wallet-security-checker      defi-threat-scanner
token-usage-tracker          prompt-engineering-toolkit
llm-context-compressor       web3-secrets-detector
solidity-deploy-guard        dev-env-bootstrapper
defi-env-auditor
```

### PyPI
```
eth-security-auditor         cryptowallet-safety
defi-risk-scanner            mnemonic-safety-check
```

### Cargo
```
sui-move-build-helper        move-compiler-tools
```

---

## 🚨 发现 FAIL 项后的处置流程

```bash
# 1. 卸载恶意 npm 包
npm uninstall <包名>

# 2. 卸载恶意 PyPI 包
pip uninstall <包名>

# 3. 立即轮换所有可能泄露的凭证
#    - API 密钥（AWS / GCP / npm token 等）
#    - 如钱包私钥所在目录有异常访问记录，考虑转移资产

# 4. 检查 git 提交历史，确认无敏感信息被提交
git log --all --full-history -- "*.env"
git log --all --full-history -- "hardhat.config.*"

# 5. 删除被篡改的 AI 配置文件
rm ~/.cursorrules ~/.claude/CLAUDE.md  # 视实际路径调整
```

---

## 📚 参考资料

- [Socket Security — TrapDoor Supply Chain Attack Report（2026.05.25）](https://socket.dev)
- [Geth Keystore 文档](https://geth.ethereum.org/docs/fundamentals/account-management)
- [Foundry cast wallet 文档](https://book.getfoundry.sh/reference/cast/cast-wallet)
- [Hardhat 配置变量（安全存储私钥）](https://hardhat.org/hardhat-runner/docs/guides/configuration-variables)

---

## 📄 License

MIT
