# Glafira for Apache Kafka — releases

Desktop client for Apache Kafka. Download the installer for your OS from [Releases](../../releases/latest).

**macOS** — open the `.dmg` and drag the app to Applications (or run the `.pkg`). The builds are signed by Apple and notarized, so the app opens right away. If macOS says the app is damaged or cannot be verified, the download is corrupted: download it again.

User guide: [English](guide/en/getting-started.md) · [Русский](guide/ru/getting-started.md) · [Deutsch](guide/de/getting-started.md) · [Français](guide/fr/getting-started.md) · [Español](guide/es/getting-started.md) · [Italiano](guide/it/getting-started.md) · [Polski](guide/pl/getting-started.md) · [Português (Brasil)](guide/pt-BR/getting-started.md) · [简体中文](guide/zh-CN/getting-started.md) · [繁體中文](guide/zh-TW/getting-started.md) · [日本語](guide/ja/getting-started.md) · [한국어](guide/ko/getting-started.md)

## Windows builds (unsigned)
macOS builds are signed by Apple and notarized. Windows builds are not signed yet, so SmartScreen may warn on first launch:

**Windows** — in the SmartScreen dialog click "More info" → "Run anyway", or unblock the installer in PowerShell:
```powershell
Unblock-File -Path "$env:USERPROFILE\Downloads\Glafira_for_Apache_Kafka_*_x64_en-US.msi"
```

## Source code and support
Glafira is free and open source under the Apache License 2.0: https://github.com/vvnovg/glafira-kafka.
All features are available to everyone. If Glafira is useful to you, see [SUPPORT.md](SUPPORT.md).

Apache Kafka is a trademark of the Apache Software Foundation. This project is not affiliated with the ASF.
