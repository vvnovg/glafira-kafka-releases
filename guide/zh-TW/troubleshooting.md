# 疑難排解

本章說明常見的連線錯誤，以及每種錯誤該檢查什麼，也會說明日誌的位置，以及支援要求中應包含哪些內容。在 macOS 上，可以從「輔助說明」選單中的**疑難排解**開啟本章。

錯誤訊息包含一段簡短的文字，在大多數畫面上還會在括號中附上錯誤代碼，例如 `TIMEOUT`。展開**詳細資訊**可查看 Kafka 用戶端（librdkafka）的原始訊息，其中通常會指出確切的 Broker 位址和原因。

<a id="cannot-connect"></a>

## 無法連線

請先在連線表單中使用**測試連線**，請參閱[測試連線](connections.md#test)。測試會在第一個失敗的步驟停止，而該步驟會指出該從哪裡著手：

- **設定**：問題出在表單本身，例如位址沒有連接埠、憑證檔案不存在或無法讀取、憑證密碼錯誤。錯誤會指出是哪個欄位。憑證檔案的問題請參閱 [TLS 錯誤](troubleshooting.md#tls-errors)。
- **DNS / TCP**：電腦無法連到 Bootstrap 伺服器。請參閱 [DNS 與 TCP](troubleshooting.md#dns-tcp)。
- **Kafka（TLS、SASL、中繼資料）**：網路正常，但 TLS 交握、SASL 登入或中繼資料要求失敗。請參閱 [TLS 錯誤](troubleshooting.md#tls-errors)、[SASL 錯誤](troubleshooting.md#sasl-errors)和[逾時](troubleshooting.md#timeouts)。

最常見的原因是通訊協定與連接埠不符。每個 Broker 監聽器都有自己的連接埠和通訊協定。請向你的 Kafka 管理員詢問哪個連接埠對應哪個通訊協定，請參閱[安全性通訊協定](connections.md#security)。

如果測試通過，但讀取或傳送失敗，原因通常出在 Broker 位址。用戶端會從 Bootstrap 伺服器取得 Broker 清單，然後依各 Broker 公告的位址（`advertised.listeners`）連線到每個 Broker。如果這些位址是內部位址，例如 Docker 或 Kubernetes 的主機名稱，從你的電腦就無法連到。這些位址列在叢集概觀的 Broker 表格中，請參閱[叢集概觀](topics.md#cluster-overview)。請向你的管理員索取可從外部連線的監聽器。

<a id="dns-tcp"></a>

## DNS 與 TCP

「**所有 Bootstrap 伺服器都無法透過 TCP 連線**」表示應用程式無法與 **Bootstrap 伺服器**中的任何位址建立 TCP 連線。每個位址最多嘗試 5 秒。詳細資訊會顯示最後嘗試的位址和原因。詳細資訊中出現 `DNS` 表示無法解析主機名稱。

請檢查：

- 主機名稱和連接埠，不要加上 `https://` 之類的通訊協定前置字元；
- 如果叢集位於私人網路中，VPN 是否已開啟；
- 防火牆是否封鎖了該連接埠；
- 如果 Broker 在 Docker 中，連接埠是否已發布到主機，例如 `-p 9092:9092`。

若要在應用程式之外檢查連接埠，請在 macOS 或 Linux 上執行 `nc -vz <host> <port>`，或在 Windows 的 PowerShell 中執行 `Test-NetConnection <host> -Port <port>`。

TCP 步驟通過後出現「**無法連線到 Broker**」，通常表示有 Broker 關閉了連線，或之後無法連到。也請檢查 Broker 位址，請參閱[無法連線](troubleshooting.md#cannot-connect)。

<a id="tls-errors"></a>

## TLS 錯誤

在「**Kafka（TLS、SASL、中繼資料）**」步驟中，TLS 交握期間出現的錯誤：

- **此位址上的 Broker 接受未使用 TLS 的連線，請檢查連接埠，或選擇不含 SSL 的通訊協定**。
- **Broker 憑證並非由受信任的 CA 簽署，請檢查「受信任的 CA」**。請將**受信任的 CA 憑證**設為簽署 Broker 憑證的 CA 檔案，你的管理員可以提供這個檔案。欄位留空時，只會信任系統根憑證。
- **Broker 主機名稱與其憑證不符**。請使用憑證核發時所用的名稱連線，而不是 IP 位址或別名。僅限測試環境中，可以關閉**驗證主機名稱**。
- **Broker 拒絕了用戶端憑證**。Broker 要求用戶端憑證，或不信任你使用的憑證。請確認已開啟**用戶端憑證（mTLS）**，且憑證是為這個叢集核發的。請參閱[用戶端憑證（mTLS）](connections.md#mtls)。
- **TLS 連線錯誤**：其他 TLS 問題。展開**詳細資訊**可查看原因。

如果你選擇了不含 SSL 的通訊協定，卻仍看到 TLS 錯誤，該連接埠很可能是 TLS 監聽器。此時詳細資訊會顯示 `connecting to a SSL listener?`。請選擇 `SSL` 或 `SASL_SSL`，或改用不含 TLS 的監聽器的連接埠。

憑證檔案的問題會在應用程式連線之前，於**設定**步驟回報。訊息會以欄位名稱開頭：受信任的 CA、用戶端憑證或用戶端私密金鑰。常見情況：

- 路徑不是絕對路徑，或無法讀取檔案。請輸入完整路徑，或按一下**瀏覽…**，選擇檔案。
- **用戶端私密金鑰：金鑰已加密，請指定金鑰密碼**、**用戶端私密金鑰：金鑰密碼錯誤**、**用戶端憑證：PKCS\#12 儲存區密碼錯誤**或 **JKS 儲存區密碼錯誤（或檔案已損毀）**：請在表單中輸入正確的密碼。
- **用戶端私密金鑰與用戶端憑證不符**：金鑰檔案屬於另一個憑證。
- 用戶端憑證已過期或尚未生效：訊息會顯示日期。請索取新的憑證。
- PKCS#12 儲存區使用舊式演算法，或儲存區為 JCEKS 格式。請依[憑證格式](connections.md#certificate-formats)中的說明轉換檔案。

<a id="sasl-errors"></a>

## SASL 錯誤

「**驗證失敗**」表示 Broker 不接受 SASL 登入。請檢查：

- 使用者名稱和密碼。編輯已儲存的連線時，密碼欄位留空會保留已儲存的密碼。如果不確定密碼是否正確，請重新輸入；
- **SASL 機制**。必須是建立你的使用者時所用的機制。SCRAM 認證資訊會分別為 `SCRAM-SHA-256` 和 `SCRAM-SHA-512` 建立，`PLAIN` 使用者則另外設定；
- 通訊協定：`SASL_SSL` 和 `SASL_PLAINTEXT` 通常使用不同的連接埠。

**未設定此連線的 SASL 密碼**：這個連線沒有儲存密碼，例如在匯入之後。請開啟連線，輸入密碼並儲存。請參閱[密碼](connections.md#passwords)。

**無法存取系統金鑰儲存區**：應用程式無法從系統金鑰儲存區讀取密碼。在 macOS 上，鑰匙圈詢問時請允許存取。在 Linux 上，請確認你的工作階段中有 GNOME Keyring、KWallet 或其他 Secret Service 提供者正在執行。

「**沒有執行此操作的權限**」則不同：登入已成功，但 Kafka 使用者沒有執行此操作的權限（ACL），例如讀取主題或群組。請向你的管理員申請權限。請注意，對於你無權描述的主題，Kafka 不會回報錯誤，這些主題只是不會出現在主題清單中。

<a id="timeouts"></a>

## 逾時

「**操作逾時**」表示 Broker 未及時回應。常見原因是網路或 VPN 速度慢、叢集負載過重，或無法連到 Broker 位址，請參閱[無法連線](troubleshooting.md#cannot-connect)。

TLS 或 SASL 問題可能看起來像逾時。應用程式會嘗試找出真正的原因並改為顯示，但並非每次都能做到。請展開**詳細資訊**：`SSL handshake failed` 或 `SASL authentication error` 之類的訊息會指出原因。

如果只是網路較慢，請在連線表單中調高「**要求逾時（ms）**」。預設為 `30000` ms。此值適用於對 Broker 的要求、傳送訊息和讀取。連線測試使用自己的限制（每個 TCP 位址 5 秒，Kafka 步驟 10 秒），因此調高此值不會影響測試。

<a id="read-only-blocked"></a>

## 動作無法使用或遭到拒絕

如果按鈕隱藏或停用，或動作遭到拒絕，請檢查以下原因：

- 連線為唯讀。名稱旁的鎖頭圖示表示唯讀。會變更叢集的按鈕會隱藏或停用。如果這類要求仍送到應用程式核心，例如在畫面開啟時開啟了唯讀模式，錯誤會說明連線為唯讀。若要進行變更，請開啟連線，取消勾選**唯讀**並儲存。請參閱[唯讀](connections.md#read-only)。
- 連線為正式環境。變更時會要求輸入連線名稱，名稱相符之前不會執行動作。請參閱[環境與正式環境](connections.md#environments)。
- 消費者群組有作用中的成員。只有群組沒有成員時，才能重設偏移量和刪除群組。請參閱[重設偏移量](groups.md#reset-offsets)。
- 設定由你的組織管理。原則可以設定介面語言和更新檢查，這些都無法在應用程式中變更。請參閱[由組織設定的選項](getting-started.md#managed-settings)。
- Kafka 以「**沒有執行此操作的權限**」拒絕操作。這是叢集中的權限問題，與 Glafira 設定無關。請參閱 [SASL 錯誤](troubleshooting.md#sasl-errors)。

<a id="logs"></a>

## 日誌

應用程式會將自身的運作情形和 Kafka 用戶端錯誤寫入日誌。確切的資料夾顯示在**關於**畫面的「**日誌：**」項目中。預設位置：

- macOS：`~/Library/Logs/io.github.vvnovg.glafira.kafka`
- Windows：`%LOCALAPPDATA%\io.github.vvnovg.glafira.kafka\logs`
- Linux：`~/.local/share/io.github.vvnovg.glafira.kafka/logs`

目前的檔案是 `app.log`。檔案達到 5 MB 時，會重新命名為 `app.1.log`，較舊的檔案則依序改為 `app.2.log`，以此類推。應用程式最多保留五個檔案，並刪除最舊的檔案。

日誌中不會寫入密碼，但會寫入 Broker 位址和 Kafka 錯誤的文字。你不需要手動收集這些檔案，診斷壓縮檔已包含它們，請參閱[診斷壓縮檔](updates.md#diagnostics)。

<a id="support"></a>

## 聯絡支援人員

聯絡支援人員之前，請先執行**測試連線**，並記下哪個步驟失敗。然後將要求傳送給為你提供 Glafira 支援的人，例如你的管理員。請附上：

- 診斷壓縮檔，請參閱[診斷壓縮檔](updates.md#diagnostics)。其中包含版本、build、日誌，以及不含密碼的連線設定檔；
- 你做了什麼、預期的結果和實際發生的情況，逐步說明；
- 完整的錯誤文字，以及**詳細資訊**的內容。你可以選取並複製這些文字；
- 如果問題出在介面上，請附上螢幕截圖；
- 錯誤發生的時間，以便在日誌中找到它。

如果無法建立壓縮檔，請提供**關於**畫面中的**版本**和 **Build** 值，並附上日誌資料夾中的日誌檔案。
