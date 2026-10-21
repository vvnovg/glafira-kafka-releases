# Consumer groups

A consumer group is a set of consumers that share the reading of topics. Kafka gives each partition to one member of the group and stores, for each partition, the offset the group has committed. Applications continue reading from that offset after a restart.

Open **Consumer groups** under the connection in the **Connections** panel. The groups of one topic are also on the **Consumer groups** tab of the topic. See [Topic screen](topics.md#topic-details).

<a id="group-list"></a>

## Group list

The list shows all consumer groups of the cluster. The columns:

- **Group** — the group ID (`group.id`).
- **State** — the state reported by Kafka, for example `Stable` (members are reading), `Empty` (no members), `PreparingRebalance` or `CompletingRebalance` (partitions are being redistributed) and `Dead` (the group no longer exists).
- **Members** — the number of active members.
- `Assignor` — the strategy used to distribute partitions, for example `range`.
- **Lag** — the total lag of the group. See [Lag](groups.md#lag).
- **Topics** — the topics the group has committed offsets for.

Type in **Search groups** to show the groups whose ID contains the text; case does not matter. The state list next to it shows only groups in one state; **All states** removes this filter. The counter shows how many groups are shown and how many there are in total.

Members, assignor and lag are loaded only for the rows you see, while you scroll. Until they arrive, the cell shows `…`. If they cannot be loaded, the cell shows `—`; point at it to see the reason.

**Refresh** reloads the list and the data in it. If some brokers did not respond, a warning says the list may be incomplete.

<a id="lag"></a>

## Lag

Lag is how many messages the group has not read yet. For each partition, it is the end of the log minus the committed offset. Lag is never shown below zero.

Lag depends only on committed offsets. An active consumer may have read further than it has committed, so its real lag can be smaller. When the group commits again, the lag drops.

A partition without a committed offset shows **no commit** and does not count in the total. The total lag of a group shows `—` when the group has no commits at all (**no commits**), or when the offsets of some partition could not be read (**lag unknown: partition offsets error**).

Lag counts offsets, not messages. In topics with compaction or transactions, some offsets hold no visible messages, so the group may have fewer messages left to read than the lag shows.

The values are a snapshot at the moment of loading. To follow the lag, open the group and turn on auto-refresh.

<a id="group-details"></a>

## Group details

Click a group in the list to open its tab. At the top, a summary shows the state, the number of members and the assignor.

- **Auto-refresh** — **Auto-refresh: off**, **every 5 s** or **every 10 s**.
- **Refresh** — reload now.
- **Read as member** — start a [consumer session](consumer-sessions.md) in this group. The session form gets the group ID and the topics the group has commits for. The button is disabled on **Read-only** connections; point at the button to see the reason.

**Partitions** lists the partitions the group has commits for, and the partitions assigned to its members. The total lag of the group is shown above the table. The columns are **Topic**, **Partition**, `Committed` (the committed offset), `Log end` (the end offset of the partition), **Lag** and **Member** (the client ID of the member that reads the partition).

**Members** lists the active members: `Client ID`, **Host**, `Member ID` and **Assignment** (the partitions of the member as `topic:partition`). A group without members shows **No active members.**

The second row has **Reset offsets** and **Delete group**. It is hidden on read-only connections.

<a id="reset-offsets"></a>

## Reset offsets

Resetting offsets moves the committed offsets of a group. The group then rereads messages or skips them. Offsets can be reset only for a group without active members (state `Empty` or `Dead`). Otherwise the members would overwrite the new offsets with their own commits. Stop the applications of the group first.

1. On the group tab, click **Reset offsets**. While the group has members, the button is disabled; point at the button to see the reason.
2. Choose the **Strategy**. See [Reset strategies](groups.md#reset-strategies).
3. Choose the **Scope**. See [Reset scope](groups.md#reset-scope).
4. Click **Preview**.
5. Check the table and click the apply button. It shows the number of partitions that will change.

The preview changes nothing. For each partition it shows the topic, the partition, **Current** (the committed offset), **New** (the target offset), and `Low / High` (the first and end offsets of the partition). A partition for which no target can be computed is marked **No target offset — will not be applied**, with the reason. Such partitions are skipped.

Only the plan shown in the preview is applied. Any change in the form hides the preview, and you need to click **Preview** again. On a prod connection, applying asks for the connection name; on other connections, it applies at once.

Before applying, the app checks again that the group has no members. After applying, **Reset result** lists each partition with its new offset or an error.

You can also prepare a reset from a message: see [Reset a group to a message](browse.md#reset-group-here). **Close** closes the reset panel.

<a id="reset-strategies"></a>

## Reset strategies

**Strategy** sets the target offset for each partition in the scope:

- **From the beginning (earliest)** — the first offset that is still stored. The group rereads everything the topic keeps.
- **From the end (latest)** — the end of the partition. The group skips everything accumulated and reads only new messages.
- **By time** — the first message written at or after **Date and time (local)**. In partitions without newer messages, the target is the end.
- **Specific offset** — the given **Offset**, an integer of 0 or more, in every partition. If a partition does not have this offset, the target is its first or end offset.
- **Shift by ±N** — moves the committed offset by **Shift (±N)**. A negative number rereads N messages back, a positive one skips N messages. The result stays within the first and end offsets. Partitions without a committed offset cannot be shifted and are skipped.

The time is your local time, with seconds. Kafka finds the offset by message timestamps.

<a id="reset-scope"></a>

## Reset scope

**Scope** sets which partitions are reset. Other partitions keep their offsets.

- **Whole group** — all partitions the group has committed offsets for. If the group has no commits yet, choose topics instead.
- **Topics** — all partitions of the listed topics, including partitions without commits. Separate the names with commas, spaces or line breaks. Use this to make a group start reading a new topic from a chosen place.
- **Partitions** — only the listed partitions. Enter them in **Partitions (topic:partition)** as `topic:partition`, separated by commas or spaces, for example `orders:0, orders:3`.

<a id="delete-group"></a>

## Delete a group

Deleting a group removes all its committed offsets. Only a group without members, in the `Empty` state, can be deleted.

1. On the group tab, click **Delete group**. While the group has members, the button is disabled; point at the button to see the reason.
2. Type the group ID to confirm. On a prod connection, also type the connection name.
3. Click **Delete**.

After deleting, the group tab closes. If an application with this `group.id` starts again, Kafka creates the group anew. Without committed offsets, the application starts where its `auto.offset.reset` setting says: from the beginning or from the end.

On **Read-only** connections, the button is hidden.
