# 常見問答

## 驗證問題

### 問：我嘗試 push/pull/clone 時遇到錯誤。我該怎麼辦？

請依照以下步驟診斷或解決問題：

1. 檢查您是否可以在網頁瀏覽器中存取遠端儲存庫。如果無法存取，這可能是權限問題，您應該聯繫儲存庫管理員以取得存取權。從終端機執行 `git remote -v` 以顯示遠端 URL。

1. 如果您在使用編輯器、IDE 或其他工具時遇到 Git 驗證問題，請嘗試從終端機執行相同的操作。這樣仍然會失敗嗎？如果操作在終端機中成功，請在任何問題報告中包含特定工具和版本的詳細資訊。

1. 設定環境變數 `GCM_TRACE` 並再次執行 Git 操作。請在 [環境文件][env-trace] 中尋找說明。

1. 如果以上方法都失敗了，請[在此][create-issue]建立一個問題，並務必附上追蹤日誌。

### 問：我收到錯誤訊息，指出不支援不安全的 HTTP

為確保您的資料安全，Git Credential Manager 不會透過未使用 TLS (HTTPS) 保護的 HTTP 連線傳送 Azure Repos、Azure DevOps Server (TFS)、GitHub 和 Bitbucket 的憑證。

請確保您的遠端 URL 使用「https://」而非「http://」。

### 問：我收到驗證錯誤，而且我位於網路代理伺服器後方

您可能需要設定 Git 和 GCM 以使用代理伺服器。詳細資訊請參閱[網路設定文件][netconfig-http-proxy]。

### 問：我在 Linux 上選擇憑證存放區時遇到錯誤

在 Linux 上，您必須[選擇並設定憑證存放區][credstores]，因為各種發行版和安裝方式的性質不同，我們無法保證有可用的合適儲存解決方案。

## 關於此專案

### 問：此專案與 [Git Credential Manager for Windows][gcm-windows] 和 [Git Credential Manager for Mac and Linux][gcm-linux] 有何關聯？

Git Credential Manager for Windows (GCM Windows) 是一個基於 .NET Framework 的 Git 憑證輔助程式，可在 Windows 上執行。同樣地，Git Credential Manager for Mac and Linux (Java GCM) 是一個基於 Java 的 Git 憑證輔助程式，僅能在 macOS 和 Linux 上執行。儘管這兩個專案都旨在解決相同的問題（提供與 Git 的無縫多重要素 HTTPS 驗證），但它們基於不同的程式碼庫和語言，這使得確保功能對等變得難以管理。

Git Credential Manager (GCM；此專案) 旨在用統一的程式碼庫取代 GCM Windows 和 Java GCM，以便未來更容易維護和增強。

### 問：這是否意味著 GCM for Windows (.NET Framework-based) 已被棄用？

是的。Git Credential Manager for Windows (GCM Windows) 已不再接收更新和修復。所有開發工作現已轉移至 GCM。GCM 在 Git for Windows 2.28 中作為憑證輔助程式選項提供，並將在 2.29 版中成為預設輔助程式。

### 問：這是否意味著基於 Java 的 GCM for Mac/Linux 已被棄用？

是的。應使用 GCM 或 SSH 金鑰取代 Git Credential Manager for Mac and Linux (Java GCM)。如果您希望在 macOS 或 Linux 上安裝 GCM，請遵循[下載與安裝說明][download-and-install]。

### 問：我想使用 SSH

GCM 僅適用於基於 HTTP(S) 的遠端儲存庫。Git 本身就支援 SSH，所以您不需要安裝任何其他東西。

若要使用 SSH，請點擊以下連結：

- [Azure DevOps][azure-ssh]
- [GitHub][github-ssh]
- [Bitbucket][bitbucket-ssh]

### 問：HTTP(S) 遠端儲存庫比 SSH 更受青睞嗎？

不，兩者皆非「首選」。SSH 不會消失，且 Git「原生」支援它。

### 問：您為何不直接將現有的 GCM Windows 程式碼庫從 .NET Framework 移植到 .NET Core？

GCM Windows 的設計並非跨平台架構。

### GCM 提供何種程度的支援？

我們將盡力提供支援。我們非常感謝您的回饋，以讓我們在所支援的每個平台上都能提供絕佳的體驗。

### 問：為什麼 GCM 不支援作業系統/發行版 'X' 或 Git 代管服務供應商 'Y'？

很可能的答案是我們還沒有時間處理！🙂

我們正在努力確保支援 Windows、macOS 和 Ubuntu 作業系統，以及下列 Git 代管服務供應商：Azure Repos、Azure DevOps Server (TFS)、GitHub 和 Bitbucket。

我們樂於接受提案和/或貢獻，讓 GCM 能在其他平台和 Git 主機供應商上執行。謝謝您！

## 技術

### 問：為什麼 `dev.azure.com` 需要 `credential.useHttpPath` 設定？

