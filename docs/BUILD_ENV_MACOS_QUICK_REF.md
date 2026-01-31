# macOS 构建环境变量快速参考

本文档提供 macOS GitHub Actions 构建所需环境变量的快速参考。详细说明请参阅 [BUILD_ENV_MACOS.md](./BUILD_ENV_MACOS.md)。

## 必需的 8 个 GitHub Secrets

| 变量名 | 用途 | 获取方式 |
|--------|------|----------|
| `TAURI_PRIVATE_KEY` | Tauri 应用更新签名私钥 | 运行 `pnpm tauri signer generate` |
| `TAURI_KEY_PASSWORD` | Tauri 私钥密码 | 生成私钥时设置的密码（可选） |
| `APPLE_CERTIFICATE` | Apple 开发者证书 (P12, Base64) | 从 Apple Developer 导出证书并转换为 Base64 |
| `APPLE_CERTIFICATE_PASSWORD` | P12 证书导出密码 | 导出证书时设置的密码 |
| `APPLE_SIGNING_IDENTITY` | Apple 签名身份 | 证书的通用名称，如 "Developer ID Application: YourName (TeamID)" |
| `APPLE_ID` | Apple Developer 账号邮箱 | 你的 Apple ID |
| `APPLE_PASSWORD` | App 专用密码 | 在 Apple ID 账户页面生成 |
| `APPLE_TEAM_ID` | Apple Developer Team ID | 在 Apple Developer Membership 页面查看 |

## 快速配置步骤

### 1. 准备 Apple Developer 账号
- 注册并付费 Apple Developer Program ($99/年)
- 访问：https://developer.apple.com/programs/

### 2. 创建和配置证书

```bash
# 在 Apple Developer 中心创建 "Developer ID Application" 证书
# 下载并导入到钥匙串
# 导出为 .p12 格式
# 转换为 Base64
base64 -i certificate.p12 | pbcopy
```

### 3. 生成 Tauri 签名密钥

```bash
pnpm tauri signer generate -w ~/.tauri/myapp.key
```

### 4. 生成 App 专用密码
- 访问 https://appleid.apple.com/
- 在"安全"部分生成 App 专用密码

### 5. 在 GitHub 配置 Secrets
前往：`Repository → Settings → Secrets and variables → Actions`

添加上述 8 个 secrets。

## 快速验证

```bash
# 查看本地签名身份
security find-identity -v -p codesigning

# 测试证书
security find-certificate -c "Developer ID Application" -p
```

## 常见错误

| 错误信息 | 解决方案 |
|----------|----------|
| "No signing identity found" | 检查 `APPLE_SIGNING_IDENTITY` 是否与证书名称完全匹配 |
| "Authentication failed" | 确认使用的是 App 专用密码，不是 Apple ID 密码 |
| "Certificate expired" | 在 Apple Developer 重新创建证书并更新 GitHub Secret |

## 需要帮助？

详细文档：
- [中文完整指南](./BUILD_ENV_MACOS.md)
- [English Guide](./BUILD_ENV_MACOS_EN.md)

社区支持：
- [GitHub Issues](https://github.com/clash-verge-rev/clash-verge-rev/issues)
- [TG 频道](https://t.me/clash_verge_re)
