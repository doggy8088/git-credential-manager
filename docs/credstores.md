# 憑證儲存庫

GCM 支援多種儲存憑證的選項：

- Windows 憑證管理員
- DPAPI 保護的檔案
- macOS 鑰匙圈
- [freedesktop.org Secret Service API][freedesktop-secret-service]
- 與 GPG/[`pass`][passwordstore] 相容的檔案
- Git 的內建[憑證快取][credential-cache]
- 純文字檔案
- 傳遞/無操作 (無憑證儲存庫)

在 macOS 和 Windows 上的預設憑證儲存庫分別是 macOS 鑰匙圈以及 Windows 憑證管理員。

GCM 在 Linux 發行版上沒有預設儲存庫。

您可以透過設定 [`GCM_CREDENTIAL_STORE`][gcm-credential-store]
環境變數，或是 [`credential.credentialStore`][credential-store]
Git 組態設定。例如：

```shell
git config --global credential.credentialStore gpg
```

某些憑證儲存庫有限制，或需要進一步的設定這取決於您的特定設定。請參閱下方關於每個憑證儲存庫的詳細資訊。

## Windows 憑證管理員

**適用於：** _Windows_

**這是 Windows 上的預設儲存庫。**

**:warning: 無法透過網路/SSH 工作階段運作。**

```batch
SET GCM_CREDENTIAL_STORE="wincredman"
```

或

```shell
git config --global credential.credentialStore wincredman
```

此憑證儲存庫使用 Windows Credential API (`wincred.h`) 來儲存安全地將資料儲存在 Windows 憑證管理員中 (在舊版 Windows 中也稱為 Windows Credential Vault)。

您可以[在憑證管理員中存取與管理資料][access-windows-credential-manager]從控制台，或透過 [`cmdkey` 命令列工具][cmdkey]。

當透過網路工作階段 (例如 SSH) 連線至 Windows 機器時，GCM 無法將憑證永久儲存至 Windows 憑證管理員，這是由於 Windows 的限制。透過遠端桌面連線則沒有此限制。

## DPAPI 保護的檔案

**適用於：** _Windows_

```batch
SET GCM_CREDENTIAL_STORE="dpapi"
```

或

```shell
git config --global credential.credentialStore dpapi
```

此憑證儲存庫使用 Windows DPAPI 來加密憑證，這些憑證被儲存為您檔案系統中的檔案。其檔案結構與[純文字檔案憑證儲存庫][plaintext-files] 相同，但第一行 (即秘密值) 受 DPAPI 保護。

預設情況下，檔案儲存在 `%USERPROFILE%\.gcm\dpapi_store`。此路徑可透過 `GCM_DPAPI_STORE_PATH` 環境變數來設定。

如果目錄不存在，將會建立。

## macOS 鑰匙圈

**適用於：** _macOS_

**這是 macOS 上的預設儲存庫。**

```shell
export GCM_CREDENTIAL_STORE=keychain
# or
git config --global credential.credentialStore keychain
```

此憑證儲存庫使用預設的 macOS 鑰匙圈，通常是
`login` 鑰匙圈。

您可以使用「鑰匙圈存取」應用程式[管理儲存在鑰匙圈中的資料][mac-keychain-management]
。

## [freedesktop.org Secret Service API][freedesktop-secret-service]

**可用於：** _Linux_

**:warning: 需要圖形化使用者介面會話。**

```shell
export GCM_CREDENTIAL_STORE=secretservice
# or
git config --global credential.credentialStore secretservice
```

此憑證儲存庫使用 `libsecret` 函式庫來與 Secret Service 互動。它會將憑證安全地儲存於「集合」中，這些集合可透過諸如 `secret-tool` 和 `seahorse` 等工具來檢視。

需要圖形化使用者介面才能顯示安全提示以請求解鎖一個祕密集合。

## 與 GPG/[`pass`][passwordstore] 相容的檔案

**可用於：** _macOS, Linux_

**:warning: 需要 `gpg`、`pass` 以及一個 GPG 金鑰對。**

```shell
export GCM_CREDENTIAL_STORE=gpg
# or
git config --global credential.credentialStore gpg
```

此憑證儲存庫使用 GPG 來加密包含憑證的檔案，這些檔案儲存在您的檔案系統中。檔案結構與廣受歡迎的 [`pass`][passwordstore] 工具相容。預設情況下，檔案儲存在 `~/.password-store` 中，但此路徑可透過 `pass` 環境變數 `PASSWORD_STORE_DIR` 進行設定。

在您可以使用此憑證儲存庫之前，必須先透過 `pass` 工具進行初始化，而該工具又需要一個有效的 GPG 金鑰對。若要初始化儲存庫，請執行：

```shell
pass init <gpg-id>
```

..其中 `<gpg-id>` 是您系統上 GPG 金鑰對的使用者 ID。若要建立一個新的 GPG 金鑰對，請執行：

```shell
gpg --gen-key
```

..並依照提示操作。

### 無周邊/僅 TTY 的會話

如果您在無周邊/僅 TTY 的環境中使用 `gpg` 憑證儲存庫，您必須確保已為 GPG 代理程式 (`gpg-agent`) 設定一個合適的終端機密碼輸入程式，例如 `pinentry-tty` 或 `pinentry-curses`。

