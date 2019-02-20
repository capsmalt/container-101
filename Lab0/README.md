# 1) WAS Libertyの基礎操作を試す
## 1-1) WAS Libertyの基本操作(wlp/bin配下にコマンド有り)
**※Windowsの場合**は，binディレクトリに移動して `server start <Libertyのサーバー名>` のように実行

```bash
- バージョン，エディション確認
$ bin/productInfo version

- Libertyバイナリにインストール済のフィーチャー(Java EEなどの各種機能群)
$ bin/productInfo featureInfo

- Libertyサーバー起動
$ bin/server start [Libertyのサーバー名]

- Libertyサーバー停止
$ bin/server stop [Libertyのサーバー名]

- Libertyサーバーのバックエンド実行 (Ctrl+Cで終了)
$ bin/server run [Libertyのサーバー名]
```

## 1-2) WAS Libertyのパッケージング機能

```bash
- LibertyサーバーのALLパッケージング

$ bin/server package [Libertyのサーバー名] --archive=[任意の名前].zip --include=all
```
>補足:  
> - パッケージング対象: `全て (Libertyのバイナリ・サーバー構成ファイル・アプリ等)`
> 
> - パッケージ対象に含まれるディレクトリ: `wlp/~~~`
> 
> - 出力されるパッケージ(zip)配置場所: `wlp/usr/servers/Libertyのサーバー名/任意の名前.zip`
> 
> - `--include=all 指定` は，任意のLibertyサーバーまるごと渡す場合に便利。他に外部接続などする必要がなければ，まったく同じ環境を簡単に用意可能


```bash
- Libertyサーバーの構成(対象ディレクトリ: wlp/bin/usr/~~~)

$ bin/server package [Libertyのサーバー名] --archive=[任意の名前].zip --include=usr
```

>補足:  
> - パッケージング対象: `サーバー構成ファイル，アプリ等`
> - パッケージ対象に含まれるディレクトリ: `wlp/usr/servers/Libertyのサーバー名/~~~`
> - 出力されるパッケージ(zip)配置場所: `wlp/usr/servers/Libertyのサーバー名/任意の名前.zip`
> `--include=usr 指定` は，Libertyの構成とアプリケーションをセットで渡す場合に便利


# 2) コンテナイメージ作成を試す
ソースコードとWAS Libertyをまとめてコンテナイメージ化

## 2-1) コンテナ内に必要なファイル群をコピー
メインとなるコンテナ(WAS Libertyコンテナ)を起動し，その中にアプリケーションやサーバー構成などを作成(コピー)

```bash
- WAS Libertyの標準イメージからコンテナを起動して，コンテナ内に入る  

コマンド構文:
$ docker run -it [イメージ名] /bin/bash

実行例:
$ docker run -it websphere-liberty:19.0.0.1-webProfile7 /bin/bash

出力:
default@6143822637f0:/$

※コンテナ内に入ったのでプロンプトが変化
```

```bash
- 起動中コンテナのコンテナIDを取得  

コマンド構文:
$ docker ps

実行例:
$ docker ps

出力:
CONTAINER ID        IMAGE                                    COMMAND                  CREATED              STATUS              PORTS                NAMES
6143822637f0        websphere-liberty:19.0.0.1-webProfile7   "/opt/ibm/helpers/ru…"   About a minute ago   Up About a minute   9080/tcp, 9443/tcp   focused_banach


上記例の場合， `6143822637f0` がコンテナID。
```

```bash
- コンテナ内にホスト(作業PC)から必要ファイルをコピー  

コマンド構文:
$ docker cp [コピーしたいファイル] [コンテナID]:[コピー先のコンテナ内のパス]

実行例:
$ docker cp test.war 6143822637f0:/opt/ibm/wlp/usr/servers/s1/dropins/test.war

出力:
なし
```

