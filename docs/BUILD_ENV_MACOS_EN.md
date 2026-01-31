# macOS Build Environment Variables Configuration Guide

> **Quick Reference:** For a quick overview of required environment variables, see the [Quick Reference Guide](./BUILD_ENV_MACOS_QUICK_REF.md)

This document provides detailed instructions on configuring all required environment variables (Secrets) for macOS builds in GitHub Actions and how to obtain their values.

## Overview

Building and distributing macOS applications requires certificates and provisioning profiles from an Apple Developer account. The following environment variables are used for code signing and notarization.

## Required Environment Variables

### 1. TAURI_PRIVATE_KEY

**Purpose:** Private key for Tauri app update signing

**How to Obtain:**
```bash
# Generate key pair using Tauri CLI
pnpm tauri signer generate -w ~/.tauri/myapp.key

# Or use this command
npm install -g @tauri-apps/cli
tauri signer generate -w ~/.tauri/myapp.key
```

This will generate:
- Private key (for `TAURI_PRIVATE_KEY`)
- Public key (configure in `tauri.conf.json`)

**Configuration Location:** GitHub Repository → Settings → Secrets and variables → Actions → New repository secret

**Notes:**
- Keep the private key secure and never expose it
- The private key content is a long string containing the entire key

---

### 2. TAURI_KEY_PASSWORD

**Purpose:** Password for the Tauri private key

**How to Obtain:**
If you set a password when generating `TAURI_PRIVATE_KEY`, use that password. If no password was set, this variable can be left empty or set to an empty string.

**Configuration Location:** GitHub Repository → Settings → Secrets and variables → Actions → New repository secret

---

### 3. APPLE_CERTIFICATE

**Purpose:** Apple Developer Certificate (P12 format, Base64 encoded)

**How to Obtain:**

