# GitLab 支援

Git Credential Manager 內建支援 [gitlab.com][gitlab]。

## 在其他實例上使用

若要在其他實例（例如 `https://gitlab.example.com`）上使用，需要進行設定與配置：

1. [建立一個 OAuth 應用程式][gitlab-oauth]。這可以在使用者、群組或實例層級進行。指定一個名稱並使用 `http://127.0.0.1/` 作為重導向 URI。_取消勾選_「Confidential」選項。設定「read_repository」和「write_repository」範圍。
1. 複製應用程式 ID 並設定
`git config --global credential.https://gitlab.example.com.gitLabDevClientId <APPLICATION_ID>`
1. 複製應用程式密鑰並設定
`git config --global credential.https://gitlab.example.com.gitLabDevClientSecret
<APPLICATION_SECRET>`
1. （可選）如果您想強制使用瀏覽器驗證：
`git config --global credential.https://gitlab.example.com.gitLabAuthModes browser`
1. 為求保險，請設定
`git config --global credential.https://gitlab.example.com.provider gitlab`。
這對於將該網域辨識為 GitLab 實例可能是必要的。
1. 驗證設定是否符合預期
`git config --global --get-urlmatch credential https://gitlab.example.com`

### 清除設定

```console
git config --global --unset-all credential.https://gitlab.example.com.gitLabDevClientId
git config --global --unset-all credential.https://gitlab.example.com.gitLabDevClientSecret
git config --global --unset-all credential.https://gitlab.example.com.provider
```

### 常用實例的設定

為方便起見，以下是由社群成員 [hickford](https://github.com/hickford/) 提供的數個常用 GitLab 實例的設定指令：

```console
# https://gitlab.freedesktop.org/
git config --global credential.https://gitlab.freedesktop.org.gitLabDevClientId 6503d8c5a27187628440d44e0352833a2b49bce540c546c22a3378c8f5b74d45
git config --global credential.https://gitlab.freedesktop.org.gitLabDevClientSecret 2ae9343a034ff1baadaef1e7ce3197776b00746a02ddf0323bb34aca8bff6dc1
# https://gitlab.gnome.org/
git config --global credential.https://gitlab.gnome.org.gitLabDevClientId adf21361d32eddc87bf6baf8366f242dfe07a7d4335b46e8e101303364ccc470
git config --global credential.https://gitlab.gnome.org.gitLabDevClientSecret cdca4678f64e5b0be9febc0d5e7aab0d81d27696d7adb1cf8022ccefd0a58fc0
# https://invent.kde.org/
git config --global credential.https://invent.kde.org.gitLabDevClientId cd7cb4342c7cd83d8c2fcc22c87320f88d0bde14984432ffca07ee24d0bf0699
git config --global credential.https://invent.kde.org.gitLabDevClientSecret 9cc8440b280c792ac429b3615ae1c8e0702e6b2479056f899d314f05afd94211
# https://salsa.debian.org/
git config --global credential.https://salsa.debian.org.gitLabDevClientId 213f5fd32c6a14a0328048c0a77cc12c19138cc165ab957fb83d0add74656f89
git config --global credential.https://salsa.debian.org.gitLabDevClientSecret 3616b974b59451ecf553f951cb7b8e6e3c91c6d84dd3247dcb0183dac93c2a26
# https://gitlab.haskell.org/
git config --global credential.https://gitlab.haskell.org.gitLabDevClientId 57de5eaab72b3dc447fca8c19cea39527a08e82da5377c2d10a8ebb30b08fa5f
git config --global credential.https://gitlab.haskell.org.gitLabDevClientSecret 5170a480da8fb7341e0daac94223d4fff549c702efb2f8873d950bb2b88e434f
# https://code.videolan.org/
git config --global credential.https://code.videolan.org.gitLabDevClientId f35c379241cc20bf9dffecb47990491b62757db4fb96080cddf2461eacb40375
git config --global credential.https://code.videolan.org.gitLabDevClientSecret 631558ec973c5ef65b78db9f41103f8247dc68d979c86f051c0fe4389e1995e8
```

另請參閱 [issue #677](https://github.com/git-ecosystem/git-credential-manager/issues/677)。

## 偏好設定

```console
為 'https://gitlab.com/' 選擇一種驗證方法：
  1. 網頁瀏覽器 (預設)
  2. 個人存取權杖
  3. 使用者名稱/密碼
選項 (按 Enter 使用預設值)：
```

如果您有偏好的驗證模式，可以指定 [credential.gitLabAuthModes][config-gitlab-auth-modes]：

```console
git config --global credential.gitLabAuthModes browser
```

## 注意事項

更完善的支援需要 GitLab 進行變更。如果下列議題對您有影響，請前往投票支持：

1. 不支援 OAuth 裝置授權（對於沒有網頁瀏覽器的機器是必要的）：[GitLab issue 332682][gitlab-issue-332682]
1. 將 Git Credential Manager 預設設定為實例層級的 OAuth 應用程式：[GitLab issue 374172](gitlab-issue-374172)
1. 即使伺服器已停用使用者名稱/密碼驗證，系統仍會提示此選項：[GitLab issue 349463][gitlab-issue-349463]

[config-gitlab-auth-modes]: configuration.md#credential.gitLabAuthModes
[gitlab]: https://gitlab.com
[gitlab-issue-332682]: https://gitlab.com/gitlab-org/gitlab/-/issues/332682
[gitlab-issue-374172]: https://gitlab.com/gitlab-org/gitlab/-/issues/374172
[gitlab-issue-349463]: https://gitlab.com/gitlab-org/gitlab/-/issues/349463
[gitlab-oauth]: https://docs.gitlab.com/ee/integration/oauth_provider.html
