---
title: 部署 Rails 到 Elastic beanstalk 流程筆記
date: 2016-10-13 16:51:15
categories: Cloud
tags: [aws, RoR]
---

* 安裝 Elastic Beanstalk 指令介面（EB CLI）
* 建立 Instance Profile （IAM）
  + 修改 IAM 可能不會馬上完整生效
* 建置 Elastic Beanstalk
* EB init & EB setup -ssh
* 設定 database.yml （使用 RDS 服務）
* 設定必要的環境變數
* 設定 pm2 + Nginx
* 使用 ebextensions 目錄建立 Elastic Cache Instance
* 部署

# 安裝 Elastic Beanstalk 指令介面（EB CLI）

Elastic Beanstalk 指令介面是一系列客戶端的指令讓我們可以用來建立、設定、管理 Elastic Beanstalk 環境。
EB CLI 是使用 Python 開發的所以需要 Python 2.7 或 3.4 版。

> 注意 Amazon Linux 從 2015.03 版本開始內建 Python 2.7 和 pip

> 注意 EB CLI 並不相容於 Python 3.5

在 Linux, Windows, OSX 上面安裝 EB CLI 的方式是透過 `pip`，pip 是 Python 的套件管理工具。協助我們輕易的安裝、升級、移除 Python 的套件。
針對 OSX 您也可以透過 Homebrew 安裝 EB CLI。

如果您已經安裝了 pip 和支援的 Python 版本，您可以直接使用下面的指令安裝：

```bash
$ pip install --upgrade --user awsebcli
```

關於 `--upgrade` 參數用於告知 pip 升級任何已安裝且 EB CLI 需要的項目。`--user` 則是安裝程式到您使用者目錄下的一個子目錄以避免修改系統上其他函式庫。

