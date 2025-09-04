# 架構

## 概觀

```text
+------------------------------------------------------------------------------+
|                                                                              |
|                           Git-Credential-Manager                             |
|                                                                              |
+-+-------------+--------------+-----+---------------------+-----------------+-+
  |             |              |     |                     |                 |
  |             |              |     |             Windows |         Windows |
  |             |              |     |                     |                 |
  | +-----------v-----------+  |     |    +----------------v---------------+ |
  | |                       |  |     |    |                                | |
  | |        GitHub         <-------------+        GitHub.UI.Windows       | |
  | |                       |  |     |    |                                | |
  | +-+---------------------+  |     |    +-+------------------------------+ |
  |   |                        |     |      |                                |
  |   |  +---------------------v-+   |      | +------------------------------v-+
  |   |  |                       |   |      | |                                |
  |   |  |  Atlassian.Bitbucket  <------------+ Atlassian.Bitbucket.UI.Windows |
  |   |  |                       |   |      | |                                |
  |   |  +-+---------------------+   |      | +---------------+----------------+
  |   |    |                         |      |                 |
  |   |    |  +----------------------v-+    |                 |
  |   |    |  |                        |    |                 |
  |   |    |  |  Microsoft.AzureRepos  |    |                 |
  |   |    |  |                        |    |                 |
  |   |    |  +-----------+------------+    |                 |
  |   |    |              |                 |                 |
+-v---v----v--------------v------------+  +-v-----------------v----------------+
|                                      |  |                                    |
|                 Core                 <--+               Core.UI              |
|                                      |  |                                    |
+--------------------------------------+  +------------------------------------+
```

