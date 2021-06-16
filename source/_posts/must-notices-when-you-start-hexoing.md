---
title: 開始玩Hexo時你必須注意的那些事
date: 2021-06-16 10:33:57
tags: hexo, problems
---

不確定有多少人輕輕鬆鬆的就把自己的Hexo部落格架好了；也不可否認步驟其實不多、不難。但至少就我的情況來說，我真是踩了不少陷阱、碰到不少預期外的錯誤，花了我整整**一天**才把部落格架起來。所以這篇文章，我想要分享我到底遇到了哪些問題，以及我怎麼解決的，希望可以稍微幫到以後也遇到這些問題的人 :)

## Topic 1: 幫自己選一個theme

開啟一個新的 hexo 專案之後要做的第一件事情，當然是先來挑一個 [theme](https://hexo.io/themes/) 啦。點選一個 theme 的標題會開啟那個 theme 的 Github 專案頁面；上面通常都已寫有安裝 theme 的方法。

![hexo-themes](https://i.imgur.com/jJhSI57.png)

大多數 theme 作者提供的安裝方法是 `git clone` 他們的專案到 `/theme/` 資料夾底下。不過這麼做等於是在一個 git 專案底下再放一個 git 專案 - 頂層的專案不會記錄該 theme 裡面的變更；也就是要分成兩個 repository 來記錄變更。所以筆者個人比較喜歡的做法是，先 clone 到另外一個地方，再把所有內容物複製起來；接著在 `/theme/` 底下開一個該 theme 的資料夾，並貼上剛剛複製的東西。這應該是一個滿快滿簡單的作法。

> 本來想找 git 有沒有只複製檔案下來的指令或選項，但是 git 目前似乎是沒有這樣的功能。唯有找到一個比較接近的是 `git arhchive --remote=<url>` ，但是它是把檔案打包成壓縮檔下載；而且筆者嘗試時也失敗了，比較不推。

## Topic 2: 在 Github Pages 部署網站

Hexo 的官網有提供在 Github 上部署網站的[方法](https://hexo.io/docs/github-pages)。其中必須注意的是 hexo 的運作模式:  hexo 需要兩條分支，一條用來平常撰寫文章，另一條用來存放 build 完的檔案 (不爭氣如我在了解這個模式時不小心就被嚇到了QQ)。官網給的範例是，在 `source` 分支撰寫、在 `master` 分支部署。

![hexo-docs-for-deploy](https://i.imgur.com/HPjp2kf.png)

```
on:
  push:
    branches:
      - source  # 平常撰寫文章的分支

jobs:
  pages:
    ... (省略)
      - name: Deploy
        uses: peaceiris/actions-gh-pages@v3
        with:
          github_token: ${{ secrets.GITHUB_TOKEN }}
          publish_dir: ./public
          publish_branch: master  # 部署時 build 出來的分支
```

這當然是可以改的。不過改完這裡之後，不要忘了還有另外一個地方也要設定。在 setting 頁面中的 page 欄裡，我們可以選擇藥用哪個分支來 host 網站。如果是照官網操作，選 `master` 即可；反之則是設定為自己先前所指定的部署分支。

![deploy-branch-setting](https://i.imgur.com/dCSPwHI.png)

## Topic 3: Github強迫我挑一個他們的theme?!

將寫好的 yml 檔推上去、選好 deploy branch，Github 就會開始幫我們 build 網站了。但是...事情還沒結束! 你大概會看到這個錯誤:

> You are attempting to use a Jekyll theme, "cactus", which is not supported by GitHub Pages.
Please visit https://pages.github.com/themes/ for a list of supported themes. 
If you are using the "theme" configuration variable for something other than a Jekyll theme, 
we recommend you rename this variable throughout your site. 


如果在 Github 上沒看到這麼一段警告訊息，也可以翻翻 email 信箱。筆者個人的狀況印象中是... Github 顯示成功 deploy，信箱卻收到好幾封這樣的郵件；網站則是顯示 `404 page not found`。這是因為 Github Pages 的 theme 需要是一個 jekyll theme；使用非 jekyll 的 theme 似乎就會出現這種警告，網站也跑不出來。

![jekyll-theme-pic](https://i.imgur.com/xL4D6nW.png)

筆者嘗試了一種 workaround，跟大家分享: 

1. 在專案裡面，先將 theme 的名字改成一個 jekyll theme 的名字。

![theme-changed-to-jekyll-name](https://i.imgur.com/jV9OBXc.png)

2. `_config.yml` 檔裡面的 theme 名字也要改掉。改完就可以推上去了。

```
# Extensions
## Plugins: https://hexo.io/plugins/
## Themes: https://hexo.io/themes/
theme: jekyll-theme-cayman     # change to this
```

3. 去 setting 頁面將 theme 改成一樣的。

![theme-changed-in-setting](https://i.imgur.com/OZh27Gu.png)

理論上...這樣應該就繞過 Github 檢查了。但是很不幸的，筆者的嘗試是失敗的。Github 送給我大大的錯誤:

> The value '{}' was passed to a date-related filter that expects valid dates in `/_layouts/default.html` or one of its layouts. 

如果翻閱 Github Docs，它會表示這是因為有無效時間存在某些檔案中 (可能格式錯誤或時間值怪怪的吧)。但是筆者翻過了自己的專案，並沒有找到疑似時間值有問題的檔案，它提示的 `_layout/default.html` 也不見蹤影 (筆者為此納悶了很久)。

雖然不知道問題出在哪裡，但是筆者有找到一個很厲害的解決方法，可以輕鬆解決這兩個問題: 直接在專案裡面新增一個 `.nojekyll` 檔案就可以了! 裡面什麼都不用打，空的就可以了。

![nojekyll](https://i.imgur.com/bUvPUCQ.png)

這麼一來，就不用特地將 theme 改成其中一個 jekyll theme 的名字，時間的問題也不再發生了。

## Conclusion & 後記

以上就是我這次架 hexo 部落格遇到的問題以及解法整理，希望對各位有幫助!

這是我第一次寫 blog，請各位大大鞭子鞭小力一點 ><。初次寫 blog ，我覺得寫 blog 最棒的地方是，為了追求文章資訊的正確性，我得查很多資料、做一些測試，因而導正了一些原先錯誤的認知，自己覺得收穫不少~ 不過如果文章內容還是有誤，請各位不要吝嗇提醒我! 用 email 好了 -> leafoliages@gmail.com

阿里嘎多 :)