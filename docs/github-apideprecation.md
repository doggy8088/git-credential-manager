# GitHub 驗證方式棄用

## 發生了什麼事？

GitHub 現在[要求使用權杖式驗證][token-auth]來呼叫其 API，未來使用 Git 本身也需要。

這表示像是 [Git Credential Manager (GCM) for Windows][gcm-windows] 這類的 Git 憑證輔助工具，以及提供使用者名稱/密碼流程的旧版 [GCM][gcm]，**將無法建立新的存取權杖**來存取 Git 儲存庫。

如果您已經有由 GCM for Windows 這類 Git 憑證輔助工具產生的權杖，它們將繼續運作直到過期或被撤銷/刪除為止。

## 我現在該怎麼辦？

### Windows 命令列使用者

目前最好的作法是升級到最新版的 Git for Windows (至少 2.29 版)，其中包含使用受支援 OAuth 權杖式驗證的 Git Credential Manager。

[下載最新版 Git for Windows ⬇️][git-windows]

### Visual Studio 使用者

請更新到 Visual Studio 最新的支援版本，該版本包含 GCM 並支援 OAuth 權杖式驗證。

- [Visual Studio 2019 ⬇️][vs-2019]
- [Visual Studio 2017 ⬇️][vs-2017]

### SSH、macOS 和 Linux 使用者

如果您使用 SSH，此變更**不會**影響到您。

如果您使用的是舊版 Git Credential Manager (2.0.124-beta 之前)，請遵循[這些說明][gcm-install]升級到最新版本。

## 如果我無法升級 Git for Windows 怎麼辦？

如果您無法升級 Git for Windows，可以手動安裝獨立版的 Git Credential Manager。這將會覆蓋 Git for Windows 安裝程式中捆綁的舊版 GCM for Windows。

[下載 Git Credential Manager 獨立版 ⬇️][gcm-latest]

## 如果我無法使用 Git Credential Manager 怎麼辦？

如果您因為錯誤或相容性問題而無法使用 Git Credential Manager，我們[想知道原因][gcm-new-issue]！

## 救命！我沒有管理員權限，無法對我的 Windows 電腦做任何變更

如果您沒有權限變更您的安裝 (例如在公司環境中)，您可以使用個別使用者安裝程式。請查看[最新版本][gcm-latest]並下載 `gcmcoreuser-win-*.exe` 可執行檔。

### 救命！我還是無法或不想安裝任何東西

有一個變通辦法應該可行，而且不需要安裝任何東西。

1. 告訴您的系統管理員，他們應該開始計畫將已安裝的 Git for Windows 版本升級到至少 2.29！ 😁

1. [建立新的個人存取權杖][github-pat] (請參閱官方[文件][github-pat-docs])

1. 為權杖輸入一個名稱 (「note」)，並確保選取 `repo`、`gist` 和 `workflow` 範圍：
   ![image][github-pat-note-image]
   ...
   ![image][github-pat-repo-scope-image]
   ...
   ![image][github-pat-gist-scope-image]
   ...
   ![image][github-pat-workflow-scope-image]

1. 點擊「Generate Token」

   ![image][github-generate-pat-image]

1. **[重要]** 請保持結果頁面開啟，因為其中包含您的新權杖（這個只會顯示一次！）

   ![image][github-display-pat-image]

1. 將產生的 PAT 儲存在 Windows 認證管理員中：

   1. 如果您偏好使用命令列，請開啟命令提示字元 (cmd.exe) 並輸入以下內容：

      ```bash
      cmdkey /generic:git:https://github.com /user:PersonalAccessToken /pass
      ```

      系統將提示您輸入密碼 – 請複製在步驟 4 中新產生的 PAT 並將其貼在此處，然後按下 `Enter` 鍵

      ![image][windows-cli-save-pat-image]

   1. 如果您不想使用命令列，請[開啟認證管理員透過控制台][windows-credential-manager]並選取「Windows 認證」索引標籤。

      ![image][windows-gui-credentials-image]

      按一下「新增一般認證」，然後輸入下列詳細資料：

      - 網際網路或網路位址： `git:https://github.com`
      - 使用者名稱： `PersonalAccessToken`
      - 密碼： _(在此複製並貼上步驟 4 中產生的 PAT)_

      ![image][windows-gui-add-pat-image]

## GitHub Enterprise Server (GHES) 的情況如何？

如[部落格文章][github-token-authentication-requirements]中所述，新的權杖式驗證要求**不**適用於 GHES：

> 我們尚未宣布對 GitHub Enterprise Server 的任何變更，其仍然目前不受影響。

[token-auth]: https://github.blog/2020-07-30-token-authentication-requirements-for-api-and-git-operations/
[gcm]: https://aka.ms/gcm
[gcm-install]: ../README.md#download-and-install
[gcm-latest]: https://aka.ms/gcm/latest
[gcm-new-issue]: https://github.com/git-ecosystem/git-credential-manager/issues/new/choose
[gcm-windows]: https://github.com/microsoft/Git-Credential-Manager-for-Windows
[git-windows]: https://git-scm.com/download/win
[github-display-pat-image]: img/github-display-pat.png
[github-generate-pat-image]: img/github-generate-pat.png
[github-pat]: https://github.com/settings/tokens/new?scopes=repo,gist,workflow
[github-pat-docs]: https://docs.github.com/en/free-pro-team@latest/github/authenticating-to-github/creating-a-personal-access-token
[github-pat-gist-scope-image]: img/github-pat-gist-scope.png
[github-pat-note-image]: img/github-pat-note.png
[github-pat-repo-scope-image]: img/github-pat-repo-scope.png
[github-pat-workflow-scope-image]: img/github-pat-workflow-scope.png
[github-token-authentication-requirements]: https://github.blog/2020-07-30-token-authentication-requirements-for-api-and-git-operations/
[windows-cli-save-pat-image]: img/windows-cli-save-pat.png
[vs-2019]: https://docs.microsoft.com/en-us/visualstudio/install/update-visual-studio?view=vs-2019
[vs-2017]: https://docs.microsoft.com/en-us/visualstudio/install/update-visual-studio?view=vs-2017
[windows-credential-manager]: https://support.microsoft.com/en-us/windows/accessing-credential-manager-1b5c916a-6a16-889f-8581-fc16e8165ac0
[windows-gui-add-pat-image]: img/windows-gui-add-pat.png
[windows-gui-credentials-image]: img/windows-gui-credentials.png
