4日目 再帰ック道場
=================

　おそらく、最近のプログラム言語であるならば、「再帰」が出来る筈だ。「再帰」というのは、「ある関数が、自らの関数を呼び出すこと」だ。Clojureのような言語がパワフルである理由の一つに、再帰的なプログラムが瞬時に書けることが挙げられるだろう。とはいえ、急にそんなことを言われても「なにいってるんだこいつ」という感じにしかならない。なので、再帰について、この日は学ぶ。

1. Dungeon Crawler !!
---------------------

　そこで、今回は例題として、洞窟の中を検索するプログラムを作ろうと思う。洞窟は、部屋と通路がある。通路は一つ以上の通路があり、次の部屋に続いている。そして、行き止まりの部屋の何処かに、宝物が隠されている。ざっと地図にすると、こういう感じだ。

```
A -> B -> C
       -> D -> E -> H
            -> F
            -> G
```

　さて、この場合に、宝物が`H`と`F`にあったとして、そこまでの経路を出力したいとする。こういうときに、「再帰」というのはとても力を発揮する。

2. 始まりの洞窟
---------------

　とはいえ、何事も一ずつ積み重ねていくことが大切だ。従って、まず最初に単純な洞窟を考えてみよう。単純な洞窟とは、即ち

```
A -> B
```

　という一直線の洞窟だ。バカにしているかもしれないが、こういう単純なところから始めたいと思う。

　まず最初に、上の洞窟をリストで表現してみよう。この場合、一直線だから

```clojure
'(A B)
```

　として表現できる。これを利用しやすいように、`dungeon`に結びつけておく。

```clojure
(def dungeon '(A B))
```

　普通、Lispだと`car`と`cdr`という記号を使うが、Clojureは人間に優しく`first`と`next`を使う。実は`rest`もあるのだが、今回は`rest`は使わない。

> **リストの残りを取り出す二つの方法**
>
> `rest`と`next`の違いが気になる人は、`(next '(will-return-nil))`といと、
> `(rest '(will-return-list))`を入力して比較してみるといいだろう。
> `next`の場合は、`nil`が帰ってきて、`rest`は`()`が返ってくるはずだ。

　さて、まず最初に探索の終了の定義を決める。上の、抽象化したダンジョンの場合、リストの行き止まりは、その先に要素が無いことだと言える。次の要素が取り出せない場合、その部屋の要素を返そう。そして、次が存在してるときは、次に進めるようにしてみよう。即ち、次のような関数が作れるはずだ。

```clojure
(defn tresure-hunt [dungeon]
  (if (nil? (next dungeon))
     (first dungeon)
     (tresure-hunt (next dungeon))))
```

　さて、これを定義して、次のように実行してみよう。

```clojure
(tresure-hunt dungeon) ;;B
```

　無事、行き止まりである`B`が帰ってきたが、しかしこれでは何をしているのかさっぱりわからない。なので、この関数の行動をトレースするために、上の関数を少し書き直してみよう。

```clojure
(defn tresure-hunt [dungeon]
  (if (nil? (next dungeon))
     (first dungeon)
     (do (println (first dungeon) 
         (tresure-hunt (next dungeon))))))
```

　とすると、`A B`と出力されたはずである。Clojureは、次のように「再帰」を実行していった、と考えることができる。

1. まず最初に、`next`の関数を'(A B)へ適応する
2. `next`で'(B)が取り出せたので、`tresure-hunt`関数に'(B)渡す
3. `next`の関数を'(B)を適応する
4. `next`で`nil`が返ってきたのでBを返す
5. 結果、`tresure-hunt`関数の結果は`B`となる

　とはいえ、これは単純にループして、最後の要素になるまで調べているのとかわらないだろう。いや、むしろこのような再帰は、むしろそのような愚直なループに比べれば、コストがかかる可能性が高い。だとすると、再帰はむしろそういうループの貧弱なバージョンだと言われる恐れがある。

3. 旅人の洞窟
-------------

　再帰はパワフルなものではなかったのか？実は著者が嘘をついているだけなのではないか？そこで、次に分岐した洞窟を検索してみよう。

```
A -> B -> C
       -> D -> E
```

　さて、これをリストに直してみる。このとき、分岐する場合は、新しいリストを作るようにしてみよう。つまり

```clojure
'(A B ((C) (D E)))
```

　とする。さて、これを`dungeon`に結びつけて、もう一度`tresure-hunt`してみよう。

```clojure
(def dungeon '(A B ((C) (D E))))
(tresure-hunt dungeon)
```

