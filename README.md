# URoadHome2+をターミナルから操作するスクリプト
シンセイコーポレーションの [URoadHome2+](http://www.shinseicorp.com/wimax2plus/uroad-home2plus/) をターミナルから操作するスクリプト。


```
usage: ./urh2ctl [-h] [--user USER] [--password PASSWORD]
                 {stat,status,switch} ...

URoadHome2+ control

optional arguments:
  -h, --help            show this help message and exit
  --user USER           BASIC auth user name (default: admin)
  --password PASSWORD   BASIC auth password

command:
  {stat,status,switch}
  stat                show statistics
  status              show status
  switch              switch mode
```


## 必要なもの
Pythonの`requests`モジュールが必須。Python3.5で動作確認しているが、3系なら動くと思われる。



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


## その他
ルーターのIPアドレスはハードコーディングで`192.168.100.254`になっている。デフォルトでこの設定だが、わざわざ変更している人はコードをいじらないと動作しないので要注意。

