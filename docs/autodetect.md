# 主機供應商自動偵測

Git Credential Manager (GCM) 支援與多個不同的 Git 服務進行驗證，其主機供應商包括：GitHub、Bitbucket 和 Azure Repos。除了雲端託管服務外，GCM 也能與自行架設或「就地部署」的版本搭配運作：GitHub Enterprise Server、Bitbucket DC Server 以及 Azure DevOps Server (TFS)。

在預設情況下，GCM 會嘗試自動偵測由哪個特定供應商支援您正在互動的 Git 遠端 URL。對於雲端版本的受支援供應商，這是透過比對遠端 URL 的主機名稱與服務的已知主機名稱來達成。例如 "github.com" 或 "dev.azure.com"。

## 自行架設/就地部署偵測

為了偵測自行架設的執行個體應使用哪個主機供應商，每個供應商都可以提供一些主機名稱的啟發式比對。例如，任何以 "github.*" 開頭的主機名稱都會被比對為 GitHub 主機供應商。

如果啟發式比對不正確，您隨時可以[明確設定][explicit-config] GCM 使用特定的供應商。

## 遠端 URL 探測

除了啟發式比對之外，GCM 也會對遠端 URL 發出網路呼叫並檢查 HTTP 回應標頭，以嘗試偵測自行架設的執行個體。

此網路呼叫僅在精確或模糊比對皆無法透過主機名稱完成時才會執行。每個憑證請求只會發出一次 HTTP `HEAD` 呼叫（由 Git 接收）。為避免此網路呼叫，請
[明確設定][explicit-config] 您自行架設的執行個體的主機供應商。

成功偵測到主機供應商後，GCM 會自動設定該 [`credential.provider`][credential-provider] 組態項目給該遠端，以避免需要在未來的請求中執行此耗費資源的網路呼叫。

### 逾時

您可以控制 GCM 等待遠端網路呼叫回應的時間長度，方法是設定 [`GCM_AUTODETECT_TIMEOUT`][gcm-autodetect-timeout] 環境變數，或是 [`credential.autoDetectTimeout`][credential-autoDetectTimeout] Git 組態設定，將其值設為最長的等待毫秒數。

預設值為 2000 毫秒（2 秒）。您可以防止網路呼叫的執行，只要將值設為零或負數即可，例如 -1。

## 手動設定

如果自動偵測機制未能選取正確的主機供應商，或如果遠端探測的網路呼叫造成效能問題，您可以設定 GCM 針對給定的遠端 URL 永遠使用特定的主機供應商。

為此，您可以使用 [`GCM_PROVIDER`][gcm-provider] 環境變數，或是 [`credential.provider`][credential-provider] Git 組態設定來達成此目的。

例如，若要告知 GCM 針對 "ghe.example.com" 主機名稱一律使用 GitHub 主機供應商，您可以執行以下指令：

```shell
git config --global credential.ghe.example.com.provider github
```

[credential-autoDetectTimeout]: configuration.md#credentialautodetecttimeout
[credential-provider]: configuration.md#credentialprovider
[explicit-config]: #manual-configuration
[gcm-autodetect-timeout]: environment.md#GCM_AUTODETECT_TIMEOUT
[gcm-provider]: environment.md#GCM_PROVIDER