　さて、この結果は`((C) (D E))`になったはずだ。`tresure-hunt`は、どっちの通路に行けばいいかわからないからだ。

　もしかしたら、これを見たときに、愚直に`(C)`を調べて、次に`(D E)`を調べるようなことを考えたかもしれない。それでもいいんだけれど、再帰が再帰として強力にしている点として、いざとなったら自らを分裂させることができる点にある。

　そこで、とりあえずダンジョンを、二つの入り口があるものとしてセットしよう。

```clojure
(def dungeon '((C) (D E)))
```

　問題は、この二つの入り口をどのようにして調べるかだ。そこで、`map`を使ってみよう。

```clojure
(map tresure-hunt '((C) (D E))) ;; C E
```

　無事、行き止まりを取り出してくれる。さて、これを`tresure-hunt`関数にどのように取り入れるか、だ。

　そこで、リストの中身が全てリストである場合は、mapを発行し、全ての通路を調べるようにしてみてはどうだろうか

　全てが何らかの値であるかどうかを調べる場合、`every?`を使う。これは`filter`と同じく、第二引数に対して真偽を調べるための関数を取る。

```clojure
(defn tresure-hunt [dungeon]
  (cond
     (every? list? dungeon) (map tresure-hunt dungeon)
     (nil? (next dungeon)) (first dungeon)
     :else (tresure-hunt (next dungeon))))
```

　これで、ちゃんと上の簡略した分岐で値が取れるかを調べてみよう。

```clojure
(tresure-hunt '((C) (D E))) ;; C E
```

　OKだ。最後に、全体のマップで、ちゃんと全ての通路の行き止まりが出てくるかどうかを調べよう。

```clojure
(def dungeon '(A B ((C) (D E))))
(tresure-hunt dungeon) ;; ((C E))
```

　成功だ。

4. 扉の洞窟
-----------

　さて、再帰が使えるのは、何もこんな深く潜れる洞窟だけじゃない。例えば扉の洞窟みたいなものを考えてみよう。

　扉の洞窟では、たくさんの扉がならんでいる。目の前には、宝が入った扉の「数」が書かれている。どの扉がどの数に対応しているかわからないが、少なくとも扉は数の小さいものから大きなものの順に並んでいることと、扉を開くと、その扉の数がわかることだけは知っている。注意しなければならないのは、宝は扉を開ける事にどんどん少なくなっていくということだ。だから、出来るだけ小さな試行回数によって、宝が入っている数の扉を開かなくてはならない。ちなみに、重複している数は存在しない。

　ちょっとわかりづらいかもしれないが、これもリストだ。わかりやすいように、あらかじめ、数字のリストはわかっているものとして、テストデータを書いてみよう。ざっとこんな感じである。

```
(1 4 9 10 20 33 62 79 90 100)
```

　さて、これらの扉の数がもし「わからなかった」として、最短で宝を見つけ出すプログラムを書いてみよう。このとき、二分探索というものがつかえる。二分探索の概要は簡単だ。例えば、上のリストから「9」と書かれた扉を見つけたいとしよう。そこで、まず仮に真ん中で扉をわけてみるとする。

```
(1 4 9 10 20)
(33 62 79 90 100)
```

　そこで、二番目の組である先頭の数字を開いてみる。すると、33だとわかる。既に数字は順番に並んでいる筈であり、33は一番小さい数字だ。なので、下の組には入っていない。入っているとするなら上の組だろう。従って、「9」は、先頭から数えて1から5番目の、何処かに並んでいることになる。

　さて、同じようにまた半分にわけてみよう。

```
(1 4)
(9 10 20)
```

　すると、下の一番下を取り出すと、`9`が取り出せた。この場合、3番目だから無事に終了だ。

　このように、既に順序よく並んでいる番号を、半分ずつ取り出す検索方法のことを、二分探索とよぶ。この二分検索を再帰によって定義しよう。

　まず最初に、リストの半分を取り出す処理を行おう。リストの要素数を調べるのは`count`だ。そして、あるリストから、要素分だけ先頭から取り出す関数は`take`だ。逆に、その要素数を先頭から消す関数は`drop`だ。そして、リストの先頭を取り出してくるものは`first`という関数だ。

　さあ、これらを組み合わせて、まず渡されたリストの先頭が、ある数字より大きいか小さいかを調べてみよう。

```clojure
(defn first-is-big?
  [checknum uselist] (< checknum (first uselist)))
```

　そして、あるリストの半分を取り出す関数も定義しよう。

