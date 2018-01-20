---
title: BigSushi - 在 HTML 5 Canvas 設計出 2D 座標 Render 系統
date: 2017-03-26 13:19:00
tags: [Dev, BigSushi]
---
[![](https://3.bp.blogspot.com/-o9WkWOBeroY/WMgI6lBxFxI/AAAAAAAABEI/d36P8wXambcaYXfJOWr1eRICFO8nhdgkQCLcB/s400/Untitled.png)](https://3.bp.blogspot.com/-o9WkWOBeroY/WMgI6lBxFxI/AAAAAAAABEI/d36P8wXambcaYXfJOWr1eRICFO8nhdgkQCLcB/s1600/Untitled.png)

這學期的物件導向程式設計實習課，老師要我們用實驗室做的HTML 5基本遊戲Framework做一款遊戲出來。這個Framework做出了基本的Game Loop，但有很多東西是不完整的。因為我之前長時間使用Unity遊戲引擎的關係，我看了Framework的基礎架構後馬上就想把他改到用起來跟Unity幾乎一樣。

這篇文章會寫到我如何在 HTML5 的 Canvas 實作像 Unity 一樣的 Render 方式。也就是**場景中的每個物件都自成一個座標系統，而每個物件都可以包含多個子物件。**只要將物件的父子關係設定好，系統就會自動算出物件的**世界座標**（**World Space Coordinate**）並且根據物件與**攝影機**的相對位置在螢幕的**螢幕座標**（**Screen Space Coordinate**）上繪圖。

### GameObject 與 Transform

首先講到實作物件的父子關係系統，這在 Unity 裡面稱作 **Transform**，Transform包含座標、旋轉、縮放還有父子物件。而表示場景中物件的叫做**GameObject**。GameObject包含許多Components（以後會介紹到，有空的話😄）和一個 Transform。由於也只有 GameObject 會有 Transform，所以我直接將 Transform 跟 GameObject 做在一起，就叫做 **GameObject**。

所有的 GameObject 都會有：

-   Parent (父 GameObject)
-   Children (子 GameObjects)
-   Position / AbsolutePosition (相對座標 / 絕對座標)
-   Rotation / AbsoluteRotation (相對旋轉 / 絕對旋轉)
-   Scale / AbsoluteScale (相對縮放 / 絕對縮放)

絕對是指GameObject相對整個世界原點的值，而相對則是它相對於其Parent的值。例如**你在我右邊7公尺（你對我的相對 x 是1m）**，而我在世界原點（？）的**右邊9410公尺**，所以**你的絕對位置就是 x: 9417m**。
座標、旋轉及縮放都是直接以相對位置儲存的，而他們的絕對數值則是透過計算計算出來的（這對我來說有點違反直覺，不過由於這個Framework原本就是存相對座標，所以我就沿用了）。

計算旋轉與縮放比起座標簡單的多，我先來說說絕對的旋轉與縮放是怎麼計算的。

#### 絕對旋轉的計算

這個Framework儲存旋轉值都是以角度來儲存的（通常好像是用徑度，我就沿用了，反正兩種都行）。**絕對旋轉的算法是 GameObject的相對旋轉 + 其Parent的絕對旋轉**。就這麼簡單，由於Parent的絕對旋轉也會需要計算其Parent的絕對旋轉，所以這個計算會遞迴一直算到最上層為止。**簡單的說，就是一路往最上層，把所有相對旋轉加起來**。

*absoluteRotation = relativeRotation + parent.absoluteRotation*

#### **絕對縮放的計算**

縮放跟旋轉幾乎一樣，就是 + 變成 *（乘） 而已。
**絕對旋轉的算法是 GameObject的相對縮放 * 其Parent的絕對縮放**。

*absoluteScale = relativeScale * parent.absoluteScale*

#### 絕對位置的計算

由於相對位置是基於其Parent的座標系統，所以如果Parent有旋轉，我的x軸是往Parent的x軸延伸的。例如：Parent的旋轉是45度，我的x是1m，那我的絕對位置的x會是在Parent絕對座標的45度角方向增加1m的位置。所以這會運用到三角函數的計算。

**先把Parent的絕對旋轉以徑度 rad 表示**（由於我們是以角度儲存，這邊要做 乘 180 除 PI 的計算）。

**Parent的絕對縮放以 scale 表示** （自身的縮放不會影響自身的位置）。

*absoluteX = ( relativeX * **cos**(rad)  - relativeY * **sin**(rad) ) * scale + parent.x*

*absoluteY = ( relativeX * **sin**(rad) + relativeY * **cos**(rad)) * scale + parent.y*

### Camera

Camera的概念是，把所有GameObject的位置，透過其相對一個 Camera（攝影機）的位置，來計算它應該出現在螢幕上的哪個位置。如果沒有這個概念，我們就只能畫出 x位置在 0 ~ 螢幕寬度 和 y 位置在 0 ～ 螢幕高度 的東西了哦～

GameObject在螢幕位置的計算是這樣的：

*x = gameObject.absoluteX - camera.x + Screen.width / 2*
*y = gameObject.absoluteY - camera.y + Screen.height / 2*

其實就是GameObject跟camera的相對位置而已。 由於我們在繪圖時會希望以Camera為中心繪圖。所以假設GameObject跟camera在同一點上，它其實應該在螢幕的中間而非 x: 0, y: 0的位子上，所以我們在 x 跟 y 上分別加上 螢幕寬度 / 2 與 螢幕高度 / 2 。

### Canvas

到這裡才講到真正在 HTML5 Canvas上繪圖的部分。但整個繪圖的系統都是基於上面的幾個概念建構出來的。

Canvas的繪圖，首先要知道的是：

-   原點 (x: 0, y: 0) **在左上角**
-   **x是往右**延伸，**y是往下**延伸

這跟我們在數學上慣用的座標系統是有差別的，所以要注意不要搞錯了。

由於我們在做的是 2D Game，這邊假設我們要繪圖的東西全部都是圖片（Image），我下面就用 canvas 的 drawImage來解說。

*context.drawImage(image, x, y)*

*context.drawImage(image, x, y, width, height)*

這個函數是在 canvas 上，從左上角往右數 x 像素，往下數 y 像素的位置畫出 image 圖片。如果你沒有指定 width 跟 height，系統會幫你預設畫出該圖片的原始大小。若指定的話，則會幫你把整張圖縮放到指定的大小繪圖。這個就是在 HTML5 Game裡面最常用到的繪圖函數了！

我們現在只要把 image 設為GameObject指定的圖片，x, y 設為GameObject經過 Camera 轉置成的螢幕座標，並把 width 與 height 設成原始大小乘以 scale 值。 就可以在 Canvas 上正確的位置畫出遊戲中的物件了！

那旋轉怎麼辦呢？Canvas 有個旋轉函數可以使用：

*context.rotate(angle)*

在繪圖之前執行這項指令設定好angle後（注意 angle 是徑度，所以要先轉換哦），接下來畫的東西都會旋轉這個角度囉！

這下就可以在 Canvas 上畫出場景上所有的物件了，而且包括位移、旋轉跟縮放。

### 進階

其實 Canvas 還提供了各種轉置函數，例如： *context.translate(x, y), context.scale(x, y) *，translate是位移，scale則是縮放。由於這又牽扯到了 stack 的概念，這些我就留在 **Part II **來講吧。

Canvas 可以用的功能其實非常多，只要觀念正確，你也可以在網頁上做出很多華麗特效的遊戲。想要深入了解的話，可以先參考 MDN 的 [Canvas Reference](https://developer.mozilla.org/en-US/docs/Web/API/CanvasRenderingContext2D)。這篇就先講到這邊吧！