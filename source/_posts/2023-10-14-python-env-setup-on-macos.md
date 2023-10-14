---
title: Python 在 macOS 的環境設定及套件管理指南
date: 2023-10-14 10:53:58
tags:
  - python
categories: Program
---

### macOS 環境安裝

使用 `pyenv` 目的是讓本地環境支援多種 Python 版本。大多的程式語言都有對應的工具如 Nodejs 的 nvm，Ruby 的 rvm，rbenv 等。

<!-- more --

```sh 
# ⭐️ 1. 安裝 Homebrew 
$ /bin/bash -c "$(curl -fsSL https://raw.githubusercontent.com/Homebrew/install/HEAD/install.sh)"

# 因版本會持續更新建議直接上官網查閱教學 https://brew.sh/

# ⭐️ 2. 安裝 pyenv
$ brew install pyenv

# ⭐️ 3. 配置 shell，
# 在你的 ~/.zshrc 或 ~/.bash_profile 中加入以下內容
# export PATH="$HOME/.pyenv/bin:$PATH"
# eval "$(pyenv init --path)"
# 具體可以參考 brew 安裝過程中的提示訊息
$ echo 'eval "$(pyenv init -)"' >> ~/.zshrc
$ source ~/.zshrc

# ⭐️ 4. 安裝 Python
$ pyenv install 3.11.4

# ⭐️ 5. pyenv 基礎
# 查詢可安裝版本
$ pyenv install -l

# 查看已安裝版本
$ pyenv versions

# 設定版本
$ pyenv global 3.11.4 # 設定全域環境版本
$ pyenv local <version>  # 在當前目錄創建一個.python-version，以後進入這個目錄自動切換爲該版本
$ pyenv rehash # 當你認為 shims 可能已經過時或不再反映當前的 Python 環境時，就應該運行

# ⭐️ 6. Python 基礎指令
# 在指令介面執行
$ python -c "import sys; print(sys.argv);"
# 使用模組；例如 啟動簡易伺服器
$ python -m http.server
```

### Python 虛擬環境

開發 Python 應用程式的時候經常會面臨：

- 相依套件衝突：兩個專案對於同一個套件需要不同版本
- 系統問題：直接在安裝在系統環境導致影響系統工具

為了解決這個問題 Python 引入了虛擬環境作為解決方案。其實就是一個隔離環境有自己獨立的套件。

#### venv

Python 3.3 之後內建了 `venv` 模組。

```sh
# 建立虛擬環境
$ python -m venv myenv
# 啟動
$ source myenv/bin/activate
# 等於
$ . myenv/bin/activate
```

#### .venv

`.venv` 是虛擬環境的命名慣例，通常會在專案目錄下建立。當你在一個專案目錄中看到 `.venv`，你可以預期這是該專案使用的虛擬環境。通常用來存放虛擬環境的所有相關文件。

也就是常見的您到專案下會先啟用該環境

```sh
$ cd your_project
$ . .venv/bin/activate
```



### Python 專案的套件管理

如果您是從其他程式語言環境進入 Python 的開發者您可以能會很困惑，這個段落希望可以釐清這些工具的關係。

從 Python `2.7.9`，`3.4` 開始，預設 Python 提供了 `pip` 套件管理工具。

Python 3.3 開始，標準庫內建了 `venv` 模組通過結合 `pip` 和 `venv`，Python 開發者可以靈活地控制和管理套件依賴和環境。

#### pip

`pip` 是 Python 的套件管理器，用於安裝和管理Python套件，它從 PyPI (Python Package Index) 下載套件。

```sh
# 安裝套件
$ pip install package-name
# 移除
$ pip uninstall package-name
# 列出
$ pip list
# 為了重現專案的環境，你可以使用 pip freeze 來紀錄目前所有的套件和它們的版本
$ pip freeze > requirements.txt
# 使用 requirements.txt 安裝套件:
$ pip install -r requirements.txt
```

但在實務上，`pip freeze` 有一些缺點，如它會列出所有套件，包括不直接依賴的套件，這可能會造成版本衝突或其他問題。
還有開發者安裝後卻忘記紀錄都是問題。

因此，人們開始選擇使用如 `pipenv` 或 `poetry` 這些工具。時至 2023/10 目前社群越來越多人採用 `poetry`

#### [poetry](https://python-poetry.org/)

```sh
# 安裝
# https://python-poetry.org/docs/
$ curl -sSL https://install.python-poetry.org | python -

# 建立新專案
$ poetry new demo

# poetry 使用 pyproject.toml 和 poetry.lock 文件來管理套件依賴
poetry-demo
├── pyproject.toml 全部相關設定，相依紀錄在這
├── README.md
├── poetry_demo
│   └── __init__.py
└── tests
    └── __init__.py
    
# 已建立專案使用
$ cd pre-existing-project
$ poetry init

# 安裝套件
$ poetry add package-name

# 移除
$ poetry remove package-name

```

> poetry 使用 pyproject.toml 和 poetry.lock 文件來管理套件依賴

- `pip` 是基礎的套件管理工具，但在大型專案中可能會遇到一些依賴問題。
- `pipenv` 和 `poetry` 提供了更進階的依賴和專案管理，解決了許多 `pip` 的問題並提供了更多功能。