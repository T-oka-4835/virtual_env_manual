# <font color="#03a9f4">仮想環境構築手順書</font>

*** 

## 手順書の内容

Vagrantを用いて仮想環境を構築し、  
構築した環境を基にLaravelのアプリを動かすまでの流れを説明しています。  

- スタート: vagrant用ディレクトリ作成
- ゴール: Laravel のプロジェクトに実際にログインすること

また、本書内で(注)となっている箇所に関しては、下部の追加説明にて詳しい説明を  
書いておりますので、適宜ご参照頂ければと存じます。

*** 

## バージョン一覧

| 名前 | バージョン |
| ---- |---- |
| PHP | 7.3 |
| Laravel | 6.0 |
| Nginx | 
| MySQL | 5.7 |
| OS | CentOS7 |

***

## 目次

1. - 1-1 Vagrant 作業用ディレクトリの作成
   - 1-2 Vagrantfile の編集
   - 1-3 Vagrant プラグインのインストール  

2. - 2-1 Vagrant を使用してゲストOSの起動
   - 2-2 ゲストOSへのログイン
   - 2-3 パッケージのインストール
   - 2-4 PHPのインストール
   - 2-5 composerのインストール

3. - 3-1 Laravel のInstallと、Project の作成

4. - 4-1 データベースのインストール
   - 4-2 データベースの作成
   - 4-3 Laravel側でDBを使用するための設定
   - 4-4 テーブルの作成

5. - 5-1 sample_appへのログイン機能の実装

6. - 6-1 Nginx のインストール
   - 6-2 Nginx内でLaravelを動かす
   - 6-3 アプリが動かなかった時(ファイヤーウォール編)
   - 6-4 アプリが動かなかった時(Forbidden 403編)
   - 6-5 アプリが動かなかった時(操作権限編)

7. - 7-1 実際に新規登録

8. - 8-1 追加説明

9. - 9-1 所感

10. - 10-1 参考サイト一覧 
***

## 環境構築の流れ
### 1-1 Vagrant 作業用ディレクトリの作成  
まず最初に、ご自身の作業用ディレクトリかデスクトップに、Vagrant の作業用ディレクトリを作成してください。  
※作成したディレクトリを後々移動すると、ゲストOSへのログインが上手く行かなくなってしまうので、ご注意ください。

```
ディレクトリ作成のコマンド  
mkdir laravel_app_by_vagrant  
```

ディレクトリは作成できましたか?  
作成できたら、下記の順番で進めて行きましょう。

``` 
mkdir で作成したディレクトリに移動  
cd laravel_app_by_vagrant  

Vagrantfileを作成  
※centos/7 はbox名(注1)を差しています。
vagrant init centos/7  
``` 

vagrant init を実行後、"A Vagrantfile has been placed in this directory" がターミナル常に表示されれば、問題ありません。

### 1-2 Vagrantfile の編集
今回は三箇所の編集を行います。  
ファイルをエディタで開いてください。

```
1. config.vm.network "forwarded_port", guest: 80, host: 8080(注2)

2. config.vm.network "private_network", ip:"192.168.33.10"(注3)
```

上記2つのコメントを外してください。

また、

```
config.vm.network "private_network", ip:"192.168.33.10"
を
config.vm.network "private_network", ip:"192.168.33.19"
```

へと変更してください。

```
3. config.vm.synced_folder "../data", "/vagrant_data" 
   のコメントを外し、  
   config.vm.synced_folder "./", "/vagrant", type:"virtualbox"(注4)
```

に変更してください。

### 1-3 Vagrant プラグインのインストール
Vagrant には様々なプラグイン(拡張機能)が用意されており、今回はvagrant-vbguest(注5) という   
プラグインをインストールします。

```
プラグインのインストールコマンド
vagrant plugin install vagrant-vbguest  

正常にインストールが完了しているか確認
vagrant plugin list 

バージョンが表示されれば、問題なくインストールされています。
vagrant-vbguest (0.21.0, global)
```

以上で、仮想環境を構築する準備は終了です。  
次で、いよいよゲストOSを起動させます。

### 2-1 Vagrantを使用してゲストOSの起動
laravel_app_by_vagrantディレクトリにて以下のコマンドを実行することで、ゲストOSを立ち上げることができます。

```
vagrant up
```

※この際に、vbguest のバージョンによってエラーが出る場合があります。  
その際には、`Vagrantで共有フォルダのエラーが出るのでその対応` の記事を参考に対応をお願い致します。

### 2-2 ゲストOSへのログイン
起動が正常に終了すれば、現在使用しているPCのOSであるホストOSの上に、全く別のゲストOSが立ち上がったことになります。  

今回は、VirtualBoxが提供するネットワークを通じて、ターミナル上でホストOSからゲストOS(リモートマシン)にログインします。  

```
ssh はリモートマシンにログインするコマンド
vagrant ssh 
```

実行後、"[vagrant@localhost ~]$" となっていれば、ゲストOSにログインしていることになります。

