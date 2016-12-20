---
title: 撰寫高效能的 Javascript 小技巧
categories: Program
date: 2016-12-19 15:28:45
tags: javascript
---

這篇文章將會介紹簡單的技巧來優化我們的程式碼讓 Javascript 編譯的過程更具效率，最終我們的程式碼可以執行的更加快速。
特別是當您在遊戲類型的專案上發現掉禎或記憶體回收機制(garbage collector)遇到大量資料無法被回收的情況。這些技巧可以協助我們增加程式碼的效能。

<!--more-->

# 單態(Monomorphism)

當我們定義一個函式搭配兩個參數的時候，如果函數的參數型別，數量，傳回的型別改變編譯器會遵循我們的指令，但效能便會開始下降。因為一般程式會預期一個單態的資料結構且相同的參數。

```js
function example(a, b) {
  console.log(++a, ++b)
}

example() // 糟糕
example(1) // 還是糟糕
example("1", 2) // 很糟糕

example(1, 2) // 良好
```

# 展開(Unfolding)

編譯器可以在編譯時期解析變數的值並且在最好的情況下可以將其展開，盡可能在程式實際執行前解析出其資料。
常數和變數只要是不需要等到執行時期才能夠計算的值都會被展開。

```js
const a = 42; // 能夠被簡單的展開
const b = 1337 * 2; // 表達式也能夠被計算
const c = a + b; // 可以被解析計算
const d = Math.random() * c; // 只有 c 可以被展開，Math.random() 要等到執行時期才能得到值
const e = "Hello " + "Medium"; // 字串格式也是可以被展開的

// 展開之前
a;
b;
c;
d;
e;

// 展開之後
// 在編譯時期就可以得到下面的結果
42
2674
2716
Math.random() * 2716
"Hello Medium"
```

# 內嵌(Inlining)

JIT 編譯器可以知道我們程式碼哪些段落特別常被執行。透過拆分 function 為更小的片段程式碼，就可以在編譯時期將其內嵌並追蹤特別常被使用的 function 同時提升效能。

```js
// Likely gets inlined 可能被內嵌的條件
// [✓] Single return statement 回傳單一語句
// [✓] Always returns 總是回傳
// [✓] Monomorphic return type 回傳單態型別
// [✓] Likely monomorphic parameters 可能是單態的參數
// [✓] Single body statement 單一語句
// [✓] Isn't wrapped inside another function 內部沒有其他 function
// 等等
function isNumeric(n) {
  return (
    n >= 48 && n <= 57
  );
};

let cc = "8".charCodeAt(0); // 56

// before inlining 內嵌之前
if (isNumeric(cc)) {

}

// after inlining 內嵌之後
if (cc >= 48 && cc <= 57) {

}
```

# 宣告

避免宣告函式/閉包和物件在經常被調用的任務中。物件會被堆進 heap 這會影響記憶體回收造成回收上的困擾。

```js
// 糟糕

function a () {
  // 請避免在 function 裡面宣告 function
  // 如此在每一次被調用時都在配置一次
  let doSomething = function () {
    return (1)
  }

  return (doSomething())
}

// 較好的作法

let doSomething = function () {
  return (1)
}

function b() {
  return (doSomething());
}
```

# 引數(Arguments)

調用函式是很耗費效能的。盡可能減少直接使用 arguments 並且不要在函式內部修改它們

```js
function mul(a, b) {
  return (arguments[0] * arguments[1]); // 非常慢
  return (a * b); // 良好
};

function test(a, b) {
  a = 5; // 糟糕, 不要直接修改引數
  let tmp = a; // 良好
  tmp *= 2;
};
```

# 資料型別(Data Types)

盡可能使用 Number 和 Boolean 他們相對於其他資料型別快非常多。

```js
const ROBOT = 0;
const HUMAN = 1;
const SPIDER = 2;

let E_TYPE = {
  Robot: ROBOT,
  Human: HUMAN,
  Spider: SPIDER
};

// 糟糕
// 在常被使用的函式或程式碼中避免直接使用字串來判斷
if (entity.type === "Robot") {

}

// 良好
if (entity.type === E_TYPE.Robot) {

}

// 完美
if (entity.type === ROBOT) {

}
```

# 嚴格與轉換型別運算子(Strict and abstract operators)

盡量使用 `===` 而不是 `==` 由於嚴格比對模式下編譯器不需要執行額外的轉換因此效能會比較好。

# 致命傷

下列功能盡可能避免使用會造成嚴重的效能問題

* eval
* with
* try/catch

# 物件

物件實例通常會分享一個隱藏且相同的類別，注意在替物件實例加上新的屬性時會建立一個新的隱藏類別，這件事對編譯器來說挺複雜的。

```js
// 隱藏的類別 'hc_0'
class Vector {
  constructor(x, y) {
    // 編譯器預期屬性成員在這邊被宣告了
    this.x = x;
    this.y = y;
  }
};

// 兩個 vector 物件共用隱藏類別 'hc_0'
let vec1 = new Vector(0, 0);
let vec2 = new Vector(2, 2);

// 糟糕, vec2 現在產生了新的隱藏類別 'hc_1'
vec2.z = 0;

// 良好, 編譯器已知的屬性
vec2.x = 1;
```

# 迴圈

盡量使用單一型別的陣列。減少使用 `for .. in` 來遍歷物件，效能很差。

在迴圈中 `continue` 和 `break` 效能比 if 條件式要快。盡可能讓迴圈要處理的資料在外頭。另外使用 `++i` 而不是 `i++` 這可以得到一點點的效能提升。

```js
let badarray = [1, true, 0]; // 糟糕, 盡量不要混合型別
let array = [1, 0, 1]; // 良好

// 糟糕
for (let key in array) {

};

// 較好的作法
let i = 0;
for (; i < array.length; ++i) {
  key = array[i];
};

// 良好
let i = 0;
let key = null;
let length = array.length;
for (; i < length; ++i) {
  key = array[i];
};
```

# 參考資料

* [Writing efficient JavaScript](https://medium.com/@xilefmai/efficient-javascript-14a11651d563#.igf03p8sb)
