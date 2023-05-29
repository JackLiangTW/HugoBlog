+++
showonlyimage = false
draft = false
date = "2016-11-05T18:25:22+05:30"
title = "Hugo markdown link轉換成<a>時候加上target=blank屬性"
weight = 1
showmoretile = false
tags = "Hugo"
+++

[參考來源](https://stackoverflow.com/questions/72046530/can-i-create-links-with-target-blank-in-hugo-posts-content-using-markdown)

如果希望Hugo當中的.md檔案中的轉成html時候能夠加上target='blank'屬性
```html
[链接文字](链接地址)
<a href=链接地址>链接文字</a>
```

成為
```html
[链接文字](链接地址)
<a target='blank' href=链接地址>链接文字</a>
```


可以屬用以下步驟:
1. 創建該檔案 themes\自己所屬用主題名\layouts\_default\_markup\render-link.html
2. 在該檔案當中使用以下html
```html
<a href="{{ .Destination | safeURL }}"{{ with .Title}} title="{{ . }}"{{ end }}{{ if strings.HasPrefix .Destination  "http" }} target="_blank" rel="noopener"{{ end }}>{{ .Text | safeHTML }}</a>
```

![結果示意][1]


[1]: /img/blog/hugoLinkChange.JPG