+++
date = '2025-09-13T13:42:15+08:00'
draft = false
title = '日本萬代一番賞 Google 地圖製作'
+++

抽一番賞想必是許多人去日本玩的時候的其中一個樂趣，但有時候搶手的系列可能需要跑好幾家店才能找到，於是最近就想了個方法透過一番賞官網的資料快速在 Google Map 上標好有販賣的店家。

照著以下教學做就可以生出特定一番賞系列及販賣地區下的 Google 地圖！

![ichiban-kuji-map](/images/ichiban-kuji-map.png)

## 1. 去官網看想要的商品

先去[萬代一番賞官網](https://1kuji.com/)看一下想要的一番賞

![ichiban-kuji-homepage](/images/ichiban-kuji-homepage.png)

點選網頁上方的 *店舖搜尋* 可以查詢一番賞商品在特定區域內哪些地方有販售


## 2. 生成地圖資料檔案

這步會需要執行已寫好的程式向一番賞官網爬取資料。
為了方便大家執行，已經把程式都放在 [Google Colab](https://colab.research.google.com/drive/1Zg_7i0mKNdnximbZtsTZOwJUl5fwwKHk?usp=sharing)。

### 1. 登入 Google 帳號並進入上面的 Colab 連結後先執行一次 *Run all*

![codelab-runall](/images/codelab-runall.png)

### 2. 勾選想要的商品及欲查詢地區

![codelab-product-selection](/images/codelab-product-selection.png)

![codelab-pref-selection](/images/codelab-pref-selection.png)

![codelab-region-selection](/images/codelab-region-selection.png)

### 3. 選擇完後再執行一次 *Run all* ，完成後點選綠色的下載鍵

![codelab-download](/images/codelab-download.png)

## 3. 將生成的地圖資料上傳到 Google Map

開啟 [Google My Maps](https://www.google.com/maps/d/u/0/) 後點選建立新地圖。

![google-my-map-create-new-map](/images/google-my-map-create-new-map.png)

點選*匯入*，把前一步生成的檔案上傳即可完成 ✨✨✨

![google-my-map-import-file](/images/google-my-map-import-file.png)

不過有個小提醒，現在 Google Map 好像無法一次匯入超過 2000 個地點，所以有超過的話可能要分次進行。

