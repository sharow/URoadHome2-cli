# URoadHome2+をターミナルから操作するスクリプト
シンセイコーポレーションの [URoadHome2+](http://www.shinseicorp.com/wimax2plus/uroad-home2plus/) をターミナルから操作するスクリプト。


```
usage: ./urh2ctl [-h] [--user USER] [--password PASSWORD]
                 {stat,status,switch,record,dump,summary} ...

URoadHome2+ control

optional arguments:
  -h, --help            show this help message and exit
  --user USER           BASIC auth user name (default: admin)
  --password PASSWORD   BASIC auth password

command:
  {stat,status,switch,record,dump,summary}
    stat                show statistics
    status              show status
    switch              switch mode
    record              record stat
    dump                dump record
    summary             transfer summary
```


## 必要なもの
Pythonの`requests`モジュールが必須。動作にはPython3.5以上が必要。



## 認証について
URoadHome2+の管理画面はBASIC認証なのでそのユーザー名とパスワードが必要。ユーザー名はデフォルトで`admin`になっている。パスワードは説明書に書かれた内容に従えば分かる。

パスワードは環境変数`URH2_PASSWORD`に設定するか、あるいは`--password`引数で指定する。



## ステータス取得
```
$ URH2_PASSWORD=<router-password> ./urh2ctl status
Mode: LTE (High Speed Mode)
LTE Signal Level: 4 (Good Signal)
LTE Connection Status: Connected
```
ステータスには`--all`引数があり、ルーター側の情報をベェっと吐かせることもできる(電話番号とかある)。


## システム統計(データ転送量)取得
```
$ ./urh2ctl --password <router-password> stat
=statistics=:      rx           tx
WiMAX(today):  900.07 MiB   47.93 MiB
WiMAX(month):  900.07 MiB   47.93 MiB
LTE(today)  :  19.09 MiB   2375.68 KiB
LTE(month)  :  19.09 MiB   2375.68 KiB
$
```
`stat`には`--json`オプションもある。

`URH2_PASSWORD=<router-password> watch -d -n 5 ./urh2ctl stat` とかできる。データ取得には1～2秒ほどかかかるので更新間隔には注意。


## モード切替

デフォルトの挙動では単純にモードを切り替える。
(ここではパスワード引数などは省略)

```
$ ./urh2ctl switch
switch LTE(High Speed Mode) -> WiMAX(No Limit Mode)
switch request sent
waiting for WiMAX connection...
waiting for WiMAX connection...
WiMAX connected!
$
```

LTE(WiMAX2+, High Speed Mode)に切り替えたい時は`--lte`オプションで指定する。既にLTE接続であれば何もしない。同様に`--wimax`オプションもある。

```
$ ./urh2ctl switch --wimax
Already WiMAX.
$
```

## 記録
`./urh2ctl record --datadir ./db`とすると`stat`の内容を`pickle`で吐く。データは日付ごとのファイルで、pickleは追記が可能なので、1日の分はそのファイルに書き出している。読み込む際は`pickle.load(fp)`を`EOFError`が出るまで繰り返せばよい。

`./urh2ctl dump --datadir ./db`とすると記録されたデータをそのまま`print`で吐きだす。

`./urh2ctl summary --datadir ./db`とすると日ごとの通信量を一覧表示する。

```
$ URH2_PASSWORD=<router-password> ./urh2ctl summary --datadir /somewhere/db
2018-01-11: LTE:3060MiB  WiMAX:   0MiB
2018-01-12: LTE:1982MiB  WiMAX:   0MiB
2018-01-13: LTE:2725MiB  WiMAX:   0MiB
2018-01-14: LTE:4250MiB  WiMAX: 341MiB
2018-01-15: LTE:   0MiB  WiMAX:3261MiB
2018-01-16: LTE:2803MiB  WiMAX: 535MiB
LTE Past 3 days        : 6976MiB
LTE Past 2 days + today: 7053MiB
```

2018年現在、WiMAX2+(LTE)は過去3日間(**今日を含まない**)の通信量が10GiBを超えると制限がかかる。



## その他
### ルーターのIPについて
ルーターのIPアドレスはデフォルトで`192.168.100.254`になっている。URoadHome2+は出荷時からこの設定だが、変更している人は環境変数`URH2_IP`に設定する必要がある。


### モード切替時
一時的に通信はできなくなり、再疎通後は前とIPアドレスが変わるので注意。環境によるが遮断は2秒～5秒程度。
