# Web Account Manager 整合

Git Credential Manager (GCM) 知道如何與 Windows 的 [Web Account Manager (WAM)][azure-refresh-token-terms] 功能整合。GCM 使用 WAM 來儲存 Azure DevOps 的憑證。驗證請求會被「代理」至作業系統。目前，GCM 會與其他一些 Microsoft 開發人員工具 (如 Visual Studio 和 Azure CLI) 共享驗證狀態，這意味著會減少驗證提示。啟用 WAM 整合也可能是某些[條件式存取政策][azure-conditional-access] 的要求，企業會使用這些政策來協助保護其資產，包括原始程式碼。

與 WAM 代理的整合提供了便利性和其他好處，但也可能對您的裝置造成意想不到的其他變更。在由您的機構或雇主擁有和管理的裝置上，WAM 可能是正確的選擇。在個人裝置或由不同機構擁有的裝置上（例如，如果您是為 A 公司工作的承包商並有權存取 B 公司的資源），在啟用 WAM 整合之前，您應該注意一些令人意外的行為。

請注意，這只會影響 [Azure DevOps][azure-devops]。它不會影響與 GitHub、Bitbucket 或任何其他 Git 主機的驗證。

## 如何啟用

您可以透過設定環境變數 [`GCM_MSAUTH_USEBROKER`][GCM_MSAUTH_USEBROKER] 或設定 Git 組態值 [`credential.msauthUseBroker`][credential.msauthUseBroker] 來選擇啟用 WAM 支援。

## 功能

當您開啟 WAM 支援時，GCM 可以與 Windows 以及您電腦上其他啟用 WAM 的軟體協同運作。這意味著更無縫的體驗、更少的多重要素驗證提示，以及使用智慧卡和 Windows Hello 等額外驗證技術的能力。這些便利性和安全性功能是啟用 WAM 的充分理由。

## 預設使用目前作業系統帳戶

啟用 WAM 目前並不會自動使用目前的 Windows 帳戶進行驗證。為了選擇加入此行為，您可以設定 [`GCM_MSAUTH_USEDEFAULTACCOUNT`][GCM_MSAUTH_USEDEFAULTACCOUNT] 環境變數，或將 [`credential.msauthUseDefaultAccount`][credential.msauthUseDefaultAccount] Git 組態值設定為 `true`。

在某些雲端託管環境中，當使用公司或學校帳戶時，例如 [Microsoft Dev Box][devbox]，此設定會 **_自動啟用_**。

若要停用此行為，請將環境變數 [`GCM_MSAUTH_USEDEFAULTACCOUNT`][GCM_MSAUTH_USEDEFAULTACCOUNT] 或 [`credential.msauthUseDefaultAccount`][credential.msauthUseDefaultAccount] Git 組態值明確設定為 `false`。

## 令人意外的行為

WAM 和 Windows 身分識別系統相當複雜，旨在解決範圍非常廣泛的客戶使用案例。適用於單一家庭用戶的解決方案，可能不適用於企業管理的 10 萬台裝置，反之亦然。GCM 團隊不
對 WAM 的使用者體驗或所做的選擇負責，但透過與WAM 整合，我們也承襲了其中一些選擇。因此，我們希望您能了解一些預設設定和體驗，如果您選擇使用 WAM 整合。

### 公司或學校帳戶 (由 Azure AD 支援的身分識別)

當您登入由 Azure AD 支援的 Azure DevOps 組織（通常是您的公司或學校電子郵件）時，如果您的電腦已加入與該 Azure DevOps 組織相符的 Azure AD，您將獲得無縫且易於使用的體驗。

如果您的電腦未加入 Azure AD，或已加入不同的租用戶，WAM 將會顯示一個對話方塊，建議您保持登入狀態並允許組織管理您的裝置。該對話方塊在不同版本的 Windows 中略有變化；以下是 2021 年的兩個範例：

![21H1 之前的同意對話方塊][aad-questions]

![21H1 之後的同意對話方塊][aad-questions-21h1]

根據您的點擊，可能會發生以下三種情況之一：

