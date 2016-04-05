---
title: 一个将Chrome中打开的标签页都在Safari中打开的AppleScript脚本
date: 2014-05-07 00:00:00
tags: Mac
---
因为我更喜欢使用Chrome作为日常浏览网页的浏览器,但是经常点着点着就打开了一大票标签页,然后一时间有没时间看完,这些东西有些也许就是一次性看完就扔的,保存到pocket感觉太大材小用而且容易让pocket中充满各种"垃圾",如果关掉的话将来再找也麻烦.

所以我一般喜欢把这些标签页都拷贝到不是最常用的Safari浏览器中,方便下次阅读.

前两天我在QQ群里灌水时说到这个问题,然后听@Dawn提到AppleScript好像可以解决这个问题.以前对AppleScript只听过没见过,今天晚上搜了下,果然神器,于是Google了一下找了几段demo代码对照着写了个,感觉还可以,对于一些小工具性质的软件来说,applescript实在是太方便了.

这个AppleScript脚本虽然只有10几行,但是已经可以完成将当期chrome窗口中打开的标签页都在Safari中打开了,达到了我的目标,很好.

直接上代码:

[https://gist.github.com/zoowii/b1818fae22792e4e9bab](https://gist.github.com/zoowii/b1818fae22792e4e9bab)


    tell application "Google Chrome"
     set cur_win to first window
     tell cur_win
      set urls to URL of tabs of cur_win
      set numOfUrls to (count urls)
      -- repeat with i from 1 to (numOfUrls)
      repeat with itemUrl in (urls)
       -- set itemUrl to (item i of urls)
       tell application "Safari" to tell first window to open location itemUrl
      end repeat
      tell application "Safari" to activate
      tell application "Safari" to display dialog "Move tabs from Chrome to Safari successfully"
     end tell
    end tell


下载地址: [http://pan.baidu.com/s/1sjHHmXJ](http://pan.baidu.com/s/1sjHHmXJ)
