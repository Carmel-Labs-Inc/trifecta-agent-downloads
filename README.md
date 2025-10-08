# trifecta-agent-downloads

Central distribution repository for Trifecta Agent releases across all platforms.

## Repository Structure

```
trifecta-agent-downloads/
├── manifest/
│   ├── latest.json          # Windows manifest (legacy)
│   └── latest-macos.json    # macOS manifest
└── .github/workflows/
    └── publish-from-windows.yml  # Legacy - no longer actively used
```

**Note:** Publishing is now done directly from build repos (trifecta-agent-windows and trifecta-native-agent-main) via tag-triggered workflows.

## Releases

### Windows
- **Tag Format:** `windows-v{major}.{minor}.{patch}-{description}-{timestamp}`
- **Latest Release:** `latest-windows`
- **Manifest:** `manifest/latest.json`

### macOS
- **Tag Format:** `macos-v{major}.{minor}.{patch}-{description}-{timestamp}`
- **Latest Release:** `latest-macos`
- **Manifest:** `manifest/latest-macos.json`

## Publishing Workflow

### For Windows

1. Create and push annotated tag in [trifecta-agent-windows](https://github.com/Carmel-Labs-Inc/trifecta-agent-windows):
   ```bash
   ts=$(date +%s); newtag=windows-v1.4.45-description-$ts && \
   git tag -a "$newtag" -m "Windows: description" && \
   git tag -a -f latest-windows -m "Update to $newtag" && \
   git push origin "$newtag" && git push -f origin latest-windows
   ```

2. GitHub Actions workflow automatically:
   - Builds and signs the Windows agent
   - Creates release in `trifecta-agent-windows`
   - **Directly publishes** to `trifecta-agent-downloads` using `CROSS_REPO_TOKEN`

3. Net effect: Tag push → CI builds → assets uploaded to Releases in both repos

### For macOS

1. Create and push annotated tag in [trifecta-native-agent-main](https://github.com/Carmel-Labs-Inc/trifecta-native-agent-main):
   ```bash
   ts=$(date +%s); newtag=macos-v1.0.0-notarized-$ts && \
   git tag -a "$newtag" -m "macOS: notarized build" && \
   git tag -a -f latest-macos -m "Update to $newtag" && \
   git push origin "$newtag" && git push -f origin latest-macos
   ```

2. GitHub Actions workflow automatically:
   - Builds the macOS agent with PyInstaller
   - Signs with Developer ID
   - Notarizes with Apple
   - Computes SHA256 checksum
   - Creates release in `trifecta-native-agent-main`
   - **Directly publishes** to `trifecta-agent-downloads` using `CROSS_REPO_TOKEN`

3. Net effect: Tag push → CI builds → assets uploaded to Releases in both repos

## Manifest Format

Both Windows and macOS manifests follow the same structure:

```json
{
  "url": "https://github.com/Carmel-Labs-Inc/trifecta-agent-downloads/releases/download/{TAG}/TrifectaAgentClient-{Platform}.zip",
  "sha256": "UPPERCASE_SHA256_HASH"
}
```

## Auto-Update Integration

The agent's `auto_updater.py` checks these manifests periodically:

```python
# Set manifest URL via environment variable
TRIFECTA_UPDATE_MANIFEST_URL="https://raw.githubusercontent.com/Carmel-Labs-Inc/trifecta-agent-downloads/main/manifest/latest-macos.json"
```

## Required Secrets

### In trifecta-agent-windows (for Windows builds):
- `WIN_CERT_PFX` - Base64-encoded code signing certificate
- `WIN_CERT_PASS` - Certificate password
- `CROSS_REPO_TOKEN` - Fine-grained PAT with:
  - **Repository access:** All repositories in organization (or at minimum: trifecta-agent-downloads)
  - **Contents:** Read and write
  - **Metadata:** Read-only (automatic)

### In trifecta-native-agent-main (for macOS builds):
- `P12_BASE64` - Base64-encoded P12 certificate
- `P12_PASSWORD` - P12 certificate password
- `KEYCHAIN_PASSWORD` - Temporary keychain password
- `DEV_IDENTITY` - Developer ID certificate SHA-1 hash
- `NOTARY_P8_BASE64` - Base64-encoded App Store Connect API Key (.p8)
- `NOTARY_KEY_ID` - App Store Connect API Key ID
- `NOTARY_ISSUER_ID` - App Store Connect Issuer ID
- `CROSS_REPO_TOKEN` - Same fine-grained PAT as Windows (shared token)

### In trifecta-agent-downloads:
- **No secrets needed** - Build repos publish directly using their own `CROSS_REPO_TOKEN`

## Release Notes

### macOS v1.0.0
- Initial automated release pipeline
- 960x540 video compression (clear quality, ~2.1GB/hour)
- Fully signed and notarized by Apple
- Organized build scripts in `build_scripts/` folder
- Comprehensive documentation in `docs/` folder