由於 Git 和像 GCM 這類憑證輔助程式的設計，我們需要此設定，讓 Git 在與 GCM 通訊時使用完整的遠端 URL（包括路徑部分）。新的 `dev.azure.com` 格式的 Azure DevOps URL 意味著帳戶名稱現在是路徑部分的一部分（例如：`https://dev.azure.com/contoso/...`）。需要 Azure DevOps 帳戶名稱才能解析正確的驗證授權單位（哪個 Azure AD 租用戶支援此帳戶，或者它是否由 Microsoft 個人帳戶支援）。

在舊版的 GCM for Windows 產品中，解決相同問題的方法是一種「取巧」的方式。GCM for Windows 會遍歷進程樹，尋找 `git-remote-https.exe` 進程，並嘗試讀取/解析進程環境區塊，以尋找命令列參數（其中包含完整的遠端 URL）。這種方法很脆弱，也不是一個跨平台的解決方案，因此 GCM 需要使用 `credential.useHttpPath` 設定。

### 問：為什麼 GCM 第一次啟動時需要這麼久？

GCM 會[自動偵測][autodetect]它正在與哪種 Git 主機通訊。GitHub、Bitbucket 和 Azure DevOps 各有其驗證形式，此外還有一個「通用」的使用者名稱和密碼選項。

對於這些服務的託管版本，GCM 可以從 URL 猜測要使用哪個服務。但對於具有唯一 URL 的內部部署版本，GCM 將透過網路呼叫進行探測。GCM 會快取探測結果，因此在第二次及之後的呼叫中應該會更快。

如果您知道您正在與哪個供應商通訊，並且想要避免探測，這是可能的。您可以明確地告訴 GCM 為 URL「example.com」使用哪個供應商，如下所示：

供應商|指令
-|-
GitHub|`git config --global credential.https://example.com.provider github`
Bitbucket|`git config --global credential.https://example.com.provider bitbucket`
Azure DevOps|`git config --global credential.https://example.com.provider azure-repos`
通用|`git config --global credential.https://example.com.provider generic`

### 問：在 Windows 7 上如何修復「無法建立 SSL/TLS 安全通道」的錯誤？

這可能表示您沒有可用的較新 TLS 版本。請[遵循 Microsoft 的指南][enable-windows-ssh]在您的電腦上啟用 TLS 1.1 和 1.2，特別是 **SChannel** 的說明。您的作業系統至少需要是 Windows 7 SP1，最後您應該有一個 `TLS 1.2` 金鑰，其 `DisabledByDefault` 設定為 `0`。您也可以閱讀[更多來自 Microsoft 的資訊][windows-server-tls]了解此變更。

### 問：如何將 GCM 與 Windows Subsystem for Linux (WSL) 搭配使用？

請仔細遵循[我們的 WSL 指南][wsl]中的說明。特別注意，如果您使用 Azure DevOps，需要在 WSL _內部_ 執行 `git config --global credential.https://dev.azure.com.useHttpPath true`。

### 問：GCM 是否支援多個使用者？如果是，如何運作？

這個問題回答起來相當複雜，但簡而言之，是的。詳情請參閱[我們關於多個使用者的文件][multiple-users]。

### 問：如何停用 GUI 對話方塊和提示？

有各種環境變數和設定選項可供自訂 GCM 如何提示您（或不提示）輸入。請參閱以下內容：

- [`GCM_INTERACTIVE`][env-interactive] / [`credential.interactive`][config-interactive]
- [`GCM_GUI_PROMPT`][env-gui-prompt] / [`credential.guiPrompt`][config-gui-prompt]
- [`GIT_TERMINAL_PROMPT`][git-term-prompt]（請注意，這是一個 _Git 設定_，會影響 Git 和 GCM）

### 問：如何擴充 GUI 提示/將提示與我的應用程式整合？

使用 Git 的應用程式開發人員——例如 Visual Studio、GitKraken 等——可能會想用看起來像他們應用程式風格的提示來取代 GCM 的預設 UI。這並不複雜（儘管需要一些工作）。

您可以透過使用 `credential.gitHubHelper`/`credential.bitbucketHelper` 設定或 `GCM_GITHUB_HELPER`/`GCM_BITBUCKET_HELPER` 環境變數，來專門取代 Bitbucket 和 GitHub 主機供應商的 GUI 提示。

將這些變數設定為會回應請求的外部輔助執行檔路徑，如同內建的 UI 輔助程式一樣。請參閱目前的 `--help` 文件以了解關於內建 UI 輔助程式 (`GitHub.UI`/`Atlassian.Bitbucket.UI`) 的更多資訊。

您也可以將這些變數設定為空字串 `""` 以強制使用終端機／文字介面的提示。

### 如何撤銷 GCM 對 GitHub.com 的授權？

在您的 GitHub 使用者設定中，前往 [整合 > 應用程式 > 已授權的 OAuth 應用程式 > Git Credential Manager][github-connected-apps] 並選擇「撤銷存取權」。

![撤銷 GCM OAuth 應用程式存取權][github-oauthapp-revoke]

撤銷存取權後，由 GCM 建立的任何權杖都將失效，且無法再用於存取您的儲存庫。下次 GCM 嘗試存取 GitHub.com 時，系統將會再次提示您授權。

### 我已使用從原始碼安裝的腳本在我的 Linux 發行版上安裝 GCM。現在要如何解除安裝 GCM 及其相依套件？

