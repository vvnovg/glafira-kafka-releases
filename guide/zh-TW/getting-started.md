# 開始使用

Glafira for Apache Kafka 是 Kafka 叢集的桌面用戶端，可以瀏覽主題和消費者群組、讀取和傳送訊息，以及重設偏移量。本章說明安裝、第一次啟動、用來練習的本機 Kafka，以及介面語言。

<a id="install-macos"></a>

## 在 macOS 上安裝

Glafira 可在 macOS 11 或更新版本上執行。請從發行頁面下載安裝程式：https://github.com/vvnovg/glafira-kafka-releases/releases

依你的 Mac 選擇檔案：Apple 晶片（M1 及更新機型）選 `aarch64`，Intel 選 `x64`。

1. 開啟 `.dmg` 檔案。
2. 將 Glafira for Apache Kafka 拖曳到「應用程式」資料夾。
3. 從「應用程式」或「啟動台」啟動應用程式。

也可以使用 `.pkg` 安裝程式，它會把應用程式安裝到 `/Applications`，供所有使用者使用。

Beta 版的 build 尚未經過 Apple 簽署，因此 macOS 會封鎖第一次啟動。若要允許啟動：

1. 先嘗試開啟應用程式一次，然後關閉警告。
2. 開啟「系統設定」→「隱私權與安全性」。
3. 向下捲動到與 Glafira for Apache Kafka 有關的訊息，按一下「強制打開」。如果系統要求，請輸入密碼確認。

在 macOS 14 及更早版本上，也可以在應用程式上按右鍵，選擇「打開」，然後再按一次「打開」。

如果這樣仍無效，請在「終端機」中執行一次以下指令，移除隔離標記：

```bash
xattr -dr com.apple.quarantine "/Applications/Glafira for Apache Kafka.app"
```

如果下載的 `.dmg` 本身打不開，請先移除它的標記：

```bash
xattr -d com.apple.quarantine ~/Downloads/Glafira_for_Apache_Kafka_*.dmg
```

從應用程式內安裝的更新不需要這些步驟。

<a id="install-windows"></a>

## 在 Windows 上安裝

從發行頁面下載 `.msi` 安裝程式（`x64`）：https://github.com/vvnovg/glafira-kafka-releases/releases

1. 執行 `.msi` 檔案。
2. 在 Windows 提示中確認安裝。應用程式會安裝給這台電腦的所有使用者，因此需要系統管理員權限。
3. 從「開始」功能表啟動 Glafira for Apache Kafka。

Beta 版的 build 尚未簽署。如果 SmartScreen 封鎖了安裝程式，請按一下「其他資訊」，再按「仍要執行」。也可以在執行前先用 PowerShell 解除封鎖檔案：

```powershell
Unblock-File -Path "$env:USERPROFILE\Downloads\Glafira_for_Apache_Kafka_*_x64_en-US.msi"
```

<a id="install-linux"></a>

## 在 Linux 上安裝

提供 `x86_64` 版本，共有三種格式。請從發行頁面下載適合你的發行版的格式：https://github.com/vvnovg/glafira-kafka-releases/releases

- Debian、Ubuntu 及其衍生版：`sudo apt install ./<file>.deb`
- Fedora、RHEL 及其衍生版：`sudo dnf install ./<file>.rpm`
- 任何發行版：用 `chmod +x <file>.AppImage` 讓 AppImage 可以執行，然後執行它。

安裝套件後，從應用程式選單啟動，或在終端機中執行 `glafira-kafka`。

Glafira 會將密碼儲存在 Secret Service 中（例如 GNOME Keyring 或 KWallet）。請確認你的工作階段中有其中一個正在執行，否則應用程式無法儲存連線的密碼。

<a id="first-run"></a>

## 第一次啟動

視窗分成兩部分：左側是**連線**面板，右側是分頁。你開啟的每個畫面（例如主題清單或 Producer）都有自己的分頁。

還沒有任何連線時，面板會顯示簡短提示，分頁區域會建議新增連線。開始使用：

1. 按一下**連線**面板標題列中的 `+`（**新增連線**）。
2. 填寫**名稱**和 **Bootstrap 伺服器**，例如 `localhost:9092`。
3. 按一下**測試連線**檢查設定，然後按**儲存**。

儲存後會開啟叢集概觀。面板中每個連線下方都有**叢集概觀**、**主題**、**消費者群組**、**Producer** 和**消費者工作階段**的連結。所有連線設定的詳細說明請參閱[連線](connections.md#create)。

面板底部有**設定**、**支持本專案**、**說明**和**關於**的連結。欄位旁的 `?` 符號會顯示該欄位的簡短提示。在視窗任何位置按 `F1`，即可開啟目前畫面對應的指南章節。在 macOS 上，也可以從「輔助說明」選單開啟指南。

如果你用過 Kafka Explorer（本應用程式的舊名稱），Glafira 會在第一次啟動時複製它的連線、範本和傳送歷程記錄，密碼也會從系統金鑰儲存區一併複製。已經存在的項目不會被覆寫，舊資料也不會被刪除。macOS 可能會詢問一次是否允許存取舊的鑰匙圈項目。

<a id="local-cluster"></a>

## 用來練習的本機 Kafka

如果想在沒有共用叢集的情況下試用，可以用 Docker 在自己的電腦上啟動 Kafka。一個 Broker 就夠了：

```bash
docker run -d --name kafka -p 9092:9092 apache/kafka:3.9.0
```

接著建立連線：Bootstrap 伺服器為 `localhost:9092`，通訊協定為 `PLAINTEXT`，環境為 `local`。不再需要這個 Broker 時，用 `docker rm -f kafka` 將它移除。

空白的**連線**面板中的提示提到 `docker compose -f dev/docker-compose.yml up -d`。這個檔案屬於 Glafira 的原始碼。如果你有原始碼，這個指令會啟動一個有三個監聽器的 Broker：

- `localhost:9092`：`PLAINTEXT`。
- `localhost:9094`：`SASL_PLAINTEXT`，支援 `PLAIN`（使用者 `tester` / `tester-secret` 和 `admin` / `admin-secret`）和 SCRAM（`SCRAM-SHA-256` 與 `SCRAM-SHA-512` 使用 `scram-user` / `scram-secret`）。
- `localhost:9095`：使用用戶端憑證的 `SSL`（mTLS）。測試憑證位於 `dev/certs/out`。

這些使用者和憑證僅供本機測試使用。

<a id="language"></a>

## 介面語言

介面提供英文、俄文、德文、法文、西班牙文、義大利文、波蘭文、葡萄牙文（巴西）、中文（簡體和繁體）、日文和韓文版本。所有語言都免費。

變更語言：

1. 按一下左側面板底部的**設定**。
2. 在**介面語言**中選擇語言。

變更會立即生效，包括本指南和 macOS 選單。**跟隨系統**（預設）會使用作業系統的語言。如果清單中沒有該語言，介面會以英文顯示。

<a id="managed-settings"></a>

## 由組織設定的選項

在公司中，管理員可以透過原則為所有使用者設定某些選項，這些選項無法在應用程式中變更。原則可以：

- 設定介面語言：此時**設定**中的語言清單無法使用，並註明語言由你的組織設定；
- 關閉更新檢查，或將它指向內部鏡像站。

若要查看是否套用了原則，請開啟**關於**，查看**組織原則**。如果原則有錯誤，會顯示在那裡。有關原則的問題，請洽詢你的管理員。更新的說明請參閱[更新與診斷](updates.md)。
