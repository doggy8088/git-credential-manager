# Bitbucket 驗證、2FA 與 OAuth

預設情況下，為了驗證私有的 Git 儲存庫，Bitbucket 支援透過 HTTPS 進行 SSH 和使用者名稱/密碼的基本驗證 (Basic Auth)。透過 HTTPS 的使用者名稱/密碼基本驗證也適用於 REST API 存取。此外，Bitbucket 還支援應用程式密碼，可透過基本驗證以「使用者名稱/應用程式密碼」的形式使用。

為增強安全性，Bitbucket 提供選用的雙重驗證 (2FA)。啟用 2FA 後，透過使用者名稱/密碼基本驗證存取 REST API 和 Git 儲存庫的功能將被暫停。屆時，使用者可選擇使用「使用者名稱/應用程式密碼」的基本驗證來進行 REST API 和 Git 互動、使用 OAuth 進行 REST API 和 Git/Hg 互動，或使用 SSH 進行 Git/HG 互動，並為 REST API 選擇前述其中一種方式。SSH 和 REST API 的存取方式超出了本文件的範圍。請閱讀關於 [Bitbucket 的 2FA 實作][2fa-impl]。

應用程式密碼不是特別方便使用者，因為一旦建立，Bitbucket 就會隱藏其值，即使是擁有者也無法看見。它們旨在用於與 Bitbucket 通訊的應用程式中，應用程式可以記住並使用該應用程式密碼。[其他資訊][additional-info]。

當 2FA 啟用時，OAuth 是使用者透過 HTTPS 遠端 URL 與 Git 儲存庫互動時所預設的驗證方法。基本上，一旦用戶端應用程式取得了 OAuth Access Token，它就可以用來取代使用者的密碼。閱讀更多關於 [Bitbucket 的 OAuth 實作][oauth-impl] 的資訊。

Bitbucket 的 OAuth 實作遵循 OAuth 2.0 的標準規範，這部分超出了本文件的範圍。然而，它實作了 OAuth 2.0 中一個相對罕見的部分：Refresh Token。Bitbucket 的 Access Token 若未被撤銷，將在一小時後過期，這與 GitHub 的一年過期不同。當 GitHub 的 Access Token 過期時，使用者必須透過標準的 OAuth 驗證流程來取得新的 Access Token。理論上，這種情況一年只發生一次，所以不會太麻煩。由於 Bitbucket 的 Access Token 每小時都會過期，期望使用者每小時都進行一次 OAuth 驗證流程是不切實際的。因此，Bitbucket 實作了 Refresh Token。Refresh Token 會與 Access Token 同時發給用戶端應用程式。它們只能用於請求新的 Access Token，且前提是它們尚未被撤銷。因此，在 Git 憑證管理器 (Git Credentials Manager) 中對 Bitbucket 及其 OAuth 的支援，與 VSTS 和 GitHub 的實作方式有顯著不同。下文將會更詳細地解釋。

## 多個使用者帳號

與 Git 憑證管理器 (GCM) 中的 GitHub 實作不同，Bitbucket 的實作是將「秘密」(密碼、應用程式密碼或 OAuth 權杖) 連同使用者名稱一起儲存在 [Windows 憑證管理器][wincred-manager] 的保存庫中。

視情況而定，這意味著要麼將明確的使用者名稱儲存到 Windows 憑證管理器/保存庫中，要麼將使用者名稱包含在用作 Windows 憑證管理器保存庫中項目識別金鑰的 URL 中，例如，使用像 `git:https://mminns@bitbucket.org/` 這樣的金鑰，而不是 `git:https://bitbucket.org`。這表示 GCM 中的 Bitbucket 實作可以為單一使用者支援多個 Bitbucket 帳號和使用者名稱，例如個人帳號和工作帳號。

## 地端部署的 Bitbucket

地端部署的 Bitbucket，更準確地說應是 Bitbucket Server 或 Bitbucket DC，與 Bitbucket 的雲端實例 [bitbucket.org][bitbucket] 相比，有許多不同之處。

可以透過 Atlassian SDK 執行以下指令，在本機執行 Bitbucket Server 進行測試：

    ❯ atlas-run-standalone --product bitbucket

請參閱 [atlas-run-standalone][atlas-run-standalone] 的開發者文件。

這將會下載並執行一個獨立的 Bitbucket Server 實例，可使用 `admin`/`admin` 憑證在以下網址存取：

    https://localhost:7990/bitbucket

Atlassian 提供關於如何下載和安裝其 SDK 的[文件][atlassian-sdk] 。

## OAuth2 設定

