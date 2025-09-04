# 安裝說明

在 macOS、Windows 和 Linux 上有多種安裝 GCM 的方式。偏好的安裝方法會以 :star: 標示。

## macOS

### Homebrew :star:

**注意：**如果您在 macOS 上已安裝 'Java GCM'，且您是透過 Homebrew 安裝的，那麼在安裝 GCM 時，此安裝將會被取消連結 (`brew unlink git-credential-manager`)。

#### 安裝

```shell
brew install --cask git-credential-manager
```

安裝後，您可以透過執行以下指令來保持最新版本：

```shell
brew upgrade --cask git-credential-manager
```

#### 解除安裝

若要解除安裝，請執行以下指令：

```shell
brew uninstall --cask git-credential-manager
```

---

### macOS 套件

#### 安裝

下載並雙擊 [安裝套件][latest-release]，然後遵循所呈現的指示。

#### 解除安裝

若要解除安裝，請執行以下指令：

```shell
sudo /usr/local/share/gcm-core/uninstall.sh
```

---

<!-- 此明確的錨點應保持穩定，以便外部文件可以連結至此 -->
<!-- markdownlint-disable-next-line no-inline-html -->
<a name="linux-install-instructions"></a>

## Linux

**注意：**所有 Linux 發行版都需要[額外設定][gcm-credstores]才能使用 GCM。

---

### .NET tool :star:

有關此安裝方法的說明，請參閱下方的 [.NET tool](#net-tool) 一節。

---

### Debian 套件

#### 安裝

下載最新的 [.deb 套件][latest-release]*，並執行以下指令：

```shell
sudo dpkg -i <path-to-package>
git-credential-manager configure
```

#### 解除安裝

```shell
git-credential-manager unconfigure
sudo dpkg -r gcm
```

*如果您想在下載後驗證套件的簽章，請查看[這裡][linux-validate-gpg-debian] 的說明。

---

### Tarball

#### 安裝

下載最新的 [tarball][latest-release]*，並執行以下指令：

```shell
tar -xvf <path-to-tarball> -C /usr/local/bin
git-credential-manager configure
```

#### 解除安裝

```shell
git-credential-manager unconfigure
rm $(command -v git-credential-manager)
```

*如果您想在下載後驗證 tarball 的簽章，請查看[這裡][linux-validate-gpg-tarball] 的說明。

---

### 從原始碼安裝輔助腳本

#### 安裝

確保已安裝 `curl`：

```shell
curl --version
```

如果 `curl` 未安裝，請使用您的發行版套件管理員來安裝它。

下載並執行腳本：

```shell
curl -L https://aka.ms/gcm/linux-install-source.sh | sh
git-credential-manager configure
```

**注意：**系統將提示您輸入憑證，以便腳本可以使用您發行版的套件管理員下載 GCM 的相依套件。

#### 解除安裝

請遵循您發行版的[這些說明][linux-uninstall]。

---

## Windows

### Git for Windows :star:

GCM 已內建於 [Git for Windows][git-for-windows] 中。在安裝過程中，系統會要求您選擇一個憑證協助程式，其中 GCM 會被列為預設選項。

![image][git-for-windows-screenshot]

---

### 獨立安裝

您也可以下載 Windows 版的 [最新安裝程式][latest-release] 來獨立安裝 GCM。

**:warning: 重要 :warning:**

在 Windows 上以獨立套件方式安裝 GCM，將會強制覆寫 Git for Windows 內建的 GCM 版本，**即使後者是較新的版本也一樣**。

Windows 上的獨立安裝有兩種版本：

- 使用者 (`gcmuser-win*`):

  不需要管理員權限。只會為目前使用者安裝且只會更新目前使用者的 Git 設定。

- 系統 (`gcm-win*`):

  需要管理員權限。會為系統上所有使用者安裝，且會更新全系統的 Git 設定。

若要安裝，請雙擊所需的安裝套件，並遵循呈現的指示操作。

### 解除安裝 (Windows 10)

若要解除安裝，請開啟「設定」應用程式並前往「應用程式」區段。選取「Git Credential Manager」，然後點擊「解除安裝」。

### 解除安裝 (Windows 7-8.1)

若要解除安裝，請開啟「控制台」並前往「程式和功能」畫面。選取「Git Credential Manager」，然後點擊「移除」。

### 適用於 Linux 的 Windows 子系統 (WSL)

Git Credential Manager 可與 [適用於 Linux 的 Windows 子系統 (WSL)][ms-wsl] 搭配使用，以啟用對您遠端 Git 存放庫從 WSL 內部進行的安全驗證。

[請參閱 GCM on WSL 文件][gcm-wsl] 以獲取更多資訊。

---

## .NET tool

GCM 可作為跨平台的 [.NET tool][dotnet-tool] 安裝。這是 Linux 的首選安裝方法，因為您可以用它在任何 [.NET 支援的發行版][dotnet-supported-distributions] 上安裝。您也可以在 macOS 上使用此方法（如果您願意的話）。

**注意：** 請確保您已安裝 [.NET 8.0 版的 SDK][dotnet-install]，然後再嘗試執行下列的 `dotnet tool` 指令。安裝後，您還需要遵循輸出指示將工具目錄新增到您的 `PATH` 中。

#### 安裝

```shell
dotnet tool install -g git-credential-manager
git-credential-manager configure
```

#### 更新

```shell
dotnet tool update -g git-credential-manager
```

#### 解除安裝

```shell
git-credential-manager unconfigure
dotnet tool uninstall -g git-credential-manager
```

[dotnet-install]: https://learn.microsoft.com/en-us/dotnet/core/install/linux#packages
[dotnet-supported-distributions]: https://learn.microsoft.com/en-us/dotnet/core/install/linux
[dotnet-tool]: https://learn.microsoft.com/en-us/dotnet/core/tools/global-tools
[gcm-credstores]: credstores.md
[gcm-wsl]: wsl.md
[git-for-windows]: https://gitforwindows.org/
[git-for-windows-screenshot]: https://user-images.githubusercontent.com/5658207/140082529-1ac133c1-0922-4a24-af03-067e27b3988b.png
[latest-release]: https://github.com/git-ecosystem/git-credential-manager/releases/latest
[linux-uninstall]: linux-fromsrc-uninstall.md
[linux-validate-gpg-debian]: ./linux-validate-gpg.md#debian-package
[linux-validate-gpg-tarball]: ./linux-validate-gpg.md#tarball
[ms-wsl]: https://aka.ms/wsl#
