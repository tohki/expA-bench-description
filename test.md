 ## はじめに

Webシステムを構築する上で，どの程度のアクセス数に耐えられるのか予めWebシステムの性能を知っておくことは重要です．単純な静的なコンテンツから成る静的ホームページであれば，大した負荷がかからないかもしれませんが，アクセス時にDBMSへ問い合わせして返信するコンテンツを動的に生成するような動的ホームページの場合，Webシステムを構成するハードウェアの性能や，使用する各種ソフトウェアの性能によって，同時に処理できるアクセス数の上限に大きな差が生じる場合があります．

本日の実験ではまず，Webシステムへ負荷をかけて定量的な性能評価を行うことのできるツールの使い方を身に着けてください（くれぐれも外部のWebシステムに対し試してみることはしないでくださいね）．

## ab: Apache HTTP server benchmarking tool

デファクトスタンダードのWebサーバといえるApacheは，**ab**（Apache Bench）というベンチマーク用のコマンドも一緒についています．このコマンドは，Webサーバに大量の同時アクセスを行い，1秒あたりのリクエスト処理数といった性能（スループット）を計測することができます．DoS (Denial of Service)攻撃と同様に，宛先に向け一度に大量のリクエストを送信しますので，くれぐれも実行時のオプション指定に誤りがないよう注意して実行してください．リクエストに対する応答が返ってきたらすぐに次のリクエストを発行するという動作を繰り返します．

### 使い方

**ab** \[ **\-A** _auth-username:password_ \] \[ **\-b** _windowsize_ \] \[ **\-c** _concurrency_ \] \[ **\-C** _cookie-name=value_ \] \[ **\-d** \] \[ **\-e** _csv-file_ \] \[ **\-f** _protocol_ \] \[ **\-g** _gnu-plot-file_ \] \[ **\-h** \] \[ **\-H** _custom-header_ \] \[ **\-i** \] \[ **\-k** \] \[ **\-n** _requests_ \] \[ **\-p** _POST-file_ \] \[ **\-P** _proxy-auth-username:password_ \] \[ **\-q** \] \[ **\-r** \] \[ **\-s** \]\[ **\-S** \] \[ **\-t** _timelimit_ \] \[ **\-T** _content-type_ \] \[ **\-v** _verbosity_\] \[ **\-V** \] \[ **\-w** \] \[ **\-x** _\-attributes_ \] \[ **\-X** _proxy\[:port\]_ \] \[ **\-y** _\-attributes_ \]\[ **\-z** _\-attributes_ \] \[ **\-Z** _ciphersuite_ \] \[http\[s\]://\]host-name\[:port\]/path

### コマンド・オプション

<table><tbody><tr><td><b>オプション</b></td><td><b>意味</b></td></tr><tr><td>-n 数値</td><td>テストで発行するリクエストの回数を数値で指定</td></tr><tr><td>-c 数値</td><td>テストで同時に発行するリクエストの数を数値で指定</td></tr><tr><td>-t 数値</td><td>サーバからのレスポンスの待ち時間（秒）を数値で指定</td></tr><tr><td>-p ファイル名</td><td>サーバへ送信するファイルがある場合に指定</td></tr><tr><td>-T コンテンツタイプ</td><td>サーバへ送信するコンテンツヘッダを指定</td></tr><tr><td>-v 数値</td><td>指定した数値に応じた動作情報を表示</td></tr><tr><td>-w</td><td>結果をHTMLで出力</td></tr><tr><td>-x 属性</td><td>HTML出力のtableタグに属性を追加（BORDERなど）</td></tr><tr><td>-y 属性</td><td>HTML出力のtrタグに属性を追加</td></tr><tr><td>-z 属性</td><td>HTML出力のtdまたはthタグに属性を追加</td></tr><tr><td>-C 'Cookie名称=値'</td><td>Cookie値を渡してテストする</td></tr><tr><td>-A ユーザー名:パスワード</td><td>ベーシック認証が必要なコンテンツにテストする</td></tr><tr><td>-P ユーザー名:パスワード</td><td>認証の必要なプロキシを通じてテストする</td></tr><tr><td>-X プロキシサーバ:port</td><td>プロキシ経由でリクエストする場合に指定</td></tr><tr><td>-V</td><td>abのバージョン番号を表示</td></tr><tr><td>-k</td><td>HTTP/1.1のKeepAliveを有効にしてテストする</td></tr><tr><td>-h</td><td>abのヘルプを表示</td></tr></tbody></table>

基本的には，-nと-cオプションを使うことが多いです．例えば，100ユーザが同時に1リクエストを発行する場合は「-n 100 -c 100」，100ユーザが同時に10リクエストを発行する場合，リクエスト総数は100ユーザ×10リクエスト=1000リクエストなので「-n 1000 -c 100」を指定することになります．「-n 10 -c 100」と指定して，100ユーザが合計10リクエスト発行するということはできませんからご注意ください（Total発行リクエスト数が少ないよというエラーメッセージが表示されます）．

-   \-n：リクエストの総数を指定
-   \-c：同時に発行するリクエスト数を指定

ここまでの段階で，皆さんのWebサーバのユーザディレクトリには，**index.php**，**prog00.php**，**prog01.html**という3つのファイルが作成されているはずです．例えば，index.phpに対するスループットを計測するには，以下のように入力してみてください．結果の例もその下に示しておきます（負荷をかけるWebサーバのIPアドレスは適宜変更してください）．

```
 $ ab -n 100 -c 10 http://192.168.1.101/~ユーザ名/index.php
 This is ApacheBench, Version 2.3 <$Revision: 655654 $>
 Copyright 1996 Adam Twiss, Zeus Technology Ltd, http://www.zeustech.net/
 Licensed to The Apache Software Foundation, http://www.apache.org/

 Benchmarking localhost (be patient).....done


 Server Software:        Apache/2.2.22
 Server Hostname:        localhost
 Server Port:            80

 Document Path:          /
 Document Length:        4609 bytes

 Concurrency Level:      10
 Time taken for tests:   0.012 seconds
 Complete requests:      100
 Failed requests:        0
 Write errors:           0
 Non-2xx responses:      106
 Total transferred:      509542 bytes
 HTML transferred:       488554 bytes
 Requests per second:    8444.52 [#/sec] (mean)
 Time per request:       1.184 [ms] (mean)
 Time per request:       0.118 [ms] (mean, across all concurrent requests)
 Transfer rate:          42019.90 [Kbytes/sec] received

 Connection Times (ms)
               min  mean[+/-sd] median   max
 Connect:        0    0   0.1      0       1
 Processing:     0    1   0.2      1       1
 Waiting:        0    0   0.1      0       1
 Total:          1    1   0.2      1       2

 Percentage of the requests served within a certain time (ms)
   50%      1
   66%      1
   75%      1
   80%      1
   90%      1
   95%      2
   98%      2
   99%      2
  100%      2 (longest request)
```

```
* 「Complete requests:      100」が，正常に応答のあったリクエスト数です．
* 「Failed requests:        0」が，正常な応答のなかったリクエスト数です．

```

上記の例では，リクエスト総数100に対して，Complete requestsが100，Failed requestsが0ですから，全てのリクエストに対し応答を正常に得られたことを示しています．

-   「Requests per second: 8444.52 \[#/sec\] (mean)」が，単位時間（一般的には1秒）あたり何回のリクエストを発行できたか（応答があったか）の総数です．

一般的には，この**Requests per second**の値が高くなることが求められます．

-   「Time per request: 1.184 \[ms\] (mean)」が，全リクエストの発行時間（全リクエストの応答時間）
-   「Time per request: 0.118 \[ms\] (mean, across all concurrent requests)」が，1リクエストあたりの平均発行時間（1リクエストの平均応答時間）


\-n と -cで指定する値を徐々に増やして検証していくことで，どの程度のリクエスト数まで耐えられそうか性能評価ができます．具体的には，**Failed requestsが0でなくなった時**が，そのWebサーバの許容負荷の限界値と考えることができます．

ただし，abで発行される全てのリクエストは指定したURLに対するものでしかありません．一般的なWebアクセスは様々なURLへ行われますので，厳密に言うと許容負荷の限界値そのものとは言えませんからご注意ください．様々なURLへ遷移する負荷パターンモデルを様々なユーザ数で設定できるような商用ベンチマークツールもあります．

皆さんのWebサーバのユーザディレクトリ**public\_html**にある**prog00.php**と**prog01.html**に対して，abで負荷をかけ**Requests per second**の値を比較し，その結果が何を意味しているか考察してください．

-   1) **prog00.php**と**prog01.html**の結果に，明確な差が出ましたか？それはどうしてでしょうか？キャプチャしたトラフィックを分析しながら説明してください．
-   2) 今回の結果を踏まえて，どのような時にどのように実装するのがよさそうか自由に考察してください（ハードウェア，ソフトウェア，アルゴリズムとデータ構造の側面からなど）．
-   3) 【発展】興味ある人は考察した内容を実際に試してみて，定量的かつ論理的に考察を深めてみてください．