如果您是透過 SSH 連線到您的系統，那麼 `SSH_TTY` 變數應該會自動設定。GCM 會將 `SSH_TTY` 的值傳遞給 GPG/GPG 代理程式作為提示輸入密碼時要使用的 TTY 裝置。

如果您不是透過 SSH 連線，或因其他原因而未設定 `SSH_TTY` 環境變數，您就必須在執行 GCM 之前設定 `GPG_TTY` 環境變數。最簡單的方式是將以下內容新增至您的設定檔 (`~/.bashrc`、`~/.profile` 等) 中：

```shell
export GPG_TTY=$(tty)
```

**注意：** 在此處使用 `/dev/tty` 似乎無效 - 您必須使用真實的 TTY 裝置路徑，即由 `tty` 工具所回傳的路徑。

## Git 的內建[憑證快取][credential-cache]

**可用於：** _macOS, Linux_

```shell
export GCM_CREDENTIAL_STORE=cache
# or
git config --global credential.credentialStore cache
```

此憑證儲存庫使用 Git 的內建短暫記憶體內[憑證快取][credential-cache]。這有助於您減少需要驗證的次數，但是不需要將憑證儲存在永久性儲存空間中。這對於像是 [Azure Cloud Shell][azure-cloudshell] 的情境或 [AWS CloudShell][aws-cloudshell] 很有幫助，在這些情境下您不希望將憑證留在磁碟上，但也不希望在每次 Git 操作時都重新驗證。

預設情況下，`git credential-cache` 會將您的憑證儲存 900 秒。這個時間以及它所接受的任何其他[選項][git-credential-cache-options]，可以透過在環境變數中設定來更改 `GCM_CREDENTIAL_CACHE_OPTIONS` 或 Git 設定值`credential.cacheOptions`。 (使用 `--socket` 選項是未經測試的且不支援的，但沒有理由它不能運作。)

```shell
export GCM_CREDENTIAL_CACHE_OPTIONS="--timeout 300"
# or
git config --global credential.cacheOptions "--timeout 300"
```

## 純文字檔案

**適用於：** _Windows, macOS, Linux_

**:warning: 這不是一種安全的憑證儲存方法！**

```shell
export GCM_CREDENTIAL_STORE=plaintext
# or
git config --global credential.credentialStore plaintext
```

此憑證儲存庫會將憑證以純文字檔案形式儲存在您的檔案系統中。預設情況下，檔案儲存在 `~/.gcm/store` 或 `%USERPROFILE%\\.gcm\\store`。可以使用環境變數 `GCM_PLAINTEXT_STORE_PATH` 進行設定環境變數。

如果目錄不存在，將會自動建立。

在 POSIX 平台上，新建立的儲存庫目錄將會設定權限使得只有擁有者可以`r`ead/`w`rite/e`x`ecute (`700` 或 `drwx---`)。現有目錄的權限將不會被修改。

注意：GCM 的純文字儲存庫與 [git-credential-store][git-credential-store] 不同，雖然格式相似，但預設路徑不同。

---

:warning: **警告** :warning:

**此儲存機制並不安全！**

**密鑰和憑證會以純文字檔案形式儲存，_沒有任何安全性_！**

**強烈建議**一律使用其他憑證儲存庫選項。此選項僅為相容性考量及在沒有其他安全選項可用的環境中使用而提供。

如果您選擇使用此憑證儲存庫，建議您設定此目錄的權限，讓其他使用者或應用程式無法存取其中的檔案。如果可能，請使用存在於外部磁碟區上的路徑以便隨身攜帶，並使用全磁碟加密。

## 通透/無操作 (無憑證儲存庫)

**適用於：** _Windows, macOS, Linux_

**:warning: 。**

```batch
SET GCM_CREDENTIAL_STORE="none"
```

或

```shell
git config --global credential.credentialStore none
```

此選項會停用內部憑證儲存庫。所有儲存或擷取憑證的操作將不會執行任何動作，且會回傳成功。如果您想透過 Git 循序串接使用不同的憑證儲存庫，並設定不讓 GCM 儲存憑證，此選項便很有用。

請注意，您需要確保將另一個憑證輔助程式放置在 GCM 之前，在 `credential.helper` Git 設定中，否則系統會提示您在每次與遠端儲存庫互動時輸入您的憑證。

[access-windows-credential-manager]: https://support.microsoft.com/en-us/windows/accessing-credential-manager-1b5c916a-6a16-889f-8581-fc16e8165ac0
[aws-cloudshell]: https://aws.amazon.com/cloudshell/
[azure-cloudshell]: https://docs.microsoft.com/azure/cloud-shell/overview
[cmdkey]: https://docs.microsoft.com/en-us/windows-server/administration/windows-commands/cmdkey
[credential-store]: configuration.md#credentialcredentialstore
[credential-cache]: https://git-scm.com/docs/git-credential-cache
[freedesktop-secret-service]: https://specifications.freedesktop.org/secret-service-spec/
[gcm-credential-store]: environment.md#GCM_CREDENTIAL_STORE
[git-credential-store]: https://git-scm.com/docs/git-credential-store
[mac-keychain-management]: https://support.apple.com/en-gb/guide/mac-help/mchlf375f392/mac
[git-credential-cache-options]: https://git-scm.com/docs/git-credential-cache#_options
[passwordstore]: https://www.passwordstore.org/
[plaintext-files]: #plaintext-files
