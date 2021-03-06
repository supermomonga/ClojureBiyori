１日目 S式の誘惑
================

　やあ、ようこそ。この文章を読んでいるということは、たぶん何らかの意味で、Clojureに興味を持っているからだろう。

　Clojureという言語は、一般的にはLispの方言の一つだと言われている。Lispを特徴付けているのは、S式と呼ばれている表現だ。とはいえ、急に「S式」とか言われても、「はて、どういうこっちゃ？」となるだろう。詳しく、そして厳密な定義はともかくとして、そのS式というやつに触れてみたいと思う。

1.0 なぜS式が重要なの？
----------------------

(準備中)

1.1 はじまりのClojure (Hello, Clojure!)
---------------------------------------

　まず最初は、プログラミングの慣習にのっとって、`Hello, Clojure!`を出力してみよう。

```clojure
(println "Hello, Clojure!")
```

　まず、基礎的なこととして、引用符で囲まれたのを文字列と呼ぶ。文字列は、今見ているものだ。そして、引用符で囲まれてないものをシンボルと呼ぶ。その理由はおいおい説明するけど、こいつらは、機械にとっては「何をするべきなのか」というものを表している。

　さて、これを実行すると、確かに画面に`Hello, Clojure!`と表示される。そして、このカッコで囲まれたものこそ、S式というヤツに他ならない。

　もう少し噛み砕いてみよう。S式の一部は(Clojureの場合はそうだが)、下のような構造を取る。

```clojure
(何かしらの処理 処理するべきものの一群)
```

つまり、上の`(println "Hello, Clojure!")`は、`"Hello, Clojure!"を一行で表示しろ`と読み替えることができる。他にも足し算をしたければ、したのようになる。

```clojure
(+ 1 2)
```

　これも同じように、`1と2を足せ！`という風に読み替えることができる。

　もちろん、このS式は一つだけであるならば、かなり非力なものである。しかし、組み合わせてみるとどうだろう？

```clojure
(println (+ 1 (+ 2 3)))
```

　これを実行したときに、目の前には6という文字が一行に表示されているはずだ。この処理を追いかけると、下のようになる。

```
(((2 と 3 を足せ) と 1を足せ）を 表示しろ！)
```

　とはいえ、このままだと正直なにをやっているのかわからない。そこで、一つ、ClojureにおけるS式についての決まりごとを書いておこう。

* ClojureにおけるS式は、それが実行されたときに、その実行されたあとの値になる。

　なんのことだ、と思うかもしれないけど、よく考えると当たり前のことなのだ。どういうことかというと、僕たちが`1 + 3`という式を計算した時、`4`という数字を期待する。でもって、ノートにこういう風に書く。

```
1 + 3 = 4
```

　このような小学生のさんすうノートみたいな記述だけど、馬鹿にはできない。これと同じようなことが、実は上でも起こっている。

```
(2 と 3 を足せ) = 5
((2 と 3 を足せ) と 1を足せ) = (5 と 1 を足せ) = 6 
(((2 と 3 を足せ) と 1を足せ）を 表示しろ！) = ((5 と 1 を足せ) を表示しろ) = (6を表示しろ)
```

　このように、S式はぱたぱたと読み替えられて、最後の命令まで届けられ、そして最終結果を出す。最終結果はとくに受けとる相手がなければ、捨てられる。

　Clojure(もしかしたら他のLisp?)において、`()`というのは「やれ！」ということだ。そして、やったあとは、その結果になる。とすると、「やれ！」という前の命令みたいなのがあるということにもなる。その通りだ。例えば、下のコマンドを入力してみよう。

```clojure
(println println)
```

　すると、したのような結果が出てくる。

```
#<core$println clojure.core$println@(ここは任意の文字列)>
```

　こいつが、「やれ！」という前の「命令」の実体になる。小難しいことを言うと、「関数オブジェクト」とか、その辺のものになるんだろうけど、おいておく。ポイントは、このような命令は、基本的には先頭に置かれて始めて、行動するってことだ。

