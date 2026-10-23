# Getting started

Glafira for Apache Kafka is a desktop client for Kafka clusters. It lets you browse topics and consumer groups, read and send messages, and reset offsets. This chapter covers installation, the first launch, a local Kafka for practice, and the interface language.

<a id="install-macos"></a>

## Install on macOS

Glafira runs on macOS 11 or later. Download the installer from the releases page: https://github.com/vvnovg/glafira-kafka-releases/releases

Pick the file for your Mac: `aarch64` for Apple silicon (M1 and later), `x64` for Intel.

1. Open the `.dmg` file.
2. Drag Glafira for Apache Kafka to the Applications folder.
3. Start the app from Applications or Launchpad.

You can also use the `.pkg` installer. It puts the app in `/Applications` for all users.

The builds are signed by Apple and notarized, so the app opens right away without security warnings.

If macOS still says the app is damaged or cannot be verified, the download is corrupted: download the `.dmg` or `.pkg` again from the releases page.

<a id="install-windows"></a>

## Install on Windows

Download the `.msi` installer (`x64`) from the releases page: https://github.com/vvnovg/glafira-kafka-releases/releases

1. Run the `.msi` file.
2. Confirm the installation in the Windows prompt. The app is installed for all users of the computer, so administrator rights are needed.
3. Start Glafira for Apache Kafka from the Start menu.

Beta builds are not signed yet. If SmartScreen blocks the installer, click "More info", then "Run anyway". You can also unblock the file in PowerShell before running it:

```powershell
Unblock-File -Path "$env:USERPROFILE\Downloads\Glafira_for_Apache_Kafka_*_x64_en-US.msi"
```

<a id="install-linux"></a>

## Install on Linux

Builds are available for `x86_64` in three formats. Download the one that suits your distribution from the releases page: https://github.com/vvnovg/glafira-kafka-releases/releases

- Debian, Ubuntu and derivatives: `sudo apt install ./<file>.deb`
- Fedora, RHEL and derivatives: `sudo dnf install ./<file>.rpm`
- Any distribution: make the AppImage executable with `chmod +x <file>.AppImage` and run it.

After installing a package, start the app from the applications menu or run `glafira-kafka` in a terminal.

Glafira stores passwords in the Secret Service (for example, GNOME Keyring or KWallet). Make sure one of them runs in your session. Otherwise the app cannot save passwords for connections.

<a id="first-run"></a>

## First run

The window has two parts. On the left is the **Connections** panel. On the right are tabs: each screen you open, such as a topic list or the Producer, gets its own tab.

When there are no connections yet, the panel shows a short hint, and the tab area suggests adding a connection. To start:

1. Click `+` ( **New connection** ) in the header of the **Connections** panel.
2. Fill in **Name** and **Bootstrap servers**, for example `localhost:9092`.
3. Click **Test connection** to check the settings, then **Save**.

After saving, the cluster overview opens. Under each connection in the panel there are links to **Cluster overview**, **Topics**, **Consumer groups**, **Producer** and **Consumer session**. Details on all connection settings are in [Connections](connections.md#create).

At the bottom of the panel there are links to **Settings**, **Support the project**, **Help** and **About**. The `?` sign next to a field shows a short hint about it. Press `F1` anywhere in the window to open the guide chapter for the current screen. On macOS the guide is also in the Help menu.

If you used Kafka Explorer (the earlier name of the app), Glafira copies its connections, templates and send history on the first launch. Passwords are copied from the system secret store too. Nothing that already exists is overwritten, and the old data is not deleted. macOS may ask once for access to the old Keychain items.

<a id="local-cluster"></a>

## Local Kafka for practice

To try the app without a shared cluster, start Kafka on your computer in Docker. One broker is enough:

```bash
docker run -d --name kafka -p 9092:9092 apache/kafka:3.9.0
```

Then create a connection with bootstrap server `localhost:9092`, protocol `PLAINTEXT` and environment `local`. When you no longer need the broker, remove it with `docker rm -f kafka`.

The hint in the empty **Connections** panel mentions `docker compose -f dev/docker-compose.yml up -d`. This file is part of the Glafira source code. If you have the source code, this command starts a broker with three listeners:

- `localhost:9092` — `PLAINTEXT`.
- `localhost:9094` — `SASL_PLAINTEXT` with `PLAIN` (users `tester` / `tester-secret` and `admin` / `admin-secret`) and SCRAM (`scram-user` / `scram-secret` for `SCRAM-SHA-256` and `SCRAM-SHA-512`).
- `localhost:9095` — `SSL` with client certificates (mTLS). The test certificates are in `dev/certs/out`.

These users and certificates are for local tests only.

<a id="language"></a>

## Interface language

The interface is available in English, Russian, German, French, Spanish, Italian, Polish, Portuguese (Brazil), Chinese (Simplified and Traditional), Japanese and Korean. All languages are free.

To change the language:

1. Click **Settings** at the bottom of the left panel.
2. Choose a language in **Interface language**.

The change applies at once, including this guide and the macOS menu. **System default** (the default) follows the language of your operating system. If that language is not in the list, the interface is in English.

<a id="managed-settings"></a>

## Settings set by your organization

In a company, administrators can set some options for all users through a policy. You cannot change these options in the app. A policy can:

- set the interface language — the language list in **Settings** is then unavailable, with a note that the language is set by your organization;
- turn off update checks, or point them to an internal mirror.

To see whether a policy applies, open **About** and look at **Organization policy**. If the policy has an error, it is shown there. For questions about the policy, contact your administrators. Updates are described in [Updates and diagnostics](updates.md).
