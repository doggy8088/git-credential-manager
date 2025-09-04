# 設定選項

[Git Credential Manager][usage] 對大多數使用者而言，無需額外設定即可運作。

Git Credential Manager (GCM) 可透過 Git 的設定檔進行設定，並遵循 Git 取用這些檔案時的所有相同規則。

全域設定會覆寫系統設定，本地設定則會覆寫全域設定；由於設定詳細資訊存在於 Git 的設定檔中，您可以使用 Git 的 `git config` 工具來設定、取消設定及變更設定值。所有 GCM 的設定皆以 `credential` 一詞開頭。

GCM 除了 Git 使用的標準本地 \> 全域 \> 系統分層外，還支援多個層級的設定。特定於 URL 的設定或覆寫可使用以下語法，套用至 `credential` 命名空間中的任何值。

此外，GCM 也會遵循數個 GCM 特有的[環境變數][envars]，**其優先權高於設定選項**。系統管理員也可以為 GCM 使用的許多設定配置[預設值][enterprise-config]。

GCM 只有在安裝和設定後才能被 Git 使用。請使用 `git config --global credential.helper manager` 將 GCM 指定為您的憑證輔助程式。使用 `git config credential.helper` 來查看目前的設定。

**範例：**

> `credential.microsoft.visualstudio.com.namespace` 比 `credential.visualstudio.com.namespace` 更具體，而後者又比 `credential.namespace` 更具體。

在上述範例中，`credential.namespace` 設定會影響任何遠端儲存庫；`credential.visualstudio.com.namespace` 會影響 'visualstudio.com' 網域及/或其任何子網域（包含 `www.`）中的任何遠端儲存庫；而 `credential.microsoft.visualstudio.com.namespace` 設定則只會套用至託管在 'microsoft.visualstudio.com' 的遠端儲存庫。

關於 GCM 可理解的完整設定清單，請參閱下方列表。

## 可用設定

### credential.interactive

允許或停用 GCM 與使用者互動（顯示 GUI 或 TTY 提示）。若需要互動但此功能已被停用，則會傳回錯誤。

這在無周邊（headless）和無人看管的環境（例如建構伺服器）中使用 GCM 時相當有幫助，在這些環境中，讓程式失敗會比無限期地等待一個不存在的使用者來得好。

若要停用互動功能，請將此值設為 `false` 或 `0`。

#### 相容性

在 GCM 的舊版本中，此設定有不同的行為並接受其他值。下表總結了行為的變化以及舊值（如 `never`）的對應關係：

值|舊有意義|新意義
-|-|-
`auto`|若需要則提示 – 可能的話使用快取憑證|_(不變)_
`never`, `false`| 永不提示 – 若需要互動則失敗|_(不變)_
`always`, `force`, `true`|一律提示 – 不使用快取的憑證|若需要則提示（與舊的 `auto` 值相同）

#### 範例

```shell
git config --global credential.interactive false
```

預設為啟用。

**另請參閱：[GCM_INTERACTIVE][gcm-interactive]**

---

### credential.trace

啟用所有活動的追蹤記錄。將 Git 與 GCM 設定為追蹤至相同位置通常是理想的做法，且 GCM 與 `GIT_TRACE` 相容並協同運作。

#### 範例

```shell
git config --global credential.trace /tmp/git.log
```

如果 `credential.trace` 的值是現有目錄中檔案的完整路徑，記錄將會附加到該檔案。

如果 `credential.trace` 的值為 `true` 或 `1`，記錄會寫入標準錯誤。

預設為停用。

**另請參閱：[GCM_TRACE][gcm-trace]**

---

### credential.traceSecrets

啟用追蹤機密與敏感資訊，這些資訊預設會在追蹤輸出中被遮罩。需要同時啟用 `credential.trace`。

#### 範例

```shell
git config --global credential.traceSecrets true
```

如果 `credential.traceSecrets` 的值為 `true` 或 `1`，追蹤記錄將會包含機密資訊。

預設為停用。

**另請參閱：[GCM_TRACE_SECRETS][gcm-trace-secrets]**

---

### credential.traceMsAuth

啟用將 Microsoft 驗證函式庫 (MSAL) 記錄包含於 GCM 追蹤輸出中。需要同時啟用 `credential.trace`。

#### 範例

```shell
git config --global credential.traceMsAuth true
```

如果 `credential.traceMsAuth` 的值為 `true` 或 `1`，追蹤記錄將會包含詳細的 MSAL 記錄。

預設為停用。

**另請參閱：[GCM_TRACE_MSAUTH][gcm-trace-msauth]**

---

### credential.debug

在 GCM 啟動時暫停執行，以等待偵錯器附加。

#### 範例

```shell
git config --global credential.debug true
```

預設為停用。

**另請參閱：[GCM_DEBUG][gcm-debug]**

---

### credential.provider