Git Credential Manager (GCM) 的建構目標是與 Git 主機和平台/作業系統無關。大部分的共享邏輯 (命令執行、抽象平台子系統等) 皆可在 `Core` 類別函式庫 (C#) 中找到。該函式庫的目標為 .NET Standard 以及 .NET Framework。

> **注意**
>
> 之所以也直接針對 .NET Framework，是因為 `Microsoft.Identity.Client` ([MSAL.NET][msal]) 函式庫需要 .NET Framework 目標，才能顯示內嵌的網頁
> 瀏覽器驗證彈出視窗。
>
> MSAL.NET 中現已存在擴充點，這意味著我們可以插入我們自己的瀏覽器彈出視窗處理程式碼，這在 .NET 上意味著 Windows 和 Mac 都可以。我們還沒有時間去探索這個部分。
>
> 更多資訊請參閱 [GCM issue 113][issue-113]。

GCM 的進入點可以在 `Git-Credential-Manager` 專案中找到，它是一個同時針對 .NET 和 .NET Framework 的主控台應用程式。此專案會產生 `git-credential-manager(.exe)` 可執行檔，並且包含極少的程式碼——註冊所有支援的主機供應商以及執行 `Core` 中的 `Application` 物件。

供應商有自己的專案/組件，它們依賴於 `Core` 核心組件，並且是主要進入點應用程式 `Git-Credential-Manager` 的相依項。這些二進位檔案中的程式碼預期在所有支援的平台上執行，並且通常（請參閱 MSAL.NET 的說明）不包含任何圖形使用者介面；它們使用終端機提示而已。

當供應商需要某些平台特定的互動或圖形使用者介面時，建議的模型是擁有一個獨立的「輔助」可執行檔，由共用的核心二進位檔呼叫。目前 Bitbucket 和 GitHub
供應商各自擁有一個 WPF (僅限 Windows) 輔助可執行檔，用以顯示驗證提示和訊息。

`Core.UI` 專案是一個 WPF (僅限 Windows) 組件其中包含在供應商之間共用的通用 WPF 元件和樣式 Windows 上的輔助程式。

### 跨平台 UI

我們希望能將僅限 WPF/Windows 的輔助程式遷移到 [Avalonia][avalonia] 以獲得跨平台的圖形使用者介面支援。請參閱 [GCM issue 136][issue-136] 以了解此項工作的最新進度。

### Microsoft 驗證

對於使用 Microsoft 帳戶或 Azure Active Directory 的驗證，情況有些不同。`MicrosoftAuthentication` 元件存在於 `Core` 核心組件中，而非與特定主機供應商捆綁。這樣做是為了允許任何未來可能希望與 Microsoft 帳戶或 Azure Active Directory 整合的服務可以使用此可重複使用的驗證元件。

## 非同步程式設計

GCM 在幾乎所有適當的程式碼庫部分都利用了 .NET 和 C# 的 `async`/`await` 模型，因為請求通常最終都會在某個時間點連上網路。

## 命令執行

```text
                             +---------------+
                             |               |
                             |      Git      |
                             |               |
                             +---+-------^---+
                                 |       |
                             +---v---+---+---+
                             | stdin | stdout|
                             +---+---+---^---+
                                 |       |
                            (2)  |       |  (7)
                          Select |       | Serialize
                         Command |       | Result
                                 |       |
                     (3)         |       |
                    Select       |       |
+---------------+  Provider  +---v-------+---+
| Host Provider |            |               |
|   Registry    <------------+    Command    |
|               |            |               |
+-------^-------+            +----+------^---+
        |                         |      |
        |                   (4)   |      |   (6)
        |                Execute  |      |  Return
        |              Operation  |      |  Result
        |    (1)                  |      |
        |  Register          +----v------+---+
        |                    |               |
        +--------------------+ Host Provider |
                             |               |
                             +-------^-------+
                                     |
                   (5) Use services  |
                                     |
                             +-------v-------+
                             |    Command    |
                             |    Context    |
                             +---------------+
```

Git Credential Manager 維護一組已知的命令，包括 `Get|Store|EraseCommand`，以及用於安裝和說明/用法的命令。

GCM 也維護一組已知且已註冊的主機供應商，這些供應商實作了 `IHostProvider` 介面。供應商透過新增一個供應商的執行個體至 `Application` 物件來完成自我註冊，這是透過 `RegisterProvider` 方法，該方法位於 [`Core.Program`][core-program] 中。最後註冊 `GenericHostProvider`，如此它便能處理所有其他以 HTTP 為基礎的遠端作為通用處理機制，並提供基本的使用者名稱/密碼驗證，以及偵測 Windows 整合式驗證 (Kerberos, NTLM, Negotiate) 支援 (1)。

對於 GCM 的每次呼叫，命令列上的第一個參數會與已知的命令進行比對，如果比對成功，則來自Git 的輸入 (透過標準輸入) 會被反序列化，且該命令會被執行 (2)。

`Get|Store|EraseCommand` 會查詢主機供應商註冊表以尋找最合適的主機供應商。預設的註冊表實作會選取一個主機供應商，其方法是依序詢問每個已註冊的供應商是否了解該請求。供應商的選取可由使用者透過 [`credential.provider`][credential-provider] 或 [`GCM_PROVIDER`][gcm-provider] 組態與環境變數來分別覆寫（3）。

`Get|Store|EraseCommand` 會呼叫對應的 `Get|Store|EraseCredentialAsync` 方法於 `IHostProvider` 上，並傳入來自 Git 的請求以及 `ICommandContext` 的一個執行個體（4）。主機供應商接著便能利用命令上下文中可用的各種服務來完成所請求的操作（5）。

一旦憑證被建立、擷取、儲存或清除後，主機供應商會將憑證（僅限於 `get` 操作）回傳給呼叫的命令（6）。該憑證接著會被序列化並透過標準輸出回傳給 Git（7），然後 GCM 會以成功的結束碼終止。

## 主機供應商

主機供應商實作 `IHostProvider` 介面。它們可以選擇直接實作該介面，也可以繼承自 `HostProvider` 抽象類別（其本身也實作了 `IHostProvider` 介面）。

`HostProvider` 抽象類別實作了 `Get|Store|EraseCredentialAsync` 方法，並改為提供 `GenerateCredentialAsync` 抽象方法，以及 `GetServiceName` 虛擬方法。對 `get`、`store` 或 `erase` 的呼叫會先觸發對 `GetServiceName` 的呼叫，它應為供應商和請求回傳一個穩定且唯一的值。此值構成任何已儲存的憑證在憑證存放區中關聯屬性的一部分。在 `get` 操作期間，會向憑證存放區查詢具有此服務名稱的現有憑證。如果找到憑證，便會立即回傳。同樣地，對 `store` 的呼叫和 `erase` 會被自動處理，以根據服務名稱儲存憑證，並清除符合該服務名稱的憑證。這些方法被實作為 `virtual`，這表示您隨時可以覆寫此行為，例如在 `erase` 請求時清除其他自訂快取，而無須重新實作查閱/儲存憑證的邏輯。

`GetServiceName` 的預設實作通常對大多數供應商而言已經足夠。它會從 Git 的輸入參數中回傳計算後的遠端 URL（不含結尾斜線）— `<protocol>://<host>[/<path>]` — 即使有使用者名稱也
不會包含在內。

主機供應商會依優先順序 (接著是註冊順序) 透過 `IHostProvider.IsSupported(InputArguments)` 方法進行查詢，並傳遞收到的輸入從 Git 收到的輸入。如果供應商辨識出該請求，例如透過一個比對已知的主機名稱，他們可以回傳 `true`。如果供應商想要取消並中止驗證請求，例如，如果這是一個 HTTP (非HTTPS) 的已知主機請求，他們仍應回傳 `true` 並稍後取消該請求。

主機供應商也可以透過 `IHostProvider.IsSupported(HttpResponseMessage)` 方法進行查詢，並傳遞對遠端 URI 進行 HEAD 呼叫所得到的回應訊息。這對於根據標頭值偵測地端執行個體很有用。GCM 只有在相同註冊優先順序的其他供應商都未對 `InputArguments` 多載回傳 `true` 的情況下，才會透過此方法多載查詢供應商。

根據來自 Git 的請求，將會呼叫 `GetCredentialAsync` (用於 `get` 請求)、`StoreCredentialAsync` (用於 `store` 請求) 或`EraseCredentialAsync` (用於 `erase` 請求) 的其中之一。參數`InputArguments` 包含透過標準輸入傳遞的請求資訊來自 Git/呼叫端；此資訊與傳遞給 `IsSupported` 的相同。

`get` 操作的傳回值必須是一個 `ICredential`，讓 Git 可以用來完成驗證。

> **注意：**
>
> 該憑證也可以是一個執行個體，其中使用者名稱和密碼皆為空字串，以向 Git 發出信號，使其應讓 cURL 使用「任何驗證」偵測 — 通常是為了使用 Windows 整合式驗證。

`store` 和 `erase` 操作沒有傳回值，因為 Git 會忽略這些指令的任何輸出或結束代碼。這些操作的失敗最好透過寫入標準錯誤串流來傳達，可透過 `ICommandContext.Streams.Error`。

## 指令上下文

`ICommandContext` 包含許多服務，可用於與各種平台子系統互動，例如檔案系統或環境變數。指令上下文中的所有服務都以介面的形式公開，以便於在不同作業系統和平台之間進行測試和移植。

元件|說明
-|-
CredentialStore|一個由作業系統控制的安全位置，用於儲存和擷取 `ICredential` 物件。
Settings|對所有 GCM 設定的抽象層。
Streams|對連線至父處理程序 (通常是 Git) 的標準輸入、輸出和錯誤串流的抽象層。
Terminal|提供與附加的終端機 (如果存在) 的互動。
SessionManager|提供有關目前使用者工作階段的資訊。
Trace|提供追蹤資訊，可用於偵錯實際環境中的問題。機密資訊「必須」被完全過濾掉，或透過 `Write___Secret` 方法過濾。
FileSystem|對檔案系統操作的抽象層。
HttpClientFactory|用於建立 `HttpClient` 執行個體的工廠，這些執行個體已設定正確的使用者代理程式、標頭和代理伺服器設定。
Git|提供與 Git 和 Git 組態的互動。
Environment|對目前系統/使用者環境變數的抽象層。
SystemPrompts|提供用於顯示系統/作業系統原生憑證提示的服務。

## 錯誤處理與追蹤

GCM 對於無法復原的錯誤採用「快速失敗」(fail fast) 的方法。這通常意味著拋出一個 `Exception`，它將向上傳播到進入點並被攔截，然後回傳一個非零的結束碼，並印出帶有「fatal:」前綴的錯誤訊息。對於源自 interop/原生程式碼的錯誤，您應該拋出 `InteropException` 型別的例外。例外中的錯誤訊息應為人類可讀。當存在已知或使用者可修復的問題時，應提供如何自行解決問題的說明，或相關文件的連結。

警告可以透過標準錯誤串流發出(`ICommandContext.Streams.Error`)，當您想提醒使用者注意其組態中的潛在問題，而該問題不一定會停止操作/驗證時。

`ITrace` 元件可以在 `ICommandContext` 物件上找到，或直接傳入某些建構函式。詳細和診斷資訊會被寫入 GCM 中大多數地方的追蹤物件。

[avalonia]: https://avaloniaui.net/
[core-program]: ../src/shared/Git-Credential-Manager/Program.cs
[credential-provider]: configuration.md#credentialprovider
[issue-113]: https://github.com/git-ecosystem/git-credential-manager/issues/113
[issue-136]: https://github.com/git-ecosystem/git-credential-manager/issues/136
[gcm-provider]: environment.md#GCM_PROVIDER
[msal]: https://github.com/AzureAD/microsoft-authentication-library-for-dotnet
