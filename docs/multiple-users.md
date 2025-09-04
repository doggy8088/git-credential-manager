# 多個使用者

如果您在單一 Git 託管服務上使用多個不同的身份，您可能會想知道 Git Credential Manager (GCM) 是否支援此工作流程。答案是肯定的，但由於其與 Git 的互通方式，會有一些複雜性。

---

**提示選擇帳號？**

請閱讀下方的 [**TL;DR** 章節][tldr]，快速了解如何讓 GCM 記住哪個儲存庫要使用哪個帳號。

---

## 基礎：Git 與 Git 託管服務

Git 本身沒有單一、明確的「使用者」概念。雖然有 `user.name` 和 `user.email` 會被嵌入到 commit 的標頭/結尾，但這些都只是任意字串。GCM 完全不與此使用者概念互動。您可以在 `user.*` 設定中填入任何內容，GCM 的行為完全不會改變。

除了 commit 中的使用者字串外，Git 還能辨識遠端 URL 或憑證中的「使用者」部分。不過，在主流 Git 託管服務的網頁介面中，這些部分預設並不常用。

Git 託管服務供應商（如 GitHub 或 Bitbucket）_確實_有「使用者」的概念。通常，這是一個像使用者名稱或電子郵件地址的身份，再加上密碼或其他憑證，以該使用者身份執行操作。您現在可能已經猜到，GCM（Git **Credential** Manager）正是與此使用者概念搭配運作的。

## 人、身份、憑證，我的天啊

您（一個自然人）可能在一個或多個 Git 託管服務供應商那裡擁有多個使用者帳號（身份）。由於大多數 Git 託管服務預設不會在其 URL 中加入「使用者」部分，因此 Git 會將遠端的使用者部分視為空字串。如果您在同一個網域上有多個身份，您需要自行為每個身份插入一個獨一的使用者部分。
在同一個網域上擁有多個身份是有充分理由的。您可能用一個 GitHub 身份處理個人工作，另一個處理開源專案，第三個則用於雇主的工作。您可以要求 Git 為同一個供應商託管的不同儲存庫指派不同的憑證。HTTPS URL 在網域名稱的 `@` 符號前包含一個可選的「名稱」部分，您可以用它來強制 Git 區分多個使用者。這部分很可能應該是您在 Git 託管服務上的使用者名稱，因為在某些情況下，GCM 會將其視為使用者名稱來使用。

## 進行設定

舉個例子，假設您正在處理託管於同一個網域名稱下的多個儲存庫。

| 儲存庫 URL | 身份 |
|----------|----------|
| `https://example.com/open-source/library.git` | `contrib123` |
| `https://example.com/more-open-source/app.git` | `contrib123` |
| `https://example.com/big-company/secret-repo.git` | `employee9999` |


當您複製這些儲存庫時，請在網域名稱前加上身份和 `@` 符號，以強制 Git 和 GCM 使用不同的身份。如果您已經複製了儲存庫，您可以更新遠端 URL 來包含該身份。

### 範例：新的複製
```shell
# 取代 `git clone https://example.com/open-source/library.git`，請執行：
git clone https://contrib123@example.com/open-source/library.git

# 取代 `git clone https://example.com/big-company/secret-repo.git`，請執行：
git clone https://employee9999@example.com/big-company/secret-repo.git
```



### 範例：現有的複製
```shell
# 在 `library` 儲存庫中，執行：
git remote set-url origin https://contrib123@example.com/open-source/library.git

# 在 `secret-repo` 儲存庫中，執行：
git remote set-url origin https://employee9999@example.com/big-company/secret-repo.git
```



## Azure DevOps

如果您正在使用 [Azure DevOps，您還應該注意它有一些額外的、可選的複雜性][azure-access-tokens]。

[azure-access-tokens]: azrepos-users-and-tokens.md

## GitHub

您可以使用 `github [list | login | logout]` 指令來管理您的 GitHub 帳號。這些指令記載於 [命令列用法][cli-usage] 文件中，或者您可以執行 `git credential-manager github --help` 來查看。

## TL;DR：讓 GCM 記住要使用哪個帳號

要為特定的遠端設定預設帳號，您只需設定以下 Git 組態：
```shell
git config --global credential.<URL>.username <USERNAME>
```



..其中 `<URL>` 是遠端 URL，而 `<USERNAME>` 是您希望設為預設的帳號。例如，對於 `github.com` 和使用者 `alice`：
```shell
git config --global credential.https://github.com.username alice
```



如果您希望為特定的儲存庫或遠端 URL 設定使用者，您可以將帳號名稱包含在遠端 URL 中。如果您使用 HTTPS 遠端，您可以將帳號名稱插入到網域名稱的 `@` 符號前。

例如，如果您希望 `mona/test` 這個 GitHub 儲存庫總是使用 `alice` 帳號，您可以在複製時使用 `alice` 帳號，執行：
```shell
git clone https://alice@github.com/mona/test
```



要更新現有的複製，您可以執行 `git remote set-url` 來更新 URL：
```shell
git remote set-url origin https://alice@github.com/mona/test
```

如果您的帳號名稱包含 `@`，請記得使用 `%40` 來逸出此字元：`https://alice%40contoso.com@example.com/test`。

[tldr]: #tldr-tell-gcm-to-remember-which-account-to-use
[cli-usage]: usage.md
