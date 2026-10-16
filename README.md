# Glafira for Apache Kafka — releases

Desktop client for Apache Kafka. Download the installer for your OS from [Releases](../../releases/latest).

## Unsigned builds (beta)
Beta builds are not signed by Apple/Microsoft yet, so the OS warns on first launch.

**macOS** — after copying the app to Applications, remove the quarantine flag once:
```bash
xattr -dr com.apple.quarantine "/Applications/Glafira for Apache Kafka.app"
```
If the downloaded `.dmg` itself won't open, remove the flag from it first:
```bash
xattr -d com.apple.quarantine ~/Downloads/Glafira_for_Apache_Kafka_*.dmg
```
Alternatively: right-click the app → Open → Open, or System Settings → Privacy & Security → "Open Anyway".
Updates installed from inside the app don't need this.

**Windows** — in the SmartScreen dialog click "More info" → "Run anyway", or unblock the installer in PowerShell:
```powershell
Unblock-File -Path "$env:USERPROFILE\Downloads\Glafira_for_Apache_Kafka_*_x64_en-US.msi"
```

## Pro license
During the beta, Pro keys are free — contact the author.

Apache Kafka is a trademark of the Apache Software Foundation. This project is not affiliated with the ASF.
