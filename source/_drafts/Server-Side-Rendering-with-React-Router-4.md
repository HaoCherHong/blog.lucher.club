---
title: Server-Side Rendering with React Router 4
date: 2018-02-05 17:11:17
tags: [dev, react, pupy]
---

最近在重寫 [PUPY](/tags/pupy) 的前端跟 Web Server。因為很久以前用第一代 Angular，體會到了沒有 SSR (Server-Side Rendering) 的痛苦，所以這次在前期就特別先開始處理 SSR 的東西了**（然後就展開了一場為期12小時的痛苦旅程⋯）**。

然後，我真的很慶幸我不是從今年開始學網頁的。

# H1

會特別寫這篇是因為網路上針對 React Router 4 SSR 的資源比較少，而我處理後又遇到了非常多棘手的問題，所以特別記錄下來。

這篇會涵蓋到：
- Server-Side Rendering
  - 使用 `redial` 預先擷取資料
  - 用一樣的 Code 分開處理 Server-Side 與 Client-Side 的 Render
  - 處理 `style-loader` 不支援 SSR 的問題
- Hydrate React - 讓瀏覽器的 React 接上 Render 的結果
- Hydrate Store - 讓瀏覽器的續用 SSR 的 Redux Store

# Server-Side Rendering