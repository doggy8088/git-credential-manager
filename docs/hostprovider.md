# Git Credential Manager 主機提供者

## 摘要

Git Credential Manager 是一個跨平台、跨主機的 Git 憑證輔助工具，可以透過實作並註冊一個「主機提供者」(host provider)，來擴充其功能以支援任何 Git 託管服務，從而實現對受保護 Git 儲存庫的無縫認證。

## 1. 簡介

Git Credential Manager (GCM) 是一個與主機和平台無關的 Git 憑證輔助工具應用程式。要為 GCM 新增對任何 Git 託管服務的認證支援，可以透過建立一個自訂的「主機提供者」並在產品中註冊它來達成。主機提供者可以透過在 [GitHub 上的 Git Credential Manager 儲存庫][gcm] 提交 pull request 來貢獻。

本文件概述了主機提供者的必要與預期行為，以及實作與註冊一個主機提供者所需滿足的條件。

### 1.1. 標記慣例

本規格文件中的關鍵字「MUST」、「MUST NOT」、「REQUIRED」、「SHALL」、「SHALL NOT」、「SHOULD」、「SHOULD NOT」、「RECOMMENDED」、「MAY」和「OPTIONAL」應根據 [[RFC2119][rfc-2119]] 中的描述進行解釋。

### 1.2. 縮寫

在整份文件中，您可能會看到產品名稱和安全性或憑證物件的多種縮寫。

「Git Credential Manager」縮寫為「GCM」。「Git Credential Manager for Windows」縮寫為「GCM for Windows」或「GCM Windows」。「Git Credential Manager for Mac & Linux」縮寫為「GCM for Mac/Linux」或「GCM Mac/Linux」。

OAuth2 [[RFC6749][rfc-6749]] 的「access tokens」縮寫為「ATs」，「refresh tokens」縮寫為「RTs」。「Personal Access Tokens」縮寫為「PATs」。

## 2. 實作

要為 GCM 編寫並新增一個主機提供者，需要兩個主要步驟：實作 `IHostProvider` 介面，以及透過主機提供者註冊表向應用程式註冊該提供者的一個實例。

主機提供者 MUST 實作 `IHostProvider` 介面。它們可以選擇直接實作此介面，也 MAY 繼承自 `HostProvider` 抽象類別（此類別本身也實作了 `IHostProvider` 介面）——請參閱 [2.6][hostprovider-base-class]。

實作者 MUST 實作所有介面屬性與抽象方法。

`Id` 和 `Name` 屬性 MUST 被實作，且 MUST NOT 回傳預設值或空值。

`Id` 欄位在所有提供者的集合中 MUST 是唯一的，否則在註冊時將會拋出錯誤。`Id` 欄位 MAY 是一個由字元和數字組成的唯一隨機字串，例如 UUID，但 RECOMMENDED 使用一個由 \[a-z\] 範圍內的字母字元組成且人類可讀的值。

`Name` 屬性 MUST 是一個使用者可讀的字串，並且 MUST 指明此提供者所支援的 Git 託管服務。

`SupportedAuthorityIds` 屬性 MUST 回傳一個物件的實例，而 NOT `null` 參考。在此集合中填入值是 OPTIONAL 的，但高度 RECOMMENDED。您應該回傳一個集合，其中包含此提供者支援認證的所有授權單位的穩定識別碼。

### 2.1. 註冊

主機提供者必須將其 `IHostProvider` 型別的實例提供給 GCM 應用程式的主機提供者註冊表，才能被納入處理請求的考量。

主要的 GCM `Application` 物件有一個主註冊表，您可以透過呼叫 `RegisterProvider` 方法來註冊提供者。

#### 2.1.2. 順序

GCM 中預設的主機提供者註冊表有多個優先級別，主機提供者可以在這些級別註冊：高 (High)、正常 (Normal) 和低 (Low)。

對於每個優先級別（從高到正常再到低），註冊表將按照註冊順序呼叫每個主機提供者，除非使用者覆寫了提供者選擇流程。

