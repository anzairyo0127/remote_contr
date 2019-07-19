Dockerコンテナを開発環境にできるVSCodeRemoteContainerがでました。

環境をコンテナに投げ込むように使えるので、ローカルがクリーンになりますし、またVSCodeの拡張機能もコンテナ内に投げ込むことが出来ますのでVSCodeもクリーンを保てます。すごい。

# VSCode RemoteContainer

以下がVSCODEの拡張機能です。

https://marketplace.visualstudio.com/items?itemName=ms-vscode-remote.remote-containers

インストールが完了してからVSCODEを再起動する。再起動するとVSCode左下に`><`のようなマークが追加されています。

## 以下のようなディレクトリ構成にする

`.devcontainer.json`というものがRemoteContainerの設定ファイルになります。

```
├── docker-compose.yml
├── .devcontainer.json
└── src
    └── app.py
```

## DockerComposeは好きなように使う

DockerComposeそのものは普段通りにしてもらえればよいと思います。

```docker-compose.yml
# docker-compose.yml
version: '3'
services:
    python:
        image: python:3.7.4
        volumes:
             - ./src:/var/src
        ports:
            - "5000:5000"
        tty: true
```

`tty: true`の記述がないとエラーになりました。`RemoteContainers version:0.66.0`時点。

## srcディレクトリにPythonのソースを入れる

今回は`flask`を少しいじってみる。このファイルが実行されている間、`http://localhost:5000/`にアクセスすると`Hello World!`と表示されます。

```python
# app.py
from flask import Flask
app = Flask(__name__)


@app.route("/")
def index():
    return "Hello World!"
```

## devcontainer.jsonを作成する

`devcontainer.json`はRemoteContainerを起動する際に必要なものですが、存在しない場合は自動で生成されます。

```json
{
    "dockerComposeFile": "docker-compose.yml",
    "service": "python",
    "workspaceFolder": "/var/src",
    "extensions": [
        "ms-python.python",
    ]
}
```

`dockerComposeFile`は使用するDockerComposeを指定します。専用のDockerComposeを使用することも出来るようです。

`service`とはDockerComposeのserviceを指定します。

`workspaceFolder`とはコンテナ内の作業ディレクトリを指します。プログラムファイルを置く場所であったり、リポジトリ等を指定すると良いでしょう。DockerComposeのVolume機能を使って共有するように使うのがベターかと思います。

`extensions`とはコンテナ内で起動する`VSCodeの拡張機能`を記載します。VSCodeの自分の使用している言語の拡張機能は自動でインストールされるようですが、それ以外はインストールされていません。`ms-python.python`はVSCodeのPythonの拡張機能になります。他にも必要なものがあればlistとして記載します。

## RemoteContainerを開く

VSCode左下に`><`のようなマークをクリックすると上部からリストが表示され、いくつかの選択肢が出てきます。`Remote-Containers:Open Folder in Container...`を選択する。今回のディレクトリを選択します。

新しいVSCodeが表示されます。その中は既にコンテナの中です。`app.py`を表示すると`pylint`等をインストールするよう求められます。既にVSCodeのPathはVSCodeを向いているので、自身のローカルにインストール済みのPylintを読みに行けていないのがわかったと思います。

VSCodeのターミナルも起動していると思いますのでそれらをインストールしたり、`pip install pylint pep8 autepep8 bandit`などでlint用の外部モジュールを追加します。これが面倒であれば自分のDockerfileを作ってDockerComposeで指定してあげると良いと思います。

終了すれば`from flask`の下部に波線が出来ると思います。これはPylintの機能でモジュールが発見出来ない場合に警告が出るようになっています。`flask`をインストールすればそれは消えるはずです。`pip install flask`を実施します。波線が消えない場合は`app.py`を再度保存してください。

`flask run --host=0.0.0.0`とすることでflaskが起動します。ホストPCのWebブラウザでhttp://localhost:5000/とアクセスしてみると`Hello World!`と表示されているはずです。

## PythonをDebugする

`foo.py`という適当なPythonプログラムを作成し、中身を以下のようにします。

```python
for i in range(5):
    print(i)
```

`for i in range(5):` の箇所でブレークポイントを指定します。その上でF5キーを押してみましょう。いくつかの選択肢の内、`Pyhon File`を選択してみてください。ブレークポイントを使ったデバッグも出来るようになっているはずです。これはVSCodeの`ms-python.python`という拡張機能のおかげのようです。
