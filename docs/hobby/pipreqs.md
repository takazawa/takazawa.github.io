# pipreqsを利用してrequirements.txtをスマートに作成

[``pipreqs``](https://github.com/bndr/pipreqs)を利用すると現在のプロジェクトに必要なパッケージのみから構成される「requirements.txt」を作成できる．



### Why not pip freeze?

[Google翻訳]

- `` pip freeze``はあなたの環境に `` pip install``でインストールされたパッケージだけを保存します。

- ``pip freeze``は、現在のプロジェクトで使用しないパッケージを含め、環境内のすべてのパッケージを保存します。 （もしあなたがvirtualenvを持っていないなら）

- モジュールをインストールせずに新しいプロジェクトのrequirements.txtを作成するだけでよい場合もあります。

### インストール
```
pip install pipreqs
```

### 使い方
プロジェクトのルートディレクトリで以下を実行するとrequirements.txtが生成される．
```
pipreqs .
```

### pigar

似たようなことができるライブラリ

https://github.com/damnever/pigar
