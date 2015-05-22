---
layout: post
title: "Javascript Unicode"
date: 2015-05-21 12:50:00
categories: Javascript
---

為了理解 ES6 到底對於 Unicode 萬國碼有哪些新的支援。我們得從原因理解起。

# Javascript 在處理 Unicode 時很有多問題
關於 Javascript 處理 Unicode 的方式...至少可以說是很奇怪。這篇文章闡述在 Javascript 中存取 Unicode 的痛點以及 ES6 如何改善這個問題。

# Unicode 基礎
在我們深入探討 Javascript 之前，讓我們先確認當我們談到 Unicode 的時候說的是相同的事情。

有關 Unicode 的觀念其實非常簡單，把它想成一個資料庫，存取著您能想到的所有文字符號，且每一個文字符號都對應著一組數字。這個數字就叫`編碼位置`(Code point)，也有人稱`碼點` `代碼點`。這個編碼位置是唯一的。透過這種方式可以簡單的存取特定文字符號而不用直接輸入符號本身。

例如:
* `A` = `U+0041`
* `a` = `U+0061`
* `©` = `U+00A9`
* `☃` = `U+2603`
* `💩` = `U+1F4A9`

編碼位置通常使用 16 進制的格式，位元左邊捕 0 到至少 4 位，使用 `U+` 當作前綴字。
編碼可能的範圍從 `U+0000` 到 `U+10FFFF` 超過 110 萬個符號。為了確保其組織性，Unicode 把這個範圍的編碼區分成 17 個區段，各自由 65536 個編碼組成。
如果你曾經看過 Wiki 百科上的翻譯，他翻成`平面`，由 17 個平面組成。

第一個`平面`稱作`基本多文種平面` Basic Multilingual Plane, 簡稱BMP。這大概是最重要的一個。它包含了大部份常用的字符。一般使用英文的情況下您不會需要 BMP 以外的編碼來編輯文件。

BMP 以外剩下大概 1 百萬個符號屬於`補充平面`(Supplementary planes or Astral planes)
補充平面的字非常好辨別: 如果某個字符需要超過 4 位元的 16 進制來表示那它就屬於補充平面。

現在我們有了對 Unicode 的基本認識了。來看看如何應用到 Javascript 的字串。

# 跳脫序列(Escape sequence)

{% highlight js %}
console.log('\x41\x42\x43');
// 'ABC'

console.log('\x61\x62\x63');
// 'abc'
{% endhighlight %}

這個東西術語叫做 16 進制的跳脫序列(字元)。由 16 進制格式的 2 個位元組成代表一個編碼位置。舉例來說 `\x41` 代表 `U+0041`。
跳脫序列可以被用來表示編碼位置從 `U+0000` 到 `U+00FF`。

另外一種常見的跳脫序列的表示類型如下

{% highlight js %}
console.log('\u0041\u0042\u0043');
// 'ABC'

console.log('I \u2661 JavaScript');
// 'I ♡ JavaScript'
{% endhighlight %}

這種格式被稱作`萬國碼跳脫序列`，算了！還是記英文吧！Unicode escape squences 由16 進制格式 4 個位元組成精準的表達編碼位置，舉例來說: `\u2661` 表示 `U+2661` 這種跳脫序列可以用來表示 `U+0000` 到 `U+FFFF` 範圍的萬國碼 Unicode 等於是整個基本多文種平面(BMP) 

那麼..其他平面呢? 我們需要大於 4 位元來表示其他編碼位置啊! 我們要如何使用跳脫序列呈現它們?

ES6 引進了新類型的跳脫序列: `Unicode code point escapes` 讓事情變得比較簡單

舉例來說: 

{% highlight js %}
console.log('\u{41}\u{42}\u{43}');
// 'ABC'

console.log('\u{1F4A9}');
// '💩' U+1F4A9
{% endhighlight %}

在大括號之間您可以使用 6 位元的 16 進制，這麼一來就足夠表示所有的 Unicode 編碼。
所以透過這種類型的跳脫序列您可以輕易的跳脫任何您想用的符號

為了兼容 ES5 和舊有的環境，一個不是很好的解決方案是使用成對編碼來代理

{% highlight js %}
console.log('\uD83D\uDCA9');
// '💩' U+1F4A9
{% endhighlight %}

