# 網路與 HTTP 組態設定

Git Credential Manager 的網路與 HTTP(S) 行為可透過幾種不同的方式，經由[環境變數][environment]和[組態設定選項][configuration]來設定。

## HTTP 代理伺服器

如果您的電腦位於網路防火牆之後，而該防火牆要求使用代理伺服器來連線儲存庫遠端或更廣泛的網際網路，那麼有多種方法可以設定 GCM 來使用代理伺服器。

為 _所有_ HTTP(S) 遠端設定代理伺服器最簡單的方法是[使用標準的 Git HTTP(S) 代理伺服器設定 `http.proxy`][git-http-proxy]。

例如，為目前使用者替所有遠端設定代理伺服器：

```shell
git config --global http.proxy http://proxy.example.com
```

若要為特定遠端指定代理伺服器，您可以[使用 `remote.<name>.proxy` 儲存庫層級的設定][git-remote-name-proxy]，例如：

```shell
git config --local remote.origin.proxy http://proxy.example.com
```

使用這些標準組態設定選項的優點是，除了 GCM 會被設定使用代理伺服器之外，Git 本身也會同時被設定。對於位於網際網路攔截防火牆後的環境而言，這很可能是最常見的期望情況。

### 需身分驗證的代理伺服器

有些代理伺服器不接受匿名連線，並且需要身分驗證。為了指定要與代理伺服器一同使用的憑證，您可以將使用者名稱和密碼指定為代理伺服器 URL 設定的一部分。

其格式遵循 [RFC 3986 第 3.2.1 節][rfc-3986-321]，做法是將憑證包含在 URI 的「使用者資訊」部分。密碼為選填項目。

```text
protocol://username[:password]@hostname
```

例如，要指定使用者名稱 `john.doe` 和密碼 `letmein123` 給代理伺服器 `proxy.example.com`：

```text
https://john.doe:letmein123@proxy.example.com
```

如果您的使用者名稱或密碼中含有特殊字元（如 [RFC 3986 第 2.2 節][rfc-3986-22] 所定義），例如 `:`、`@` 或任何其他非 URL 友善的字元，您可以對它們進行 URL 編碼 （[第 2.1 節][rfc-3986-21]）。

例如，一個空格字元會被編碼為 `%20`。

### 其他代理伺服器選項

為了便利性與相容性，GCM 支援其他設定代理伺服器的方式。

1. GCM 特定的組態設定選項（_**僅** GCM 採用；**已棄用**_）：
   - `credential.httpProxy`
   - `credential.httpsProxy`
1. cURL 環境變數（_Git 也採用_）：
   - `http_proxy`
   - `https_proxy`/`HTTPS_PROXY`
   - `all_proxy`/`ALL_PROXY`
1. `GCM_HTTP_PROXY` 環境變數（_**僅** GCM 採用；**已棄用**_）

請注意，cURL 環境變數同時有小寫和大寫兩種版本。

**_小寫版本的優先級高於大寫版本。_** 這與 libcurl（以及 Git）的運作方式一致。

`http_proxy` 變數僅存在小寫版本，且 libcurl _不_會考慮任何大寫形式。_GCM 也反映了此行為。_

更多資訊請參閱 [curl 文件][curl-proxy-env-vars]。

### 繞道位址

在某些情況下，您可能會希望針對特定的位址繞過已設定的代理伺服器。GCM 支援 cURL 環境變數 `no_proxy` （以及 `NO_PROXY`）用於此情境，Git 本身也支援。

如同[其他 cURL 代理伺服器環境變數][other-proxy-options]，小寫版本的優先級將高於大寫版本。

此環境變數應包含一個以逗號或空格分隔的主機名稱列表，這些主機不應透過代理伺服器連線（應直接連線）。

GCM 試圖比對 [libcurl 的行為][curlopt-noproxy]，其行為簡要總結如下：

- 值為 `*` 會對所有主機停用代理；
- **不**支援使用其他萬用字元；
- 列表中的每個名稱會被比對為包含該主機名稱的網域，
  或主機名稱本身
- 開頭的句點/點 `.` 會與提供的主機名稱進行比對

例如，將 `NO_PROXY` 設定為 `example.com` 會產生以下結果：

主機名稱|是否匹配？
-|-
`example.com`|:white_check_mark:
`example.com:80`|:white_check_mark:
`www.example.com`|:white_check_mark:
`notanexample.com`|:x:
`www.notanexample.com`|:x:
`example.com.othertld`|:x:

**範例：**

```text
no_proxy="contoso.com,www.fabrikam.com"
```

## TLS 驗證

如果您使用自簽章 TLS (SSL) 憑證，並搭配自架的主機服務供應商（例如 GitHub Enterprise Server 或 Azure DevOps Server（先前稱為 TFS）），那麼在嘗試使用
Git 和/或 GCM 連線時，您可能會看到以下錯誤訊息：

