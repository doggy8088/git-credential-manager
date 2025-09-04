# Git Credential Manager 更名

在 2021 年 11 月，_「Git Credential Manager Core」_ [更名為][rename-pr] _"Git Credential Manager"_，移除了 "Core" 這個名稱。我們在一篇 [GitHub 部落格文章][rename-blog] 中宣布了新名稱，同時也公布了該專案位於其專屬 [組織][gcm-org] 的新家。

![Git Credential Manager Core 更名](img/gcmcore-rename.png)

當時，實際的執行檔名稱並未更新，仍然是 `git-credential-manager-core`。自 [2.0.877][rename-ver] 版本起，執行檔已更名為 `git-credential-manager`，與新的專案名稱一致。

---

:warning: **Update:** :warning:

自 [2.3.0][no-symlink-ver] 版本起，`git-credential-manager-core` 的符號連結已被移除。

若您尚未更新設定，將會看到類似以下的錯誤訊息：

```console
git: 'credential-manager-core' is not a git command. See 'git --help'.
```

要修正您的設定，請依照下方的 [說明][instructions] 操作。

---

## 更名過渡期

若您繼續使用 `git-credential-manager-core` 這個執行檔名稱，您可能會看到如下的警告訊息：

```console
warning: git-credential-manager-core was renamed to git-credential-manager
warning: see https://aka.ms/gcm/rename for more information
```

自 2.0.877 版本執行檔更名以來，GCM 也包含了使用舊名稱的符號連結，以確保任何人的設定不會立即中斷。

這些連結將會保留，直到 GCM 2.0.877 之後發布 _兩個_ 主要的 Git 版本為止，2.0.877，_**屆時將不再包含這些符號連結**_。

建議您盡快更新您的 Git 設定，以使用新的執行檔名稱，以避免未來發生任何問題。

## 如何更新

### Git for Windows

若您正在使用與 Git for Windows 捆綁的 GCM (建議方式)，您應確保您已更新至最新版本。

[下載最新版 Git for Windows ⬇️][git-windows]

### Windows 獨立安裝程式

若您是透過使用者 (`gcmuser-*.exe`) 或系統 (`gcm-*.exe`) 安裝程式在 Windows 上安裝 GCM，您應先解除安裝目前的版本，然後再下載並安裝 [最新版本][gcm-latest]。

適用於您 Windows 版本的解除安裝說明可以在[這裡][win-standalone-instr] 找到。

### macOS Homebrew

> **注意：** 自 2022 年 10 月起，舊的 `git-credential-manager-core` cask 名稱仍在使用中。未來我們計劃將套件更名，移除 `-core` 後綴。

若您使用 Homebrew 在 macOS 上安裝 GCM，您應使用 `brew upgrade` 來安裝最新版本。

```sh
brew upgrade git-credential-manager-core
```

### macOS 套件

若您使用 .pkg 檔案在 macOS 上安裝 GCM，您應先解除安裝目前的版本，然後再安裝 [最新的套件][gcm-latest]。

```sh
sudo /usr/local/share/gcm-core/uninstall.sh
installer -pkg <path-to-new-package> -target /
```

### Linux Debian 套件

如果您在 Linux 上使用 .deb Debian 套件安裝 GCM，您應該先 `unconfigure` 目前的版本，解除安裝套件，然後安裝並 `configure` [最新版本][gcm-latest]。

```sh
git-credential-manager-core unconfigure
sudo dpkg -r gcmcore
sudo dpkg -i <path-to-new-package>
git-credential-manager configure
```

### Linux tarball

如果您正在使用我們 tarball 中預先建構的 GCM Linux 二進位檔，您應該先 `unconfigure` 目前的版本，然後再解壓縮 [最新的二進位檔][gcm-latest]。

```sh
git-credential-manager-core unconfigure
rm $(command -v git-credential-manager-core)
tar -xvf <path-to-new-tarball> -C /usr/local/bin
git-credential-manager configure
```

### 疑難排解

如果在更新 GCM 安裝後，您仍然看到[警告][warnings] 訊息，您可以嘗試手動編輯您的 Git 設定以指向正確的 GCM 執行檔名稱。

首先，使用以下指令列出所有關於 `credential.helper` 的 Git 設定，包括特定設定項目所在的檔案：

```sh
git config --show-origin --get-all credential.helper
```

在 Mac 或 Linux 上，您應該會看到類似這樣的內容：

<!-- markdownlint-disable MD010 -->
```shell-session
$ git config --show-origin --get-all credential.helper
file:/opt/homebrew/etc/gitconfig	credential.helper=osxkeychain
file:/Users/jdoe/.gitconfig	credential.helper=
file:/Users/jdoe/.gitconfig	credential.helper=/usr/local/share/gcm-core/git-credential-manager-core
```

在 Windows 上，您應該會看到類似這樣的內容：

```shell-session
> git config --show-origin --get-all credential.helper
file:C:/Program Files/Git/etc/gitconfig	credential.helper=manager-core
```
<!-- markdownlint-enable MD010 -->

請留意包含 `git-credential-manager-core` 或 `manager-core` 的項目；這些項目應該被取代並更新為 `git-credential-manager` 或分別更新為 `manager`。

> **注意：** 當更新您家目錄中的 Git 設定檔 (`$HOME/.gitconfig` 或 `%USERPROFILE%\.gitconfig`) 時，您應確保有在 GCM 項目之前有一個額外的空白 `credential.helper` 項目。
>
> **Mac/Linux**
>
> ```ini
> [credential]
>     helper =
>     helper = /usr/local/share/gcm-core/git-credential-manager
> ```
>
> **Windows**
>
> ```ini
> [credential]
>     helper =
>     helper = C:/Program\\ Files\\ \\(x86\\)/Git\\ Credential\\ Manager/git-credential-manager.exe
> ```
>
> 這個空白項目很重要，因為它能確保 GCM 是唯一設定的憑證輔助程式，並覆寫任何在系統層級/全機層級設定的輔助程式。

[rename-pr]: https://github.com/git-ecosystem/git-credential-manager/pull/541
[rename-blog]: https://github.blog/2022-04-07-git-credential-manager-authentication-for-everyone/#universal-git-authentication
[gcm-org]: https://github.com/git-ecosystem
[rename-ver]: https://github.com/git-ecosystem/git-credential-manager/releases/tag/v2.0.877
[git-windows]: https://git-scm.com/download/win
[gcm-latest]: https://aka.ms/gcm/latest
[warnings]: #rename-transition
[win-standalone-instr]: ../README.md#standalone-installation
[instructions]: #how-to-update
[no-symlink-ver]: https://github.com/git-ecosystem/git-credential-manager/releases/tag/v2.3.0
