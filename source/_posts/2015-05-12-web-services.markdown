---
layout: post
title: 'XML 筆記'
date: 2015-05-14 05:10:00
categories: Program
tags: [xml]
---

# XML
先從 XML 說起，XML 被設計用來描述資料。
XML 看起來就像是 HTML，但他不是用來取代 HTML 的，HTML 設計的目的是用來呈現資料，而 XML 是紀錄資料。
XML 本身並不會完成任何事情，他就是一種資料的紀錄結構
<!--more-->

~~~xml
<note>
  <to>Tove</to>
  <from>Jani</from>
  <heading>Reminder</heading>
  <body>Don't forget me this weekend!</body>
</note>
~~~

XML 不像 HTML 預先定義好所有標籤，在 XML 中標籤是由開發者定義的。
XML 是一種簡單的資料分享格式。在真實世界中，電腦系統和資料庫儲存的資料使用不同的格式，彼此不相容。而 XML 透過純文字的格式儲存，提供一種跟不相依於任何軟硬體的方式儲存資料。

XML 從根節點出發，一份 XML 文件只會有一個 root
一個標準的 XML

~~~xml
<?xml version="1.0" encoding="UTF-8"?>
<note>
  <to>Tove</to>
  <from>Jani</from>
  <heading>Reminder</heading>
  <body>Don't forget me this weekend!</body>
</note>
~~~

每一個元素一定都要關閉，且元素名稱區分大小寫
巢狀時一定要符合規則，屬性一定要用 `"` 包起來。

一些特殊字元會有不一樣的意義，所以不能亂用
這些字元共有 5 個 `<` `>` `&` `'` `"` 但其中只有 `<` 和 `&` 是嚴格說起來被限制的。
空白會保留不會跟 HTML 一樣標籤中連續多個空白會被縮到剩一個

註解 `<!---->`

元素的命名規則
  * 區分大小寫
  * 開頭必須要是英文或底線
  * 不可以用 XML Xml 開頭
  * 雖然可以用 英文，數字 `-`, `_`, `.` 但是 `-` 在某些系統會被當成減號，`.`則會當成連接屬性和物件的符號
  * 名稱不能包含空白
  * 可以用各種語言的字元，但要考慮系統是否支援

雖然沒有規則限制屬性和子元素該怎麼用，不過大原則是：屬性是用來放跟資料本身沒直接關係卻跟標籤元素有關係的資訊，例如 id

設計時的原則是只要資料有機會擴展就用子元素，因為屬性只能單純的放文字

namespace 提供一種方式避免元素衝突，透過 `xmlns:ns="namespace URI"` 來定義，定義在起始的標籤。
namespace 的 uri 並不會用來解析資訊，目的只是要給 namespace 定義一個唯一的名稱

三組 namespace 的放法

~~~xml

<root>
<!--有設定 prefix 就要用 -->
<h:table xmlns:h="http://www.w3.org/TR/html4/">
  <h:tr>
    <h:td>Apples</h:td>
    <h:td>Bananas</h:td>
  </h:tr>
</h:table>

<f:table xmlns:f="http://www.w3schools.com/furniture">
  <f:name>African Coffee Table</f:name>
  <f:width>80</f:width>
  <f:length>120</f:length>
</f:table>

</root>

<!--集中在 root 定義-->

<root xmlns:h="http://www.w3.org/TR/html4/"
xmlns:f="http://www.w3schools.com/furniture">

<h:table>
  <h:tr>
    <h:td>Apples</h:td>
    <h:td>Bananas</h:td>
  </h:tr>
</h:table>

<f:table>
  <f:name>African Coffee Table</f:name>
  <f:width>80</f:width>
  <f:length>120</f:length>
</f:table>

</root>

<!--直接在開始的標籤設定，但不用 prefix -->

<table xmlns="http://www.w3.org/TR/html4/">
  <tr>
    <td>Apples</td>
    <td>Bananas</td>
  </tr>
</table>

<table xmlns="http://www.w3schools.com/furniture">
  <name>African Coffee Table</name>
  <width>80</width>
  <length>120</length>
</table>
~~~

因為 XML 支援不同的國際編碼，所以在一開始定義時要指定 `encoding`