　「基本的には」ということは、例外もあるということなんだけど、あとで説明したいと思う。

1.2 Do it your function!!(defn)
--------------------------------

　さて、最初に見た「println」というのは、最初からある関数なので、「ビルドイン関数」と呼ばれる。とはいえ、これを組み合わせているだけでは辛い。きっと君のプログラミングは、地獄のような「カッコモンスター」のような見た目になるはずだ。

　だから、まず最初に関数の作り方を学ぶ必要がある。

　最初から用意されている命令ではなく、自分で命令を追加したい場合、`defn`というのを使う。例えば、任意の数を二倍にする命令を定義してみよう。これは下のように書ける。

```clojure
(defn mul-two [x] (* x 2))
(mul-two 2) ;; => 4
```

> **Lisp散策** さて、Lispを知っている人であるならば、基本的にLispとは`()`を使う言語ではないのか、というのを
> 疑問に思う筈だ。例えば、Common Lispであるならば、上の`mul-two`は`(defun mul-two (x) (* x 2))`と書ける。
>
> Clojureが、他のLisp族と比較しての特徴をあげるならば、Clojureはある程度*一貫性を崩す*ことで、取っ付きやすさ
> を作っているように感じる。逆に、他のLispは、どちらかといえば全てを「リストで扱える」という一貫性のほうが
> 重要だろう。

　これはどういうことをしているんだろう？これは、たぶん下のように書ける。

```
(mul-twoという関数を定義する。
　[こいつは一つデータを取る。そのデータの名前をxとする]
  (xに２倍する))
```

　普通、プログラミングの世界では、関数に入ってくる要素のことを「引数」とよぶ。先ほどのソースコード、つまり`(mul-two 2)`の`2`は引数だ。そして、この`2`が、`x`と名付けられる。

　もちろん、引数は1つ、２つ、3つ……いくらでも取ることができる。例えば、二つの文字列を"and"で結びつける関数を定義したければ、下のように書ける。

```clojure
(defn and-string [prev next] (str prev " and " next))
(and-string "Ricky" "Esehara") ;; => "Ricky and Esehara"
```
　strという関数は、文字列を作る関数だ。これで好きな関数を作ることが出来るようになった。

> ** ヒント: 慣れるまでは細かく宣言を **
>
> 『On Lisp』(ポールグラハム, オーム社, 2007)の本では、頻繁に関数をテストすることを推奨している。
> そのサイクルは、関数一つ、いやそれ以下になるだろう。
> 自分の場合もそうだったけれども、既に関数型以外の考え方になれてしまった人は、ある関数の中で、処理を一気に書き上げてしまおうとする。
> だけれども、慣れるまでは、ある処理を構成するために、もっと細かい処理に分割し、そしてそれに名前を付けながら宣言していくといいかもしれない。

1.3 そいつに名前を(def)
-----------------------

　しかし、ちょっと疑問が残る。例えば"Esehara"という文字列は"my-name"と名付けることができるなら、単なる文字列であったり、数字でやるより、はるかにその意図がわかりやすい。もちろん、下のようにも出来る。

```clojure
(defn and-string [prev next] (str prev " and " next))
(defn my-name [] "Esehara")
(and-string "Ricky" (my-name)) ;; => "Ricky and Esehara" 
```

　でも、わざわざカッコをつけて取り出すのは面倒くさい。理想をいうなら、下のように書きたい。

```clojure
(and-string "Ricky" my-name)
```

　こういうものを宣言するものがある。それは`def`だ。

```clojure
(def my-name "esehara")
(println my-name) ;; => output: esehara
```

　このように、名前をつけることが出来る。もちろん、こういう書き方もできる。

```clojure
(def my-name "esehara")
(def and-string-result (and-string "Ricky" my-name))
(println and-string-result) ;; output: Ricky and esehara
```

