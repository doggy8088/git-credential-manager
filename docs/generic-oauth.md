# 通用主機供應商 OAuth

許多 Git 託管服務使用流行的標準 OAuth2 或 OpenID Connect (OIDC)驗證機制，以保護其託管的儲存庫。Git Credential Manager 僅需透過簡單的設定，即可支援任何基於 OAuth2 的通用 Git 主機設定一些組態。

## 註冊 OAuth 應用程式

為了在支援 OAuth 的 Git 主機上使用 GCM，您必須先向您的主機註冊一個 OAuth 應用程式。如何執行的說明可以在您的 Git 主機供應商的文件中找到。

註冊新應用程式時，您應確保設定一個基於 HTTP 的指向 `localhost` 的重新導向 URL；例如：

```text
http://localhost
http://localhost:<port>
http://127.0.0.1
http://127.0.0.1:<port>
```

請注意，您不能使用 HTTPS 重新導向 URL。GCM 不需要使用特定的連接埠號碼；如果您的 Git 主機要求您在重新導向 URL 中指定連接埠號碼，那麼 GCM 就會使用該號碼。否則，將會在驗證開始時選取一個可用的連接埠。

您必須確保讀取和寫入 Git 儲存庫所需的所有範圍都已授予該應用程式，否則產生的憑證在使用 Git 推送或擷取時將會導致錯誤。

在註冊過程中，您應該也會獲得一個用戶端 ID 以及，（選擇性）一個用戶端密鑰。您將需要這兩者來設定 GCM。

## 設定 GCM

為了設定 GCM 以便在您的 Git 主機上使用 OAuth，您需要設定在您的 Git 組態中設定以下值：

- 用戶端 ID
- 用戶端密鑰（選擇性）
- 重新導向 URL（選擇性，預設為 `http://127.0.0.1`）
- 範圍（選擇性）
- OAuth 端點
  - 授權端點
  - 權杖端點
  - 裝置碼授權端點（選擇性）

OAuth 端點可透過查閱您 Git 主機的 OAuth 應用程式開發文件找到。URL 可以是絕對路徑，也可以是相對於主機名稱的路徑；例如：`https://example.com/oauth/authorize` 或 `/oauth/authorize`。

為了設定這些值，您可以執行以下指令，其中 `<HOST>` 是您的 Git 主機的主機名稱：

```shell
git config --global credential.<HOST>.oauthClientId <ClientID>
git config --global credential.<HOST>.oauthClientSecret <ClientSecret>
git config --global credential.<HOST>.oauthRedirectUri <RedirectURL>
git config --global credential.<HOST>.oauthAuthorizeEndpoint <AuthEndpoint>
git config --global credential.<HOST>.oauthTokenEndpoint <TokenEndpoint>
git config --global credential.<HOST>.oauthScopes <Scopes>
git config --global credential.<HOST>.oauthDeviceEndpoint <DeviceEndpoint>
```

**指令範例：**

- `git config --global credential.https://example.com.oauthClientId C33F2751FB76`

- `git config --global credential.https://example.com.oauthScopes "code:write profile:read"`

**Git 組態範例**

```ini
[credential "https://example.com"]
    oauthClientId = 9d886e36-5771-4f2b-8c8b-420c68ad5baa
    oauthClientSecret = 4BC5BD4704EAE28FD832
    oauthRedirectUri = "http://127.0.0.1"
    oauthAuthorizeEndpoint = "/login/oauth/authorize"
    oauthTokenEndpoint = "/login/oauth/token"
    oauthDeviceEndpoint = "/login/oauth/device"
    oauthScopes = "code:write profile:read"
    oauthDefaultUserName = "OAUTH"
    oauthUseClientAuthHeader = false
```

### 額外設定

根據您的 Git 託管服務對 OAuth 的具體實作，您可能也需要指定額外的行為。

#### 權杖使用者名稱

如果您的 Git 託管服務要求您指定與 OAuth 權杖一起使用的使用者名稱您可以將使用者名稱包含在 Git 遠端 URL 中，或指定一個預設選項透過 Git 設定。

帶有使用者名稱的 Git 遠端範例：`https://username@example.com/repo.git`。為了使用特殊字元，您需要對值進行 URL 編碼；例如範例 `@` 變成 `%40`。

預設情況下，GCM 會使用 `OAUTH-USER` 這個值，除非在遠端 URL 中有指定，或使用 `credential.<HOST>.oauthDefaultUserName` 設定來覆寫。

#### 在標頭中包含客戶端驗證

如果您的 Git 託管服務的 OAuth 實作對於是否應將客戶端 ID 與 secret 包含在 `Authorization` 標頭中，您可以使用以下設定來控制：

```shell
git config --global credential.<HOST>.oauthUseClientAuthHeader <true|false>
```

預設行為是包含這些值；即 `true`。