XML 的第一行定義又稱 prolog 這是可選的，通常會在這邊定義版本和編碼。
預設是 UTF-8，XML 有提供 XML Validator 來驗證格式是否正確。
一個通過驗證的 XML 跟格式正確的 XML 文件並不完全相同
一個通過癌症的 XML 是格式正確加上符合文件類型定義的 XML 文件
我們會用 Document Type Definitions(DTD)或者 XML Schemas 來替 XML 定義何謂合法的元素與屬性的規則。

驗證 XML 的文件格式分成兩種類型

* DTD - 原生的文件類型定義
* XML Schema - 一種 XML 格式的文件用來替代 DTD

# DTD
文件型別定義(Document Type Definition)，DTD 的功能就是
定義該類型文件所包含的元素(Element)，並定義每個元素的內容，包含子元素、文字內容與屬性
規範各元素(Element)的排列組合方式，包含
出現的順序與可出現的次數。順序性與重複性。

如果在 XML 裡面宣告 DTD

~~~xml
<?xml version="1.0"?>
<!DOCTYPE note [
  <!ELEMENT note (to,from,heading,body)>
  <!ELEMENT to      (#PCDATA)>
  <!ELEMENT from    (#PCDATA)>
  <!ELEMENT heading (#PCDATA)>
  <!ELEMENT body    (#PCDATA)>
]>
<note>
  <to>George</to>
  <from>John</from>
  <heading>Reminder</heading>
  <body>Don't forget the meeting!</body>
</note>

<!--引用外部 dtd-->
<!DOCTYPE note SYSTEM "http:
//mydtds.com/note.dtd">

~~~

上面的 DTD 解釋如下
* `<!DOCTYPE note` 宣告這個 XML root 是 note
* 接著跟在 note 後面用陣列 [] 圈起來
* `<!ELEMENT note (to, form, heading, body)>` 宣告 note 至少要有這些東西，一個 `()` 是一個表示式
* `<!ELEMENT br EMPTY>` 定義 br 元素為空的
* `<!ELEMENT span ANY>` 定義 span 為任意元素
* `<!ELEMENT to (#PCDATA)>` 可被解析的文字資料
* `<!ELEMENT div (foo+)>` 定義 div 裡面至少要有一個 foo 或多個
* + 一個或多哥
* * 零或多
* ? 零或一

簡單介紹完 DTD 結論就是這會用在一種情況當開發者分別處於不同團隊或公司要互相交換分享資料時用來確保雙方給的 XML 是否正確。因為 W3C 明確的定義當解析 XML 出現異常時必須終止，不能像 HTML 一樣容錯。

# XML Schema
一個 XML Schema 描述一個 XML 的結構功能就像 DTD 一樣
一個 XML 符合基本結構規範稱為 `Well formed`
一個 XML 針對 XML Schema 規則檢查後才能保證是 `Well Formed` 和 `Valid`


XML Schema 使用 XML 的格式實作 DTD 的功能

~~~xml
<xs:element name="note">
  <xs:complexType>
    <xs:sequence>
      <xs:element name="to" type="xs:string" />
      <xs:element name="from" type="xs:string" />
      <xs:element name="heading" type="xs:string" />
      <xs:element name="body" type="xs:string" />
    </xs:sequence>
  </xs:complexType>
</xs:element>
~~~

上面這段 XML Schema 解釋
* `<xs:element name="note">` 定義一個叫做 note 的元素
* `<xs:complexType>` 在 note 裡面定義 note 是一個複雜型別
* `<xs:sequence>` 設定這個 complexType 是一個元素序列
* `<xs:element name="to" type="xs:string">` 定義元素序列裡面有一個 to 元素型別是字串

XML Schema 比起 DTD 功能更強大，可以用 XML 語法結構定義比較清楚，支援型別限制也支援 namespace
透過 XML Schema 可以對 XML 檔案附加其他描述的資訊
同時可以驗證資料。

因為 XML Schema 也是一種 XML 所以也可以使用 XSLT


# XMLHttpRequest Object
XMLHttpRequest 物件被用來和伺服器交換資料
您可以透過 XMLHttpRequest 執行

* 更新網頁而不需要重新載入整個頁面
* 在頁面載入完成後對伺服器請求資料
* 從伺服器取得其他資料
* 在背後送資料給伺服器

## 如何建立一個 XMLHttpRequest 物件

所有新的瀏覽器都支援此物件

~~~js
var xmlhttp = new XMLHttpRequest();
~~~

# XML Parser
所有新的瀏覽器都支援一個內建的 XML 解析器
XML parser 可以轉換一個 XML 文件檔案成為一個 XML DOM 物件，意思是之後可以透過 JS 操作
下面的簡易程式碼示範如何轉換

~~~js
xmlhttp = new XMLHttpRequest();
xmlhttp.open("GET", "books.xml", false); //open(method,url,async)
xmlhttp.send();
xmlDoc = xmlhttp.responseXML;

txt = "<bookstore><book><title>Hello Italian</title><author>ANDY</author></book></bookstore>";

if (window.DOMParser) {
  parser = new DOMParser();
  xmlDoc = parser.parserFromString(txt, "text/xml");
} else {
  xmlDoc = new ActiveXObject("Microsoft.XMLDOM");
  xmlDoc.async = false;
  xmlDoc.loadXML(txt);
}
~~~

# XPath
XPath 是一種用來找到 XML 資訊的語法
XPath 是根據 XML 需求而定義的局部語法
XPath 使用路徑表示式來搜尋 XML 中的資訊
XPath 包含著一組標準的函式庫
XPath 是 XSLT 的主要元素之一
XPath 也可以用在 XQuery XPointer 和 XLink

XPath 使用路徑表示式來選取某個 XML 中的節點 Node 或符合規則的節點集合。這個表示式跟我們在電腦系統上面找檔案的路徑格式非常相似。目前 XPath 表示式可以被用在 Javascript, Java, XML Schema, PHP, Python, C, C++ 等等

XPaht 是 XSL 標準中的主要內容之一，如果沒有 XPath XSL 將無法建立 XSLT

## XPath 表達式
* `bookstore/book[1]` 從 bookstore 子元素中選取第一個 book 元素，注意從 1 開始而不是 0
* `/bookstore/book[last()]` 選最後一個
* `/bookstore/book[last()-1]` 倒數第 2 個
* `/bookstore/book[position() < 3]` 前兩個
* `//title[@lang]` 相對路徑比對到 title 標籤且有 attribute 名稱是 lang 的。注意在 [] 裡面通常指的是子元素的規則，加上 `@` 就是元素本身的 attributes

# XLink
XLink 定義一種方式讓我們可以在 XML 文件中建立連結
XLink 被用來在 XML 中建立超連結
任何在 XML 中的元素都可以具有超連結的行為
XLink 支援單純連結(類似 HTML) 以及擴展的連結(將多個資源連在一起)
XLink 可以被定義在被聯結的外部檔案中

在 HTML 裡面 `<a>` 元素可以定義一個超連結，然後並沒有這樣的東西存在 XML 中，因為在 XML 是您定義元素。因此您不可能叫瀏覽器預測 XML 文件中什麼元素會有超連結的行為
下面是 XLink 的範例

~~~xml
<?xml version="1.0" encoding="UTF-8"?>
<homepages xmlns:xlink="http://www.w3.org/1999/xlink">
  <homepage xlink:type="simple" xlink:href="http://www.w3schools.com">Visit W3Schools</homepage>
  <homepage xlink:type="simple" xlink:href="http://www.w3.org">Visit w3c</homepage>
</homepages>
~~~

為了能夠取得 XLink 的功能我們必須宣告 XLink 命名空間。
而這個 namespace 是 `http://www.w3.org/1999/xlink`

# XML 相關技術整理
* XML Extensible Markup Language一種簡單的資料結構格式類似 HTML
* XSL Extensible Style sheet Language XSL 由三個部分組成：
  - XSLT  用來轉換 XML 的語言，用來把一個 XML 轉換成另外一種格式如 XHTML 的樣式結構與法
  - XPath 在 XML 中用來巡覽的語言，類似 CSS 中的 Selector
  - XSL-FO 格式化 XML 的語言
* XSLT 一種 XSL 用來把一個 XML 轉換成另外一種格式如 XHTML 的樣式結構與法
* XML Schema 用 XML 取代 DTD 的功能
* XLink 建立超連結的行為
* XPointer 類似 HTML 中的錨點

# CDATA
CDATA 在 XML 解析的時候會被忽略
PCDATA 格式的文字會被解析
會區分 PCDATA 的原因是因為元素裡面還能夠包含子元素
不管是元素或者是元素內一般的內容文字都被歸類為 PCDATA

而 CDATA 就是不被解析的文字部分
`<` 和 `&` 在 XML 元素中都是非法字元
`<` 字元被認定為非法字元的原因是因為解析器會認為這是一個元素的開始
`&` 不行則是因為這是一個 character entity 就是例如 `&amp;` 或 `&gt;` 這種格式編碼的開頭

一些文字像是 Javascript 包含很多的 `<` `&` 字元，為了避免錯誤 script 可以被定義成 CDATA 這樣一來就會被忽略，任何在 CDATA 段落裡面的只會被 XML Parser 忽略。

問題：您可以能想說那 CDATA 跟一般註解有什麼差別？
* CDATA 仍然是文件的一部分，而註解不是。意思是告訴 Parser 就不要幫我解析了直接把全部的東西當作內容輸出
在某些時候內容中含有 HTML 標籤或者是一些特殊字元﹙如﹕<、>、&﹚，當這些字元出現在內容裡，通常都會出現 XML 分析錯誤的情況，這時候就必須將這些字元作些轉換的工作（如︰< / &lt;、> / &gt;、& / &amp;）。

其實並不需要如此，CDATA 區段提供了一種通知剖析器的方法，說明 CDATA 區段所包含的字元沒有標記。

當 XML 剖析器遇到開頭的『<![CDATA[』，會將接下來的內容報告成字元，而不會嘗試將其解譯成項目或實體標籤。字元參考不能在 CDATA 區段內運作。當它遇到結尾的『]]>』時，剖析器會停止報告並回到正常的剖析

定義 CDATA
`<![CDATA[ 資料在這邊 像是 < 這時就合法了]]>` 注意裡面不可以再放 `]]>`

# XSL
XSL 代表的是 Extensible Stylesheet Language
W3C 發展 XSL 主要是因為有 XML Stylesheet Language 的需求。

## HTML 的樣式表是 CSS
HTML 使用預先定義好的標籤，而這些標籤的意義在使用前就知道用途
`<table>` 標籤在 HTML 裡面定義一個表格，接著瀏覽器就知道該怎麼呈現他。
特元素加入樣式在 HTML 非常簡單，只需要在 CSS 中告訴瀏覽器哪個元素該使用什麼背景色或字體即可。

## XML 的樣式表是 XSL
XML 並沒有預先定義好的標籤，因此我們並不知道每個標籤是在做什麼的。
一個 `<table>` 標籤可能跟在 HTML 一樣或者代表一個傢俱，或其他的東西，瀏覽器並不知道該怎麼呈現
XSL 會描述該如何呈現 XML

## XSL 不只是單純的樣式表語言
XSL 由三個部分組成：
  - XSLT  用來轉換 XML 的語言
  - XPath 在 XML 中用來巡覽的語言，類似 CSS 中的 Selector
  - XSL-FO 格式化 XML 的語言

XSLT 是用來把 XML 文件轉成 XHTML 文件或者其他 XML 文件的語言。

過程是 `XML` -> `XSLT` -> `XHTML(another xml doc)`
並且大部份的瀏覽器都支援。

撰寫 XSL 一開始需要定義格式

~~~xml
<xsl:stylesheet version="1.0"
xmlns:xsl="http://www.w3.org/1999/XSL/Transform">

<xsl:transform version="1.0" xmlns:xsl="http://www.w3.org/1999/XSL/Transform">
~~~

為了能夠使用 XSLT 的元素，屬性和功能必須要先定義 XSLT 的 namespace 在文件最上面。

`xmlns:xsl="http://www.w3.org/1999/XSL/Transform"` 指向 W3C XSLT 的命名空間，如果您使用這個命名空間那就必須要加入 `version="1.0"` 的屬性

就跟 HTML 搭配 CSS 一樣，一個 XML 可以引入一個 XSL
透過 `<?xml-stylesheet type="text/xsl" href="cdcatalog.xsl"?>`

一個 XSL 樣式由一個或一些規則組成，這些規則稱之為樣板 `<xsl:template>` ，首先我們透過樣板來建立規則，然後 `<xsl:template>` 標籤會有一個屬性 `match` ，我們用 `match` 來關聯到 xml 的 element。這個 `match` 可以關聯定義整個 XML，而這個 `match` 的值是用 `XPath` 表示式，例如 `/` 就是根，指的是整份文件

我們可以想成一個 template 從 match 的元素取資料並套用到裡面的樣板。

因為 XSL 也是一種 XML 所以第一行宣告還是 XML
接下來會用 `<xsl:stylesheet>` 定義這份文件是一個 XSLT 當然要加上 xmlns 和 version

瀏覽器執行時起始點是 XML 然後 XML 載入 XSL ，XSL 取得 XML 中的資料並且呈現。

`<xsl:value-of>` 標籤是用來取得 XML 資料用的。透過 `select` 屬性來取得 XML Tag 中的資料

~~~xml
<?xml version="1.0" encoding="UTF-8"?>
<xsl:stylesheet version="1.0"
xmlns:xsl="http://www.w3.org/1999/XSL/Transform">

<xsl:template match="/">
  <html>
  <body>
  <h2>My CD Collection</h2>
  <table border="1">
    <tr bgcolor="#9acd32">
      <th>Title</th>
      <th>Artist</th>
    </tr>
    <tr>
      <td><xsl:value-of select="catalog/cd/title"/></td>
      <td><xsl:value-of select="catalog/cd/artist"/></td>
    </tr>
  </table>
  </body>
  </html>
</xsl:template>

</xsl:stylesheet>
~~~

`select` 屬性的值一樣是使用 XPath Expression，XPath 運作的原理類似檔案系統路徑，從 root `/` 開始透過斜線一層一層對應

`<xsl:value-of>` 只會取得單一標籤，如果要迭代或者選取一整組則用 `<xsl:for-each>` 使用 `for-each` 可以把符合 `select` 規則的元素全部都取得然後全部一個一個套用內部的樣式。

另外要注意的是 `select` 的 XPath Expression 運作模式既然類似檔案系統，也就是說你可以使用相對路徑或者絕對路徑的方式。父元素已經取得的標籤，如果要繼續往內部取就像上面例子一樣就可以

~~~xml
<xsl:for-each select="catalog/cd">
  <xsl:value-of select="title" />
</xsl:for-each>
~~~

除了選取之後 XPath Express 也可以像 CSS Selector 過濾一樣加上屬性過濾例如 `<xsl:for-each select="catalog/cd[artist='andy']">`
但判斷邏輯就不像 CSS 這麼多種，只有4種，而且要注意 [] 並不是同層元素的屬性，而是子元素的值。
如果要指定屬性則是
`<xsl:for-each select="Factory/Car[@price>=1100]">` 完整範例參考下面

  * =
  * !=
  * &lt;
  * &gt;

~~~xml
<!-- XML -->
<?xml version="1.0" encoding="UTF-8"?>
<?xml-stylesheet type="text/xsl" href="car.xsl"?>
<Factory>
  <Car price="1000">
    <Brand>Honda</Brand>
    <Type>Civic</Type>
    <Displacement>2000</Displacement>
  </Car>
  <Car price="2000">
    <Brand>Toyota</Brand>
    <Type>Viso</Type>
    <Displacement>1800</Displacement>
  </Car>
</Factory>

<!-- XSL -->
<xsl:stylesheet version="1.0" xmlns:xsl="http://www.w3.org/1999/XSL/Transform">

  <xsl:template match="/">
    <html>
      <body>
        <h2>CAR</h2>
        <ul>
          <xsl:for-each select="Factory/Car[@price>=1100]">
            <li><xsl:value-of select="Brand" /></li>
          </xsl:for-each>

        </ul>
      </body>
    </html>
  </xsl:template>
</xsl:stylesheet>
~~~

如果要排序則使用 `<xsl:sort>` 元素，如下用法

~~~xml
<xsl:for-each select="catalog/cd">
  <xsl:sort select="artist"/>
  <tr>
    <td><xsl:value-of select="title"/></td>
    <td><xsl:value-of select="artist"/></td>
  </tr>
</xsl:for-each>
~~~

`<xsl:if>` 跟 filter 有點類似，但是是針對內部的元素

~~~xml
<xsl:for-each select="catalog/cd">
  <xsl:if test="price &gt; 10">
    <tr>
      <td><xsl:value-of select="title"/></td>
      <td><xsl:value-of select="artist"/></td>
      <td><xsl:value-of select="price"/></td>
    </tr>
  </xsl:if>
</xsl:for-each>
~~~


`<xsl:choose>` 類似其他語言中的 switch

~~~xml

<xsl:choose>
  <xsl:when test="expression">
    ... some output ...
  </xsl:when>
  <xsl:otherwise>
    ... some output ....
  </xsl:otherwise>
</xsl:choose>
~~~


# Web Services

Web Services 是網頁應用程式元件。
Web Services 可以被網站發佈，搜尋與使用。
這份筆記會說明 WSDL, SOAP, UDDI, RDF

WSDL 意思是 Web Services Description Language
WSDL 是一種基於 XML 的語言用來描述 Web Services

SOAP 意思為 Simple Object Access Protocol
SOAP 是一種基於 XML 存取 Web Services 的協議
SOPA 架構在 XML 之上

UDDI 代表的是 Universal Description Discovery and Integration 統一描述、發現和集成
UDDI 是一個目錄服務讓公司可以搜尋 Web Service
UDDI 被描述在 WSDL 裡面
UDDI 透過 SOAP 溝通

RDF 代表的是 Resource Description Framework
RDF 是一個框架，目的是在 Web 上描述資源
RDF 一樣透過 XML 來撰寫

Web Services 是應用程式的組件
Web Services 透過開放式協定
Web Services 可以透過 UDDI 來找到
Web Services 可以被其他應用程式使用
HTTP 和 XML 是組成 Web Services 的基礎

因為所有主流平台可以透過瀏覽器存取網頁，但不同平台無法互相溝通。
為了使這些平台能夠一起工作，而發展了網頁應用程式。
網頁應用程式可以簡單地在網頁上執行，而這些都圍繞在瀏覽器標準上且可以被用在任何平台的任何瀏覽器上

Web Services 帶領網頁應用程式到了另一個境界
透過使用 Web Services 您的應用程式可以發布自己的函式或訊息給其他人使用
Web Service 使用 XML 來編譯，解譯資料。然後用 SOAP 來傳輸。
透過 Web Services 您部門的 Win 2k 的帳單系統可以和 IT 部門的 Unix Server 溝通

Web Services 有兩種使用方式
* 可重複使用的應用程序的組件
* 連結已存在的軟體

# Postman 基本操作
Postman 介面被區分成兩個區塊，左邊的 sidebar 和右邊的 request builder。request builder 可以讓我們建立幾乎所有種類的 request。HTTP request 分成 4 個部分 URL, method, header, body

## URL
URL 是我們首先要設定的東西，對某個網址發送請求，這個欄位有 autocomplete 的功能會記錄之前發送的紀錄
URL 欄位旁邊讓我們可以選擇 method(GET, POST...)，接著點擊URL params 按鈕可以在下方開啟參數表單
這些參數通常會直接帶到網址列(GET的時候)，這些參數並不會自動 `URL-encoded` 對著欄位點右鍵可以 `EncodeURIComponent`
您可以分開輸入每一組 key/value 然後 Postman 會自動整合。
如果您把這些參數直接帶入 URL 欄位 Postman 會自動幫你切割帶入 params

## Headers
點擊 headers 按鈕可以顯示另外一個鍵值編輯器，您可以設定 header 資料。常用共通的 HTTP 規格的 header 屬性都可以透過 autocomplete 來找到，另外像是 Content-Type 的值也有列表。

## 被限制的 headers 和 cookies
很不幸的一些 headers 被 Chrome 和 XMLHttpRequest 規格所限制，下面這些 headers 是被禁止發送的

  _ Accept-Charset
  _ Accept-Encoding
  _ Access-Control-Request-Headers
  _ Access-Control-Request-Method
  _ Connection
  _ Content-Length
  _ Cookie
  _ Cookie 2
  _ Content-Transfer-Encoding
  _ Date
  _ Expect
  _ Host
  _ Keep-Alive
  _ Origin
  _ Referer
  _ TE
  _ Trailer
  _ Transfer-Encoding
  _ Upgrade
  _ User-Agent
  _ Via

新版的 Postman 可以安裝 Postman interceptor 就可以使用這些禁止的 headers

## Cookies
當您使用新版的 packaged postman app 他會執行在一個沙箱裡跟瀏覽器是分開的，如此是不能存取瀏覽器中的 cookie，這個限制仍然可以透過 interceptor 來克服。Postman 透過 interceptor 掌管了所有 request 的路由資訊。透過在 Header 中加入 "Cookie" 然後在 value 欄位使用 `name=123` 可以設定 cookie 或者讀取

## Request body 和 parameter
當我們要建構一個 request 的時候通常都需要處理 request body(就是 form post 出去的那些資料)，Postman 讓我們幾乎可以發送任何種類的 HTTP Request，body 編輯區分成 4 個區塊

## form-data
這種格式採用 `multipart/form-data` Content-Type 預設會編碼一個 web form 用來傳輸資料。這就類似在網頁上的 form 輸入完資料送出，不過根據 W3C 規範 form post 預設是採用 `x-www-form-urlencoded`

[multipart/form-data application/x-www-form-urlencoded  差異](http://www.w3.org/TR/html401/interact/forms.html#h-17.13)
[第二篇](http://stackoverflow.com/questions/4007969/application-x-www-form-urlencoded-or-multipart-form-data)

# 使用 Postman 發送 SOAP
[教學](http://blog.getpostman.com/2014/08/22/making-soap-requests-using-postman/)

# WSDL
WSDL 代表的是 Web Services Description Language
WSDL 使用 XML 寫成
WSDL 就是一份 XML 文件
WSDL 用來描述 Web Service
WSDL 也被用來找尋對應的 Web Services
WSDL 就是一份 XML 文件用來描述某個 Web Service 的 XML 文件，這份文件會紀錄 Web Service 的位置和 Method 怎麼操作，Service 會透露什麼訊息

## WSDL 的文件結構

~~~xml
<definitions>
  <types>
    Web services 資料格式定義 一個容器
  </types>
  <message>
  用來溝通的資料定義
  </message>

  <portType>
    支援的操作
  </portType>

  <binding>
    針對特定 portType 的協定和資料格式
  </binding>
</definitions>
~~~

一份 WSDL 文件可以包含其他元素，例如擴展元素和 Service 元素，讓其可以把一些 WebService 的定義彙整到一份 WSDL

## WSDL Ports
`<portType>` 元素幾乎是最重要的元素，用來描述一個 Web Service 關於其可以被執行的操作及包含的訊息，用程式語言來比喻一個 `<portType>` 元素可以被當成一個 function 函式庫或者一個 class

## WSDL Message
`<message>` 定義關於一個操作的資料元素，每一個 message 可以由一個或多個 parts 組成，parts 可以想成是參數

## WSDL Types
`<types>` 元素用來定義資料類型，給 Web Service 使用。

## WSDL Bindings
`<binding>` 元素用來給不同的 `prot` 定義資料格式

## 簡易的範例

~~~xml
<message name="getTermRequest">
  <part name="term" type="xs:string" />
</message>

<message name="getTermResponse">
  <part name="value" type="xs:string"/>
</message>

<portType name="glossaryTerms">
  <operation name="getTerm">
    <input message="getTermRequest" />
    <output message="getTermResponse" />
  </operation>
</portType>
~~~

`<portType>` 元素幾乎是 WSDL 最重要的元素，一個 portType 定義一個 Web Service 有哪些操作可以被執行，還有對應的 message。
`<portType>` 定義 Web Service 的一個連接點，
portType 就是一個函式庫然後呼叫的時候有一個對應執行 function
return 回傳值的時候有另外一個 function 就是 message
message 裡面的 part 就是參數

`<operations>` 又區分四種類型
- 只收資料
- 收資料，回傳資料
- 主動發送 request 然後等待 response
- 只送資料

~~~xml
<!-- 只收資料範例 -->
<message name="newTermValues">
  <part name="term" type="xs:string"/>
  <part name="value" type="xs:string"/>
</message>

<portType name="glossaryTerms">
  <operation name="setTerm"> <!--實際 call 的 method name -->
    <input name="newTerm" message="newTermValues" /> <!-- 其實只負責關聯函式的參數部分 message 很重要，name 不重要 ，另一個角度思考就是一個要傳入或 return 的 object 格式 - 訊息格式。-->
  </operation>
</portType>

<!-- 一般發收資料 -->
<message name="getTermRequest">
  <part name="term" type="xs:string" />
</message>

<message name="getTermResponse">
  <part name="value" type="xs:string" />
</message>

<portType name="glossaryTerms">
  <operation name="getTerm">
    <input message="getTermRequest" />
    <input message="getTermResponse" />
  </operation>
</portType>
~~~

WSDL bindings 定義一個 Web Service 的 message 格式和協定的細節
例如

~~~xml
<message name="getTermRequest">
  <part name="term" type="xs:string" />
</message>

<message name="getTermResponse">
  <part name="value" type="xs:string" />
</message>

<portType name="glossaryTerms">
  <operation name="getTerm">
    <input message="getTermRequest" />
    <output message="getTermResponse" />
  </operation>
</portType>

<binding type="glosaryTerms" name="b1">
  <soap:binding style="document" transport="http://schemas.xmlsoap.org/soap/http" />

  <operation>
    <soap:operation soapAction="http://example.com/getTerm" />
    <input><soap:body use="literal" /></input>
    <output><soap:body use="literal" /></output>
  </operation>
</binding>
~~~

`binding` 元素有兩個屬性 `name` `type`
`name` 您可以隨意定義一個 binding 的名稱，`type` 指向該綁定誰 `portType`
`soap:binding` 有兩個屬性 `style` `transport`
`stype` 可以是 `rpc` 或者 `document`
`transport` 定義 soap 協定
裡面的 `operation` 定義 portType 裡面的每個 operation 該揭露哪些訊息
soapAction 指向該 method 的 url 對該網址 post

## WSDL 和 UDDI
UDDI 是一個目錄服務，讓公司可以註冊或搜尋 Web Service
UDDI 代表 Universal Description, Discovery Integration
UDDI 是一個目錄用來儲存關於 Web Service 的資訊
UDDI 是 Web Service 介面的一個目錄，這個介面透過 WSDL 描述
UDDI 透過 SOAP 溝通


# SOAP
SOAP 代表 Simple Object Access Protocol
SOAP 是一種協定，用來存取 Web Service
SOAP 架構在 XML 之上
SOAP 是用來讓應用程式之間溝通的
SOAP 透過網路溝通

一個 SOAP 訊息就只是一個普通的 XML 不過包含下面的訊息
一個 Envelope 元素定義一個 XML 為 SOAP 訊息
一個 Header 元素包含表頭資訊
一個 Body 包含要回應的訊息
一個 Fault 包含錯誤和狀態訊息

在 Header 裡面加上 mustUnderstand 屬性表示這個標籤一定要給

~~~xml
<soap:Header>
  <m:Trans xmlns:m="http://www.w3schools.com/transaction/"
  soap:mustUnderstand="1">234
  </m:Trans>
</soap:Header>
~~~

### TODO 問題
1. 用 .NET 實作 WebServices 然後用 OSX Post 取得資料？

ANS: 在 Window 8 底下 PostMan 對 localhost 開發伺服器發送 Post 成功

2. 為什麼用 HTML Form 可以直接對 W3C http://www.w3schools.com/webservices/tempconvert.asmx?op=FahrenheitToCelsius
丟資料會成功，但是 PostMan 失敗哪邊搞錯了

ANS: 用 PostMan 要在 `x-www-form-urlencoded` 放資料就會成功了

3. 對自己的 Web Services(NET 版)測試用 Form 和 PostMan 丟資料

4. from-data, x-www-form-urlencoded, raw 的差異？

ANS: 在傳輸表單資料時候(Form) ，W3C 針對 Form 的內容定義了不同的類型，如果您希望傳送的是簡單的文字資料那麼 `x-www-form-urlencoded` 就能夠運作，這是預設的格式。
不過如果您要傳送非 ASCII 文字或者 Binary 資料，那麼就要改用 `form-data`





5. 模擬對 Sails 發送 SOAP 接到 XML 資料

6-1-1. 定義一個簡單的 WSDL(手刻)
6-1-2. 從 Sails 發出一份 WSDL，用 SOAP UI 檢查
6-2. 根據這份定義 User 要知道怎麼寫 SOAP Request
6-3. 開發者也要知道會收到什麼資料，回傳什麼資料