在這種情況下每一個跳脫字元(跳脫序列)代表一半的編碼位置，2 個`代理編碼`組成一個字符的 Code point。

注意到這個編碼沒辦法很直覺的看出其規則，這是有一套公式的

例如一個 `C` 字符大於 `0xFFFF` 就得對應到 `<H, L>` 成對的代理編碼

~~~
H = Math.floor((C - 0x10000) / 0x400) + 0xD800
L = (C - 0x10000) % 0x400 + 0xDC00
~~~

> 之後我們提到代理編碼指的就是兩個編碼其中之一 <H,L> 第一個的是 H, 第二個是 L 

要反轉回來則是

~~~
C = (H - 0xD800) * 0x400 + L - 0xDC00 + 0x10000
~~~

透過這種代理編碼的機制所有補充平面的編碼位置(U+010000 - U+10FFFF) 都可以使用。不過使用單一跳脫字元來表示 BMP 裡面的字，兩個跳脫字元(代理編碼)來處理剩下補充平面的字很容易讓人搞混，造成很多惱人的後果。

# 計算 JavaScript 字串的文字(符號)
假設您想計算一個字串的文字有幾個，您會怎麼處理呢?

直覺的想法大概是使用 `length`

{% highlight js %}
console.log('A'.length);
// 1

console.log('A' == '\u0041');
// true
{% endhighlight %}

上面這個例子 `length` 剛好是字元的數量，說有 1 個文字這很合理。
很顯然的我們每一個文字只需要一個跳脫字元，但實際上卻不是這樣。例如:

{% highlight js %}
console.log('𝐀'.length); // U+1D400 注意這不只是全形Ａ
// 2 

console.log('𝐀' == '\uD835\uDC00');
// true

console.log('𝐁'.length) // U+1D401 
// 2

console.log('𝐁' == '\uD835\uDC01');
// true

console.log('💩'.length);
// 2

console.log('💩' == '\uD83D\uDCA9');
// true
{% endhighlight %}

在內部 JavaScript 把補充平面的字符視為兩個跳脫字元(代理編碼)表示一個字。如果您在 ES5 兼容的瀏覽器輸出您會看到他把他視為兩個跳脫字元 length 為 2 ，人們對於字面上只顯示一個字但是 `length` 卻為 `2` 會產生困惑。


# 計算補充平面裡的文字
回到剛剛的問題，那我們如何計算 JS 字串中有幾個字?
這個小技巧針對`代理編碼`做處理，當我們認出這兩個跳脫字元會組成一個字的時候只計算一次

{% highlight js %}
var regexAstralSymbols = /[\uD800-\uD8FF][\uDC00-\uDCFF]/g;

function countSymbols(string) {
  return string.replace(regexAstralSymbols, '_').length;
}
{% endhighlight %}

或者您也可以使用 `Punycode.js`，`punycode.ucs2.decode` 方法可以取得一個字串並回傳一個包含 Unicode 編碼位置的陣列。如此一來您就可以計算幾個字了。

在 ES6 您可以透過 `Array.form` 做類似的事情，透過使用字串的 iterator 來切割字串成為一個陣列

{% highlight js %}
var astral = Array.from("𝐀𝐁💩");
console.log(astral);
console.log(astral.length);
// 3
{% endhighlight %}

或者使用 `...`

{% highlight js %}
console.log([..."𝐀𝐁💩"].length)
// 3
{% endhighlight %}

使用上面提到的這些方法，我們可以解決計算幾個字的問題。

# 看起來一樣，但卻不一樣
但是如果我們開始去賣弄我們從文章中學到的知識，計算文字的數量甚至更多複雜的操作例如下面這段程式碼

{% highlight js %}
console.log('mañana' == 'mañana');
// false
{% endhighlight %}

