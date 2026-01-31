# macOS 构建所需的 GitHub 环境变量配置指南

> **快速参考：** 如果你需要快速查看所需的环境变量列表，请参阅 [快速参考指南](./BUILD_ENV_MACOS_QUICK_REF.md)

本文档详细说明了在 GitHub Actions 中进行 macOS 构建时需要配置的所有环境变量（Secrets），以及如何获取这些变量的值。

## 概述

macOS 应用程序的构建和分发需要苹果开发者账号提供的证书和配置文件。以下环境变量用于应用程序的代码签名和公证过程。

## 必需的环境变量

### 1. TAURI_PRIVATE_KEY

**用途：** Tauri 应用更新签名的私钥

**如何获取：**
```bash
# 使用 Tauri CLI 生成密钥对
pnpm tauri signer generate -w ~/.tauri/myapp.key

# 或者使用以下命令
npm install -g @tauri-apps/cli
tauri signer generate -w ~/.tauri/myapp.key
```

生成后会得到：
- 私钥（用于 `TAURI_PRIVATE_KEY`）
- 公钥（需要配置到 `tauri.conf.json` 中）

**配置位置：** GitHub Repository → Settings → Secrets and variables → Actions → New repository secret

**注意事项：**
- 私钥必须妥善保管，不要泄露
- 私钥内容是一个长字符串，包含整个密钥内容

---

### 2. TAURI_KEY_PASSWORD

**用途：** Tauri 私钥的密码

**如何获取：**
在生成 `TAURI_PRIVATE_KEY` 时，如果设置了密码，则使用该密码。如果没有设置密码，此变量可以留空或设置为空字符串。

**配置位置：** GitHub Repository → Settings → Secrets and variables → Actions → New repository secret

---

### 3. APPLE_CERTIFICATE

**用途：** Apple 开发者证书（P12 格式，Base64 编码）

**如何获取：**

