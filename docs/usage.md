# 命令列用法

安裝後，Git 將會使用 Git Credential Manager，您只需要與任何要求憑證的驗證對話框互動即可。GCM 會盡可能地保持隱形，所以理想情況下，您會忘記自己正在依賴 GCM。

假設 GCM 已安裝，請使用您慣用的終端機執行下列指令來直接與 GCM 互動。

```shell
git credential-manager [<command> [<args>]]
```

## 命令

### --help / -h / -?

顯示可用指令的列表。

### --version

顯示目前版本。

### get / store / erase

與 Git 互動的指令。您不需要手動執行這些指令。

閱讀 [Git 手冊][git-credentials-custom-helpers] 中關於自訂輔助程式的說明以取得
更多資訊。

### configure/unconfigure

設定您的使用者層級 Git 設定 (`~/.gitconfig`) 以使用 GCM。如果您傳遞 `--system` 給這些指令，它們將會改為作用於系統層級的 Git 設定 (`/etc/gitconfig`)。

### azure-repos

與 Azure Repos 主機提供者互動，以將使用者帳號綁定/取消綁定至 Azure DevOps 組織或特定的遠端 URL，並管理驗證授權快取。
有關管理使用者帳號綁定的更多資訊，請參閱[此處][azure-access-tokens-ua]。
github

[azure-access-tokens-ua]: azrepos-users-and-tokens.md#useraccounts
[git-credentials-custom-helpers]: https://git-scm.com/docs/gitcredentials#_custom_helpers

### github

與 GitHub 主機提供者互動，以管理您在 GitHub.com 和 GitHub Enterprise Server 執行個體上的帳號。
