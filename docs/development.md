# 開發與除錯

首先複製此儲存庫：

```shell
git clone https://github.com/git-ecosystem/git-credential-manager
```

您還需要最新版的 .NET SDK，可以從 [.NET 網站][dotnet-web] 下載並安裝。

## 建構

`Git-Credential-Manager.sln` 解決方案可以在 Visual
Studio、Visual Studio for Mac、Visual Studio Code 或 JetBrains Rider 中開啟與建構。

### macOS

若要從 IDE 內部建構，請務必選取 `MacDebug` 或 `MacRelease` 解決方案組態。

若要從指令列建構，請執行：

```shell
dotnet build -c MacDebug
```

您可以在 `out/osx/Installer.Mac/pkg/Debug` 中找到安裝程式 .pkg 檔案的複本。

扁平二進位檔也可以在 `out/osx/Installer.Mac/pkg/Debug/payload` 中找到。

### Windows

若要從 IDE 內部建構，請務必選取 `WindowsDebug` 或
`WindowsRelease` 解決方案組態。

若要從指令列建構，請執行：

```powershell
dotnet build -c WindowsDebug
```

您可以在 `out\windows\Installer.Windows\bin\Debug\net472` 中找到安裝程式 .exe 檔案的複本。

扁平二進位檔也可以在 `out\windows\Payload.Windows\bin\Debug\net472\win-x86` 中找到。

### Linux

可用的兩種解決方案組態為 `LinuxDebug` 和 `LinuxRelease`。

若要從指令列建構，請執行：

```shell
dotnet build -c LinuxDebug
```

如果您想為特定架構建構，可以提供 `linux-x64`、`linux-arm64` 或 `linux-arm` 作為執行階段：

```shell
dotnet build -c LinuxDebug -r linux-arm64
```

您可以在 `out/linux/Packaging.Linux/deb/Debug` 中找到 Debian 套件 (.deb) 檔案的複本。

扁平二進位檔也可以在 `out/linux/Packaging.Linux/payload/Debug` 中找到。

## 除錯

若要從 IDE 內部除錯，您需要將 `Git-Credential-Manager` 設定為啟始專案，並指定 `get`、`store` 或 `erase` 其中之一作為程式引數。

若要模擬 Git 與 GCM 的互動，當您從選擇的 IDE 啟動時，您需要在[標準輸入][ioformat]中輸入以下資訊：

```text
protocol=http<LF>
host=<HOSTNAME><LF>
<LF>
<LF>
```

..其中 `<HOSTNAME>` 是支援的主機名稱，例如 `github.com`，而 `<LF>` 是換行符（或 CRLF，我們兩者都支援！）。

根據您的情境，您可能還需要包含以下可選欄位：

```text
username=<USERNAME><LF>
password=<PASSWORD><LF>
```

有關 Git 如何與憑證輔助程式互動的更多資訊，請閱讀 Git 關於[自訂輔助程式][custom-helpers]的文件。

### 附加到執行中的程序

如果您想對一個已在執行的 GCM 程序進行除錯，請設定 `GCM_DEBUG` 環境變數為 `1` 或 `true`。該程序將在啟動時等待除錯器附加後才會繼續執行。

這在除錯 GCM 與 Git 之間的互動時很有用，而且您希望由 Git 來啟動我們。

### 收集追蹤輸出

GCM 有兩個追蹤系統——一個是 GCM 專屬的，另一個則是實作了 [Git 的 Trace2 API][trace2] 的某些功能。以下是如何使用這兩種系統的說明。

#### `GCM_TRACE`

如果你想對 GCM 的發行版本或安裝進行除錯，可以設定 `GCM_TRACE` 環境變數為 `1`，將追蹤資訊印到標準錯誤，或設為一個絕對檔案路徑，將追蹤資訊寫入檔案。

例如：

```shell
$ GCM_TRACE=1 git-credential-manager version
> 18:47:56.526712 ...er/Application.cs:69 trace: [RunInternalAsync] Git Credential Manager version 2.0.124-beta+e1ebbe1517 (macOS, .NET 5.0) 'version'
> Git Credential Manager version 2.0.124-beta+e1ebbe1517 (macOS, .NET 5.0)
```

#### Git 的 Trace2 API

此 API 也可用於印出除錯、效能與遙測資訊到 stderr 或檔案中，並支援多種格式。

##### 支援的格式目標

