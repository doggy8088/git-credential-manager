# Git 憑證管理員

[![Build Status][build-status-badge]][workflow-status]

---

[Git 憑證管理員][gcm] (GCM) 是一個基於 [.NET][dotnet] 建置的安全
[Git 憑證助手][git-credential-helper]，可在 Windows、macOS 和 Linux 上執行。
它旨在為每個主要的原始碼控制託管服務和平台提供一致且安全的身份驗證體驗，
包括多因素驗證。

GCM 支援（按字母順序）[Azure DevOps][azure-devops]、Azure DevOps
Server（前身為 Team Foundation Server）、Bitbucket、GitHub 和 GitLab。
與 Git 的[內建憑證助手][git-tools-credential-storage]相比
（Windows：wincred，macOS：osxkeychain，Linux：gnome-keyring/libsecret），
後者僅提供使用者名稱/密碼的單因素驗證支援。

GCM 取代了基於 .NET Framework 的
[Git Credential Manager for Windows][gcm-for-windows] 和基於 Java 的
[Git Credential Manager for Mac and Linux][gcm-for-mac-and-linux]。

## 安裝

請參閱 GCM 當前版本的[安裝說明][install]，了解您作業系統的安裝選項。

## 目前狀態

Git 憑證管理員目前可用於 Windows、macOS 和 Linux\*。
GCM 僅適用於 HTTP(S) 遠端；您仍可以使用 Git 與 SSH：

- [Azure DevOps SSH][azure-devops-ssh]
- [GitHub SSH][github-ssh]
- [Bitbucket SSH][bitbucket-ssh]

功能|Windows|macOS|Linux\*
-|:-:|:-:|:-:
安裝程式/解除安裝程式|&#10003;|&#10003;|&#10003;
安全平台憑證儲存 [(查看更多)][gcm-credstores]|&#10003;|&#10003;|&#10003;
Azure DevOps 多因素驗證支援|&#10003;|&#10003;|&#10003;
GitHub 雙因素驗證支援|&#10003;|&#10003;|&#10003;
Bitbucket 雙因素驗證支援|&#10003;|&#10003;|&#10003;
GitLab 雙因素驗證支援|&#10003;|&#10003;|&#10003;
Windows 整合驗證 (NTLM/Kerberos) 支援|&#10003;|_不適用_|_不適用_
基本 HTTP 驗證支援|&#10003;|&#10003;|&#10003;
代理伺服器支援|&#10003;|&#10003;|&#10003;
`amd64` 支援|&#10003;|&#10003;|&#10003;
`x86` 支援|&#10003;|_不適用_|&#10007;
`arm64` 支援|盡力而為|&#10003;|&#10003;
`armhf` 支援|_不適用_|_不適用_|&#10003;

(\*) GCM 僅保證支援 [dotnet 官方支援的 Linux 發行版][dotnet-distributions]。

## 支援的 Git 版本

Git 憑證管理員試圖與最廣泛的 Git 版本集合相容（在合理範圍內）。
然而，有一些已知的問題版本的 Git 與 GCM 不相容。

- Git 1.x

  不支援或測試 Git 的初始主要版本與 GCM。

- Git 2.26.2

  這個版本的 Git 在解析 GCM 依賴的憑證配置時引入了一個重大變更。
  這個問題在 Git 專案的提交 [`12294990`][gcm-commit-12294990] 中修復，
  並在 Git 2.27.0 中發布。

## 如何使用

一旦安裝和配置完成，Git 憑證管理員會被 Git 隱式呼叫。您不需要做任何特殊的事情，
GCM 也不打算由使用者直接呼叫。例如，當推送（`git push`）到
[Azure DevOps][azure-devops]、[Bitbucket][bitbucket] 或 [GitHub][github] 時，
會自動開啟一個視窗並引導您完成登入過程。（此過程對於每個 Git 主機看起來會略有不同，
甚至在某些情況下，取決於您是否連接到本地或雲端託管的 Git 主機。）
在同一個儲存庫中的後續 Git 命令將重新使用 GCM 儲存的現有憑證或權杖，
只要它們仍然有效。

閱讀完整的命令列使用說明[在此][gcm-usage]。

### 配置代理伺服器

詳細資訊請參閱[此處][gcm-http-proxy]。

## 其他資源

請參閱[文件索引][docs-index]以獲取其他資源的連結。

## 實驗性功能

- [Windows 代理程式（實驗性）][gcm-windows-broker]

## 未來功能

想知道 GCM 專案的下一步計畫嗎？查看[專案路線圖][roadmap]！
您可以在[此處][roadmap-announcement]找到有關路線圖構建方式和解讀方法的更多詳細資訊。

## 貢獻

本專案歡迎貢獻和建議。
請參閱[貢獻指南][gcm-contributing]開始參與。

本專案遵循 [GitHub 的開源行為準則][gcm-coc]。

## 授權條款

我們使用 [MIT][gcm-license] 授權條款。
使用 GitHub 標誌時，請務必遵循 [GitHub 標誌指南][github-logos]。

[azure-devops]: https://azure.microsoft.com/en-us/products/devops
[azure-devops-ssh]: https://docs.microsoft.com/en-us/azure/devops/repos/git/use-ssh-keys-to-authenticate?view=azure-devops
[bitbucket]: https://bitbucket.org
[bitbucket-ssh]: https://confluence.atlassian.com/bitbucket/ssh-keys-935365775.html
[build-status-badge]: https://github.com/git-ecosystem/git-credential-manager/actions/workflows/continuous-integration.yml/badge.svg
[docs-index]: https://github.com/git-ecosystem/git-credential-manager/blob/release/docs/README.md
[dotnet]: https://dotnet.microsoft.com
[dotnet-distributions]: https://learn.microsoft.com/en-us/dotnet/core/install/linux
[git-credential-helper]: https://git-scm.com/docs/gitcredentials
[gcm]: https://github.com/git-ecosystem/git-credential-manager
[gcm-coc]: CODE_OF_CONDUCT.md
[gcm-commit-12294990]: https://github.com/git/git/commit/12294990c90e043862be9eb7eb22c3784b526340
[gcm-contributing]: CONTRIBUTING.md
[gcm-credstores]: https://github.com/git-ecosystem/git-credential-manager/blob/release/docs/credstores.md
[gcm-for-mac-and-linux]: https://github.com/microsoft/Git-Credential-Manager-for-Mac-and-Linux
[gcm-for-windows]: https://github.com/microsoft/Git-Credential-Manager-for-Windows
[gcm-http-proxy]: https://github.com/git-ecosystem/git-credential-manager/blob/release/docs/netconfig.md#http-proxy
[gcm-license]: LICENSE
[gcm-usage]: https://github.com/git-ecosystem/git-credential-manager/blob/release/docs/usage.md
[gcm-windows-broker]: https://github.com/git-ecosystem/git-credential-manager/blob/release/docs/windows-broker.md
[git-tools-credential-storage]: https://git-scm.com/book/en/v2/Git-Tools-Credential-Storage
[github]: https://github.com
[github-ssh]: https://help.github.com/en/articles/connecting-to-github-with-ssh
[github-logos]: https://github.com/logos
[install]: https://github.com/git-ecosystem/git-credential-manager/blob/release/docs/install.md
[roadmap]: https://github.com/git-ecosystem/git-credential-manager/milestones?direction=desc&sort=due_date&state=open
[roadmap-announcement]: https://github.com/git-ecosystem/git-credential-manager/discussions/1203
[workflow-status]: https://github.com/git-ecosystem/git-credential-manager/actions/workflows/continuous-integration.yml
