# 企業組態預設值

Git Credential Manager (GCM) 可透過多種機制進行組態設定不同的機制。按優先順序排列，這些機制是：

1. [環境變數][environment]
1. 標準 [Git 組態][config] 檔案
   1. 儲存庫/本機組態 (`.git/config`)
   1. 使用者/全域組態 (`$HOME/.gitconfig` 或 `%HOME%\.gitconfig`)
   1. 安裝/系統組態 (`etc/gitconfig`)
1. 企業系統管理員預設值
1. 編譯的預設值

此模型大致上與 Git 本身支援的內容相符，也就是環境變數的優先順序高於 Git 組態檔案。

新增企業系統管理員預設值的功能，讓那些管理員能使用熟悉的 MDM 工具來設定許多 GCM 設定，而不必修改 Git 的安裝組態檔案。

## 使用者自由

我們相信使用者應_永遠_保有自由來設定 Git 和 GCM，完全依照其個人意願。透過將環境變數和 Git 組態檔案的優先順序設定在系統管理員值之上，這些值僅作為_預設值_使用者隨時能以慣用方式覆寫。

## Windows

預設設定值來自 Windows 登錄檔，具體而言是下列機碼：

### 32 位元 Windows

```text
HKEY_LOCAL_MACHINE\SOFTWARE\GitCredentialManager\Configuration
```

### 64 位元 Windows

```text
HKEY_LOCAL_MACHINE\SOFTWARE\WOW6432Node\GitCredentialManager\Configuration
```

> GCM 在 Windows 上是一個 32 位元的可執行檔。在 64 位元

Windows 上執行時，對登錄檔的存取會被自動重新導向至`WOW6432Node` 節點。

透過使用 Windows 登錄檔，系統管理員可以利用群組原則輕鬆地為 GCM 的設定設定預設值。

在此機碼下的所有設定名稱與可能值，都與 [Git 組態][config] 設定中的相同。

每個登錄機碼的類型可以是 `REG_SZ` (字串) 或 `REG_DWORD` (整數)。

## macOS

預設設定值來自 macOS 的偏好設定系統。組態描述檔可以透過相容的行動裝置管理 (MDM) 解決方案部署至裝置。

Git Credential Manager 的組態必須採用字典的形式，設定於網域 `git-credential-manager` 下的 `configuration` 機碼。例如：

```shell
defaults write git-credential-manager configuration -dict-add <key> <value>
```

..其中 `<key>` 是 [Git 組態][config] 參考文件中的設定名稱，而 `<value>` 是所需的值。

在 `configuration` 字典中的所有值都必須是字串。對於布林值，請使用 `true` 或 `false`；對於整數值，請使用字串形式的數字。

讀取目前的組態：

```console
$ defaults read git-credential-manager configuration
{
    <key1> = <value1>;
    ...
    <keyN> = <valueN>;
}
```

## Linux

預設組態設定存放區尚未實作。

[environment]: environment.md
[config]: configuration.md
