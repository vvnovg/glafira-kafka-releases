# Browsing messages

You browse messages on the **Messages** tab of a topic. Browsing does not join a consumer group and commits nothing, so the cluster does not change. It also works on **Read-only** connections. To read as a member of a group, use a [consumer session](consumer-sessions.md).

<a id="start-reading"></a>

## Start reading

1. Open a topic from the **Topics** list. The **Messages** tab opens first.
2. Choose where to start in **From**. See [Start position](browse.md#start-position).
3. Optionally, set the formats and the filters.
4. Click **Read**.

Messages appear in the table as they arrive. The line next to the buttons shows how many messages were scanned, how many matched the filters and how many are in the table. It also shows the state: **reading…**, **limit reached**, **reached end of topic** or **stopped**.

Without `Live tail`, reading stops at the end of the topic as it was when you clicked **Read**, or earlier when the **Limit** is reached. The limit counts messages that match the filters. It can be from 1 to 100000; large values make reading slower.

With `Live tail` on, reading does not stop at the end: new messages are added as they are written. When all old messages are read, the state shows **waiting for new messages…**. The table keeps the last messages up to the limit and drops the oldest ones.

**Stop** stops reading and keeps the table. While reading runs, the **Read** button becomes **Restart**: it starts again with the current settings and clears the table. If the connection to the broker is lost, the app reconnects by itself and shows that in the state line.

Messages from different partitions are mixed in the table. Kafka keeps the order only within one partition. The `P` column shows the partition, and `Offset` shows the position in it. Select `UTC` to show times in UTC instead of local time.

<a id="start-position"></a>

## Start position

**Start position** sets where reading begins in each partition:

- **Latest N** — the last N messages. N is split evenly between the partitions, so with 100 messages and 4 partitions the app takes up to 25 from each.
- **From the beginning** — from the oldest message that is still stored.
- **From offset** — from the same offset in each partition. If a partition has no such offset, reading starts at its beginning or end. This mode is most useful with one partition.
- **From time** — from the first message with a timestamp at or after the given local time. In partitions without newer messages, reading starts at the end.
- **Last X minutes** — the same, counted back from now.

The field with **partitions: all** limits reading to some partitions. Enter partition numbers separated by commas, for example `0,2`. Leave it empty to read all partitions.

<a id="filters"></a>

## Filters

Filters show only matching messages. The app still reads all messages from the start position and checks each one, so filters do not make reading faster. All filled fields must match.

- **Key contains** and **Value contains** — text in the key or the value.
- `regex` — treat these fields, and the header value, as regular expressions.
- `Aa` — match case. By default, case does not matter.
- **Header** — the message has a header with this exact name. **Header value** also checks its value.
- `JSONPath` — a condition on a JSON value, for example `$.order.status == "FAILED"`. Without `==` or `!=`, the path must simply exist, for example `$.error`. Messages whose value is not valid JSON do not match.
- **Time from** and **Time to** — the message timestamp is within this range, in local time. Both ends are included.

Filters check the text shown in the table, after decoding. If the value format is `hex`, the value filter searches the hex text. Filters apply when you click **Read**; after changing them, read again.

A filter does not move the start position. For example, **Latest N** with a filter finds matches only among the last N messages. To search the whole topic, start from **From the beginning**.

<a id="formats"></a>

## Formats

**Format** sets how keys and values are decoded, separately for the **key** and the **value**. Only the display changes, not the data.

- `auto` — the default. Valid JSON objects and arrays are shown as JSON, other UTF-8 text as a string, binary data as hex.
- `string` — text in UTF-8. Bytes that are not valid UTF-8 are replaced with `�`.
- `json` — JSON with indents in the message details. If the value is not valid JSON, the details show the error.
- `hex` — every byte as two hex digits.
- `base64` — the bytes in Base64.

The details show which format was actually used, for example `json` when `auto` detected JSON. Header values are shown as text; binary header values are shown as `0x` followed by hex. A missing key or value is shown as `null`.

Very large keys and values are shown only partly, about the first 1 MB of text. This also applies to `hex` and `base64`. The message details then show a warning.

<a id="message-details"></a>

## Message details

Click a row to open the message details next to the table. At the top are the partition, the offset and the time. Below:

- **Value** and **Key** — the full text, with the format and size in bytes. JSON is shown with indents. **Copy** copies the text.
- **Headers** — a table of header names and values.

The table columns are `P`, `Offset`, **Time**, **Key**, **Value**, **Headers** (their number) and **Size** (the key and value size in bytes). The time is the message timestamp. Depending on the topic setting `message.timestamp.type`, it is set by the producer or by the broker.

<a id="export-messages"></a>

## Copy messages

Glafira does not save browsed messages to a file. To take a message out of the app, copy it:

- **Copy all as JSON** copies the whole message as JSON: topic, partition, offset, timestamp, key, value, headers and size. Key and value are included as decoded, with their format.
- **Copy** next to **Value** or **Key** copies only that text.

The copied text uses the format chosen for reading. To copy binary data exactly, read it as `hex` or `base64`. Very large keys and values are copied only partly, as shown, even in `hex` and `base64`. See [Formats](browse.md#formats).

<a id="open-in-producer"></a>

## Open in Producer

**Resend** opens the message in the **Producer** of the same connection. The form gets the topic, key, value, headers and formats of the message. The partition is left to Kafka. Placeholders in the message text are not replaced: the copy is sent as it was read.

Nothing is sent right away. Check the form, change it if needed, and send it from the Producer. See [Producer](producer.md).

The button is disabled in two cases, and the reason is shown:

- the key or value was shown only partly because it is too large;
- the text was decoded with losses because it is not valid UTF-8. Read it as `hex` or `base64` to send an exact copy.

On **Read-only** connections, the button is hidden.

<a id="reset-group-here"></a>

## Reset a group to a message

**Reset group here…** prepares an offset reset so that a consumer group starts reading from this message.

1. In the message details, click **Reset group here…**.
2. Choose a group in the list. Only existing groups are listed.
3. Click **Go to reset**.

The group tab opens with the reset form filled in. **Strategy** is **Specific offset** with the offset of the message. **Scope** is **Partitions** with the topic and partition of the message. Nothing changes yet: click **Preview** to see the new offsets, then apply the reset there. The group must have no active members. See [Consumer groups](groups.md).

On **Read-only** connections, the button is hidden.
