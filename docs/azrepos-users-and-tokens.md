# Azure Repos：存取權杖與帳戶

## 不同的憑證類型

Azure Repos 主機提供者支援建立多種類型的憑證：

- Azure DevOps 個人存取權杖
- Microsoft 身分識別 OAuth 權杖

若要選擇 Azure Repos 主機提供者將建立及使用的憑證類型，您可以設定 [`credential.azreposCredentialType`][credential-azreposCredentialType] 組態項目 (或 [`GCM_AZREPOS_CREDENTIALTYPE`][gcm-azrepos-credential-type] 環境變數)。

### Azure DevOps 個人存取權杖

過去，Azure Repos 主機提供者唯一支援的選項是 Azure DevOps 個人存取權杖 (PATs)。

這些 PATs 僅供 Azure DevOps 使用，且必須透過 [Azure DevOps 使用者設定頁面][azure-devops-pats] 或 [REST API][azure-devops-api] 進行管理。

PATs 的生命週期有限，一旦過期就必須建立新的權杖。在 Git Credential Manager 中，當 PAT 過期 (或被手動撤銷) 時，就會導致新的驗證提示。

### Microsoft 身分識別 OAuth 權杖

「Microsoft 身分識別 OAuth 權杖」是 Azure Active Directory 針對公司與學校帳戶 (AAD 權杖) 或個人帳戶 (Microsoft Account/MSA 權杖) 所發行的 OAuth 型存取權杖的通用術語。

Azure DevOps 支援使用 Microsoft 身分識別 OAuth 權杖以及 PATs 進行 Git 驗證。由 Git Credential Manager 建立的 Microsoft 身分識別 OAuth 權杖範圍僅限於 Azure DevOps。

與 PATs 不同，只要您持續使用 Microsoft 身分識別 OAuth 權杖來執行 Git 操作，它們就會自動重新整理和續期。

這些權杖也會與其他 Microsoft 開發人員工具 (包括 Visual Studio IDE 和 Azure CLI) 安全地共用。這表示只要您使用相同的帳戶來操作 Git 或這些工具之一，就永遠不會因為權杖過期而需要重新驗證！

#### 使用者帳戶

在支援 Microsoft 身分識別 OAuth 權杖的 Git Credential Manager 版本中，現在會記住用於驗證特定 Azure DevOps 組織的使用者帳戶。

當您第一次從 Azure DevOps 組織複製 (clone)、擷取 (fetch) 或推送 (push) 時，系統會提示您登入並選取一個使用者帳戶。Git Credential Manager 會記住您使用的帳戶，並在未來所有遠端 Git 操作 (複製/擷取/推送) 中繼續使用該帳戶。一個帳戶據說「繫結 (bound)」到一個 Azure DevOps 組織。

---

**注意：** 如果 GCM 設定為使用 PAT 憑證，這個帳戶將 **不會** 被使用，而且您將繼續被提示選取一個使用者帳戶來更新憑證。這種情況未來可能會改變。

---

通常您不需要擔心管理 Git Credential Manager 使用哪些使用者帳戶，因為這在您首次為特定 Azure DevOps 組織進行驗證時會自動設定。

在進階情境中 (例如使用多個帳戶)，您可以與並使用 'azure-repos' 提供者指令來管理已記憶的使用者帳戶：

```shell
git-credential-manager azure-repos [ list | bind | unbind | ... ] <options>
```

##### 列出已記憶的帳戶

您可以列出 Git Credential Manager 為每個 Azure DevOps 組織所綁定的所有使用者帳戶，請使用 `list` 指令：

```shell
$ git-credential-manager azure-repos list
contoso:
  (global) -> alice@contoso.com
fabrikam:
  (global) -> user42@fabrikam.com
```

在上述範例中，`contoso` Azure DevOps 組織與 `alice@contoso.com` 使用者帳戶關聯，而 `fabrikam` 組織則與 `user42@fabrikam.com` 使用者帳戶關聯。

全域「綁定」適用於目前電腦使用者設定檔的所有遠端 Git 操作使用者設定檔，並儲存在 `~/.gitconfig` 或 `%USERPROFILE%\.gitconfig` 中。

##### 在儲存庫內使用不同帳戶

如果您通常為一個 Azure DevOps 組織使用單一帳戶，預設的全域綁定就已足夠。然而，如果您希望使用不同的使用者帳戶為特定儲存庫中的組織進行操作，您可以使用本機綁定。