```clojure
(defn count-half-list
  [uselist] (/ (count uselist) 2))

(defn drop-half-list
  [uselist] (drop (count-half-list uselist) uselist)) 

(defn take-half-list
  [uselist] (take (count-half-list uselist) uselist))
```

　まずは再帰させずにテストしてみよう。

```clojure
(defn binary-search
  [get-num uselist now-num]
  (cond (= (first (drop-half-list uselist)) get-num) (+ now-num (count-half-list uselist))
        (first-is-big? get-num (drop-half-list uselist)) "Yes, it is big!"
        :else "No, it is small!"))
```

　そして、テストデータをセットし、チェックする。

```clojure
(def door-dungeon '(1 4 9 10 20 33 62 79 90 100))
(binary-search 9 door-dungeon 0) ;; "Yes, it is big!"
(binary-search 33 door-dungeon 0) ;; 5
```

　実は、既にあるリストから任意の番号のリストを取り出すための関数が用意されている。それが`nth`だ。ちゃんと`33`が5番目にあることを確認しよう。

```clojure
(nth door-dungeon 5) ;; => 5
```

　さて、今度は再帰させてみよう。

```clojure
(defn binary-search
  [get-num uselist now-num]
  (cond (= (first (drop-half-list uselist)) get-num) (+ now-num (count-half-list uselist))
        (first-is-big? get-num (drop-half-list uselist))
          (binary-search get-num (take-half-list uselist) now-num)
        :else
          (binary-search get-num (drop-half-list uselist) (+ now-num (count-half-list uselist)))))
```

　しかし、意図に反して、以下のような変な数字が返ってくるだろう。

```clojure
(binary-search 9 door-dungeon 0) :: 3/2
```

　これをfixするために、結果を返す最初のチェックに、`int`を付けよう。

```clojure

(defn count-half-list
  [uselist] (int (/ (count uselist) 2)))

(defn binary-search
  [get-num uselist now-num]
  (cond (= (first (drop-half-list uselist)) get-num)
          (+ now-num (count-half-list uselist))
        (first-is-big? get-num (drop-half-list uselist))
          (binary-search get-num (take-half-list uselist) now-num)
        :else
          (binary-search get-num (drop-half-list uselist) (+ now-num (count-half-list uselist)))))
```

　さあ、この関数をテストしてみよう。

```clojure
(binary-search 1 door-dungeon 0) ;; => 0
(binary-search 100 door-dungeon 0) ;; => 9
(binary-search 9 door-dungeon 0) ;; => 2
```

　上手く定義できているなら、ちゃんと場所が取り出せているだろう。

　このように、再帰が素敵なのは、個々のパーツと、終了条件、それとそれに伴うデータの取扱いさえきちんとしていれば、自ずと自然な解決を導きだしてくれる点にある。もちろん、このような解決方法は、再帰を用いずとも出来る。

　さて、この関数には、非常に厄介な欠点が存在する。そこで、存在しない要素を調べようとしてみよう。

```
(binary-search 15 door-dungeon 0)
=> StackOverflowError   clojure.lang.LazySeq.seq (LazySeq.java:60)
```

　この`StackOverflowError`は、今回の場合は「再帰させすぎて、解決できない」というClojureの悲鳴だ。実は、`cond`の前に`println`を挿入しみればわかるのだが、リストを`(10)`にまで絞り込んだあと、延々と再帰し続けている！つまり、問題は、リストが一つまでに絞り込まれたとき、その要素がそれ自体でない場合は、その旨のメッセージを送らなければならない。

```clojure
(defn binary-search
  [get-num uselist now-num]
  (cond (and (= (count uselist) 1)
             (not (= (first uselist) get-num)))
          (str "Not Found:" get-num)
        (= (first (drop-half-list uselist)) get-num)
          (+ now-num (count-half-list uselist))
        (first-is-big? get-num (drop-half-list uselist))
          (binary-search get-num (take-half-list uselist) now-num)
        :else
          (binary-search get-num (drop-half-list uselist) (+ now-num (count-half-list uselist)))))
```

　これで、ちゃんと存在しない要素を調べようとしたときには、その要素が存在しない旨をメッセージにすることが出来た。

　この二分検索は、まだ十分に完成されているとはいいがたい（例えば、空リストが入ってきたときの挙動はどうだ？)。とはいえ、再帰なるものを理解するのにはこれで十分だろう。

5. パンくずの洞窟
----------------

