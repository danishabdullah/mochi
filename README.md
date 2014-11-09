Mochi
====

Mochiは動的型付けの関数型言語及び、そのインタープリタです。


## 概要
Mochiは動的型付けの関数型言語です。
インタープリタはPython3で記述されています。

インタープリンタは、Mochi言語で書かれたコードを
PythonのAST/バイトコードに変換し、Python仮想マシン上で実行します。


## 特徴
- Pythonぽい構文
- Pythonモジュールを簡単に利用可
- 動作環境のPython（CPython or PyPy）と同等の実行速度
- 末尾再帰最適化（自己末尾再帰のみ）あり、ループ構文なし（内包表記はある）
- 関数定義中のローカル変数への再代入不可、関数からグローバル変数の変更（再代入）不可
- 基本のデータ型は永続データ構造（Pyrsistentを利用）
- パターンマッチ/代数的データ型みたいなもの
- パイプライン演算子
- 無名関数定義の糖衣構文
- Python3のitertools、functools、operatorモジュールの関数、itertoolsレシピ相当の関数が組み込み


## 依存モジュール
- CPython >= 3.2 or PyPy >= 3.2.1 
- rply >= 0.7.2
- pyrsistent >= 0.6.2


## インストール

```sh
$ pip install git+https://github.com/i2y/mochi.git
```


## 使い方

### REPL
```sh
$ mochi
>>> 
```

### ファイルを読んで実行
```sh
$ cat kinako.mochi
print('kinako') 
$ mochi kinako.mochi
kinako
$
```

### ファイルをバイトコンパイル
```sh
$ mochi -c kinako.mochi > kinako.mochic
```

### 予めバイトコンパイルしたものを実行
```sh
$ mochi -e kinako.mochic
kinako
$
```

## 簡単な例


### 永続データ構造
```python

[1, 2, 3]
# => pvector([1, 2, 3])

v(1, 2, 3)
# => pvector([1, 2, 3])

{'x': 100, 'y': 200}
# => pmap({'y': 200, 'x': 100})

m(x=100, y=200)
# => pmap({'y': 200, 'x': 100})

s(1, 2, 3)
# => pset([1, 2, 3])

b(1, 2, 3)
# => pbag([1, 2, 3])

# その他の永続データ構造関連の例は、Pyrsistenceのドキュメントを参照してください。
```

### 関数定義
```
def hoge(x):
    hoge + str(x)
     
print(hoge(3))
# hoge3を表示
```

### パターンマッチ
```python
# 注意：matchはスコープを作りません。
#      これはPythonやMochiのif文と同様です。
lis = [1, 2, 3]
match lis:
    [1, 2, x]: x
    _: None
# => 3

match lis:
    [1, &rest]: rest
    _: None

# => pvector([2, 3])
```

### 代数的データ型
```python
data Point:
    Point2D(x, y)
    Point3D(x, y, z)
    
p1 = Point2D(x=1, y=2)
# => Point2D(x=1, y=2)
p2 = Point2D(3, 4)
# => Point2D(x=3, y=4)
```

### パターンマッチ関数定義
```python
data Point:
    Point2D(x, y)
    Point3D(x, y, z)

defm offset:
    [Point2D(x1, y1), Point2D(x2, y2)]:
        Point2D([x1 + x2, y1 + y2])
    [Point3D(x1, y1, z1), Point3D(x2, y2, z2)]:
        Point3D([x1 + x2, y1 + y2, z1 + z2])
    _: None

offset(Point2D(1, 2), Point2D(3, 4))
# => Point2D(x=3, y=4)
offset(Point3D(1, 2, 3), Point3D(4, 5, 6))
# => Point3D(x=5, y=7, z=9)
```

### パイプライン演算子（fizzbuzz)
[こちらのページ](http://wllwmiilmmw.tumblr.com/post/72971456714/yeti-lang)に記載のコードを参考に、
Mochiでfizzbuzz。
```
def fizzbuzz(n):
    match [n % 3, n % 5]:
        [0, 0]: "fizzbuzz"
        [0, _]: "fizz"
        [_, 0]: "buzz"
         _: n

range(1, 31) |> map(fizzbuzz) |> pvector() |> print()
# => pvector([1, 2, 'fizz', 4, 'buzz', 'fizz', 7, 8, 'fizz', 'buzz', 11, 'fizz', 13, 14, 'fizzbuzz', 16, 17, 'fizz', 19, 'buzz', 'fizz', 22, 23, 'fizz', 'buzz', 26, 'fizz', 28, 29, 'fizzbuzz'])
# python3ではmapはイテレータを返すため、そのままprintするとイテレータオブジェクト自体を表示するので、
# print前にpvectorでpesrsistent vectorに変換しています。


### 無名関数
```python
# アロー式。CoffeeScriptに類似。
add = (x, y) -> x + y
add(1, 2)
# => 3

add = -> $1 + $2
add(1, 2)
# => 3

foo = (x, y) ->
    if x == 0:
        y
    else:
        x

foo(1, 2)
# => 1

foo(0, 2)
# => 2

pvector(map(-> $1 * 2, [1, 2, 3]))
# => pvector([2, 4, 6])
```


## TODO
- 説明の追加
- 構文解析の改善
- クラス定義のサポート？

## Licence
MIT Licence

## Author

[i2y](https://github.com/i2y)