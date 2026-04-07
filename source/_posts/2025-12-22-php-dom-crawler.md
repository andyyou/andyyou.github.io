---
title: "DomCrawler 爬蟲入門手冊"
date: 2025-12-22 09:30:15
tags:
  - php
categories: Program
---


`DomCrawler` 元件主要在簡化 HTML 和 XML 的檢索。

## 安裝

```sh
$ composer require symfony/dom-crawler

# 支援 CSS Selector
$ composer require symfony/css-selector
```

<!-- more -->

如果在 Symfony 應用程式以外的地方安裝此元件，你需要載入 `vendor/autoload.php` 檔案以支援 Composer 提供的自動載入類別的機制。

> Laravel 內建使用相同的機制，因此不需額外的處理。

## 使用

本文主要說明如何在任何 PHP 應用程式中將 `DomCrawler` 作為獨立元件的功能使用。若需要建立測試則請閱讀 [Symfony Functional Tests](https://symfony.com/doc/current/testing.html#functional-tests)。

`Crawler` 類別提供了一些方法用來查詢和操作 HTML 以及 XML。其物件實例表示一系列 `DOMElement` 物件，這些節點可以遍歷檢索。範例如下：

```php
use Symfony\Component\DomCrawler\Crawler;

$html = <<<'HTML'
<!DOCTYPE html>
<html>
    <body>
        <p class="message">Hello World!</p>
        <p>Hello Crawler!</p>
    </body>
</html>
HTML;

$crawler = new Crawler($html);
foreach ($crawler as $domElement) {
  var_dump($domElement->nodeName);
}
```

特定類別例如 [Link](https://github.com/symfony/symfony/blob/8.0/src/Symfony/Component/DomCrawler/Link.php)、[Image](https://github.com/symfony/symfony/blob/8.0/src/Symfony/Component/DomCrawler/Image.php)、[Form](https://github.com/symfony/symfony/blob/8.0/src/Symfony/Component/DomCrawler/Form.php) 可以和 HTML 的連結、圖片、表單進行互動。

`DomCrawler` 會嘗試自動修正 HTML 以符合官方規範。例如，若將 `<p>` 標籤嵌入另一個 `<p>` 裡面，它會被移至和父層的 `<p>` 同階層。這是 HTML5 規範的一部分。如果你遭遇一些非預期的行為或問題，可能是這個原因造成的。雖然 `DomCrawler` 不是為了匯出內容，但可以匯出 HTML 來查看修正後的版本。

### 過濾節點

使用 `XPath` 表達式，你可以選取特定節點：

```php
$crawler = $crawler->filterXPath('descendant-or-self::body/p');
```

實際上內部使用 `DOMXPath::query` 來執行 XPath 查詢。若你偏好 CSS Selector 可以安裝 [CssSelector Component](https://symfony.com/doc/current/components/css_selector.html)。它讓你可以使用類似 jQuery 選擇器語法。

```php
$crawler = $crawler->filter('body > p');
```

使用匿名函式可以查詢更加複雜的情況：

```php
use Symfony\Component\DomCrawler\Crawler;

$crawler = $crawler->fliter('body > p')
  ->reduce(function (Crawler $node, $i): bool {
    return ($i % 2) === 0;
  });
```

要篩選移除掉一個節點，匿名函式須回傳 `false`。

所有的過濾函式會回傳一個新的 `Crawler` 物件實例包含過濾後的內容。要檢查過濾的結果是否找到任何東西可以使用 `$crawler-count() > 0`。

[filterXPath()](https://github.com/symfony/symfony/blob/8.0/src/Symfony/Component/DomCrawler/Crawler.php#:~:text=function%20filterXPath) 和 [filter()](https://github.com/symfony/symfony/blob/8.0/src/Symfony/Component/DomCrawler/Crawler.php#:~:text=function%20filter) 方法都可以處理 XML 命名空間，命名空間可以被**自動偵測**或**明確手動註冊**。

例如下面的 XML：

```xml
<?xml version="1.0" encoding="UTF-8" ?>
<entry
    xmlns="http://www.w3.org/2005/Atom"
    xmlns:media="http://search.yahoo.com/mrss/"
    xmlns:yt="http://gdata.youtube.com/schemas/2007"
>
    <id>tag:youtube.com,2008:video:kgZRZmEc9j4</id>
    <yt:accessControl action="comment" permission="allowed"/>
    <yt:accessControl action="videoRespond" permission="moderated"/>
    <media:group>
        <media:title type="plain">Chordates - CrashCourse Biology #24</media:title>
        <yt:aspectRatio>widescreen</yt:aspectRatio>
    </media:group>
</entry>
```

傳統處理 XML 時通常我們需要手動註冊命名空間例如：

```php
$xml->registerXPathNamespace('yt', 'http://gdata.youtube.com/schemas/2007');
```

但是，Symfony DomCrawler 會自動處理，也就是我們可以直接使用上面提到的方法而不需要註冊例如：

```php
$crawler = $crawler->filterXPath('//default:entry/media:group//yt:aspectRatio');
```

或者

```php
$crawler = $crawler->filter('default|entry media|group yt|aspectRatio');
```

命名空間預設前綴為 `default` 也就是 `xmlns="http://www.w3.org/2005/Atom"` ，可以使用 `setDefaultNamespacePrefix()` 方法變更。當載入內容時，如果預設命名空間時文件中唯一的命名空間，則會被自動移除，這是為了簡化 XPath 查詢。

命名空間可以使用 [registerNamespace()](https://github.com/symfony/symfony/blob/8.0/src/Symfony/Component/DomCrawler/Crawler.php#:~:text=function%20registerNamespace) 明確手動註冊。

```php
$crawler->registerNamespace('m', 'http://search.yahoo.com/mrss/');
$crawler = $crawler->filterXPath('//m:group/yt:aspectRatio');
```

檢查目前的節點是否符合選擇器：

```php
$crawler->matches('p.lorem');
```

### 遍歷節點

通過位置擷取

```php
$crawler->filter('body > p')->eq(0);
```

取得目前選擇區域中第一個或最後節點：

```php
$crawler->filter('body > p')->first();
$crawler->filter('body > p')->last();
```

取得目前選區中同樣階層的節點：

```php
$crawler->filter('body > p')->siblings();
```

取得目前節點相同階層之前或之後的節點：

```php
$crawler->filter('body > p')->nextAll();
$crawler->filter('body > p')->previousAll();
```

取得全部子節點或上層節點：

```php
$crawler->filter('body')->children();
$crawler->filter('body > p')->ancestors();
```

取得第一層子元素符合 CSS 選擇器的節點：

```php
$crawler->filter('body')->children('p.lorem');
```

取得最接近上層符合選擇器的節點：

```php
$crawler->closest('p.lorem');
```

全部遍歷的方法都會回傳新的 `Crawler` 物件實例。

### 存取節點值

存取目前選取的第一個節點名稱（HTML標籤名稱）

```php
$tag = $crawler->filterXPath('//body/*')->nodeName();
```

```php
// 如果節點不存在，則呼叫 text() 會發生例外
$message = $crawler->filterXPath('//body/p')->text();

// 為了避免例外，可以傳入一個參數，當沒有內容的時候則回傳該參數
$message = $crawler->filterXPath('//body/p')->text('Default text');

// 預設，text() 會移除前後空白，連續空白會壓縮成一個
// 使用 false 作為第二個參數可以保留原始內容
$crawler->filterXPath('//body/p')->text('預設', false);

// innerText() 類似於 text()，但只回傳當前節點的直接文字內容，不包含子節點的文字
$text = $crawler->filterXPath('//body/p')->innerText();
// 如果內容是 <p>Foo <span>Bar</span></p> 或 <p><span>Bar</span> Foo</p>
// innerText() 在兩種情況下都回傳 'Foo'
// 而 text() 則分別回傳 'Foo Bar' 和 'Bar Foo'

// 如果有多個文字節點，分散在其他子節點之間，例如：
// <p>Foo <span>Bar</span> Baz</p>
// innerText() 只回傳第一個文字節點 'Foo'

// 跟 text() 一樣，innerText() 預設也會去除空白字元，
// 但你可以傳入 false 作為參數來取得未修改的原始文字
$text = $crawler->filterXPath('//body/p')->innerText(false);
// ⚠️ innerText() 若找不到節點，沒有預設值的功能。
```

存取選取節點的屬性值：

```php
$class = $crawler->filterXPath('//body/p')->attr('class');
```

我們可以通過第二個參數設定預設值，當節點或屬性為空的時候使用預設值。

```php
$class = $crawler->filterXPath('//body/p')->attr('class', 'default-class');
```

從節點列表擷取屬性

```php
$attributes = $crawler->filterXPath('//body/p')
  ->extract(['_name', '_text', 'class']);
```

特殊屬性 `_text` 表示節點的內容值，而 `_name` 表示元素名稱即 HTML 標籤名稱。

在列表的每一個節點呼叫匿名函式：

```php
use Symfony\Component\DomCrawler\Crawler;

$nodeValues = $crawler->filter('p')->each(function (Crawler $node, $i): string {
  return $node->text();
});
```

匿名函式會收到節點的 `Crawler` 物件和位置索引。其結果會是由匿名函式處理過回傳的值組成的陣列。

當搭配 `each` 進行巢狀處理時，請注意 `filterXPath()` 會從當前 `Crawler` 整個文件開始：

```php
// 假設我們有一個例子
$html = <<<'HTML'
<div>
    <parent id="1">
        <sub-tag>
            <sub-child-tag>內容 A</sub-child-tag>
        </sub-tag>
    </parent>
    <parent id="2">
        <sub-tag>
            <sub-child-tag>內容 B</sub-child-tag>
        </sub-tag>
    </parent>
</div>
HTML;

$crawler->filterXPath('parent')->each(function (Crawler $parentCrawler, $i): void {
  	// ❌錯誤，這樣檢索不到節點
  	$subCrawler = $parentCrawler->filterXPath('sub-tag/sub-child-tag');
  	
  	// ✅根節點也要指定，`node()` 表示當前節點
  	$subCrawler = $parentCrawler->filterXPath('parent/sub-tag/sub=child-tag');
  	$subCrawler = $parentCrawler->filterXPath('node()/sub-tag/sub-child-tag');
});
```

### 加入內容

Crawler 支援多種加入內容的方式，但它們是互斥的，因此你只能使用其中一種。例如如果你在 `Crawler` 建構子傳入內容，那麼就不能使用 `addContent()` 方法。

```php
$crawler = new Crawler('<html><body /></html>');

$crawler->addHtmlContent('<html><body /></html>'); // Attaching DOM nodes from multiple documents in the same crawler is forbidden.
$crawler->addXmlContent('<root><node/></root>');

$crawler->addContent('<html><body/></html>');
$crawler->addContent('<root><node/></root>', 'text/xml');

$crawler->add('<html><body/></html>');
$crawler->add('<root><node/></root>');
```

`addHtmlContent()` 和 `addXmlContent()` 方法預設使用 UTF-8 編碼，也就是我們可以正常使用中文、日文等，但你可以通過第二個參數變更這個行為。

`addContent()` 會自動推測編碼，若沒有指定 charset，會使用 ISO-8859-1 西歐語言不支援中文。

由於 `Crawler` 是基於 DOM 擴展的實作，它也可以和 [DOMDocument](https://secure.php.net/manual/en/class.domdocument.php)、[DOMNodeList](https://secure.php.net/manual/en/class.domnodelist.php)、[DOMNode](https://secure.php.net/manual/en/class.domnode.php) 物件互動：

```php
$domDocument = new \DOMDocument();
$domDocument->loadXml('<root><node /><node /></root>');
$nodeList = $domDocument->getElementsByTagName('node');
$node = $domDocument->getElementsByTagName('node')->item(0);

$crawler->addDocument($domDocument);
$crawler->addNodeList($nodeList);
$crawler->addNodes([$node]);
$crawler->addNode($node);
$crawler->add($domDocument);
```

### 操作和輸出 `Crawler`

 `Crawler` 的方法的目標在初始化將內容填入 `Crawler` 而不是操作 DOM，然而由於 `Crawler` 本質也是一系列 `DOMElement` 物件，我們可以使用 `DOMElement` 、`DOMNode`、`DOMDocument` 任何的屬性和方法。舉例來說你可以使用下面範例取得 HTML

```php
$html = '';

foreach ($crawler as $domElement) {
    $html .= $domElement->ownerDocument->saveHTML($domElement);
}
```

或者

```php
// 若節點不存在，則呼叫 html() 會產生例外
$html = $crawler->html();

// 為了避免例外，可以設定預設值
$html = $crawler->html('Default <strong>HTML</strong> content');

$html = $crawler->outerHtml();
```

### 表達式

`evaluate()` 方法會解析給予的 XPath 表達式。根據表達式回傳值。如果表達式的評估結果是標量值（Scalar value，例如 HTML 屬性。在這裡指的是字串、數字或布林值。最常見的情況是你只想抓取某個 HTML 標籤內的「屬性文字」或「純文字內容」，而不是整個標籤物件。），則會回傳一個結果陣列；如果評估結果是 DOM 文件（節點），則會回傳一個新的 `Crawler` 實例。

```php
use Symfony\Component\DomCrawler\Crawler;

$html = '<html>
<body>
    <span id="article-100" class="article">Article 1</span>
    <span id="article-101" class="article">Article 2</span>
    <span id="article-102" class="article">Article 3</span>
</body>
</html>';

$crawler = new Crawler();
$crawler->addHtmlContent($html);

$crawler->filterXPath('//span[contains(@id, "article-")]')->evaluate('substring-after(@id, "-")');
/*
[
    0 => '100',
    1 => '101',
    2 => '102',
];
*/

$crawler->evaluate('substring-after(//span[contains(@id, "article-")]/@id, "-")');
/*
[
    0 => '100',
]
*/

$crawler->filterXPath('//span[@class="article"]')->evaluate('count(@id)');
/*
[
    0 => 1.0,
    1 => 1.0,
    2 => 1.0,
]
*/

$crawler->evaluate('count(//span[@class="article"])');
/*
[
    0 => 3.0,
]
*/
```

### 連結

使用 `filter()` 方法可以根據 `id` 或 `class` 找到連結。使用 `selectLink()` 方法可以根據連結的內容查找連結（它還會查找 `alt` 屬性包含該內容的可點擊圖片）。

```php
// 取得 Crawler 物件
$linkCrawler = $crawler->filter('#sign-up');
$linkCrawler = $crawler->filter('.user-profile');
$linkCrawler = $crawler->selectLink('Log in');

// 取得 Link 物件
$link = $linkCrawler->link();

// 或者可以直接去的連結物件
$link = $crawler->filter('#sign-up')->link();
$link = $crawler->filter('.user-profile')->link();
$link = $crawler->selectLink('Log in')->link();
```

[Link](https://github.com/symfony/symfony/blob/8.0/src/Symfony/Component/DomCrawler/Link.php) 物件支援一些實用的方法取得更多資訊例如取得超連結

```PHP
$uri = $link->getUri();
```

`getUri()` 尤其實用，因為它會整理 `href` 值並將其轉換成可用的形式。例如 `href="#foo"` 會回傳完整的 URI ，然後我們可以直接使用。

### 圖片

要通過 `alt` 查找圖片可以使用 `selectImage` 方法。一樣會先回傳 `Crawler` 物件，然後調用 `image()` 取得 `Image` 物件：

```php
$imageCrawler = $crawler->selectImage('Kitten');
$image = $imageCrawler->image();
```

`Image` 物件也有 `getUri()` 方法。

### 表單

表單也有特殊的處理。`Crawler` 支援 `selectButton()` 方法，該方法會回傳另一個 `Crawler` 物件代表 `<button>` 、`<input type="submit">` 或 `<input type="button">` 元素。其參數的字串會拿來搜尋這些元素的 `id` 、`alt` 、`name` 、 `value` 和元素內容。這個方法非常實用，因為我們可以用它回傳按鈕所在的 `Form` 物件

```php
// 假設按鈕範例 <button id="my-super-button" type="submit">My super button</button>
$form = $crawler->selectButton('My super button')->form();

$form = $crawler->selectButton('my-super-button')->form();

// 我們也可以直接查詢表單本身
$form = $crawler->filter('.form-vertical')->form();

// 然後可以幫欄位加入資料
$form = $crawler->selectButton('my-super-button')->form([
  'name' => 'andyyou'
]);
```

[Form](https://github.com/symfony/symfony/blob/8.0/src/Symfony/Component/DomCrawler/Form.php) 表單物件也有很多方法

```php
$uri = $form->getUri();
$method = $form->getMethod();
$name = $form->getName();
```

`getUri()` 方法的作用不僅是回傳 `action` 屬性。如果表單的 `method` 是 `GET` 那麼它會模擬瀏覽器的行為，回傳 `action` 屬性包含表單欄位值的 QueryString。

另外，還支援可選的按鈕屬性 `formaction` 和 `formmethod` 。`getUri()` 和 `getMethod()` 會考慮這些屬性，確保回傳正確的 `action` 和 `method` ，也就是如果表單有 action，但提交按鈕有 formaction，則實際提交時會優先使用按鈕的 formaction。

我們可以在表單上設定和取得值：

```php
$form->setValues([
  'registration[username]' => 'symfonyfan',
  'registration[terms]' => 1,
]);

// 取得一個表單資料的陣列
$values = $form->getValues();
/*
[
  "registration[username]" => "symfonyfan",
  "registration[terms]" => "1",
]
*/

$values = $form->getPhpValues();
/*
[
  "registration" => [
    "username" => "symfonyfan",
    "terms" => "1",
  ],
]
*/
```

如果要處理多階層欄位：

```html
<form>
    <input name="multi[]">
    <input name="multi[]">
    <input name="multi[dimensional][]" value="1">
    <input name="multi[dimensional][]" value="2">
    <input name="multi[dimensional][]" value="3">
</form>
```

傳入值的陣列：

```php
// 等於設定第一個 multi[0] 的值
$form->setValues(['multi' => ['value']]);

// 設定多個欄位
$form->setValues([
  'multi' => [
    1 => 'value',
    'dimensional' => ['其他']
  ]
]);
```

另外，`Form` 物件可讓我們像使用瀏覽器一樣和表單互動，例如選擇單選按鈕，勾選和上傳檔案：

```php
$form['registration[username]']->setValue('symfonyfan');

// 勾選 checkbox
$form['registration[terms]']->tick();
$form['registration[terms]']->untick();

// 選擇 select option
$form['registration[birthday][year]']->select(1986);

// 多選
$form['registration[interests]']->select(['symfony', 'cookies']);

// 上傳
$form['registration[photo]']->upload('/path/to/lucas.jpg');
```

### 使用 Form Data

如果你只是在執行內部測試，你可以直接擷取表單提交的資料

```php
$values = $form->getPhpValues();
$files = $form->getPhpFiles();
```

若是使用外部 HTTP 客戶端，則可以使用表單取得建立 POST 發出的資訊：

```php
$uri = $form->getUri();
$method = $form->getMethod();
$values = $form->getValues();
$files = $form>getFiles();
```

[BrowserKit](https://symfony.com/doc/current/components/browser_kit.html) 元件提供的 `HttpBrowser` 就是一個很好的整合範例，它能識別 Symfony Crawler 物件，並且可以用它來提交表單。

```php
use Symfony\Component\BrowserKit\HttpBrowser;
use Symfony\Component\HttpClient\HttpClient;

$browser = new HttpBrowser(HttpClient::create());
$crawler = $browser->request('GET', 'https://github.com/login');

$form = $crawler->selectButton('Sign in')->form();
$form['login'] = 'symfonyfan';
$form['password'] = 'password';
$crawler = $browser->submit($form);
```

### 選擇無效選項值

預設，選擇類型欄位如 `select` 、`radio` 有內部檢查機制，防止設定無效值。如果你希望可以設定無效的值，你可以使用 `disableValidation()` 方法：

```php
$form['country']->disableValidation()->select('Invalid value');

// 關閉整個表單的驗證
$form->disableValidation();
$form['country']->select('Invalid value');
```

### 解析 URI

[UriResolver](https://github.com/symfony/symfony/blob/8.0/src/Symfony/Component/DomCrawler/UriResolver.php) 類別接受一個 URI 包含相對路徑、絕對路徑、URI 片段等，然後將其轉換層一個絕對路徑

```php
use Symfony\Component\DomCrawler\UriResolver;

UriResolver::resolve('/foo', 'http://localhost/bar/foo/'); // http://localhost/foo
UriResolver::resolve('?a=b', 'http://localhost/bar#foo'); // http://localhost/bar?a=b
UriResolver::resolve('../../', 'http://localhost/'); // http://localhost/
```

### 其他

搭配 Laravel 的時候一般我們可以使用 `GuzzleHttp\Client`

```php
use GuzzleHttp\Client;

$client = new Client([
    'timeout' => 30,
    'headers' => ['User-Agent' => 'Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36']
]);
$url = '...';
$response = $client->get($url);
$html = $response->getBody()->getContents();
$crawler = new Crawler($html);
```



### 參考資源

- [DomCrawler](https://symfony.com/doc/current/components/dom_crawler.html)