> 注意 如果您在透過 pip 安裝 EB CLI 過程中遇到問題，您可以安裝[EB CLI a virtual environment](http://docs.aws.amazon.com/elasticbeanstalk/latest/dg/eb-cli3-install-virtualenv.html)來分離工具與相依函式庫，或者使用不同版本的 Python。

當 EB CLI 安裝完畢之後，記得加上執行檔路徑到 `PATH` 環境變數：

* Linux - `~/.local/bin`
* OSX - `~/Library/Python/2.7/bin`
* Windows - `%USERPROFILE%\AppData\Roaming\Python\Scripts`

### 修改 PATH 環境變數

#### Linux, OSX, Unix

  1. 找到您 shell 的 profile。如果您不確定您的 shell 是什麼，執行 `echo $SHELL`

  ```bash
  $ ls -a ~
  ```

  * Bash - `.bash_profile`、`.profile`、`.bash_login`
  * Zsh - `.zshrc`
  * Tcsh - `.tcshrc`、`.cshrc`、`.login`

  2. 加入 export 的指令到 profile script

  ```rc
  export PATH=~/.local/bin:$PATH
  ```

  3. 載入 profile 到當前的工作階段（seesion）。

#### Windows

  1. 點擊 Windows 鍵並輸入 `environment variables`。
  2. 選擇 `Edit environment variables for your account`
  3. 選擇 `PATH` 然後 `Edit`
  4. 加入路徑到 `Variable value` 欄位，並用 `;` 分號隔開，例如：`C:\esisting\path;C:\new\path`
  5. 點擊 `OK` 完成設定
  6. 關閉目前的指令介面，重新開啟。

接著驗證 EB CLI 是否正確安裝完成

```bash
$ eb --version
EB CLI 3.7.8 (Python 2.7.1)
```

為了支援 Elastic Beanstalk 最新的功能，EB CLI 會經常性的更新。要更新最新版的 EB CLI 只需執行下面指令

```bash
$ pip install --upgrade --user awsebcli
```

如果您需要移除 EB CLI 則使用 `pip uninstall`

```bash
$ pip uninstall awsebcli
```

如果您還沒有 Python 和 pip 請參考下面的文章

* [Install Python, pip, and the EB CLI on Linux](http://docs.aws.amazon.com/elasticbeanstalk/latest/dg/eb-cli3-install-linux.html)
* [Install Python, pip, and the EB CLI on Windows](http://docs.aws.amazon.com/elasticbeanstalk/latest/dg/eb-cli3-install-windows.html)
* [Install the EB CLI on OS X](http://docs.aws.amazon.com/elasticbeanstalk/latest/dg/eb-cli3-install-osx.html)
* [Install the EB CLI in a Virtual Environment](http://docs.aws.amazon.com/elasticbeanstalk/latest/dg/eb-cli3-install-virtualenv.html)

# 建立 Instance Profile （IAM Role）

所謂的 Instance Profile 就是套用到 Elastic Beanstalk 環境的 IAM 角色。

### IAM 角色（IAM Role）

應用程式必須要替 API 請求簽署憑證，因此假如您是一個開發者，您會需要替在 EC2 上執行的應用程式管理憑證。例如您可以安全地將憑證發佈到 EC2 執行個體上，提供在那些執行個體上的程式使用您的憑證去簽署請求以避免其他使用者盜用您的資源。 - AWS 上的服務要使用資源時，任何請求都需要依靠憑證來判斷是否具有存取權限。然而要安全的發派憑證到每一個執行個體是個挑戰，特別是那些 AWS 替您創建的部分，像是競價型執行個體或是 Auto Scaling 群組中的執行個體。另外當您更換 AWS 憑證實還必須要更新每一個執行個體上的憑證。

我們設計了 IAM 角色以便於在執行個體中的程式可以安全地送出 API request （執行個體為 request 簽署憑證），而不需要您去管理那些程式會用到的安全憑證。比起自己建立並分配 AWS 憑證，您可以透過 IAM 角色替 API request 設定權限。

1. 建立 IAM 角色
2. 定義哪些帳號或 AWS 服務可以具備該角色權限
3. 定義哪些 API 和資源在賦予角色權限後是應用程式可以使用
4. 啟動的執行個體時指派角色權限
5. 讓應用程式能夠檢索和使用一組暫時的憑證

舉例來說，您可以使用 IAM 角色賦予執行個體中的程式權限，使其能夠存取 Amazon s3 的某組 bucket。

注意 Amazon EC2 使用 Instance Profile 執行個體配置文件作為 IAM 角色的容器。當使用 console （控制台）建立一組 IAM 角色時，console （控制台）會自動建立 Instance Profile 並同時給予相同的名稱。當您使用 AWS CLI、API、或 AWS SDK 來建立角色時，您會需要分別建立角色和 Instance Profile，這時您可能會給他們不同的名稱。當要啟動執行個體並使用 IAM 角色時，您需要指定的是 Instance Profile 的名稱。

而當我們透過 Amazon EC2 Console （控制台）啟動時，我們可以選擇與執行個體關聯的角色; 但是注意上面顯示的是 Instance Profile 的名稱。

您可以透過 JSON 格式建立授權規則（Policy）替 IAM 角色指定權限。這跟 IAM 使用者（IAM User）建立 Policy。如果您修改了 Policy 那這些變更會被傳到所有的執行個體，簡化憑證的管理。

注意：您不能替一個已存在的執行個體設定角色，只能夠在創建時設定。

# 新增 Instance Profile

創建之前需要安裝 `awsebcli` 上面步驟我們已經安裝完成了，接著需要一組 AWS 帳號登入控制台 Console。

{% asset_img 1.png %}

在建立 Beanstalk 之前，上面已經說過需要定義一個 IAM Role 配置該角色能夠使用的資源與權限（S3, SQS, RDS, CloudWatch）

{% asset_img 2.png %}

進到控制台後找到 `Security & Identity` -> `IAM` 照著步驟設定

{% asset_img 3.png %}
{% asset_img 4.png %}
{% asset_img 5.png %}
{% asset_img 6.png %}

> 注意：IAM Role 之後無法更名。

# 建立 Beanstalk App

在控制台頁面找到 Elastic Beanstalk 並選擇建立新的應用程式 `Create New Application`