1. **Create Certificate in Apple Developer Center:**
   - Visit [Apple Developer](https://developer.apple.com/account/)
   - Go to Certificates, Identifiers & Profiles
   - Click "+" to create a new certificate
   - Select "Developer ID Application" (for distribution outside Mac App Store)
   - Follow the prompts to complete the certificate request (requires CSR from macOS Keychain Access)

2. **Download and Export Certificate:**
   - Download the created certificate (.cer file)
   - Double-click to import into Keychain Access
   - Find the certificate in Keychain Access
   - Right-click the certificate and select "Export"
   - Choose `.p12` file format
   - Set an export password (this will be used for `APPLE_CERTIFICATE_PASSWORD`)

3. **Convert to Base64:**
   ```bash
   # Convert .p12 file to Base64 encoding
   base64 -i certificate.p12 | pbcopy
   # Base64 encoded content is now copied to clipboard
   
   # Or save to file
   base64 -i certificate.p12 -o certificate.b64
   ```

**Configuration Location:** GitHub Repository → Settings → Secrets and variables → Actions → New repository secret

**Notes:**
- Certificates typically expire after 1-2 years and need regeneration
- Ensure you're using "Developer ID Application" certificate, not other types

---

### 4. APPLE_CERTIFICATE_PASSWORD

**Purpose:** Password set when exporting the P12 certificate

**How to Obtain:**
This is the password you set when exporting the `.p12` certificate file.

**Configuration Location:** GitHub Repository → Settings → Secrets and variables → Actions → New repository secret

---

### 5. APPLE_SIGNING_IDENTITY

**Purpose:** Apple signing identity identifier

**How to Obtain:**
This is the certificate's Common Name, typically formatted as: `Developer ID Application: Your Name (Team ID)`

View methods:
```bash
# Method 1: View certificate details
# Double-click certificate in Keychain Access, check "Common Name" field

# Method 2: Use command line
security find-identity -v -p codesigning
```

Example output:
```
1) XXXXXX "Developer ID Application: YourName (TEAM123456)"
```

Use the complete string within quotes as the `APPLE_SIGNING_IDENTITY` value.

**Configuration Location:** GitHub Repository → Settings → Secrets and variables → Actions → New repository secret

---

### 6. APPLE_ID

**Purpose:** Apple Developer account Apple ID (email address)

**How to Obtain:**
This is your Apple Developer account login email.

**Configuration Location:** GitHub Repository → Settings → Secrets and variables → Actions → New repository secret

---

### 7. APPLE_PASSWORD

**Purpose:** App-Specific Password

**How to Obtain:**

1. Visit [Apple ID Account Page](https://appleid.apple.com/)
2. Sign in with your Apple ID
3. In the "Security" section, find "App-Specific Passwords"
4. Click "Generate Password"
5. Enter a label (e.g., "GitHub Actions")
6. Copy the generated password

**Configuration Location:** GitHub Repository → Settings → Secrets and variables → Actions → New repository secret

**Notes:**
- This is NOT your Apple ID password
- App-Specific Passwords are for third-party app access to Apple services
- Password is shown only once after generation, keep it safe
- Two-factor authentication must be enabled to generate App-Specific Passwords

---

### 8. APPLE_TEAM_ID

**Purpose:** Apple Developer Team ID

**How to Obtain:**

1. Visit [Apple Developer](https://developer.apple.com/account/)
2. After signing in, find it at the top of the page or on the "Membership" page
3. Team ID is a 10-character string (e.g., ABC1234567)

**Configuration Location:** GitHub Repository → Settings → Secrets and variables → Actions → New repository secret

---

## Configuration Steps Summary

1. **Register Apple Developer Account**
   - Requires payment (Individual: $99/year, Enterprise: $299/year)
   - Visit: https://developer.apple.com/programs/

2. **Create and Export Certificate**
   - Create "Developer ID Application" certificate in Apple Developer Center
   - Export as .p12 format
   - Convert to Base64 encoding

3. **Generate Tauri Signing Key**
   ```bash
   pnpm tauri signer generate -w ~/.tauri/myapp.key
   ```

4. **Create App-Specific Password**
   - Generate on Apple ID account page

5. **Configure All Secrets in GitHub**
   - Go to Repository → Settings → Secrets and variables → Actions
   - Add these 8 secrets:
     - `TAURI_PRIVATE_KEY`
     - `TAURI_KEY_PASSWORD`
     - `APPLE_CERTIFICATE`
     - `APPLE_CERTIFICATE_PASSWORD`
     - `APPLE_SIGNING_IDENTITY`
     - `APPLE_ID`
     - `APPLE_PASSWORD`
     - `APPLE_TEAM_ID`

## Verify Configuration

After configuration, verify by:

1. **Manually Trigger Workflow:**
   - Visit Repository → Actions
   - Select relevant workflow (e.g., `release.yml` or `alpha.yml`)
   - Click "Run workflow"

2. **Check Build Logs:**
   - Review macOS build task logs
   - Confirm signing and notarization steps complete successfully

## Common Issues

### Q: Build fails with "No signing identity found"
**A:** Check if `APPLE_SIGNING_IDENTITY` is configured correctly and matches the certificate's Common Name exactly.

### Q: Notarization fails with authentication error
**A:** 
- Verify `APPLE_ID` is correct
- Confirm `APPLE_PASSWORD` uses App-Specific Password, not Apple ID password
- Verify Apple ID has two-factor authentication enabled

### Q: What to do when certificate expires?
**A:** 
- Revoke old certificate in Apple Developer Center
- Create new certificate
- Re-export and convert to Base64
- Update `APPLE_CERTIFICATE` and related password in GitHub Secrets

### Q: Where to find Team ID?
**A:** Sign in to Apple Developer website, Team ID is visible on the Membership page or at the top of the page.

## Security Recommendations

1. **Regularly update passwords and certificates**
2. **Never hardcode any keys or passwords in code**
3. **Limit access to GitHub Secrets**
4. **Regularly check GitHub Actions logs to ensure no sensitive information is leaked**
5. **If key compromise is suspected, immediately revoke certificate in Apple Developer and regenerate**

## Reference Links

- [Apple Developer Program](https://developer.apple.com/programs/)
- [Tauri Documentation](https://tauri.app/)
- [GitHub Actions Secrets](https://docs.github.com/en/actions/security-guides/encrypted-secrets)
- [Apple Developer Documentation](https://developer.apple.com/documentation/)
- [App-Specific Passwords](https://support.apple.com/en-us/HT204397)

## Support

If you encounter issues during configuration:
- Submit an issue on the GitHub repository
- Join TG channel: [@clash_verge_rev](https://t.me/clash_verge_re)
