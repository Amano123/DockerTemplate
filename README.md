# Docker Template

## TODO
* [x] ~~Owner問題解決~~
* [ ] .envファイルをshellで生成 

## 構成について
以下のような構成になっている。

```
.
├── .env
├── README.md
├── data_ubuntu
├── docker-compose.yml
└── ubuntu
    └── Dockerfile
```

.emvにホストの`USER_ID`、`GROUP ID`を記述している


#### `user idを確認するコマンド`
```
id -u
```

#### `group idを確認するコマンド` 
```
id -g
```
上記のコマンドで確認したIDを`USER_ID`、`GROUP ID`に対応するように書き換える。  
USER NAMEはお好みで指定して。

`.emvの中身`
```.emv
# User name
USER_NAME="yo"

# User ID
USER_ID=1000

# GROUP ID
GROUP_ID=1000
```

次に、docker-composeでbuildする。
```
docker-compose build
```

Containerを立ち上げる
```
docker-compose up -d
```

中に入る
今回はContainer名を`ubuntu`にしているので
```
docker exec -it `ubuntu` bash
```

ホストとコンテナの`ID`を統一し、`USER NAME`を異なる場合での検証

![sample](https://user-images.githubusercontent.com/39152214/104838502-9a624100-58fe-11eb-84c2-1e78e644be2a.gif)

`USER_ID`, `GROUP ID`が一致していれば、`USER NAME`が異なっても動作することが確認できた。

## 環境変数について
docker build時に`USER NAME`, `USER ID`, `GROUP ID`が必要になるが、Dockerfileでの環境変数を指定には外部ファイル(.env)では不可。

dockerfileに直接記述するのも不格好なのでDocker-compose経由で`USER NAME`, `USER ID`, `GROUP ID`を渡すことにした。


```
.env -> docker-compose.yml -> dockerfile
```


## Owner問題をについて
dockerではrunするときに`-vオプション`、docker-composeでは`volume`を指定することでホストのディレクトリをコンテナ内にマウントすることができる。

マウントすることにより、dockerを再起動してもデータを永続化することができ、双方からファイル、ディレクトリが確認できるので便利である。

しかし、ホストとコンテナの片方でファイル、フォルダを生成した場合、他方では他ユーザーが作成したファイル、フォルダと判断されてまう。

このような場合に修正、追記をする際に`パーミッションエラー`が発生する。回避するために様々な手順を踏む必要がある。(やっかい)

コンテナで作成したファイルをホストで書き換えようとするとホスト側で
* `sudo`で強制的に書き換える
* `chmod`を使用してアクセス許可
* `chown`で所有者やグループを変更

上記の方法で現在まで対処してきたが、事ある毎にコマンドを実行するのが煩雑なので対策した。これでQOLが上がる

原因として、ホストとコンテナでの`USER ID`, `GROUP ID`が異なる為である。
（USER NAMEは異なっていても大丈夫だった。）

|  | USER ID | GROUP ID |
| :-: | :-: | :-: |
| `host` | 1000 | 1000 |
| `Container` | 100 | 100 |

このように`ID`が異なっていると上記の問題が起こる。なので、一緒にしてしまえばいいのだ！

