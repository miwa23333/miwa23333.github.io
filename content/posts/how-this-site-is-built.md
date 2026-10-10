+++
date = '2026-10-28T21:00:00+08:00'
draft = false
title = '這個網站怎麼架的：Hugo、PaperMod、GitHub Pages 與自訂網域'
description = '整理 miwa23333.xyz 的架站方式：用 Hugo 產生靜態網頁、PaperMod 主題、GitHub Actions 自動部署到 GitHub Pages，自訂網域要設哪些 DNS 記錄，以及怎麼把幾個獨立的工具網站掛在同一個網域底下。'
categories = ['技術']
+++

偶爾有人問這個網站是用什麼架的，這篇把整個流程寫下來，也當作自己的備忘。整體來說花的錢只有網域，主機和流量都是免費的。

## 架構一覽

- **Hugo**：靜態網站產生器，把 Markdown 文章轉成 HTML
- **PaperMod**：Hugo 的主題，負責外觀
- **GitHub**：放原始碼
- **GitHub Actions**：每次 push 就自動建置並部署
- **GitHub Pages**：負責提供網頁
- **自訂網域**：`www.miwa23333.xyz`，DNS 指向 GitHub Pages

文章就是 `content/posts/` 底下的一個個 Markdown 檔，開頭用 TOML 寫標題、日期、描述和分類，接著就是內文。寫完 push 到 GitHub，幾分鐘後網站就更新了。

## 為什麼選 Hugo

試過幾種做法之後選 Hugo 的原因很單純：

- 建置很快，幾十篇文章一秒內就產生完
- 單一執行檔，不用裝 Node 或 Python 的一堆套件
- 文章就是純文字檔，放在 git 裡，不會被綁在某個平台
- 主題選擇多，PaperMod 夠簡潔，也內建了搜尋、深色模式、文章目錄這些功能

缺點是模板語言有點特別，要改主題的東西需要查文件。不過一般寫文章不會碰到。

## 部署流程：GitHub Actions

Hugo 官方提供了一個現成的 GitHub Actions workflow，放在 `.github/workflows/hugo.yaml`。它做的事情是：

1. 在 GitHub 的機器上裝指定版本的 Hugo
2. 把 repo 連同主題（git submodule）抓下來
3. 執行 `hugo --gc --minify` 產生 `public/` 目錄
4. 把 `public/` 上傳並部署到 GitHub Pages

在 repo 的 Settings → Pages 裡把 Source 設成「GitHub Actions」就會用這個流程。之後每次 push 到 `main` 都會自動跑，不用自己在本機建置再上傳。

要注意的是 Hugo 版本要固定寫在 workflow 裡，不然哪天 GitHub 上的版本更新，主題可能就不相容了。這個網站目前釘在 0.148.0。

## 自訂網域要設什麼

GitHub Pages 預設的網址是 `帳號.github.io`。要換成自己的網域，需要兩邊都設定：

**在 DNS 那邊**，替 `www` 加一筆 CNAME 記錄指向 `帳號.github.io`。如果想讓不帶 `www` 的網域也能用，要替根網域加 A 記錄指向 GitHub Pages 的四個 IP（GitHub 文件有列），GitHub 會自動把它轉到 `www`。

**在 GitHub 那邊**，repo 的 Settings → Pages → Custom domain 填入 `www.miwa23333.xyz`，等 DNS 驗證通過後勾選 Enforce HTTPS，GitHub 會自動申請憑證。

Hugo 這邊則要把 `hugo.toml` 的 `baseURL` 改成新網域，不然產生出來的連結還是舊的。

## 把多個工具掛在同一個網域下

這個網站除了部落格之外，還有 [Dodorama](/dodorama/)、[動漫中文填字遊戲](/animanga-crossword/)、[動漫週邊圖鑑](/acg-goods/) 幾個獨立的小網站，它們各自是一個 repo，但網址都在 `www.miwa23333.xyz/` 底下。

這是 GitHub Pages 的一個特性：帳號的主 repo（`帳號.github.io`）設了自訂網域之後，同一個帳號底下其他 repo 開啟 Pages，網址會自動變成 `自訂網域/repo名稱/`。所以 `dodorama` 這個 repo 的 Pages 就是 `www.miwa23333.xyz/dodorama/`，完全不用額外設定。

好處是所有東西都在同一個網域，對搜尋引擎和使用者都比較一致。要注意的是每個 repo 的 Pages 可以指定從哪個分支部署，有時候部署用的分支不是 `main`，改東西之前要先確認。

## 本機預覽

寫文章的時候用 `hugo server` 可以在本機開一個即時預覽，存檔就會自動重新整理。要確認和正式環境一樣的輸出，用

```bash
hugo --gc --minify --baseURL https://www.miwa23333.xyz/
```

產生到 `public/` 再檢查。像是 `sitemap.xml`、`robots.txt`、每頁的 meta description 有沒有正確，都可以在這一步看。

## 費用

網域一年幾百塊台幣，其他全部免費。GitHub Pages 的限制是網站大小 1 GB 以內、每月流量 100 GB，個人部落格離這個上限非常遠。

整套東西架好之後，之後的工作就只剩寫文章和 push，這也是我一直沒換平台的原因。