1. 一般格式目標：類似於 `GCM_TRACE`，此目標會寫入人類可讀的輸出，最適合用於除錯。可透過環境變數或設定檔啟用，例如：

    ```shell
    export GIT_TRACE2=1
    ```

    或

    ```shell
    git config --global trace2.normalTarget ~/log.normal
    ```

0. 效能格式目標：此格式為欄位式，旨在分析開發與測試期間的效能。可透過環境變數或設定檔啟用，例如：

    ```shell
    export GIT_TRACE2_PERF=1
    ```

    或

    ```shell
    git config --global trace2.perfTarget ~/log.perf
    ```

0. 事件格式目標：此格式為 JSON 式，旨在收集大量資料以進行進階分析。可透過環境變數或設定檔啟用，例如：

    ```shell
    export GIT_TRACE2_EVENT=1
    ```

    或

    ```shell
    git config --global trace2.eventTarget ~/log.event
    ```

您可以在 Git 的 Trace2 API 文件中 [相對應的章節][trace2-targets] 閱讀關於各個格式目標的更多資訊。

##### 支援的事件

以下概括描述了目前在 GCM 中支援的 Trace2 API 事件，以及它們所提供的資訊：

1. `version`：包含目前可執行檔（例如 GCM 或某個輔助執行檔）的版本
0. `start`：包含目前可執行檔的 `Main()` 方法所收到的完整 argv
0. `exit`：包含目前可執行檔的離開碼
0. `child_start`：描述一個即將被生成的子程序
0. `child_exit`：描述一個子程序在離開時的狀態
0. `region_enter`：描述一個區域（例如，一段值得關注的程式碼的計時器）的進入事件
0. `region_leave`：描述離開一個區域時的狀態

您可以在 [對應的章節][trace2-events] 中，閱讀更多關於 Git Trace2 API 文件中各個事件的資訊。

想看到更多事件嗎？歡迎貢獻！我們 :love: 看到您的傑出貢獻，一同支援建構此 API。

### 程式碼覆蓋率指標

如果您想要程式碼覆蓋率指標，可以從指令列產生：

```shell
dotnet test --collect:"XPlat Code Coverage" --settings=./.code-coverage/coverlet.settings.xml
```

或透過 VSCode 終端機/執行工作：

```console
test with coverage
```

HTML 報告可透過 ReportGenerator 產生，它應該在建構過程中從指令列安裝：

```shell
dotnet ~/.nuget/packages/reportgenerator/*/*/net8.0/ReportGenerator.dll -reports:./**/TestResults/**/coverage.cobertura.xml -targetdir:./out/code-coverage
```

或

```shell
dotnet {$env:USERPROFILE}/.nuget/packages/reportgenerator/*/*/net8.0/ReportGenerator.dll -reports:./**/TestResults/**/coverage.cobertura.xml -targetdir:./out/code-coverage
```

或透過 VSCode 終端機/執行工作：

```console
report coverage - nix
```

或

```console
report coverage - win
```

## 文件語法檢查

文件是使用 [markdownlint][markdownlint] 進行語法檢查，它可以安裝為透過 NPM 安裝的 CLI 工具，或作為 [VSCode 中的擴充功能][vscode-markdownlint]。請參閱 GitHub 上的[文件][markdownlint]。用於 markdownlint 的設定檔位於 [.markdownlint.jsonc][markdownlint-config] 中。

文件使用 [lychee][lychee] 檢查連結有效性。Lychee 可以根據您的平台以多種方式安裝，請參閱 [GitHub 上的文件][lychee-docs]。根據 [lycheeignore][lycheeignore]，lychee 會忽略某些 URL。

[dotnet-web]: https://dotnet.microsoft.com/
[custom-helpers]: https://git-scm.com/docs/gitcredentials#_custom_helpers
[ioformat]: https://git-scm.com/docs/git-credential#IOFMT
[lychee]: https://lychee.cli.rs/
[lychee-docs]: https://github.com/lycheeverse/lychee
[lycheeignore]: ../.lycheeignore
[markdownlint]: https://github.com/DavidAnson/markdownlint-cli2
[markdownlint-config]: ../.markdownlint.jsonc
[trace2]: https://git-scm.com/docs/api-trace2
[trace2-events]: https://git-scm.com/docs/api-trace2#_event_specific_keyvalue_pairs
[trace2-targets]: https://git-scm.com/docs/api-trace2#_trace2_targets
[vscode-markdownlint]: https://github.com/DavidAnson/vscode-markdownlint
