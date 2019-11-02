---
title: PandasのDataFrameのappendの高速化
date: 2019-2-02
categories:
    - hobby
---


`pandas.DataFrame.append`がかなり遅いときの対策メモ

<!-- more -->

### タスク

空のDataFrameに大量のDataFrame(or Series)を付け加える．

今回は例として，以下の空の`df_data`に(2×3の)`df_add`を1万個付け加える．（本当は`df_add`は毎回違うデータと思うが今回は例なので同じものを付け加える想定）

```python
import pandas as pd
df = pd.DataFrame()
df_add = pd.DataFrame([[1, 2, 3], [4, 5, 6]])

# タスク
# dfにdf_addを1万回くっつける
```

### 1. `DataFrame.append`による実現（遅い)

```python
df = pd.DataFrame()
df_add = pd.DataFrame([[1, 2, 3], [4, 5, 6]])

for i in range(10000):
    df = df.append(df_add, ignore_index=True)
```

これは結構時間がかかる．（`df`が毎回コピーされるため?）


### 2. `DataFrame.from_dict`による実現（速い）

```python
df = pd.DataFrame()
df_add = pd.DataFrame([[1, 2, 3], [4, 5, 6]])

counter = 0
dict_tmp = {}

for i in range(10000):
    for _, row in df_add.iteritems():
        dict_tmp[counter] = row
        counter += 1
df = df.from_dict(dict_tmp, orient="index")
```

空の辞書を作ってあげて，そこに`key=counter`として`pandas.Series`を追加していく．最後に[`pandas.DataFrame.from_dict`](https://pandas.pydata.org/pandas-docs/stable/generated/pandas.DataFrame.from_dict.html)によって辞書から`DataFrame`を作る．
動かしてみたらわかるがこちらのほうがだいぶ速い．

!!! Tip
    今回は`df_add`を`Series`にするために`iteritems()`を使っている．

### 参考リンク
- https://stackoverflow.com/questions/27929472/improve-row-append-performance-on-pandas-dataframes
- https://aniharu.hatenablog.com/entry/2017/12/17/172753