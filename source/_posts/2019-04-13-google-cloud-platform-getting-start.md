---
title: Google Cloud Platform - Compute Engine 入門
date: 2019-04-13 09:59:53
tags:
  - vm
categories: Cloud
---

本文僅筆記在 Google Cloud Platform 上搭配本地端 ssh 金鑰快速建立設定  Compute Engine。

<!--more-->

1. 您需要先註冊/登入 GCP

{% asset_img 1.png %}

2. 搜尋切換至 Compute Engine

{% asset_img 2.png %}
<br />
{% asset_img 3.png %}
<br>
{% asset_img 4.png %}

3. 設定名稱與選擇區域

{% asset_img 5.png %}

4. 本筆記需求是建立一台測試機，使用最小 CPU 即可。

{% asset_img 6.png %}

5. 選擇您需要的作業系統

{% asset_img 7.png %}
<br>
{% asset_img 8.png %}

6. 因為需安裝 HTTP 伺服器測試，請記得設定防火牆標記。

{% asset_img 9.png %}

7. 加入本地金鑰 `*.pub`

{% asset_img 10.png %}
