# Azure 受控識別與服務主體

Git Credential Manager 支援使用受控識別與服務主體來驗證 Azure Repos。本文件概述了受控識別與服務主體，以及如何將其與 GCM 搭配使用。

## 受控識別

Azure 受控識別可用於驗證和授權應用程式與服務，以存取 Azure 資源。受控識別是一種安全的方式，無需在程式碼或組態檔中儲存憑證即可存取 Azure 資源。

受控識別有兩種類型：

**系統指派**

系統指派的受控識別與特定的 Azure 資源（例如虛擬機器或應用程式服務）綁定。啟用系統指派的受控識別後，Azure 會在受訂閱信任的 Azure AD 租用戶中為該資源建立一個身分。該身分的生命週期與其所指派的資源綁定。

**使用者指派**

使用者指派的受控識別是作為獨立的 Azure 資源建立，並且可以指派給一個或多個 Azure 資源。這允許您在多個資源之間使用相同的受控識別。

您可以在 [Azure 文件][az-mi] 中閱讀更多關於受控識別的資訊。

### 如何設定受控識別

為了將受控識別與 GCM 搭配使用，您需要確保該受控識別具有存取 Azure Repos 儲存庫所需的權限。

您可以在 [Azure Repos 文件][azdo-misp] 中閱讀更多關於如何設定受控識別的資訊。

一旦設定了受控識別，您只需設定下列其中一個環境變數或 Git 組態選項，即可將其與 GCM 搭配使用：

**Git 組態：** [`credential.azreposManagedIdentity`][gcm-mi-config]

**環境變數：** [`GCM_AZREPOS_MANAGEDIDENTITY`][gcm-mi-env]

值|說明
-|-
`system`|系統指派的受控識別
`[guid]`|具有指定用戶端 ID 的使用者指派受控識別
`id://[guid]` **|具有指定用戶端 ID 的使用者指派受控識別
`resource://[guid]` **|關聯資源的使用者指派受控識別

您可以從 Azure 入口網站取得 `[guid]`，或使用 Azure CLI 來檢查受控識別或資源。

** 請注意，有一個未解決的問題會導致使用這些格式時驗證失敗：https://github.com/git-ecosystem/git-credential-manager/issues/1570

## 服務主體

Azure 服務主體用於驗證和授權應用程式與服務以存取 Azure 資源。服務主體在許多方面與受控識別相似（事實上，服務主體在底層被用來實作受控識別），但它們有明確定義的憑證，這些憑證不由 Azure 管理。

有許多不同的方法可以建立和設定服務主體，包括使用 Azure 入口網站或 Azure CLI。您可以閱讀更多關於服務主體的資訊，請參閱 [Azure 文件][az-sp]。

### 如何設定服務主體

與受控識別非常類似，若要搭配 GCM 使用服務主體，您首先需要確保該主體具有存取 Azure Repos 存放庫的必要權限。

您可以閱讀更多關於如何設定服務主體的資訊，請參閱 [Azure Repos 文件][azdo-misp]。

一旦您設定好服務主體，便可以透過設定下列其中一個環境變數或 Git 組態選項來搭配 GCM 使用：

**Git 組態：** [`credential.azreposServicePrincipal`][gcm-sp-config]

**環境變數：** [`GCM_AZREPOS_SERVICE_PRINCIPAL`][gcm-sp-env]

這些選項值的格式必須如下：

```text
{tenantId}/{clientId}
```

其中 `{tenantId}` 是 Azure 租用戶 ID，而 `{clientId}` 是服務主體的用戶端 ID。這些值可以在 Azure 入口網站中找到，或透過使用Azure CLI 來檢查服務主體。

#### 使用服務主體驗證

當您搭配 GCM 使用服務主體時，您也需要提供與該服務主體相關聯的用戶端密碼或憑證。

您可以將用戶端密碼或憑證提供給 GCM，方法是設定下列其中一個環境變數或 Git 組態選項。

類型|Git 組態|環境變數
-|-|-
用戶端密碼|[`credential.azreposServicePrincipalSecret`][gcm-sp-secret-config]|[`GCM_AZREPOS_SP_SECRET`][gcm-sp-secret-env]
憑證|[`credential.azreposServicePrincipalCertificateThumbprint`][gcm-sp-cert-config]|[`GCM_AZREPOS_SP_CERT_THUMBPRINT`][gcm-sp-cert-env]
傳送 X5C|[`credential.azreposServicePrincipalCertificateSendX5C`][gcm-sp-cert-x5c-config]|[`GCM_AZREPOS_SP_CERT_SEND_X5C`][gcm-sp-cert-x5c-env]

這些選項的值應為用戶端密碼或與服務主體相關聯之憑證的指紋。

憑證本身應安裝在執行 GCM 的機器上並應安裝在個人憑證存放區，該存放區屬於目前使用者或本機電腦。

[az-mi]: https://learn.microsoft.com/en-us/entra/identity/managed-identities-azure-resources/overview
[az-sp]: https://learn.microsoft.com/en-us/entra/identity-platform/app-objects-and-service-principals?tabs=browser
[azdo-misp]: https://learn.microsoft.com/en-us/azure/devops/integrate/get-started/authentication/service-principal-managed-identity?view=azure-devops
[gcm-mi-config]: https://gh.io/gcm/config#credentialazreposmanagedidentity
[gcm-mi-env]: https://gh.io/gcm/env#GCM_AZREPOS_MANAGEDIDENTITY
[gcm-sp-config]: https://gh.io/gcm/config#credentialazreposserviceprincipal
[gcm-sp-env]: https://gh.io/gcm/env#GCM_AZREPOS_SERVICE_PRINCIPAL
[gcm-sp-secret-config]: https://gh.io/gcm/config#credentialazreposserviceprincipalsecret
[gcm-sp-secret-env]: https://gh.io/gcm/env#GCM_AZREPOS_SP_SECRET
[gcm-sp-cert-config]: https://gh.io/gcm/config#credentialazreposserviceprincipalcertificatethumbprint
[gcm-sp-cert-x5c-config]: https://gh.io/gcm/config#credentialazreposserviceprincipalcertificatesendx5c
[gcm-sp-cert-env]: https://gh.io/gcm/env#GCM_AZREPOS_SP_CERT_THUMBPRINT
[gcm-sp-cert-x5c-env]: https://gh.io/gcm/env#GCM_AZREPOS_SP_CERT_SEND_X5C