```shell
$ git clone https://ghe.example.com/john.doe/myrepo
fatal: The remote certificate is invalid according to the validation procedure.
```

**建議且最安全的選項**是取得由公開受信任的憑證授權單位 (CA) 所簽署的 TLS 憑證。有多個公開的 CA；此處提供一個非詳盡的列表供您參考：[Let's Encrypt][lets-encrypt]、[Comodo][comodo]、[Digicert][digicert]、[GoDaddy][godaddy]、[GlobalSign][globalsign]。

如果無法**從受信任的第三方取得 TLS 憑證**，那麼您應該嘗試將*特定的*自簽章憑證或驗證鏈中的其中一個 CA 憑證新增至您作業系統的受信任憑證存放區中（[macOS][mac-keychain-access]、[Windows][install-cert-vista]）。

如果您*無法***取得受信任的憑證**，或信任該自簽章憑證，您可以停用 Git 和 GCM 中的憑證驗證。

---
**Security Warning** :warning:

停用 TLS (SSL) 憑證驗證會移除對[中間人攻擊 (MITM attack)][mitm-attack] 的防護。

只有在您確定有此需要、並了解所有風險，且無法信任特定的自簽章憑證（如上所述）時，才應停用憑證驗證。

---

[環境變數 `GIT_SSL_NO_VERIFY`][git-ssl-no-verify] 與 [Git 組態設定選項 `http.sslVerify`][git-http-ssl-verify] 可用來控制 TLS (SSL) 憑證驗證。

若要針對特定遠端停用驗證（例如 `https://example.com`）：

```shell
git config --global http.https://example.com.sslVerify false
```

若要為目前使用者停用 **_所有遠端_** 的驗證（**不建議**）：

```shell
# Environment variable (Windows)
SET GIT_SSL_NO_VERIFY=1

# Environment variable (macOS/Linux)
export GIT_SSL_NO_VERIFY=1

# Git configuration (Windows/macOS/Linux)
git config --global http.sslVerify false
```

---

**注意：** 如果您正在使用網路流量檢測工具，例如 [Telerik Fiddler][telerik-fiddler]，您可能也會遇到類似的驗證錯誤。如果您正在使用這類工具，請參閱其文件以信任代理伺服器根憑證。

---

## 不安全的遠端 URL

如果您使用的遠端 URL 被認為不安全，例如未加密的 HTTP（以 `http://` 開頭的遠端 URL），主機供應商可能會阻止您使用您的憑證進行驗證。

在這種情況下，您應該考慮使用 HTTPS（以 `https://` 開頭）的遠端 URL，以確保您的憑證安全傳輸。

如果您接受使用不安全遠端 URL 的相關風險，您可以設定 GCM 以允許使用不安全的遠端 URL，方法是設定環境變數 [`GCM_ALLOW_UNSAFE_REMOTES`][unsafe-envar]，或使用 Git
組態設定選項 [`credential.allowUnsafeRemotes`][unsafe-config] 為 `true`。

[environment]: environment.md
[configuration]: configuration.md
[git-http-proxy]: https://git-scm.com/docs/git-config#Documentation/git-config.txt-httpproxy
[git-remote-name-proxy]: https://git-scm.com/docs/git-config#Documentation/git-config.txt-remoteltnamegtproxy
[rfc-3986-321]: https://www.rfc-editor.org/rfc/rfc3986#section-3.2.1
[rfc-3986-22]: https://www.rfc-editor.org/rfc/rfc3986#section-2.2
[rfc-3986-21]: https://www.rfc-editor.org/rfc/rfc3986#section-2.1
[curl-proxy-env-vars]: https://everything.curl.dev/usingcurl/proxies#proxy-environment-variables
[other-proxy-options]: #other-proxy-options
[curlopt-noproxy]: https://curl.se/libcurl/c/CURLOPT_NOPROXY.html
[lets-encrypt]: https://letsencrypt.org/
[comodo]: https://www.comodoca.com/
[digicert]: https://www.digicert.com/
[godaddy]: https://www.godaddy.com/
[globalsign]: https://www.globalsign.com
[mac-keychain-access]: https://support.apple.com/en-gb/guide/keychain-access/kyca2431/mac
[install-cert-vista]: https://blogs.technet.microsoft.com/sbs/2008/05/08/installing-a-self-signed-certificate-as-a-trusted-root-ca-in-windows-vista/
[mitm-attack]: https://en.wikipedia.org/wiki/Man-in-the-middle_attack
[git-ssl-no-verify]: https://git-scm.com/book/en/v2/Git-Internals-Environment-Variables#_networking
[git-http-ssl-verify]: https://git-scm.com/docs/git-config#Documentation/git-config.txt-httpsslVerify
[telerik-fiddler]: https://www.telerik.com/fiddler
[unsafe-envar]: environment.md#gcm_allow_unsafe_remotes
[unsafe-config]: configuration.md#credentialallowunsaferemotes
