# trifecta-agent-downloads

Central distribution repository for Trifecta Agent releases across all platforms.

## Repository Structure

```
trifecta-agent-downloads/
├── manifest/
│   ├── latest.json          # Windows manifest (legacy)
│   └── latest-macos.json    # macOS manifest
└── .github/workflows/
    ├── publish-from-windows.yml
    └── publish-from-macos.yml
```

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

1. Build runs automatically in [trifecta-agent-windows](https://github.com/Carmel-Labs-Inc/trifecta-agent-windows) repo
2. Go to Actions → "Publish Windows Artifact"
3. Run workflow (optionally specify a run ID)
4. Workflow will:
   - Download build artifact
   - Create dated release tag
   - Upload to `latest-windows`
   - Update `manifest/latest.json`

### For macOS

1. Build runs in [trifecta-native-agent-main](https://github.com/Carmel-Labs-Inc/trifecta-native-agent-main) repo:
   - Go to Actions → "Build and Sign macOS App"
   - Run workflow (choose environment: notarized/public/dev)
   - Wait for build to complete (includes signing + notarization)

2. Publish to downloads repo:
   - Go to Actions → "Publish macOS Artifact"
   - Run workflow (optionally specify a run ID)
   - Workflow will:
     - Download notarized build artifact
     - Calculate SHA256
     - Create dated release tag (`macos-v1.0.0-notarized-{timestamp}`)
     - Upload to `latest-macos`
     - Update `manifest/latest-macos.json`

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

### In trifecta-native-agent-main (for macOS builds):
- `P12_BASE64` - Base64-encoded P12 certificate
- `P12_PASSWORD` - P12 certificate password
- `KEYCHAIN_PASSWORD` - Temporary keychain password
- `DEV_IDENTITY` - Developer ID certificate name or SHA-1 hash
- `NOTARY_P8_BASE64` - Base64-encoded App Store Connect API Key (.p8)
- `NOTARY_KEY_ID` - App Store Connect API Key ID
- `NOTARY_ISSUER_ID` - App Store Connect Issuer ID

### In trifecta-agent-downloads (for publishing):
- `CROSS_REPO_TOKEN` - GitHub token with permissions to:
  - Read artifacts from source repos (trifecta-agent-windows AND trifecta-native-agent-main)
  - Create releases in downloads repo
  - Push to main branch
  - **Note:** Same token works for both Windows and macOS workflows

## Release Notes

### macOS v1.0.0
- Initial automated release pipeline
- 960x540 video compression (clear quality, ~2.1GB/hour)
- Fully signed and notarized by Apple
- Organized build scripts in `build_scripts/` folder
- Comprehensive documentation in `docs/` folder
