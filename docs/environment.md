# 環境變數

[Git Credential Manager][gcm] 對於大多數使用者來說是開箱即用的。提供了一些組態選項來自訂或調整其行為。

Git Credential Manager (GCM) 可以使用環境變數進行設定。
**環境變數的優先級高於[設定][configuration]選項和企業系統管理員的[預設值][default-values]**。

有關 GCM 能理解的完整環境變數列表，請參閱下文。

## 可用設定

### GCM_TRACE

啟用所有活動的追蹤記錄。
將 Git 和 GCM 設定為追蹤到相同的位置通常是理想的作法，且 GCM 與 `GIT_TRACE` 相容並協同運作。

#### 範例

##### Windows

```batch
SET GIT_TRACE=%UserProfile%\git.log
SET GCM_TRACE=%UserProfile%\git.log
```

##### macOS/Linux

```bash
export GIT_TRACE=$HOME/git.log
export GCM_TRACE=$HOME/git.log
```

如果 `GCM_TRACE` 的值是現有目錄中檔案的完整路徑，記錄將會附加到該檔案。

如果 `GCM_TRACE` 的值是 `true` 或 `1`，記錄將會寫入標準錯誤。

預設為停用。

**另請參閱：[credential.trace][credential-trace]**

---

### GCM_TRACE_SECRETS

啟用追蹤秘密和敏感資訊，這些資訊預設在追蹤輸出中會被遮罩。需要同時啟用 `GCM_TRACE`。

#### 範例

##### Windows

```batch
SET GCM_TRACE=%UserProfile%\gcm.log
SET GCM_TRACE_SECRETS=1
```

##### macOS/Linux

```bash
export GCM_TRACE=$HOME/gcm.log
export GCM_TRACE_SECRETS=1
```

如果 `GCM_TRACE_SECRETS` 的值是 `true` 或 `1`，追蹤記錄將包含秘密資訊。

預設為停用。

**另請參閱：[credential.traceSecrets][credential-trace-secrets]**

---

### GCM_TRACE_MSAUTH

在 GCM 追蹤輸出中包含 Microsoft Authentication Library (MSAL) 的記錄。需要同時啟用 `GCM_TRACE`。

#### 範例

##### Windows

```batch
SET GCM_TRACE=%UserProfile%\gcm.log
SET GCM_TRACE_MSAUTH=1
```

##### macOS/Linux

```bash
export GCM_TRACE=$HOME/gcm.log
export GCM_TRACE_MSAUTH=1
```

如果 `GCM_TRACE_MSAUTH` 的值是 `true` 或 `1`，追蹤記錄將包含詳細的 MSAL 記錄。

預設為停用。

**另請參閱：[credential.traceMsAuth][credential-trace-msauth]**

---

### GCM_DEBUG

在 GCM 啟動時暫停執行，以等待偵錯工具附加。

#### 範例

##### Windows

```batch
SET GCM_DEBUG=1
```

##### macOS/Linux

```bash
export GCM_DEBUG=1
```

預設為停用。

**另請參閱：[credential.debug][credential-debug]**

---

### GCM_INTERACTIVE

允許或停用 GCM 與使用者互動（顯示 GUI 或 TTY 提示）。如果需要互動但已被停用，則會傳回錯誤。

這在無周邊（headless）和無人值守的環境中（例如建置伺服器）很有用，在這些環境中，失敗比無限期地掛起等待一個不存在的使用者要好。

若要停用互動性，請將此設定為 `false` 或 `0`。

#### 相容性

在 GCM 的舊版本中，此設定有不同的行為並接受其他值。下表總結了行為的變化以及舊值（如 `never`）的對應關係：

值|舊意義|新意義
-|-|-
`auto`|如果需要則提示 – 盡可能使用快取憑證|_(不變)_
`never`, `false`| 永不提示 – 如果需要互動則失敗|_(不變)_
`always`, `force`, `true`|總是提示 – 不使用快取憑證|如果需要則提示（與舊的 `auto` 值相同）

#### 範例

##### Windows

```batch
SET GCM_INTERACTIVE=0
```

##### macOS/Linux

```bash
export GCM_INTERACTIVE=0
```

預設為啟用。

**另請參閱：[credential.interactive][credential-interactive]**

---

### GCM_PROVIDER

定義驗證時要使用的主機提供者。

ID|提供者
-|-
`auto` _(預設)_|_\[自動\]_ ([了解更多][autodetect])
`azure-repos`|Azure Repos
`github`|GitHub
`gitlab`|GitLab _(支援瀏覽器中的 OAuth、個人存取權杖和基本驗證)_
`generic`|通用（任何其他未列出的提供者）

自動提供者選擇是基於遠端 URL。

此設定通常與範圍 URL 一起使用，以將一組特定的遠端 URL 對應到提供者，例如將主機標記為 GitHub Enterprise 執行個體。

#### 範例