定義驗證時要使用的主機供應商。

ID|供應商
-|-
`auto` _(預設)_|_\[自動\]_ ([了解更多][autodetect])
`azure-repos`|Azure Repos
`github`|GitHub
`bitbucket`|Bitbucket
`gitlab`|GitLab _(支援瀏覽器中的 OAuth、個人存取權杖與基本驗證)_
`generic`|通用 (任何其他未列於上方的供應商)

自動供應商選擇是根據遠端 URL。

此設定通常與限定範圍的 URL 一起使用，以對應一組特定的遠端 URL 至供應商，例如將主機標記為 GitHub Enterprise 執行個體。

#### 範例

```shell
git config --global credential.ghe.contoso.com.provider github
```

**另請參閱：[GCM_PROVIDER][gcm-provider]**

---

### credential.authority _(已棄用)_

> 此設定已棄用，應由 `credential.provider` 取代並使用對應的供應商 ID 值。
>
> 如需更多資訊，請參閱[遷移指南][provider-migrate]。

選擇驗證時要使用的主機供應商，此選擇是根據供應商支援的授權單位。

授權機構|供應商
-|-
`auto` _(預設)_|_\[自動\]_
`msa`, `microsoft`, `microsoftaccount`, `aad`, `azure`, `azuredirectory`, `live`, `liveconnect`, `liveid`|Azure Repos _(支援 Microsoft 驗證)_
`github`|GitHub _(支援 GitHub 驗證)_
`bitbucket`|Bitbucket.org _(支援基本驗證與 OAuth)_，Bitbucket Server _(支援基本驗證)_
`gitlab`|GitLab _(支援瀏覽器中的 OAuth、個人存取權杖與基本驗證)_
`basic`, `integrated`, `windows`, `kerberos`, `ntlm`, `tfs`, `sso`|通用 _(支援基本與 Windows 整合式驗證)_

#### 範例

```shell
git config --global credential.ghe.contoso.com.authority github
```

**另請參閱：[GCM_AUTHORITY][gcm-authority]**

---

### credential.guiPrompt

允許或停用 GCM 顯示 GUI 提示。如果存在等效的終端機/文字型提示，則會改為顯示該提示。

若要停用所有互動功能，請參閱 [credential.interactive][credential-interactive]。

#### 範例

```shell
git config --global credential.guiPrompt false
```

預設為啟用。

**另請參閱：[GCM_GUI_PROMPT][gcm-gui-prompt]**

---

### credential.guiSoftwareRendering

強制 GUI 提示使用軟體渲染。

此設定目前僅適用於 Windows。

#### 範例

```shell
git config --global credential.guiSoftwareRendering true
```

預設為 false (在可用時使用硬體加速)。

> [!NOTE]
> ARM 裝置上的 Windows 預設使用軟體渲染以解決已知的 Avalonia 問題：<https://github.com/AvaloniaUI/Avalonia/issues/10405>

**另請參閱：[GCM_GUI_SOFTWARE_RENDERING][gcm-gui-software-rendering]**

---

### credential.allowUnsafeRemotes

允許將憑證傳輸到不安全的遠端 URL，例如未加密的 HTTP URL。不建議在一般情況下使用此設定，僅應在必要時使用。

預設為 false (不允許不安全的遠端 URL)。

#### 範例

```shell
git config --global credential.allowUnsafeRemotes true
```

**另請參閱：[GCM_ALLOW_UNSAFE_REMOTES][gcm-allow-unsafe-remotes]**

---

### credential.autoDetectTimeout

設定 GCM 在主機供應商自動偵測探測期間應等待網路回應的最長時間 (以毫秒為單位)。

有關更多資訊，請參閱 [auto-detection][auto-detection]。

**注意：** 使用負值或零可完全停用探測。

預設為 2000 毫秒 (2 秒)。

#### 範例

```shell
git config --global credential.autoDetectTimeout -1
```

**另請參閱：[GCM_AUTODETECT_TIMEOUT][gcm-autodetect-timeout]**

---

### credential.allowWindowsAuth

允許偵測通用主機供應商是否支援 Windows 整合式驗證 (WIA)。將此值設定為 `false` 將防止使用 WIA，並在使用通用主機供應商時強制顯示基本驗證提示。

**注意：** WIA 僅在 Windows 上受支援。

**注意：** WIA 是 NTLM 和 Kerberos (以及 Negotiate) 的總稱。

值|WIA 偵測
-|-
`true` _(預設)_|允許
`false`|不允許

#### 範例

```shell
git config --global credential.tfsonprem123.allowWindowsAuth false
```

**另請參閱：[GCM_ALLOW_WINDOWSAUTH][gcm-allow-windowsauth]**

---

### credential.httpProxy _(已棄用)_

