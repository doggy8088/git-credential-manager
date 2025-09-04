# 遷移指南

## 從 Git Credential Manager for Windows 遷移

### GCM_AUTHORITY

此設定（以及對應的 `credential.authority` 組態）已棄用，應更換為 `GCM_PROVIDER`（或對應的 `credential.provider` 組態）設定。

由於 Basic HTTP 驗證與 Windows 整合式驗證 (WIA) 現在皆由單一 provider 處理，如果您先前指定 `basic` 為您的
authority，您也需要使用 `GCM_ALLOW_WINDOWSAUTH` / `credential.allowWindowsAuth` 來停用 WIA。

下表顯示所有舊版 authorities 的正確替代方案值：

GCM_AUTHORITY (credential.authority)|&rarr;|GCM_PROVIDER (credential.provider)|GCM_ALLOW_WINDOWSAUTH (credential.allowWindowsAuth)
-|-|-|-
`msa`, `microsoft`, `microsoftaccount`, `aad`, `azure`, `azuredirectory`, `live`, `liveconnect`, `liveid`|&rarr;|`azure-repos`|_不適用_
`github`|&rarr;|`github`|_不適用_
`basic`|&rarr;|`generic`|`false`
`integrated`, `windows`, `kerberos`, `ntlm`, `tfs`, `sso`|&rarr;|`generic`|`true` _(預設)_

例如，如果您先前已將 `example.com` 主機的 authority 設定為
`basic`..

```shell
git config --global credential.example.com.authority basic
```

..那麼您可以將其更換為以下設定..

```shell
git config --global --unset credential.example.com.authority
git config --global credential.example.com.provider generic
git config --global credential.example.com.allowWindowsAuth false
```
