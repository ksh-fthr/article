# 環境

|                    | バージョン        | 備考                                                         |
| ------------------ | ----------------- | ------------------------------------------------------------ |
| macOS              | ~~Catalina 10.15.7~~       |                                                                  |
|                    | Monterey 12.6              |                                                              |
| JMeter             | ~~5.4~~               | ~~2020/12/07 現在の [ダウンロードサイト](https://jmeter.apache.org/download_jmeter.cgi) から~~ <br /> 2022/11/30 現在､ [アーカイブ](https://archive.apache.org/dist/jmeter/binaries/)済みなのでそちらからダウンロード<br />  |
|                    | 5.5                   | 2022/11/30 現在の [ダウンロードサイト](https://jmeter.apache.org/download_jmeter.cgi) からダウンロード |
| Java( `openjdk`  ) | ~~11.0.9 2020-10-20~~ | ~~`java --version` で確認~~                                     |
|      | 19.0.1 2022-10-18     | `java --version` で確認                                      |

( 本記事は JMeter `v5.4` で撮ったキャプチャを貼付しております )

# 前準備
まず Java がインストールされている必要がある｡
本記事では次の手順で Java をインストールしていく｡


```bash
# brew 経由で Java のインストール
$ brew install java
...
...
...
For the system Java wrappers to find this JDK, symlink it with
  sudo ln -sfn /usr/local/opt/openjdk/libexec/openjdk.jdk /Library/Java/JavaVirtualMachines/openjdk.jdk

openjdk is keg-only, which means it was not symlinked into /usr/local,
because macOS provides similar software and installing this software in
parallel can cause all kinds of trouble.

If you need to have openjdk first in your PATH, run:
  echo 'export PATH="/usr/local/opt/openjdk/bin:$PATH"' >> ~/.zshrc

For compilers to find openjdk you may need to set:
  export CPPFLAGS="-I/usr/local/opt/openjdk/include"
```

```bash
# インストール時のログに従いシンボリックリンクを貼る
$ sudo ln -sfn /usr/local/opt/openjdk/libexec/openjdk.jdk /Library/Java/JavaVirtualMachines/openjdk.jdk
```


```bash
# jdk に path を通す
$ echo 'export PATH="/usr/local/opt/openjdk/bin:$PATH"' >> ~/.zshrc
```

```bash
# 一旦 shell の再起動
$ exec $SHELL -l

# $JAVA_HOME に path を通す
$ echo 'export JAVA_HOME=`/usr/libexec/java_home -v`' >> ~/.zshrc
```

```bash
# バージョン確認
$ java --version
openjdk 19.0.1 2022-10-18
OpenJDK Runtime Environment Homebrew (build 19.0.1)
OpenJDK 64-Bit Server VM Homebrew (build 19.0.1, mixed mode, sharing)
```

# JMeter とは

[公式](https://jmeter.apache.org/) から抜粋

> The **Apache JMeter™** application is open source software,      a 100% pure Java application designed      to load test functional behavior and measure performance.  It was      originally designed for testing Web Applications but has      since expanded to other test functions.    

訳

> **Apache JMeter™ **アプリケーションはオープンソースのソフトウェアで、機能動作をロードテストし、パフォーマンスを測定するために設計された100%純粋なJavaアプリケーションです。 元々はWebアプリケーションのテスト用に設計されていましたが、他のテスト機能にも拡張されています。   

上記の通りパフォーマンス計測のための Java アプリケーションである。



## ダウンロード

JMeter のダウンロードは下記リンクから行える。

- https://jmeter.apache.org/download_jmeter.cgi
- 今回は Binaries をダウンロードした( `*.tgz`, `*.zip` はどちらでも OK )

![0000_download.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/193342/0d7d9a4a-dd59-daa9-08df-ab9ea189790e.png)




## インストール

インストールはダウンロードしたバイナリを解凍し、任意のディレクトリに配置するだけ。
今回は次のように配置した。

```bash:JMeterを任意のディレクトリに配置する
$ pwd
/Users/hoge/workspace                           # workspace は作業用ディレクトリとして元々作成済み
$ mkdir performance                             # 負荷試験用のディレクトリを掘って
$ mv ~/Download/apache-jmeter-5.4 ./performance # どこにバイナリを配置する
$ cd performance/
$ ls -l
total 0
drwxr-xr-x@ 12 hoge  staff  384 12  6 20:38 apache-jmeter-5.4
```



## 起動

起動は次のコマンドを実行する。

```bash:JMeterをGUIで起動する
$ cd apache-jmeter-5.4/
$ sh ./bin/jmeter       # これで JMeter の GUI が起動する
```



### 注意事項

起動にあたり以下が設定されていることが前提条件となる。そうでないと起動エラーとなる。

- jdk の PATH が通っていること
- `JAVA_HOME` の環境変数が設定されていること



### 起動エラーの例

```bash:起動エラー時のログ
$ sh ./bin/jmeter
Unable to find any JVMs matching version "(null)".
No Java runtime present, try --request to install.
Neither the JAVA_HOME nor the JRE_HOME environment variable is defined
At least one of these environment variable is needed to run this program
```



# 負荷試験の実施

## 本記事で扱う負荷試験の要件

以下の要件に対して負荷試験を実施する。

- Web Server へのアクセス負荷を検証する
  - プロトコルは `https`
  - サーバ名は `hogehoge.com`
  - HTTP メソッドは `GET`
  - アクセス先 PATH は `test-hoge/hoge.m3u8`
- リファラが設定されている
  - リファラは `https://cdn.test-hoge.com/hoge.html`
- 負荷
  - アクセス数 / 秒 がどこまで正常に処理できるか



## テスト計画の作成と設定

というわけで、上記の要件に沿うように JMeter で行う負荷試験のシナリオを設定していく。

JMeter は起動時、`Test Plan` という項目が左側ペインにポツンとあるだけなので、これを元にテスト計画を作っていく。



#### 1. 初期起動時の状態

![0001_boot.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/193342/ff8f0ee4-8056-4bba-a67a-7861b161e350.png)
起動時はご覧の通りなにも設定されていない。`TestPlan` という項目があるだけ。
これに対してテスト要件にあわせて設定を追加していく。


#### 2. Thread Group の追加と設定

![0201_add_thread_group.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/193342/df9e6609-4dc1-b0ec-897d-4b873ff5207e.png)
`TestPlan` を選択した状態で右クリック ( mac なら 2 本指でクリック ) するとメニューが表示されるので、`Add -> Threads(Users) -> Thread Group)` の順で Thread Group を追加する。

![0202_thread_group_settings.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/193342/ce0af9bf-caa7-42cd-c7cf-c56d57a54d59.png)
今回実施する負荷試験で重要な設定内容となるのが下記表の 3つ。
( Name は任意。デフォルトのままでも良い )

| 項目                      | 説明                       | 備考                                                         |
| ------------------------- | -------------------------- | ------------------------------------------------------------ |
| Number of Threads (users) | スレッド数の設定           | 各スレッドは **他のテストスレッドとは完全に独立** してテスト計画を実行する.<br />複数のスレッドを使用してサーバーアプリケーションへの **同時接続をシミュレート** する. |
| Ramp-up period (seconds)  | ランプアップ期間の設定     | 選択したスレッドの数だけ「ランプアップ」するのにかかる時間を JMeter に通知する.<br /><br />例)<br />10個のスレッドを使用し、ランプアップ期間を100秒とした場合、10個のスレッドが全て立ち上がるまでに100秒かかる.<br />各スレッドは、前のスレッドが開始されてから10（100/10）秒後に開始される.<br />スレッドが30個あり、ランプアップ期間が120秒の場合、各スレッドは4秒遅れて開始される.<br/><br/>ランプアップはテスト開始時の作業負荷が大きくなりすぎないように十分な長さが必要.<br /><br/>**Ramp-up = スレッドの数** でスタートし、**必要に応じて上下に調整** する. |
| Loop Count                | テストの実行回数を設定する |                                                              |

キャプチャした画像を例にとると、このケースでは「**1つのスレッドを1秒間に処理する。各スレッドはシナリオを一回だけ実行する**」、つまり「**1秒間に一回のアクセスがサーバに対して行われる**」というシナリオになる。

秒あたりのアクセス負荷を上げたい場合、これら **3つの項目を調整** していくのだが、ここで注意点が一つ。

「秒あたりのアクセス負荷を上げるんだったら、スレッド数とランプアップ期間で調整すれば実行回数まで考えなくても良いよね」といった感じで設定すると、テストを実行する PC の負荷が上がってテストが失敗することがある。



**[参考までに]**

本稿での設定とは別に次の設定をして試験を実施したところ失敗した。

| Number of Threads (users) | Ramp-up period (seconds) | Loop Count | Reesult        | 試験要件             |
| ------------------------- | ------------------------ | ---------- | -------------- | -------------------- |
| 4,000                     | 1                        | 1          | 動かなくなった | 4,000アクセス / 1sec |

次の設定だと成功した。

| Number of Threads (users) | Ramp-up period (seconds) | Loop Count | Reesult | 試験要件             |
| ------------------------- | ------------------------ | ---------- | ------- | -------------------- |
| 400                       | 100                      | 1,000       | OK      | 4,000アクセス / 1sec |

という感じで、何度も繰り返して恐縮だが **3つの項目を調整** してテストを実行する環境の負荷にも目を向けつつ、テストされる側の負荷を上げていく。



#### 3. HTTP Request の追加と設定

[公式 >3.1 Thread Group](https://jmeter.apache.org/usermanual/test_plan.html#thread_group) には次の記載がある。

> Thread group elements are the beginning points of any test plan. All controllers and samplers must be under a thread group. Other elements, e.g. Listeners, may be placed directly under the test plan, in which case they will apply to all the thread groups. As the name implies, the thread group element controls the number of threads JMeter will use to execute your test.  

訳

> スレッドグループの要素は、テスト計画の最初のポイントです。すべてのコントローラとサンプラーはスレッドグループの下になければなりません。リスナーなどの他の要素は、テスト計画の直下に配置することもできますが、その場合はすべてのスレッドグループに適用されます。その名の通り、スレッドグループ要素は、JMeter がテストを実行するために使用するスレッドの数を制御します。

ということで、以降に出てくる設定は全て Thread Group 配下、もしくは更に下に要素を追加して行っていく。

ここでは試験対象となるアクセス先の設定を行う。

![0301_add_http_request.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/193342/12674770-01fb-79ca-2f69-2620b8a00fda.png)
`Thread Group` を選択した状態で右クリック ( mac なら 2 本指でクリック ) するとメニューが表示されるので、`Add -> Sampler -> HTTP Request` の順で HTTP Request を追加する。

![0302_http_request_settings.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/193342/a9bf7089-783a-363e-e634-24c0a98c19ce.png)
次の設定項目を実際の環境に合わせて設定する。( Name は任意。デフォルトのままでも良い )

| 項目               | 説明                                                 | 備考                                   |
| ------------------ | ---------------------------------------------------- | -------------------------------------- |
| Protocol           | アクセスする際のプロトコルを設定する                 |                                        |
| Server Namse or IP | アクセス先のサーバ名、もしくは IP アドレスを設定する |                                        |
| メソッド           | アクセス時のメソッドを設定する                       |                                        |
| Path               | アクセス先のパスを設定する                           | Server Name or IP で指定した内容に絡む |

今回の設定例ならば **`https://hogehoge.com/test-hoge/hoge.m3u8` に対して `GET` メソッドでアクセスする** という意味になる。



#### 4. HTTP Header Manager の追加と設定

今回の試験では `リファラが設定されている` ことが要件に挙がっているので、`HTTP Header` にリファラを設定する。

![0401_add_http_header_manager.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/193342/81ec087e-ee65-535e-4e87-e84db563faba.png)
`HTTP Request` を選択した状態で右クリック ( mac なら 2 本指でクリック ) するとメニューが表示されるので、`Add -> Config Element -> HTTP Header Mamnager` の順で HTTP Header Mamnager を追加する。

![0402_http_header_manager_settings.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/193342/3bf3e034-3c53-d073-58bc-b2cedbaa01f4.png)
`Add` ボタンをクリックし、refer 情報を設定する。( Name は任意。デフォルトのままでも良い )

| 項目    | 説明                                                         | 備考 |
| ------- | ------------------------------------------------------------ | ---- |
| referer | 現在リクエストされているページへのリンク先を持った直前のウェブページのアドレス |      |

今回の設定例ならば **`https://hogehoge.com/test-hoge/hoge.m3u8` のリファラとして `https://cdn.test-hoge.com/hoge.html` を設定した** という意味になる。



#### 5. Aggregate Report の追加と設定

上記までが試験実施のための設定。
ここでは試験実施後の結果をみるために `Aggregate Report` を追加する。

![0501_add_aggregate_report.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/193342/71b64be3-8a7d-ef73-9a80-05ab66cd8626.png)
`Thread Group` を選択した状態で右クリック ( mac なら 2 本指でクリック ) するとメニューが表示されるので、`Add -> Listener -> Aggregate Report` の順で Aggregate Report を追加する。

![0502_aggregate_report_settings.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/193342/23d4989c-bef1-e444-ab82-7176aa862a9b.png)
Filename にレポート保存時のファイル名を設定する。( Name は任意。デフォルトのままでも良い )



## 実行( とクリア )

![0601_command.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/193342/65c15010-dff1-f9f6-069b-a82cffd49b74.png)
試験の実行は **ピンク枠** のどちらかをクリックする。

- 左側: Start
- 右側: Start no pauses

試験結果のクリアは **オレンジ枠** のどちらかをクリックする。

- 左側: clear
- 右側: Clear All



### 実行前のファイル確認

試験実行前に `report.csv` が出力されていないことを確認する。

```bash:実行前のファイル一覧
$ ls -l
total 3785128
-rw-rw-r--@   1 hoge  staff       15631  2  1  1980 LICENSE
-rw-rw-r--@   1 hoge  staff         172  2  1  1980 NOTICE
-rw-r--r--@   1 hoge  staff        5523 12  7 19:47 PerformanceTest.jmx
-rw-rw-r--@   1 hoge  staff       10089  2  1  1980 README.md
drwxr-xr-x   12 hoge  staff         384 12  7 19:47 backups
drwxrwxr-x@  44 hoge  staff        1408 12  6 23:32 bin
drwxr-xr-x@   6 hoge  staff         192  2  1  1980 docs
drwxrwxr-x@  22 hoge  staff         704  2  1  1980 extras
-rw-------    1 hoge  staff  1936927555 12  7 15:33 java_pid57152.hprof
-rw-r--r--    1 hoge  staff      140194 12  7 19:47 jmeter.log
drwxrwxr-x@ 104 hoge  staff        3328  2  1  1980 lib
drwxrwxr-x@ 104 hoge  staff        3328  2  1  1980 licenses
drwxr-xr-x@  19 hoge  staff         608  2  1  1980 printable_docs
```



### 実行後のファイル確認

試験実行前に `report.csv` が出力されていることを確認する。

```bash:実行後のファイル一覧
$ ls -l
total 3785136
-rw-rw-r--@   1 hoge  staff       15631  2  1  1980 LICENSE
-rw-rw-r--@   1 hoge  staff         172  2  1  1980 NOTICE
-rw-r--r--@   1 hoge  staff        5523 12  7 19:54 PerformanceTest.jmx
-rw-rw-r--@   1 hoge  staff       10089  2  1  1980 README.md
drwxr-xr-x   12 hoge  staff         384 12  7 19:47 backups
drwxrwxr-x@  44 hoge  staff        1408 12  6 23:32 bin
drwxr-xr-x@   6 hoge  staff         192  2  1  1980 docs
drwxrwxr-x@  22 hoge  staff         704  2  1  1980 extras
-rw-------    1 hoge  staff  1936927555 12  7 15:33 java_pid57152.hprof
-rw-r--r--    1 hoge  staff      141450 12  7 19:54 jmeter.log
drwxrwxr-x@ 104 hoge  staff        3328  2  1  1980 lib
drwxrwxr-x@ 104 hoge  staff        3328  2  1  1980 licenses
drwxr-xr-x@  19 hoge  staff         608  2  1  1980 printable_docs
-rw-r--r--    1 hoge  staff         311 12  7 19:54 report.csv
```



### 出力された csv ファイル( `report.csv` )

試験結果は `report.csv` で出力するよう設定したので、テキストファイルからも確認できる。

```csv:report.csv
timeStamp,elapsed,label,responseCode,responseMessage,threadName,dataType,success,failureMessage,bytes,sentBytes,grpThreads,allThreads,URL,Latency,IdleTime,Connect
1607303084226,47,HTTP Request,200,OK,Thread Group 1-1,text,true,,584,215,1,1,https://hogehoge.com/test-hoge/hoge.m3u8,47,0,38
```



### JMeter 上でのレポートの確認

`Aggregate Report `から試験結果の統計情報を見ることができる。
![0701_aggregate_report.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/193342/7caaa652-3f41-c89c-7fd3-b4b5c6632b75.png)
それぞれの値の意味は次の通り。

| カラム          | 意味                               | 備考                                                         |
| --------------- | ---------------------------------- | ------------------------------------------------------------ |
| Label           | 試験を実施したテストケースのラベル | `HTTP Request` のみ. <br />[3. HTTP REQUEST の追加と設定](#3-http-request-の追加と設定) で設定した `Name` がこれ |
| Sample          | 試験の実行回数                     | Number of Threads と Ramp-up period と Loop Count の値による |
| Average         | 実行結果の平均値                   |                                                              |
| Median          | 実行結果における中央値             | テストの50%がこの時間以上かかっていないことを示す            |
| 90% Line        | 実行結果の 90% の集合              | 90%のサンプルはこの時間以上かかっていないことを示す.<br />10% はこれと同じくらいの時間がかかった |
| 95% Line        | 実行結果の 95% の集合              | 95%のサンプルはこの時間以上かかっていないことを示す.<br />5% はこれと同じくらいの時間がかかった |
| 99% Line        | 実行結果の 99% の集合              | 99%のサンプルはこの時間以上かかっていないことを示す.<br />1% はこれと同じくらいの時間がかかった |
| Min             | 実行完了までの最短時間             |                                                              |
| Maximum         | 実行完了までの最長時間             |                                                              |
| Error %         | エラー発生率                       |                                                              |
| Throughput      | スループット                       | 時間単位(秒、分、時間)で処理したリクエストの数.<br />最初のサンプルの開始から最後のサンプルの終了までの時間で計算される.<br />**スループットが大きいほど良い.** |
| Received KB/Sec | 受信量                             | 実行中にサーバーからダウンロードされたデータ量.<br />-> **キロバイト/秒で測定したスループット**. |
| Sent KB/Sec     | 送信量                             | 実行中にサーバーに送ったデータ量.<br />-> **キロバイト/秒で測定したスループット**. |

なお、テーブルに表示されている結果は画面下部の `Save Table Data` から csv 出力することもできる。
下記キャプチャは `Save Table Data` クリックで表示されたダイアログ。デフォルトで `aggregate.csv` というファイル名で保存するようになっている。
![0702_aggregate_report_save.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/193342/39ddbd3c-cfb6-e58b-6ff6-29c5d2c1f047.png)
出力した csv の中身は以下の通り。

```csv:統計情報
$ cat aggregate.csv
Label,# Samples,Average,Median,90% Line,95% Line,99% Line,Min,Max,Error %,Throughput,Received KB/sec,Sent KB/sec
HTTP Request,1,328,328,328,328,328,328,328,0.000%,3.04878,1.71,0.66
TOTAL,1,328,328,328,328,328,328,328,0.000%,3.04878,1.71,0.66
```



## CUI での実行

テスト計画が作成されていることが前提だが、CUI からも JMeter は実行できる。
実行時のコマンドは以下の通り。

```bash:CUIで試験を実施する
$ ./jmeter -n -t PerformanceTest.jmx # PerformanceTest.jmx はこれまで設定してきたシナリオファイル
```



なお JMeter は GUI から実行すると、それだけで実行マシンのリソースを余計に消費するので、CUI から実行するほうが望ましいとのこと。
[公式 > 1.0.2 Load Test running](https://jmeter.apache.org/usermanual/get-started.html#load_test_running) には次のように書かれている。

> Don't run load test using GUI mode !



というわけで設定ファイルを作って試行できたら、実際の負荷試験は CUI から実行するのが良い。

CUI でのコマンドオプションは下記を参照。

- [公式 > 1.4.4 CLI Mode (Command Line mode was called NON GUI mode)](https://jmeter.apache.org/usermanual/get-started.html#non_gui)



# まとめ

- `Thread Group` では次の 3つの項目を調整して、テストを実行する環境の負荷にも目を向けつつ、テストされる側の負荷を上げていく
  - Number of Threads (users)
  - Ramp-up period (seconds)
  - Loop Count
- スレッドグループの要素はテスト計画の最初のポイントであり、すべてのコントローラとサンプラーはスレッドグループの下に設定する
- テストの実行は GUI ベースではなく **CUI ベース が望ましい**



# 参考

## 公式

- [Apache JMeter™](https://jmeter.apache.org/)

## その他

### インストール〜テスト実施

- [Jmeter のインストールから負荷テストまで](https://qiita.com/shotets/items/d553d7be0d407a9a9a53)
- [負荷ツールのスレッド数・Ramp-Up期間・ループ回数の関係](https://christina04.hatenablog.com/entry/2017/10/03/190000)
- [JMeter の利用方法(1) – Ramp－up、スレッド数、ループ回数の誤用](https://keis-software.com/2013/09/02/jmeter-の利用方法1-ramp－up、スレッド数、ループ回数の誤/)
- [【JMeter】負荷テスト実行はGUIから行ってはならない](https://qiita.com/tatesuke/items/827e6190753964e46814)

### Aggregate Report

- [Understand and Analyze Aggregate Report in Jmeter](http://www.testingjournals.com/understand-aggregate-report-jmeter/)
- [Analyze the Aggregate Report in JMeter](https://jmetervn.com/2016/10/10/analyze-the-aggregate-report-in-jmeter/)