　こいつはS式の結果に対しても名前を付けることが出来る。これらの名前のこと、つまり上記でいうなら
`my-name`や`and-string`、また`println`もそうだが、これらに関しては`シンボル`と呼ばれる。

　`シンボル`は、恰も値か、あるいは関数のように振る舞うことが出来る。

> **Lisp散策** 他のLispだと、これらの一つ一つのことを`atom`と呼ぶことがある。しかし、
> `atom`という名称は、Clojureの場合、別の文脈で使われる。

　もう一つ重要なことを付け加えておくと、この`シンボル`もまた`S式`だ。

1.4 順番に気をつけて！
---------------------

　ただし、気をつけなきゃいけないことがある。実は当たり前なんだけど、前の行で宣言されたものしか、次では使えない。だから、例えば

```clojure
(defn and-string [prev next] (str prev " and " next))
```

　という関数を作った時に、「あ、そうだ、この" and "という文字列に名前を付けよう！」と思ったとして

```clojure
(defn and-string [prev next] (str prev center next))
(def center " and ")
```

　とかやっちゃうと、「centerなんてまだ定義されてないぞ！」と言われてしまう。例えばこういうの。

```
Exception in thread "main" java.lang.Exception: Unable to resolve symbol: center in this context (NO_SOURCE_FILE:3)
```

　プログラミングの世界では、これを「エラーメッセージ」と呼ぶ。なんでエラーかというと、僕たちが間違えたからだ。というより、プログラムの世界は、その言語が「間違いだ！」と思えば、それは間違いだ（もちろん、そもそもおかしいこともあるんだけど）。

> **エラーログはしっかり読もう**
>
> 著者もよく経験することだが、大抵の問題は、エラーログを読むことによって、解決することが多い。
> 事実、人が「このコード、上手く動かないんだけど」と相談されるとき、大抵エラーログにその解決が
> 載っていることが多い。エラーログをちゃんと読む癖はしっかりとつけておこう。

　なんかjava.langがどうのこうの、といっているが、ClojureはJavaという言語が動く、JVMというバーチャルなマシンで動いている。だから、Javaがエラーメッセージを出す。

　僕たちの文章でも、急に定義されていない言葉が出てくると戸惑う。だから、まず始めに定義をしてあげないといけない。なぜなら、基本的には、カッコの中のシンボルは逐一「お前は何者なんだ」とJavaに問われるからだ。この場合は、「あっしは何者でもないんでっせ」と答えられたというわけだ。

　意識しておかなければならないのは、宣言の順番が大切だということだ。何らかの名前は、実行途中の何処かで宣言されておかなければならない。

1.5 とりあえず、私が何だっていいじゃない？ (quote)
--------------------------------------------------

　とはいえ、これはこれで面倒くさいこともある。というのも、「これからおきゃくさんが家にくるわよ」と言われて、「そのおきゃくさんの名前は誰だ」と問われても、「いや……そんなのわかんないし」となるだろう。

```clojure
(defn comming [] (println (str "Comming" my-guest))) ;; error
(def my-guest "esehara")
(comming)
```

　このコードは、単純に順番を入れ替えればいいだけなんだけど、ちょっとズルをして、「まだ`my-guest`が何なのか効かないで」ということは出来るのだろうか。実は、できる。`'`という符号を使う。

```clojure
(defn comming [] (println (str "Comming" 'my-guest)))
```

　これは、要するに「シンボル自体を指し示せ！」ということになる。だから

```clojure
(type 'not-define-value)
```

　とすると、`clojure.lang.Symbol`という言葉が出てくるだろう。`type`は、「お前はどういうヤツなんだ」ということを調べるのに有効だ。例えば、次のを試してみよう。

```clojure
(type "esehara")
```

　とすると、`java.lang.String`と出てくるはずだ。

