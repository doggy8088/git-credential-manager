# 從原始碼安裝後移除

這些說明將引導您，在執行您 Linux 發行版上的 [從原始碼安裝腳本][install-from-source] 後，移除 GCM。

 :rotating_light: 請小心執行 :rotating_light: 

為求完整，我們提供了針對 _GCM 應用程式、GCM repo，以及所有發行版中 _最大數量的相依套件*_ 的移除說明。此 repo 與這些相依套件可能已經存在、也可能尚未存在於您執行從原始碼安裝腳本時的系統中，而移除它們可能會影響其他程式及/或您的正常工作流程。在遵循以下說明時，請將此謹記在心。

*特定發行版需要腳本的某些相依套件才能如預期般運作，因此我們僅提供移除非必要相依套件的說明。

## 所有發行版

**注意：** 如果您從一個已存在的 `git-credential-manager` repo 複本或您的 `$HOME` 目錄以外的地方執行「從原始碼安裝」腳本，您將需要修改下方的最後兩個指令，使其指向您已存在的複本，或您執行從原始碼安裝腳本所在的目錄。

```console
git-credential-manager unconfigure &&
sudo rm $(command -v git-credential-manager) &&
sudo rm -rf /usr/local/share/gcm-core &&
sudo rm -rf ~/git-credential-manager &&
sudo rm ~/install-from-source.sh
```

## Debian/Ubuntu

**注意：** 如果您在執行從原始碼安裝腳本時，已有一個 dotnet 的既有安裝，且該安裝不是透過 `apt` 或 `apt-get` 所安裝，您將需要使用 [這些說明][uninstall-dotnet] 將其移除，並從下方的指令中移除 `dotnet-*`。

```console
sudo apt remove dotnet-* dpkg-dev apt-transport-https git curl wget
```

## Linux Mint

**注意：** 如果您在執行從原始碼安裝腳本時，已有一個不在 `~/.dotnet` 的 dotnet 既有安裝，您將需要修改下方的第一個指令，使其指向自訂的安裝位置。如果您想要移除腳本所安裝的特定 dotnet 版本並保留其他版本，您可以透過 [這些說明][uninstall-dotnet] 來完成。

```console
sudo rm -rf ~/.dotnet &&
sudo apt remove git curl
```

## Fedora/CentOS/RHEL

**注意：** 如果您在執行從原始碼安裝腳本時，已有一個不在 `~/.dotnet` 的 dotnet 既有安裝，您將需要修改下方的第一個指令，使其指向自訂的安裝位置。如果您想要移除腳本所安裝的特定 dotnet 版本並保留其他版本，您可以透過 [這些說明][uninstall-dotnet] 來完成。

```console
sudo rm -rf ~/.dotnet
```

## Alpine

**注意：** 如果您在執行從原始碼安裝的腳本時，已有一個不安裝在 `~/.dotnet` 的既有 dotnet，您將需要修改下方的第一個指令，以指向該自訂安裝位置。如果您想移除此腳本所安裝的特定 dotnet 版本並保留其他版本，可以依據[這些指示][uninstall-dotnet]進行。

```console
sudo rm -rf ~/.dotnet &&
sudo apk del icu-libs krb5-libs libgcc libintl libssl1.1 libstdc++ zlib which
bash coreutils gcompat git curl
```

[install-from-source]: ../src/linux/Packaging.Linux/install-from-source.sh
[uninstall-dotnet]: https://docs.microsoft.com/en-us/dotnet/core/install/remove-runtime-sdk-versions?pivots=os-linux#uninstall-net