#### 今後の注意点
この先でのコマンドや操作において、今ご自身がホストOSとゲストOSのどちらをコマンドラインで操作しているのか、常に意識しながら進めてください。

```
- ホストOS  
ユーザー名noMacbook:~ ユーザー名$
- ゲストOS  
[vagrant@localhost ~]$
```

### 2-3 パッケージのインストール

```
このコマンドで、開発に必要なパッケージを一括でインストールすることができる(注6〜9)  
sudo yum -y groupinstall "development tools"  
```

### 2-4 PHP のインストール
次は、PHPをインストールしていきます。  
yumコマンドを使用してPHPをインストールした場合、古いバージョンのPHPがインストールされてしまいます。  
そのため、yumではなく外部パッケージツールをダウンロードして、そこからPHPをインストールしていきます。

```
sudo yum -y install epel-release wget (注10)

sudo wget http://rpms.famillecollet.com/enterprise/remi-release-7.rpm (注11)

sudo rpm -Uvh remi-release-7.rpm (注12, 13)

sudo yum -y install --enablerepo=remi-php73 php php-pdo php-mysqlnd php-mbstring php-xml php-fpm php-common php-devel php-mysql unzip (注14)

php -v  

バージョンが表示されれば、問題なくインストールされています。  

PHP 7.3.28 (cli) (built: Apr 27 2021 13:57:06) ( NTS )
```

PHPのバージョンが確認できれば、PHPのインストールは完了です。 

### 2-5 composerのインストール 
次にPHPのパッケージ管理ツールであるcomposerをインストールしていきます。  

```
composer の一部である、installer ファイルと、セットアップ用PHPスクリプトの`composer-setup.php`をダウンロード
php -r "copy('https://getcomposer.org/installer', 'composer-setup.php');"  

ダウンロードした composer-setup.php を実行して、Composer の実行ファイルを作成
php composer-setup.php

composert-setup.phpは不要なので削除
php -r "unlink('composer-setup.php');"  

どのディレクトリにいてもcomposerコマンドを使用できるようfileの移動を行う
sudo mv composer.phar /usr/local/bin/composer 

バージョンの確認
composer -v  

以下が出力されれば、問題なくインストールできています。
  ______
  / ____/___  ____ ___  ____  ____  ________  _____
 / /   / __ \/ __ `__ \/ __ \/ __ \/ ___/ _ \/ ___/
/ /___/ /_/ / / / / / / /_/ / /_/ (__  )  __/ /
\____/\____/_/ /_/ /_/ .___/\____/____/\___/_/
                    /_/
Composer version 2.0.14 2021-05-21 17:03:37

```

ここまでの処理が完了しましたら、ゲストOS内にPHP と composer コマンドの実行環境が整ったことになります。


### 3-1 LaravelのInstallとProjectの作成
```
下記のコマンドで、LaravelのインストールとProject の作成を同時に行う
composer create-project laravel/laravel --prefer-dist sample_app 6.0(注15,16)
```

sample_app というディレクトリが作成されましたか? 作成されていれば、下記を実行してください。

```
sample_app ディレクトリに移動
cd sample_app  

Laravel のバージョンを確認
php artisan --version  

Laravel のバージョンが確認できれば完了です。 
Laravel Framework 6.20.27
```

### 4-1 データベースのインストール  
今回使用するデータベースである、MySQLのバージョン5,7をインストールします。  
centos7は、デフォルトでmariaDBというデータベースがインストールされていますが、MariaDBはMySQLと互換性があるため、そのままMySQLのインストールを進めていきます。

```
wget で、mysql.com から、mysql5.7のファイルをダウンロード 
sudo wget https://dev.mysql.com/get/mysql57-community-release-el7-7.noarch.rpm  

rpm -Uvh で、パッケージを更新
sudo rpm -Uvh mysql57-community-release-el7-7.noarch.rpm  

MySQLのインストール
sudo yum install -y mysql-community-server 

バージョンの確認
mysql --version

バージョンが表示されれば、問題なくインストールされています。  

mysql  Ver 14.14 Distrib 5.7.34, for Linux (x86_64) using  EditLine wrapper
```

バージョンの確認ができたら、インストール完了です。  
次にMySQLを起動し接続を行います。  
ただその前に、デフォルトでrootパスワードが設定されているため、passwordを調べ、接続し、password の再設定を行う必要があります。

```
MySQLの起動
sudo systemctl start mysqld  

sudo cat /var/log/mysqld.log | grep 'temporary password' (注17,18)
```

このコマンド実行後、下記のように表示されると、問題ないです。

```
2017-01-01T00:00:00.000000Z 1 [Note] A temporary password is generated for root@localhost: hogehoge 
```

hogehoge と記載されている箇所に存在するランダムな文字列がパスワードとなります。

出力したランダムな文字列をコピーし、以下のコマンドを実行してください。

```
MySQLへと接続
mysql -u root -p

パスワードの入力
Enter password:  

