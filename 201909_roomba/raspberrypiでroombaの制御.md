## 概要
- raspberry piを利用してIOT機器（ルンバ）を操作する

- できること
  - Siriで操作可能
  - その他のスマート家具と組み合わせてオートメーション化できる

## 作業内容
### ラズベリーパイの初期設定
1. スターターキットを組み立て
2. OSは最初から入っているため、各種アップデート
```
# ソフトウェアアップデート
$ sudo apt-get update
$ sudo apt-get upgrade

# 日本語設定 (下記インストールしたらGUIで日本語環境に設定)
$ sudo apt-get install fonts-ipaexfont
```
3. ssh利用の設定
```
# デフォルトの場合、外部からssh接続できないため許可する
$ sudo raspi-config

# Interfacing Options -> P2 SSH -> はい
```
4. パスワードの変更：いつもの
```
$ sudo raspi-config

# Change User Passwordを選択
```
5. 外部からssh接続
```
$ ssh pi@<ラズパイのIPアドレス>
```
### node.jsの利用
1. nvmのインストール
```
$ curl -o- https://raw.githubusercontent.com/creationix/nvm/v0.33.11/install.sh | bash
$ nvm install v10.15.0

# 一緒にnodeのバージョン確認も行う
$ which node
$ node -v
```
2. npmのインストール
```
$ sudo apt-get install -y nodejs npm
$ npm -v
6.11.3
```

### ルンバとラズパイの同期
1. homebridgeのインストール
```
$ sudo apt-get install libavahi-compat-libdnssd-dev
$ npm install -g --unsafe-perm homebridge
/home/pi/.nvm/versions/node/v10.15.0/bin/homebridge -> /home/pi/.nvm/versions/node/v10.15.0/lib/node_modules/homebridge/bin/homebridge
```
2. ルンバIPアドレスをiphoneアプリから取得
3. homebridge-roombaのインストール
```
$ npm install -g homebridge-roomba
+ homebridge-roomba@1.0.1
updated 1 package in 10.013s
```
4. ルンバの情報取得
```
# ルンバのIPアドレスを設定
$ npm run getrobotpwd xxx.xxx.xxx.xxx
added 356 packages from 658 contributors and audited 1182 packages in 52.975s
found 2 vulnerabilities (1 low, 1 critical)
  run `npm audit fix` to fix them, or `npm audit` for details
Make sure your robot is on the Home Base and powered on (green lights on). Then press and hold the HOME button on your robot until it plays a series of tones (about 2 seconds). Release the button and your robot will flash WIFI light.
Then press any key here...

# ルンバのHomeボタンを押すと下記情報が取得できる
Robot Data:
{ ver: '3',
  hostname: 'Roomba-XXXXXXXXXXXXXXXX',
  robotname: 'Roomba',
  ip: '192.168.XXX.XXX',
  mac: 'DC:F5:XX:XX:XX:XX',
  sw: '3.4.46',
  sku: 'e515060',
  nc: 0,
  proto: 'mqtt',
  cap: { ota: 1, eco: 1, svcConf: 1 },
  blid: '3192C000000000000' }
Password=> :1:1563448824:AAAAAAAAAAAAAAAAa <= Yes, all this string.
Use this credentials in dorita980 lib :)

```
5. ルンバ操作設定ファイルの作成

```
# ~/.homebridge/config.jsonに作成
{
	"bridge":
	{
		"name": "Homebridge",
		"username": "DC:F5:XX:XX:XX:XX",
	        "port": 51826,
	        "pin": "031-45-154"
	},
        "accessories": [
	{
		"accessory": "Roomba",
		"name": "Roomba",
		"blid":"3192C000000000000",
		"robotpwd":":1:1563448824:AAAAAAAAAAAAAAAAa",
		"ipaddress": "192.168.XXX.XXX"
	}
	]
}
```
### ルンバの実行
1. 起動確認
```
$ homebridge
# Iphone設定用のQRコードが表示され、Iphoneで読み取りするとタスクが設定できる
```
![](/source/img/2019-09-07-14-00-29.png)
2. ラズパイで実行
```
# 下記コマンドでルンバが走り出す
systemctl start homebridge
```
