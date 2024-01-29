---
title: Golang构建桌面程序之webview
comments: true
banner_img_height: 30
date: 2022-11-03 12:19:36
categories:
- Golang
tags:
- Golang
- webview
- embed
- vue
- gin
- GUI
---

## 序
最近接到的任务是开发一个可以在mac和windows上运行的程序，作为一个golang程序员首先想到的自然是golang。跨平台编译运行也让他可以做这件事。结合前面我了解到的一些框架（[govcl](https://github.com/ying32/govcl),[fyne](https://github.com/fyne-io/fyne),[wails](https://github.com/wailsapp/wails),[walk](https://github.com/lxn/walk),[wxwidgets](https://www.wxwidgets.org/)），考虑学习的时间成本我本来是想用[fyne](https://github.com/fyne-io/fyne)来做。直到后来我找到了这个能更快完成需求的包[webview](https://github.com/webview/webview)。所以今天主要来介绍[webview](https://github.com/webview/webview)这个可以跨平台生成可执行文件的框架。

## Golang 的 webview package