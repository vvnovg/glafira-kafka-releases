# Troubleshooting

This chapter explains common connection errors and what to check for each of them. It also shows where the logs are and what to include in a support request. On macOS, the Help menu opens this chapter with **Troubleshooting**.

An error message has a short text and, on most screens, the error code in brackets, for example `TIMEOUT`. Expand **Details** to see the original message from the Kafka client (librdkafka). It often names the exact broker address and cause.

<a id="cannot-connect"></a>

## Cannot connect

Start with **Test connection** in the connection form, see [Test the connection](connections.md#test). The test stops at the first failed step, and the step shows where to look:

- **Configuration** — the problem is in the form itself: an address without a port, a missing or unreadable certificate file, a wrong certificate password. The error names the field. For certificate files, see [TLS errors](troubleshooting.md#tls-errors).
- **DNS / TCP** — the computer cannot reach the bootstrap servers. See [DNS and TCP](troubleshooting.md#dns-tcp).
- **Kafka (TLS, SASL, metadata)** — the network works, but the TLS handshake, the SASL login or the metadata request failed. See [TLS errors](troubleshooting.md#tls-errors), [SASL errors](troubleshooting.md#sasl-errors) and [Timeouts](troubleshooting.md#timeouts).

The most common cause is a protocol that does not match the port. Each broker listener has its own port and protocol. Ask your Kafka administrators which port belongs to which protocol, see [Security protocols](connections.md#security).

If the test passes but reading or sending fails, the cause is often the broker addresses. The client gets the list of brokers from the bootstrap server and then connects to each broker at the address the broker advertises (`advertised.listeners`). If these addresses are internal, for example Docker or Kubernetes host names, they cannot be reached from your computer. The addresses are listed in the broker table of the cluster overview, see [Cluster overview](topics.md#cluster-overview). Ask your administrators for a listener that is reachable from outside.

<a id="dns-tcp"></a>

## DNS and TCP

**None of the bootstrap servers is reachable over TCP** means that the app could not open a TCP connection to any address in **Bootstrap servers**. Each address gets up to 5 seconds. The details show the last address tried and the reason. `DNS` in the details means the host name could not be resolved.

Check:

- the host name and port, without a protocol prefix such as `https://`;
- that a VPN is on, if the cluster is in a private network;
- that a firewall does not block the port;
- for a broker in Docker, that the port is published to the host, for example `-p 9092:9092`.

To check the port outside the app, run `nc -vz <host> <port>` on macOS or Linux, or `Test-NetConnection <host> -Port <port>` in PowerShell on Windows.

**Could not connect to the brokers** after the TCP step has passed usually means that a broker closed the connection or could not be reached later. Also check the broker addresses, see [Cannot connect](troubleshooting.md#cannot-connect).

<a id="tls-errors"></a>

## TLS errors

Errors on the **Kafka (TLS, SASL, metadata)** step during the TLS handshake:

- **The broker accepts connections without TLS on this address — check the port or choose a protocol without SSL**.
- **The broker certificate is not signed by a trusted CA — check “Trusted CA”**. Set **Trusted CA certificate** to the file of the CA that signed the broker certificates. Your administrators can give you this file. With an empty field, only the system root certificates are trusted.
- **The broker host name does not match its certificate**. Connect by the name that the certificate was issued for, not by an IP address or an alias. In test environments only, you can turn off **Verify hostname**.
- **The broker rejected the client certificate**. The broker requires a client certificate, or it does not trust the one you use. Check that **Client certificate (mTLS)** is on and that the certificate was issued for this cluster. See [Client certificate (mTLS)](connections.md#mtls).
- **TLS connection error** — another TLS problem. Expand **Details** for the cause.

If you chose a protocol without SSL and still see a TLS error, the port is probably a TLS listener. The details then say `connecting to a SSL listener?`. Choose `SSL` or `SASL_SSL`, or use the port of a listener without TLS.

Problems with certificate files are reported on the **Configuration** step, before the app connects. The message starts with the field name: trusted CA, client certificate or client key. Common cases:

- The path is not absolute or the file cannot be read. Enter the full path or choose the file with **Browse…**.
- **Client key: the key is encrypted — specify the key password**, **Client key: wrong key password**, **Client certificate: wrong PKCS\#12 store password** or **Wrong JKS store password (or the file is damaged)** — enter the right password in the form.
- **The client key does not match the client certificate** — the key file belongs to another certificate.
- The client certificate has expired or is not valid yet — the message gives the date. Ask for a new certificate.
- The PKCS#12 store uses a legacy algorithm, or the store is in JCEKS format. Convert the file as shown in [Certificate formats](connections.md#certificate-formats).

<a id="sasl-errors"></a>

## SASL errors

**Authentication failed** means that the broker did not accept the SASL login. Check:

- the username and password. When you edit a saved connection, an empty password field keeps the saved password. Type the password again if you are not sure it is right;
- **SASL mechanism**. It must be the mechanism your user was created for. SCRAM credentials are created separately for `SCRAM-SHA-256` and `SCRAM-SHA-512`, and `PLAIN` users are configured separately again;
- the protocol: `SASL_SSL` and `SASL_PLAINTEXT` usually use different ports.

**The SASL password for the connection is not set** — no password is saved for this connection, for example after an import. Open the connection, enter the password and save it. See [Passwords](connections.md#passwords).

**Cannot access the system secret store** — the app cannot read the password from the system secret store. On macOS, allow access when Keychain asks. On Linux, check that GNOME Keyring, KWallet or another Secret Service provider runs in your session.

**Not authorized to perform the operation** is different: the login succeeded, but the Kafka user has no rights (ACLs) for this operation, for example reading a topic or a group. Ask your administrators for the rights. Note that Kafka does not report an error for topics that you are not allowed to describe: they are simply missing from the topic list.

<a id="timeouts"></a>

## Timeouts

**The operation timed out** means that the broker did not answer in time. Typical causes are a slow network or VPN, an overloaded cluster, or broker addresses that cannot be reached, see [Cannot connect](troubleshooting.md#cannot-connect).

A TLS or SASL problem can look like a timeout. The app tries to find the real cause and shows it instead, but this is not always possible. Expand **Details**: messages such as `SSL handshake failed` or `SASL authentication error` point to the cause.

If the network is just slow, increase **Request timeout, ms** in the connection form. The default is `30000` ms. This value applies to requests to the brokers, to sending messages and to reading. The connection test uses its own limits (5 seconds for each TCP address and 10 seconds for the Kafka step), so a larger value does not change the test.

<a id="read-only-blocked"></a>

## An action is unavailable or refused

If a button is hidden or disabled, or an action is refused, check these causes:

- The connection is read-only. A lock icon next to its name marks it. Buttons that change the cluster are hidden or disabled. If such a request still reaches the app core, for example after the mode was turned on while a screen was open, the error says that the connection is read-only. To make changes, open the connection, clear **Read-only** and save. See [Read-only](connections.md#read-only).
- The connection is prod. Changes ask you to type the connection name. Until the name matches, the action is not performed. See [Environments and prod](connections.md#environments).
- The consumer group has active members. Offsets can be reset and a group can be deleted only when it has no members. See [Reset offsets](groups.md#reset-offsets).
- The setting is managed by your organization. A policy can set the interface language and update checks, and these cannot be changed in the app. See [Settings set by your organization](getting-started.md#managed-settings).
- Kafka refused the operation with **Not authorized to perform the operation**. This is a matter of rights in the cluster, not of Glafira settings. See [SASL errors](troubleshooting.md#sasl-errors).

<a id="logs"></a>

## Logs

The app writes a log of its work and of Kafka client errors. The exact folder is shown on the **About** screen in **Logs:** By default:

- macOS: `~/Library/Logs/io.github.vvnovg.glafira.kafka`
- Windows: `%LOCALAPPDATA%\io.github.vvnovg.glafira.kafka\logs`
- Linux: `~/.local/share/io.github.vvnovg.glafira.kafka/logs`

The current file is `app.log`. When it reaches 5 MB, it is renamed to `app.1.log`, and older files move on to `app.2.log` and so on. The app keeps up to five files and deletes the oldest one.

Passwords are not written to the logs. Broker addresses and the texts of Kafka errors are written. You do not need to collect the files by hand: the diagnostics archive includes them, see [Diagnostics archive](updates.md#diagnostics).

<a id="support"></a>

## Contact support

Before you contact support, run **Test connection** and note which step fails. Then send the request to whoever supports Glafira for you, for example your administrators. Include:

- the diagnostics archive, see [Diagnostics archive](updates.md#diagnostics). It contains the version, the build, the logs and the connection profiles without passwords;
- what you did, what you expected and what happened, step by step;
- the full error text, together with the **Details** content. You can select and copy this text;
- a screenshot, if the problem is in the interface;
- the time when the error happened, to find it in the logs.

If you cannot create the archive, give the **Version** and **Build** values from the **About** screen and attach the log files from the logs folder.