　さて、再帰について少しながらに知る事が出来た。確かに、結果を取り出すだけなら、上のような方法で十分である。しかし、僕たちの目的は、経路を調べ、そこから宝のある通路の場所を知りたいということが問題だった。というわけで、`旅人の洞窟`で使ったソースコードを、改めて再掲しよう。

```clojure
(defn tresure-hunt [dungeon]
  (cond
     (every? list? dungeon) (map tresure-hunt dungeon)
     (nil? (next dungeon)) (first dungeon)
     :else (tresure-hunt (next dungeon))))
(def dungeon '(A B ((C) (D E))))
(tresure-hunt dungeon) ;; ((C E))
```

　さて、ここから全ての通路のパターンを取り出してみよう。この場合は、以下の二つの経路が取り出せればいいのだから、気が楽だ。

```
A -> B -> C
A -> B -> D -> E
```

　しかし、今、定義されている`tresure-hunt`は、いままでの通路を保存しておくような場所が無い。恐らく、過程として補完しておくべきところが必要だ。例えば、次のような感じで

```clojure
(defn tresure-hunt [walk dungeon]
  (cond
     (every? list? dungeon)
       (map (fn [next] (tresure-hunt walk next)) dungeon)
     (nil? (next dungeon)) (reverse (conj walk (first dungeon)))
     :else (tresure-hunt (conj walk (first dungeon)) (next dungeon))))    
```

　さて、この関数をテストしてみよう。

```
(def dungeon '(A B ((C) (D E))))
(tresure-hunt '() dungeon)
```

　そうすると、

```
(((A B C) (A B D E)))
```

　という結果が得られたはずだ。

　一つ目の引数を追加したのには理由がある。それは、引数が現在の履歴を補完するための場所として機能させるためだ。しかし、初期値である空リストをいちいち指定するのは正直いって面倒臭いことこの上ないだろう。

　もちろん、関数の定義をわけることは可能だ。例えば次のように。

```clojure
(defn tresure-hunt-start [dungeon]
  (tresure-hunt '() dungeon))

(defn tresure-hunt
  [walk dungeon]
  (cond
     (every? list? dungeon)
       (map (fn [next] (tresure-hunt walk next)) dungeon)
     (nil? (next dungeon)) (reverse (conj walk (first dungeon)))
     :else (tresure-hunt (conj walk (first dungeon)) (next dungeon))))    
```

　しかし、実際はもっとスマートな方法がある。引数によって、関数の振る舞いが変わるように制御が出来る。

```clojure
(defn tresure-hunt
 ([dungeon] (tresure-hunt '() dungeon))
 ([walk dungeon]
  (cond
     (every? list? dungeon)
       (map (fn [next] (tresure-hunt walk next)) dungeon)
     (nil? (next dungeon)) (reverse (conj walk (first dungeon)))
     :else (tresure-hunt (conj walk (first dungeon)) (next dungeon)))))    
```

　二つの関数を用意するよりも、十分にスマートになった。

7. 複雑さと戦う冒険者のために(let)
------------------------------------------

　さて、ここまでで幾つかのダンジョンについて攻略をしてきた。しかし、前回の洞窟において、関数の中で同じような処理をしていることに気がつくだろう。例えば、`(conj walk (first dungeon))`という文句は幾つか出てくる。また、`(next dungeon)`も、同じように出てくる。

　可読性ということを考えた時、このようなリストのまま投げ出していくと、そもそもそのデータが何を表しているのか不明になる。何らかのシンボルに、値を結びつけるのは、人間にとってもっと読みやすいものにするためである。

　まず最初に、`(conj walk (first dungeon))`は、`今まで歩いてきた洞窟の履歴`であるという事が出来るだろう。`(next dungeon)`は文字通り、`残りの通路の情報`と捉える事が出来る。

　そこで、あるリストの中だけで通用するようなシンボルを臨時に定義できるといいだろう。そういう場合、`let`を使うことができる。

```clojure
(defn tresure-hunt
 ([dungeon] (tresure-hunt '() dungeon))
 ([walk dungeon]
  (let [history (conj walk (first dungeon))
        next-dungeon (next dungeon)]
    (cond
       (every? list? dungeon)
         (map (fn [next] (tresure-hunt walk next)) dungeon)
       (nil? next-dungeon) (reverse history)
       :else (tresure-hunt history next-dungeon)))))
```

　この`let`に関しては、下のような形を取る。

```
(let [何らかのシンボル 何らかの値]
  処理の内容)
```

　さて、`let`の中の`[]`においては、やはりペアでないといけない。実際に、奇数で定義しようとするとエラーが出る。