##### Windows

```batch
SET GCM_PROVIDER=github
```

##### macOS/Linux

```bash
export GCM_PROVIDER=github
```

**另請參閱：[credential.provider][credential-provider]**

---

### GCM_AUTHORITY _(已棄用)_

> 此設定已棄用，應由 `GCM_PROVIDER` 及其對應的提供者 ID 值取代。
>
> 更多資訊請參閱[遷移指南][migration-guide]。

根據提供者支援的授權單位來選擇驗證時要使用的主機提供者。

授權單位|提供者
-|-
`auto` _(預設)_|_\[自動\]_
`msa`, `microsoft`, `microsoftaccount`, `aad`, `azure`, `azuredirectory`, `live`, `liveconnect`, `liveid`|Azure Repos _(支援 Microsoft 驗證)_
`github`|GitHub _(支援 GitHub 驗證)_
`gitlab`|GitLab _(支援瀏覽器中的 OAuth、個人存取權杖和基本驗證)_
`basic`, `integrated`, `windows`, `kerberos`, `ntlm`, `tfs`, `sso`|通用 _(支援基本和 Windows 整合驗證)_

#### 範例

##### Windows

```batch
SET GCM_AUTHORITY=github
```

##### macOS/Linux

```bash
export GCM_AUTHORITY=github
```

**另請參閱：[credential.authority][credential-authority]**

---

### GCM_GUI_PROMPT

允許或停用 GCM 顯示 GUI 提示。如果存在等效的終端機/文字型提示，則會改為顯示該提示。

若要停用所有互動性，請參閱 [GCM_INTERACTIVE][gcm-interactive]。

#### 範例

##### Windows

```batch
SET GCM_GUI_PROMPT=0
```

##### macOS/Linux

```bash
export GCM_GUI_PROMPT=0
```

預設為啟用。

**另請參閱：[credential.guiPrompt][credential-guiprompt]**

---

### GCM_GUI_SOFTWARE_RENDERING

強制使用軟體渲染來顯示 GUI 提示。

這目前僅適用於 Windows。

#### 範例

##### Windows

```batch
SET GCM_GUI_SOFTWARE_RENDERING=1
```

##### macOS/Linux

```bash
export GCM_GUI_SOFTWARE_RENDERING=1
```

預設為 false（在可用時使用硬體加速）。

> [!NOTE]
> Windows on ARM 裝置預設使用軟體渲染以解決已知的 Avalonia 問題：<https://github.com/AvaloniaUI/Avalonia/issues/10405>

**另請參閱：[credential.guiSoftwareRendering][credential-guisoftwarerendering]**

---

### GCM_ALLOW_UNSAFE_REMOTES

允許將憑證傳輸到不安全的遠端 URL，例如未加密的 HTTP URL。不建議一般使用此設定，僅在必要時使用。

預設為 false（不允許不安全的遠端 URL）。

#### 範例

##### Windows

```batch
SET GCM_ALLOW_UNSAFE_REMOTES=true
```

##### macOS/Linux

```bash
export GCM_ALLOW_UNSAFE_REMOTES=true
```

**另請參閱：[credential.allowUnsafeRemotes][credential-allowunsaferemotes]**

---

### GCM_AUTODETECT_TIMEOUT

設定 GCM 在主機提供者自動偵測探測期間等待網路回應的最長時間（以毫秒為單位）。

更多資訊請參閱[自動偵測][autodetect]。

**注意：** 使用負值或零值可完全停用探測。

預設為 2000 毫秒（2 秒）。

#### 範例

##### Windows

```batch
SET GCM_AUTODETECT_TIMEOUT=-1
```

##### macOS/Linux

```bash
export GCM_AUTODETECT_TIMEOUT=-1
```

**另請參閱：[credential.autoDetectTimeout][credential-autodetecttimeout]**

---

### GCM_ALLOW_WINDOWSAUTH

允許為通用主機提供者偵測 Windows 整合驗證 (WIA) 支援。將此值設定為 `false` 將阻止使用 WIA，並在使用通用主機提供者時強制進行基本驗證提示。

**注意：** WIA 僅在 Windows 上受支援。

**注意：** WIA 是 NTLM 和 Kerberos（以及 Negotiate）的總稱。

值|WIA 偵測
-|-
`true`, `1`, `yes`, `on` _(預設)_|允許
`false`, `0`, `no`, `off`|不允許

#### 範例

##### Windows

```batch
SET GCM_ALLOW_WINDOWSAUTH=0
```

##### macOS/Linux

```bash
export GCM_ALLOW_WINDOWSAUTH=0
```

**另請參閱：[credential.allowWindowsAuth][credential-allowwindowsauth]**

---

### GCM_HTTP_PROXY _(已棄用)_

> 此設定已棄用，應由[標準的 `http.proxy` Git 設定選項][git-httpproxy]取代。
>
> 更多資訊請參閱 [HTTP 代理伺服器設定][network-http-proxy]。

