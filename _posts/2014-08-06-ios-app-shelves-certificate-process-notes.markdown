---
layout: post
title: 'iOS App Store 上架憑證流程筆記'
date: 2014-08-06 15:01:00
categories: ios Mobile
---

在開發幾個 App 之後對於上架簽署憑證之間的關係一直沒有搞的很清楚，所以做了一下筆記。

當 APP 開發完成之後要安裝到實機會需要在建置應用程式時(build)加上憑證，我們一般稱為應用程式憑證簽署(code signing)，白話文就是替應用程式綁上名牌以確保這個 APP 是合法的。
總結來說要完成一個可以上架的 APP 需要完成下面這些步驟：

1. 在Mac上產生憑證密鑰 `.certSigningRequest`
2. 是拿第一步所產生的密鑰到 `apple developer` 網站來產生密鑰憑證檔案 `.cer`
3. 將第二步的 `.cer` 裝到鑰匙圈中。
4. 回到 `apple developer` 申請產生iOS的憑證檔 `provisioning profile`。
5. 將裝置憑證檔案 `provisioning profile` 安裝到 `xcode` 。

# 概念說明
本文不是要做流程記錄如果想學習整個流程可以參考[教學](http://j.mp/1pdWyOr)
這篇文章是想針對各種憑證做些概念上的筆記
其實當你到 Apple Developer 的 `Certificates, Identifiers & Profiles` 界面時你可以觀察到全部大概就是下列這些東西

* `Certificates`
* `Identifiers`
* `Devices`
* `Provisioning Profiles`

我們先用 Development APP 建置的流程來概略說明一下關係，當你想要申請一張 Certificates 除了先購買開發者帳號外，第一步就是要先在鑰匙圈管理程式中產生`Certificate Signing Request (CSR)` 檔案，實際上他就是一對公私鑰。
![](http://i.imgur.com/1i62YdS.png)
接著在製作 Certificates 的時候 Apple 會跟你要這個 CSR ，最後完成這張憑證，下載回來後點擊就可以安裝。

當你完成上敘動作，此時 Xcode 就可以把 APP 安裝到裝置上了。當你連接裝置由於你已經有憑證了，Xcode 會自動幫你產生一個通用的 App ID (Xcode iOS Wildcard App ID)，幫你把裝置的 `UDID` 上傳到網站上，合成一個開發用的 Provisioning Profile (iOS Team Provisioning Profile) 並在安裝到裝置上，接著 APP 就可以部署到裝置上了。

看完流程你就可以歸納出整個流程是這樣的

1. Apple 跟你要了一組 CSR 來識別身份順便幫你產了一組 Certificates 驗明你這個開發者。
2. Identifiers 就像在註冊網址一樣，替 APP 產一組唯一的識別名稱，Apple 可以透過這個 ID 授權開放一些功能給這個 APP。
3. Devices 就是可用的裝置清單。
4. 最後你需要合出一個 `Provisioning Profile` 在你的 Build Settings 設定這張 Profile. 而這張 Profile 就是把上述的那些資訊全部整合在一起的一個檔案。最終會被打包進 ipa 中。
5. 如果是測試用的 APP 或使用 Testflight 你會需要再把這個 Profiile 安裝到機器上。



>我們可以合理的猜想第 5 步驟應該是 Apple 需要判斷這個安裝程式誰可以安裝，如果是在 App Store 上的 App 你需要登入 Apple ID 然後系統就可以註記這個 APP 現在歸屬于誰或者說誰可以安裝，這可以說明當你在一台裝置使用不同帳號安裝軟體後，當 User A 安裝了某軟體X，接著 User B 使用了自己的帳號，當某軟體X出更新的時候，彈出來的帳號預設是 User A 因為 Apple 知道這個應用程式是屬於誰的。但 Development 的 APP 並沒有，所以需要在機器本身再安裝一個 Profile 來判斷誰能夠安裝。您可以在 iPhone 的 設定 > 一般 > 描述檔中觀察到這些 Profile.



## Certificate
`Certificate` 憑證簡單說就是一個身份的證明，證明你是一個 Apple 開發者，然後才可以把 APP 安裝到機器上，當你對 APPLE 購買了開發者之後就可以取得，共分成兩大類 `Development` 和 `Production`
![](http://i.imgur.com/QL2aNR3.png)

Development 認證可以讓你部署 APP 到裝置上(上限100台)做測試，而 Product 則是用來發佈到 App Store。

## Development Certificate
  
  1. iOS App Development
  簽署測試用的APP以安裝在實機上。
    
  2. Apple Push Notification service SSL (Sandbox)
	當你的APP需要使用到Apple Push Notification功能時需要使用此種憑證，開啟沙箱功能。

## Production
Production 憑證又分成5種各針對不同需求的 APP

1. App Store and Ad Hoc 使用於提交到 App Store 或特殊用途的簽證如：TestFlight。
2. Apple Push Notification service SSL (Production) 當提交的應用程式需要使用 Apple Push Notification 功能的時候。
3. Pass Type ID Certificate 當APP需要更新Passbook資料的時候	
4. Website Push ID Certificate
5. VOIP Services Certificate
  
 
~~~~~
Ad hoc是拉丁文常用短語中的一個短語。這個短語的意思是「特設的、特定目的的（地）、即席的、臨時的、將就的、專案的」。這個短語通常用來形容一些特殊的、不能用於其它方面的的，為一個特定的問題、任務而專門設定的解決方案。這個詞彙須與a priori區分。
~~~~~

## App IDs
App ID 就是 APP 唯一的識別名稱，App ID 應該和 Xcode 中的 Bundle ID 一樣。App ID 分成兩種 

1. Explicit App ID：唯一的 App ID，用來針對單一 APP 命名，例如 com.apple.MyApp。
2. Wildcard App ID：類似使用正規式匹配符號的概念，指的是某些符合命名的 APP。例如 `*` 可表示所有 APP，`com.apple.*` 指的是開頭是 `com.apple` 的 APP。 App ID 可以設定相關 APP Services授權。例如，如果要使用 Apple Push Notification Services，則必須使用 Explicit App ID，以方便能識別唯一的應用程式。

## Provisioning Profile
簡單說 Provisioning Profile 就是包含 App ID, Certificates, Devices 列表資訊的一個檔案。
其用途就是用來確認這個 APP 是否合法，首先就是我們會需要 Certificate 來證明開發者是誰，然後 App ID 用來設定該 App 的授權的其他功能並驗證 Bundle ID 是否唯一。再來就是在測試的時候確定裝置是否能安裝。
所有這個 Profile 可以說是用來規範驗證這個 App 的檔案。

了解觀念之後就不需要死記簽證的流程了。