Bitbucket DC [7.20](https://confluence.atlassian.com/bitbucketserver/bitbucket-data-center-and-server-7-20-release-notes-1101934428.html) 新增了對 OAuth2 傳入應用程式連結的支援，可用於支援 Git 的 OAuth2 驗證。這在 Bitbucket 使用 SSO 的環境中特別有用，因為它移除了使用者管理 [SSH 金鑰](https://confluence.atlassian.com/display/BITBUCKETSERVER0717/Using+SSH+keys+to+secure+Git+operations) 或手動管理 [HTTP Access Token](https://confluence.atlassian.com/display/BITBUCKETSERVER0717/Personal+access+tokens) 的需求。

### 主機設定

更多詳細資訊，請參閱 [Bitbucket 關於 Data Center 和 Server 應用程式連結至其他應用程式的文件](https://confluence.atlassian.com/bitbucketserver/link-to-other-applications-1018764620.html)

建立傳入的 OAuth 2 應用程式連結：
<!-- markdownlint-disable MD034 -->
1. 前往 Administration/Application Links
1. 建立連結
   1. 畫面 1
      - 外部應用程式 [勾選]
      - 傳入應用程式 [勾選]
   1. 畫面 2
      - 名稱：GCM
      - 重新導向 URL：`http://localhost:34106/`
      - 應用程式權限：Repositories.Read [勾選], Repositories.Write [勾選]
   1. 儲存
   <!-- markdownlint-enable MD034 -->
   1. 複製 `ClientId` 和 `ClientSecret` 以設定 GCM

### 用戶端設定

使用上方複製的 `ClientId` 和 `ClientSecret` 來設定 OAuth2，(詳細資訊請參閱 [credential.bitbucketDataCenterOAuthClientId](configuration.md#credential.bitbucketDataCenterOAuthClientId) 和 [credential.bitbucketDataCenterOAuthClientSecret](configuration.md#credential.bitbucketDataCenterOAuthClientSecret))

    ❯ git config --global credential.bitbucketDataCenterOAuthClientId {`已複製的 ClientId`}

    ❯ git config --global credential.bitbucketDataCenterOAuthClientSecret {`已複製的 ClientSecret`}
<!-- markdownlint-disable MD034 -->
如 [設定選項](configuration.md#Configuration%20options) 中所述可以讓設定更加具體，使其僅適用於特定的 Bitbucket DC 主機，方法是指定主機 URL，例如 https://bitbucket.example.com/
<!-- markdownlint-enable MD034 -->

    ❯ git config --global credential.https://bitbucket.example.com.bitbucketDataCenterOAuthClientId {`已複製的 ClientId`}

    ❯ git config --global credential.https://bitbucket.example.com.bitbucketDataCenterOAuthClientSecret {`已複製的 ClientSecret`}
<!-- markdownlint-disable MD034 -->
由於 GCM 解析主機和決定 REST API URL 的方式，如果 Bitbucket DC 執行個體託管於相對 URL 下 (例如 https://example.com/bitbucket) 則必須設定 Git 將完整路徑傳送給 GCM。這可以透過使用 [credential.useHttpPath](configuration.md#credential.useHttpPath) 設定來完成。
    ❯ git config --global credential.https://example.com/bitbucket.usehttppath true
<!-- markdownlint-enable MD034 -->

如果 Bitbucket DC 執行個體的 URL 中使用了連接埠號碼，那麼 Git 設定需要反映這一點。然而，由於 [問題 608](https://github.com/git-ecosystem/git-credential-manager/issues/608) 的緣故在解析 [credential.bitbucketDataCenterOAuthClientId](configuration.md#credential.bitbucketDataCenterOAuthClientId) 時會忽略連接埠
和 [credential.bitbucketDataCenterOAuthClientSecret](configuration.md#credential.bitbucketDataCenterOAuthClientSecret)。
<!-- markdownlint-disable MD034 -->
例如，一個位於 https://example.com:7990/bitbucket 的 Bitbucket DC 主機將會需要以下形式的設定：
<!-- markdownlint-enable MD034 -->
    ❯ git config --global credential.https://example.com/bitbucket.bitbucketDataCenterOAuthClientId {`已複製的 ClientId`}

    ❯ git config --global credential.https://example.com/bitbucket.bitbucketDataCenterOAuthClientSecret {`已複製的 ClientSecret`}

    ❯ git config --global credential.https://example.com:7990/bitbucket.usehttppath true

[additional-info]:https://confluence.atlassian.com/display/BITBUCKET/App+passwords
[atlas-run-standalone]: https://developer.atlassian.com/server/framework/atlassian-sdk/atlas-run-standalone/
[bitbucket]: https://bitbucket.org
[2fa-impl]: https://confluence.atlassian.com/bitbucket/two-step-verification-777023203.html
[oauth-impl]: https://confluence.atlassian.com/bitbucket/oauth-on-bitbucket-cloud-238027431.html
[atlassian-sdk]: https://developer.atlassian.com/server/framework/atlassian-sdk/
[wincred-manager]: https://msdn.microsoft.com/en-us/library/windows/desktop/aa374792(v=vs.85).aspx
