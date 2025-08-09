+++
date = '2025-06-09T16:03:46+08:00'
draft = false
title = '透過 SSH 將遠端主機作為 Proxy 瀏覽網頁'
+++

在某些情況下，我們可能需要透過遠端主機的網路來瀏覽網頁。例如有些學校的課程網站可能會限制學校的 IP。

這時，一個常見且有效的方法就是利用 **SSH（Secure Shell）** 建立一個 [**SOCKS Proxy**](https://en.wikipedia.org/wiki/SOCKS)，然後讓你的 Chrome 瀏覽器透過這個 proxy 連線。

這個方法特別適用於當你需要存取受地理限制的服務，或是想在一個較為安全的環境下進行網路活動時。

---

## 步驟 1: 建立 SSH 代理

首先，你需要開啟終端機，並使用 `ssh -D` 指令來建立一個動態的 SOCKS proxy。

```bash
ssh -D 3636 user@remote_host_address
```

* `-D 3636`: 這部分的意思是在你的本地電腦上，開啟一個監聽在 port 3636 的 SOCKS 代理。你可以將 3636 替換成任何你喜歡的、未被佔用的 port number。

* `user@remote_host_address`: 這裡請替換成你的遠端主機使用者名稱和 IP 位址。

執行這個指令後，SSH 會建立一個加密的通道，所有傳輸到你本地 port 3636 的流量都會被加密並轉送到遠端主機。這時，你的終端機會保持在連線狀態，不要關閉它。

## 步驟 2: 設定 Chrome 瀏覽器

接下來，我們需要告訴 Chrome 瀏覽器使用這個 SOCKS proxy。

你可以在啟動 Chrome 時，在指令列中加入 --proxy-server 參數。

```bash
chromium --proxy-server="socks://localhost:3636"
```

* `chromium`: 在某些作業系統（例如 Linux）上，Chrome 的執行檔名稱可能是 `chromium`。在 `macOS` 上可能需要使用 `/Applications/Google\ Chrome.app/Contents/MacOS/Google\ Chrome` 或其他啟動指令。

* `--proxy-server="socks://localhost:3636"`: 這部分就是告訴 Chrome，將所有的流量都透過本地的 port3636  傳送到一個 SOCKS 代理伺服器。

執行這個指令後，一個新的 Chrome 視窗就會開啟。現在，這個瀏覽器視窗中的所有網路流量都會經過你遠端主機的網路進行傳輸。

---

小提醒：

請確保你的 SSH 連線保持開啟，一旦你關閉了終端機視窗，proxy 也會隨之失效。

這個方法只會影響透過這個特別啟動的 Chrome 視窗，並不會影響你電腦上其他的應用程式。
