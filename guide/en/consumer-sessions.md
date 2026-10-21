# Consumer sessions

A consumer session reads topics as a real member of a consumer group.

<a id="what-is"></a>

## What a session is

[Browsing](browse.md) reads messages without joining a group and changes nothing in the cluster. A session is different: it joins a group with the given `group.id`, gets partitions from Kafka and can commit offsets. Use it to read what a group would read, to move the offsets of a group by committing, or to watch how a group rebalances.

A session affects the cluster. When it joins or leaves, Kafka redistributes the partitions of the group, and the other members go through a rebalance. Commits change the offsets of the group, which its applications will use.

Sessions are not available on **Read-only** connections: the tab shows **Consumer sessions are unavailable for read-only connections: joining a group changes its state.** If a connection becomes read-only while a session runs, the session leaves the group.

<a id="start-session"></a>

## Start a session

Open a session in one of three ways:

- **Consumer session** under the connection in the **Connections** panel — an empty form;
- **Read as group** on a topic tab — the topic is filled in;
- **Read as member** on a group tab — the group and the topics it has commits for are filled in.

Each time, a new tab opens, so several sessions can run at once.

In **Group and topics**:

- `group.id` — the group to join. If you do not enter one, the app suggests a new name such as `glafira-kafka-anna-1a2b3c`. A new group has no commits.
- **Topics** — topic names separated by commas or spaces. Or select **topics by regex** and enter a **Topic regular expression**, for example `orders\..*`. The expression is matched from the start of the name: `^` is added if it is missing. Topics that match and are created later are added within a few minutes (on the metadata refresh); this causes a rebalance.
- `auto.offset.reset` — where to start in partitions for which the group has no committed offset: **from the end (latest)** (the default) or **from the beginning (earliest)**. Where the group has a committed offset, reading continues from it.
- **auto-commit** — commit read offsets automatically. Off by default. See [Commits](consumer-sessions.md#commits).
- `isolation.level` — `read_committed` (the default) shows only messages of committed transactions. `read_uncommitted` also shows messages of open and aborted transactions.

In **Format**, choose the **Key format** and the **Value format**, as in browsing. See [Formats](browse.md#formats). **Buffer, messages** sets how many messages the table keeps, from 1 to 100000; the default is 5000.

The filters work as in browsing. See [Filters](browse.md#filters).

Click **Join group**. On a prod connection, the app asks you to type the connection name and confirm with **Confirm**. While the session runs, the form cannot be changed.

<a id="reading"></a>

## Reading

The state line next to the buttons shows **connecting…**, then **in group**. It also shows how many messages were read, how many matched the filters and how many are in the table. If the connection to the broker is lost, the session reconnects by itself and shows this in the state line.

Messages appear in the table as they arrive, and the session keeps reading new messages until you stop it. When the table is full, the oldest messages are removed. Click a row to see the message details, as in browsing. See [Message details](browse.md#message-details). Select `UTC` to show times in UTC.

The session reads only the partitions Kafka has assigned to it. If the group has other members, they read the other partitions. If the group has more members than partitions, the session may get no partitions at all.

**Disconnect** leaves the group, and the other members get its partitions. Closing the session tab also leaves the group. When the session has ended, the state line shows **disconnected**.

If a topic does not exist, the session stops with an error and shows the names to check.

<a id="commits"></a>

## Commits

A commit saves an offset for the group. The next member that reads the partition, or the same application after a restart, starts from that offset.

With **auto-commit** on, the session commits read offsets in the background every few seconds, and once more when it leaves the group. Messages hidden by filters count as read.

Without auto-commit, the session commits nothing by itself. Use the buttons:

- **Commit up to selected** — commits the offset after the selected message, in its partition only. The group will continue from the next message. If the selected message is older than the current commit, the offset moves back.
- **Commit everything read** — commits the current position in every assigned partition. This includes messages hidden by filters and messages already removed from the table.

After a commit, a line shows the committed offsets as `topic:partition → offset`. On a prod connection, each commit asks for the connection name.

If the session does not answer in time, the app warns that it is unknown whether the commit was applied. Check the offsets on the group tab. See [Group details](groups.md#group-details).

<a id="pause-partitions"></a>

## Pause partitions

The panel to the right of the table lists the partitions assigned to the session as `topic:partition`. Click **Pause** next to a partition to stop reading it, and **Resume** to continue. A paused partition is marked **paused**.

Pausing does not leave the group, so there is no rebalance. The partition stays assigned to the session, and other members do not read it while it is paused. Pausing changes nothing in the cluster and needs no confirmation.

After a rebalance, newly assigned partitions start unpaused. When the session leaves the group, all pauses end.

<a id="rebalance-log"></a>

## Rebalance log

**Rebalance log** under the partition list records every change of the assignment, newest first. Each entry has a time and one of three kinds: partitions assigned to the session, partitions revoked from it, or a rebalance error with its text.

A rebalance happens when a member joins or leaves the group, when the subscribed topics change, or when a member stops responding in time. To watch it, start a second session with the same `group.id` in another tab: the partitions are split between the two sessions, and both logs show it.

The log keeps the last 200 entries. Before the first rebalance, it shows **No rebalances yet.**

<a id="limits"></a>

## Limits

- Sessions are not available on **Read-only** connections.
- A session cannot start from a chosen offset. To set the start, reset the offsets of the group first: see [Reset offsets](groups.md#reset-offsets). The reset needs a group without members, so disconnect sessions of this group before resetting.
- The table keeps at most 100000 messages. Older messages are removed, but they still count as read.
- Very large keys and values are shown only partly, as in browsing.
- Do not join the group of a running application unless you mean to. The session takes partitions from the application, and the application does not process them while the session holds them. Commits from the session, including auto-commits, move the offsets of the application.
