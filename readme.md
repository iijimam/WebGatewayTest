# ApacheコンテナのHTTPS化＋IRIS for Healthへの接続サンプル

- ※1 サンプルではIRIS2022.2を利用しています。[iris](/iris/)以下に iris.keyを配置すればコンテナビルド時にキーが有効化されます。
- ※2 コミュニティエディション利用時は、[docker-compose.yml](docker-compose.yml) 19行目をコメント化（#）し、IRISの[Dockerfile](/iris/Dockerfile)のFROMのコンテナを適切なものに変更してください。

## コンテナの開始

```
docker-compose up -d --build
```
Apacheコンテナ内の80ポートはホストの**8080**に、443は**8443**を使用するように記述しています。

サンプルの [docker-compose.yml](/docker-compose.yml) を修正せずにそのままコンテナを開始した場合は、管理ポータルへは以下のURLでアクセスできます。

通信方式|URL
--|--
http|http://localhost:8080/csp/sys/UtilHome.csp
https|https://localhost:8443/csp/sys/UtilHome.csp

ユーザ名：SuperUser、パスワード：SYS　でアクセスできます。

## コンテナ停止

```
docker-compose stop
```

## コンテナ削除
```
docker-compose down
```
___

**【Apacheのコンテナについて補足】**

サンプルでは、下記コンテナを利用。コンテナ実行で80ポートのApacheが立ち上がる。

`containers.intersystems.com/intersystems/webgateway:2022.2.0.372.0`


IRISの管理ポータル用にこのApacheを利用するためには、IRISへ接続を行うためのパスの追加が必要。

## 1) Apacheコンテナのconfファイルを変更する

Apacheコンテナ起動時、以下にIRIS用のconfファイルが存在する。

`/etc/apache2/mods-available/CSP.conf`

デフォルトの状態ではIRISの管理ポータル用設定が行われていないため、/ のパスが来たときApacheからWebゲートウェイに転送されるように設定を追加する。

追加内容は[webgateway.conf](/web/webgwfiles/webgateway.conf)の中身

Apache用[Dockerfile](/web/Dockerfile) のRUNの行でCSP.confに[webgateway.conf](/web/webgwfiles/webgateway.conf)の中身を追記している（5行目）。

## 2) WebゲートウェイからIRISへ接続するための設定追加

（メモ：Apacheコンテナ立ち上げ後に手動でWebゲートウェイ管理画面でも設定できる　👉　[http://webサーバ名/csp/bin/Systems/Module.cxw](http://localhost:8080/csp/bin/Systems/Module.cxw)）

> このコンテナは80ポートをホストの**8080**に割り当てています。

CSP.iniファイルの中身を書き換えればよいので、書き換えの中身のみを [CSP.ini](/web/webgwfiles/CSP.ini)に用意

コンテナではCSP.iniファイルはWebゲートウェイのインストールディレクトリ以下に配置される。

ディレクトリは環境変数 **${ISC_PACKAGE_INSTALLDIR}** で確認できる。

```
root@318a6f20db77:/opt/webgateway# ls ${ISC_PACKAGE_INSTALLDIR}/bin/CSP.ini
/opt/webgateway/bin/CSP.ini
```
コンテナビルド時、IRISへの接続設定を **${ISC_PACKAGE_INSTALLDIR}/bin/CSP.ini** に追記するように Apache用[Dockerfile](/web/Dockerfile) の6行目のRUNコマンドで実行（実行内容は以下）
```
cat /etc/apache2/mods-available/webgwfiles/CSP.ini >> ${ISC_PACKAGE_INSTALLDIR}/bin/CSP.ini && \
```

ここまでで、80（ホスト側は8080）でIRISへ接続を行う設定は完了。

＜メモ1＞
サンプルでは、接続先もコンテナのIRISを指している。

- 接続先IRISのホスト名：**iris4h**
- スーパーサーバーポート：**1972**
- アクセスユーザ名：**CSPSystem**
- パスワード：**SYS**
- CSP.iniに記載するIRISへの接続名は **[LOCAL]** を利用。サーバ名として設定した[LOCAL]に対して、**LOCAL=Enabled** の記述も必要
- **接続先IRISのCSPSystemのパスワードがサンプルと異なる場合は、 [CSP.ini](/web/webgwfiles/CSP.ini) 16行目の変更が必要。**

＜メモ2＞
IRISコンテナ側では、コンテナビルド時事前定義ユーザに対するパスワードをSYSに固定としています。適宜変更お願いします。
>IRISコンテナビルド時に実行される[Dockerfile](/iris/Dockerfile) で実行している [iris.script](/iris/iris.script)内で、初回アクセス時の事前定義ユーザデフォルトパスワードの変更手続きを無効にしています。

___
以下、HTTPSで管理ポータルに接続したい場合の設定


## 3) HTTPS通信用に証明書ファイルを準備

サンプルではオレオレ証明書ファイルを準備
- [server.crt](/web/webgwfiles/server.crt)
- [server.key](/web/webgwfiles/server.key)

key はパスフレーズが聞かれないように変更したほうが良い。
> openssl rsa -in キーファイル -out キーファイル

## 4) ApacheのSSL化

[3) HTTPS通信用に証明書ファイルを準備](#3-https通信用に証明書ファイルを準備) のファイルを、default-ssl.conf で指定されているディレクトリにコピーしています。

default-ssl.confはApacheコンテナ起動時、以下にあります。
> /etc/apache2/sites-available/default-ssl.conf

それぞれのファイルを以下に配置
ファイル名|配置場所
--|--|
server.crt|/etc/ssl/certs/server.crt
server.key|/etc/ssl/private/server.key

default-ssl.confを書き換えるため、オリジナルをコピーし、以下の項目名に用意した証明書ファイルのフルパスを設定


項目名|ファイル指定
--|--|
SSLCertificateFile|/etc/ssl/certs/server.crt
SSLCertificateKeyFile|/etc/ssl/private/server.key

これらの文字列の加工は、Apacheの[Dockerfile](/web/Dockerfile) のRUNコマンド（7行目以降）で実行しています。

default-ssl.confの書き換えが終了したら、Apacheの[Dockerfile](/web/Dockerfile) のRUNコマンド（13～14行目）でSSL設定ファイルの有効化を行うため、以下実行しています。
```
a2ensite default-ssl
a2enmod ssl
```

コンテナビルド時に上記設定が完了するため、あとはコンテナを開始するだけでhttpsでIRISの管理ポータルにアクセスできます。