設定 GCM 使用代理伺服器進行網路操作。

**注意：** Git 本身並*不*遵守此設定；這*僅*影響 GCM。

#### Windows

```batch
SET GCM_HTTP_PROXY=http://john.doe:password@proxy.contoso.com
```

#### macOS/Linux

```bash
export GCM_HTTP_PROXY=http://john.doe:password@proxy.contoso.com
```

**另請參閱：[credential.httpProxy][credential-httpproxy]**

---

### GCM_BITBUCKET_AUTHMODES

覆寫 Bitbucket 驗證期間顯示的可用驗證模式。如果未設定此選項，則會自動偵測可用的驗證模式。

**注意：** 此設定僅適用於 Bitbucket.org，不適用於 Server 或 DC 執行個體。

**注意：** 此設定支援以逗號分隔的多個值。

值|驗證模式
-|-
_(未設定)_|自動偵測模式
`oauth`|基於 OAuth 的驗證
`basic`|基於基本/PAT 的驗證

#### Windows

```batch
SET GCM_BITBUCKET_AUTHMODES="oauth,basic"
```

#### macOS/Linux

```bash
export GCM_BITBUCKET_AUTHMODES="oauth,basic"
```

**另請參閱：[credential.bitbucketAuthModes][credential-bitbucketauthmodes]**

---

### GCM_BITBUCKET_ALWAYS_REFRESH_CREDENTIALS

強制 GCM 忽略任何現有的已儲存基本驗證或 OAuth 存取權杖，並在將憑證返回給 Git 之前，總是執行刷新憑證的流程。

這對於 OAuth 憑證尤其重要。Bitbucket.org 存取權杖在 2 小時後過期，之後必須使用刷新權杖來取得新的存取權杖。

如果平均提交頻率低於每 2 小時一次，啟用此選項將在使用 Oauth2 與 Bitbucket.org 互動時提高效能。

啟用此選項將在使用基本驗證時降低效能，因為每次都需要使用者重新輸入憑證。

值|返回前刷新憑證
-|-
`true`, `1`, `yes`, `on` |總是
`false`, `0`, `no`, `off`_(預設)_|僅當發現憑證無效時

#### Windows

```batch
SET GCM_BITBUCKET_ALWAYS_REFRESH_CREDENTIALS=1
```

#### macOS/Linux

```bash
export GCM_BITBUCKET_ALWAYS_REFRESH_CREDENTIALS=1
```

預設為 false/停用。

