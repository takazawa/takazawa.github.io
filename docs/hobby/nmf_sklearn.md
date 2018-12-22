# Python(scikit-learn)でNMFを利用したレコメンド

scikit-learnを利用したNMF利用の方法の解説の記事はいくつかあったのですが，
実際の用途にあった記事がみつからなかったためメモとして残します．


### ざっくりとしたタスク

まずNMFとか抜きに扱うタスクを説明します．

    (既存)ユーザー×商品のレーティングの行列 (学習データ，大量にあるイメージ)

が与えられたときに，何らかの学習モデルを作っておき（時間がかかっても良い），

    (新規 or ターゲット)ユーザー×商品のレーティングの行列（検証データ，少量）

が新しく得られたときに，レーティングしていない商品の値を（高速に）出して，値が高い商品をレコメンドをしたい．

（あまりこの枠組でscikit-learnのNMFの使い方を書いている人がいないのでメモしました．）


### NMFによる実現

上記のタスクはNMFを利用する場合どのようにして実現されるのかという話をします．


#### 1.学習
`(既存)ユーザー×商品のレーティングの行列`を$X$とします．そうすると，`何らかの学習モデルを作る`ということは，
NMFでは$W$の列（$H$の行）ベクトルの数をパラメータとして与えたもと

$$
X \simeq W \times H 
$$

となるような$W$と$H$を作るということになります．ここで以下のような解釈が可能です．

- $W$はユーザー×特徴の行列で，ユーザーごとの特徴を意味
- $H$は特徴×商品の行列で，商品ごとの特徴を意味

ここで大事なこととしては，$H$は商品ごとの特徴を表しているため，今後もデータの特徴が大きく変わらないとすれば
新しいデータ等が来ても変わらない値と考えらます．(ex. もしホラーの映画に対して，ホラーという特徴が取れたとするとこの特徴は今後も変わらない)
したがってNMFにおいては行列$H$が学習モデルと捉えることができると思います．

ちなみに，この時点で得られた$W$と$H$を使ってあげて
$$
X' = W \times H 
$$
のように$X'$を計算してあげることによって，既存ユーザー相手にはレコメンド可能となっています（$X$において0だった要素に$X'$においては正の値が入る場合があるため）．

NMFの多くの記事ではこの話がメインでした．

#### 2.予測

では，ここで新しく`(新規 or ターゲット)ユーザー×商品のレーティングの行列`が得られたとして，
これを$X_{\text{new}}$としましょう．このとき，上で求めた$X'$のようなものを$X_{\text{new}}$に対して求めてあげるというのが目的です．

NMFの予測フェーズでは，$H$をインプットとして
$$
X_{\text{new}} \simeq W_{\text{new}} \times H 
$$
となるような$W_{\text{new}}$を求めます．ここで先ほどとの違いを説明すると，学習時は

$X \simeq W \times H$となる$W$と$H$を求めていたのに対して，今回は$H$は動かさずに$W_{\text{new}}$だけを求めています．
ここで新しいユーザーの特徴行列$W_{\text{new}}$が得られたため，これを使って
$$
X_{\text{new}} ' = W_{\text{new}} \times H 
$$
を計算してあげます．$X_{\text{new}}'$の0の部分が非負の値で埋まっているため，ユーザーごとにこの値が高い商品をレコメンドすればよいという結果です．


### Python(scikit-learn)による実装

上記の話をPythonにて実装します．scikit-learnを利用するため以下のようにインポートします．
インプットのデータの$X$はテキトウに作りました．

```python
import numpy as np 
from sklearn.decomposition.nmf import non_negative_factorization

num_users = 10000
num_items = 100

# 0と1からなるの行列
X = np.random.randint(0, 1, size = [num_users, num_items], dtype = 'int')

``` 

#### 1.学習
学習は次のように行います．その他のパラメータは[ドキュメント](https://scikit-learn.org/stable/modules/generated/sklearn.decomposition.NMF.html)を参照ください．

```python
n_components = 3 # 特徴量の数（自分で決める）
W, H, n_iter = non_negative_factorization(X, n_components=n_components)
```

$X'$は以下で求めらます．
```python
X_predict = np.dot(W,H)
```

#### 2.予測
新しいユーザーデータ100件に対してレコメンドします．
```python
num_new_users = 100
X_new = np.random.randint(1, 2, size = [num_new_users, num_items], dtype = 'int')
```

$W_{\text{new}}$を以下で求めます．学習で求めた$H$を渡すことと`update_H=False`とすることで$H$を固定しています．
```python
W_new, H, n_iter = non_negative_factorization(X_new,
                                              H=H,
                                              update_H=False,
                                              n_components=n_components)
```
最終的にほしかった$X_{\text{new}}'$は以下で求められます．
```python
X_new_predict = np.dot(W_new, H)
```


