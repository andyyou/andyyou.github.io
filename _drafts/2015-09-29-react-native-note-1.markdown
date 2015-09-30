---
layout: post
title: 'React Native 教學 - 1'
date: 2015-09-29 06:30:00
categories: Javascript, React, Mobile
---

# 前言

這篇教學的目標是學習使用 React Native 加快開發 iOS 與 Android 應用程式的速度. 如果您想知道什麼是 React Native 以及為什麼 Facebook 創造了它您可以在[這篇 Blog 文章](https://code.facebook.com/posts/1014532261909640/react-native-bringing-modern-web-techniques-to-mobile/)找到答案.

我們假設您有過使用 React 開發應用程式的經驗

# 設定

React Native 需要一些基本的設定

1. 建議使用 Homebrew 來安裝 watchman 和 flow.
2. 必須要安裝 nodejs 4.0 以上
  * 使用 nvm 安裝 nodejs, 安裝前請注意您的機器上需要有 C++ 編譯器, nvm 並不支援 windows 您可以改用 `nvmw` 或 `nvm-windows`.
  * 使用 script 安裝 nvm  `curl -o- https://raw.githubusercontent.com/creationix/nvm/v0.27.1/install.sh | bash`
  * `nvm install && nvm alias default node` 會安裝最新版
  * `brew install watch`
  * `brew install flow` 

## iOS

需要安裝 Xcode 6.3+

## Android

安裝 Android SDK 包括 Android 模擬器
[詳細步驟](https://facebook.github.io/react-native/docs/android-setup.html)

1. 安裝最新版的 [JDK](http://www.oracle.com/technetwork/java/javase/downloads/jdk8-downloads-2133151.html)
2. OSX 直接使用 `brew install android-sdk` 安裝 SDK
3. 將 `export ANDROID_HOME=/usr/local/opt/android-sdk` 加入到 `~/.bashrc`, `~/.zshrc` 或其他您的 shell 使用的設定檔
4. 現在您可以在 terminal 輸入 `android;` 會開啟一個 SDK 各種工具的安裝管理視窗, 確認 `Android SDK Build-tools 23.01`, `Android 6.0(API 23)`, `Android Support Repository` 勾選並安裝
5. [設定 HAXM](http://developer.android.com/tools/devices/emulator.html#vm-mac)
  * `android;` 開起 Android SDK Manager 選取裡面 `Extras` > `Intel Hardware Accelerated Execution Manager` 安裝
  * 裝完之後到 `<sdk>/extras/intel/Hardware_Accelerated_Execution_Manager` 如果你照上面設定就是 `/usr/local/opt/android-sdk/extras/intel/Hardware_Accelerated_Execution_Manager` 找到 `IntelHAXM.dmg` 安裝
  * 執行 `kextstat | grep intel` 確認看到 `com.intel.kext.intelhaxm`
  * 啟動模擬器 `emulator -avd <avd_name>`
6. 執行 `android avd` 建立模擬器

在安裝了上面這些相依的工具之後只需要兩個簡單的指令就可以建立一個 React Native 專案

1. `npm install -g react-native-cli`
2. `react-native init [project_name]`

第一個指令用來安裝指令介面, 這個指令用來協助我們處理剩下的設定. cli 是透過 npm 來安裝的, 完成之後我們就可以使用 `react-native` 指令
接著第二的 `init` 指令用來建立 React native 專案, 運作的機制是擷取相關的原始碼和其他相依的東西最後建立出我們要的 React Native 專案

現在當你建立專案完成的同時你會看到, 這個專案

```
To run your app on iOS:
   Open /Users/andyyou/Projects/ios/test/ios/test.xcodeproj in Xcode
   Hit Run button
To run your app on Android:
   Have an Android emulator running, or a device connected
   cd /Users/andyyou/Projects/ios/test
   react-native run-android
```

# 開發