mysql >
```

接続できましたでしょうか。
次に接続した状態でpasswordの変更を行います。

MySQL5.7のパスワードポリシーは厳格で開発段階では非常に面倒のため、以下の設定を行いシンプルなパスワードに初期設定できるようにMySQLの設定ファイルを変更します。

```
sudo vi /etc/my.cnf
```

```
[mysqld]

read_rnd_buffer_size = 2M  
datadir=/var/lib/mysql  
socket=/var/lib/mysql/mysql.sock

下記の一行を追加
validate-password=OFF
```

編集後はMySQLサーバの再起動が必要です。

```
sudo systemctl restart mysqld
```

再起動後、再度MySQLにログインし、下記のコマンドを実行してください。

```
※新たなpasswordには、必ず大文字小文字の英数字 + 記号かつ8文字以上の設定をする必要があります。  
上記が無事終了すれば、MySQLの導入と設定が完了となります。
mysql > set password = "新たなpassword";
```

### 4-2 データベースの作成
実際にLaravelのTodoアプリケーションを動かす上で使用するデータベースの作成を行います。

```
新しいデータベースを作成
mysql > create database sample_app;

データベースの一覧を表示
mysql > show databases;

+--------------------+
| Database           |
+--------------------+
| information_schema |
| mysql              |
| performance_schema |
| sample_app         |
| sys                |
+--------------------+

```

で、sample_appが表示されていれば、作成は完了となります。

### 4-3 Laravel側でDBを使用するための設定
Laravelに今回使用するDBは、XXXだよとDBの接続情報を教えてあげる必要があります。  
Laravelのプロジェクト直下に .env というfileがありますので、これに情報を書いていきます。

```
APP_NAME=Laravel  
APP_ENV=local  
APP_KEY=base64:Rs6WHziGChaNJGg0o1mBOidiKaFZPkKeHNt6aGamvYk=  
APP_DEBUG=true  
APP_LOG_LEVEL=debug  
APP_URL=http://localhost  

DB_CONNECTION=mysql  
DB_HOST=127.0.0.1  
DB_PORT=3306  
DB_DATABASE=sample_app      # 編集  
DB_USERNAME=root            # 編集   
DB_PASSWORD=                # 編集(データベースのインストールで設定したパスワードを入力)   
# 省略
```

上記のように変更することによって、Laravelで先程作成したdatabaseが使用可能になります。

### 4-4 テーブルの作成
Laravelをインストールした時から、databases/migrations/には以下の2ファイルが元々用意されています。これがテーブル作成の素（もと）になります。

- 2014_10_12_000000_create_users_table
- 2014_10_12_100000_create_password_resets_table 

※2014_10_12_100000_create_password_resets_tableは今回使用しませんので、ファイル丸ごと削除してしまいましょう。

ファイルを削除したら、artisan コマンドをこのファイルで実行してみましょう。  
artisanコマンドは必ずLaravelプロジェクト直下に移動してから実行しましょう。  
今回だとゲストOS内のsample_app/ です。

```
php artisan migrate (注19)
```

以下のようなメッセージが表示されれば成功です。

```
Migration table created successfully.
Migrating: 2014_10_12_000000_create_users_table
Migrated:  2014_10_12_000000_create_users_table
```

うまくいかない場合は、  
- MySQLが立ち上がっているか  
- .envファイルの記述が間違っていないか

確認しましょう。

created successfully のメッセージが表示されていれば、users テーブルが作成されています。  
これがユーザーの情報を保存する、メインのテーブルになって来ます。 

### 5-1 ログイン機能の実装
次は、sample_app にログイン機能を付けていきます。  
Laravel 6.0からはログイン機能は、laravel/uiという名前の別パッケージとして管理されるようになりました。そのため、まずはこのパッケージをインストールします。

```
composer require laravel/ui:^1.0 --dev
```

インストールが完了したら、下記のコマンドを実行してください。

```
php artisan ui vue --auth
```

php artisan ui vue --authコマンドで、認証に必要なすべてのビューがresources/views/authディレクトリに生成されます。

ここまで完了したら、次から実際にサーバーを立ち上げ、ゲストOS内で動かすための処理を行っていきます。  
複雑な作業も多くなって来ますが、一つずつ落ち着いて実行していきましょう。

<br>

### 6-1 Nginxのインストール
まずは、Nginx(注20)の最新版をインストールしていきます。  
vi エディタを使用して以下のファイルを作成します。

```
sudo vi /etc/yum.repos.d/nginx.repo
```

ファイルが作成されたら、以下の内容を書き込んでください。

```
[nginx]  
name=nginx repo  
baseurl=https://nginx.org/packages/mainline/centos/\$releasever/\$basearch/  
gpgcheck=0  
enabled=1
(注21)
```

書き終えたら保存し、以下のコマンドでNginx のインストールを実行します。

``` 
Nginx のインストール
sudo yum install -y nginx  

バージョンの確認
nginx -v  

下記のように表示されると、問題なくインストールできています。