主機提供者的順序沒有規則或限制，但 `GenericHostProvider` MUST 最後註冊且優先級為低。通用提供者是一個包羅萬象的提供者實作，它會以標準方式處理任何請求。

### 2.2. 處理請求

當一個 `get`、`store` 或 `erase` 請求被呼叫時，`IsSupported(InputArguments)` 方法將會依序在所有已註冊的主機提供者上被呼叫。第一個回傳 `true` 的主機提供者將被指派處理該特定請求。如果使用者覆寫了主機提供者的選擇流程，則可能會改為選定一個特定的主機提供者，且 `IsSupported(InputArguments)` 方法將 NOT 被呼叫。

此方法 MUST 回傳 `true` 的時機，若且唯若該提供者理解該請求且能夠服務或處理該請求。如果提供者不知道如何處理請求，則 MUST 回傳 `false`。

如果在每個主機提供者優先級別中，都沒有主機提供者對 `IsSupported(InputArguments)` 方法的呼叫回傳 `true`，那麼將會向遠端 URL 發送一個 HTTP HEAD 請求，並透過 `IsSupported(HttpResponseMessage)` 方法呼叫每個主機提供者。主機提供者 SHOULD 利用此呼叫來檢查可識別的本地部署實例（例如，透過檢查回應標頭），如果它希望被指派處理憑證請求，則回傳 `true`，否則 MUST 回傳 `false`。

主機提供者 SHOULD 盡可能避免在任何 `IsSupported` 方法的多載中進行額外的網路呼叫，以避免降低整體應用程式的效能。

#### 2.2.1. 拒絕請求

如果主機提供者希望根據目前的上下文或輸入取消認證操作，`IsSupported` 方法 MUST 回傳 `true`。例如，當提供者要求安全協定，但支援的主機名稱所請求的協定是 `http` 而非 `https` 時。

主機提供者 MUST 改為從 `GetCredentialAsync` 方法中透過拋出 `Exception` 來取消請求。實作者 MUST 提供關於為何無法繼續認證的詳細資訊，例如「HTTP 不安全，請使用 HTTPS」。

### 2.3. 取得憑證

當發出 `get` 請求時，將會呼叫 `GetCredentialAsync` 方法。該方法 MUST 回傳一個能夠滿足特定存取請求的 `ICredential` 實例。傳遞給 `GetCredentialAsync` 的引數包含屬性，指明此請求所需的 `protocol` 和 `host`。`username` 和 `path` 屬性是 OPTIONAL 的，但如果它們存在，則 MUST 將其納入考量並用於引導認證過程。

在建立新憑證之前，主機提供者 MAY 嘗試尋找任何由 `StoreCredentialAsync` 方法儲存的既有憑證。

主機提供者 MAY 透過檢查與值相關聯的任何已儲存的 Metadata 來確認儲存的憑證是否仍然有效。主機提供者也 MAY 選擇透過發送網頁請求來進一步驗證取出的已儲存憑證。然而，NOT RECOMMENDED 發送任何已知速度緩慢或通常會產生不確定驗證結果的請求。

如果提供者選擇發送驗證用的網頁請求，而該請求失敗或結果不確定，它 SHOULD 假定該憑證仍然有效並照常回傳，讓 Git（呼叫者）自行嘗試使用並驗證它。

回傳的 `ICredential` MAY 將使用者名稱和密碼值都保留為空字串或 `null`。這向 Git（或者說是 cURL）發出信號，表示它應該自行與遠端協商認證機制。這通常用於 Windows 整合式驗證。

#### 2.3.1 認證提示

當無法找到適合當前請求的既有憑證時，主機提供者 SHOULD 提示使用者完成一個認證流程。

執行認證的方法、模式和互動方式會因不同的 Git 託管服務及其支援的認證授權單位而有很大的差異。主機提供者 SHOULD 嘗試根據當前的環境或上下文偵測最佳的認證體驗，並選擇該體驗作為首次嘗試。

RECOMMENDED 主機提供者盡可能嘗試不需要使用者互動的認證機制。如果有多個可被同等視為「最佳」的認證機制，它們 MAY 提示使用者進行選擇。主機提供者 MAY 希望記住這樣的選擇以供將來使用，但它們 MUST 向使用者清楚說明如何清除這個已儲存的選擇。