　さて、この`'`とかいうやつはパワフルだ。なんだって「とりあえず、それ聞かないでくれる？」と言うことができるからだ。例えば

```clojure
(2 + 1)
```

　とか間違えてやると、エラーになる。当然で、"2"という数字は、関数としては存在しないからだ。とはいえ

```clojure
'(2 + 1)
```

　だとエラーにはならない。「まあそういうもんだろう」と思ってスルーしてくれるわけだ。

　しかし、これだけだと困ったことになる。確かに評価をそのままにしてくれるのはあり難いけれども、このままだとずーっとそのままだ。実際

```clojure
(defn comming [] (println (str "Comming " 'my-guest))) 
(def my-guest "esehara")
(comming)
```

　とやると`Comming my-guest`と出力されるはずだ。目的は"esehara"という名前を取り出すことで、`my-guest`という仮の名前が欲しいわけじゃない。そこで、下の関数を使うといいだろう。

```clojure
(def my-guest "esehara")
(eval 'my-guest)
```

　`eval`は、要するに「ちょっと待って」と後回しにしたことを、もう一度「お前はなんだ！」と問うことが出来る。従って、上の場合、`'my-guest`は`"esehara"`になる。（そして、このことをプログラミング言語では「評価」と呼ぶ）

　従って、下のように書き直すことができる。

```clojure
(defn comming [] (println (str "Comming " (eval 'my-guest)))) 
(def my-guest "esehara")
(comming)
```

　他の言語とは違い、`eval`は、データ構造を読み取って、その結果を返すだけで、テキストを評価しない。

　当然のことながら、この`quote`を人為的に作り上げる方法もある。実は、`quote`関数というものが存在している。だから、

```clojure
(= '(1 + 2) (quote (1 + 2))) 
```

　は、Trueになる。

1.6 あっちにいったり、こっちにいったり(値のキャスト)
---------------------------------------------------

　さて、説明が幾分か後回しになってしまったけれども、Clojureは、数字、文字列、シンボル、それぞれ違うものだと考える。例えば、次の式を評価してみよう。

```clojure
(+ 1 "1")
```

　すると

```
ClassCastException java.lang.String cannot be cast to java.lang.Number  clojure.lang.Numbers.add (Numbers.java:126)
```

　と、出る。これは前にもいったとおりエラーメッセージだ。何か言ってるぞ。キャスト？なんだそれ。

　キャストとは、要するに文字にしたり、数字にしたり、あるいはシンボルにしたりすることだ。僕たちは、あるときは文字列にしたいし、あるときは数字にしたいと思う筈だ。しかし、それが出来なかった！困ったと言っているわけだ。

　だから、Clojureを困らせないように、数字なのか？文字列なのか？それともシンボルなのか？ということを考えないといけない。

　あとからしっかり教えるけれども、上のエラーを解決してみよう。そのために`Integer/parseInt`というのをつかってみよう。

```clojure
(+ 1 (Integer/parseInt 1)) ;; => 2
```

　これはつまり、「この文字列を数字にかえて」ということを伝えたのだ。なもんだから、数字になりそうもないやつはエラーを出すことになる

```clojure
(+ 1 (Integer/parseInt "no Integer"))
```

　というのを試すと

```
NumberFormatException For input string: "no Integer"  java.lang.NumberFormatException.forInputString (NumberFormatException.java:65)
```

　「これ数字に見えるのか？ああん？」という感じのエラーをはいている。もうちょっとちゃんとしたいいかたをすると、「数字の形式ではないね」と述べているわけだ。

1.7 型ありにかたなし！
---------------------

　さて、プログラミング言語の場合、大抵は「型」というものが存在している。型が存在している理由と言うのは、色々と説明が出来る。例えば、整数であるならば、それは「数を増やしたり、減らしたりできるもの」として、機械は捉えることができる。