本機帳戶綁定僅適用於單一儲存庫內，並儲存在 `.git/config` 檔案中。如果儲存庫中有本機綁定，您可以顯示它們，請使用 `list` 指令：

```shell
~/myrepo$ git-credential-manager azure-repos list
contoso:
  (global) -> alice@contoso.com
  (local)  -> alice-alt@contoso.com
```

在 `~/myrepo` 儲存庫中，`alice-alt@contoso.com` 帳戶將被 Git 和 GCM 用於 `contoso` Azure DevOps 組織。

若要建立本機綁定，當您位於儲存庫內時，請使用帶有 `--local` 選項的 `bind` 指令：

```shell
~/myrepo$ git-credential-manager azure-repos bind --local contoso alice-alt@contso.com
```

```diff
  contoso:
    (global) -> alice@contoso.com
+   (local)  -> alice-alt@contoso.com
```

##### 忘記帳戶

若要讓 Git Credential Manager 忘記某個使用者帳戶，請使用 `unbind` 指令：

```shell
git-credential-manager azure-repos unbind fabrikam
```

```diff
  contoso:
    (global) -> alice@contoso.com
- fabrikam:
-   (global) -> user42@fabrikam.com
```

在上述範例中，`fabrikam` 組織的全域帳戶綁定將會被忘記。下次您需要更新 PAT (若使用 PATs) 或執行任何遠端 Git 操作 (若使用 Azure 權杖) 時，系統將提示您再次進行驗證。

若要忘記或移除本機綁定，請在儲存庫內執行 `unbind` 指令並加上 `--local` 選項：

```shell
~/myrepo$ git-credential-manager azure-repos unbind --local contoso
```

```diff
  contoso:
    (global) -> alice@contoso.com
-   (local)  -> alice-alt@contoso.com
```

##### 為特定的 Git 遠端使用不同帳戶

除了全域和本機使用者帳戶綁定，您還可以指示 Git Credential Manager 為個別的 Git 遠端使用特定的使用者帳戶在相同的本機儲存庫中。

若要顯示儲存庫中每個 Git 遠端正在使用的帳戶，請使用 `list` 指令搭配 `--show-remotes` 選項：

```shell
~/myrepo$ git-credential-manager azure-repos list --show-remotes
contoso:
  (global) -> alice@contoso.com
  origin:
    (fetch) -> (inherit)
    (push)  -> (inherit)
fabrikam:
  (global) -> alice@fabrikam.com
```

在上述範例中，`~/myrepo` 儲存庫有一個名為 `origin` 的單一 Git 遠端，指向 `contoso` Azure DevOps 組織。沒有與 `origin` 遠端特別關聯的使用者帳戶，因此將使用 `contoso` 的全域使用者帳戶綁定 (全域綁定是繼承的)。

若要將使用者帳戶與特定的 Git 遠端關聯，您必須手動編輯遠端 URL，使用 `git config` 指令將使用者名稱包含在URL 的 [使用者資訊][rfc3986-s321] 部分。

```shell
git config --local remote.origin.url https://alice-alt%40contoso.com@contoso.visualstudio.com/project/_git/repo
```

在上述範例中，`alice-alt@contoso.com` 帳戶被設定為用於 `origin` Git 遠端的帳戶。

---

**注意：** 所有特殊字元都必須進行 URL 編碼/逸出，例如 `@` 會變成 `%40`。

---

`list --show-remotes` 指令將會顯示在
遠端 URL 中指定的使用者帳戶：

```shell
~/myrepo$ git-credential-manager azure-repos list --show-remotes
contoso:
  (global) -> alice@contoso.com
  origin:
    (fetch) -> alice-alt@contoso.com
    (push)  -> alice-alt@contoso.com
fabrikam:
  (global) -> alice@fabrikam.com
```

[azure-devops-pats]: https://docs.microsoft.com/en-us/azure/devops/organizations/accounts/use-personal-access-tokens-to-authenticate?view=azure-devops&tabs=preview-page
[credential-azreposCredentialType]: configuration.md#credentialazreposcredentialtype
[gcm-azrepos-credential-type]: environment.md#GCM_AZREPOS_CREDENTIALTYPE
[azure-devops-api]: https://docs.microsoft.com/en-gb/rest/api/azure/devops/tokens/pats
[rfc3986-s321]: https://www.rfc-editor.org/rfc/rfc3986#section-3.2.1