如果需要互動才能完成認證，主機提供者 MUST 首先檢查互動是否已被禁用 (`ISettings.IsInteractionAllowed`)，如果互動已被禁止，則 MUST 拋出例外。

當有互動式「桌面」會話可用時，MUST 優先選用顯示圖形使用者介面（如視窗）的認證提示。

如果在沒有互動式會話可用但已連接終端機/TTY 的情況下需要認證提示，那麼提供者 MUST 首先檢查終端機提示是否已啟用 (`ISettings.IsTerminalPromptsEnabled`)，如果互動已被禁止，則 MUST 拋出例外。

### 2.4. 儲存憑證

主機提供者 MAY 在典型認證流程的各個階段，或在 `StoreCredentialAsync` 的呼叫中被明確要求時儲存憑證。

提供者 SHOULD 使用憑證儲存庫（以 `ICredentialStore` 的形式公開）來持久化保存密碼、PAT 和 OAuth 權杖等機密值與憑證實體。

典型的 Git 憑證輔助工具呼叫模式是一次 `get` 呼叫，接著若收到 HTTP 200 (OK) 回應則發出 `store` 請求，若收到 HTTP 401 (Unauthorized) 回應則發出 `erase` 請求。在某些情況下，`get` 請求或新憑證產生過程中存在的額外上下文，在隨後的 `store`（或 `erase`）呼叫中並不存在。在這些情況下，提供者 MAY 在 `get` 過程中儲存憑證，而不是在 `store` 過程中儲存，或兩者都儲存。

如果需要，主機提供者 MAY 在同一個請求中儲存多個憑證或權杖。需要儲存多個憑證的一個範例是 OAuth2 的 access tokens (AT) 和 refresh tokens (RT)。AT 和 RT BOTH SHOULD 使用憑證儲存庫儲存在相同的位置，並使用互補的憑證服務名稱。

### 2.5. 清除憑證

如果主機提供者已將憑證儲存在憑證儲存庫中，它們 MUST 在 `EraseCredentialAsync` 的呼叫中回應清除憑證的請求。

如果主機提供者找不到要清除的憑證，它 MUST NOT 引發錯誤，且 MUST 成功退出。可以向追蹤系統發出一條警告訊息。

主機提供者 MUST NOT 為了忽略清除請求而自行重複驗證憑證。憑證有效性的最終決定權在於呼叫者 (Git)。

當收到清除主要憑證（如 OAuth AT）的請求時，提供者 MAY 驗證任何額外或輔助憑證（如 OAuth RTs）是否仍然有效，並選擇不刪除那些額外的憑證。但在任何情況下，主要憑證 MUST 始終被清除。

### 2.6 `HostProvider` 基礎類別

提供 `HostProvider` 抽象基礎類別是為了方便主機提供者的實作者。這個基礎類別實作了 `IHostProvider` 介面的大部分必要方法，並具備常見的憑證取回與儲存行為。

`GetCredentialAsync`、`StoreCredentialAsync` 和 `EraseCredentialAsync` 方法被實作為 `virtual`，這意味著它們 MAY 被衍生類別覆寫以自訂這些操作的行為。如果實作者必須覆寫大部分已實作的方法，則 NOT RECOMMENDED 從 `HostProvider` 基礎類別衍生——實作者 SHOULD 直接實作 `IHostProvider` 介面。

選擇從此基礎類別衍生的實作者 MUST 實作所有抽象方法和屬性。要實作的主要抽象方法是 `GenerateCredentialAsync`。

還有一個額外的 `virtual` 方法名為 `GetServiceName`，`Get|Store|EraseCredentialAsync` 方法的預設實作會使用它來尋找和儲存憑證。

#### 2.6.1 `GetServiceName`

`GetServiceName` 虛擬方法如果被覆寫，MUST 回傳一個字串，用以識別此請求的服務/提供者，並用於儲存憑證。回傳的值 MUST 是穩定的——也就是說，對於相同或等效的輸入引數，它 MUST 回傳相同的值。

