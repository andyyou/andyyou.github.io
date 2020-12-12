---
title: gvm on macOS installation cheatsheet
date: 2019-04-14 10:09:30
tags:
  - go
categories: Program
---


# OSX $PATH 設定路徑

```bash
# ~/.profile
# ~/.bash_profile
# ~/.bashrc
# ~/.zshrc

# /etc/profile
# /etc/paths
# /etc/paths.d

$ /usr/libexec/path_helper -s
```
<!-- more -->


# 移除 OSX 官方 go 安裝檔

```bash
# https://golang.org/doc/install#uninstall
$ rm /usr/local/go
$ rm /etc/paths.d/go

# 使用 gvm https://github.com/moovweb/gvm
```



# 查詢環境

```bash
$ go env
```



# 安裝

```bash
# gvm
$ bash < <(curl -s -S -L https://raw.githubusercontent.com/moovweb/gvm/master/binscripts/gvm-installer)
# OR
$ zsh < <(curl -s -S -L https://raw.githubusercontent.com/moovweb/gvm/master/binscripts/gvm-installer)
# 重啓終端機

# 列出所有版本
$ gvm listall

# 列出本地版本
$ gvm list

# OSX 環境有問題的話
$ xcode-select --install
$ brew update
$ brew install mercurial

# 重新編譯（Compile）特定版本，需要先安裝好 Go1.4
$ gvm install <version>

# 直接安裝 Binary，不需要編譯
$ gvm install <version> -B

# 刪除 gvm
$ gvm implode
Are you sure? [Y/n] y
GVM successfully removed
# 或者
$ rm -rf ~/.gvm

# 2018-12-06 - Fix Compile Error: failed MSpanList_Insert & Missing go1.4
$ gvm install go1.7.3 -B
$ gvm use go1.7.3
$ gvm install go1.11.2
$ gvm use go1.11.2 --default

# 編輯 zshrc
[[ -s "/Users/andyyou/.gvm/scripts/gvm" ]] && source "/Users/andyyou/.gvm/scripts/gvm"
# export GOPATH=~/Projects/go
# gvm 預設 GOPATH
# ~/.gvm/pkgsets/go<version>/global
export PATH=$PATH:$(go env GOPATH)/bin
```

