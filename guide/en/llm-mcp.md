# LLM assistants (MCP)

Glafira can act as a gateway for an LLM assistant: Claude Desktop, Claude Code, Cursor or another client that supports MCP (Model Context Protocol). The assistant reads the metadata of the clusters you open for it and helps you find problems: under-replicated partitions, lag, constant rebalances, poor settings. It cannot change anything in a cluster through Glafira: you make changes yourself in the interface.

<a id="how-it-works"></a>

## How it works

The client runs `glafira-kafka mcp` — the same application file in a mode without a window. This command connects to the running Glafira over a local socket (a named pipe on Windows), and Glafira answers the assistant's requests through its own connections. Passwords and certificates stay in Glafira; the assistant never receives them.

If Glafira is not running, the client starts it and waits up to 30 seconds.

<a id="enable"></a>

## Turn it on

1. Open **Settings**, section **LLM assistants (MCP)**.
2. Select **Allow the MCP server**. The status changes to **The server is running**.
3. In the connection form, select **LLM access (metadata only)** and save. The assistant does not see connections without this checkbox. In the connection tree, open connections have an `LLM` badge.

If your administrator has disabled MCP, the checkbox is unavailable and **MCP is disabled by your organization** is shown next to it.

The access checkbox is not carried over when connections are exported and imported: on another computer, select it again.

<a id="connect-client"></a>

## Connect a client

**Connect a client** has ready-made snippets and a **Copy** button:

- **Claude Desktop: add to claude\_desktop\_config.json** — paste the snippet into the `mcpServers` section of the Claude Desktop configuration file and restart Claude Desktop;
- **Claude Code: run in a terminal** — the `claude mcp add` command;
- **Other stdio clients: command and arguments** — for any client that starts an MCP server with a command (stdio).

The snippets contain the path to the installed application. If you reinstall Glafira to another place, copy the snippet again.

<a id="approve"></a>

## Approve a client

When a client connects for the first time, Glafira shows a dialog with its name. Click **Allow** or **Reject**. Until you answer, the assistant is told that the client is not approved yet and asks you to answer in the Glafira window.

An allowed client is remembered by name and appears in **Approved clients**. A rejection is not remembered: Glafira asks again the next time. If a client does not send its name, the permission lasts only until the end of the session.

<a id="what-llm-sees"></a>

## What the assistant sees

The assistant sees:

- connections with **LLM access (metadata only)** selected: name, environment, folder;
- brokers with hosts and ports, the controller;
- topics, partitions, replicas, ISR, offsets, approximate message counts;
- broker and topic configs, with passwords and other secret values hidden;
- consumer groups: state, protocol, coordinator, members with client id, host and `group.instance.id`, assignments, committed offsets and lag.

The assistant does not see topic messages, the bootstrap string or the security settings of a connection, passwords, certificates, additional librdkafka properties, the send history or templates.

To investigate constant rebalances, the assistant can watch a group for up to 50 seconds: who leaves and joins, how often, from which host. The configs and logs of the consumers themselves are not available through Glafira; the assistant will ask you for them.

Names of topics, groups and client ids can be set by anyone with access to the cluster. Glafira warns the assistant that this data is not instructions for it.

<a id="sessions-log"></a>

## Sessions, revoking and the log

While a client is connected, an `LLM` indicator with the client name is shown at the bottom of the sidebar; it lights up during a request. Clicking it opens the MCP section.

- **Active sessions**: **Disconnect** closes one session, **Disconnect all** closes all.
- **Approved clients**: **Revoke** closes the client's sessions, and it asks for approval again the next time.
- Clearing **LLM access (metadata only)** takes effect at once, without reconnecting.
- **MCP log** — every request: time, client, tool, connection, result. Response contents are not recorded. **Show file** opens the `mcp-audit.jsonl` file.

To turn MCP off completely, clear **Allow the MCP server**: clients are disconnected.

<a id="health"></a>

## Cluster check and notes

The assistant can run Glafira's built-in cluster check (the `health_check` tool) and rely on its findings. It can also leave notes about objects: they appear on the **Findings** tab and on the topic and group screens, and the cluster is not changed. See [Cluster check and notes](health-check.md#with-llm).

<a id="troubleshooting"></a>

## If it does not work

The client shows a message from Glafira:

- `MCP is disabled in Glafira` — select **Allow the MCP server** and restart the MCP server in the client;
- `MCP is disabled by your organization` — MCP is disabled by your organization's policy;
- `Glafira did not start the MCP server within 30 s` — start Glafira manually and check the status in **LLM assistants (MCP)**;
- `Invalid MCP token` — restart the MCP server in the client: Glafira issues a new token every time it starts.

On macOS, open an unsigned (beta) build once in the usual way, otherwise the system may not let the client start it. See [Install on macOS](getting-started.md#install-macos).