請參閱完整說明[此處][linux-uninstall-from-src]。

### 如何撤銷 GitLab OAuth 應用程式的存取權？

在某些情況下（例如，更新的範圍），您將需要手動撤銷並重新授權 GitLab OAuth 應用程式的存取權。您可以透過以下方式進行：

1. 前往[您 **使用者設定** 中的 **應用程式** 頁面][gitlab-apps]。
2. 捲動至 **已授權的應用程式**。
3. 點擊您想撤銷其存取權的應用程式名稱旁的 **撤銷** 按鈕（此處以 Git Credential Manager 為展示目的）。

   ![撤銷 GitLab OAuth 應用程式存取權的按鈕][gitlab-oauthapp-revoke]
4. 等待出現內容為 **已撤銷該應用程式的存取權** 的通知。

   ![成功撤銷的通知][gitlab-oauthapp-revoked]
5. 使用新的範圍重新授權應用程式（GCM 應該會自動在下次請求存取時為您啟動此流程）。

### 問：`configure` 和 `unconfigure` 指令有什麼作用？

#### `configure`

`configure` 指令會將 Git 設定為專用 GCM 作為憑證助手。`configure` 指令在 Windows 和 macOS 的安裝程式中會自動呼叫，但您也可以手動執行它。

它也會設定 Git 將完整的遠端 URL（包含路徑）提供給使用 `dev.azure.com` URL 格式的 Azure Repos 遠端儲存庫的憑證輔助程式。這是必要的，以便能夠正確識別該 Azure DevOps 組織的正確授權單位。

具體來說，`configure` 指令將會修改您的使用者 Git 設定檔以包含以下幾行：

```ini
[credential]
    helper =
    helper = <path-to-gcm>
[credential "https://dev.azure.com"]
    useHttpPath = true
```

..其中 `<path-to-gcm>` 是 GCM 執行檔的絕對路徑。

空的 `helper =` 行可確保任何可能已設定於系統 Git 設定中的現有認證輔助程式不會被使用。更多詳細資訊請參閱 [credential.helper][helper-config-docs]。

如果您傳遞 `--system` 選項，`configure` 指令將會改為修改系統 Git 設定。如果您想為所有電腦上的使用者設定 GCM，這將非常有用。

#### `unconfigure`

此指令基本上是復原 `configure` 指令所做的事情。它會檢查您的 Git 設定中由 `configure` 指令新增的行，並將其移除。`unconfigure` 指令由 Windows 的解除安裝程式以及 macOS 上的解除安裝腳本執行。

在 Windows 上，若以 `--system` 選項執行，`unconfigure` 指令將也會確保系統 Git 設定中的 `credential.helper` 設定不會被移除，並保留為 `manager`，此為 Git for Windows 設定的預設值。

[autodetect]: autodetect.md
[azure-ssh]: https://docs.microsoft.com/en-us/azure/devops/repos/git/use-ssh-keys-to-authenticate?view=azure-devops
[bitbucket-ssh]: https://confluence.atlassian.com/bitbucket/ssh-keys-935365775.html
[config-gui-prompt]: configuration.md#credentialguiprompt
[config-interactive]: configuration.md#credentialinteractive
[create-issue]: https://github.com/git-ecosystem/git-credential-manager/issues/create
[credstores]: credstores.md
[download-and-install]: ../README.md#download-and-install
[enable-windows-ssh]: https://support.microsoft.com/topic/update-to-enable-tls-1-1-and-tls-1-2-as-default-secure-protocols-in-winhttp-in-windows-c4bd73d2-31d7-761e-0178-11268bb10392
[env-gui-prompt]: environment.md#GCM_GUI_PROMPT
[env-interactive]: environment.md#GCM_INTERACTIVE
[env-trace]: environment.md#GCM_TRACE
[gcm-linux]: https://github.com/Microsoft/Git-Credential-Manager-for-Mac-and-Linux
[gcm-windows]: https://github.com/Microsoft/Git-Credential-Manager-for-Windows
[git-term-prompt]: https://git-scm.com/docs/git#Documentation/git.txt-codeGITTERMINALPROMPTcode
[github-connected-apps]: https://github.com/settings/connections/applications/0120e057bd645470c1ed
[github-oauthapp-revoke]: img/github-oauthapp-revoke.png
[github-ssh]: https://help.github.com/en/articles/connecting-to-github-with-ssh
[gitlab-apps]: https://gitlab.com/-/profile/applications
[gitlab-oauthapp-revoke]: ./img/gitlab-oauthapp-revoke.png
[gitlab-oauthapp-revoked]: ./img/gitlab-oauthapp-revoked.png
[helper-config-docs]: https://git-scm.com/docs/gitcredentials#Documentation/gitcredentials.txt-helper
[multiple-users]: multiple-users.md
[netconfig-http-proxy]: netconfig.md#http-proxy
[linux-uninstall-from-src]: ./linux-fromsrc-uninstall.md
[windows-server-tls]: https://docs.microsoft.com/previous-versions/windows/it-pro/windows-server-2012-R2-and-2012/dn786418(v=ws.11)#tls-12
[wsl]: wsl.md
