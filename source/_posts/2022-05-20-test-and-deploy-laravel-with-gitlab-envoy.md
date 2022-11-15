---
title: 使用 Gitlab CI/CD, Envoy 測試與佈署 Laravel 應用程式 (Ubuntu 22.04)
date: 2022-05-20 14:16:12
tags:
  - gitlab
  - envoy
  - laravel
categories: System
---

## 介紹

Gitlab 支援持續整合的功能，讓我們可以簡單的佈署變更的程式碼到伺服器。

<!-- more -->

本文我們將學習如何為 Laravel 專案設定 [Envoy](https://laravel.com/docs/master/envoy) 任務, 然後到通過 [持續交付](https://about.gitlab.com/blog/2016/08/05/continuous-integration-delivery-and-deployment-with-gitlab/) 使用 [Gitlab CI/CD]() 測試和佈署。

這裡假設您對於 Laravel, Linux 伺服器和 Gitlab 有基本的的經驗。

Laravel 是使用 PHP 撰寫的高品質網頁開發框架。有不錯的社群，優秀的文件。除了一般路由，Controllers，請求，回應，視圖，樣板這些內建功能，Laravel 還包含其他豐富的功能例如 Cache，事件，語系，身份驗證等等。

我們將使用一個 PHP 的 SSH 任務執行器 - **Envoy**。其採用簡單，輕量化的 [Blade 語法](https://laravel.com/docs/master/blade) 設定任務，然後我們可以在遠端伺服器上執行指令，例如從 Git 檔案庫複製專案，安裝 Composer 相依套件，執行 Artisan 指令。

## 初始化 Laravel 專案

我們假設您已經安裝好了一個新的 Laravel 專案, 因此讓我們從單元測試和初始化 Git 開始。

### 單元測試

每一個新建的 Laravel 會有包含兩種類型的測試: 功能 `Feature` 和單元 `Unit`, 它們在 `tests` 目錄下。這裡是 `tests/Unit/ExampleTest.php`

```php
<?php

namespace Tests\Unit;

...

class ExampleTest extends TestCase
{
    public function testBasicTest()
    {
        $this->assertTrue(true);
    }
}
```

這個測試只是單純判斷給定的值是否為 `true`。

Laravel 預設使用 `PHPUnit` 測試。如果我們執行 `vendor/bin/phpunit`

```sh
$ vendor/bin/phpunit
# OK (1 test, 1 assertions)
```

這個測試將會在後續 Gitlab CI/CD 持續交付時用來檢查我們的應用程式。

### push 至 Gitlab

因為我們已經可以在本地端執行應用程式了，是時候將程式碼推送到 git 檔案庫。我們在 Gitlab 上建立一個新專案例如名稱 `example` 然後可以遵循下面指令

```sh
$ cd example
$ git init
$ git remote add origin git@gitlab.example.com:<USERNAME>/example.git
$ git add .
$ git commit -m 'Initial commit'
$ git push origin main
```

## 設定正式伺服器

在開始設定 Envoy 和 Gitlab CI/CD 之前，讓我們快速確認伺服器環境已經準備好可以佈署。我們需要在 Ubuntu 環境安裝 LEMP

- [How To Install Linux, Nginx, MySQL, PHP (LEMP stack) on Ubuntu 22.04](https://www.digitalocean.com/community/tutorials/how-to-install-linux-nginx-mysql-php-lemp-stack-on-ubuntu-22-04)
- [How To Install and Configure Laravel with Nginx on Ubuntu 22.04 (LEMP)](https://www.digitalocean.com/community/tutorials/how-to-install-and-configure-laravel-with-nginx-on-ubuntu-22-04)
- [How To Install Node.js on Ubuntu 22.04](https://www.digitalocean.com/community/tutorials/how-to-install-node-js-on-ubuntu-22-04)
- [How To Install and Use PostgreSQL on Ubuntu 22.04](https://www.digitalocean.com/community/tutorials/how-to-install-and-use-postgresql-on-ubuntu-22-04)

```sh
### Nginx
$ sudo apt update
$ sudo apt install nginx
# 檢查設定
$ sudo nginx -t
# 重載設定
$ sudo systemctl reload nginx

### PostgreSQL
$ sudo apt install postgresql postgresql-contrib

### PHP
$ sudo apt install php8.1-fpm php-pgsql
# For Larvel
$ sudo apt install php-mbstring php-xml php-bcmath php-curl

### Composer
# https://www.itzgeek.com/how-tos/linux/ubuntu-how-tos/install-laravel-on-ubuntu-22-04.html
$ sudo apt install -y curl
$ curl -sS https://getcomposer.org/installer | sudo php -- --install-dir=/usr/bin --filename=composer

### Nodejs through nvm
$ curl -o- https://raw.githubusercontent.com/nvm-sh/nvm/v0.39.1/install.sh | bash
$ nvm install 18

# Laravel 權限設定
$ sudo chown -R www-data.www-data /var/www/<YOUR_PROJECT>/storage
$ sudo chown -R www-data.www-data /var/www/<YOUR_PROJECT>/bootstrap/cache

```

### 新增使用者

接著讓我們建立新的使用者用來佈署我們的應用程式, 且我們使用 [Linux ACL](https://serversforhackers.com/c/linux-acls) 賦予這個使用者必須的權限

```sh
$ sudo adduser deployer
# 賦予權限
$ sudo setfacl -R -m u:deployer:rwx /var/www
```

如果您的主機未安裝 ACL 可以使用下面指令

```sh
# 安裝 ACL
$ sudo apt install acl
```

### 新增 SSH Key

假設我們想要從我們私人的 Gitlab 檔案庫佈署應用程式到伺服器。首先我們需要為 `deployer` [產生一組無密碼的的 SSH Key Pair](https://docs.gitlab.com/ee/user/ssh.html)。

```sh
# 擇一
$ ssh-keygen -t rsa -b 2048 -C "<comment>"
# 或
$ ssh-keygen -t ed25519 -C "<comment>"
```

> - [如何將具有 SSH 存取權的使用者帳戶新增至 Amazon EC2 Linux 執行個體？](https://aws.amazon.com/tw/premiumsupport/knowledge-center/new-user-accounts-linux-instance/)

接著我們需要複製私鑰，用於 SSH 連線到伺服器以完成自動佈署的功能。

```sh
# 複製公鑰到 authorized_keys 如此 SSH 才能登入
$ cat ~/.ssh/id_rsa.pub >> ~/.ssh/authorized_keys

# 複製私鑰
$ cat ~/.ssh/id_rsa
```

將私鑰加入到 Gitlab 專案的 [CI/CD 變數](https://docs.gitlab.com/ee/ci/variables/index.html)。專案的 CI/CD 變數是使用者定義的變數，放在 `.gitlab-ci.yml` 之外是為了安全性的目的。在 Gitlab 專案下 **Settings > CI/CD** 可以找到。

我們欄位的 **KEY** 為 `SSH_PRIVATE_KEY` ， **VALUE** 則是剛剛複製的私鑰。後續我們會在 `.gitlab-ci.yml` 使用這個變數，讓 `deployer` 帳號不用密碼連線到遠端伺服器。

同時我們也需要將公鑰加入到專案 **Project > Settings > Repository** 的 **Deploy Keys** 如此外部才能夠利用 SSH 存取檔案庫(我們的伺服器主機下載 git repo)。

**Title** 可以隨意命名，然後將公鑰貼到 **Key** 欄位。

接著可以 `git clone` 我們的檔案庫到伺服器上，注意要使用 `deployer` 帳戶，因為它才有相關檔案權限。

```sh
$ git clone <YOUR_GIT_REPO_SSH_URL>
```

### 設定 Nginx

```sh
# (可選)手動新增專案目錄 - 若非直接使用 deployer 拉取檔案庫
$ sudo mkdir /var/www/app
$ sudo chown -R $USER:$USER /var/www/app

# 新增設定檔
$ sudo vi /etc/nginx/sites-available/<YOUR_DOMAIN>
# 編輯設定
$ sudo ln -s /etc/nginx/sites-available/<YOUR_DOMAIN> /etc/nginx/sites-enabled/
```

Nginx `<YOUR_DOMAIN>` 設定:

```nginx
server {
    listen 80;
    server_name server_domain_or_IP;
    root /var/www/app/current/public;

    add_header X-Frame-Options "SAMEORIGIN";
    add_header X-XSS-Protection "1; mode=block";
    add_header X-Content-Type-Options "nosniff";

    index index.html index.htm index.php;

    charset utf-8;

    location / {
        try_files $uri $uri/ /index.php?$query_string;
    }

    location = /favicon.ico { access_log off; log_not_found off; }
    location = /robots.txt  { access_log off; log_not_found off; }

    error_page 404 /index.php;

    location ~ \.php$ {
        fastcgi_pass unix:/var/run/php/php8.1-fpm.sock;
        fastcgi_index index.php;
        fastcgi_param SCRIPT_FILENAME $realpath_root$fastcgi_script_name;
        include fastcgi_params;
    }

    location ~ /\.(?!well-known).* {
        deny all;
    }
}
```

確認 `root`指向正確的目錄。

- [補充: 設定 SSL 搭配 Let's Encrypt](https://certbot.eff.org/instructions?ws=nginx&os=ubuntufocal)
- [補充: 解析 Certbot（Let's encrypt） 使用方式](https://andyyou.github.io/2019/04/13/how-to-use-certbot/)

## 設定 Envoy

到此我們已經準備好 Laravel 應用程式的正式環境了。接著就是使用 Envoy 執行佈署。要使用 Envoy 我們需要先在本機安裝 - 更多資訊可以參考[Laravel 教學](https://laravel.com/docs/master/envoy/#introduction)

```sh
$ composer require laravel/envoy --dev
```

### Envoy 運作機制

Envoy 雖然使用 Blade 語法來定義任務，但其實不需要 Blade 引擎。我們在專案根目錄建立一個 `Envoy.blade.php`

```php
@servers(['web' => 'remote_username@remote_host'])

@task('list', ['on' => 'web'])
    ls -l
@endtask
```

檔案頂部的 `@servers` 有個陣列參數，包含一個鍵 `web` 對應的值是伺服器的位置，例如 `deployer@192.168.1.1` 。然後我們使用了 `@task` 來定義 bash 指令，這些指令會在調用任務時在伺服器端執行。在本機我們可以執行

```sh
$ php vendor/bin/envoy run list
```

上面指令會執行我們定義的 `list` 任務，它會連線到伺服器然後列出目錄下的內容。

Envoy 並不相依於 Laravel 因此任何 PHP 應用程式都可以使用。

### 零停機時間佈署

每次當我們要佈署專案到正式伺服器時，Envoy 會從 Gitlab 下載最新版本。Envoy 不需要任何停機就可以完成這些任務，因此我們不需要擔心在佈署的時候有人造訪我們的網站。我們的佈署流程為從 Gitlab 複製最新版本，安裝 Composer 相依套件，最後切換為最新版本。

### `@setup`

佈署流程的第一步是在 `@setup` 設定一系列的變數。您可以變更 `app` 的名稱:

```php
@setup
    $repository = 'git@gitlab.com:<USERNAME>/example.git';
    $releases_dir = '/var/www/app/releases';
    $app_dir = '/var/www/app';
    $release= date('YmdHis');
    $new_release_dir = $releases_dir .'/'. $release;
@endsetup
```

- `$respository` 是 git 檔案庫的路徑
- `$releases_dir` 是應用程式佈署的目錄
- `$app_dir` 為整個應用程式的實際路徑
- `$release` 包含日期，因此每一次佈署新版本，我們會得到一個以日期作為名稱的新目錄
- `$new_release_dir` 新版本的完整路徑，目的只是讓任務撰寫可以更單純清楚

### `@story`

`@story` 定義了任務清單，整合成單一任務調用。這裡我們有三個任務分別是 `clone_repository`, `run_composer`, `update_symlinks`。

```php
@story('deploy')
    clone_repository
    run_composer
    update_symlinks
@endstory
```

接著我們來逐一建立這些任務。

#### 複製檔案庫

第一個任務是如果目錄不存在的話，建立 `releases` 目錄，然後複製檔案庫的 `main` 分支到新版本目錄。`releases` 目錄下會包含我們佈署的所有版本。

```php
@task('clone_repository')
    echo 'Cloning repository'
    [ -d {{ $releases_dir }} ] || mkdir {{ $releases_dir }}
    git clone --depth 1 {{ $repository }} {{ $new_release_dir }}
    cd {{ $new_release_dir }}
    git reset --hard {{ $commit }}
@endtask
```

隨著專案持續開發，git 歷史紀錄可能會非常長。由於每一個版本我們都會建立一個目錄，因此沒必要每次都包含全部紀錄。 `--depth 1` 參數就可以節省系統空間。

#### 安裝 Composer 相依套件

```php
@task('run_composer')
 echo "Starting deployment ({{ $release }})"
 cd {{ $new_release_dir }}
 composer install --prefer-dist --no-scripts -q -o
@endtask
```

- `--prefer-dist` 安裝套件使用編譯後版本 (`source` | `dist`)
- `--no-scripts` 關閉在根套件定義的 Scripts
- `-q` 不輸出執行相關資訊
- `-o` 轉換 PSR-0/4 自動載入為 classmap 以取得更快速的 autoloader。建議在正式環境使用，但執行會耗費多一點時間

#### 切換新版本

在準備好新版本之後，接著是移除 `storage` 目錄，建立兩個軟連結指向 `storage` 和 `.env` 。然後我們需要建立另一個軟連結用新版本取代目前 app 目錄下的 `current`。`current` 連結永遠指向最新版本。

```php
@task('update_symlinks')
    echo "Linking storage directory"
    rm -rf {{ $new_release_dir }}/storage
    ln -nfs {{ $app_dir }}/storage {{ $new_release_dir }}/storage

    echo "Linking .env"
    ln -nfs {{ $app_dir }}/.env {{ $new_release_dir }}/.env

    echo "Linking current release"
    ln -nfs {{ $new_release_dir }} {{ $app_dir }}/current
@endtask
```

如您所見，我們使用 `ln` 搭配 `-nfs` 參數將 `storage`, `.env`, `current` 指向新版本。

> `bootstrap/cache` 目錄可能需要相同處理。

#### 完整腳本

現在我們的任務腳本已經完成了，但仍需要確認 `deployer@192.168.1.1` 和 `/var/www/app` 目錄名稱是否符合您伺服器上的設定。

```php
@servers(['web' => 'deployer@192.168.1.1'])

@setup
    $repository = 'git@gitlab.com:<USERNAME>/example.git';
    $releases_dir = '/var/www/app/releases';
    $app_dir = '/var/www/app';
    $release= date('YmdHis');
    $new_release_dir = $releases_dir .'/'. $release;
@endsetup

@story('deploy')
    clone_repository
    run_composer
    update_symlinks
@endstory

@task('clone_repository')
    echo 'Cloning repository'
    [ -d {{ $releases_dir }} ] || mkdir {{ $releases_dir }}
    git clone --depth 1 {{ $repository }} {{ $new_release_dir }}
    cd {{ $new_release_dir }}
    git reset --hard {{ $commit }}
@endtask

@task('run_composer')
    echo "Starting deployment ({{ $release }})"
    cd {{ $new_release_dir }}
    composer install --prefer-dist --no-scripts -q -o
@endtask

@task('update_symlinks')
    echo "Linking storage directory"
    rm -rf {{ $new_release_dir }}/storage
    ln -nfs {{ $app_dir }}/storage {{ $new_release_dir }}/storage

    echo "Linking .env"
    ln -nfs {{ $app_dir }}/.env {{ $new_release_dir }}/.env

    echo "Linking current release"
    ln -nfs {{ $new_release_dir }} {{ $app_dir }}/current
@endtask

```

另外還有一件事情需要在執行自動佈署之前完成，就是第一次佈署需要手動複製應用程式的 `storage` 目錄到伺服器的`/var/www/app` 目錄下。您可能希望建立另一個 Envoy 任務來完成這件事。環境變數也需要為 Laravel 應用程式建立 `.env` 到相同路徑。每次佈署新版本將使用這些相同的資料。

> 如果遭遇目錄權限問題，可嘗試將目錄的權限要換成 `www-data` 或其他附加權限的方式如調整 Nginx `user`。

在本機執行 `php vendor/bin/envoy run deploy` 就可以佈署應用程式，但透過 [CI 環境](https://docs.gitlab.com/ee/ci/environments/index.html) Gitlab 也可以幫我們處理這個步驟。

接著，把剛剛寫的 `Envoy.blade.php` 推到檔案庫。為了讓事情單純一些我們直接 commit 到主線 `main` 不用其他分支。

## 使用 GItlab 持續整合

我們已經在 Gitlab 上準備好了我們的應用程式，我們當然可以手動佈署。但也可以進一步使用自動化[持續交付](https://about.gitlab.com/blog/2016/08/05/continuous-integration-delivery-and-deployment-with-gitlab/#continuous-delivery)的方式。我們會使用自動化測試檢查每一個 commit 盡可能提早發現問題，然後如果測試一切正常則佈署到目標環境。

[Gitlab CI/CD](https://docs.gitlab.com/ee/ci/index.html)支援 [Docker](https://www.docker.com/)來處理測試和佈署流程。如果您不熟悉 Docker 可以參考[設定自動建置](https://docs.docker.com/get-started/)

為了能夠使用 Gitlab CI/CD 建置, 測試和佈署，我們需要準備工作環境。因此我們將使用符合 Laravel 應用程式的最低需求的 Docker Image 。當然還有[其他方式](https://docs.gitlab.com/ee/ci/examples/php.html#test-php-projects-using-the-docker-executor)可以完成這個需求，但可能導致建置變慢，這不是我們想要。

### 建立 Container Image

在專案根目錄建立 [Dockerfile](https://gitlab.com/mehranrasulian/laravel-sample/blob/master/Dockerfile)

> 原文 Dockerfile 使用 php7 搭配 MySQL，下面範例則更新為 8 與搭配 PostgreSQL。如果您使用的是 MySQL 可以[參考原文](https://docs.gitlab.com/ee/ci/examples/laravel_with_gitlab_and_envoy/#configure-the-production-server)或上方連結。

```dockerfile
FROM php:8.1

RUN apt-get update

RUN apt-get install -y git curl libjpeg-dev libpng-dev libfreetype6-dev libbz2-dev libpq-dev libzip-dev

# RUN apt-get install -y nodejs npm

RUN apt-get clean

RUN docker-php-ext-configure pgsql -with-pgsql=/usr/local/pgsql

RUN docker-php-ext-install gd pdo_pgsql zip
# gd mbstring xml

RUN curl --silent --show-error "https://getcomposer.org/installer" | php -- --install-dir=/usr/local/bin --filename=composer

RUN composer global require laravel/envoy
```

我們使用[官方 PHP Docker Image](https://hub.docker.com/_/php)，由 Debian buster 搭配最基本的安裝，預先安裝了 PHP。然後使用 `docker-php-ext-install` 來安裝需要的 PHP 擴充。

### 設定 Gitlab 容器註冊

現在我們有了 `Dockerfile` 讓我們繼續建置和 push 到 [Gitlab Container Registry](https://docs.gitlab.com/ee/user/packages/container_registry/index.html)

> Registry 是儲存 Image 的地方。開發者可能會需要維護私有的 Image 。使用 Gitlab Container Registry 表示您不需設定和管理其他服務或使用其他 Registry

在您的 Gitlab 專案底下找到 **Container Registry**。如果您沒看到該功能可能需要到 **Seetings > General > Visibility 的 Project Features, Permissions** 開啟該功能。

要使用 Container Registry 首先需要登入 Gitlab Registry。確認本機安裝好了 Docker 然後執行下面指令

```sh
$ docker login registry.gitlab.com
```

接著 `build` 和 `push`

```sh
$ docker build -t registry.gitlab.com/<USERNAME>/example .
$ docker push registry.gitlab.com/<USERNAME>/example
```

恭喜! 您剛剛 push 了第一個 Docker Image 到 Gitlab Registry。如果您重新載入頁面將會看到剛剛的 Image。當然您也可以使用 [Gitlab CI/CD](https://about.gitlab.com/blog/2016/05/23/gitlab-container-registry/#use-with-gitlab-ci) 來建置和 push Docker Image。

後續我們將會在 `.gitlab-ci.yml` 使用這個 Image 來處理測試流程和佈署。現在讓我們先 commit `Dockerfile`

### 設定 Gitlab CI/CD

為了能夠使用 Gitlab CI/CD 來建置和測試我們的應用程式，我們需要 `.gitlab-ci.yml` 這和 Circle CI 與 Travis CI 類似，但是 Gitlab 內建的。

我們的 `.gitlab-ci.yml` 如下

```yml
image: registry.gitlab.com/<USERNAME>/example:latest

services:
  - postgres:12.2-alpine

variables:
  POSTGRES_DB: $POSTGRES_DB
  POSTGRES_USER: $POSTGRES_USER
  POSTGRES_PASSWORD: $POSTGRES_PASSWORD
  POSTGRES_HOST_AUTH_METHOD: trust

stages:
  - test
  - deploy

unit_test:
  stage: test
  script:
    - cp .env.example .env
    - composer install
    - php artisan key:generate
    - php artisan migrate
    - venor/bin/phpunit

deploy_production:
  stage: deploy
  script:
    - 'which ssh-agent || ( apt-get update -y && apt-get install openssh-client -y )'
    - eval $(ssh-agent -s)
    - echo "$SSH_PRIVATE_KEY" | tr -d '\r' | ssh-add -
    - mkdir -p ~/.ssh
    - '[[ -f /.dockerenv ]] && echo -e "Host *\n\tStrictHostKeyChecking no\n\n" > ~/.ssh/config'
    # - ~/.composer/vendor/bin/envoy run deploy --commit="$CI_COMMIT_SHA"
    - ~/.config/composer/vendor/bin/envoy run deploy --commit="$CI_COMMIT_SHA"
  environment:
    name: production
    url: http://192.168.1.1
  when: manual
  only:
    - main
```

> 注意如果無法取得變數，可能是變數預設只 export 到受保護的分支。另外上面 `~/.composer/vendor/bin/envoy` 的路徑可能會不同，`~/.config/composer/vendor/bin/envoy` 為測試可執行的路徑。

### Image 與服務

[Runner](https://docs.gitlab.com/ee/ci/runners/index.html) 主要會執行 `.gitlab-ci.yml` 定義的腳本。`image` 設定則告訴 Runner 使用哪個 Image 。`services` 定義額外的 Image 可以和主要 Image 連結。這裡我們使用之前定義的 Image，同時加上使用 `postgres`。

如果您希望測試不同的 PHP 版本和資料庫，您可以定義不同的 `image` 和 `services`。[參考 `services` 可連接的 Image 設定](https://docs.gitlab.com/ee/ci/services/index.html)

### CI/CD 變數

Gitlab CI/CD 支援 [CI/CD 變數](https://docs.gitlab.com/ee/ci/yaml/index.html#variables)。這裡我們使用了 PostgreSQL 作為資料庫。因此我們需要設定 PostgreSQL 的名稱和密碼`POSTGRES_DB`, `POSTGRES_USER`, `POSTGRES_PASSWORD` 。

> MySQL 的變數:
>
> ```
> variables:
> 	MYSQL_DATABASE: homestead
> 	MYSQL_ROOT_PASSWORD: secret
> 	DB_HOST: mysql
> 	DB_USERNAME: root
> ```

同時 Laravel 的 `.env` 的 `DB_HOST` 要從 127.0.0.1 換成 `postgres`。或者把要覆寫的環境變數寫在上面 `variables` 下。

例如:

```yaml
variables:
  POSTGRES_DB: my_db
  POSTGRES_USER: homestead
  POSTGRES_PASSWORD: secret
  POSTGRES_HOST_AUTH_METHOD: trust
  DB_HOST: postgres
  DB_USERNAME: homestead
	DB_PASSWORD: secret
```

更多環境變數的使用與說明可以參考 Docker 頁面說明:

- [PostgreSQL](https://hub.docker.com/_/postgres)
- [MySQL](https://hub.docker.com/_/mysql)

### 第一個任務 - Unit Test

我們將需要的指令定義到 `unit_test` 的 `script` 底下。這些指令包含 Artisan 指令是用來準備 Laravel 應用程式的最終我們會執行 `phpunit` 測試。

```yml
unit_test:
  stage: test
  script:
    - cp .env.example .env
    - composer install
    - php artisan key:generate
    - php artisan migrate
    - vendor/bin/phpunit
```

### 佈署到正式環境

`deploy_production` 任務則是將應用程式佈署到伺服器。要使用 Envoy 我們需要 `$SSH_PRIVATE_KEY`，若已經加入到 Gitlab 專案則 Envoy 就可以執行。注意 Gitlab 預設只將 CI/CD 變數 export 到受保護的分支。如果您解除了保護有可能無法取得變數。

如之前提到的，Gitlab 也提供持續交付 `environment` 設定，可以告訴 Gitlab 任務要佈署到正式環境。`url` 是用來產生連結的您可以在 Gitlab 專案 **Deployents > Environments** 找到開啟網站的連結。

`only` 則是告訴 Gitlab CI/CD 任務只有在 `main` 分支 Pipeline 建置的時候執行。

最後 `when: manual` 則是使用手動執行，讓我們可以自行在 Gitlab Pipeline 頁面上點擊執行。

```yaml
deploy_production:
  stage: deploy
  script:
    - 'which ssh-agent || ( apt-get update -y && apt-get install openssh-client -y )'
    - eval $(ssh-agent -s)
    - echo "$SSH_PRIVATE_KEY" | tr -d '\r' | ssh-add -
    - mkdir -p ~/.ssh
    - '[[ -f /.dockerenv ]] && echo -e "Host *\n\tStrictHostKeyChecking no\n\n" > ~/.ssh/config'
    # - ~/.composer/vendor/bin/envoy run deploy --commit="$CI_COMMIT_SHA"
    - ~/.config/composer/vendor/bin/envoy run deploy --commit="$CI_COMMIT_SHA"
  environment:
    name: production
    url: http://192.168.1.1
  when: manual
  only:
    - main
```

您可能也希望在[Staging environment](https://about.gitlab.com/blog/2021/02/05/ci-deployment-and-environments/)加入其他任務，方便在最後佈署之前測試您的應用程式。

### 開啟 Gitlab CI/CD

到此我們已經準備好所有需要的東西了。要使用 Gitlab CI/CD 您只需要 commit 我們的 `.gitlab-ci.yml` 且 push 到 `main` 分支。就會觸發 Pipeline 執行了。

在單元測試通過之後可以在頁面上點擊按鈕執行 `deploy_production` 。如果一切順利執行您的應用程式就會佈署。

佈署成功之後可以到 **Deployments > Environments** 查看。

在我們這個範例，您可能會想理解我們應用程式的目錄架構。在佈署之後 `/var/www/app` 下會有三個目錄 `current`，`releases`，`storage` 以及 `.env` 檔案。`current` 為軟連結指向最新版本。`.env` 是 Laravel 使用的環境變數。如果您 `cd current` 應該會發現 `.env` 是指向 `/var/www/app/.env` 的連結，同樣 `storage` 也是指向上層目錄。

到此我們完成了全部的教學。您可以依據您的需求做更多的調整。

## 參考資源

- [Test and deploy Laravel applications with GitLab CI/CD and Envoy](https://docs.gitlab.com/ee/ci/examples/laravel_with_gitlab_and_envoy/#configure-the-production-server)
- [Gitlab CI/CD using PostgreSQL](https://docs.gitlab.com/ee/ci/services/postgres.html)
- [Using SSH keys with GitLab CI/CD](https://docs.gitlab.com/ee/ci/ssh_keys/)
- [~/.composer not found issue](https://forum.gitlab.com/t/gitlab-cicd-for-laravel-app-not-deploying/28031)
