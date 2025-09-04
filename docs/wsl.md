# 適用於 Linux 的 Windows 子系統 (WSL)

GCM 只要遵循以下說明，即可與[適用於 Linux 的 Windows 子系統 (WSL)][wsl] (WSL1 和 WSL2) 搭配使用。

若要將 GCM 與 WSL 搭配使用，您的 Windows 10 版本必須為 1903 或更新版本。
這是第一個包含 GCM 與 WSL 發行版中的 Git 互相操作時所需的 `wsl.exe` 工具的 Windows 版本。

強烈建議您安裝 Windows 版 Git，以便同時安裝 GCM 並在 WSL 和 Windows 主機之間享有共用認證與設定的最佳體驗。或者，您必須使用 2.0.XXX 或更新版本的 GCM，並按照[下文所述][configuring-wsl-without-git-for-windows]設定 `WSLENV` 環境變數。

## 使用 Windows 版 Git 設定 WSL (建議方式)

首先安裝[最新版的 Windows 版 Git ⬇️][latest-git-for-windows]

_在您的 WSL 安裝中_，執行下列命令以將 GCM 設定為 Git 認證輔助程式：

```shell
git config --global credential.helper "/mnt/c/Program\ Files/Git/mingw64/bin/git-credential-manager.exe"
```

> **注意：** 在您的 Windows 版 Git 安裝中，git-credential-manager.exe 的位置可能會有所不同。

如果您打算使用 Azure DevOps，您還必須_在您的 WSL 安裝中_設定下列 Git 組態。

```shell
git config --global credential.https://dev.azure.com.useHttpPath true
```

## 不使用 Windows 版 Git 設定 WSL

如果您希望在 WSL 中使用 GCM _而不安裝 Windows 版 Git_，您必須完成額外的設定，GCM 才能回呼至您 WSL 安裝中的 Git。

首先安裝[最新版的 Windows 版 GCM⬇️][latest-gcm]

_在您的 WSL 安裝中_，執行下列命令以將 GCM 設定為 Git 認證輔助程式：

```shell
git config --global credential.helper "/mnt/c/Program\ Files\ \(x86\)/Git\ Credential\ Manager/git-credential-manager.exe"

# 僅支援 Azure DevOps
git config --global credential.https://dev.azure.com.useHttpPath true
```

在 **_Windows_** 中，您需要更新 `WSLENV` 環境變數以包含 `GIT_EXEC_PATH/wp` 的值。請從_系統管理員_命令提示字元執行下列命令：

```batch
SETX WSLENV %WSLENV%:GIT_EXEC_PATH/wp
```

更新 `WSLENV` 環境變數後，請重新啟動您的 WSL 安裝。

### 使用僅限使用者的 GCM 安裝程式？

如果您是使用僅限使用者的安裝程式 (即 `gcmuser-*.exe` 安裝程式，而非全系統/需要系統管理員權限的安裝程式) 來安裝 GCM，您需要修改上述說明，改為指向 `/mnt/c/Users/<USERNAME>/AppData/Local/Programs/Git\ Credential\ Manager/git-credential-manager.exe`。

## 運作方式

GCM 利用了 Microsoft 提供的 Windows 與 WSL 之間的內建互通性。您可以在[此處][wsl-interop]閱讀更多關於 Windows/WSL 互通性的資訊。

WSL 安裝中的 Git 可以透明地啟動 GCM _Windows_ 應用程式以取得認證。以 Windows 應用程式的形式執行 GCM，使其能充分利用主機作業系統來安全地儲存認證，並顯示用於驗證的 GUI 提示。

使用主機作業系統 (Windows) 儲存認證也意味著您的 Windows 應用程式和 WSL 發行版都可以共用這些認證，無需多次登入。

## 共用組態

將 GCM 作為 WSL Git 安裝的認證輔助程式，意味著在 WSL Git 中設定的任何組態預設情況下都「不會」被 GCM 採用。這是因為 GCM 是以 Windows 應用程式的形式執行，因此它會使用 Windows 版 Git 的安裝來查詢組態。

這表示像 GCM 的代理伺服器設定等，需要在 Windows 版 Git 和 WSL Git 中都進行設定，因為它們儲存在不同的檔案中 (`%USERPROFILE%\.gitconfig` vs `\\wsl$\distro\home\$USER\.gitconfig`)。

您可以依照[上述說明][configuring-wsl-without-git-for-windows]設定 WSL，讓 GCM 使用 WSL Git 的組態。然而，這也意味著像代理伺服器設定等將會是特定 WSL 安裝所獨有，而不會與其他發行版或 Windows 主機共用。

## 我可以直接在 WSL 中安裝 Git 認證管理員嗎？

可以。您可以不安裝 GCM 作為 Windows 應用程式 (並讓 WSL Git 呼叫 Windows GCM)，而是將 GCM 安裝為 Linux 應用程式。

要這麼做，只需遵循 [GCM for Linux 的安裝說明][linux-installation] 即可。

**注意：** 在此情境下，由於 GCM 是以 Linux 應用程式的形式執行，它無法利用主機 Windows 作業系統的驗證或認證儲存功能。

[wsl]: https://aka.ms/wsl
[configuring-wsl-without-git-for-windows]: #configuring-wsl-without-git-for-windows
[latest-git-for-windows]: https://github.com/git-for-windows/git/releases/latest
[latest-gcm]: https://aka.ms/gcm/latest
[wsl-interop]: https://docs.microsoft.com/en-us/windows/wsl/interop
[linux-installation]: ../README.md#linux