1. **在 Apple 开发者中心创建证书：**
   - 访问 [Apple Developer](https://developer.apple.com/account/)
   - 进入 Certificates, Identifiers & Profiles
   - 点击 "+" 创建新证书
   - 选择 "Developer ID Application" （用于在 Mac App Store 外分发）
   - 按照提示完成证书请求（需要使用 macOS 的钥匙串访问工具创建 CSR）

2. **下载并导出证书：**
   - 下载创建好的证书（.cer 文件）
   - 双击导入到钥匙串访问
   - 在钥匙串访问中找到该证书
   - 右键点击证书，选择"导出"
   - 选择文件格式为 `.p12`
   - 设置导出密码（这个密码将用于 `APPLE_CERTIFICATE_PASSWORD`）

3. **转换为 Base64：**
   ```bash
   # 将 .p12 文件转换为 Base64 编码
   base64 -i certificate.p12 | pbcopy
   # 现在 Base64 编码的内容已复制到剪贴板
   
   # 或者保存到文件
   base64 -i certificate.p12 -o certificate.b64
   ```

**配置位置：** GitHub Repository → Settings → Secrets and variables → Actions → New repository secret

**注意事项：**
- 证书有效期通常为 1-2 年，过期后需要重新生成
- 确保使用的是 "Developer ID Application" 证书，不是其他类型的证书

---

### 4. APPLE_CERTIFICATE_PASSWORD

**用途：** 导出 P12 证书时设置的密码

**如何获取：**
这是在导出 `.p12` 证书文件时设置的密码。

**配置位置：** GitHub Repository → Settings → Secrets and variables → Actions → New repository secret

---

### 5. APPLE_SIGNING_IDENTITY

**用途：** Apple 签名身份标识符

**如何获取：**
这是证书的通用名称（Common Name），格式通常为：`Developer ID Application: Your Name (Team ID)`

查看方法：
```bash
# 方法 1：查看证书详情
# 在钥匙串访问中双击证书，查看"通用名称"字段

# 方法 2：使用命令行
security find-identity -v -p codesigning
```

示例输出：
```
1) XXXXXX "Developer ID Application: YourName (TEAM123456)"
```

使用引号内的完整字符串作为 `APPLE_SIGNING_IDENTITY` 的值。

**配置位置：** GitHub Repository → Settings → Secrets and variables → Actions → New repository secret

---

### 6. APPLE_ID

**用途：** Apple 开发者账号的 Apple ID（电子邮件地址）

**如何获取：**
这就是你的 Apple 开发者账号登录邮箱。

**配置位置：** GitHub Repository → Settings → Secrets and variables → Actions → New repository secret

---

### 7. APPLE_PASSWORD

**用途：** App 专用密码（App-Specific Password）

**如何获取：**

1. 访问 [Apple ID 账户页面](https://appleid.apple.com/)
2. 登录你的 Apple ID
3. 在"安全"部分，找到"App 专用密码"
4. 点击"生成密码"
5. 输入一个标签（例如："GitHub Actions"）
6. 复制生成的密码

**配置位置：** GitHub Repository → Settings → Secrets and variables → Actions → New repository secret

**注意事项：**
- 这不是你的 Apple ID 密码
- App 专用密码用于第三方应用访问 Apple 服务
- 密码生成后只显示一次，请妥善保管
- 如果启用了双重认证，才能生成 App 专用密码

---

### 8. APPLE_TEAM_ID

**用途：** Apple 开发者团队 ID

**如何获取：**

1. 访问 [Apple Developer](https://developer.apple.com/account/)
2. 登录后在页面顶部或"Membership"页面可以找到
3. Team ID 是一个 10 位字符的字符串（例如：ABC1234567）

**配置位置：** GitHub Repository → Settings → Secrets and variables → Actions → New repository secret

---

## 配置步骤总结

1. **注册 Apple 开发者账号**
   - 需要付费（个人：$99/年，企业：$299/年）
   - 访问：https://developer.apple.com/programs/

2. **创建和导出证书**
   - 在 Apple 开发者中心创建 "Developer ID Application" 证书
   - 导出为 .p12 格式
   - 转换为 Base64 编码

3. **生成 Tauri 签名密钥**
   ```bash
   pnpm tauri signer generate -w ~/.tauri/myapp.key
   ```

4. **创建 App 专用密码**
   - 在 Apple ID 账户页面生成

5. **在 GitHub 配置所有 Secrets**
   - 进入 Repository → Settings → Secrets and variables → Actions
   - 添加以下 8 个 secrets：
     - `TAURI_PRIVATE_KEY`
     - `TAURI_KEY_PASSWORD`
     - `APPLE_CERTIFICATE`
     - `APPLE_CERTIFICATE_PASSWORD`
     - `APPLE_SIGNING_IDENTITY`
     - `APPLE_ID`
     - `APPLE_PASSWORD`
     - `APPLE_TEAM_ID`

## 验证配置

配置完成后，可以通过以下方式验证：

1. **手动触发工作流：**
   - 访问 Repository → Actions
   - 选择相关的工作流（如 `release.yml` 或 `alpha.yml`）
   - 点击 "Run workflow"

2. **查看构建日志：**
   - 检查 macOS 构建任务的日志
   - 确认签名和公证步骤成功完成

## 常见问题

### Q: 构建失败，提示 "No signing identity found"
**A:** 检查 `APPLE_SIGNING_IDENTITY` 是否配置正确，确保与证书的通用名称完全匹配。

### Q: 公证失败，提示认证错误
**A:** 
- 确认 `APPLE_ID` 是否正确
- 确认 `APPLE_PASSWORD` 使用的是 App 专用密码，不是 Apple ID 密码
- 确认 Apple ID 已启用双重认证

### Q: 证书过期了怎么办？
**A:** 
- 在 Apple 开发者中心撤销旧证书
- 创建新证书
- 重新导出并转换为 Base64
- 更新 GitHub Secrets 中的 `APPLE_CERTIFICATE` 和相关密码

### Q: Team ID 在哪里找？
**A:** 登录 Apple Developer 网站，在 Membership 页面或页面顶部可以看到 Team ID。

## 安全建议

1. **定期更新密码和证书**
2. **不要在代码中硬编码任何密钥或密码**
3. **限制对 GitHub Secrets 的访问权限**
4. **定期检查 GitHub Actions 日志，确保没有泄露敏感信息**
5. **如果怀疑密钥泄露，立即在 Apple Developer 中撤销证书并重新生成**

## 参考链接

- [Apple Developer Program](https://developer.apple.com/programs/)
- [Tauri Documentation](https://tauri.app/)
- [GitHub Actions Secrets](https://docs.github.com/en/actions/security-guides/encrypted-secrets)
- [Apple Developer Documentation](https://developer.apple.com/documentation/)
- [App-Specific Passwords](https://support.apple.com/en-us/HT204397)

## 联系支持

如果在配置过程中遇到问题，可以：
- 在 GitHub 仓库提交 Issue
- 加入 TG 频道：[@clash_verge_rev](https://t.me/clash_verge_re)
