# はじめに

SQLAlchemy を使って `insert` 処理を実行した際にトランザクションの扱い方で処理時間に結構な差がでました。(※注)
以下、サンプルコードと自身の環境での検証結果ではありますが知見を共有したく思います。

(※注)
本記事のサンプルコードでは `insert` 処理のみを扱いますが、`update` 処理でもトランザクションの扱いによって実行時間に差が出ます。


# 環境

| 環境                                                         | バージョン              | 備考                         |
| ------------------------------------------------------------ | ----------------------- | ---------------------------- |
| macOS Catalina                                               | v10.15.7                |                              |
| [Docker Desktop for Mac](https://hub.docker.com/editions/community/docker-ce-desktop-mac/) | v3.3.3                  |                              |
| Docker                                                       | v20.10.6, build 370c289 | `$ docker --version`         |
| Docker Compose                                               | v1.29.1, build c34c88b2 | `$ docker-compose --version` |
| [Docker MySQL](https://hub.docker.com/_/mysql)               | v8.0.x                  | Dockerfile で指定            |
| [Docker Python](https://hub.docker.com/_/python)             | v3.9.x                  | 同上                         |
| [SQLAlchemy](https://www.sqlalchemy.org/)                    | v1.4.17                 | requirement.txt で指定       |



# 検証

以下の設定でそれぞれ `10,000` 行の `insert` と `update` 処理で検証。

- autocommit ありでループ内で flush()
- autocommit ありでループ外で flush()
- autocommit なしでループ外で commit()



## 結果から

### insert

|      | トランザクション | flush() / commit() の場所 | 結果                              | 備考                              |
| ---- | ---------------- | ------------------------- | --------------------------------- | --------------------------------- |
| 1    | autocommit あり  | ループ内で flush()        | 10,000件 insert するのに**約3分** | 毎行 commit が実行され､下の 2 つに比べて大幅に時間がかかる                                  |
| 2    | autocommit あり  | ループ外で flush()        | 10,000件 insert するのに**約8秒** | commit は全行に対して実行され､下記 `3` と同じクエリが発行された |
| 3    | autocommit なし  | ループ外で commit()       | 10,000件 insert するのに**約8秒** | commit は全行に対して実行され､上記 `2` と同じクエリが発行された |



## コードサンプルとログ

### autocommit ありでループ内で flush()

コードサンプル

```python
# セッションの設定(autocommit あり)
Session = sessionmaker(
    autocommit=True,
    class_=MasterSlaveSession
)
```

```python
# ループ内で flush()
def insert():
    rdb_session = Session()
    for i in range(10000):
        work_num = i + 1
        work_time = jst_time(time.time())
        employee = Employee()
        employee.name = 'employee' + str(work_num).zfill(6)
        employee.phone = str(work_num).zfill(6)
        employee.created_at = work_time
        employee.updated_at = work_time
        rdb_session.add(employee)

        # autocommit ありでここで flush() すると毎行トランザクションが発生するので遅い
        rdb_session.flush()

    # autocommit ありの場合は flush(), なしの場合は commit() するとトランザクションは処理対象行全体にかかるので早い
    # rdb_session.flush()
    # rdb_session.commit()
```



ログ(実行結果)

```bash
##
# 10,000 件処理したときの時間
#  * autocommit あり
#  * loop 内で flush
#
2021-06-08 15:36:16.736761+09:00 process start
process had got 171.57751655578613sec to done.         # 10,000件 insert するのに約3分
2021-06-08 15:39:08.314277+09:00 process had finished

##
# ログ( これは 10件処理したときのログから2件抜粋 )
# 1件ずつトランザクションがかかっている事がわかる
#
2021-06-08 15:46:56,357 INFO sqlalchemy.engine.Engine BEGIN (implicit) ##トランザクション開始
2021-06-08 15:46:56,360 INFO sqlalchemy.engine.Engine INSERT INTO employee (name, phone, created_at, updated_at) VALUES (%(name)s, %(phone)s, %(created_at)s, %(updated_at)s)
2021-06-08 15:46:56,360 INFO sqlalchemy.engine.Engine [generated in 0.00036s] {'name': 'employee000001', 'phone': '000001', 'created_at': datetime.datetime(2021, 6, 8, 15, 46, 56, 341841, tzinfo=datetime.timezone(datetime.timedelta(seconds=32400), 'JST')), 'updated_at': datetime.datetime(2021, 6, 8, 15, 46, 56, 341841, tzinfo=datetime.timezone(datetime.timedelta(seconds=32400), 'JST'))}
2021-06-08 15:46:56,365 INFO sqlalchemy.engine.Engine COMMIT           ##トランザクション終了
2021-06-08 15:46:56,377 INFO sqlalchemy.engine.Engine BEGIN (implicit) ##トランザクション開始
2021-06-08 15:46:56,377 INFO sqlalchemy.engine.Engine INSERT INTO employee (name, phone, created_at, updated_at) VALUES (%(name)s, %(phone)s, %(created_at)s, %(updated_at)s)
2021-06-08 15:46:56,377 INFO sqlalchemy.engine.Engine [cached since 0.01766s ago] {'name': 'employee000002', 'phone': '000002', 'created_at': datetime.datetime(2021, 6, 8, 15, 46, 56, 377034, tzinfo=datetime.timezone(datetime.timedelta(seconds=32400), 'JST')), 'updated_at': datetime.datetime(2021, 6, 8, 15, 46, 56, 377034, tzinfo=datetime.timezone(datetime.timedelta(seconds=32400), 'JST'))}
2021-06-08 15:46:56,379 INFO sqlalchemy.engine.Engine COMMIT           ##トランザクション終了
```

いちレコード単位でトランザクションが発生されているのがわかる。遅い。


### autocommit ありでループ外で flush()

コードサンプル

```python
# セッションの設定(autocommit あり)
Session = sessionmaker(
    autocommit=True,
    class_=MasterSlaveSession
)
```

```python
# ループ外で flush()
def insert():
    rdb_session = Session()
    for i in range(10000):
        work_num = i + 1
        work_time = jst_time(time.time())
        employee = Employee()
        employee.name = 'employee' + str(work_num).zfill(6)
        employee.phone = str(work_num).zfill(6)
        employee.created_at = work_time
        employee.updated_at = work_time
        rdb_session.add(employee)

        # autocommit ありでここで flush() すると毎行トランザクションが発生するので遅い
        # rdb_session.flush()

    # autocommit ありの場合は flush(), なしの場合は commit() するとトランザクションは処理対象行全体にかかるので早い
    rdb_session.flush()
    # rdb_session.commit()
```



ログ(実行結果)

```bash
##
# 10,000 件処理したときの時間
#  * autocommit あり
#  * loop 外で flush
#
% python sqlalchemy_work.py
2021-06-08 15:34:45.603209+09:00 process start
process had got 8.878259658813477sec to done.         # 10,000件 insert するのに約8秒
2021-06-08 15:34:54.481468+09:00 process had finished

##
# ログ( これは 10件処理したときのログ )
# 10件に対してまとめてトランザクションがかかっている事がわかる
2021-06-08 15:49:04,296 INFO sqlalchemy.engine.Engine BEGIN (implicit) ##トランザクション開始
2021-06-08 15:49:04,298 INFO sqlalchemy.engine.Engine INSERT INTO employee (name, phone, created_at, updated_at) VALUES (%(name)s, %(phone)s, %(created_at)s, %(updated_at)s)
2021-06-08 15:49:04,298 INFO sqlalchemy.engine.Engine [generated in 0.00040s] {'name': 'employee000001', 'phone': '000001', 'created_at': datetime.datetime(2021, 6, 8, 15, 49, 4, 277344, tzinfo=datetime.timezone(datetime.timedelta(seconds=32400), 'JST')), 'updated_at': datetime.datetime(2021, 6, 8, 15, 49, 4, 277344, tzinfo=datetime.timezone(datetime.timedelta(seconds=32400), 'JST'))}
2021-06-08 15:49:04,302 INFO sqlalchemy.engine.Engine INSERT INTO employee (name, phone, created_at, updated_at) VALUES (%(name)s, %(phone)s, %(created_at)s, %(updated_at)s)
2021-06-08 15:49:04,302 INFO sqlalchemy.engine.Engine [cached since 0.004541s ago] {'name': 'employee000002', 'phone': '000002', 'created_at': datetime.datetime(2021, 6, 8, 15, 49, 4, 279356, tzinfo=datetime.timezone(datetime.timedelta(seconds=32400), 'JST')), 'updated_at': datetime.datetime(2021, 6, 8, 15, 49, 4, 279356, tzinfo=datetime.timezone(datetime.timedelta(seconds=32400), 'JST'))}
2021-06-08 15:49:04,305 INFO sqlalchemy.engine.Engine INSERT INTO employee (name, phone, created_at, updated_at) VALUES (%(name)s, %(phone)s, %(created_at)s, %(updated_at)s)
2021-06-08 15:49:04,306 INFO sqlalchemy.engine.Engine [cached since 0.00756s ago] {'name': 'employee000003', 'phone': '000003', 'created_at': datetime.datetime(2021, 6, 8, 15, 49, 4, 279418, tzinfo=datetime.timezone(datetime.timedelta(seconds=32400), 'JST')), 'updated_at': datetime.datetime(2021, 6, 8, 15, 49, 4, 279418, tzinfo=datetime.timezone(datetime.timedelta(seconds=32400), 'JST'))}
2021-06-08 15:49:04,309 INFO sqlalchemy.engine.Engine INSERT INTO employee (name, phone, created_at, updated_at) VALUES (%(name)s, %(phone)s, %(created_at)s, %(updated_at)s)
2021-06-08 15:49:04,309 INFO sqlalchemy.engine.Engine [cached since 0.01123s ago] {'name': 'employee000004', 'phone': '000004', 'created_at': datetime.datetime(2021, 6, 8, 15, 49, 4, 279465, tzinfo=datetime.timezone(datetime.timedelta(seconds=32400), 'JST')), 'updated_at': datetime.datetime(2021, 6, 8, 15, 49, 4, 279465, tzinfo=datetime.timezone(datetime.timedelta(seconds=32400), 'JST'))}
2021-06-08 15:49:04,312 INFO sqlalchemy.engine.Engine INSERT INTO employee (name, phone, created_at, updated_at) VALUES (%(name)s, %(phone)s, %(created_at)s, %(updated_at)s)
2021-06-08 15:49:04,312 INFO sqlalchemy.engine.Engine [cached since 0.01392s ago] {'name': 'employee000005', 'phone': '000005', 'created_at': datetime.datetime(2021, 6, 8, 15, 49, 4, 279509, tzinfo=datetime.timezone(datetime.timedelta(seconds=32400), 'JST')), 'updated_at': datetime.datetime(2021, 6, 8, 15, 49, 4, 279509, tzinfo=datetime.timezone(datetime.timedelta(seconds=32400), 'JST'))}
2021-06-08 15:49:04,314 INFO sqlalchemy.engine.Engine INSERT INTO employee (name, phone, created_at, updated_at) VALUES (%(name)s, %(phone)s, %(created_at)s, %(updated_at)s)
2021-06-08 15:49:04,314 INFO sqlalchemy.engine.Engine [cached since 0.01604s ago] {'name': 'employee000006', 'phone': '000006', 'created_at': datetime.datetime(2021, 6, 8, 15, 49, 4, 279603, tzinfo=datetime.timezone(datetime.timedelta(seconds=32400), 'JST')), 'updated_at': datetime.datetime(2021, 6, 8, 15, 49, 4, 279603, tzinfo=datetime.timezone(datetime.timedelta(seconds=32400), 'JST'))}
2021-06-08 15:49:04,315 INFO sqlalchemy.engine.Engine INSERT INTO employee (name, phone, created_at, updated_at) VALUES (%(name)s, %(phone)s, %(created_at)s, %(updated_at)s)
2021-06-08 15:49:04,316 INFO sqlalchemy.engine.Engine [cached since 0.01768s ago] {'name': 'employee000007', 'phone': '000007', 'created_at': datetime.datetime(2021, 6, 8, 15, 49, 4, 279643, tzinfo=datetime.timezone(datetime.timedelta(seconds=32400), 'JST')), 'updated_at': datetime.datetime(2021, 6, 8, 15, 49, 4, 279643, tzinfo=datetime.timezone(datetime.timedelta(seconds=32400), 'JST'))}
2021-06-08 15:49:04,318 INFO sqlalchemy.engine.Engine INSERT INTO employee (name, phone, created_at, updated_at) VALUES (%(name)s, %(phone)s, %(created_at)s, %(updated_at)s)
2021-06-08 15:49:04,318 INFO sqlalchemy.engine.Engine [cached since 0.01985s ago] {'name': 'employee000008', 'phone': '000008', 'created_at': datetime.datetime(2021, 6, 8, 15, 49, 4, 279680, tzinfo=datetime.timezone(datetime.timedelta(seconds=32400), 'JST')), 'updated_at': datetime.datetime(2021, 6, 8, 15, 49, 4, 279680, tzinfo=datetime.timezone(datetime.timedelta(seconds=32400), 'JST'))}
2021-06-08 15:49:04,319 INFO sqlalchemy.engine.Engine INSERT INTO employee (name, phone, created_at, updated_at) VALUES (%(name)s, %(phone)s, %(created_at)s, %(updated_at)s)
2021-06-08 15:49:04,319 INFO sqlalchemy.engine.Engine [cached since 0.02132s ago] {'name': 'employee000009', 'phone': '000009', 'created_at': datetime.datetime(2021, 6, 8, 15, 49, 4, 279720, tzinfo=datetime.timezone(datetime.timedelta(seconds=32400), 'JST')), 'updated_at': datetime.datetime(2021, 6, 8, 15, 49, 4, 279720, tzinfo=datetime.timezone(datetime.timedelta(seconds=32400), 'JST'))}
2021-06-08 15:49:04,320 INFO sqlalchemy.engine.Engine INSERT INTO employee (name, phone, created_at, updated_at) VALUES (%(name)s, %(phone)s, %(created_at)s, %(updated_at)s)
2021-06-08 15:49:04,321 INFO sqlalchemy.engine.Engine [cached since 0.02268s ago] {'name': 'employee000010', 'phone': '000010', 'created_at': datetime.datetime(2021, 6, 8, 15, 49, 4, 279759, tzinfo=datetime.timezone(datetime.timedelta(seconds=32400), 'JST')), 'updated_at': datetime.datetime(2021, 6, 8, 15, 49, 4, 279759, tzinfo=datetime.timezone(datetime.timedelta(seconds=32400), 'JST'))}
2021-06-08 15:49:04,323 INFO sqlalchemy.engine.Engine COMMIT           ##トランザクション終了
```

全レコードに対してトランザクションが発生していることがわかる。早い。


### autocommit なしでループ外で commit()

コードサンプル

```python
# セッションの設定(autocommit なし)
Session = sessionmaker(
    # autocommit=True,
    class_=MasterSlaveSession
)
```

```python
# ループ外で commit()
def insert():
    rdb_session = Session()
    for i in range(10000):
        work_num = i + 1
        work_time = jst_time(time.time())
        employee = Employee()
        employee.name = 'employee' + str(work_num).zfill(6)
        employee.phone = str(work_num).zfill(6)
        employee.created_at = work_time
        employee.updated_at = work_time
        rdb_session.add(employee)

        # autocommit ありでここで flush() すると毎行トランザクションが発生するので遅い
        # rdb_session.flush()

    # autocommit ありの場合は flush(), なしの場合は commit() するとトランザクションは処理対象行全体にかかるので早い
    # rdb_session.flush()
    rdb_session.commit()
```



ログ(実行結果)

```bash
##
# 10,000 件処理したときの時間
#  * autocommit なし
#  * loop 外で commit
#
% python sqlalchemy_work.py
2021-06-08 15:31:14.633875+09:00 process start
process had got 8.680613994598389sec to done.         # 10,000件 insert するのに約8秒
2021-06-08 15:31:23.314489+09:00 process had finished

##
# ログ( これは 10件処理したときのログ )
# 10件に対してまとめてトランザクションがかかっている事がわかる
2021-06-08 15:52:46,799 INFO sqlalchemy.engine.Engine BEGIN (implicit) ##トランザクション開始
2021-06-08 15:52:46,801 INFO sqlalchemy.engine.Engine INSERT INTO employee (name, phone, created_at, updated_at) VALUES (%(name)s, %(phone)s, %(created_at)s, %(updated_at)s)
2021-06-08 15:52:46,801 INFO sqlalchemy.engine.Engine [generated in 0.00026s] {'name': 'employee000001', 'phone': '000001', 'created_at': datetime.datetime(2021, 6, 8, 15, 52, 46, 780246, tzinfo=datetime.timezone(datetime.timedelta(seconds=32400), 'JST')), 'updated_at': datetime.datetime(2021, 6, 8, 15, 52, 46, 780246, tzinfo=datetime.timezone(datetime.timedelta(seconds=32400), 'JST'))}
2021-06-08 15:52:46,804 INFO sqlalchemy.engine.Engine INSERT INTO employee (name, phone, created_at, updated_at) VALUES (%(name)s, %(phone)s, %(created_at)s, %(updated_at)s)
2021-06-08 15:52:46,804 INFO sqlalchemy.engine.Engine [cached since 0.003321s ago] {'name': 'employee000002', 'phone': '000002', 'created_at': datetime.datetime(2021, 6, 8, 15, 52, 46, 782529, tzinfo=datetime.timezone(datetime.timedelta(seconds=32400), 'JST')), 'updated_at': datetime.datetime(2021, 6, 8, 15, 52, 46, 782529, tzinfo=datetime.timezone(datetime.timedelta(seconds=32400), 'JST'))}
2021-06-08 15:52:46,806 INFO sqlalchemy.engine.Engine INSERT INTO employee (name, phone, created_at, updated_at) VALUES (%(name)s, %(phone)s, %(created_at)s, %(updated_at)s)
2021-06-08 15:52:46,806 INFO sqlalchemy.engine.Engine [cached since 0.00499s ago] {'name': 'employee000003', 'phone': '000003', 'created_at': datetime.datetime(2021, 6, 8, 15, 52, 46, 782593, tzinfo=datetime.timezone(datetime.timedelta(seconds=32400), 'JST')), 'updated_at': datetime.datetime(2021, 6, 8, 15, 52, 46, 782593, tzinfo=datetime.timezone(datetime.timedelta(seconds=32400), 'JST'))}
2021-06-08 15:52:46,807 INFO sqlalchemy.engine.Engine INSERT INTO employee (name, phone, created_at, updated_at) VALUES (%(name)s, %(phone)s, %(created_at)s, %(updated_at)s)
2021-06-08 15:52:46,807 INFO sqlalchemy.engine.Engine [cached since 0.006349s ago] {'name': 'employee000004', 'phone': '000004', 'created_at': datetime.datetime(2021, 6, 8, 15, 52, 46, 782633, tzinfo=datetime.timezone(datetime.timedelta(seconds=32400), 'JST')), 'updated_at': datetime.datetime(2021, 6, 8, 15, 52, 46, 782633, tzinfo=datetime.timezone(datetime.timedelta(seconds=32400), 'JST'))}
2021-06-08 15:52:46,809 INFO sqlalchemy.engine.Engine INSERT INTO employee (name, phone, created_at, updated_at) VALUES (%(name)s, %(phone)s, %(created_at)s, %(updated_at)s)
2021-06-08 15:52:46,809 INFO sqlalchemy.engine.Engine [cached since 0.008276s ago] {'name': 'employee000005', 'phone': '000005', 'created_at': datetime.datetime(2021, 6, 8, 15, 52, 46, 782673, tzinfo=datetime.timezone(datetime.timedelta(seconds=32400), 'JST')), 'updated_at': datetime.datetime(2021, 6, 8, 15, 52, 46, 782673, tzinfo=datetime.timezone(datetime.timedelta(seconds=32400), 'JST'))}
2021-06-08 15:52:46,812 INFO sqlalchemy.engine.Engine INSERT INTO employee (name, phone, created_at, updated_at) VALUES (%(name)s, %(phone)s, %(created_at)s, %(updated_at)s)
2021-06-08 15:52:46,813 INFO sqlalchemy.engine.Engine [cached since 0.01162s ago] {'name': 'employee000006', 'phone': '000006', 'created_at': datetime.datetime(2021, 6, 8, 15, 52, 46, 782711, tzinfo=datetime.timezone(datetime.timedelta(seconds=32400), 'JST')), 'updated_at': datetime.datetime(2021, 6, 8, 15, 52, 46, 782711, tzinfo=datetime.timezone(datetime.timedelta(seconds=32400), 'JST'))}
2021-06-08 15:52:46,814 INFO sqlalchemy.engine.Engine INSERT INTO employee (name, phone, created_at, updated_at) VALUES (%(name)s, %(phone)s, %(created_at)s, %(updated_at)s)
2021-06-08 15:52:46,814 INFO sqlalchemy.engine.Engine [cached since 0.01341s ago] {'name': 'employee000007', 'phone': '000007', 'created_at': datetime.datetime(2021, 6, 8, 15, 52, 46, 782786, tzinfo=datetime.timezone(datetime.timedelta(seconds=32400), 'JST')), 'updated_at': datetime.datetime(2021, 6, 8, 15, 52, 46, 782786, tzinfo=datetime.timezone(datetime.timedelta(seconds=32400), 'JST'))}
2021-06-08 15:52:46,816 INFO sqlalchemy.engine.Engine INSERT INTO employee (name, phone, created_at, updated_at) VALUES (%(name)s, %(phone)s, %(created_at)s, %(updated_at)s)
2021-06-08 15:52:46,816 INFO sqlalchemy.engine.Engine [cached since 0.01495s ago] {'name': 'employee000008', 'phone': '000008', 'created_at': datetime.datetime(2021, 6, 8, 15, 52, 46, 782848, tzinfo=datetime.timezone(datetime.timedelta(seconds=32400), 'JST')), 'updated_at': datetime.datetime(2021, 6, 8, 15, 52, 46, 782848, tzinfo=datetime.timezone(datetime.timedelta(seconds=32400), 'JST'))}
2021-06-08 15:52:46,817 INFO sqlalchemy.engine.Engine INSERT INTO employee (name, phone, created_at, updated_at) VALUES (%(name)s, %(phone)s, %(created_at)s, %(updated_at)s)
2021-06-08 15:52:46,818 INFO sqlalchemy.engine.Engine [cached since 0.01665s ago] {'name': 'employee000009', 'phone': '000009', 'created_at': datetime.datetime(2021, 6, 8, 15, 52, 46, 782887, tzinfo=datetime.timezone(datetime.timedelta(seconds=32400), 'JST')), 'updated_at': datetime.datetime(2021, 6, 8, 15, 52, 46, 782887, tzinfo=datetime.timezone(datetime.timedelta(seconds=32400), 'JST'))}
2021-06-08 15:52:46,819 INFO sqlalchemy.engine.Engine INSERT INTO employee (name, phone, created_at, updated_at) VALUES (%(name)s, %(phone)s, %(created_at)s, %(updated_at)s)
2021-06-08 15:52:46,820 INFO sqlalchemy.engine.Engine [cached since 0.01851s ago] {'name': 'employee000010', 'phone': '000010', 'created_at': datetime.datetime(2021, 6, 8, 15, 52, 46, 782928, tzinfo=datetime.timezone(datetime.timedelta(seconds=32400), 'JST')), 'updated_at': datetime.datetime(2021, 6, 8, 15, 52, 46, 782928, tzinfo=datetime.timezone(datetime.timedelta(seconds=32400), 'JST'))}
2021-06-08 15:52:46,823 INFO sqlalchemy.engine.Engine COMMIT           ##トランザクション終了
```

こちらも同じ。
全レコードに対してトランザクションが発生していることがわかる。早い。


# 補足

## autocommit モードは非推奨 -> 廃止に

autocommit モードは v1.4.x で非推奨、v2.0.x からは廃止になるそうです。

- 参考
  - https://docs.sqlalchemy.org/en/13/orm/session_transaction.html#autocommit-mode

- 転載

> Deprecated since version 1.4: “autocommit” mode is a legacy mode of use and should not be considered for new projects. The feature will be deprecated in SQLAlchemy 1.4 and removed in version 2.0; both versions provide a more refined “autobegin” approach that allows the Session.begin() method to be used normally. If autocommit mode is used, it is strongly advised that the application at least ensure that transaction scope is made present via the Session.begin() method, rather than using the session in pure autocommit mode.



## バルク処理

今回はトランザクションによって処理時間の短縮に努めましたが、

- [bulk_inserts](https://docs.sqlalchemy.org/en/13/_modules/examples/performance/bulk_inserts.html)

- [bulk_updates](https://docs.sqlalchemy.org/en/13/_modules/examples/performance/bulk_updates.html)

といった並列処理もあります。

これらの学習を進め、今後機会があればはそちらも積極的に使っていきたいと思います。


# 参考

- [公式ドキュメント-Transactions and Connection Management](https://docs.sqlalchemy.org/en/13/orm/session_transaction.html)
- [SQLAlchemyを使ってPythonでORM ― SQLAlchemy ORMを知る](https://www.kaitoy.xyz/2020/11/08/sqlalchemy-orm/)


# ソースコード

本記事であげたコードは [こちら](https://github.com/ksh-fthr/python-work-in-docker) にあげております。
ご興味あればご覧ください。
( 上記には冒頭で触れた `update` 処理のサンプルコードもあります )