ここで，abはWebサーバ上で実行するのではなく，皆さんのノートPC上のUbuntuからabコマンドで負荷試験を実行することに気を付けてください（皆さんのノートPCからネットワーク越しにRaspberry Piへ．なぜか分かりますよね？負荷試験ツールの実行にも負荷がかかりますから，同じハードウェア上で動作させたら正確な性能を計測できませんよね）．また，abコマンドがないよとエラーの表示される方は，以下のコマンドなどでabコマンドをインストールしてください．

```sh
 $ sudo apt install apache2-utils

```

各ページへ負荷をかける際，負荷試験の条件をそろえるために同時接続数と総リクエスト数は，**\-n 25 -c 5**といったように同じにしてください．また目的にもよりますが，ここでの各負荷試験では，**時間を空けずに連続して何度も実行しない**ようにしましょう．その理由は今回の負荷試験で使用するサンプルページは，内部的にFlickrに対してHTTPリクエストを発行しています．つまり，-cを高い値にしてしまうと，大量のリクエストが静大（やあなたの自宅）からFlickrへ発行されることになりますので，FlickrにしてみればDoS攻撃だと捉えられるかもしれません（学内情報基盤センタや，あなたの自宅が契約しているISPからDoS攻撃の可能性ということで調査依頼メールが来るかもしれませんのでご注意ください）．またCPUによっては，高負荷になってCPU温度が高くなると動作周波数を落として性能を抑えるものもあります．何度も連続で負荷試験を行うと計測値が不安定になっていくことがありますのでご注意ください．

さらに，これも重要ですが，実験用に構築したRaspberry Pi上のWebサーバへ負荷試験を行う際は，ノートPC上のベンチマークソフトからのアクセス以外の意図せぬネットワークアクセス（たとえば，ブラウザによる通信など）が発生する可能性を可能な限り少なくして行いましょう．またバックグラウンドで動作する不要なサービスはできるだけ起動しないように設定しておくことも重要です．意図していたよりも多くの負荷がかかってしまうと，正しい性能評価ができません．また今後実際にこのような負荷試験を行うことがあった場合は，くれぐれもサービス提供中の本番サーバで実施することはやめましょう．

## おわりに

abを用いて，Webシステムへ負荷をかけて定量的な性能評価を行う基本的な方法は分かりましたか？

また，一般的な使い方におけるPHPとjavascriptの特徴も理解できましたか？