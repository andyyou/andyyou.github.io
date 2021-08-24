---
title: '[譯]您閱讀的 Nodejs/Express.js 驗證機制教學（很可能)錯了'
tags:
  - javascript nodejs
categories: Program
date: 2017-09-06 10:42:18
---


> 原文：[Your Node.js authentication tutorial is (probably) wrong](https://hackernoon.com/your-node-js-authentication-tutorial-is-wrong-f1a3bf831a46)

最近花了些時間搜尋、研究 Node.js/Express.js 有關驗證機制的教學。大部分的文章要不是不夠完整，要嘛就是在安全性的部分有錯，並可能造成用戶潛在的風險。
這篇文章預計要探討這些常見驗證機制的問題、陷阱，以及點出該如何避免發生。同時也檢視一下自己閱讀或寫作的教學。在撰寫文章的同時我仍然在尋找更穩固的全方位驗證方案（會員功能）。
同時我也希望可以找到 Nodejs/Express.js 中可以和 Rails 的 Devise 匹敵的方案。

<!--more-->

一直以來，在我閒暇的時候一直都在研究、學習各種 Node.js 的教學，似乎每個 Node.js 的開發者都有自己的部落格且習慣分享他們如何`正確的完成任務`，更準確的說是`他們如何完成任務`並不一定是正確的。大量的前端開發者開始需要處理後端的程式，進入了後端的世界，嘗試從散落各地片段的教學和程式碼組合出`能用的東西`，不管是只會複製貼上不懂原理類型的開發者（Cargo Cult Programmer）又或者極度依賴 npm 套件類型的開發者，大家都希望快速的在期限前完成專案。

糟糕的是在 Node.js 世界裡，實作驗證機制某種程度上其實是對開發者的考驗，大部分的開發者都是在摸索中完成功能的，您大概能找到的解決方案就是 [Passport](http://passportjs.org/)。如果我們想要更完整的方案，類似 Ruby on Rails 的套件 Devise 也只能勉強參考 [Auth0](https://auth0.com/) 這樣的服務。
比起 Devise，Passport 只是一個簡單的中介軟體（Middleware），除了驗證之外並沒有提供其他完整驗證機制需要的功能，意思是說 Node.js 的開發者需要自己處理 API token 的機制，重設密碼，更新 token，驗證機制，路由設計，甚至是樣板的部分。有很多的教學在介紹如何使用 Passport 但它們幾乎都有些錯，也沒有完整的實作和處理一個網頁應用程式應該要具備的完整功能。

> 這裡並不是要跟其他提供教學的作者引戰，不過我們的確會透過他們的範例來說明一些安全性議題上的錯誤。如果您也是某篇教學的作者歡迎您在更新您的教學之後和[原文作者聯絡](https://hackernoon.com/your-node-js-authentication-tutorial-is-wrong-f1a3bf831a46)。讓我們一起為了 Node 的安全性作些貢獻。

# 容易犯錯的地方 1 -  憑證儲存 Credential Storage

讓我們從儲存憑證開始。儲存及使用憑證來核對對於認證身份是一種非常基本的方式，這裡說憑證可能比較術語一些，其實就是密碼的角色。過去我們比較常在資料庫或應用程式中處理。而 passport 作為一個 middleware 它其實只幫我們執行 "這個使用者可以放行" 或者 "不行喔！這個使用者沒有權限"的判斷。所以 passport 需要搭配使用驗證機制（策略）像是 `passport-local` 模組來完成儲存憑證的部分即協助處理存資料庫的部分，此時驗證的憑證就是存在資料庫的密碼。passport-local 也是 possport 作者寫的 strategy。

> strategy 翻成策略，大略來說就是`如何執行驗證的方式，憑證怎麼來？`像是從資料庫的會員資料驗證或者 Facebook 登入等方式。

在我們更深入之前，我們可以參考一份 OWASP 的建議說明 [great cheat sheet for password storage](https://www.owasp.org/index.php/Password_Storage_Cheat_Sheet)，總結來說就是使用 salt （鹽）加密並儲存高度複雜性不可逆的密碼。
[Coda Hale 的寫的如何儲存密碼](https://codahale.com/how-to-safely-store-a-password/)也提供了一些建議，雖然它有些爭議。

> 一些有經驗的開發者可能想要使用 `Argon2` - 密碼雜湊競賽的贏家，而且在 Nodejs 可以[輕鬆的透過函式庫使用](https://github.com/emilbayes/secure-password)。不過比較可惜的是關於 Argon2 的實作文件和資料在 Nodejs 社群中是比較不足的。畢竟這樣的主題針對性比較強，不如直接使用 bcrypt 。

作為一個 Express.js 和 Passport 新手，第一個看的資料便是 passport-local 的範例，感謝提供了 [express 4.x 的範例](https://github.com/passport/express-4.x-local-example)，不過在這個範例中並不支援資料庫，同時它假設我們會寫死一些帳號，在實際應用上沒啥太大的幫助。

這個範例只展示了一個流程架構，範例中的密碼並沒有加密，並且使用明碼儲存，驗證資料的儲存和相關的加密處理更不在它包含的範圍中。

好吧！讓我們 Google 看看有沒有其他使用 passport-local 的教學。首先找到了 RisingStack 的 [Node Hero 系列文章](https://blog.risingstack.com/node-hero-node-js-authentication-passport-js/)，但這篇文章和官方的範例有相同的問題（該文章在 2017-08-07 補上 bcrypt 加密的部分)

可以說我們提到的文章都是挑過的。沒錯，挑過的意思其實就是 Google 搜尋的前幾個結果。有關 passport-local 稍微完整一點的教學是來自 TutsPlus 的[這篇](https://code.tutsplus.com/tutorials/authenticating-nodejs-applications-with-passport--cms-21619)。它使用了  bcrypt 搭配 cost 參數為 10 （定義參數你想要跑多慢，2 的幾次方次）。另外一篇比較值得推薦的教學是來自 scotch.io 的[教學](https://scotch.io/tutorials/easy-node-authentication-setup-and-local) 同樣是使用 bcrypt 搭配成本參數為 8。不過這些成本參數真的太低了，目前大部分的 bcrypt 函式庫預設使用 12。

除了儲存密碼的部分這些教學都沒有實作關於重設密碼等相關功能。

* [補充](https://github.com/lustan3216/BlogArticle/wiki/BCrypt-%E5%8A%A0%E5%AF%86%E6%BC%94%E7%AE%97%E6%B3%95%E7%B2%BE%E9%97%A2%E8%A7%A3%E9%87%8B)


# 容易犯錯的地方 2 - 密碼重置

僅次於密碼儲存問題的就是重設密碼，然而在這個部分卻沒有任何基本的教學說明如何使用 passport 實作。
密碼重置功能實作上有太多地方可能犯錯，但在這方面卻沒有比較完整的教學，其中最常見犯的錯誤：

1. 可預測的 token：重新設定密碼時需要的驗證 token 是個容易犯錯的地方，使用時間戳記來當 token 就是一個常見的錯誤，寧可使用基本的亂數產生器也不要讓 token 容易被推測出來。
2. 錯誤的儲存：在資料庫中使用未加密的重設密碼 token，這意味著一旦資料庫資料外洩，這些 token 等於有效的密碼。使用隨機碼加密來產生長度較長的 token ，這樣也可以防範攻擊者直接從遠端針對 token 暴力破解。 重置密碼的 token 就跟密碼本身一樣重要應該要被謹慎處理。
3. token 不會過期失效：一旦 token 不會過期就表示我們給予攻擊者更多的時間去試。
4. 缺少第二層（階段)的資料驗證：常見的安全問題其實就是一種針對重設密碼的第二層驗證，不過關於[安全問題也是有這個機制的問題](https://www.kaspersky.com/blog/security-questions-are-insecure/13004/)存在。

在驗證的三種方式中

    * something you Know
    * something you Have
    * something you Are

我們知道 Email 是屬於 something you have 你有這個東西才能通過認證，而不像密碼僅屬於 something you know。也因此我們有必要使用這種方式來實作`忘記密碼`可以增加一層保護，於是 Email 在流程上便扮演著非常重要的角色，因為我們只會把 token 寄送到該信箱。

* [Something You Know, Have, or Are ](https://www.cs.cornell.edu/courses/cs513/2005fa/NNLauthPeople.html)

如果您剛接觸處理重設定密碼的功能，建議您看看 [Password Reset Cheat Sheet](https://www.owasp.org/index.php/Forgot_Password_Cheat_Sheet)。現在讓我們回到 Node 的世界。

我們到 npm 上面搜尋看看有沒有人實作`重置密碼`相關的函式庫，我們只找到一個 5 年前出自於大神 substack 的東西。我們都知道 Node 的世界進化的非常快速，5 年前的東西等於是侏羅紀的產物，而且我們還得在雞蛋裡挑骨頭 - [V8 的 Math.random() 是可以被預測的](https://security.stackexchange.com/questions/84906/predicting-math-random-numbers)，所以不應該使用 Math.random 來產 token，而且它也不是使用 passport。

接著，我們到 Stack Overflow 找找，但這邊沒有太多有幫助的資料，不過你大概可以一直找到一間 Stormpath 的公司不斷在相關的問題張貼他們的服務。
不過這些資訊也沒有用了，因為 Stormpath 在 2017-08-17 停止營業了。

好吧！我們回到 Google，似乎有一篇唯一的教學。當我們搜尋 `express passport reset` 所找到的[第一個結果](http://sahatyalkabov.com/how-to-implement-password-reset-in-nodejs/)，教學中包含使用了我們先前介紹的 bcrypt 但是計算成本參數為 5 的確遠小於推薦值，除此之外這篇教學和其他文章相較之下算是比較完整的，在 token 部分使用了 `crypto.randomBytes` 也具備逾期的功能。不過 `#2` 錯誤的儲存 和 `#4` 第二層的驗證這兩點並不符合我們的期待。幸好這兩點所造成風險在有`逾期`限制的情況下降低了些。一般來說如果 token 和 密碼都有加密的話即便資料外洩攻擊者也無法直接偽造加密結果或者使用這些 token。照著上面這篇教學的作法，攻擊者就可以拿著這些未加密的 token 來重設密碼。

# 容易犯錯的地方 3 - API Tokens

API tokens 就是一種憑證，它和密碼、重設密碼的 token 一樣都是機敏資訊。大部分的開發者也都知道這點，對於他們自己的 AWS key, Twitter secret 都很重視，不過當自己編寫程式的時候這樣的觀念似乎卻沒有一併轉過來。

讓我們從使用 [JSON Web Tokens](https://jwt.io/) 來實作 API 的驗證憑證。 jwt 具備無狀態的特性，[支援黑名單功能](https://auth0.com/blog/blacklist-json-web-token-api-keys/)，可包含一些聲明資訊這些特性比起單純只有 key/secret 的設計更加優秀。
可能連 Node 新手都聽過 jwt 也可能看過 `passport-jwt` 這東西然後就直接拿它來實作。無論如何，jwt 看似是每個 Node.js 開發者都認為他們應該使用。
（[Thomas Ptacek 認為 jwt 不好](https://news.ycombinator.com/item?id=13866883)，但恐怕也阻止不了發展的趨勢，在這邊我們先不討論他的觀點）。

我們先 Google `express js jwt` 然後找到 Soni Pandey 的教學 [User Authentication using JWT in Node.js](https://medium.com/@pandeysoni/user-authentication-using-jwt-json-web-token-in-node-js-using-express-framework-543151a38ea1)。不過這篇文章對我們沒有太多幫助，因為這篇教學並沒有使用 passport，於此同時我們也發現這篇教學有關於憑證儲存方面的錯誤。

1. 直接將[私鑰存在檔案庫](https://github.com/pandeysoni/User-Authentication-using-JWT-JSON-Web-Token-in-Node.js-Express/blob/master/server/config/config.js#L13)中
2. 使用對稱式(可逆)加密來存密碼，然後加密的金鑰跟 JWT secret 共用。
3. 使用 AES-256-CTR 來儲存密碼，正確來說使用 AES 這種模式並沒有幫助，不是很確定為什麼要這麼作，但這麼作可以讓加密的內容增加其他彈性的作法。

讓我們回到 Google 搜尋其他教學。我們在 [Scotch 找到一篇還算OK的教學](https://scotch.io/tutorials/authenticate-a-node-js-api-with-json-web-tokens)，不過他用了 passport-local 把密碼儲成明碼。

我們點出的這些教學都不能完全照抄，然後這篇教學除了明碼的問題這篇教學也把整個 mongoose 的 User 物件資料都序列化進 jwt 了。
我們 clone 了 scotch 的教學，照著步驟執行 `http://localhost:8080/setup` 建立 User，接著用 Postman 請求 `/api/authenticate` 來取得 token。

！[](https://cdn-images-1.medium.com/max/800/1*wvb2F4-Rx4I1ji2EJIyXZg.png)

注意：JSON Web Token 是 signed 不是加密，小弟(譯者)認為 sign 簽章其實是一種統一格式的用途，容易對資料進行判斷和過濾，它不能算是加密，也就是說這個 base64 編碼我們是可以反解回明碼資訊的

！[](https://cdn-images-1.medium.com/max/800/1*5KcDyNtIfWXVe9uVUD0A_g.png)

現在，每個人都知道密碼和 token 的逾期資訊還有存在 mongoose 物件上的資料。資料在 http 傳輸過程中，攻擊者就可以用一些側錄工具取得這些資訊。

下一篇教學呢？ [Express, Passport and JSON Web Token （jwt) Authentication for Beginners](https://jonathanmh.com/express-passport-json-web-token-jwt-authentication-beginners/)也是一樣有洩漏的問題，大部分的教學都有一樣的問題，到這邊我已經放棄尋找了。

# 容易犯錯的地方 4 - 限制登入錯誤的頻率

另外上面這些教學都沒提到關於鎖定帳號的部分，沒有這個機制攻擊者可以使用像是 [Burp Intruder](https://portswigger.net/burp/help/intruder_using.html) 這種檢查工具使用字典檔不斷的嘗試登入暴力破解。
鎖定帳號的機制也可以在使用者下一次登入時增加其他登入所需的資訊藉此防範這種攻擊手法。

同時限制登入頻率也可以幫助我們避免不斷被嘗試登入而導致 bcrypt 大量使用浪費 CPU 效能，甚至導致程式 crash。
這個部分沒有找到完整的教學文章，但我們找到許多 Express Middleware 例如：[express-rate-limit](https://github.com/nfriedly/express-rate-limit)，[express-limiter](https://www.npmjs.com/package/express-limiter)，[express-brute](https://github.com/AdamPflug/express-brute)。關於這些函式庫或 Middleware 我無法給予太多建議，甚至我沒有研究它們。一般來說我的建議是使用[反向代理功能](https://expressjs.com/en/advanced/best-practice-performance.html#use-a-reverse-proxy)然後使用 [Nginx 的請求（頻率）限制](https://www.nginx.com/blog/rate-limiting-nginx/) 或負載平衡器來負責這個功能。

# 驗證機制非常困難

我很確定部分文章的作者會說：他只是針對基礎的部分講解！而且不會有人在產品上這麼作。然而我再三強調這是個錯誤的行為，特別是當你寫了篇教學又提供程式碼的時候你可能不知道會影響多少人，畢竟你比那些新手知道更多知識。至少在 Nodejs 的世界裡，如果你還是新手請不要輕易全然相信找到的教學，應該多方謹慎求證，直接複製教學很容易給你的產品帶來災難性的問題。現階段如果你需要一個完整的驗證機制那麼你最好回到基礎一步一步的掌握概念靠自己實作，你可以參考 Rails/Devise 自己一步步實作。雖然 Node.js 看似很容易使用，且大量的開發者在推廣，但不得不說對於那些想要用 javascript 快速開發產品的人來說真的需要注意 Node.js 的世界很多東西的細節還夠不完善，這個社群似乎比較專注那些新穎酷炫的東西。如果你是前端出身的且沒學過其他語言，我個人推薦您選擇 Ruby，借鑑神人們的經驗比較不會自己找自己麻煩。

如果您有在寫教學文，期望您盡可能提供的程式碼可以直接在 production 上使用，至少備註一下該注意什麼。

如果您是死忠的 Node.js 開發者希望您可以從這篇簡短的說明中知道在使用 passport 的時候什麼事情需要避免。這篇文章沒有完整的列出所有該注意的地方，只是起個頭希望您注意一下您的 Express 程式。

# 附錄（2017-08-10 更新）

這篇文章源自於自己發現的問題。現在已收到許多回應，更引起超過預期的爭議。很顯然的這個問題對開發者來說的確是個痛點。如果您還發現其他錯誤，請告知原文作者，我們將會儘快修正，實作是不希望有更多誤導別人的文章。

同時我希望大家不要只是評論這篇文章，讓我們具體的為社群作些貢獻。如果您有發現其他不應該出現在 production 的問題也請您直接告知原文作者。
最後，原作者發起了一個 [GitLab](https://gitlab.com/micaksica/nodeauth-best-practices) 用來紀錄相關的議題。
