---
title: 
  【APGS2021】Using third-party tools to lower indie development costs, a case of
  M.A.S.S. Builder
author: Karlun Lai
tags:
  - APGS2021
  - APGS亞太國際高峰會
  - Procedural Generation
categories:
  - APGS2021_亞太國際高峰會
date: 2021-01-28 19:25:00
---
講者：Nontawat Thanakiatkrai
<!--more-->
[線上直播連結](https://youtu.be/kLXB57-rHTg?t=2664)

Procedural Generation（循序生成技術）
---
- 透過程式產生隨機或特定規則的遊戲資源與物件的過程。
- 提供無限的資源，從岩石、樹木與建築物等小型物件，到地形、山脈，甚至整個關卡都適用。
- 網路上有許多用此技術開發的工具（e.g. Houdini、World Creator），而時機成熟後也可以自己開發。

Procedural Generation Example
---
- 低面數小型物件
以電纜線為例，透過隨機化，可以設定包括管內纜線數量、彎曲程度等，任何相似的東西都可以自行調整。
![](https://i.imgur.com/lhU1boG.jpg)

- 高面數岩石資源
可以設定密度、大小和細緻程度，讓這些岩石在每個seed中可能都不一樣。
而稱作高面數的原因，因為這些岩石可以用來裝飾關卡，提供關卡深度和細節。
![](https://i.imgur.com/NaGVjbz.jpg)

- 樹葉
如不用此技術，這些樹葉可能將耗費團隊大量時間為每一種類型的樹木建立library；如果使用此技術，我們就可以針對每一種關卡建立不同的樹葉類型，e.g. 雪山、森林or平原。
![](https://i.imgur.com/G8moeCB.jpg)