　Clojureにも、当然型というものが存在している。例えば、`1`や`100`や`9999999`というものは、ClojureではまずLong型として取り扱われる。そして`"ほげほげ"`とか`"Clojure"`とか、ダブル引用符で囲まれたものはString型と呼ばれる。

　さて、これらの型を調べるのに、Clojureはそれ用の関数を用意している。あるデータが数字かどうかを調べるためにはは`number?`を使い、その型が文字列かどうかを調べるためには`string?`を使う。

```clojure
(number? 100) ;; true
(number? "100") ;; false
(string? 100) ;; false
(string? "100") ;; true
```

　もしかしたら、`1`と`"1"`について、どちらも数字のように感じるかもしれない。そういう仕様の言語もあるだが、Clojureの中では区別する。

　また、`number?`とか`my-define-function`とか、こういうものに関してはSymbolと呼ばれる。

```clojure
(symbol? 'number?) ;;true
```

1.8 名前重要
------------

　さて、ここまで読んでみて、上記の変数について、ある命名の規則があることに気がついた人もいるかもしれない。

　Clojureも一応はLispの席を与えられる存在である。そして、Lispが脈々と暗黙のうちに持っている命名規則を、その内部に持っている。もちろん、これらは守る必要がないものではあるものの、しかし同じClojurenを苛立たせないためには、命名規則を守った方が、あとから固くて丸いもので、後ろからパカンとやられないためにも重要である。

　まず最初にわかりやすいのが、関数の後ろにつく`?`だ。上でも`number?`や`symbol?`が出てきた。これらは`true`か`false`が返ってくるものである。これは端的に質問のことを指している。

　次に、何らかの「副作用」を与える関数は、`!`を一番後ろにつける。

　関数型では、「副作用」という言葉がいやというほど出てくる。たぶん、日常生活では「薬の副作用が云々」ということで使われる言葉だと思う。この「副作用」という言葉は、例えば変数の値を暗黙に変えたり、あるいはある関数の挙動が変わる可能性があるときに、「副作用がある」とされる。このあたりの詳しい解説については、晴れの日に説明を行う。

　Clojureのような言語は、ある引数を関数が取ると、同じ結果を返すことを大切にする。しかし、大切にするとはいえ、現実は理想との妥協と折合わせである。何処かで副作用をもたなければいけないときがある。だから、そういうときは、そういう風に知らせる必要があるということだ。

1.9 その式を無効に
-----------------

　基本的に、私たちはプログラミングをしているのだから、そのコードの意味について、コード自体で表記したほうがいいことは確実である。しかし、現実はそうではない。このような文章でもそうだけれど、日本語で書いたほうが十分意図が伝わる場合も多い。そういうとき、「ここは、別にプログラミングのコードではないですから、実行しなくて結構ですよ」ということを教えてあげなくてはならない。その場合、`;;`を使う。

```clojure
;; これはコメントです
;; これもコメントです
(defn foo ;;コメントは途中に挟むことが可能です
      [bar ;; できます
       ] (+ bar 1)) 
```

　このように、間にはさまれたコメントは、全て「意味のないもの」としてClojureでは処理される。しかし、この`;;`は、その開始から行末までを全て無効にする。そこで、任意の部分をコメントに出来るような式があると良い。実はその手のものが用意されている。`comment`だ。

```clojure
(comment この部分はコメントです)
(comment これもコメントです)
```

　`comment`の使い道については、複数行にまたがる式を一時的に無効にしたい場合に有効だろう。ただし、`;;`と違うのは、`comment`はあくまでも式の一部であり、その内部の式を「無いもの」として置き換えているだけなので、`comment`以下は式として成立するようにしてあげないといけない。

> **最も簡単なマクロ**
>
> のちにマクロについて書くことになるが、`comment`は一番簡単なマクロと言うことも可能だ。
> 実際に、`comment`の定義は`(defmacro comment [& body])`だけで行える。
> つまり、`comment`マクロの正体は「渡される式自体を何も無いものとして置き換える」マクロなのだ。
