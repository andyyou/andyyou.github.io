---
layout: post
title: '快速 JS 筆記'
date: 2015-04-28 05:30:00
categories: Program
tags: [javascript]
---

基本的 Javascript 型別定義有 6 個

* Number
* String
* Boolean
* Function
* Object
* Undefined

<!--more-->

如果把比較特殊一點的做歸納整理則如下


* Number （數字）
* String （字串）
* Boolean （布林）
* Object （物件）
    - Function （函式）
    - Array （陣列）
    - Date （日期）
    - RegExp
* Null （空）
* Undefined （未定義）

---

字串可以用 `+` 串接，然後比較特別的是 backslash `\` 跳脫字元。

不是所有的 operator 操作子都是符號，例如 `typeof` 就不是，他是一元的操作子

---

Javascript 中只有 `NaN` 不會等於 `NaN`

~~~js
NaN == NaN // #=> false
~~~

NaN 表示不合理的計算 Not a Number

---

`||` 處理優先權最小，再來是 `&&` 然後則是比較子 `>`, `==`, `<` 等等

---

Javascript 討厭的地方在於自動轉型常常會導致誤會

~~~js
console.log(8 * numm);
// 0

console.log("5" - 1);
// 4

console.log("5" + 1);
// 51

console.log("five" * 2);
// NaN

console.log(false == 0);
// true

console.log(null == undefined);
// true

console.log(0 == null);
~~~

---

如果要測試一個值是否存在可以簡單的直接拿該值跟 `null` 或 `undefined` 比較即可

~~~js
5 == null
// false
~~~

此外 JS 還對一些值轉換成 Boolean 有自己的規則 `0`, `NaN`, `""` 空字串

~~~js
0 == false
// true

"" == false
// true

NaN == false
// false

if (!NaN) {
  console.log("In this caes NaN equal to false")
}
// In this caes NaN equal to false

~~~

總結在單純 if expression 中 `0`, `NaN`, `""`, `undefined`, `null` 會等於 false 其他值都是 true。
不過注意 `" " == false`, `0 == false` 會是 `true`，如果不想自動轉型請用
`===` 和 `!==`

---

`&&` 和 `||` 的行為是會先將兩側的操作元轉成 Boolean (true, false)，不過奇特的是當比完時結果會從兩側來取。

舉例來說 `||` 當他能夠先將左側的值轉成 true 時他就先回傳左側的值，否則就傳右邊的值。

~~~js
null || undefined
// undefined

"foo" || "bar"
// foo

Infinity || true
// Infinity
~~~

這個功能讓我們能夠透過 `||` 來實現預設值的功能，當左邊沒值的時候採用右邊的預設值。

`&&` 功能也類似，當左邊的值能被轉成 `false` 的時候，就回傳左邊的值，否則就傳回右邊。

另外 `&&` 和 `||` 有一個重點行為是只有當需要時才評估右邊的值，舉例如果 `true || X` 此時就不需要評估 X 直接回傳 true。

---

### 小結判斷規則
0, 空字串, NaN, undefined, null 在遇上自動轉型成 Boolean 時都是 false 不管您是使用一般 if 表示式或者 &&, || ，不過如果遇到 `== false` 操作子時只有 0 和 空字串成立


---

一個片段程式碼產生一個值稱之為 expression
expression 類似語言中的片語，一個短句。
statement 則是一句完整的句子，在 JS 中用 `;` 結束當作一個句子。

通常一個 statement 是獨立的，只會完成某項任務，不過如果它影響了整個程式例如: 異動了機器內部的狀態，或者影響後面的 statement，這些造成的改變我們就稱為 side effect (副作用)

---

# 關鍵字

~~~
break case catch class const continue debugger
default delete do else enum export extends false
finally for function if implements import in
instanceof interface let new null package private
protected public return static super switch this
throw true try typeof var void while with yield
~~~

# in 的用法

在 JS 中如果要檢查一個物件有沒有 property 我們可以使用 `in` 來檢查

~~~js
var car = {
  name: 'Toyota'
}

console.log("name" in car);
// true

console.log("brand" in car);
// false

// 也可以在 for 裡面遍歷所有的屬性

for (var property in object) {
  console.log(property);
}

~~~


# 屬性
一般來說物件的屬性名稱必須要是合法的變數名稱。使用 object.property `.` 的方式取得。不過 JS 有特殊的取法可以用 `[]` 來取，因為陣列也是一種特殊的物件，同時語法允許我們透過索引和不符合變數命名規則的方式來取得

~~~js
var person = {
  "first name": "You",
  1: 2
}

console.log(person["first name"]);
// You

console.log(person[1]);
// 2
~~~

# function 中的 arguments
arguments 是一個類陣列物件，它並不是陣列，只是類似但並不包含陣列的 methods 除了 length 外

~~~js
// 如果要用陣列的 methods
Array.prototype.slice.call(arguments);

function func() {

}

func.apply(context, array);
func.call(context, 1, 2, 3, 4);

// 判斷物件是不是陣列
Object.prototype.toString.call( anObject )
// [object Array]
// [object Arguments]

[].concat( anObject );
Array.isArray( [] )
// Check an object is array

~~~

>apply 要丟陣列，丟物件會無法取得 arguments，因為 arguments 是一個類 array 的物件，可以想成它是從 array 轉型而來，砍掉一些 methods

~~~js
function Call() {
  console.log("arguments type: ", Object.prototype.toString.call(arguments));
  for(var i = 0; i < arguments.length; i++) {
    console.log(arguments[i]);
  }
  console.log("name: ", arguments["name"]);
  console.log(arguments.toString());
}

// Call.apply(null, {name: "andy", age: 28});
// Call.apply(null, [1, 2, 3]);
~~~

# 高階 function 總結筆記
一個大型的程式是昂貴的，不只是因為需要花比較多的時間來建置
大小永遠牽扯到複雜度，越複雜越容易造成開發者困惑，反過來說也就容易產生 bug，越大的程式越容易躲藏 bug 且不好找。

~~~js
var total = 0, count = 1;
while(count <= 10) {
  total += count;
  count += 1;
}
console.log(total);

// 抽象化後
console.log(sum(range(1, 10)));
~~~

抽象化一詞代表將瑣碎的程式片段包裝成有語意的 function 或 method 讓開發者可以更清楚其代表的意義。

在程式的執行環境裡這些有意義的單字詞彙通常就稱為抽象化。
抽象化把實際執行的細節隱藏起來，讓我們能夠用比較高階的方式直接處理問題


JSON.stringify() // -> 轉成文字
JSON.parse() // -> 轉成物件

`forEach` 直接把陣列元素傳入另外一個 function
`filter` `map` 都會產生新的陣列
`reduce` 合併一個新值
`bind` 會產生另外一個 function

~~~js
var arrays = [1, 2, 3, 4];

arrays.reduce(處理函式(結果, 每次帶入的元素), 結果變數的初始值)

arrays.reduce(function(total, num){
  total += num;
}, 10);

[1, 2, 3].reduce(function(total, num){return total += num}, 10);
// 16


// 常用的示範
function average(array) {
  function plus(a, b) { return a + b; }
  return array.reduce(plus) / array.length;
}

var byName = {};
ancestry.forEach(function(person) {
  byName[person.name] = person;
});

var ageOfMothers = ancestry.filter(function(person) {
  return byName[person.mother] != null
}).map(function(person) {
  return person.born - byName[person.mother].born;
});
console.log(average(ageOfMothers));
~~~

Methods 單純就是一個屬性其 value 是一個 function
當一個 function 被當成 method 且被呼叫調用時 `object.method()` 會產生一個特殊的變數 `this` 在該 function 的 block 中(Javascript 沒有 block 的概念，應該說是 context) 這個 this 會指向該物件。

~~~js
function speak(line) {
  console.log("The" + this.type + " rabbit says: " + line );
}

var whiteRabbit = {type: 'white', speak: speak};
var fatRabbit = {type: 'fat', speak: speak };

whiteRabbit.speak("Oh my ears and whiskers");
~~~

`bind` `apply` `call` 的第一個參數可以簡單地想成設定 `this` 物件值

在 JS 中物件搜尋屬性會先從物件本身開始，如果找不到則找物件的祖先，就是物件的 `prototype`

~~~js
console.log(Object.getPrototypeOf({}) == Object.prototype);
// true

console.log(Object.getPrototypeOf(Object.prototype));
// → null

console.log(Object.getPrototypeOf(isNaN) ==
            Function.prototype);
// → true
console.log(Object.getPrototypeOf([]) ==
            Array.prototype);
// → true
~~~

所有 JS 中物件繼承的根 root 就是 `Object.prototype`
它提供了一些方法給所有的物件使用，例如 `toString`

`Obejct.create` 可以建立物件並指定繼承的對象

~~~js
var protoRabbit = {
  speak: function(line) {
    console.log("The " + this.type + " rabbit says '" +
                line + "'");
  }
};
var killerRabbit = Object.create(protoRabbit);
killerRabbit.type = "killer";
killerRabbit.speak("SKREEEE!");
~~~

另外一個建立物件更方便方式是使用 `constructor` 因為目前 JS 沒有 class 的概念。在 JS 使用 new 搭配 Function 會產生類似 constructor 的效果。建構子會將該物件綁定 this 變數上。當我們使用 new Function() 時會回傳一個新的物件。其意義為請根據建構子回傳一個物件實例。

~~~js
function Rabbit(type) {
  this.type = type;
}

var blackRabbit = new Rabbit("black");
blackRabbit.type;
// black
~~~

建構子(在 JS 中也就是所有 function) 會自動取得一個 prototype 屬性且預設會帶一個單純的空物件，繼承自 `Object.prototype` 這個物件(即複製一個一樣結構的新物件到 prototype)
然後每一個透過該`建構子`產生的物件實例(instance)都會有上面說的 `prototype` 物件，也就是會共享同一個物件來完成`繼承`這件事
之後修改 prototype 每個用該 function 產生的物件實例都可以取用該物件的屬性


`toString` 用在一般物件上會顯示 `[object Object]` 的字串資訊但是用在 Array 上則會像使用 `join(',')` 的效果。

因為覆寫的關係所以 Array 的 toString 和 Object 的不同
我們可以用

~~~js
console.log(Object.prototype.toString.call([1, 2]));
// -> [object Array]

[1, 2].toString();
// 1,2
~~~

我們現在可以透過 function 的建構子機制來建立物件，然後透過 prototype 來分享一些方法。但是這有一個問題，在上面我們曾經透過
下面這種方式把資料存進物件中

~~~js
var map = {};
function storePhi(event, phi) {
  map[event] = phi;
}

storePhi("pizza", 0.069);
storePhi("touched tree", -0.081);
~~~

會發生一種問題是在 prototype 裡面加入的 methods 也會被輸出

~~~js
Object.prototype.nonsense = "hi";
for (var name in map)
  console.log(name);
// → pizza
// → touched tree
// → nonsense
console.log("nonsense" in map);
// → true
console.log("toString" in map);
// → true

// Delete the problematic property again
delete Object.prototype.nonsense;
~~~

但是我們發現 `toString` 並沒有被 `for in` 輸出
這是因為 JS 會區分 `enumerable` 和 `nonenumerable` 屬性，我們自己透過 prototype 加進去的屬性會被迭代屬於 enumerable 但是預設在 Object.prototype 裡面的不會，屬於 `nonenumerable`

要定義 `nonenumerable` 的屬性

~~~js
Object.defineProperty(Object.prototype,"hiddenNonsense",{enumerable: false, value: "hi"});
~~~

透過這種方式屬性被加進去了，但是不會被 for/in 取得。
不過當我們用 `"hiddenNonsense" in object` 時還是會出現 `true` 為了區分屬性是物件自己的還是 prototype 的，我們有 `hasOwnProperty(name)` 這個方法來判斷。

當我們覺得問題都解決的時候，事實上還有一種狀況，就是開發者覆寫了 `hasOwnProperty`，在上面這種情況我們只想單純取得物件本身的屬性而不要 prototype 的東西，此時我們可以透過 `Object.create(null)` 這樣一來就沒有任何 `prototype` 的屬性了

或者 `Object.hasOwnProperty.call(obj, "property")`

~~~js
var map = Object.create(null);
map["pizza"] = 0.069;
console.log("toString" in map);
// → false
console.log("pizza" in map);
// → true

// 除了上面這種方式，當 hasOwnProperty 被 override 之後我們仍可以用這種方式確保方法
Object.hasOwnProperty.call(map, "pizza");
// true
Object.hasOwnProperty.call(map, "toString");
// false
~~~


當您呼叫 `String()` 函式時他會把傳進去的值轉成字串，透過 `toString` 這個方法，注意之前有提到有些物件的 toString 方法和預設並不相同，如 Array


我將透過稍微複雜的範例試圖解釋關於 polymorphism 以及一般物件導向的觀念。在下面這個專案我們將會寫一段程式
透過表格中表示欄位(Table cell)陣列中的陣列資料，建立一個字串。這個字串包含著一個表格格式。
意思是透過欄和列來呈現，列同時也會對齊。
看起來就像下面這樣

name         height country
------------ ------ -------------
Kilimanjaro    5895 Tanzania
Everest        8848 Nepal
Mount Fuji     3776 Japan
Mont Blanc     4808 Italy/France
Vaalserberg     323 Netherlands
Denali         6168 United States
Popocatepetl   5465 Mexico

表格系統建立的方式將會透過建置函式取得每一個 cell 的寬高
透過偵測一欄的寬度以及列的高度來畫出正確的尺寸接著組合它們變成一個字串

layout 程式會透過我們已經定義好的介面跟 cell 物件溝通
進一步說透過這種方式 cell 將會是動態的，後續我們將會再加入一些樣式例如 header 的底線，同時我們也不用再修改 layout 部分的程式碼

下面是我們定義的介面
minHeight() 傳回 cell 最小需要的高度(文字行數)
minWidth() 傳回 cell 最少需要的寬度(文字字數)
draw() 回傳一個 `height` 長度的陣列其中包含一系列的文字，這些文字也是代表 width 的字元寬。這是用來表示 cell 的內容

這邊將會大量使用高階的陣列方法，因為這也是比較好的方式

首先第一個部分是計算最小欄位寬的陣列組和列高的陣列組
rows 變數會保存一個二維陣列組，內部的陣列代表一列
rows 是一個陣列 裡面的 row 也是陣列
[ row, row, row, [// It's a row] ]

~~~js
// 從 rows 裡面計算出每一個 row 的高度
function rowHeights(rows) {
  return rows.map(function(row){
    return row.reduce(function(max, cell) {
      return Math.max(max, cell.minHeight);
    }, 0);
  });
}

function colWidths(rows) {
  return rows[0].map(function(_, index) {
    return rows.reduce(function(max, row){
      // 一次先比每一個 row 的 col-inex 比出誰比較寬
      // 最少需要多寬，就是取最大的寬
      return Math.max(max, row[index]);
    }, 0);
  });
}
~~~

當我們對變數名稱使用 `_` 開頭或者整個變數都是由 `_` 組成是讓我們人看到的時候可以直接知道這個變數將不會使用

`rowHeights` 應該不難理解，透過使用 `reduce` 方法去找出每一 row 陣列中最大的高，再透過 map 把這些高及合成一個陣列 `heights`

`colWidths` 函式比較起來稍微難一點，因為最外層是使用一個 row 的陣列來取得欄位的數量，並不是直接用。剛剛忘記提到 `map` `forEach` `filter` 和其他類似的陣列方法都可以再傳入第二個參數，這個參數會取得當前元素的索引。如此一來我們就可以只透過 index 表示目前到了第幾個欄位 colWidths 類似 rowHeights 目的是要取得所有欄位最寬的值


~~~js
function drawTable(rows){
  var heights = rowHeights(rows);
  var widths = colWidths(rows);

  function drawLine(blocks, lineNo) {
    return blocks.map(function(block) {
      return block[lineNo];
    }).join(" ");
  }

  function drawRow(row, rowNum) {
    var blocks = row.map(function(cell, colNum) {
      return cell.draw(widths[colNum], heights[rowNum]);
    });

    return blocks[0].map(function(_, lineNo) {
      return drawLine(blocks, lineNo);
    }).join("\n");
  }

  return rows.map(drawRow).join("\n");
}
~~~

`drawTable` 函式使用內部輔助函式 drawRow 來畫出所有列然後把這些 rows 串在一起每一列之間隔一個 `\n`

drawRow 函式先把 row 陣列中的 cell 物件轉成 blocks
一 row = 一 blocks 現在 blocks 使用字串的陣列來表示 cell 的內容，一個 row 對應一個 blocks 陣列。
而裡面原本的單一 cell 例如本來是  cell.height 的值是 3776 現在就會換成 ["3776"] 來呈現，如果要加上底線則可能會像這樣 ["name", "----"]

一列的 blocks 擁有一樣的高，應該會彼此相連直到最後

>接著在 drawRow 中呼叫 map （這邊看不懂）

~~~js
function repeat(string, times) {
  var result = "";
  for(var i = 0; i < times; i++) {
    result += string;
  }

  return result;

}

function TextCell(text) {
  this.text = text.split("\n");
}

TextCell.prototype.minWidth = function() {
  // 格子內的文字會先用 `\n` 拆成一個陣列，一行一個 element
  return this.text.reduce(function(width, line) {
    return Math.max(width, line.length);
  }, 0);
};

TextCell.prototype.minHeight = function() {
  return this.text.length; // 幾行
}

TextCell.prototype.draw = function(width, height) {
  var result = [];
  for (var i = 0; i < height; i++) {
    var line = this.text[i] || "";
    result.push(line + repeat(" ", width - line.length));
  }  
  return result;
}
~~~



`Object.keys` 會把屬性輸出成一個陣列，當然這跟上面提到的一致 nonenumerable 的屬性不會出現。

blocks -> [["name", "-----"], ["height", "------"]]

rows Array[5]
[
  TextCell,
  TextCell,
  TextCell,
  TextCell,
  TextCell
]

TextCell.text (Array)


# Array 常用

~~~js
// 複製一份新的陣列
var clone = array.slice(0);

// 長度
array.length

// 排序 小->大
array.sort(); // 有副作用，NO-COPY

// 排序 大->小
array.reverse(); // 有副作用，NO-COPY

// 串連，合併陣列
array.concat(another_array); // COPY

// 搜尋陣列中的元素
array.indexOf("1"); // return index 找不到回傳 -1

// 把陣列的元素串成一個字串
array.join();

// 從尾巴搜尋回來
array.lastIndexOf("b");

// 從尾巴移除元素
array.pop();

// 從尾巴加入元素
array.push(ele);

// 從頭移除元素
array.shift();

// 從頭加入元素
array.unshift(ele);

// 回傳陣列的參考，意為賦予的新變數記憶體是同一個位址
array.valueOf();

// 選擇部分元素，並回傳一個新的陣列
array.slice(1, 3); // COPY
array.slice(從 index 元素開始, 到這個 index 之前一位)
array..slice(-3, -1); // 可以用負數，最後一位是 -1 往回算

// 刪除陣列中的元素，副作用 NO-COPY
var o = [1, 2, 3, 4];
delete o[2];
// #=> [ 1, 2, <1 empty slot>, 4 ]

// 刪除並且新增元素
array.splice(1, 0, ele1, ele2 );
array.splice(index, 刪除幾個, 加入元素1, 加入元素2); // 可用負數，回傳的是被刪掉的值的陣列，會從 index 開始塞資料

// map
var numbers = [1, 4, 9];
var doubles = numbers.map(function(num) {
  return num * 2;
});
// #=> doubles is now [2, 8, 18]. numbers is still [1, 4, 9]

// 是否是陣列
Array.isArray()

// 如果陣列中所有元素都滿足就回傳 true 否則 false
[1, 2, 3].every(function(element, index, array){})
// #=> true, false
// element 當前元素, index 當前索引, array 整個陣列

// 只要有一些符合就 true
array.some(function(element, index, array){});

// 過濾並回傳新元素
array.filter(function(element, index, array){ return })

// 陣列遍歷
array.forEach(function(element, index, array) {})

// 元素一個一個濃縮合併
[0, 1, 2, 3, 4].reduce(function(sum, current, index, array){}, init);
array.reduceRight(); // 從右邊到左邊

// keys 陣列元素一個一個取
var arr = ["a", "b", "c"];
var iterator = arr.keys();

console.log(iterator.next()); // { value: 0, done: false }
console.log(iterator.next()); // { value: 1, done: false }
console.log(iterator.next()); // { value: 2, done: false }
console.log(iterator.next()); // { value: undefined, done: true }

// 搜尋元素
array.includes(searchElement[, fromIndex])


~~~