nginx version: nginx/1.19.10
``` 

Nginx のバージョンが確認できれば、Nginx を起動させてみましょう。

```
Nginx の起動
sudo systemctl start nginx  
```

ブラウザにて http://192.168.33.19 と入力し、NginxのWelcomeページが表示されましたら、問題なく動いています。

### 6-2 Laravelを動かす
Laravel を動かすために、Nginx内の設定ファイルと、php-fpmの設定ファイルの編集を行います。

使用しているOSがCentOSの場合、/etc/nginx/conf.d ディレクトリ下の default.conf ファイルが設定ファイルとなります。

```
sudo vi /etc/nginx/conf.d/default.conf
```

下記のように変更してください。 

```
server {
  listen       80; (注22)
  server_name  192.168.33.19; # Vagranfileでコメントを外した箇所のipアドレスを記述してください。
  root /vagrant/laravel_app/public; # 追記 (注23)
  index  index.html index.htm index.php; # 追記

  #charset koi8-r;
  #access_log  /var/log/nginx/host.access.log  main;

  location / {
      #root   /usr/share/nginx/html; # コメントアウト
      #index  index.html index.htm;  # コメントアウト
      try_files $uri $uri/ /index.php$is_args$args;  # 追記 (注24)
  }

  # 省略

  # 該当箇所のコメントを解除し、必要な箇所には変更を加える
  # 下記は root を除いたlocation { } までのコメントが解除されていることを確認してください。

  location ~ \.php$ {
  #   root           html;
      fastcgi_pass   127.0.0.1:9000;
      fastcgi_index  index.php;
      fastcgi_param  SCRIPT_FILENAME  /$document_root/$fastcgi_script_name;  # $fastcgi_script_name以前を /$document_root/に変更
      include        fastcgi_params;
  }

  # 省略

```

Nginx の設定ファイルの変更は、以上です。  
次に php-fpm の設定ファイルを編集していきます。

```
sudo vi /etc/php-fpm.d/www.conf
```

変更箇所は以下になります。

```
24行目近辺
user = apache
# ↓ 以下に編集
user = nginx

group = apache
# ↓ 以下に編集
group = nginx
```

設定ファイルの変更に関しては、以上となります。
では、下記の2つのコマンドを実行し、ブラウザを確認してみましょう。

```
nginx の再起動
sudo systemctl restart nginx

php-fpmの起動
sudo systemctl start php-fpm
```

再度ブラウザにて、 http://192.168.33.19 を入力して確認してください。
アプリケーションを動かすことはできたでしょうか。  

もし動かなかった場合、下記の処理を1つずつ行ってみてください。

***

### 6-3 ファイヤーウォールに対してhttp通信によるアクセスを許可
表示されている文言の下部に「ファイヤーウォール」(注25)という単語が確認できますでしょうか？  
聞きなれない単語かと思いますが、セキュリティの観点では「ファイヤーウォール」は大切な機能であるため、起動状態のままホストOS側からアクセスできるようにしてあげましょう。

Vagrantfileの編集をした際、
```
config.vm.network "forwarded_port", guest: 80, host: 8080 
```

と記述されていた箇所のコメントを解除したと思います。

この 80 という数字は、httpという通信を行うためのポートと呼ばれる窓口番号です。  

なのでファイヤーウォールに対してこの80ポートを経由したhttp通信によるアクセスを許可するためのコマンドを実行します。

```
ファイヤーウォールの起動
sudo systemctl start firewalld.service
sudo firewall-cmd --add-service=http --zone=public --permanent

新たに追加を行ったのでそれをファイヤーウォールに反映させるコマンドも合わせて実行
sudo firewall-cmd --reload
```

では、一旦この状態で画面を確認してください。  
Laravelのwelcome画面が表示されたでしょうか。
もし表示されない場合は、次の処理に進んでください。

### 6-4 Forbidden 403 というエラーが出た場合
(Laravelのwelcomeページが表示されたり、他のエラーが出ている場合は3に進んでください。)

viエディタを使用してSELinux(注26)の設定を変更します。
「SELinux コンテキスト」の不一致によりエラーが出ているので、SELinuxを無効化します。

```
sudo vi /etc/selinux/config
```

viエディタが開き設定ファイルが表示されるので下記の部分を探してください。

```
# This file controls the state of SELinux on the system.
# SELINUX= can take one of these three values:
# enforcing - SELinux security policy is enforced.
# permissive - SELinux prints warnings instead of enforcing.
# disabled - No SELinux policy is loaded.
SELINUX=enforcing
```

一番下を、

```
SELINUX=disabled
```

に変更した後、設定を反映させるためにゲストOSを再起動させます。

```
exit
vagrant reload
```

リロードが完了したら再度ゲストOSにログインしましょう。

```
vagrant ssh

コマンドで打ち、Disabled になっていれば完璧
getenforce

Disabled
```

再度Nginx を起動してみてください。

```
sudo systemctl start nginx
```

これでForbidden 403 のページは出なくなったはずです。

### 6-5 操作権限の付与
画面上に、以下のようなLaravelのエラーが表示される場合は、操作権限がないことが原因となります。

```
The stream or file "/vagrant/laravel_app/storage/logs/laravel.log" could not be opened: failed to open stream: Permission denied
```

これは 先程php-fpmの設定ファイルの user と group を nginx に変更したと思いますが、ファイルとディレクトリの実行 user と group に nginx が許可されていないため起きているエラーです。  
試しに以下のコマンドを実行してみてください。

```
ls -la ./ | grep storage && ls -la storage/ | grep logs && ls -la storage/logs/ | grep laravel.log
```

出力結果から、storageディレクトリも logsディレクトリも laravel.logファイルも全て user と group が
vagrant となっていますので、これでは nginx というユーザーの権限をもってlaravel.logファイルへの書き込みができません。

そのため、以下のコマンドを実行して、nginx というユーザーでもログファイルへの書き込みができる権限を付与しましょう。

```
sample_appのディレクトリへと移動
cd /vagrant/sample_app

操作権限の付与
sudo chmod -R 777 storage(注27, 注28)
```

権限を付与した後に、再度 http://192.168.33.19 にアクセスしてみてください。

***

<br>

ここまで完了すれば、Laravelの最初のページが表示されると思います!! 

### 7-1 実際の新規登録
それでは、実際に新規登録を行ってみましょう。  
http://192.168.33.19/register  
が新規登録のURLとなります。

- 名前
- メールアドレス 
- パスワード
- パスワードの確認

を入力し、Resister を押しましょう。

画面一番上にLaravel、次に自分の名前が記載されているページが現れれば、完了となります。  
お疲れ様でした!!

<br>

***
## 8-1 追加説明
手順書の流れをより理解頂くために、こちらに追加の説明を記述しています。  
より詳細を知りたい場合には、参考サイトにリンクを載せてますので、ご確認頂ければ幸いです。

<br>

### (注1) Box  
仮想マシンのテンプレートとなるファイルです。  
手動で１からOSをインストールしていくのは大変ですが、例えばCentOSのBoxでCentOS環境を構築することができます。  
今回はCentOS7 のBoxファイルを使用しています。

### (注2) forwarded_port  
Vagrant仮想マシン内でサーバーを立ち上げた場合、デフォルトでは外の世界(ホスト側)からアクセスできません。  
外の世界から仮想マシン内のサーバにアクセスできるようにするための方法として、ポートフォワードがあります。

ポートフォワードの設定では、ホストマシンのあるポート番号を、特定の仮想マシンのポート番号にマッピングします。  
つまりホストマシンの特定のポート番号にアクセスが会ったときに、ホストマシンが仮想マシンに対してリクエストを転送することで、間接的に仮想マシンへと接続されます。  

今回の設定では、仮想マシン内のHTTP(80) を、ホスト側のポート8080にマッピングしています。

### (注3) private_network  
プライベートネットワークとは、プライベートIPアドレスで構築されたネットワークのことです。  
社内などの限られた空間にある端末同士でプライベートIPアドレスを用いて通信を行うため、外部からのアクセスの心配がなく、セキュアなネットワーク空間と言えます。  

２番目のコメントを外すことで、ホスト(ローカルマシン)から仮想マシンへ接続ができるようになります。  
今回のipは192.168.33.19 のため、最後を19に変更してください。

### (注4) config.vm.synced_folder "./", "/vagrant", type:"virtualbox
./はカレントディレクトリ(laravel_app_by_vagrant)を示しており、  
- ホストOSのlaravel_app_by_vagrantディレクトリ
  
  と

- ゲストOS(Vagrant)の/vagrant ディレクトリ

をリアルタイムで同期するための設定です。

### (注5) vagrant-vbguest   
vagrant-vbguestは、初めに追加したBoxの中にインストールされているGuestAdditions というもののバージョンを、VirtualBoxのバージョンに合わせて最新化してくれるプラグインです。

### (注6) sudo
vagrant ssh でゲストOSにログインした場合、ログインユーザーはvagrant になります。
sudo と入れることで、「スーパーユーザー(rootユーザー)」の権限が必要なコマンドを、sudoコマンド経由で実行させることができます。 

### (注7) yum
yum とは、LinuxのRedHat系ディストリビューション(CentOS)で利用されるパッケージ管理ツールです。  
また、-y というオプションをつけることで、インストール実行途中でyes/no と聞かれた場合に全て自動でyesと回答してくれます。

### (注8) groupinstall 
yum install とは異なり、まとめてパッケージのインストールが可能になります。

### (注9) development tools 
今回インストールするグループパッケージです。  
具体的にどのようなパッケージがインストールされるかは、参考サイトの`yum groupinstall "Development tools" で入るパッケージ一覧`でご確認ください。

### (注10) epel-release 
epel とは、「サードパーティ・リポジトリ」と呼ばれる、ディストリビューション本家以外が提供するリポジトリ のことです。  
- 使いたいアプリケーションが標準のYumリポジトリ に含まれていない
- 含まれていてもバージョンが古い

などの場合に活用されます。  
今回は次で使うRemiリポジトリ をインストールするためには、epel リポジトリ も必須となって来るので、先にインストールします。

### (注11) wget 
wget はURLを指定することで指定先URLのファイルをダウンロードすることができるコマンドです。  
今回は rpms.famillecollet.com と言うドメイン名を持つサーバのenterprise ディレクトリ下のremi-release-7.rpm というファイルをダウンロードしています。

### (注12) rpm 
rpmとは、RedHatという会社が開発した、パッケージ管理ツールです。  
ただyum と異なる点として、rpmはパッケージ自体を管理することはできますが、その管理対象のパッケージの依存関係までは管理することはできません。

### (注13) remi 
remi というプロジェクトのリポジトリを利用することで、yumを使って必要なバージョンのPHPをインストールすることができます。

### (注14) install 以降の処理
install 以降で、PHPのインストールと同時に、PHPアプリケーションを動かす上で必要となる拡張機能をインストールしています。  

| コマンド名 | 役割 |
| ---- |---- |
| enablerepo=remi-php73 | 今回使用するPHP7.3 |
| php-pdo | PHPとデータベースを接続するためのもの |
| php-mysqlnd | 遅延接続やクエリのキャッシュなどの機能が搭載されている |
| php-mbstring | 日本語などマルチバイト文字が使えるようになる |
| php-xml | PHPからXMLを取り扱うために必要 |
| php-fpm | 主に高負荷のサイトで有用な追加機能を用意している |
| php-common | php package と php-cli package の両方で使われているファイルを含んでいる |
| php-devel | 開発に必要なヘッダファイル等が含まれている |
| php-mysql | MySQL データベースサーバーへとアクセスできるようにするためのもの |
| unzip | zip ファイルからファイルを取り出すためのコマンド |

### (注15) create-project
Composer のcreate-project コマンドを実行することで、Laravelをインストールできます。今回は6.0のバージョンをインストールしています。  
また、6.0の前がproject 名で、今回はsample_appとしています。

### (注16) prefer-dist
パッケージをダウンロードする方法は、
- prever-source 
- prefer-dist 

の２種類あります。  
dist の方が安定したバージョンであるため、今回はdistを使っています。

### (注17) catコマンド
ファイルの中身を見ることができます。  
例えば、fileAの中身を見たいとき、cat fileAと打つことで、fileAの中身を出力することができます。

### (注18) grepコマンド
ファイルに特定の文字列が存在するか検索する際に使うコマンドです。  
grep の前の| はパイプラインと呼ばれ、左辺 | 右辺と指定してコマンドを実行することで、左辺の実行結果を右辺に引き渡し、右辺で実行することができます。 

左辺のcat /var/log/mysqld.log というファイルを出力した結果を右辺に渡し、右辺ではその結果からtemporaray password という文字列を持つ行を出力しました。

### (注19) php artisan migrate 
アプリケーションで用意したマイグレーションを全て実行するためのコマンドです。  

マイグレーションとはデータベースのバージョンコントロールのような機能で、どのようなSQLをどの順番で実行したかを管理してくれます。

メリットとしては、データベースをあるべき状態に再現することが容易になるため、
- ある環境にアプリケーションをリリースする
- 新しいメンバーを迎える

などの際に、効率よく進めることができます。

### (注20) Nginx 
Nginx は、webサーバーソフトの一つです。  

※Webサーバーソフトとは、Webサーバーを構成するソフトウェアの総称で、HTTPをプロトコルとして通信し、Webブラウザなどからのリクエストに応じてHTMLファイルや画像ファイルなどを送信する機能を持っています。

以前はApache と呼ばれるwebサーバーソフトが多く利用されていましたが、近年はNginx が急速にシェアを伸ばしています。

- 処理能力が高く、高速で動作する
- メモリ消費が少なく、高い深谷多くの同時接続に耐えられる

などの強みを持っています。

### (注21) nginx インストール時の流れ
今回のnginx のインストールに関しては、公式に配布されているプリコンパイルパッケージのリポジトリを利用してインストールしています。  

nginxにはStableとMainlineの2つのバージョンが存在し、yumリポジトリのURLも両者で分かれています。  

- 通常は、新機能追加とバグフィックスが常時行われるMainlineバージョン
- 新機能は必要無く、APIの互換性を重視する場合は、Stableバージョン

と使い分けることができます。今回はmainline を使用します。

追記した内容の具体的な意味については、[Nginx 公式](https://nginx.org/en/)をご参照ください。

### (注22) listen
listen には、サーバーがリクエストを受け付けるIPアドレスやポート番号、あるいはUNIXドメインソケットを設定します。  

IPアドレスを指定するときは、  
```
listen IPアドレス:ポート番号;
```

としますが、ポート番号のデフォルト値は80であるため、今回のように

```
listen 80;
```

と記述することができます。

### (注23) root(ドキュメントルート)
ドキュメントルートを設定します。  
ドキュメントルートとは、サイトを公開する為に決められた場所（ディレクトリ）であり、また公開されるディレクトリの最上部にもなる部分です。  
この場所にサイトを設置することでインターネット上にサイトが公開されます。

### (注24) try_files
try_filesディレクティブには存在をチェックするファイルやディレクトリと、存在しなかったときにリダイレクトするURIのパスを指定します。  

try_filesという名前の通り、指定したファイルやディレクトリの存在を順番に調べ、存在すれば、そのファイルやディレクトリに対応したファイルを返します。一つも存在しなかったら、最後に記述したパスに内部リダイレクトします。

### (注25) ファイヤーウォール
ファイヤーウォールとは、自社のサーバーを不正アクセスやサイバー攻撃などから守るために使われるセキュリティ機能です。  

- 発信元IPや宛先IPに応じた通信の許可・遮断
- 通信プロトコルに応じた通信の許可・遮断
- 監視・管理

などが基本的な役割となって来ます。

### (注26) SELinux
SELinux(Security-Enhanced Linux)とは、システムにアクセス可能なユーザーをより詳細に制御できるようにする、Linuxシステム用のセキュリティ・アーキテクチャです。  
SELinux には3つのモードがあります。

- Disabled  
SELinux が適用されておらず、リソースへと自由にアクセスできる

- Permissive  
SELinux は適用されているがアクセス制御は実行されておらず、ポリシに違反するアクセスは監査ログに記録される

- Enforcing  
SELinux が適用され，アクセス制御が有効になっている状態で、ポリシに違反するアクセスは全て遮断される

今回はEnforcing になっていたため、アクセスが遮断され、Forbidden 403がブラウザ上に出ていました。

### (注27) chmod 
chmodコマンドは、操作権限を変更するためのコマンドです。  
例えばls -laコマンドを打ち、下記のように表示されたとします。

```
-rw-r--r--  1 user group      9  1月 1 00:00 hoge.txt
```

最初の1文字目は、ファイルかディレクトリを示しています。

| 種別 | 意味 |
| ----|---- |
| - | ファイル |
| d | ディレクトリ |  

<br>

2文字目から4文字目は、ファイルの所有者に対する権限  
5文字目から7文字目はファイルの所有グループに対する権限  
8文字目から10文字目はその他に対する権限

をそれぞれ表しています。

また、

```
r ＝ 読み取り / w ＝ 書き込み / x ＝ 実行
```

を意味しています。

つまり上の例だと、
- 「ファイル種別」が「ディレクトリ」
- 「所有者」に「読み取り」と「書き込み」と「実行」の権限がある
- 「所有グループ」に「読み取り」の権限がある
- 「その他」に「読み取り」の権限がある

ことを示しています。

### (注28) -R 777 
-R とオプションをつけることで、指定したディレクトリ以下のディレクトリ・ファイル全ての権限を一括で変更できます。

777 は、操作権限を8進数の数字で表現しています。   
0は権限がない状態、1は権限がある状態を意味します。

```
8進数777 
↓
2進数111111111
```

となり、ユーザー・グループ・その他の全ての権限が付与されたことになります。

***

## 9-1 環境構築の所感
環境構築を通して、学んだことや感じたことは、下記でございます。

①バージョンを確認することの重要性  
恥ずかしながら、プログラミングスクールや過去のカリキュラムにおいて、記載されているバージョンを何も考えず使っていました。  
ただ今回、使用するバージョンが最初に指定されていたため、

- 正しいバージョンがインストールされたかの確認
- 記事を読む際に、どのバージョンが使われているかの確認

が習慣として身についたと感じております。  

②他の人が全く同じ環境を構築できるようにするための「手順書作成」の難しさ  

カリキュラム内の指示において、

- 他の人が手順書を見て、全く同じ環境を構築できること
- 全体の流れが把握できること

とあり、手順書の作成中、常に意識していました。

だからこそ、「この表現で理解いただけるのか」や、「そもそもこの手順で本当にあっているのか」と考えることが多くありました。

そのために、
- 手順書内の可読性を上げるために、説明は注略として下にまとめておく
- 手順書を少し書く→自分でコードを動かす→手順書の追記or修正と、チェックしながら進める
- できる限り細かく調べる

ような工夫を行いました。

長文となりましたが、ご確認いただきまして、誠にありがとうございました。

***

<br>

## 10-1 参考サイト
- [Qiita Markdown 書き方 まとめ](https://qiita.com/shizuma/items/8616bbe3ebe8ab0b6ca1)

- [Markdown記法 チートシート](https://qiita.com/Qiita/items/c686397e4a0f4f11683d)

- [Vagrantで仮想環境構築入門](https://webbibouroku.com/Blog/Article/vagrant#:~:text=Vagrant%20Box%E3%81%A8%E3%81%AF%E3%80%81%E4%BB%AE%E6%83%B3,%E3%81%A7%E3%82%88%E3%81%84%E3%81%A8%E6%80%9D%E3%81%84%E3%81%BE%E3%81%99%E3%80%82)  

- [ポートフォワードにより仮想マシン内のサーバにアクセスする](https://maku77.github.io/vagrant/port-forward.html)

- [ホストオンリーアダプター](https://qiita.com/centipede/items/64e8f7360d2086f4764f)

- [用語集|プライベートネットワーク](https://www.idcf.jp/words/private-network.html)

- [Vagrant の Box の Guest Additions を最新化する方法](https://weblabo.oscasierra.net/vagrant-vbguest-plugin-1/)

- [saharaでVagrantの状態管理](https://qiita.com/kidach1/items/ba365905b2a770c72be1)

- [Vagrantで共有フォルダのエラーがでるのでその対応](https://yk5656.hatenablog.com/entry/20201202/1609916685)

- [sudoコマンドについて](https://www.atmarkit.co.jp/ait/articles/1611/28/news036.html)

- [【yum入門】yumとは何か？](https://uxmilk.jp/9715)

- [yum groupinstall "Development tools" で入るパッケージ一覧](https://qiita.com/old_/items/6f9da09b9af795c11b71)

- [あらためてEPELリポジトリの使い方をまとめてみた](https://qiita.com/yamada-hakase/items/fdf9c276b9cae51b3633)

- [CentOS 7 に PHP 7.2 を yum でインストールする手順](https://weblabo.oscasierra.net/centos7-php72-install/)

- [PHPリファレンス MySQL Native Driver](https://www.php.net/manual/ja/mysqlnd.overview.php) 

- [日本語利用に関する設定(mbstring)](https://www.javadrive.jp/php/install/index8.html)

- [php-xml ](http://alphasis.info/2010/11/php-xml/)

- [XMLとは？IT初心者にもわかりやすい基礎知識とHTMLとの違い](https://hnavi.co.jp/knowledge/blog/xml/)

- [FastCGI Process Manager (FPM)](https://www.php.net/manual/ja/install.fpm.php)

- [RPM resource php-common](https://rpmfind.net/linux/rpm2html/search.php?query=php-common)

- [無印とdevelの違いについて](https://www.unknownengineer.net/entry/2016/05/11/120014)

- [【 unzip 】コマンド――ZIPファイルからファイルを取り出す](https://www.atmarkit.co.jp/ait/articles/1607/26/news014.html)

- [Composer を CentOS にインストールする手順](https://weblabo.oscasierra.net/php-composer-centos-install/)

- [公式リファレンス Laravelインストール](https://readouble.com/laravel/6.x/ja/installation.html)

- [prefer-dist と prefer-sourceについて](https://kohkimakimoto.github.io/getcomposer.org_doc_jp/doc/03-cli.html)

- [【 rpm 】コマンド（基礎編）](https://www.atmarkit.co.jp/ait/articles/1609/13/news024.html)

- [grepで特定の文字列を抽出する方法](https://www.sejuku.net/blog/48915)

- [Laravelにおけるマイグレーションの仕組み](https://note.com/kodokuna_dancer/n/n68c8ef4c7af3)

- [公式レファレンス 6.0 認証](https://readouble.com/laravel/6.x/ja/authentication.html)

- [Laravel6.x以降でログイン機能をインストールする方法](https://blog.capilano-fw.com/?p=4576)

- [Nginxとは？](https://www.winserver.ne.jp/column/about_nginx/)

- [weサーバーソフト](https://www.weblio.jp/content/Web%E3%82%B5%E3%83%BC%E3%83%90%E3%83%BC%E3%82%BD%E3%83%95%E3%83%88#:~:text=Web%E3%82%B5%E3%83%BC%E3%83%90%E3%83%BC%E3%82%BD%E3%83%95%E3%83%88%E3%81%A8%E3%81%AF%E3%80%81Web%E3%82%B5%E3%83%BC%E3%83%90%E3%83%BC%E3%82%92%E6%A7%8B%E6%88%90%E3%81%99%E3%82%8B,%E9%80%81%E4%BF%A1%E3%81%99%E3%82%8B%E6%A9%9F%E8%83%BD%E3%82%92%E6%8C%81%E3%81%A4%E3%80%82)

- [CentOSサーバ構築術 文具堂](https://centos.bungu-do.jp/archives/245)

- [nginx連載4回目: nginxの設定、その2 - バーチャルサーバの設定](https://heartbeats.jp/hbblog/2012/04/nginx04.html)

- [ドキュメントルートとは？](https://wp-fan.com/e-words/documentroot-beginner/)

- [nginx連載5回目: nginxの設定、その3 - locationディレクティブ](https://heartbeats.jp/hbblog/2012/04/nginx05.html)

- [ファイアウォール（FW）とは？](https://www.rworks.jp/system/system-column/sys-entry/21277/)

- [SELinux とは](https://www.redhat.com/ja/topics/linux/what-is-selinux)

- [SELinux を使おう．使ってくれ．](https://qiita.com/chi9rin/items/af532d0dd9237cc65741)

- [Linuxの権限確認と変更(chmod)](https://qiita.com/shisama/items/5f4c4fa768642aad9e06)