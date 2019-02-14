---
title: PUPY噗比 開發日記
date: 2017-02-5 09:25:00
tags: [pupy, dev, react, angular]
---

文／洪晧哲

[![](https://3.bp.blogspot.com/-LCVuEGyzcLk/WJVW5wSkH2I/AAAAAAAABDE/kdoi98B1XeM7AUqkptjH7-k9Fe1unZ0XgCLcB/s320/com_logo%25402x.png)](https://3.bp.blogspot.com/-LCVuEGyzcLk/WJVW5wSkH2I/AAAAAAAABDE/kdoi98B1XeM7AUqkptjH7-k9Fe1unZ0XgCLcB/s1600/com_logo%25402x.png)

前言
--

這是無名小站關站以來第一次寫Blog，想說開始來記錄一下我的開發跟學習過程。這篇裡面寫到了很多技術上的名詞，寫得內容不深也不淺，也不知道寫給誰看，不過就記錄一下吧。

PUPY噗比(<https://pupy.tw/>) 是一個2015年我跟我的好朋友開始一起做的寵物認養平台，目的是建立一個快速方便的寵物送養、認養平台，給大家可以非常快速的找到適合自己的寵物。

前一年(2015)我用[Angular.js](https://angularjs.org/)（第一版）做了電腦版的網站。後來發現Angular在手機上的效能不佳，所以打算整個用現在最流行的[React.js](https://facebook.github.io/react/)來重作。但是怕整個全部重做花太長時間，網站會難產，所以先做手機板部分，之後再慢慢把整個全部改成用React.js實作。

正文開始啦
-----

### 環境(?)

如前文所述，寫這篇文章的時候，給電腦版的網頁早就做好了，但是手機板的網頁因為各種原因遲遲沒有出來。為了不要讓網頁難產，也讓以後開發網頁更快速更有系統，我開始學習用React.js來做前端，[Node.js](https://nodejs.org/)來做後端。不過這樣說有點太簡略，這邊先來寫寫整個環境。

React.js使用一個叫做JSX的JS擴充語言，用來更快速的表示React的Component。這當然不是瀏覽器原生支援的語言，所以要先經過[babel](https://babeljs.io/)編譯才能在瀏覽器正常使用。
在React裡面又有很多地方需要用到ES6, ES7的功能，寫起來才會比較簡潔快速，所以也是要經過babel編譯回ES5。目前前端沒有用太多額外的Library，所以大概就先這樣。

後端的話如剛剛說到的是用Node.js來寫，而資料庫則是使用[MongoDB](https://www.mongodb.com/)（因為他的資料儲存格式可以讓開發很方便快速）。網頁伺服器是用很流行的[express.js](http://expressjs.com/) Module。為了解決JS Callback Hell的問題，我前陣子花了時間把幾乎所有的Async Call都改寫成Promise的作法，搭配ES7的Async, Await語法，整個寫起來變得非常的乾淨。這邊很多東西也是Node.js還沒有完全支援的，所以我也是乾脆都透過babel把他編譯回ES5來執行。

資料庫（MongoDB）其實不是用原來的Driver，而是使用[mongoose](http://mongoosejs.com/)這個ODM（Object Document Mapping）Module來寫，整體的操作都變得跟JS的物件一樣，非常淺顯易懂。

可以看到很多東西都是透過babel編譯後才能執行，我當然不是每次都寫完手動拿去編譯再來執行啦。後面其實是用[gulp](http://gulpjs.com/)這套系統來管理發布流程。每次有更動都會自動叫babel去編譯更動後的檔案，然後有個[livereload](https://github.com/vohof/gulp-livereload)的module來幫我自動重新整理前端。伺服器端就是在手動關掉開起來就好。也許以後可以再特別寫一篇講解整個gulp腳本。有興趣的可以看看gulp這個Module。

#### 開發

目前手機板的進度已經做到大概一半了，剩下帳號設定、問題回報還有新增文章沒做。目前做下來其實都蠻快的，因為Angular跟React都一樣是把一個一個Component分出來實作的，只是資料傳遞的方式不同。跟React一樣，Angular的每個Controller都有自己的State，但是React沒有Angular的Service，所以要共用整個App的最上層State時會變得很麻煩，不太可能都一層一層的把State用Props往下傳。所以就去爬文發現了有Flux這個開發模式，配合Provider配合React的Context功能會讓資料傳遞變得非常容易，基本上就是只要有用connect的Component都可以輕鬆存取到Redux的唯一State樹。

Redux的開發模式有提到一個原則，整個App只有唯一一個State樹。但我覺得這是有點不切實際的。有很多小Component其實可以自成一個完全獨立的Component，但依照這個開發模式，它們都必須把自己的State放在App唯一的State樹底下。這讓整個開發過程變得很繁瑣。所以我還是讓很多Component如果沒有跟App狀態相依，都讓他們有自己的State。這個問題可能之後還要花時間爬個文看看大家是怎麼處理的。

### 幾個問題

現在正在思考幾個問題。

首先，上傳圖片的功能要怎麼實作，因為手機的硬體資源比較寶貴，如果像網頁版一樣在客戶端做自動切圖然後順便製作預覽圖放在畫面上，低階手機大概會受不了吧？也許這些工作都應該丟給伺服器去做。

用網頁伺服器Serve圖片似乎會影響整體的效能，也許應該找個CDN服務來儲存使用者上傳的圖片，這也是我第一次從頭到尾設計一整套系統，對這種東西也是一知半解，感覺還有很多東西要去摸索呢。

因為之前小測試後發現許多人不喜歡使用Facebook帳號登入，而我們當初又是設計只有用Facebook登入。所以之後還要放入使用Email申請帳號，這就要來做Email驗證功能，要在Godaddy提供的虛擬主機跟網域上做發信功能似乎很麻煩。也許可以用Gmail代發，但是發信網域不是來自@pupy.tw就覺得不太舒服呢...

P.S.
----

這個網站的所有設計(像是上面那個可愛的Logo)都是出自設計師XunYu之手呦，歡迎到她的blog看看 [XUN DESIGN](http://xundesign.blogspot.tw/) ～