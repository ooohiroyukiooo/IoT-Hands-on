# Vol.002 OpenCVで画像処理


## 準備するもの


### 必須

* [Vol.000でセットアップしたRaspberry Pi](../vol000-raspberrypi-setup/README.md)
（体験会では既にご用意しております。）
* [Vol.001で作成した電子回路](../vol001-pyroelectric-sensor/README.md)
* [PiCamera](https://www.amazon.co.jp/dp/B01D1D0DJ0)


#### PC環境

### 任意

* モバイルバッテリ（離れた場所に Raspberry Piを置く場合）
* Slackアカウント


## 体験会

### 今回やること

**OpenCVを用いた、人物判定システム** の作成。

vol001で撮影した写真を OpeCVで分析することで、撮影した動体が人物かどうかを判定します。

その判定結果を Slackに通知しましょう。

* 練習1「PiCameraの映像をストリーミング再生します」

* 練習2「OpenCVを実装します」

* 練習3「システムを作動させて人物判定を行います」

* 発展課題1「動体を検知したら、イベント録画をしましょう」


### 練習1「PiCameraの映像をストリーミング再生します」

いよいよ、本体験会の醍醐味である人物判定に進みます。


#### 「ストリーミング再生に必要なパッケージのインストール」

PiCameraで撮影している映像をブラウザでストリーミング再生して、動画を撮影できることを確認します。

以下のコマンドを Raspberry Piで実行して、必要なパッケージをインストールします。

```
$ sudo apt -y install ffmpeg python3-picamera python3-ws4py
$ git clone https://github.com/waveform80/pistreaming.git ~/pistreaming
```


#### 「ブラウザでストリーミング再生」

アプリを実行します。

```
$ cd ~/pistreaming
$ python3 server.py
```

インストールに成功していれば、このように表示されますので、

<img src="./docs/exercise1/pistreaming.png" alt="pistreaming" title="pistreaming">

ご自身のブラウザで

`http://[Raspberry PiのIPアドレス]:8082/index.html`

にアクセスしてみましょう！

PiCameraで撮影している映像がリアルタイムでストリーミングされているはずです。

アプリを終了させるには Raspberry Piのコマンドラインで、`Ctrl` + `C` を同時に押下します。


### 練習2「OpenCVを実装します」

人物かどうかを判定するために、今回は顔検出を行います。


#### OpenCVのインストール

[OpenCV 公式ドキュメント](https://docs.opencv.org/3.4.3/d7/d8b/tutorial_py_face_detection.html) をもとに実装していきます。

顔検出の詳しい解説は論文等に譲り、割愛させていただきます。
(参照=> [Face Recognition with OpenCV](https://docs.opencv.org/3.4.3/da/d60/tutorial_face_main.html))

OpenCVで使用するライブラリをインストールします。

インストールするライブラリを記載したシェルスクリプトを配置しているディレクトリに移動します。

```
$ cd ../IoT-Hands-on/vol002-opencv/
```

ディレクトリの中には、下記のファイルやディレクトリがございます。

<img src="./docs/exercise2/ls.png" alt="ls" title="ls">

シェルスクリプトを実行して、ライブラリをインストールします。

```
$ bash setup.sh
```

インストールが開始されます。

<img src="./docs/exercise2/bash_setup.png" alt="bash_setup" title="bash_setup">


#### 分類器の準備

今回の顔検出では、学習済みの [分類器](https://github.com/OpenCV/OpenCV/blob/master/data/haarcascades/haarcascade_frontalface_default.xml) を使用します。

`curl` コマンドで、分類器を Raspberry Piに直接ダウンロードしましょう。

curlの [公式ドキュメント](https://github.com/curl/curl/tree/master/docs/cmdline-opts) より、

```
$ curl --create-dirs -o "./haarcascades/haarcascade_frontalface_default.xml" https://raw.githubusercontent.com/openCV/openCV/master/data/haarcascades/haarcascade_frontalface_default.xml
```

と実行することで、

<img src="./docs/exercise2/haarcascade_download.png" alt="bash_setup" title="bash_setup">

`vol002-opencv` 配下に分類器を保存できました。


### 練習3「システムを作動させて人物判定を行います」

#### YAMLファイルの編集

Slackの認証情報とOpenCVで用いる分類器を使用するために、YAMLファイルを編集しましょう。

YAMLファイル `config.sample.yml` を nanoコマンド等で開いて、【設定情報】のように3箇所ほど編集します。

```
$ nano config.sample.yml
```

<img src="./docs/exercise3/nano_config-sample-yml.png" alt="config" title="config">

**【設定情報】**
```
# Slack
slack_token: # Slackの generalチャンネルに貼ってある "xoxp-"で始まる文字列
slack_channel: 'security-camera'

# OpenCV
opencv_classifier_name: './haarcascades/haarcascade_frontalface_default.xml'
```

`slack_token` はご招待した Slackのワークスペースの `general` チャンネルに記載してあります。

vol001と同じ手順で `config.yml` ファイル名を変更して保存します。


#### システムの作動

それでは、準備が整いましたので、ソースコードを実行して、動作を確認してみましょう！

```
python3 main.py
```

成功すれば、動体を検知すると写真を撮影し、人物判定を行った後に、Slackに通知を送信します。





### 発展課題1「動体を検知したら、イベント録画をしましょう」

vol001 も vol002 の練習も終わったという方は、発展課題に移りましょう。

picamera の
[circularIO](https://picamera.readthedocs.io/en/latest/recipes1.html?highlight=Circular#recording-to-a-circular-stream)
を使って、イベント録画を行います。

このクラスを使えば、ドライブレコーダーのように、あるイベントを起点として、そのイベントの前後 n秒間を録画することができます。

あるイベントは、例えば、動体を検知したとき、揺れを検知したときなどがあります。

Python で Raspberry Pi の GPIO を制御するモジュールの RPi.GPIO には、
[event_detected() 関数](https://sourceforge.net/p/raspberry-gpio-python/wiki/Inputs/)
が用意されています。

この関数を用いて、「焦電センサで動体を検知した」のような起点となるイベントを検知します。


#### CircularIO.py を編集して、main.py から実行できるようにする

`vol002-open-cv` には既に、`circularIO` のサンプルプログラムが用意してあります。

`main.py` と `CircularIO.py` を編集することによって、イベント録画を実現してみましょう！


#### Google Drive にファイルをアップロードする

撮影した動画は、`.h264` 形式で保存されます。

ディスプレイがないと Raspberry Pi では再生できませんので、`main.py` にはあらかじめ GoogleDrive にアップロードするようなコードが書かれています。

`main.py` を `vi` エディターで開くと、コメントアウトの指示が 5か所ありますので、指示を見ながら編集してください。

GoogleDrive にファイルをアップロードするには、
[API](https://developers.google.com/drive/api/v3/manage-uploads)
を使います。

OAuth2.0 認証が必要ですので、
[こちら](https://support.google.com/googleapi/answer/6158849?hl=en&ref_topic=7013279)
を見ながら認証してください。

手順に沿ってダウンロードした `credentials.json` を Raspberry Pi に SCP 通信で移動させます。

`credentials` ディレクトリを作成してから、移動させます。

```
@raspberrypi:~/IoT-Hands-on/vol002-opencv $ mkdir credentials
```

ここまで出来たら、あとは動画をアップロードする GoogleDrive のフォルダを指定するだけです。

```
https://drive.google.com/drive/[your_folder_id]
```

GoogleDrive の該当フォルダに移動して、URLを確認してください。

[your_folder_id] の部分でアップロード先のフォルダを指定することができます。

取得したIDで、`UploadGoogleDrive.py` の

```
FOLDER_ID = `your_folder_id`
```

を書き換えます。


#### イベント録画を起動し、撮影し動画をGoogleDriveにアップロードする

ここまで出来ましたら、あとは `main.py` を実行するだけです。

今回は、コマンドラインで認証を行うので、オプションをつける必要があります。

```
python3 main.py --noauth_local_webserver
```

指示通りに認証を完了させると、イベント録画が始まります。