預設情況下，此方法會回傳完整的遠端 URI，不含結尾的斜線，並包含輸入引數中存在的協定/方案、主機名稱和路徑。輸入引數中的任何使用者名稱都不會包含在 URI 中。

#### 2.6.2 `GenerateCredentialAsync`

如果在憑證儲存庫中找不到具有相符服務（來自 `GetServiceName`）和帳戶的既有憑證，則會呼叫 `GenerateCredentialAsync` 方法。

此方法 MUST 回傳一個全新建立/產生的憑證，而不是任何既有或已儲存的憑證。它 MAY 使用既有或已儲存的輔助資料或權杖（例如 OAuth refresh tokens）來產生新的權杖（例如 OAuth AT）。

### 2.7. 外部 Metadata

主機提供者 MAY 希望儲存關於在認證操作期間收集或產生的認證或使用者的額外資料。這些資料 SHOULD 儲存在每個使用者的本地位置，例如使用者的家目錄或設定檔目錄。

機密、憑證或其他敏感資料 SHOULD 儲存在憑證儲存庫中，或透過某種形式的每個使用者本地加密來加以保護。

對於已儲存的資料快取，當呼叫 `EraseCredentialAsync` 時，提供者 SHOULD 使相關部分或整個快取失效。

## 3. 輔助工具

主機提供者 MAY 希望利用平台或作業系統特定的功能，例如原生 API 和原生圖形使用者介面，以提供更好的認證體驗。

主機提供者 MUST 在沒有輔助工具的情況下也能運作，即使該運作方式是優雅地失敗並顯示使用者友善的錯誤訊息，其中應包含修正其安裝的方法。主機提供者 SHOULD 總是在輔助工具提供的任何圖形介面之外，另外提供一個終端機/TTY 或基於文字的認證機制。

為了達成此目的，主機提供者必須引入一個跨程序的「輔助」可執行檔，可從 GCM 主程序呼叫。這允許「輔助」可執行檔在執行時期、語言等方面擁有完全的實作自由，等等。

主程序與輔助程序之間的通訊可使用任何 IPC 機制。建議實作者使用標準輸入/輸出串流或檔案描述符來傳送和接收資料，因為這與 Git 和GCM 的通訊方式一致。當需要持續的來回通訊時，也可使用 UNIX 通訊端或 Windows 具名管道。

### 3.1. 探索

建議透過簡單地檢查預期可執行檔是否存在來達成輔助程式的探索。輔助可執行檔的名稱與路徑應可讓使用者透過 Git 的設定檔進行設定。

## 4. 錯誤處理

若發生無法復原的錯誤，主機提供者必須擲出例外，且必須在錯誤訊息中包含詳細的失敗資訊。如果失敗原因可由使用者修正，錯誤訊息必須包含修正問題的說明，或提供線上文件的連結。

在發生可復原錯誤的情況下，主機提供者應印出警告訊息至標準錯誤串流，並且必須在追蹤日誌中包含錯誤資訊與所採取的復原步驟。

在發生驗證錯誤的情況下，提供者應嘗試再次提示使用者，並顯示訊息指出已輸入不正確的驗證詳細資料。

## 5. 自訂指令

如果主機提供者希望提供自訂指令，則應實作`ICommandProvider` 介面。

每個提供者都有機會建立一個單一的 `ProviderCommand` 執行個體，其他子指令可隸屬於其下。指令功能是由 `System.CommandLine` API 函式庫 [[1][references]] 提供。

對於子指令、引數或選項必須採用的格式並無限制，但實作者應嘗試遵循現有的實務與風格。

## 參考資料

1. [`System.CommandLine` API][github-dotnet-cli]

[gcm]: https://github.com/git-ecosystem/git-credential-manager
[github-dotnet-cli]: https://github.com/dotnet/command-line-api
[hostprovider-base-class]: #26-hostprovider-base-class
[references]: #references
[rfc-2119]: https://www.rfc-editor.org/rfc/rfc2119
[rfc-6749]: https://www.rfc-editor.org/rfc/rfc6749