JavaScript 會告訴我們這兩個字串不一樣，但看起來明明就一樣。
試著到[這個網址](https://mothereff.in/js-escapes#1ma%C3%B1ana%20man%CC%83ana)看看

Javascript escapes 工具告訴我們其中的不同

{% highlight js %}
console.log('ma\xF1ana' == 'man\u0303ana');
// false

console.log('ma\xF1ana'.length);
// 6

console.log('man\u0303ana'.length);
// 7
{% endhighlight %}

第一個字串包含的是 `U+00F1` 是一個拉丁字小寫 N 加上波浪號。而第二個字串裡面的事 `U+006E` 拉丁字小寫 N 加上 `U+0303` 波浪號，兩個編碼合體成一個字。這樣你明白了為什麼他們不一樣了吧。

然而如果我們希望兩個字串計算結果都會是 6 個字呢?
在 ES6 也相當直覺

{% highlight js %}
var normalized = "mañana".normalize('NFC'); // 把字串標準化

console.log(Array.from(normalized).length);
// 6
console.log([...normalized].length);
// 6
{% endhighlight %}

這個標準化 `normalize` 方法是內建 String.prototype 的方法，他會根據[Unicode normalization](http://unicode.org/reports/tr15/)的規則執行，找出那些字的差異，如果找到那種由兩個`代理編碼組成的字`卻長得跟`另一單一編碼位置`一樣的字，它會把它轉成單一的那種編碼。

{% highlight js %}
[...'mañana'].lenght // U+00F1
// 6
[...'mañana'].length // U+006E + U+0303
// 6

// 透過程式碼驗證
var normalized = "mañana".normalize('NFC');
console.log(normalized[2] + " = " + normalized.charCodeAt(2))
// ñ = 241, 241 轉成 16 進制 F1
{% endhighlight %}

為了向下相容 ES5 和舊環境可以使用這個[Polyfill](https://github.com/walling/unorm)

# 事情還很複雜 - 計算其他組合式的代理編碼
光上面這些還不夠完美，編碼位置可以有多種組合方式其結果看起來是一個字，但是卻沒有標準化的格式(或者說沒有相同樣子的字取代)。
這種時後 normalization 就幫不上忙了。

> 大部份開發者應該很少遇到這類問題吧???

{% highlight js %}
var q = 'q\u0307\u0323'.normalize('NFC') // q̣̇
// 經過 normalize 還是 q\u0307\u0323

console.log([...q].length);
// 是 3 不是 1

console.log([...'Z͑ͫ̓ͪ̂ͫ̽͏̴̙̤̞͉͚̯̞̠͍A̴̵̜̰͔ͫ͗͢L̠ͨͧͩ͘G̴̻͈͍͔̹̑͗̎̅͛́Ǫ̵̹̻̝̳͂̌̌͘!͖̬̰̙̗̿̋ͥͥ̂ͣ̐́́͜͞'].length);
// 是 74 不是 6
{% endhighlight %}


此時您可以使用正規式來移除那些組合的符號

{% highlight js %}
var sample = "Z͑ͫ̓ͪ̂ͫ̽͏̴̙̤̞͉͚̯̞̠͍A̴̵̜̰͔ͫ͗͢L̠ͨͧͩ͘G̴̻͈͍͔̹̑͗̎̅͛́Ǫ̵̹̻̝̳͂̌̌͘!͖̬̰̙̗̿̋ͥͥ̂ͣ̐́́͜͞";
var pattern = /([\0-\u02FF\u0370-\u1DBF\u1E00-\u20CF\u2100-\uD7FF\uDC00-\uFE1F\uFE30-\uFFFF]|[\uD800-\uDBFF][\uDC00-\uDFFF]|[\uD800-\uDBFF])([\u0300-\u036F\u1DC0-\u1DFF\u20D0-\u20FF\uFE20-\uFE2F]+)/g;
var stripped = sample.replace(pattern, function(ele, symbol, marks) {
  return symbol;
});
console.log(stripped.length);
// 6
{% endhighlight %}

這種做法可以移除其他用來組合的符號，只留下那些我們要的字。這種解決方案甚至連 ES3 都能使用。

# 計算其他類型的字形集合(`語素簇`) Grapheme Cluster
對於像 `நி`(`U+0BA8` + `U+0BBF` = `\u0ba8\u0bbf`) 這種字集或者像韓文是由一堆母音和子音組成的例如 `깍`(`ᄁ` + `ᅡ` + `ᆨ`) 上面的邏輯仍然是相對粗糙。

Unicode 標準附件 #29 提供一個[演算法](http://www.unicode.org/reports/tr29/#Grapheme_Cluster_Boundaries)用來判斷字形的邊界，就是怎麼樣算是人看的一個字。
如果是為了給所有 Unicode 絕對精準的解決方案那麼時做這個邏輯可能是比較正解的方法。

# 反轉字串
你可能覺得在 JavaScript 中反轉一個字串很簡單。對嗎?
通常你會這麼做

{% highlight js %}
function reverse(string) {
  return string.split('').reverse().join('');
}
{% endhighlight %}

看起似乎可行

{% highlight js %}
reverse('abc');
// 'cba'

reverse('mañana'); // U+00F1
// 'anañam'
{% endhighlight %}

然而當字串混雜著一些`代理編碼`即兩個編碼位置組合成的一個字或者`補充字面`裡的字時全部就都亂了套。

{% highlight js %}
reverse('mañana'); // U+006E + U+0303
// 'anãnam' 
// 看吧！ `~` 跑到 `a` 上面了

reverse('💩') // U+1F4A9
// '��' => `'\uDCA9\uD83D'`
// `💩` 大便符號完全錯誤因為兩個編碼位置反了
{% endhighlight %}

在 ES6 裡面修正了這個問題

{% highlight js %}
console.log(Array.from("𝐀𝐁💩").reverse().join(''));
// "💩𝐁𝐀"
console.log([..."𝐀𝐁💩"].reverse().join(''));
// "💩𝐁𝐀"
{% endhighlight %}

但仍然沒有解決涉及多個編碼組合成看起來像一個字的問題。

幸運的是有人解決了`反轉字串遇到怪字`這個問題，只要透過[Esrever](https://mths.be/esrever)

{% highlight js %}
// 使用 Esrever (https://mths.be/esrever)

esrever.reverse('mañana') // U+006E + U+0303
// 'anañam'

esrever.reverse('💩') // U+1F4A9
// '💩' U+1F4A9
{% endhighlight %}

# 在字串方法中使用 Unicode 的問題
除了陣列 `reverse()` 的行為外，這種問題也影響到字串的方法。

### 轉換編碼位置成文字(符號)
`String.fromCharCode` 讓我們可以用 Unicode 編碼位置來建立一個字串，不過呢只有在`基本多文種平面`(BMP)範圍內是正常的(U+0000 到 U+FFFF)，如果我們用了在補充平面的字將會得到非預期的結果:

{% highlight js %}
String.fromCharCode(0x0041); // U+0041
// 'A' U+0041

String.fromCharCode(0x1F4A9) // U+1F4A9
// '' 是 U+F4A9, 而不是 U+1F4A9
{% endhighlight %}

而解決方法就是用上面提到的公式自己計算，並且把拆開的兩個編碼當作參數帶入

{% highlight js %}
String.fromCharCode(0xD83D, 0xDCA9);
// '💩'  U+1F4A9
{% endhighlight %}

>16 進制在 C語言、Shell、Python、Java語言及其他相近的語言使用字首「0x」，例如「0x5A3」。

>而在HTML，十六進制可以用「x」，例如 `&#x0041;` 會等於 「A」。

如果你不想要自己處理這些麻煩的計算您可以使用 [Punycode.js](https://mths.be/punycode) 提供的工具

{% highlight js %}
punycode.ucs2.encode([0x1F4A9]);
// '💩' U+1F4A9
{% endhighlight %}

除了上面這些方法，幸運的是 ES6 也引入了新的方法 `String.fromCodePoint()` 就可以直接用來轉換補充平面裡面的字了

{% highlight js %}
String.fromCodePoint(0x1F4A9);
// '💩' U+1F4A9
{% endhighlight %}

同時呢為了向下相容舊環境您可以使用 Polyfill。

### 從字串中取出一個字
如果您想用 `String.prototype.charAt(index)` 來擷取第一個字符，遇上`大便符號💩`這種代理編碼類型的字，這個方法只能夠抓出第一個編碼。

{% highlight js %}
'💩'.charAt(0) // U+1F4A9
// '\uD83D' 
// U+1F4A9 代理編碼的兩個編碼中第一個是 U+D83D
{% endhighlight %}

在 ES7 的建議中已經有提出 `String.prototype.at(index)` 來處理這個問題了。

{% highlight js %}
'💩'.at(0) // U+1F4A9
// '💩' U+1F4A9
{% endhighlight %}

同樣的在 ES5 和舊環境中還是可以找到 Polyfill 來處理。

### 從字串取得字的編碼位置
類似於上面的狀況，如果您使用 `Strint.prototype.charCodeAt(index)` 來檢索字串中第一個字的編碼位置，您一樣會取得代理編碼 2 個編碼中的第一個編碼(就是上面提到的 `H`)

{% highlight js %}
'💩'.charCodeAt(0);
// 0xD83D
// 瀏覽器會給 10 進制 55357 換算之後的確是 D83D
{% endhighlight %}

再一次感謝 ES6，一樣提供了 `String.prototype.codePointAt(index)` 方法來解決這個問題。

{% highlight js %}
'💩'.codePointAt(0)
// 0x1F4A9
{% endhighlight %}

### 遍歷字串中的每個字
假設您想要用迴圈遍歷(就是一個字一個字取出)一個字串，分別對每個字做點處理。

在 ES5 裡我們可能要先處理成陣列

{% highlight js %}
function getSymbols(string) {
  var length = string.length;
  var index = -1;
  var output = [];
  var character;
  var charCode;
  while (++index < length) {
    character = string.charAt(index);
    charCode = character.charCodeAt(0);
    if (charCode >= 0xD800 && charCode <= 0xD8FF) {
      // 這邊我們假設不會出現那種只有一半的代理編碼
      output.push(character + string.charAt(++index));
    } else {
      output.push(character);
    }
  }
  return output;
}

var symbols = getSymbols('💩');
console.log(symbols);
{% endhighlight %}

不意外的 ES6 又出來拯救我們了，在 ES6 裡只要用 `for...of` 就可以準確的取出每一個`字`。

{% highlight js %}
for (let symbol of '💩') {
  console.log(symbol == '💩');
}
// true
{% endhighlight %}

### 其他問題
Unicode 的確影響了很多 String Method 的行為，包含我們在這裡沒提到的 `substring`, `slice` 所以實作時請小心。

# Unicode 在正規式中的問題

### 匹配`代碼位置`(Code Point)與 `Unicode 標量值`(Unicode Scalar Values)

`.` 句點在正規式中只會匹配`一個字元`即我們上面說的一個編碼(U+0041)，本來這都很合理，但因為 JavaScript 採用了`代理編碼`的機制，用了兩個實際的編碼組合成一個字。一旦我們要匹配補充平面的字時，永遠不會匹配成功。

{% highlight js %}
/foo.bar/.test('foo💩bar')
// false
{% endhighlight %}

讓我們再想想...還有什麼正規式的寫法可以匹配 Unicode 字符

{% highlight js %}
console.log(/^[\s\S]$/.test('💩'));
// false
{% endhighlight %}

還是 GG ，事實證明正規式要匹配一個 Unicode 編碼位置並不是那麼直覺。

{% highlight js %}
console.log(/[\0-\uD7FF\uE000-\uFFFF]|[\uD800-\uDBFF][\uDC00-\uDFFF]|[\uD800-\uDBFF](?![\uDC00-\uDFFF])|(?:[^\uD800-\uDBFF]|^)[\uDC00-\uDFFF]/.test('💩'));
// true
{% endhighlight %}

當然啦！在實際開發時我是絕對不想寫這些正規式，光寫就暈了更何況還要 debug。為了取得上面那堆正規式我們可以偷吃步使用 [regenerate](https://github.com/mathiasbynens/regenerate) 函式庫。它可以很輕鬆地幫我們產出這些正規式

{% highlight js %}
regenerate().addRange(0x0, 0x10FFFF).toString()
{% endhighlight %}

這個正規式可以匹配基本多文種平面，補充平面的字，甚至只有一個代理編碼。

只寫一個代理編碼在技術上是可行的，但因為他們不會對應到任何實際的字，所以應該避免。
Unicode 標量值(Unicode Scalar Values)指的是所有的編碼位置但扣掉那些代理編碼的`編碼位置` 
下面是如何產出符合 Unicode Scalar Values 規則的正規式

{% highlight js %}
regenerate()
 .addRange(0x0, 0x10FFFF) // 所有 Unicode 編碼位置
 .removeRange(0xD800, 0xDBFF) // 減去第一位的代理編碼
 .removeRange(0xDC00, 0xDFFF) // 減去第二位的代理編碼
 .toRegExp()
{% endhighlight %}

`regenerate` 可以協助我們建立那些複雜的正規式，用相對語意化的方式讓我們比較好維護。

ES6 替正規式加入了 `u` 修飾符(flag) 就是在正規式尾巴那些用來設定比對方式的參數
例如 `/[\w]/g`

* g 全域比對
* i 忽略大小寫
* gi 全域比對 + 忽略大小寫 

現在多了 `u` 讓正規式用 `.` 在匹配時可以正確的匹配到補充平面裡的字

{% highlight js %}
/foo.bar/.test('foo💩bar');
// false

/foo.bar/u.test('foo💩bar');
// true
{% endhighlight %}

注意: `.` 仍然不會匹配到換行字元，當設定了 `u` flag 之後在兼容環境中等於是使用下面的程式碼

{% highlight js %}
regenerate()
  .addRange(0x0, 0x10FFFF)
  .remove(
    0x000A, // <LF> 
    0x000D, // <CR>
    0x2028, // <LS>
    0x2029, // <PS>
  ).toString();
{% endhighlight %}


### 補充平面的範圍設定
先想想這種狀況 `/[a-c]/` 這樣寫可以比對 `U+0061` 到 `U+0063` 即 a 到 c 。那如果換成 `/[💩-💫]/` 勒?
理論上應該是要匹配 `U+1F4A9` 到 `U+1F4AB`。

但實際上卻是...

{% highlight js %}
console.log(/[💩-💫]/);
// SyntaxError: invalid range in character class
{% endhighlight %}

原因是實際上這個正規式長成這樣

{% highlight js %}
var pattern = /[\uD83D\uDCA9-\uD83D\uDCAB]/;
// SyntaxError: invalid range in character class
{% endhighlight %}

我們原本想要的是從 `U+1F4A9` 到 `U+1F4AB` 但現在正規式卻變成先來一個 `\uD830` 然後從 `\uDCA9-\uD83D` 範圍就錯了。

ES6 的 `u` flag 和新版 Unicode 表示法又一次解決了我們的困難

{% highlight js %}
console.log(/[\uD83D\uDCA9-\uD83D\uDCAB]/u.test('\uD83D\uDCA9'));
// true
// 匹配 U+1F4A9

console.log(/[\u{1F4A9}-\u{1F4AB}]/u.test('\u{1F4A9}'));
// true
// 匹配 U+1F4A9

console.log(/[💩-💫]/u.test('💩'));
// true
// 匹配 U+1F4A9

console.log(/[\uD83D\uDCA9-\uD83D\uDCAB]/u.test('\uD83D\uDCAA'));
// true
// 匹配 U+1F4AA

console.log(/[\u{1F4A9}-\u{1F4AB}]/u.test('\u{1F4AA}'));
// true
// 匹配 U+1F4AA

console.log(/[💩-💫]/u.test('💪'));
// true
// 匹配 U+1F4AA

console.log(/[\uD83D\uDCA9-\uD83D\uDCAB]/u.test('\uD83D\uDCAB'));
// true
// 匹配 U+1F4AB

console.log(/[\u{1F4A9}-\u{1F4AB}]/u.test('\u{1F4AB}'));
// true
// 匹配 U+1F4AB

console.log(/[💩-💫]/u.test('💫'));
// true
// 匹配 U+1F4AB
{% endhighlight %}

可惜的是這個方法不能兼容 ES5 和舊環境，如果您真的必須要兼容舊環境那麼您可能只能使用 `regenerate`, 或其他類似的[函式庫](https://github.com/mathiasbynens/regexpu) 來產出那些正規式進行匹配了。


# 真實世界中的其他 Bug 及該如何避免
看到了吧！Unicode 在 JavaScript 中奇怪的行為造成許多問題。許多開發者在處理字串時並沒有考慮到補充字面的問題(舉手; 我就是其一)，甚至包含知名的函式庫 `Underscore.string` 的 `reverse` 也沒有處理關於補充字面產生的問題。

畢竟處理這些問題的確很麻煩，而且好像沒必要???

### 在測試中參一坨屎吧 XD
無論您正在寫什麼樣功能的 JavaScript 試著在測試行為的字串中加入 `💩` 然後看看會不會炸掉。
這有助於您發現 Unicode 的問題。

# 結論
一般處理單一語系的開發者其實不太容易注意到這些問題。這也是在學習 Babel 的過程中為了理解為什麼要特別強調 Unicode 而做些研究寫的學習筆記。