> 此設定已棄用，應改用[標準的 `http.proxy` Git 設定選項][git-config-http-proxy]。
>
> 請參閱 [HTTP Proxy][http-proxy] 以取得更多資訊。

設定 GCM 使用代理伺服器以進行網路操作。

**注意：**Git 本身並 _不_ 遵守此設定；此設定 _僅_ 影響 GCM。

#### 範例

```shell
git config --global credential.httpsProxy http://john.doe:password@proxy.contoso.com
```

**另請參閱：[GCM_HTTP_PROXY][gcm-http-proxy]**

---

### credential.bitbucketAuthModes

覆寫在 Bitbucket 驗證期間所呈現的可用驗證模式。如果未設定此選項，則可用的驗證模式將會被自動偵測。

**注意：**此設定僅適用於 Bitbucket.org，不適用於 Server 或 DC 實例。

**注意：**此設定支援多個值，以逗號分隔。

值|驗證模式
-|-
_(未設定)_|自動偵測模式
`oauth`|基於 OAuth 的驗證
`basic`|基於基本/PAT 的驗證

#### 範例

```shell
git config --global credential.bitbucketAuthModes "oauth,basic"
```

**另請參閱：[GCM_BITBUCKET_AUTHMODES][gcm-bitbucket-authmodes]**

---

### credential.bitbucketAlwaysRefreshCredentials

強制 GCM 忽略任何現有已儲存的基本驗證或 OAuth 存取權杖，並在將憑證回傳給 Git 之前一律執行重新整理程序。

這與 OAuth 憑證尤其相關。Bitbucket.org 的存取權杖會在 2 小時後過期，之後必須使用重新整理權杖來取得新的存取權杖。

啟用此選項將能提升效能，當使用 Oauth2 並與 Bitbucket.org 互動，且平均提交頻率低於每 2 小時一次時。

當使用基本驗證時，啟用此選項將會降低效能，因為它會要求使用者每次都重新輸入憑證。

值|回傳前重新整理憑證
-|-
`true`、`1`、`yes`、`on` |一律
`false`、`0`、`no`、`off`_(預設)_|僅當憑證被發現無效時

#### 範例

```shell
git config --global credential.bitbucketAlwaysRefreshCredentials true
```

預設為 false/停用。

**另請參閱：[GCM_BITBUCKET_ALWAYS_REFRESH_CREDENTIALS][gcm-bitbucket-always-refresh-credentials]**

---

### credential.bitbucketValidateStoredCredentials

強制 GCM 在將任何已儲存的憑證回傳給 Git 之前驗證它們。它是透過呼叫一個需要驗證的 REST API 資源來完成此操作。

停用此選項可減少 GCM 在擷取時的 HTTP 流量憑證。這可能會改善使用者效能，但會增加 Git 遠端呼叫無法對主機進行身份驗證的次數，因此需要使用者重試 Git 遠端呼叫。

啟用此選項有助於確保 Git 始終獲得有效的憑證。

值|驗證憑證
-|-
`true`, `1`, `yes`, `on`_(預設)_|總是
`false`, `0`, `no`, `off`|從不

#### 範例

```shell
git config --global credential.bitbucketValidateStoredCredentials true
```

預設為 true/啟用。