**另請參閱：[credential.bitbucketAlwaysRefreshCredentials](configuration.md#credentialbitbucketAlwaysRefreshCredentials)**

---

### GCM_BITBUCKET_VALIDATE_STORED_CREDENTIALS

強制 GCM 在將任何已儲存的憑證返回給 Git 之前對其進行驗證。它透過呼叫需要驗證的 REST API 資源來實現此目的。

停用此選項可減少 GCM 在擷取憑證時的 HTTP 流量。這可能會改善使用者效能，但會增加 Git 遠端呼叫無法向主機進行驗證的次數，因此需要使用者重試 Git 遠端呼叫。

啟用此選項有助於確保 Git 始終獲得有效的憑證。

值|驗證憑證
-|-
`true`, `1`, `yes`, `on`_(預設)_|總是
`false`, `0`, `no`, `off`|永不

#### Windows

```batch
SET GCM_BITBUCKET_VALIDATE_STORED_CREDENTIALS=1
```

#### macOS/Linux

```bash
export GCM_BITBUCKET_VALIDATE_STORED_CREDENTIALS=1
```

預設為 true/啟用。

**另請參閱：[credential.bitbucketValidateStoredCredentials](configuration.md#credentialbitbucketValidateStoredCredentials)**

---

### GCM_BITBUCKET_DATACENTER_CLIENTID

若要搭配 Bitbucket DC 使用 OAuth，必須建立一個外部的、傳入的 [AppLink](https://confluence.atlassian.com/bitbucketserver/configure-an-incoming-link-1108483657.html)。

然後，必須使用 AppLink 中的 OAuth [ClientId](environment.md#GCM_BITBUCKET_DATACENTER_CLIENTID) 和 [ClientSecret](environment.md#GCM_BITBUCKET_DATACENTER_CLIENTSECRET) 來設定本地 GCM 安裝。

#### Windows

```batch
SET GCM_BITBUCKET_DATACENTER_CLIENTID=1111111111111111111
```

#### macOS/Linux

```bash
export GCM_BITBUCKET_DATACENTER_CLIENTID=1111111111111111111
```

預設為未定義。

**另請參閱：[credential.bitbucketDataCenterOAuthClientId](configuration.md#credentialbitbucketDataCenterOAuthClientId)**

---

### GCM_BITBUCKET_DATACENTER_CLIENTSECRET

若要搭配 Bitbucket DC 使用 OAuth，必須建立一個外部的、傳入的 [AppLink](https://confluence.atlassian.com/bitbucketserver/configure-an-incoming-link-1108483657.html)。

然後，必須使用 AppLink 中的 OAuth [ClientId](environment.md#GCM_BITBUCKET_DATACENTER_CLIENTID) 和 [ClientSecret](environment.md#GCM_BITBUCKET_DATACENTER_CLIENTSECRET) 來設定本地 GCM 安裝。

#### Windows

```batch
SET GCM_BITBUCKET_DATACENTER_CLIENTSECRET=222222222222222222222
```

#### macOS/Linux

```bash
export GCM_BITBUCKET_DATACENTER_CLIENTSECRET=222222222222222222222
```

預設為未定義。

**另請參閱：[credential.bitbucketDataCenterOAuthClientSecret](configuration.md#credentialbitbucketDataCenterOAuthClientSecret)**

---

### GCM_GITHUB_ACCOUNTFILTERING

當有多個可用帳戶時，根據伺服器提示啟用或停用 GitHub 的自動帳戶篩選。此設定僅適用於具有[企業受管理使用者][github-emu]的 GitHub.com。

值|描述
-|-
`true` _(預設)_|根據伺服器提示篩選可用帳戶。
`false`|顯示所有可用帳戶。

#### Windows

```batch
SET GCM_GITHUB_ACCOUNTFILTERING=false
```

#### macOS/Linux

```bash
export GCM_GITHUB_ACCOUNTFILTERING=false
```

**另請參閱：[credential.gitHubAccountFiltering][credential-githubaccountfiltering]**

---

### GCM_GITHUB_AUTHMODES

覆寫 GitHub 驗證期間顯示的可用驗證模式。如果未設定此選項，則會自動偵測可用的驗證模式。

**注意：** 此設定支援以逗號分隔的多個值。

值|驗證模式
-|-
_(未設定)_|自動偵測模式
`oauth`|展開為：`browser, device`
`browser`|透過網頁瀏覽器進行 OAuth 驗證 _(需要 GUI)_
`device`|使用裝置代碼進行 OAuth 驗證
`basic`|使用使用者名稱和密碼進行基本驗證
`pat`|基於個人存取權杖 (pat) 的驗證

#### Windows

```batch
SET GCM_GITHUB_AUTHMODES="oauth,basic"
```

#### macOS/Linux

```bash
export GCM_GITHUB_AUTHMODES="oauth,basic"
```

**另請參閱：[credential.gitHubAuthModes][credential-githubauthmodes]**

---

### GCM_GITLAB_AUTHMODES

覆寫 GitLab 驗證期間顯示的可用驗證模式。如果未設定此選項，則會自動偵測可用的驗證模式。

**注意：** 此設定支援以逗號分隔的多個值。

值|驗證模式
-|-
_(未設定)_|自動偵測模式
`browser`|透過網頁瀏覽器進行 OAuth 驗證 _(需要 GUI)_
`basic`|使用使用者名稱和密碼進行基本驗證
`pat`|基於個人存取權杖 (pat) 的驗證

#### Windows

```batch
SET GCM_GITLAB_AUTHMODES="browser"
```

#### macOS/Linux

```bash
export GCM_GITLAB_AUTHMODES="browser"
```

**另請參閱：[credential.gitLabAuthModes][credential-gitlabauthmodes]**

---

### GCM_NAMESPACE

在作業系統憑證存放區中讀取和寫入憑證時，使用自訂的命名空間前綴。憑證將以 `{namespace}:{service}` 的格式儲存。

預設值為 `git`。

#### Windows

```batch
SET GCM_NAMESPACE="my-namespace"
```

#### macOS/Linux

```bash
export GCM_NAMESPACE="my-namespace"
```

**另請參閱：[credential.namespace][credential-namespace]**

---

### GCM_CREDENTIAL_STORE

在支援的平台上選擇要使用的憑證存放區類型。

在 Windows 上的預設值是 `wincredman`，在 macOS 上是 `keychain`，在 Linux 上則未設定。

**注意：** 關於設定秘密存放區的更多資訊，請參閱[憑證存放區文件][credential-stores]。

值|憑證存放區|平台
-|-|-
_(未設定)_|Windows: `wincredman`, macOS: `keychain`, Linux: _(無)_|-
`wincredman`|Windows Credential Manager (無法透過 SSH 使用)。|Windows
`dpapi`|DPAPI 保護的檔案。使用 [`GCM_DPAPI_STORE_PATH`][gcm-dpapi-store-path] 自訂 DPAPI 存放區位置|Windows
`keychain`|macOS Keychain。|macOS
`secretservice`|[freedesktop.org Secret Service API][freedesktop-ss] 透過 [libsecret][libsecret] (需要圖形介面來解鎖秘密集合)。|Linux
`gpg`|使用 GPG 儲存與 [`pass` 工具][passwordstore] 相容的加密檔案 (需要 GPG 和 `pass` 來初始化存放區)。|macOS, Linux
`cache`|Git 內建的[憑證快取][git-credential-cache]。|Windows, macOS, Linux
`plaintext`|將憑證儲存在純文字檔案中 (**不安全**)。使用 [`GCM_PLAINTEXT_STORE_PATH`][gcm-plaintext-store-path] 自訂純文字存放區位置。|Windows, macOS, Linux
`none`|不透過 GCM 儲存憑證。|Windows, macOS, Linux

#### Windows

```batch
SET GCM_CREDENTIAL_STORE="gpg"
```

#### macOS/Linux

```bash
export GCM_CREDENTIAL_STORE="gpg"
```

**另請參閱：[credential.credentialStore][credential-credentialstore]**

---

### GCM_CREDENTIAL_CACHE_OPTIONS

當 [`GCM_CREDENTIAL_STORE`][gcm-credential-store] 設定為 `cache` 時，將[選項][git-cache-options]傳遞給 Git 憑證快取。這允許您透過傳遞 `"--timeout <seconds>"` 來選擇不同的憑證快取時間（預設為 900 秒）。使用其他選項如 `--socket` 未經測試且不受支援，但沒有理由它不應該工作。

預設為空。

#### Windows

```batch
SET GCM_CREDENTIAL_CACHE_OPTIONS="--timeout 300"
```

#### macOS/Linux

```shell
export GCM_CREDENTIAL_CACHE_OPTIONS="--timeout 300"
```

**另請參閱：[credential.cacheOptions][credential-cacheoptions]**

---

### GCM_PLAINTEXT_STORE_PATH

當 [`GCM_CREDENTIAL_STORE`][gcm-credential-store] 設定為 `plaintext` 時，指定一個自訂目錄來儲存純文字憑證檔案。

預設值為 `~/.gcm/store` 或 `%USERPROFILE%\.gcm\store`。

#### Windows

```batch
SETX GCM_PLAINTEXT_STORE_PATH=D:\credentials
```

#### macOS/Linux

```shell
export GCM_PLAINTEXT_STORE_PATH=/mnt/external-drive/credentials
```

**另請參閱：[credential.plaintextStorePath][credential-plain-text-store]**

---

### GCM_DPAPI_STORE_PATH

當 [`GCM_CREDENTIAL_STORE`][gcm-credential-store] 設定為 `dpapi` 時，指定一個自訂目錄來儲存 DPAPI 保護的憑證檔案。

預設值為 `%USERPROFILE%\.gcm\dpapi_store`。

#### Windows

```batch
SETX GCM_DPAPI_STORE_PATH=D:\credentials
```

**另請參閱：[credential.dpapiStorePath][credential-dpapi-store-path]**

---

### GCM_GPG_PATH

指定 `pass` 使用的 `gpg` 版本的路徑（_包含_可執行檔名稱）（如果存在 `gpg2` 則為 `gpg2`，否則為 `gpg`）。這主要是為了允許手動解決在舊版 Linux 系統上並行安裝 `gpg` 和 `gpg2` 時發生的衝突。

如果未指定，GCM 預設使用 `$PATH` 上的 `gpg2` 版本，如果找不到 `gpg2`，則退回到 `gpg`。

#### macOS/Linux

```bash
export GCM_GPG_PATH="/usr/local/bin/gpg2"
```

_無對應的設定。_

---

### GCM_MSAUTH_FLOW

指定在執行 Microsoft 驗證且需要互動式流程時應使用的驗證流程。

預設為 `auto`。

**注意：** 如果 [`GCM_MSAUTH_USEBROKER`][gcm-msauth-usebroker] 設定為 `true` 且作業系統驗證代理程式可用，則所有流程都將委派給代理程式。如果這兩者都為真，則 `GCM_MSAUTH_FLOW` 的值沒有效果。

值|驗證流程
-|-
`auto` _(預設)_|根據當前環境和平台選擇最佳選項。
`embedded`|顯示帶有嵌入式網頁檢視控制項的視窗。
`system`|開啟使用者的預設網頁瀏覽器。
`devicecode`|顯示裝置代碼。

#### Windows

```batch
SET GCM_MSAUTH_FLOW="devicecode"
```

#### macOS/Linux

```bash
export GCM_MSAUTH_FLOW="devicecode"
```

**另請參閱：[credential.msauthFlow][credential-msauth-flow]**

---

### GCM_MSAUTH_USEBROKER _(實驗性)_

在可用時使用作業系統帳戶管理員。

預設為 `false`。在某些雲端託管環境中，當使用工作或學校帳戶時，例如 [Microsoft DevBox][devbox]，預設為 `true`。

這些預設值未來可能會變更。

_**注意：** 在 Windows 上啟用此選項之前，請[檢閱詳細資訊][windows-broker]，了解這對您的本地 Windows 使用者帳戶意味著什麼。_

值|描述
-|-
`true`|使用作業系統帳戶管理員作為驗證代理程式。
`false` _(預設)_|不使用代理程式。

#### Windows

```batch
SET GCM_MSAUTH_USEBROKER="true"
```

#### macOS/Linux

```bash
export GCM_MSAUTH_USEBROKER="false"
```

**另請參閱：[credential.msauthUseBroker][credential-msauth-usebroker]**

---

### GCM_MSAUTH_USEDEFAULTACCOUNT _(實驗性)_

當代理程式啟用時，預設使用目前的作業系統帳戶。

預設為 `false`。在某些雲端託管環境中，當使用工作或學校帳戶時，例如 [Microsoft DevBox][devbox]，預設為 `true`。

這些預設值未來可能會變更。

值|描述
-|-
`true`|預設使用目前的作業系統帳戶。
`false` _(預設)_|不預設使用任何帳戶。

#### Windows

```batch
SET GCM_MSAUTH_USEDEFAULTACCOUNT="true"
```

#### macOS/Linux

```bash
export GCM_MSAUTH_USEDEFAULTACCOUNT="false"
```

**另請參閱：[credential.msauthUseDefaultAccount][credential-msauth-usedefaultaccount]**

---

### GCM_AZREPOS_CREDENTIALTYPE

指定 Azure Repos 主機提供者應返回的憑證類型。

預設值為 `pat`。在某些雲端託管環境中，當使用工作或學校帳戶時，例如 [Microsoft DevBox][devbox]，預設值為 `oauth`。

值|描述
-|-
`pat`|Azure DevOps 個人存取權杖
`oauth`|Microsoft 身分識別 OAuth 權杖 (AAD 或 MSA 權杖)

關於 Azure 存取權杖的更多資訊可以在[這裡][azure-access-tokens]找到。

#### Windows

```batch
SET GCM_AZREPOS_CREDENTIALTYPE="oauth"
```

#### macOS/Linux

```bash
export GCM_AZREPOS_CREDENTIALTYPE="oauth"
```

**另請參閱：[credential.azreposCredentialType][credential-azrepos-credential-type]**

---

### GCM_AZREPOS_MANAGEDIDENTITY

使用[受控識別][managed-identity]向 Azure Repos 進行驗證。

值 `system` 將告訴 GCM 使用系統指派的受控識別。

若要指定使用者指派的受控識別，請使用格式 `id://{clientId}`，其中 `{clientId}` 是受控識別的用戶端 ID。或者，任何 GUID 格式的值也將被解釋為使用者指派的受控識別用戶端 ID。

若要指定與 Azure 資源相關聯的受控識別，您可以使用格式 `resource://{resourceId}`，其中 `{resourceId}` 是資源的 ID。

有關受控識別的更多資訊，請參閱 Azure DevOps [文件][azrepos-sp-mid]。

值|描述
-|-
`system`|系統指派的受控識別
`[guid]`|具有指定用戶端 ID 的使用者指派的受控識別
`id://[guid]`|具有指定用戶端 ID 的使用者指派的受控識別
`resource://[guid]`|與相關聯資源的使用者指派的受控識別

#### Windows

```batch
SET GCM_AZREPOS_MANAGEDIDENTITY="id://11111111-1111-1111-1111-111111111111"
```

#### macOS/Linux

```bash
export GCM_AZREPOS_MANAGEDIDENTITY="id://11111111-1111-1111-1111-111111111111"
```

**另請參閱：[credential.azreposManagedIdentity][credential-azrepos-managedidentity]**

---

### GCM_AZREPOS_SERVICE_PRINCIPAL

指定在為 Azure Repos 執行 Microsoft 驗證時要使用的[服務主體][service-principal]的用戶端和租用戶 ID。

此設定的值應為格式：`{tenantId}/{clientId}`。

如果您設定此值，還必須設定至少一種驗證機制：

- [GCM_AZREPOS_SP_SECRET][gcm-azrepos-sp-secret]
- [GCM_AZREPOS_SP_CERT_THUMBPRINT][gcm-azrepos-sp-cert-thumbprint]

有關服務主體的更多資訊，請參閱 Azure DevOps [文件][azrepos-sp-mid]。

#### Windows

```batch
SET GCM_AZREPOS_SERVICE_PRINCIPAL="11111111-1111-1111-1111-111111111111/22222222-2222-2222-2222-222222222222"
```

#### macOS/Linux

```bash
export GCM_AZREPOS_SERVICE_PRINCIPAL="11111111-1111-1111-1111-111111111111/22222222-2222-2222-2222-222222222222"
```

**另請參閱：[credential.azreposServicePrincipal][credential-azrepos-sp]**

---

### GCM_AZREPOS_SP_SECRET

在設定 [GCM_AZREPOS_SERVICE_PRINCIPAL][gcm-azrepos-sp] 的情況下，為 Azure Repos 執行 Microsoft 驗證時，指定[服務主體][service-principal]的用戶端密碼。

#### Windows

```batch
SET GCM_AZREPOS_SP_SECRET="da39a3ee5e6b4b0d3255bfef95601890afd80709"
```

#### macOS/Linux

```bash
export GCM_AZREPOS_SP_SECRET="da39a3ee5e6b4b0d3255bfef95601890afd80709"
```

**另請參閱：[credential.azreposServicePrincipalSecret][credential-azrepos-sp-secret]**

---

### GCM_AZREPOS_SP_CERT_THUMBPRINT

在設定 [GCM_AZREPOS_SERVICE_PRINCIPAL][gcm-azrepos-sp] 的情況下，為 Azure Repos 作為[服務主體][service-principal]進行驗證時，指定要使用的憑證指紋。

#### Windows

```batch
SET GCM_AZREPOS_SP_CERT_THUMBPRINT="9b6555292e4ea21cbc2ebd23e66e2f91ebbe92dc"
```

#### macOS/Linux

```bash
export GCM_AZREPOS_SP_CERT_THUMBPRINT="9b6555292e4ea21cbc2ebd23e66e2f91ebbe92dc"
```

**另請參閱：[credential.azreposServicePrincipalCertificateThumbprint][credential-azrepos-sp-cert-thumbprint]**

---

### GCM_AZREPOS_SP_CERT_SEND_X5C

當使用憑證進行服務主體驗證時，此設定指定是否應將 X5C 宣告傳送至 STS。傳送 x5c 使應用程式開發人員能夠在 Azure AD 中輕鬆實現憑證輪替：此方法會將公開憑證連同權杖請求一起傳送至 Azure AD，以便 Azure AD 可以根據受信任的簽發者政策使用它來驗證主體名稱。這省去了應用程式管理員明確管理憑證輪替的需要。詳細資訊請參閱 [https://aka.ms/msal-net-sni](https://aka.ms/msal-net-sni)。

#### Windows

```batch
SET GCM_AZREPOS_SP_CERT_SEND_X5C="true"
```

#### macOS/Linux

```bash
export GCM_AZREPOS_SP_CERT_SEND_X5C="true"
```

**另請參閱：[credential.azreposServicePrincipalCertificateSendX5C][credential-azrepos-sp-cert-x5c]**

---

### GIT_TRACE2

開啟 Trace2 Normal Format 追蹤 - 更多詳細資訊請參閱 [Git 的 Trace2 Normal Format 文件][trace2-normal-docs]。

#### Windows

```batch
SET GIT_TRACE2=%UserProfile%\log.normal
```

#### macOS/Linux

```bash
export GIT_TRACE2=~/log.normal
```

如果 `GIT_TRACE2` 的值是現有目錄中檔案的完整路徑，記錄將會附加到該檔案。

如果 `GIT_TRACE2` 的值是 `true` 或 `1`，記錄將會寫入標準錯誤。

預設為停用。

**另請參閱：[trace2.normalFormat][trace2-normal-config]**

---

### GIT_TRACE2_EVENT

開啟 Trace2 Event Format 追蹤 - 更多詳細資訊請參閱 [Git 的 Trace2 Event Format 文件][trace2-event-docs]。

#### Windows

```batch
SET GIT_TRACE2_EVENT=%UserProfile%\log.event
```

#### macOS/Linux

```bash
export GIT_TRACE2_EVENT=~/log.event
```

如果 `GIT_TRACE2_EVENT` 的值是現有目錄中檔案的完整路徑，記錄將會附加到該檔案。

如果 `GIT_TRACE2_EVENT` 的值是 `true` 或 `1`，記錄將會寫入標準錯誤。

預設為停用。

**另請參閱：[trace2.eventFormat][trace2-event-config]**

---

### GIT_TRACE2_PERF

開啟 Trace2 Performance Format 追蹤 - 更多詳細資訊請參閱 [Git 的 Trace2 Performance Format 文件][trace2-performance-docs]。

#### Windows

```batch
SET GIT_TRACE2_PERF=%UserProfile%\log.perf
```

#### macOS/Linux

```bash
export GIT_TRACE2_PERF=~/log.perf
```

如果 `GIT_TRACE2_PERF` 的值是現有目錄中檔案的完整路徑，記錄將會附加到該檔案。

如果 `GIT_TRACE2_PERF` 的值是 `true` 或 `1`，記錄將會寫入標準錯誤。

預設為停用。

**另請參閱：[trace2.perfFormat][trace2-performance-config]**

[autodetect]: autodetect.md
[azure-access-tokens]: azrepos-users-and-tokens.md
[configuration]: configuration.md
[credential-allowwindowsauth]: configuration.md#credentialallowwindowsauth
[credential-allowunsaferemotes]: configuration.md#credentialallowunsaferemotes
[credential-authority]: configuration.md#credentialauthority-deprecated
[credential-autodetecttimeout]: configuration.md#credentialautodetecttimeout
[credential-azrepos-credential-type]: configuration.md#credentialazreposcredentialtype
[credential-azrepos-managedidentity]: configuration.md#credentialazreposmanagedidentity
[credential-bitbucketauthmodes]: configuration.md#credentialbitbucketAuthModes
[credential-cacheoptions]: configuration.md#credentialcacheoptions
[credential-credentialstore]: configuration.md#credentialcredentialstore
[credential-debug]: configuration.md#credentialdebug
[credential-dpapi-store-path]: configuration.md#credentialdpapistorepath
[credential-githubaccountfiltering]: configuration.md#credentialgitHubAccountFiltering
[credential-githubauthmodes]: configuration.md#credentialgitHubAuthModes
[credential-gitlabauthmodes]: configuration.md#credentialgitLabAuthModes
[credential-guiprompt]: configuration.md#credentialguiprompt
[credential-guisoftwarerendering]: configuration.md#credentialguisoftwarerendering
[credential-httpproxy]: configuration.md#credentialhttpProxy-deprecated
[credential-interactive]: configuration.md#credentialinteractive
[credential-namespace]: configuration.md#credentialnamespace
[credential-msauth-flow]: configuration.md#credentialmsauthflow
[credential-msauth-usebroker]: configuration.md#credentialmsauthusebroker-experimental
[credential-msauth-usedefaultaccount]: configuration.md#credentialmsauthusedefaultaccount-experimental
[credential-plain-text-store]: configuration.md#credentialplaintextstorepath
[credential-provider]: configuration.md#credentialprovider
[credential-stores]: credstores.md
[credential-trace]: configuration.md#credentialtrace
[credential-trace-secrets]: configuration.md#credentialtracesecrets
[credential-trace-msauth]: configuration.md#credentialtracemsauth
[default-values]: enterprise-config.md
[devbox]: https://azure.microsoft.com/en-us/products/dev-box
[freedesktop-ss]: https://specifications.freedesktop.org/secret-service-spec/
[gcm]: usage.md
[gcm-interactive]: #gcm_interactive
[gcm-credential-store]: #gcm_credential_store
[gcm-dpapi-store-path]: #gcm_dpapi_store_path
[gcm-plaintext-store-path]: #gcm_plaintext_store_path
[gcm-msauth-usebroker]: #gcm_msauth_usebroker-experimental
[git-cache-options]: https://git-scm.com/docs/git-credential-cache#_options
[git-credential-cache]: https://git-scm.com/docs/git-credential-cache
[git-httpproxy]: https://git-scm.com/docs/git-config#Documentation/git-config.txt-httpproxy
[github-emu]: https://docs.github.com/en/enterprise-cloud@latest/admin/identity-and-access-management/using-enterprise-managed-users-for-iam/about-enterprise-managed-users
[network-http-proxy]: netconfig.md#http-proxy
[libsecret]: https://wiki.gnome.org/Projects/Libsecret
[managed-identity]: https://docs.microsoft.com/en-us/azure/active-directory/managed-identities-azure-resources/overview
[migration-guide]: migration.md#gcm_authority
[passwordstore]: https://www.passwordstore.org/
[trace2-normal-docs]: https://git-scm.com/docs/api-trace2#_the_normal_format_target
[trace2-normal-config]: configuration.md#trace2normalTarget
[trace2-event-docs]: https://git-scm.com/docs/api-trace2#_the_event_format_target
[trace2-event-config]: configuration.md#trace2eventTarget
[trace2-performance-docs]: https://git-scm.com/docs/api-trace2#_the_performance_format_target
[trace2-performance-config]: configuration.md#trace2perfTarget
[windows-broker]: windows-broker.md
[service-principal]: https://docs.microsoft.com/en-us/azure/active-directory/develop/app-objects-and-service-principals
[azrepos-sp-mid]: https://learn.microsoft.com/en-us/azure/devops/integrate/get-started/authentication/service-principal-managed-identity
[gcm-azrepos-sp]: #gcm_azrepos_service_principal
[gcm-azrepos-sp-secret]: #gcm_azrepos_sp_secret
[gcm-azrepos-sp-cert-thumbprint]: #gcm_azrepos_sp_cert_thumbprint
[gcm-azrepos-sp-cert-x5c]: #gcm_azrepos_sp_cert_send_x5c
[credential-azrepos-sp]: configuration.md#credentialazreposserviceprincipal
[credential-azrepos-sp-secret]: configuration.md#credentialazreposserviceprincipalsecret
[credential-azrepos-sp-cert-thumbprint]: configuration.md#credentialazreposserviceprincipalcertificatethumbprint
[credential-azrepos-sp-cert-x5c]: configuration.md#credentialazreposserviceprincipalcertificatesendx5c
