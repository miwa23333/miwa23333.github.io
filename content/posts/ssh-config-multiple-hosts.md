+++
date = '2026-10-26T21:00:00+08:00'
draft = false
title = '用 ~/.ssh/config 管理多台主機：別名、跳板、金鑰與常用設定'
description = '每次 ssh 都要打完整的使用者名稱、IP 和 port 很煩。整理 ~/.ssh/config 的寫法：幫主機取別名、指定金鑰、透過跳板機連內網（ProxyJump）、保持連線不斷、把常用的 port forwarding 寫進設定檔，以及幾個常見的錯誤。'
categories = ['技術']
+++

之前寫過[用 SSH 遠端轉發讓外部存取本機服務](/posts/ssh-remote-port-forwarding/)和[把遠端主機當 proxy 瀏覽網頁](/posts/set-remote-host-as-proxy-for-chrome-through-ssh/)，兩篇的指令都蠻長的。管的主機一多，每次都要打使用者名稱、IP、port、金鑰路徑，很容易打錯。這些東西其實都可以寫進 `~/.ssh/config`，之後只要打一個別名。

## 最基本的寫法

```
Host work
    HostName 203.0.113.10
    User miwa23333
    Port 2222
    IdentityFile ~/.ssh/id_ed25519_work
```

存好之後，原本要打的

```bash
ssh -p 2222 -i ~/.ssh/id_ed25519_work miwa23333@203.0.113.10
```

就變成

```bash
ssh work
```

`scp`、`rsync`、`git` 用到 ssh 的地方也都認得這個別名，例如 `scp file.txt work:~/`。

幾個欄位的意思：

- `Host`：你自己取的別名，可以用萬用字元
- `HostName`：真正的位址，IP 或網域都可以
- `User`：登入的使用者名稱
- `Port`：預設 22，不是 22 才需要寫
- `IdentityFile`：用哪把私鑰

## 透過跳板機連內網：ProxyJump

公司或學校常常只有一台主機對外開放，其他機器都在內網，要先連到那台再連進去。以前要用 `ProxyCommand` 寫一串，現在用 `ProxyJump` 一行就好：

```
Host gateway
    HostName gateway.example.com
    User miwa23333

Host lab-*
    User miwa23333
    ProxyJump gateway

Host lab-01
    HostName 10.0.0.11

Host lab-02
    HostName 10.0.0.12
```

之後 `ssh lab-01` 會自動先連 gateway 再跳到 10.0.0.11。`Host lab-*` 那段是用萬用字元把共用的設定寫一次，所有 `lab-` 開頭的主機都會套用。

注意設定檔是**由上往下，第一個符合的值優先**，所以共用的萬用字元區塊要放在個別主機的後面或前面，取決於你想讓哪個優先。一般習慣是把個別主機放前面、萬用字元放後面。

## 連線一直斷：ServerAliveInterval

連線閒置幾分鐘就被防火牆踢掉，是最常見的抱怨。在設定檔最上面加一段對所有主機生效的設定：

```
Host *
    ServerAliveInterval 60
    ServerAliveCountMax 3
```

意思是每 60 秒送一個封包確認對方還在，連續 3 次沒回應才斷線。放在 `Host *` 裡就不用每台都寫。

## 把 port forwarding 寫進去

之前那兩篇的指令也可以寫成設定：

```
Host proxy
    HostName remote.example.com
    User miwa23333
    DynamicForward 3636

Host expose
    HostName example.com
    User miwa23333
    RemoteForward 3636 localhost:8000
```

`ssh proxy` 就會在本機開 SOCKS proxy，`ssh expose` 就會做遠端轉發。`LocalForward` 則是把遠端的 port 拉到本機，例如 `LocalForward 5432 localhost:5432` 可以連遠端的資料庫。

## 多把金鑰的時候

GitHub 一個帳號、公司一個帳號，兩把金鑰都在，ssh 可能會拿錯的那把去試，被對方拒絕。解法是指定金鑰並加上 `IdentitiesOnly yes`，告訴 ssh 只用這把、不要把 agent 裡的其他金鑰也拿去試：

```
Host github.com
    IdentityFile ~/.ssh/id_ed25519_personal
    IdentitiesOnly yes

Host github-work
    HostName github.com
    IdentityFile ~/.ssh/id_ed25519_work
    IdentitiesOnly yes
```

第二個別名的用法是把 remote 改成 `git@github-work:公司/repo.git`，ssh 會依別名找設定，實際還是連到 github.com。

## 常見錯誤

- **權限太開**：`~/.ssh/config` 和私鑰的權限如果其他人可讀，ssh 會直接拒絕使用。設 `chmod 600 ~/.ssh/config` 和私鑰，`chmod 700 ~/.ssh`
- **別名和真實主機名搞混**：`Host` 是別名，`HostName` 才是位址。只寫 `Host 10.0.0.11` 也可以用，但那樣就沒有取別名的意義
- **不知道最後套用了什麼**：用 `ssh -G work` 可以印出 ssh 對這個別名最後算出來的所有設定值，debug 很好用
- **改了設定沒生效**：ssh 每次連線都會重讀設定檔，不用重啟什麼；沒生效通常是上面「第一個符合優先」的順序問題

把主機都寫進設定檔之後，連線指令會變得很短，也比較不會因為打錯 IP 或 port 浪費時間。之後寫教學文的指令也可以短很多。