- 如果您勾選「允許我的組織管理我的裝置」並點擊「確定」，您的電腦將會註冊到支援該組織的 Azure AD 租用戶。它也可能被納入 MDM（「行動裝置管理」——例如 Intune、AirWatch、MobileIron 等）管理，這意味著系統管理員可以將政策部署到您的電腦上：要求特定類型的登入、開啟防毒和防火牆軟體，以及啟用 BitLocker。您的身分識別也將可用於電腦上的其他應用程式進行登入，其中一些應用程式可能會自動登入。

![推送到已註冊 Intune 的裝置上的政策範例][aad-bitlocker]

- 如果您取消勾選「允許我的組織管理我的裝置」並點擊「確定」，您的電腦將會註冊到 Azure AD，但不會被納入 MDM 管理。您的身分識別將可用於電腦上的其他應用程式進行登入。其他應用程式可能會自動將您登入，或再次提示您允許您的組織管理您的裝置。儘管加入了 Azure AD，您組織的「條件式存取」政策仍可能阻止您存取 Azure DevOps。如果是這樣，系統將提示您如何註冊 MDM 的說明。

- 如果您改為點擊「否，僅登入此應用程式」，您的電腦將不會加入 Azure AD 或被納入 MDM 管理，因此無法強制執行任何政策，您的身分識別也不會提供給電腦上的其他應用程式。與上述情況類似，您組織的「條件式存取」政策可能會阻止您繼續操作。

如果需要「條件式存取」才能存取您組織的 Git 儲存庫，您可以 [啟用 WAM 整合][GCM_MSAUTH_USEBROKER]（或遵循您組織提供的其他說明）。

#### 移除裝置管理

如果您已允許電腦被管理並想要復原此操作，您可以進入 **設定** > **帳戶** > **存取公司或學校資源**。在您看到您的電子郵件地址和組織名稱的區塊中，點擊**中斷連線**。

![尋找您的公司或學校帳戶][aad-work-school]

![從 Azure AD 中斷連線][aad-disconnect]

### Microsoft 帳戶

當您登入由 Microsoft 帳戶 (MSA) 身分識別支援的 Azure DevOps 組織（例如 `@outlook.com` 或 `@gmail.com` 這類電子郵件地址）時，系統可能會提示您選取現有的「公司或學校帳戶」或使用不同的帳戶。

為了使用 MSA 登入，您應該繼續並選取「使用不同的[公司或學校] 帳戶」，但在出現提示時輸入您的 MSA 憑證。這是由於我們無法控制的設定所致。我們預期此體驗會隨著時間改善，未來將會出現「個人帳戶」選項。

![選擇現有或不同帳戶的初始對話方塊][ms-sign-in]

如果您已將 MSA 連接到 Windows 或登入其他 Microsoft 應用程式（例如 Office），那麼在使用 GCM 時，您可能會在驗證提示中看到此帳戶列出。

---

⚠️ **重要事項** ⚠️

在 Windows 中新增 MSA 時，系統会詢問您是要在所有裝置上使用此帳戶（**選項 1**），還是僅允許 Microsoft 應用程式存取您的身分識別（**選項 2**）。如果您選擇在所有地方使用該帳戶，那麼您的本機 Windows 使用者帳戶將會連接到該 MSA。這意味著您未來需要使用您的 MSA 憑證來登入 Windows。

選擇「僅此應用程式」或「僅限 Microsoft 應用程式」仍然允許您在 Windows 的各個應用程式中使用此 MSA，但不會要求您使用您的 MSA 憑證來登入 Windows。

![確認將您的 MSA 連接到 Windows][msa-confirm]

若要中斷使用選項 1 新增的 MSA，您可以進入 **設定** > **帳戶** > **您的資訊**，然後點擊 **停止自動登入所有 Microsoft 應用程式**。

![從 Windows 移除您的 Microsoft 帳戶][msa-remove]

對於「僅限 Microsoft 應用程式」新增的 MSA，您可以修改這些帳戶是否可用於其他應用程式，也可以從 **設定** > **帳戶** > **電子郵件與帳戶** 中移除帳戶：

