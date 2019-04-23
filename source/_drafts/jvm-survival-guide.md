---
title: "[譯] JVM 生存指南"
categories: Program
tags: java
---

# 關於這份指南

當我們進入一個新平台時，我們常常試著從已知的經驗中找尋對照概念和框架，不過問題是因為他們擁有不同的名稱導致有時候我們反而花了更多時間。
本指南目標在協助新手解決問題。

# 目標群眾

本指南目標主要是 .NET 開發者，事實上從[原文網址](http://hadihariri.com/2013/12/29/jvm-minimal-survival-guide-for-the-dotnet-developer/)已經說明了這點，不過也適用於那些非 .NET 的新手。

# 基礎

## Java 的意思

Java 一詞意義上泛指著不同的 3 件事，一是程式語言（例如 C#），再者是指生態圈（例如 .NET Ecosystem），最後是指平台 - JVM（例如 CLR）。雖然有很多人不太喜歡 Java 這個語言，但其生態圈卻非常活躍時常有革命性的產品出現。事實上如果您是位 .NET 開發者，您大概聽過這 NHibernate、NUnit、NLog、NAnt 等等。這些都是像 Java 社群借鏡的。

## 支援多語言的通用語言執行平台

概念上在 Java 世界中， JVM 就對應到 .NET 的 CLR。它們都是虛擬機器，提供平台，可在上面執行多種語言。當然它們各自有不同的地方，不過都支援多種語言。舉例來說：CLR 支援 C#，VB.NET，F#。而在 JVM 上則支援 Java、Scala、Clojure、Ceylon、Groovy、Jruby、Kotlin 等等。

## JVM Bytecode

JVM Bytecode 就是支援 JVM 的語言最終編譯出來，給 JVM 執行用的東西。類似於 .NET 的 IL。

## 跨平台

JVM 跨平台支援，除了 Window，OSX，Linux 同時還可以在各種裝置上運行。

# JVM 實作與版本

JVM 有[多種實作](https://en.wikipedia.org/wiki/List_of_Java_virtual_machines)，最常見的是 Oracle 和 OpenJDK 的實作。甚至還有 .NET 實作的 JVM 版本 - IKVM.NET。

## Editions 和 Versions

這部分大概是本指南最複雜的一個段落。令人吃驚的是怎麼可能因為`版本`這麼簡單的事情把事情搞砸呢？甚至比起來微軟的產品命名還比較好懂。

> 關於標題的部分我們就不翻譯了，因為需要具體的理解它們各自表達的意思。下面就是各種 JVM 虛擬機器的實作版本。

就讓我們開始吧：

### Editions

* JRE - Java Runtime Environment 這個東西呢？是用來執行 JVM 應用程式。當我們不需要開發程式，我們只需要執行那些編譯好的結果時就使用 JRE。
* Java SE(SDK) - Java Standard Edition 也就是眾所週知的 JDK ，當我們需要開發 JVM 程式時，這就是我們至少需要的版本。
* Java EE - Java Enterprise Edition 這個名稱已經解釋了功能，當我們需要開發像是分散式、高擴展性應用程式時序呀使用它。裡面包含著 Java SE。
* Java ME - Java Micro Edition 這是一個相對小的子集，主要是用在行動裝置或小型裝置上。有點類似 .NET Micro Framework。
* JavaFx - Swing 的取代品，提供 Java 主要的 GUI toolkit。雖然有些爭議，但它同時也提供關於 RIA (Rich Internet Application) 的用途，在 HTML5 尚未出現之前，可以用它來執行一些舊版 HTML/CSS/JS 辦不到的事情。

到這邊我們可以推測，所有的 Java XY 也就是 JDK 的一種，就是我們需要開發。關於更多細節可以參考[發展說明](http://www.oracle.com/technetwork/java/javase/namechange-140185.html)。

### Versions

目前發佈的 Java 版本是 8。如果要檢查您所安裝的 Java 版本可以用指令

```bash
$ java -version
``` 

您會得到像下面的資訊

```
java version "1.8.0_131"
Java(TM) SE Runtime Environment (build 1.8.0_131-b11)
Java HotSpot(TM) 64-Bit Server VM (build 25.131-b11, mixed mode)
```

這就是 Java 8。我們把 `1.8.0` 前面的 `1` 拿掉就是 `8.0` 後面的 `0_131` 則是更新的版本。基本上 1.5 是 Java 5，1.6 指的是 Java 6，1.7是 Java 7。

# 安裝 Java

上面我們搞清楚版本之間的差異了，一旦確定我們的需求我們可以直接到 [Oracle 的安裝教學頁面](https://www.java.com/en/download/help/download_options.xml)。

# 應用程式

在 .NET 或其他系統中的原生應用程式，當我們完成編譯程式後，我們通常會得到一個執行檔或 DLL。不過在 Java 我們會在輸出目錄下得到一堆 `.class` 檔案。每一個 `.class` 檔案對應一個 Java 的 class 。這些 `.class` 就是 JVM Bytecode，類似於 CLR 中的 IL。

# JAR 檔案

如果要搬移上百隻的 `.class` 檔案我們可能會崩潰，這時可以建立一個 JAR 檔，其實一個 JAR 檔就是把這些 `.class` 打包起來變成一隻檔案。我們可以使用任何習慣的工具或者執行

```
$ jar cf jar-file input-file(s)
```

這個 `jar` 指令內建在 JDK 中。

# WAR 檔案

一個 WAR 其實就是網路應用程式的 JAR。WAR 檔一樣包含著一堆 `.class` 檔案和一些額外的給 Web Server 使用的 metadata 與目錄，例如：TomCat 這個 Server 需要使用 WAR 檔。

# 執行 Java 應用程式

任何一個 Java 應用程式都有一個進入點 `main class` 可以通過指令來執行：

```
$ java <class_containing_main_method>
```

> 我們需要在`輸出目錄`（即 `.class` 檔案的目錄）執行該指令。

## Classpath

當我們執行程式時，JVM 會在當前的目錄尋找所有相依的檔案，於是 `CLASSPATH` 這個環境變數可以用來設定一或多個包含 `.class` 或 JAR 的路徑。後續 Java 就會使用這個路徑。除了環境變數外也可以在執行指令時代入參數。
多個路徑可以用 `:` 分開。

```
$ java <class_main_method> -cp <class_path>
```

# 建置工具

在 .NET 中有許多建置工具包含 MS Build，NAnt，Albacore，Fake 等等。而 JVM 也不落人後。雖然有些支援的語言有自己專屬的建置工具例如：`clojure` 的 `Leiningen`，`Scala` 的 `SBT` 但大部分的語言都可以使用下面這些標準的建置工具（前兩者也可以使用）。

## Ant

Apache Ant 是一個將軟體編譯、測試、部署整合得自動化工具，預設使用一個 XML(build.xml)來設定自動化流程。NAnt 也是基於此概念。Ant 有點類似 MS-Build，也是透過 XML 來設定。

## Maven

Apache Maven 曾是非常受歡迎的工具，當您看到專案中有 `pom.xml` 時，那就是 Maven，不過它已經壞了。請參考 [Maven is broken by Design](http://blog.ltgt.net/maven-is-broken-by-design/)。Maven 也跟 Ant 一樣採用 XML 的方式，然而 Maven 不只是建置工具，它同時也是套件管理工具，像是 .NET 的 NuGet，Nodejs 的 npm。同時跟 NuGet 一樣您也可以自己維運自己的套件庫。[Artifactory](http://www.jfrog.com/home/v_artifactory_opensource_overview)這個工具可以協助我們完成這個任務。

## Gradle

Gradle 是升級版的 Maven，由於它的設定是基於 Groovy 這種語言，所以我們擺脫了使用 XML 的方式使用另一種更好的方式來管理相依套件。

## IntelliJ IDEA Build

IntelliJ IDEA 也提供了其擁有的建置系統，不過我們必須在 IntelliJ IDEA 整合工具的環境下才能使用。

# Frameworks 和函式庫

關於框架和函式庫的部分實在是太多了，這邊我們只列出一些常用或者推薦的。

## JSON Serializatin

* [Jackson](http://jackson.codehaus.org/)

## 單元測試

* [Junit](http://junit.org/) - 作為標準，許多工具都支援。
* [JBehave](http://jbehave.org/)
* [TEstNG](http://testng.org/doc/index.html) - 類似於 JUnit。

## Mocking Frameworks

* [Mockito](https://code.google.com/p/mockito/)

## Logging

* [SLF4J](http://www.slf4j.org/)

## IoC

* [Guice](https://code.google.com/p/google-guice/) - Google 提供，相當不錯。
* [Spring](http://spring.io/) - Spring Framework

## HTTP Client

* [Apache HTTP Client](http://hc.apache.org/httpclient-3.x/)

## Web 框架

大部分的網頁框架是基於 [Java Servlet API](http://en.wikipedia.org/wiki/Java_Servlet)，這些應用程式可以部署在 GlassFish，Jetty，TomCat 上。值得一提的是，Oracle 宣布不再繼續 GlassFish 社群的支援了且主要的推廣成員已經離開 Oracle 到 RedHat。現在他們提供了替代方案 - [WildFly](http://blog.arungupta.me/2013/12/webinar-wildfly-for-innovation-redhat-jboss-eap-for-commercial-support/)。
目前來說比較先進的框架是[Vert.x](http://vertx.io/)，它是基於[Netty](https://netty.io/)開發的。

## 網路

* [Netty](https://netty.io/) - 非同步事件驅動的框架，用於開發高效能的網頁應用程式。

## 其他

* [JodaTime](http://www.joda.org/joda-time/) - 在 Java 中日期和時間管理的部分異常難用，比 .NET 還糟糕，使用 JodaTime 是明智之舉。
* [NodaTime](https://code.google.com/p/noda-time/)
* [Reflections](https://code.google.com/p/reflections/) - 反射工具
* [Apache Commons](http://commons.apache.org/) - 系列常用的函式庫

# 慣例

這裡列出一些常見的慣例

## Namespace

命名空間通常是把網域名稱逆排

```
com.company_name.app
```

不幸的是有時候會變成目錄，例如

！[](http://hadihariri.com/images/jvm-guide-1.png)

不過好的 IDE 會幫我們處理這個問題。

## Property 和 Method 名稱

在 Java 通常使用 `lowerCamelCase` 來命名方法和屬性。

# 工具

當您安裝完 JDK 之後，您會得到編譯器 `javac`， jar 打包工具 `jar`，`javadoc` 等指令工具，有這些工具我們可以使用文字編輯器撰寫程式
然後用這些工具建置執行應用程式。

## IDE

一般來說我們不會直接使用文字編輯器進行開發，而是使用整合開發工具 IDE。主流的 Java 開發工具有

* NetBeans
* Eclipse
* IntelliJ IDEA

上述這些都是免費的 Open Source 軟體。

## 持續整合工具

* TeamCity
* Jenkins

## 其他工具

* JRebel - IntelliJ IDEA 套件，執行程式不需要重新編譯
* YourKit - Java Profilter

## IntelliJ IDEA

這邊有另外一份[IntelliJ IDEA 指南](http://hadihariri.com/2014/01/06/intellij-idea-minimal-survival-guide/)


# 參考

* [Java 生存指南](http://hadihariri.com/2013/12/29/jvm-minimal-survival-guide-for-the-dotnet-developer/)
* [IntelliJ IDEA 指南](http://hadihariri.com/2014/01/06/intellij-idea-minimal-survival-guide/)
