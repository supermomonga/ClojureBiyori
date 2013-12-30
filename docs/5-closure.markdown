5日目 ClojureでClosureを作ろう、アレ？
=====================================

　さて、Clojureのような、関数をオブジェクトとして扱えるような言語なら、当然のことながら`クロージャ`が使えることが期待されている。そこで、この章では`クロージャ`の作り方、さらには`カリー化`について考えてみよう。

1. これくらいのお弁当箱におにぎりをつめよう
-----------------------------------------------

　今日は、天気もいいので、外で弁当を食べたいと思う。もちろん、出来合いの弁当を買ってもいいかもしれないが、どうせだったら自分でお弁当を作ってみよう。

　お弁当箱のデータは、主食とおかずがセットになっているものと考えよう。

```clojure
(defn bentobox [main-food sub-food]
  (list main-food sub-food))
```

　さて、この何の変哲もない`bentobox`という関数だ。これで弁当を作るためには、`(bentobox 'rice 'karaage)`とすれば、リスト化された`(rice karaage)`が取れる。確かに主食は変更が出来たほうがいいかもしれないが、普段主食として入れるものは`rice`であるから、最初から`rice`がセットされている関数があると良い。つまり

```clojure
(defn bentobox-with-rice [sub-food]
  (bentobox 'rice sub-food))
```

　なるほど、普段は`rice`であっても構わないかもしれないが、あるときは`bread`かもしれない。これらを一つ一つ定義してもいいだろう。しかし、主食は何が出てくるかわからない。将来的にはタイ米とか、タロ芋とか、あるいはナンの可能性がある。それらに関していちいち定義すると、主食n種類の定義が必要になる。それよりもいっそのこと

```clojure
(def current-bentobox (bentobox-with 'rice))
(current-bentobox 'karaage) ;; => (rice karaage)
(current-bentobox 'gyoza) ;; => (rice gyoza)
```

　みたいに定義できれば、そのような心配をしなくてもすむ。そして、そのようなことをするための考え方が`クロージャ`に他ならない。

2. そのときのことを胸に持って
----------------------------

　`クロージャ`とは、教科書的に説明するなら、`その関数が定義されたスコープの各種定義を参照する関数`と述べることが出来る。しかし、このように言っても意味がわからないだろう。そこで、入れ子構造的に`defn`を定義した関数を用意してみよう。

```clojure
(defn set-bentobox! [main-food sub-food]
  (defn bentobox [sub-food] (list main-food sub-food))
  (bentobox sub-food))
```

　さて、ここで`set-bentobox!`を実行してみよう。

```clojure
(set-bentobox! 'rice 'karaage) ;; (rice karaage)
```

　`defn`は、`let`と違い、そこで定義されたものは、どの場所でも参照できるようになってしまう。なので、内部の`bentobox`は、外からでも参照できるようになっている筈だ。そこで、中の`bentobox`を使ってみよう。

```clojure
(bentobox 'gyoza) ;; (rice gyoza)
```

　なんと、最初から`rice`が入った弁当が帰ってきた。というのも、`set-bentobox!`のときに使われた`main-food`の値がそのまま使われるのだ。従って、`(set-bentobox! 'bread 'karaage)`とすると、内部の`bentobox`では、`bread`が使われるのだ！

　とりあえず、`クロージャ`と呼ばれる手法の正体みたいなのは垣間見れたと思う。先ほども言ったように、その関数が定義されたときの、シンボルにセットされた値がそのまま保持される。

　さて、ここで一つ疑問が生じる。もし関数は定義されたときの値を持ってしまうとするならば、その関数自体を投げるとどういう風になるのだろうか。例えば

```clojure
(defn bento-factory [main-food]
  (defn bentobox [sub-food] (list main-food sub-food))
  bentobox)
```

　という、`bentobox`という関数オブジェクトを投げる関数を定義してみよう。これを`bentobox-with-rice`というシンボルに結びつけてみよう。

```clojure
(def bentobox-with-rice (bento-factory 'rice))
(bentobox-with-rice 'karaage)
```

　さて、こんどはもう一つ、`bentobox-with-bread`を作ってみよう。

```clojure
(def bentobox-with-bread (bento-factory 'bread))
(bentobox-with-bread 'kiraage)
```

　問題は、両者の関数の間で、定義中に`main-food`として結びつけられた値が変更されるかどうかだ。さてどうなったかは、実際に調べてみればわかる。

```clojure
(bentobox-with-rice 'gyoza) ;; (rice gyoza)
(bentobox-with-bread 'gyoza) ;; (bread gyoza)
```

　それぞれがそれぞれの値がセットされた状態を保持しているのがわかる。恰もタイムトラベルのように、彼らが定義された状態を持ったままになる。これを利用することで、あとから汎用的に使えるような関数オブジェクトを習得することが出来る。

　さて、上の例は、実はあまりよくない。なぜなら、関数の内部で関数を定義しているからだ。むしろ、返す関数自体は全部一緒なのだから、名前を付けなくてもいいのではないかと考えるかも知れない。その通りだ。ここで、匿名関数を使って書き直してみよう。

```clojure
(defn bento-factory [main-food]
  (fn [sub-food] (list main-food sub-food)))
```

　挙動が本当に同じかどうかを確認してみよう。

```clojure
(def bentobox-with-bread (bento-factory 'bread))
(bentobox-with-bread 'kiraage)`
```

　特殊な事情がない限り、クロージャのアプローチを使う場合は、匿名関数を利用すると便利だろう。

3. カリー化でインド人を納得させろ
--------------------------------
(未完)
