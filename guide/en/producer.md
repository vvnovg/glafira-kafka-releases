# Producer

The **Producer** sends messages to Kafka: one at a time from a form, or many at once. Open it from the connection in the **Connections** panel, or with **Send message** on a topic tab, which fills in the topic. Each connection has its own Producer tab.

On **Read-only** connections, the Producer opens with the warning **The connection is read-only — sending is unavailable.** The send buttons are hidden, but templates and history can still be viewed.

<a id="send-message"></a>

## Send a message

1. Enter the **Topic**. The field suggests the topics of the cluster, without internal topics.
2. Optionally, enter a **Partition** number. When the field shows **auto**, the partition is chosen by the key: messages with the same key go to the same partition, as with the Java client. Messages without a key go to a random partition.
3. Fill in the **Key** and the **Value**, and choose their formats. See [Key and value formats](producer.md#key-value-formats).
4. Optionally, add headers.
5. Click **Send**.

After a successful send, a line below the form shows the partition, the offset and the time of the message. With `acks` set to `0`, the broker does not return an offset, and `—` is shown instead.

If the partition number is too large, or the broker does not confirm the message within the connection timeout, the app shows an error. A topic that does not exist also gives an error, unless the broker creates topics automatically (`auto.create.topics.enable`). With auto-creation off, this error can take up to about 30 s to appear (the connection timeout). Errors in the form, such as invalid JSON or an unknown placeholder, are shown before anything is sent.

On a prod connection, sending asks you to type the connection name and confirm with **Send to prod**. See [Environments and prod](connections.md#environments).

**Clear form** empties the form and returns the producer settings to their defaults. The topic stays.

<a id="key-value-formats"></a>

## Key and value formats

**Key format** and **Value format** set how the text in the field is turned into bytes:

- `string` — the text as is, in UTF-8.
- `json` — the text is checked to be valid JSON and sent as typed, without reformatting.
- `hex` — two hex digits per byte, for example `48656c6c6f`. Spaces and line breaks are ignored.
- `base64` — bytes in standard Base64.
- `null` — no key or no value at all. The field is empty, and typing text switches the format to `string`.

The default is `null` for the key and `json` for the value. To send a message with a key, just type it: the format changes to `string`. An empty string is still a key, not a missing one: if you choose `string` and leave the field empty, all such messages get the same hash and go to one partition.

A message with a `null` value is a tombstone. In a topic with `cleanup.policy=compact`, it tells Kafka to remove older messages with the same key.

<a id="headers"></a>

## Headers

Headers are name and value pairs sent with the message. Click **Add header** to add a row, then fill in **Name**, **Value** and **Format**. The `×` button removes the row.

The format of a header value is `string` (text in UTF-8) or `hex` (bytes as hex digits). Rows with an empty name are not sent. Kafka allows several headers with the same name, and they are all sent.

Placeholders work in header values, but not in header names.

<a id="placeholders"></a>

## Placeholders

Placeholders are replaced with values at every send. They work in the key, the value and the header values. The list is shown below the headers:

- `{{uuid}}` — a new random UUID, for example `0b6f6f2c-3c1e-4a8e-9a54-2f0d6f1d7c21`.
- `{{now}}` — the current time in UTC in ISO 8601 format with milliseconds, for example `2026-10-15T12:30:00.000Z`.
- `{{nowMillis}}` — the current time in milliseconds since 1970-01-01 UTC.
- `{{seq}}` — the number of the copy in a batch, starting at 0. For a single send, it is `0`.
- `{{randomInt 1 100}}` — a random integer between the two numbers, both included.
- `{{randomString 8}}` — a random string of Latin letters and digits of the given length, up to 100000.
- `{{env NAME}}` — the value of the environment variable `NAME`. If it is not set, nothing is sent and an error is shown.

Spaces inside the braces are allowed. An unknown placeholder or a missing `}}` stops the send with an error. To send `{{` as text, write `\{{`.

With the `json` format, the JSON is checked after the placeholders are replaced. Placeholders that produce text need quotes: `{"id": "{{uuid}}"}`. Numbers do not: `{"n": {{seq}}}`.

`{{env NAME}}` reads the environment of the Glafira process. If you start Glafira from the Dock or the Start menu, variables set only in your terminal profile may be missing.

When a message comes from [Open in Producer](browse.md#open-in-producer) or from the history, every `{{` in it is written as `\{{`. The message is then sent exactly as it was, without replacing anything.

<a id="templates"></a>

## Templates

A template saves the whole form: topic, partition, key, value, headers, formats and producer settings. Placeholders are saved as written and are replaced at each send.

To save a template, click **Save as template**, enter the **Template name** and click **Save**. The topic may be empty. By default, the template belongs to this connection. Select **For all connections** to show it on every connection; such templates are marked **(all connections)** in the list.

To use a template, choose it in the **Template** list. The form is replaced with the template. You can edit the form before sending; the template does not change.

Saving always creates a new template. To change one, load it, edit the form, save it under a new name and delete the old one. To delete the selected template, click **Delete template** and confirm with **Delete**.

Templates of a connection are deleted together with the connection.

<a id="history"></a>

## History

**History (last 100)** lists the last 100 single sends of this connection, newest first. Messages from batch sends are not added. Each row shows the time, the topic, the partition and offset, and the start of the key.

History stores the message as it was sent, with the placeholders already replaced. **Resend** loads it into the form together with its producer settings. Nothing is sent until you click **Send**. The copy is sent exactly as stored: for example, the same UUID is sent again.

A key, value or header value longer than 64 KB is stored only partly. For such rows, **Resend** is disabled.

History is stored on this computer in plain text, including the values of `{{env NAME}}`. Do not send secrets through the Producer if this matters. The history of a connection is deleted together with the connection.

<a id="batch-copies"></a>

## N copies of the form

**Batch send** below the form sends many messages in one run. In the **N copies of the form** mode, it sends the form the given number of times.

1. Fill in the form as for a single send.
2. Select **N copies of the form** and enter the number of **Copies**, from 1 to 1000000.
3. Optionally, set the **Rate**. See [Send rate](producer.md#batch-rate).
4. Click **Start**.

Placeholders are replaced separately for each copy. `{{seq}}` is the copy number, from 0 to N−1, so each message can get its own key, for example `order-{{seq}}`. Placeholder errors are checked on the first copy before the run starts.

During the run, the panel shows how many messages were sent and how many failed. At the end, it shows **done**. Errors are listed with the copy number, counted from 1. The list shows the first 20 errors; if there are more, a note below gives the total.

**Cancel** stops the run. Messages that have not left the app yet are not sent, and the panel shows their count. The app waits up to 2 seconds for the broker to confirm messages already sent. Messages that are still not confirmed count as errors, because it is unknown whether they were written. Closing the Producer tab also stops the run.

On a prod connection, the run asks for the connection name and is confirmed with **Start on prod**.

<a id="batch-file"></a>

## Send from a file

In the **from file** mode, each line or row of a file becomes a message. Supported files are JSON Lines (`.jsonl`, `.ndjson`) and CSV (`.csv`).

1. Enter the **Topic** in the form above. All messages go to this topic.
2. Select **from file** and click **Choose file…**.
3. Click **Start**.

The key, value, headers and partition from the form are not used, and placeholders are not replaced. The producer settings of the form apply.

In JSON Lines, each line is one JSON object, for example `{"key": "k1", "value": {"id": 1}, "headers": {"source": "test"}, "partition": 0}`. All fields are optional:

- `key` and `value` — a JSON string is sent as its text, without quotes. Any other JSON value, such as an object or a number, is sent as JSON text. `null` or a missing field means no key or no value.
- `headers` — an object of string values.
- `partition` — a partition number. Without it, the partition is chosen by the key.

Empty lines are skipped. The file must be in UTF-8.

In CSV, the first row holds the column names. The `value` column is required; `key`, `partition` and `header.NAME` columns are optional, for example `header.source`. An empty `key` cell means no key, and an empty header cell means no such header. An empty `value` cell sends an empty value, not `null`, so a tombstone cannot be sent from CSV.

The file is read as a stream, so the total number is not known in advance and is shown as `—`. A line or record larger than 16 MB, invalid JSON or a negative partition counts as an error for that line, and the run goes on with the next line. Errors are shown with the line number.

<a id="batch-rate"></a>

## Send rate

**Rate** limits a batch to the given number of messages per second. Enter an integer of 1 or more. An empty field means **unlimited**.

The messages are spread evenly over time: at 10 messages per second, one message leaves about every 100 ms. Use the limit to avoid overloading a shared cluster, or to imitate a steady load.

Without a limit, the app sends as fast as the broker accepts. The rate applies to both batch modes.

<a id="producer-settings"></a>

## Producer settings

**Producer settings** apply to single sends and batches. They are saved in templates and history.

`acks` sets how many replicas must confirm a write:

- `all` — the default. The leader waits for all in-sync replicas. This is the safest choice. If fewer replicas are in sync than the topic setting `min.insync.replicas` requires, the write fails.
- `1` — only the leader confirms. Faster, but a message can be lost if the leader fails before the followers copy it.
- `0` — the producer does not wait for a confirmation. Messages can be lost, and no offset is returned.

**compression** compresses message batches: `none` (the default), `gzip`, `snappy`, `lz4` or `zstd`. Compression saves network traffic and disk space at the cost of some CPU. `zstd` and `lz4` are common choices. For a single small message, the gain is small.

**idempotence** sets `enable.idempotence`. It prevents duplicates when the client retries a send after a network error. It requires `acks` set to `all`; with other values, the app shows an error and sends nothing.
