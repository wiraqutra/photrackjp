# はじめに #

開発関連の情報です。

# AWS #
Amazon Web Services(AWS)にてお試し環境を作ってみました。URLは以下です。

http://ec2-175-41-212-49.ap-northeast-1.compute.amazonaws.com/trac/test

注意点を以下に挙げます。
  * 現状では EC2 に Trac-0.12.2 を突っ込んだだけです。S3 でのバックアップはまだしていません。
  * URL は EC2 インスタンスを起動するたびに変わるようです。諸事情のため、1次情報はこちらに記載します。
  * 現状、Free Teir での運用を考えていますが、AWSの場合は料金が従量制です。ですので有料になりそうになったときはインスタンスを停止するかもしれません。

## 環境構築 ##
  1. Free Teir の AMI でインスタンスを作成
    * 8GB という表記はインスタンスで初期の上限?が 8GB の模様
      * インスタンス作成時では 10% ほどしか使われていません
      * 8GB 超えたら自動でスケーリングされるのか?
  1. 秘密鍵を作成して ssh でログイン
    * 秘密鍵が必要な方は Google Group で連絡をとってください
    * (蛇足)[こんな情報](http://www.studio-fix.com/blog/kaoru/2009/05/17/post175/)を発見して一瞬不安になったが...
      * 現時点の AMI だとちゃんとしているっぽい
      * sshd\_config 見た感じでも大丈夫
  1. 各種パッケージのインストール
    1. apache 周り
```
sudo yum install httpd
sudo yum install mod_wsgi
```
    1. trac 周り
```
sudo yum install python-babel
```
  1. trac インストール
    1. wget で Trac-0.12.2.tar.gz を取得
    1. tar で解凍
    1. `sudo python setup.py install`
      * babel が入っていれば compile\_catalog もされるはず?
    1. trac-admin で環境を作る
      * ディレクトリは `/var/trac`
      * DB はとりあえず sqlite
        * sqlite はなにもしなくても AMI で使えた
  1. apache 周り
    * mod\_wsgi を入れると `/etc/httpd/conf.d/wsgi.conf` が出来るのでそれに設定を追加(とりあえず Basic認証)
```
LoadModule wsgi_module modules/mod_wsgi.so

WSGIScriptAlias /trac /var/trac/trac.wsgi
<LocationMatch "/trac/[^/]+/login">
  AuthType Basic
  AuthName "Trac"
  AuthUserFile /var/trac/trac.htpasswd
  Require valid-user
</LocationMatch>
```
    * `/var/trac/trac.wsgi`
```
import os

os.environ['TRAC_ENV_PARENT_DIR'] = '/var/trac/'
os.environ['PYTHON_EGG_CACHE'] = '/var/trac/egg-cache'

import trac.web.main
application = trac.web.main.dispatch_request
```
    * `/var/trac/trac.htpasswd`
```
sudo htpasswd -c /var/trac/trac.htpasswd ユーザー名
```
    * apache から trac のファイルが見えるようにする
```
sudo chown apache:apache -R /var/trac
```
    * apache を起動