![允許所有 Microsoft 應用程式存取您的身分識別][all-ms-apps]

![Microsoft 應用程式必須請求才能存取您的身分識別][apps-must-ask]

## 以管理員身分執行

### GCM 2.1 及更新版本

從 2.1 版開始，GCM 使用的 [Microsoft Authentication Library (MSAL)][msal-dotnet] 版本支援從提升權限的處理序中使用 Windows 代理。

### 先前版本

Windows 代理人（「WAM」）大量使用 [COM][ms-com]，這是一種遠端程序呼叫（RPC）技術，內建於 Windows。為了與 WAM 整合，Git Credential Manager 和底層的 [Microsoft Authentication Library (MSAL)][msal-dotnet] 必須使用 COM 介面和 RPC。當您以提升權限的處理程序執行 Git Credential Manager 時，部分的 GCM 和 WAM 之間的呼叫可能會因處理程序安全性不同而失敗層級。當您從系統管理員命令提示字元執行 `git` 時，或從以系統管理員身分執行的 Visual Studio 執行 Git 操作時，就可能發生這種情況。

如果您已啟用使用代理人，GCM 將會檢查其是否在一個提升權限的處理程序中執行。如果是，GCM 將自動嘗試修改 COM 執行中處理程序的安全性設定，以便 GCM 和 WAM 可以協同運作。然而，這個自動的處理程序安全性變更不保證會成功。諸如登錄檔或全系統 COM 設定等各種外部因素可能會導致其失敗。如果 GCM 無法修改處理程序的 COM 安全性設定，GCM 會印出一個警告訊息，並且將無法使用代理人。

```text
warning: broker initialization failed
Failed to set COM process security to allow Windows broker from an elevated process (0x80010119).
See https://aka.ms/gcm/wamadmin for more information.
```

### 可能的解決方案

為了解決這個問題，有以下幾個選項：

1. 更新至 [最新的 Git for Windows][git-for-windows-latest] **(建議)**。
2. 從非提升權限的處理程序執行 Git 或 Git Credential Manager。
3. 停用代理人，方法是設定 [`GCM_MSAUTH_USEBROKER`][GCM_MSAUTH_USEBROKER] 環境變數或 [`credential.msauthUseBroker`][credential.msauthUseBroker] Git 組態設定為 `false`。

[azure-refresh-token-terms]: https://docs.microsoft.com/azure/active-directory/devices/concept-primary-refresh-token#key-terminology-and-components
[azure-conditional-access]: https://docs.microsoft.com/azure/active-directory/conditional-access/overview
[azure-devops]: https://azure.microsoft.com/en-us/products/devops
[GCM_MSAUTH_USEBROKER]: environment.md#GCM_MSAUTH_USEBROKER-experimental
[GCM_MSAUTH_USEDEFAULTACCOUNT]: environment.md#GCM_MSAUTH_USEDEFAULTACCOUNT-experimental
[credential.msauthUseBroker]: configuration.md#credentialmsauthusebroker-experimental
[credential.msauthUseDefaultAccount]: configuration.md#credentialmsauthusedefaultaccount-experimental
[aad-questions]: img/aad-questions.png
[aad-questions-21h1]: img/aad-questions-21H1.png
[aad-bitlocker]: img/aad-bitlocker.png
[aad-work-school]: img/aad-work-school.png
[aad-disconnect]: img/aad-disconnect.png
[ms-sign-in]: img/get-signed-in.png
[all-ms-apps]: img/all-microsoft.png
[apps-must-ask]: img/apps-must-ask.png
[ms-com]: https://docs.microsoft.com/en-us/windows/win32/com/the-component-object-model
[msa-confirm]: img/msa-confirm.png
[msa-remove]: img/msa-remove.png
[msal-dotnet]: https://aka.ms/msal-net
[devbox]: https://azure.microsoft.com/en-us/products/dev-box
[git-for-windows-latest]: https://git-scm.com/download/win
