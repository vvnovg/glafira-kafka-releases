# Updates and diagnostics

This chapter covers how to update the app, the **About** screen, the diagnostics archive and how to support the project.

<a id="updates"></a>

## Updates

When it starts, the app checks once whether a new version is available. If there is one, a notice above the tabs gives its version. **Details** opens the **About** screen. If the check fails, for example without internet access, no message is shown.

To check manually:

1. Open **About** at the bottom of the left panel.
2. In **Updates**, click **Check for updates**.
3. If a new version is available, the screen shows its number and release notes. Click **Install and restart**.

The app downloads the update, checks its signature, installs it and restarts. A download interrupted by a network problem is retried automatically, up to three attempts in total. If all three attempts fail, the app reports **The update download was interrupted — check your network and try again**. If the signature check fails, the app reports **The update signature failed verification — the update was not installed**.

Installing may ask for an administrator password, because the app is installed for all users of the computer: by the `.pkg` installer on macOS, the `.msi` installer on Windows, or a `.deb` or `.rpm` package on Linux. Users without administrator rights cannot update such an installation.

If messages are being read, a consumer session is running or a batch is being sent, the app asks first and says how many sessions are active. Click **Stop sessions and update** to stop them and continue, or **Cancel**. Connections, settings and other data are kept after an update.

Updates are downloaded from the releases page on GitHub. If the check fails with **Failed to check for updates**, check your network or proxy. **Updates are not configured in this build** means that this build was made without the updater key, for example a development or test build. In that case, download new versions from the releases page: https://github.com/vvnovg/glafira-kafka-releases/releases

An organization can turn off update checks through a policy, or point them to an internal mirror. When checks are turned off, the **Updates** section says **Updates are managed by your organization** after you click **Check for updates**. Like **Updates are not configured in this build**, this message appears only after the click; the check at startup shows nothing in these cases.

<a id="about"></a>

## About

Open **About** at the bottom of the left panel. On macOS, it is also in the Help menu as **About Glafira**.

The screen shows a short description of the app and:

- **Version** — the version number, and **Build** — the identifier of the source code it was built from. Include both in a support request.
- **System** — the operating system and processor architecture.
- **License** — Apache 2.0; the link opens the license text in the browser.
- **Organization policy** — **set** or **none**. Errors in the policy are listed below.
- **Data:** — the folder with connection profiles, templates, history and settings.
- **Logs:** — the folder with log files. See [Logs](troubleshooting.md#logs).

The **Updates** and **Diagnostics** sections are also on this screen.

<a id="diagnostics"></a>

## Diagnostics archive

The diagnostics archive collects what support needs to understand a problem. To create it:

1. Open **About**.
2. In **Diagnostics**, click **Collect diagnostics**. On macOS, you can also choose **Collect Diagnostics…** in the Help menu.
3. Choose where to save the file. The suggested name is `diagnostics-<date>.zip`.

When the archive is saved, the screen shows its path. The ZIP file contains:

- `info.json` — the app version and build, the operating system, the data and log folder paths and the policy status;
- `connections.json` — connection profiles: names, environments, bootstrap servers, protocols, SASL usernames, certificate file paths, extra properties and timeouts;
- `logs/` — the app log files.

The archive contains no passwords: they are kept in the system secret store, and extra properties with passwords and secrets are refused when a connection is saved. Certificate files are not included either. Host names, usernames and file paths are stored as plain text, and the logs contain broker addresses from Kafka error messages. Look through the archive before you send it outside your company.

The app sends nothing by itself. Attach the archive to your support request yourself, see [Contact support](troubleshooting.md#support).

<a id="support"></a>

## Support the project

Glafira is free and open source under the Apache License 2.0. Donations keep it going. **Support the project** at the bottom of the left panel, on the **About** screen and, on macOS, in the Help menu opens the support page in the browser: https://github.com/vvnovg/glafira-kafka-releases/blob/main/SUPPORT.md
