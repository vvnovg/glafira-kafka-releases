# 故障排除

本章说明常见的连接错误以及每种错误需要检查的内容，还介绍日志的位置以及支持请求中应包含哪些信息。在 macOS 上，可以通过“帮助”菜单中的**故障排除**打开本章。

错误消息包含简短的文本，并且在大多数页面上，括号中还会显示错误代码，例如 `TIMEOUT`。展开**详细信息**可查看 Kafka 客户端（librdkafka）的原始消息，其中通常会指明具体的 Broker 地址和原因。

<a id="cannot-connect"></a>

## 无法连接

先在连接表单中点击**测试连接**，参见[测试连接](connections.md#test)。测试会在第一个失败的步骤处停止，而该步骤指明了应检查的方向：

- **配置**：问题出在表单本身，例如地址缺少端口、证书文件缺失或无法读取、证书密码错误。错误会指明相应字段。证书文件问题参见 [TLS 错误](troubleshooting.md#tls-errors)。
- **DNS / TCP**：计算机无法访问 Bootstrap 服务器。参见 [DNS 和 TCP](troubleshooting.md#dns-tcp)。
- **Kafka（TLS、SASL、元数据）**：网络正常，但 TLS 握手、SASL 登录或元数据请求失败。参见 [TLS 错误](troubleshooting.md#tls-errors)、[SASL 错误](troubleshooting.md#sasl-errors)和[超时](troubleshooting.md#timeouts)。

最常见的原因是协议与端口不匹配。每个 Broker 监听器都有自己的端口和协议。请向你的 Kafka 管理员询问哪个端口对应哪种协议，参见[安全协议](connections.md#security)。

如果测试通过，但读取或发送失败，原因通常在于 Broker 地址。客户端从 Bootstrap 服务器获取 Broker 列表，然后按每个 Broker 公布的地址（`advertised.listeners`）连接到该 Broker。如果这些地址是内部地址，例如 Docker 或 Kubernetes 主机名，你的计算机就无法访问它们。这些地址列在集群概览的 Broker 表中，参见[集群概览](topics.md#cluster-overview)。请向管理员申请一个可从外部访问的监听器。

<a id="dns-tcp"></a>

## DNS 和 TCP

**所有 Bootstrap 服务器均无法通过 TCP 访问**表示应用无法与 **Bootstrap 服务器**中的任何地址建立 TCP 连接。每个地址最多尝试 5 秒。详细信息中会显示最后尝试的地址和原因。详细信息中的 `DNS` 表示无法解析主机名。

请检查：

- 主机名和端口，不要带 `https://` 之类的协议前缀；
- 如果集群位于专用网络中，VPN 是否已开启；
- 防火墙是否阻止了该端口；
- 对于 Docker 中的 Broker，端口是否已发布到主机，例如 `-p 9092:9092`。

要在应用之外检查端口，可在 macOS 或 Linux 上运行 `nc -vz <host> <port>`，或在 Windows 的 PowerShell 中运行 `Test-NetConnection <host> -Port <port>`。

TCP 步骤通过后出现**无法连接到 Broker**，通常表示某个 Broker 关闭了连接，或之后无法访问。另请检查 Broker 地址，参见[无法连接](troubleshooting.md#cannot-connect)。

<a id="tls-errors"></a>

## TLS 错误

在 **Kafka（TLS、SASL、元数据）** 步骤中，TLS 握手期间的错误：

- **该地址上的 Broker 接受不使用 TLS 的连接，请检查端口或选择不带 SSL 的协议**。
- **Broker 证书不是由受信任的 CA 签发的，请检查“受信任的 CA”**。请将**受信任的 CA 证书**设为为 Broker 证书签名的 CA 的文件，该文件可向管理员索取。字段为空时，只信任系统根证书。
- **Broker 主机名与其证书不匹配**。请使用证书签发时所对应的名称连接，而不是 IP 地址或别名。仅在测试环境中，可以关闭**验证主机名**。
- **Broker 拒绝了客户端证书**。Broker 要求提供客户端证书，或不信任你使用的证书。请检查是否已开启**客户端证书（mTLS）**，以及证书是否为此集群签发。参见[客户端证书（mTLS）](connections.md#mtls)。
- **TLS 连接错误**：其他 TLS 问题。展开**详细信息**可查看原因。

如果选择了不带 SSL 的协议却仍出现 TLS 错误，说明该端口很可能是 TLS 监听器。此时详细信息中会显示 `connecting to a SSL listener?`。请选择 `SSL` 或 `SASL_SSL`，或使用不带 TLS 的监听器的端口。

证书文件的问题会在应用连接之前，于**配置**步骤中报告。消息以字段名称开头：受信任的 CA、客户端证书或客户端私钥。常见情况：

- 路径不是绝对路径，或文件无法读取。请输入完整路径，或点击**浏览…**，然后选择文件。
- **客户端私钥：私钥已加密，请指定私钥密码**、**客户端私钥：私钥密码错误**、**客户端证书：PKCS\#12 存储密码错误**或 **JKS 存储密码错误（或文件已损坏）**：请在表单中输入正确的密码。
- **客户端私钥与客户端证书不匹配**：私钥文件属于另一个证书。
- 客户端证书已过期或尚未生效：消息中会给出日期。请申请新证书。
- PKCS#12 存储使用了旧算法，或存储为 JCEKS 格式。请按[证书格式](connections.md#certificate-formats)中所示转换文件。

<a id="sasl-errors"></a>

## SASL 错误

**身份验证失败**表示 Broker 未接受 SASL 登录。请检查：

- 用户名和密码。编辑已保存的连接时，密码字段留空会保留已保存的密码。如果不确定密码是否正确，请重新输入；
- **SASL 机制**。它必须是创建你的用户时所针对的机制。SCRAM 凭据需要分别为 `SCRAM-SHA-256` 和 `SCRAM-SHA-512` 创建，`PLAIN` 用户又是单独配置的；
- 协议：`SASL_SSL` 和 `SASL_PLAINTEXT` 通常使用不同的端口。

**未设置该连接的 SASL 密码**：此连接没有保存密码，例如在导入之后。请打开该连接，输入密码并保存。参见[密码](connections.md#passwords)。

**无法访问系统密钥存储**：应用无法从系统密钥存储中读取密码。在 macOS 上，钥匙串询问时请允许访问。在 Linux 上，请检查你的会话中是否运行着 GNOME Keyring、KWallet 或其他 Secret Service 提供程序。

**无权执行该操作**则不同：登录已成功，但该 Kafka 用户没有执行此操作的权限（ACL），例如读取主题或组。请向管理员申请权限。请注意，对于你无权查看描述信息的主题，Kafka 不会报告错误：它们只是不会出现在主题列表中。

<a id="timeouts"></a>

## 超时

**操作超时**表示 Broker 未及时响应。典型原因是网络或 VPN 较慢、集群过载，或 Broker 地址无法访问，参见[无法连接](troubleshooting.md#cannot-connect)。

TLS 或 SASL 问题也可能表现为超时。应用会尝试找出真正的原因并改为显示该原因，但并非总能做到。展开**详细信息**：`SSL handshake failed` 或 `SASL authentication error` 之类的消息会指明原因。

如果只是网络较慢，请在连接表单中调大**请求超时（ms）**，默认为 `30000` ms。该值适用于对 Broker 的请求、发送消息和读取。连接测试使用自己的时限（每个 TCP 地址 5 秒，Kafka 步骤 10 秒），因此调大该值不会影响测试。

<a id="read-only-blocked"></a>

## 操作不可用或被拒绝

如果按钮被隐藏或禁用，或者操作被拒绝，请检查以下原因：

- 连接为只读。只读连接的名称旁带有锁形图标。会更改集群的按钮会被隐藏或禁用。如果此类请求仍到达了应用内核，例如在页面打开期间开启了该模式，错误会提示连接为只读。如需进行更改，请打开该连接，取消选中**只读**并保存。参见[只读](connections.md#read-only)。
- 连接为 prod。更改时会要求输入连接名称，名称一致之前不会执行操作。参见[环境与 prod](connections.md#environments)。
- 消费者组有活动成员。只有在组中没有成员时，才能重置偏移量或删除组。参见[重置偏移量](groups.md#reset-offsets)。
- 该设置由你的组织管理。策略可以设置界面语言和更新检查，这些都无法在应用中更改。参见[由组织设置的选项](getting-started.md#managed-settings)。
- Kafka 以**无权执行该操作**拒绝了操作。这是集群中的权限问题，与 Glafira 设置无关。参见 [SASL 错误](troubleshooting.md#sasl-errors)。

<a id="logs"></a>

## 日志

应用会将自身的工作情况和 Kafka 客户端错误写入日志。确切的文件夹显示在**关于**页面的“**日志：**”一项中。默认位置：

- macOS：`~/Library/Logs/io.github.vvnovg.glafira.kafka`
- Windows：`%LOCALAPPDATA%\io.github.vvnovg.glafira.kafka\logs`
- Linux：`~/.local/share/io.github.vvnovg.glafira.kafka/logs`

当前文件为 `app.log`。达到 5 MB 时，它会被重命名为 `app.1.log`，更早的文件依次变为 `app.2.log`，以此类推。应用最多保留五个文件，并删除最旧的文件。

日志中不会写入密码，但会写入 Broker 地址和 Kafka 错误文本。无需手动收集这些文件：诊断归档中已包含它们，参见[诊断归档](updates.md#diagnostics)。

<a id="support"></a>

## 联系技术支持

联系技术支持之前，请先运行**测试连接**，并记下哪个步骤失败。然后将请求发送给为你提供 Glafira 支持的人员，例如你的管理员。请附上：

- 诊断归档，参见[诊断归档](updates.md#diagnostics)。其中包含版本、build、日志以及不含密码的连接配置；
- 你执行了哪些操作、预期结果是什么、实际发生了什么，请逐步说明；
- 完整的错误文本以及**详细信息**中的内容。这些文本可以选中并复制；
- 如果问题出在界面上，请附上截图；
- 错误发生的时间，以便在日志中查找。

如果无法创建归档，请提供**关于**页面上的**版本**和 **Build** 的值，并附上日志文件夹中的日志文件。
