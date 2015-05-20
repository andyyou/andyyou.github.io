---
layout: post
title: 'IISExpress 使用 IP 連線'
date: 2015-05-14 05:30:00
categories: Net Windows
---

這個需求是我在 OSX VM 底下用 Windows 開發 Web 時希望從 OSX 這邊或者給內網的其他使用者快速連到 Visual Studio 的 Development Server 而產生的。

因為 IIS Express 預設是綁定只能透過 `localhost` 網域名稱來連線，所以當您透過內網 IP 連線時會出現 `HTTP Error 400. The request hostname is invalid.` 錯誤。

此時只需要如下步驟設定，我們假設您 on 起來的 server port 是 9999 實際設定時請改成您的 port 號

## 用管理者權限使用指令 

{% highlight bash %}
$ netsh http add urlacl url=http://*:9999/ user="NT AUTHORITY\INTERACTIVE"
{% endhighlight %}

## 修改 IISExpress 設定檔

在 `%USERPROFILE%\Documents\IISExpress\config\applicationhost.config` 檔案中找到對應網站的設定，把這個屬性換成如下 `bindingInformation="*:9999:*"`

{% highlight xml %}
<site name="..." id="...">
    <!-- application settings omitted for brevity -->
    <bindings>
        <binding protocol="http" bindingInformation="*:5555:*" />
    </bindings>
</site>
{% endhighlight %}

搞定！！