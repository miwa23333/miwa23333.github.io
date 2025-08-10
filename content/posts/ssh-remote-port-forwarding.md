+++
date = '2025-01-28T09:12:38+08:00'
draft = true
title = 'SSH 遠端 port 轉發'
+++

有時候，我們會在本機電腦上啟動一個 HTTP 伺服器，例如進行測試或開發。但如果你沒有**公開 IP**，外部使用者就無法直接存取你的伺服器。

不過，如果你手上有一台**具有公開 IP 的 SSH 伺服器**，就可以利用 **SSH Remote Port Forwarding（遠端 port 轉發）**，讓外界透過該伺服器存取你的本機服務。

---

## SSH 遠端 port 轉發的語法

```bash
ssh -R [bind_address:]port:host:hostport destination
```

- **bind_address**：綁定的位址，可用 `*` 代表所有 interface。
- **port**：遠端 SSH 伺服器要開放的 port。
- **host** 與 **hostport**：要轉發到的主機與 port（通常是你的本機服務）。
- **destination**：SSH 連線的目標（你的具有公開 IP 的 SSH 伺服器）。

---

## 範例：本機 HTTP 伺服器

假設你在本機（localhost:8000）啟動了一個 HTTP 伺服器：

```bash
python -m http.server --bind localhost 8000
```

接著，你想讓外界透過 `example.com:3636` 連線到你的本機服務，可以執行：

```bash
ssh -R '*:3636:localhost:8000' miwa23333@example.com
```

---

## 可能遇到的問題：GatewayPorts 限制

根據 [man ssh(1)](https://man.archlinux.org/man/ssh.1) 說明：

> By default, TCP listening sockets on the server will be bound to the loopback interface only. This may be overridden by specifying a _bind_address_. An empty _bind_address_, or the address ‘`*`’, indicates that the remote socket should listen on all interfaces. Specifying a remote _bind_address_ will only succeed if the server's `GatewayPorts` option is enabled (see [sshd_config(5)](https://man.archlinux.org/man/sshd_config.5.en)).

預設情況下，SSH 伺服器上的 TCP 監聽 Socket 只會綁定到 loopback 介面（127.0.0.1）。

如果要讓它綁定到所有介面（`*`），必須在伺服器的 `sshd_config` 中啟用 `GatewayPorts` 選項。

也就是說，如果 SSH 伺服器沒有開啟 `GatewayPorts`，你的遠端 port 轉發可能只能在伺服器內部使用，外部訪客無法直接連線。

---

## 解法 1：修改 SSH 伺服器設定

如果你有權限修改 `sshd_config`，可以加入或修改以下設定：

```
GatewayPorts yes
```

然後重新啟動 SSH 服務。

---

## 解法 2：使用 socat 建立 relay

若你無法更改 SSH 伺服器設定，可以透過 [socat(1)](https://man.archlinux.org/man/socat.1.en) 建立一個 TCP relay，突破 `GatewayPorts` 限制。

### 步驟 1：本機 SSH 遠端轉發到伺服器內部 port

```bash
ssh -R 23333:localhost:8000 miwa23333@example.com
```

這會將本機的 `localhost:8000` 轉發到 SSH 伺服器的 `localhost:23333`。

### 步驟 2：在 SSH 伺服器上用 socat 建立對外 port

```bash
socat TCP-LISTEN:3636,fork,reuseaddr TCP:localhost:23333
```

這樣，外界訪客連線到 `example.com:3636` 時，會先進入伺服器透過 socat 轉發到 port `23333`，再透過 SSH 從 port `23333` 轉發到你的本機 HTTP 伺服器。

---

## 最終效果

現在，你在瀏覽器輸入 http://example.com:3636/ 就能存取你本機的 HTTP 伺服器了 🎉