>補足:  
> 例えばWAS Liberty関連の4つのファイルをコンテナ内にコピーする場合は以下のように実行。
> 
> 1~3は，`wlp/usr/servers/Libertyのサーバー名/~~~` にコピー。
> 
> 4は，`wlp/usr/servers/Libertyのサーバー名/dropins/~~~`にコピー
> 
> 1) server.xml の場合  
>   `$ docker cp test.war 6143822637f0:/opt/ibm/wlp/usr/servers/s1/server.xml`
> 2) server.env の場合  
>   `$ docker cp test.war 6143822637f0:/opt/ibm/wlp/usr/servers/s1/server.env`
> 3) jvm.options の場合  
>   `$ docker cp test.war 6143822637f0:/opt/ibm/wlp/usr/servers/s1/jvm.options`
> 4) test.war の場合  
>   `$ docker cp test.war 6143822637f0:/opt/ibm/wlp/usr/servers/s1/dropins/test.war`


## 2-2) コンテナイメージの作成
必要なファイル群などが用意された **起動中のコンテナからコンテナイメージを作成**

```bash
- 起動中コンテナをイメージとして保存

コマンド構文:
$ docker commit [前の手順で操作中のコンテナID] [任意のコンテナ名]

実行例:  
$ docker commit 6143822637f0 mylibertyapp:v20190220

出力: 
sha256:fb383d592ff53db22af1faddfdcd711cfd963e346316b044355aff3a5cee375a
```

```
- 作成したイメージを確認

コマンド構文:
$ docker images

実行例:  
$ docker images

出力:
REPOSITORY              TAG                      IMAGE ID            CREATED             SIZE
mylibertyapp            v20190220                fb383d592ff5        57 seconds ago      622MB
```


```bash
- 作成したイメージの名前やタグの変更して，再度イメージを確認

コマンド構文:
$ docker tag [作成したコンテナ名]:[現在のタグ名] [変更後のコンテナ名]:[変更後のタグ名]

実行例:  
$ docker tag mylibertyapp:v20190220 myrestapp:v20190221
$ docker images

出力:
REPOSITORY              TAG                      IMAGE ID            CREATED             SIZE
mylibertyapp            v20190220                fb383d592ff5        4 minutes ago       622MB
myrestapp               v20190221                fb383d592ff5        4 minutes ago       622MB

``` 

>補足:  
> 上記例の 2つのコンテナイメージの `IMAGE ID` は共に`fb383d592ff5`。
> つまり同一のコンテナイメージに対して2パターンのタグをつけて管理している。


```bash
- 不要なイメージ&タグの削除(※同じコンテナIDを指しているタグがある場合はタグ外し)

コマンド構文:
$ docker rmi [イメージ名]:[タグ名]

実行例:  
$ docker rmi mylibertyapp:v20190220
$ docker images

出力:
Untagged: mylibertyapp:v20190220

REPOSITORY              TAG                      IMAGE ID            CREATED             SIZE
myrestapp               v20190221                5a4c357e7941        21 minutes ago      622MB
```

>補足:  
> 上記例では，mylibertyapp:v20190220が削除されて，myrestapp:v20190221のみになったことが確認できる。

### コンテナイメージのtarball化
docker pullなどできない環境(e.g. オフライン環境)に対して，tarballをコピーしてdocker loadすればコンテナイメージを取り込むことが可能。

```bash
- コンテナイメージをtar.gzで保存(docker save)

コマンド構文:
$ docker save [イメージ名] > [任意のファイル名].tar.gz

実行例:  
$ docker save myrestapp > myrestapp.tar.gz

出力:
なし
```

```bash
- コンテナイメージをtarballから各端末にロード

コマンド構文:
$ docker load < [コンテナイメージのtarball]

実行例:  
$ docker load < myrestapp.tar.gz

出力:
Loaded image: myrestapp:v20190221
```

>補足:  
> すでにmyrestapp:v20190221が手元にある場合は，再度ロードしても変化はない。
> ロードを試す場合は，他の端末で`docker load`するか，一度 `docker rmi myrestapp:v20190221` で削除してから行う。

***
