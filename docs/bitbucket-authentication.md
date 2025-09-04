# Bitbucket 身份驗證

當 GCM 由 Git 觸發時，它會檢查傳遞給它的 `host` 參數。如果此參數包含 `bitbucket.org`，它將觸發 Bitbucket 身份驗證並提示您輸入憑證。在這種情況下，您有兩種身份驗證選項：`OAuth` 或 `密碼/權杖`。

### OAuth

GCM 呈現的身份驗證對話方塊包含兩個分頁。第一個分頁（標示為 `Browser`）將觸發 OAuth 身份驗證。點擊 `Sign in with your browser` 按鈕會開啟一個瀏覽器請求，網址為 `_https://bitbucket.org/site/oauth2/authorize?response_type=code&client_id={consumerkey}&state=authenticated&scope={scopes}&redirect_uri=http://localhost:34106/_`。這會觸發 Bitbucket 上的一個流程，要求您登入（並可能完成兩步驟驗證）以授權 GCM 使用指定的範圍存取 Bitbucket。然後，GCM 將產生一個暫時的本機網頁伺服器，在通訊埠 34106 上監聽，以處理 OAuth 重新導向/回呼。假設您成功登入 Bitbucket 並授權 GCM，此回呼將包含 GCM 處理身份驗證所需的適當權杖。這些權杖隨後會儲存在您設定的 [憑證儲存區][credstores] 中，並返回給 Git。

### 密碼/權杖

**注意：** Bitbucket Data Center，也稱為 Bitbucket Server 或 Bitbucket On Premises，僅支援基本驗證 - 如果您使用此產品，請遵循以下說明。

GCM 呈現的身份驗證對話方塊包含兩個分頁。第二個分頁（標示為 `Password/Token`）將觸發基本驗證。此分頁包含兩個欄位，一個用於您的使用者名稱，另一個用於您的密碼或權杖。如果`username` 參數已傳遞給 GCM，它將預先填入使用者名稱欄位，但可以被覆寫。輸入您的使用者名稱（如果需要）和您的密碼或權杖（例如 Bitbucket 應用程式密碼），然後點擊 `Sign in`。

:rotating_light: 應用程式密碼要求 :rotating_light:

如果您打算使用 [應用程式密碼][app-password] 進行基本驗證，它至少必須具有_帳號讀取_權限（如下所示）。如果您的應用程式密碼沒有這些權限，您將在每次與伺服器互動時被重新提示輸入憑證。

![][app-password-example]

當您提交使用者名稱和密碼時，GCM 將嘗試透過 Bitbucket REST API 為這些憑證擷取基本驗證權杖。如果成功，憑證、使用者名稱和密碼/權杖將儲存在您設定的[憑證儲存區][credstores] 中，並返回給 Git。

如果 API 請求失敗並傳回 401 狀態碼，表示輸入的使用者名稱/密碼組合無效；不會儲存任何內容，也不會將任何內容返回給 Git。在這種情況下，請重新嘗試驗證，並確保您的憑證正確無誤。

如果 API 請求失敗並回傳 403 (Forbidden) 狀態碼，表示使用者名稱和密碼是有效的，但在對應的 Bitbucket 帳戶上已啟用兩步驟驗證 (2FA)。在此情況下，系統將提示您完成 OAuth 身份驗證流程。 如果成功，憑證、使用者名稱和密碼/權杖將會儲存在您設定的 [credential store][credstores] 中，並回傳給 Git。

[app-password]: https://support.atlassian.com/bitbucket-cloud/docs/app-passwords/
[app-password-example]: img/app-password.png
[credstores]: ./credstores.md