**另請參閱：[GCM_BITBUCKET_VALIDATE_STORED_CREDENTIALS](environment.md#GCM_BITBUCKET_VALIDATE_STORED_CREDENTIALS)**

---

### credential.bitbucketDataCenterOAuthClientId

若要搭配 Bitbucket DC 使用 OAuth，必須建立一個外部的、傳入的
[AppLink](https://confluence.atlassian.com/bitbucketserver/configure-an-incoming-link-1108483657.html)。

接著便需要使用 OAuth 的
[ClientId](configuration.md#credential.bitbucketDataCenterOAuthClientId) 與
[ClientSecret](configuration.md#credential.bitbucketDataCenterOauthSecret) 從
AppLink 來設定本地端的 GCM 安裝。

#### 範例

```shell
git config --global credential.bitbucketDataCenterOAuthClientId 1111111111111111111
```

預設為未定義。

**另請參閱：[GCM_BITBUCKET_DATACENTER_CLIENTID](environment.md#GCM_BITBUCKET_DATACENTER_CLIENTID)**

---

### credential.bitbucketDataCenterOAuthClientSecret

若要搭配 Bitbucket DC 使用 OAuth，必須建立一個外部的、傳入的
[AppLink](https://confluence.atlassian.com/bitbucketserver/configure-an-incoming-link-1108483657.html)。

接著便需要使用 OAuth 的
[ClientId](configuration.md#credential.bitbucketDataCenterOAuthClientId) 與
[ClientSecret](configuration.md#credential.bitbucketDataCenterOauthSecret)
從 AppLink 來設定本地端的 GCM 安裝。

#### 範例

```shell
git config --global credential.bitbucketDataCenterOAuthClientSecret 222222222222222222222
```

預設為未定義。

**另請參閱：[GCM_BITBUCKET_DATACENTER_CLIENTSECRET](environment.md#GCM_BITBUCKET_DATACENTER_CLIENTSECRET)**

---

### credential.gitHubAccountFiltering

根據伺服器提示，為 GitHub 啟用或停用自動帳號篩選當有多個可用帳號時。此設定僅適用於具有 [Enterprise Managed Users][github-emu] 的 GitHub.com。

值|說明
-|-
`true` _(預設)_|根據伺服器提示篩選可用帳號。
`false`|顯示所有可用帳號。

#### 範例

```shell
git config --global credential.gitHubAccountFiltering "false"
```

**另請參閱：[GCM_GITHUB_ACCOUNTFILTERING][gcm-github-accountfiltering]**

---

### credential.gitHubAuthModes

覆寫 GitHub 身份驗證期間呈現的可用驗證模式。如果未設定此選項，則可用的驗證模式將會被自動偵測。

**注意：**此設定支援以逗號分隔的多個值。

值|驗證模式
-|-
_(未設定)_|自動偵測模式
`oauth`|擴充為：`browser, device`
`browser`|透過網頁瀏覽器的 OAuth 驗證 _(需要 GUI)_
`device`|使用裝置碼的 OAuth 驗證
`basic`|使用使用者名稱與密碼的基本驗證
`pat`|個人存取權杖 (pat) 型驗證

#### 範例

```shell
git config --global credential.gitHubAuthModes "oauth,basic"
```

**另請參閱：[GCM_GITHUB_AUTHMODES][gcm-github-authmodes]**

---

### credential.gitLabAuthModes

覆寫在 GitLab 驗證期間呈現的可用驗證模式。若未設定此選項，則可用的驗證模式將會被自動偵測。

**注意：**此設定支援以逗號分隔的多個值。

值|驗證模式
-|-
_(未設定)_|自動偵測模式
`browser`|透過網頁瀏覽器的 OAuth 驗證 _(需要 GUI)_
`basic`|使用使用者名稱與密碼的基本驗證
`pat`|個人存取權杖 (pat) 型驗證

#### 範例

```shell
git config --global credential.gitLabAuthModes "browser"
```

**另請參閱：[GCM_GITLAB_AUTHMODES][gcm-gitlab-authmodes]**

---

### credential.namespace

為在作業系統中讀取與寫入的憑證使用自訂命名空間前綴憑證儲存庫。憑證將以下列格式儲存
`{namespace}:{service}`。

預設值為 `git`。

#### 範例

```shell
git config --global credential.namespace "my-namespace"
```

**另請參閱：[GCM_NAMESPACE][gcm-namespace]**

---

### credential.credentialStore

選擇在支援的平台上使用的憑證儲存庫類型。

在 Windows 上的預設值為 `wincredman`，在 macOS 上為 `keychain`，且未設定在 Linux 上。

**注意：**關於設定秘密儲存庫的更多資訊，請參閱 [cred-stores][cred-stores]。

值|憑證儲存庫|平台
-|-|-
_(未設定)_|Windows：`wincredman`，macOS：`keychain`，Linux：_(無)_|-
`wincredman`|Windows 憑證管理員 (無法透過 SSH 使用)。|Windows
`dpapi`|DPAPI 保護的檔案。使用 [credential.dpapiStorePath][credential-dpapistorepath] 自訂 DPAPI 儲存庫位置|Windows
`keychain`|macOS 鑰匙圈。|macOS
`secretservice`|[freedesktop.org Secret Service API][freedesktop-ss] 透過 [libsecret][libsecret] (需要圖形介面來解鎖密碼集合)。|Linux
`gpg`|使用 GPG 儲存與 [pass][pass] 相容的加密檔案 (需要 GPG 與 `pass` 來初始化儲存庫)。|macOS, Linux
`cache`|Git 的內建 [credential cache][credential-cache]。|macOS, Linux
`plaintext`|以純文字檔案儲存憑證 (**不安全**)。使用 [`credential.plaintextStorePath`][credential-plaintextstorepath] 自訂純文字儲存庫位置。|Windows, macOS, Linux
`none`|不透過 GCM 儲存憑證。|Windows, macOS, Linux

#### 範例

```bash
git config --global credential.credentialStore gpg
```

**另請參閱：[GCM_CREDENTIAL_STORE][gcm-credential-store]**

---

### credential.cacheOptions

將 [options][cache-options] 傳遞至 Git 憑證快取，當 [`credential.credentialStore`][credential-credentialstore] 設為 `cache` 時。這可讓您選取不同的憑證快取時間 (預設為 900 秒)，方法是透過傳遞
`"--timeout <seconds>"`。使用其他選項 (如 `--socket`) 尚未經過測試且不受支援，但沒有理由它無法運作。

預設為空。

#### 範例

```shell
git config --global credential.cacheOptions "--timeout 300"
```

**另請參閱：[GCM_CREDENTIAL_CACHE_OPTIONS][gcm-credential-cache-options]**

---

### credential.plaintextStorePath

指定一個自訂目錄來儲存純文字憑證檔案，當 [`credential.credentialStore`][credential-credentialstore] 設為 `plaintext` 時。

預設值為 `~/.gcm/store` 或 `%USERPROFILE%\.gcm\store`。

#### 範例

```shell
git config --global credential.plaintextStorePath /mnt/external-drive/credentials
```

**另請參閱：[GCM_PLAINTEXT_STORE_PATH][gcm-plaintext-store-path]**

---

### credential.dpapiStorePath

指定一個自訂目錄來儲存受 DPAPI 保護的憑證檔案，當 [`credential.credentialStore`][credential-credentialstore] 設為 `dpapi` 時。

預設值為 `%USERPROFILE%\.gcm\dpapi_store`。

#### 範例

```batch
git config --global credential.dpapiStorePath D:\credentials
```

**另請參閱：[GCM_DPAPI_STORE_PATH][gcm-dpapi-store-path]**

---

### credential.gpgPassStorePath

指定一個自訂目錄，用以儲存經 GPG 加密且與 [pass][pass] 相容的憑證檔案於 [`credential.credentialStore`][credential-credentialstore] 設為 `gpg` 時。

預設值為 `~/.password-store` 或 `%USERPROFILE%\.password-store`。

#### 範例

```shell
git config --global credential.gpgPassStorePath /mnt/external-drive/.password-store
```

**注意：** [pass][pass] 所使用的密碼儲存庫位置可被 `PASSWORD_STORE_DIR` 環境變數覆寫，詳情請參閱 [man page][pass-man]。

---

### credential.msauthFlow

指定在執行 Microsoft 驗證時，且需要互動式流程時，應使用何種驗證流程。

預設為 `auto`。

**注意：** 若 [`credential.msauthUseBroker`][credential-msauthusebroker] 設為 `true` 且作業系統驗證代理程式可用，則所有流程將會委派給該代理程式。如果這兩項條件都成立，則 `credential.msauthFlow` 的值將沒有任何作用。

值|驗證流程
-|-
`auto` _(預設)_|根據目前的環境與平台選取最佳選項。
`embedded`|顯示一個內嵌網頁檢視控制項的視窗。
`system`|開啟使用者的預設網頁瀏覽器。
`devicecode`|顯示裝置碼。

#### 範例

```shell
git config --global credential.msauthFlow devicecode
```

**另請參閱：[GCM_MSAUTH_FLOW][gcm-msauth-flow]**

---

### credential.msauthUseBroker _(實驗性)_

在可用時使用作業系統帳戶管理員。

預設為 `false`。在某些雲端託管環境中，當使用公司或學校帳戶時，例如 [Microsoft DevBox][devbox]，預設值為 `true`。

這些預設值未來可能會變更。

_**注意：** 在 Windows 上啟用此選項之前，請檢閱 [Windows Broker][wam] 的詳細資訊，以了解這對您本機 Windows 使用者帳戶的意義。_

值|說明
-|-
`true`|使用作業系統帳戶管理員作為驗證代理人。
`false` _(預設)_|不使用代理人。

#### 範例

```shell
git config --global credential.msauthUseBroker true
```

**另請參閱：[GCM_MSAUTH_USEBROKER][gcm-msauth-usebroker]**

---

### credential.msauthUseDefaultAccount _(實驗性)_

當代理人啟用時，預設使用目前的作業系統帳戶。

預設為 `false`。在某些雲端託管環境中，當使用公司或學校帳戶時，例如 [Microsoft DevBox][devbox]，預設值為 `true`。

這些預設值未來可能會變更。

值|說明
-|-
`true`|預設使用目前的作業系統帳戶。
`false` _(預設)_|預設不假設使用任何帳戶。

#### 範例

```shell
git config --global credential.msauthUseDefaultAccount true
```

**另請參閱：[GCM_MSAUTH_USEDEFAULTACCOUNT][gcm-msauth-usedefaultaccount]**

---

### credential.useHttpPath

告知 Git 傳遞整個儲存庫 URL，而不僅僅是主機名稱，當呼叫憑證提供者時。（此設定[來自 Git 本身][use-http-path]，而非 GCM。）

預設為 `false`。

**注意：** GCM 預設會在安裝後，為 `dev.azure.com` (Azure Repos) 主機將此值設定為 `true`。

這是因為僅憑 `dev.azure.com` 的資訊不足以判斷正確的 Azure 驗證授權單位 - 我們需要路徑的一部分。其後果是，對於 `dev.azure.com` 遠端 URL，我們不支援針對完整路徑儲存憑證。我們總是針對 `dev.azure.com/org-name` 存根進行儲存。

為了使用 Azure Repos 並針對完整路徑 URL 儲存憑證，您必須改用 `org-name.visualstudio.com` 遠端 URL 格式。

值|Git 行為
-|-
`false` _(預設)_|Git 將僅使用 `user` 和 `hostname` 來查詢憑證。
`true`|Git 將使用完整的儲存庫 URL 來查詢憑證。

#### 範例

在 Windows 上使用 GitHub，對於登入名稱為 `alice` 的使用者，且 `credential.useHttpPath` 設為 `false` (或未設定)，下列遠端 URL 將使用相同的憑證：

```text
Credential: "git:https://github.com" (user = alice)

   https://github.com/foo/bar
   https://github.com/contoso/widgets
   https://alice@github.com/contoso/widgets
```

```text
Credential: "git:https://bob@github.com" (user = bob)

   https://bob@github.com/foo/bar
   https://bob@github.com/example/myrepo
```

在相同使用者下，但將 `credential.useHttpPath` 設為 `true`，這些憑證將被使用：

```text
Credential: "git:https://github.com/foo/bar" (user = alice)

   https://github.com/foo/bar
```

```text
Credential: "git:https://github.com/contoso/widgets" (user = alice)

   https://github.com/contoso/widgets
   https://alice@github.com/contoso/widgets
```

```text
Credential: "git:https://bob@github.com/foo/bar" (user = bob)

   https://bob@github.com/foo/bar
```

```text
Credential: "git:https://bob@github.com/example/myrepo" (user = bob)

   https://bob@github.com/example/myrepo
```

---

### credential.azreposCredentialType

指定 Azure Repos 主機提供者應傳回的憑證類型。

預設值為 `pat`。在某些雲端託管環境中，當使用公司或學校帳戶時，例如 [Microsoft DevBox][devbox]，預設值為 `oauth`。

值|說明
-|-
`pat`|Azure DevOps 個人存取權杖
`oauth`|Microsoft 身分識別 OAuth 權杖 (AAD 或 MSA 權杖)

更多關於 [Azure Access tokens][azure-tokens] 的資訊。

#### 範例

```shell
git config --global credential.azreposCredentialType oauth
```

**另請參閱：[GCM_AZREPOS_CREDENTIALTYPE][gcm-azrepos-credentialtype]**

---

### credential.azreposManagedIdentity

使用 [Managed Identity][managed-identity] 向 Azure Repos 進行驗證。

值 `system` 會告知 GCM 使用系統指派的受控識別。

若要指定使用者指派的受控識別，請使用 `id://{clientId}` 格式其中 `{clientId}` 是受控識別的用戶端 ID。或者，任何類似 GUID 的值也將被解譯為使用者指派的受控識別用戶端 ID。

若要指定與 Azure 資源相關聯的受控識別，您可以使用 `resource://{resourceId}` 格式，其中 `{resourceId}` 是資源的 ID。

更多關於受控識別的資訊，請參閱 Azure DevOps [文件][azrepos-sp-mid]。

值|說明
-|-
`system`|系統指派的受控識別
`[guid]`|具有指定用戶端 ID 的使用者指派受控識別
`id://[guid]`|具有指定用戶端 ID 的使用者指派受控識別
`resource://[guid]`|關聯資源的使用者指派受控識別

```shell
git config --global credential.azreposManagedIdentity "id://11111111-1111-1111-1111-111111111111"
```

**另請參閱：[GCM_AZREPOS_MANAGEDIDENTITY][gcm-azrepos-credentialmanagedidentity]**

---

### credential.azreposServicePrincipal

指定 [服務主體][service-principal] 的用戶端和租用戶 ID 在為 Azure Repos 執行 Microsoft 驗證時使用。

此設定的值應為 `{tenantId}/{clientId}` 格式。

如果您設定此值，則還必須設定至少一種驗證機制：

- [credential.azreposServicePrincipalSecret][credential-azrepos-sp-secret]
- [credential.azreposServicePrincipalCertificateThumbprint][credential-azrepos-sp-cert-thumbprint]
- [credential.azreposServicePrincipalCertificateSendX5C][credential-azrepos-sp-cert-x5c]

如需有關服務主體的更多資訊，請參閱 Azure DevOps [文件][azrepos-sp-mid]。

#### 範例

```shell
git config --global credential.azreposServicePrincipal "11111111-1111-1111-1111-111111111111/22222222-2222-2222-2222-222222222222"
```

**另請參閱：[GCM_AZREPOS_SERVICE_PRINCIPAL][gcm-azrepos-service-principal]**

---

### credential.azreposServicePrincipalSecret

指定 [服務主體][service-principal] 的用戶端密碼，當為 Azure Repos 執行 Microsoft 驗證，且 [credential.azreposServicePrincipalSecret][credential-azrepos-sp] 已設定時。

#### 範例

```shell
git config --global credential.azreposServicePrincipalSecret "da39a3ee5e6b4b0d3255bfef95601890afd80709"
```

**另請參閱：[GCM_AZREPOS_SP_SECRET][gcm-azrepos-sp-secret]**

---

### credential.azreposServicePrincipalCertificateThumbprint

指定憑證的指紋，用於當[服務主體][service-principal] 為 Azure Repos 進行驗證，且 [GCM_AZREPOS_SERVICE_PRINCIPAL][credential-azrepos-sp] 已設定時。

#### 範例

```shell
git config --global credential.azreposServicePrincipalCertificateThumbprint "9b6555292e4ea21cbc2ebd23e66e2f91ebbe92dc"
```

**另請參閱：[GCM_AZREPOS_SP_CERT_THUMBPRINT][gcm-azrepos-sp-cert-thumbprint]**

---

### credential.azreposServicePrincipalCertificateSendX5C

當使用憑證進行 [服務主體][service-principal] 驗證時，此設定指定是否應將 X5C 宣告傳送至 STS。傳送 x5c 可讓應用程式開發人員在 Azure AD 中輕鬆實現憑證輪替：此方法會將公開憑證連同權杖請求一起傳送至 Azure AD，以便 Azure AD 能根據受信任的簽發者原則來驗證主體名稱。這讓應用程式管理員無需明確管理憑證輪替。詳情請參閱 [https://aka.ms/msal-net-sni](https://aka.ms/msal-net-sni)。

#### 範例

```shell
git config --global credential.azreposServicePrincipalCertificateSendX5C true
```
**另請參閱：[GCM_AZREPOS_SP_CERT_SEND_X5C][gcm-azrepos-sp-cert-x5c]**

---

### trace2.normalTarget

開啟 Trace2 一般格式追蹤 - 請參閱 [Git 的 Trace2 一般格式文件][trace2-normal-docs] 以取得更多詳細資訊。

#### 範例

```shell
git config --global trace2.normalTarget true
```

如果 `trace2.normalTarget` 的值是位於現有目錄中檔案的完整路徑，日誌將會附加至該檔案。

如果 `trace2.normalTarget` 的值為 `true` 或 `1`，日誌會寫入至標準錯誤。

預設為停用。

**另請參閱：[GIT_TRACE2][trace2-normal-env]**

---

### trace2.eventTarget

開啟 Trace2 事件格式追蹤 - 請參閱 [Git 的 Trace2 事件格式文件][trace2-event-docs] 以取得更多詳細資訊。

#### 範例

```shell
git config --global trace2.eventTarget true
```

如果 `trace2.eventTarget` 的值是位於現有目錄中檔案的完整路徑，日誌將會附加至該檔案。

如果 `trace2.eventTarget` 的值為 `true` 或 `1`，日誌將會寫入標準錯誤輸出。

預設為停用。

**另請參閱：[GIT_TRACE2_EVENT][trace2-event-env]**

---

### trace2.perfTarget

開啟 Trace2 效能格式追蹤 - 請參閱 [Git 的 Trace2 效能格式文件][trace2-performance-docs] 以取得更多詳細資訊。

#### 範例

```shell
git config --global trace2.perfTarget true
```

如果 `trace2.perfTarget` 的值為一個位於現有目錄中檔案的完整路徑，日誌將會附加到該檔案。

如果 `trace2.perfTarget` 的值為 `true` 或 `1`，日誌將會寫入標準錯誤輸出。

預設為停用。

**另請參閱：[GIT_TRACE2_PERF][trace2-performance-env]**

[auto-detection]: autodetect.md
[azure-tokens]: azrepos-users-and-tokens.md
[use-http-path]: https://git-scm.com/docs/gitcredentials/#Documentation/gitcredentials.txt-useHttpPath
[credential-credentialstore]: #credentialcredentialstore
[credential-dpapistorepath]: #credentialdpapistorepath
[credential-interactive]: #credentialinteractive
[credential-msauthusebroker]: #credentialmsauthusebroker-experimental
[credential-plaintextstorepath]: #credentialplaintextstorepath
[credential-cache]: https://git-scm.com/docs/git-credential-cache
[cred-stores]: credstores.md
[devbox]: https://azure.microsoft.com/en-us/products/dev-box
[enterprise-config]: enterprise-config.md
[envars]: environment.md
[freedesktop-ss]: https://specifications.freedesktop.org/secret-service-spec/
[gcm-allow-windowsauth]: environment.md#GCM_ALLOW_WINDOWSAUTH
[gcm-allow-unsafe-remotes]: environment.md#GCM_ALLOW_UNSAFE_REMOTES
[gcm-authority]: environment.md#GCM_AUTHORITY-deprecated
[gcm-autodetect-timeout]: environment.md#GCM_AUTODETECT_TIMEOUT
[gcm-azrepos-credentialtype]: environment.md#GCM_AZREPOS_CREDENTIALTYPE
[gcm-azrepos-credentialmanagedidentity]: environment.md#GCM_AZREPOS_MANAGEDIDENTITY
[gcm-bitbucket-always-refresh-credentials]: environment.md#GCM_BITBUCKET_ALWAYS_REFRESH_CREDENTIALS
[gcm-bitbucket-authmodes]: environment.md#GCM_BITBUCKET_AUTHMODES
[gcm-credential-cache-options]: environment.md#GCM_CREDENTIAL_CACHE_OPTIONS
[gcm-credential-store]: environment.md#GCM_CREDENTIAL_STORE
[gcm-debug]: environment.md#GCM_DEBUG
[gcm-dpapi-store-path]: environment.md#GCM_DPAPI_STORE_PATH
[gcm-github-accountfiltering]: environment.md#GCM_GITHUB_ACCOUNTFILTERING
[gcm-github-authmodes]: environment.md#GCM_GITHUB_AUTHMODES
[gcm-gitlab-authmodes]:environment.md#GCM_GITLAB_AUTHMODES
[gcm-gui-prompt]: environment.md#GCM_GUI_PROMPT
[gcm-gui-software-rendering]: environment.md#GCM_GUI_SOFTWARE_RENDERING
[gcm-http-proxy]: environment.md#GCM_HTTP_PROXY-deprecated
[gcm-interactive]: environment.md#GCM_INTERACTIVE
[gcm-msauth-flow]: environment.md#GCM_MSAUTH_FLOW
[gcm-msauth-usebroker]: environment.md#GCM_MSAUTH_USEBROKER-experimental
[gcm-msauth-usedefaultaccount]: environment.md#GCM_MSAUTH_USEDEFAULTACCOUNT-experimental
[gcm-namespace]: environment.md#GCM_NAMESPACE
[gcm-plaintext-store-path]: environment.md#GCM_PLAINTEXT_STORE_PATH
[gcm-provider]: environment.md#GCM_PROVIDER
[gcm-trace]: environment.md#GCM_TRACE
[gcm-trace-secrets]: environment.md#GCM_TRACE_SECRETS
[gcm-trace-msauth]: environment.md#GCM_TRACE_MSAUTH
[github-emu]: https://docs.github.com/en/enterprise-cloud@latest/admin/identity-and-access-management/using-enterprise-managed-users-for-iam/about-enterprise-managed-users
[usage]: usage.md
[git-config-http-proxy]: https://git-scm.com/docs/git-config#Documentation/git-config.txt-httpproxy
[http-proxy]: netconfig.md#http-proxy
[autodetect]: autodetect.md
[libsecret]: https://wiki.gnome.org/Projects/Libsecret
[managed-identity]: https://docs.microsoft.com/en-us/azure/active-directory/managed-identities-azure-resources/overview
[provider-migrate]: migration.md#gcm_authority
[cache-options]: https://git-scm.com/docs/git-credential-cache#_options
[pass]: https://www.passwordstore.org/
[pass-man]: https://git.zx2c4.com/password-store/about/
[trace2-normal-docs]: https://git-scm.com/docs/api-trace2#_the_normal_format_target
[trace2-normal-env]: environment.md#GIT_TRACE2
[trace2-event-docs]: https://git-scm.com/docs/api-trace2#_the_event_format_target
[trace2-event-env]: environment.md#GIT_TRACE2_EVENT
[trace2-performance-docs]: https://git-scm.com/docs/api-trace2#_the_performance_format_target
[trace2-performance-env]: environment.md#GIT_TRACE2_PERF
[wam]: windows-broker.md
[service-principal]: https://docs.microsoft.com/en-us/azure/active-directory/develop/app-objects-and-service-principals
[azrepos-sp-mid]: https://learn.microsoft.com/en-us/azure/devops/integrate/get-started/authentication/service-principal-managed-identity
[credential-azrepos-sp]: #credentialazreposserviceprincipal
[credential-azrepos-sp-secret]: #credentialazreposserviceprincipalsecret
[credential-azrepos-sp-cert-thumbprint]: #credentialazreposserviceprincipalcertificatethumbprint
[credential-azrepos-sp-cert-x5c]: #credentialazreposserviceprincipalcertificatesendx5c
[gcm-azrepos-service-principal]: environment.md#GCM_AZREPOS_SERVICE_PRINCIPAL
[gcm-azrepos-sp-secret]: environment.md#GCM_AZREPOS_SP_SECRET
[gcm-azrepos-sp-cert-thumbprint]: environment.md#GCM_AZREPOS_SP_CERT_THUMBPRINT
[gcm-azrepos-sp-cert-x5c]: environment.md#GCM_AZREPOS_SP_CERT_SEND_X5C
