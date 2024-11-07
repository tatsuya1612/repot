+++
title = "Operator addition"
hascode = true
date = Date(2024, 11, 2)
rss = "A short description of the page which would serve as **blurb** in a `RSS` feed; you can use basic markdown here but the whole description string must be a single line (not a multiline string). Like this one for instance. Keep in mind that styling is minimal in RSS so for instance don't expect maths or fancy styling to work; images should be ok though: ![](https://upload.wikimedia.org/wikipedia/en/3/32/Rick_and_Morty_opening_credits.jpeg)"

tags = ["syntax", "code"]
+++

# Juliaに演算子を追加してみた

\toc

今回は、言語処理系のOSSの一つである**JULIA**言語について、ソースコードを手探り、演算子を追加することを目指しました。

あらかじめ`2024-05-julia`ディレクトリに移動しておきましょう。

## 要件

組み合わせの総数を計算する*Combination*演算は、現状
```julia
binomial(4,2)
> 6
```
と、関数として定義されている。

この`binomial(M,N)`を`M~~N`という短縮された**二項演算子**にする。

演算子には優先順位がある。

!!! Example title="演算子の優先順位の代表例"
3+5×2に対して、3+5よりも5×2が先に行われる。
!!!

既存の演算子の優先順位は、

```scheme
(define prec-assignment
  (append! (add-dots '(= += -= −= *= /= //= |\\=| ^= ÷= %= <<= >>= >>>= |\|=| &= ⊻= ≔ ⩴ ≕))
           (add-dots '(~))
           '(:= $=)))
;; comma - higher than assignment outside parentheses, lower when inside
(define prec-pair (add-dots '(=>)))
(define prec-conditional '(?))
(define prec-arrow       (add-dots '(← → ↔ ↚ ↛ ↞ ↠ ↢ ↣ ↦ ↤ ↮ ⇎ ⇍ ⇏ ⇐ ⇒ ⇔ ⇴ ⇶ ⇷ ⇸ ⇹ ⇺ ⇻ ⇼ ⇽ ⇾ ⇿ ⟵ ⟶ ⟷ ⟹ ⟺ ⟻ ⟼ ⟽ ⟾ ⟿ ⤀ ⤁ ⤂ ⤃ ⤄ ⤅ ⤆ ⤇ ⤌ ⤍ ⤎ ⤏ ⤐ ⤑ ⤔ ⤕ ⤖ ⤗ ⤘ ⤝ ⤞ ⤟ ⤠ ⥄ ⥅ ⥆ ⥇ ⥈ ⥊ ⥋ ⥎ ⥐ ⥒ ⥓ ⥖ ⥗ ⥚ ⥛ ⥞ ⥟ ⥢ ⥤ ⥦ ⥧ ⥨ ⥩ ⥪ ⥫ ⥬ ⥭ ⥰ ⧴ ⬱ ⬰ ⬲ ⬳ ⬴ ⬵ ⬶ ⬷ ⬸ ⬹ ⬺ ⬻ ⬼ ⬽ ⬾ ⬿ ⭀ ⭁ ⭂ ⭃ ⥷ ⭄ ⥺ ⭇ ⭈ ⭉ ⭊ ⭋ ⭌ ￩ ￫ ⇜ ⇝ ↜ ↝ ↩ ↪ ↫ ↬ ↼ ↽ ⇀ ⇁ ⇄ ⇆ ⇇ ⇉ ⇋ ⇌ ⇚ ⇛ ⇠ ⇢ ↷ ↶ ↺ ↻ --> <-- <-->)))
(define prec-lazy-or     (add-dots '(|\|\||)))
(define prec-lazy-and    (add-dots '(&&)))
(define prec-comparison
  (append! '(in isa)
           (add-dots '(> < >= ≥ <= ≤ == === ≡ != ≠ !== ≢ ∈ ∉ ∋ ∌ ⊆ ⊈ ⊂ ⊄ ⊊ ∝ ∊ ∍ ∥ ∦ ∷ ∺ ∻ ∽ ∾ ≁ ≃ ≂ ≄ ≅ ≆ ≇ ≈ ≉ ≊ ≋ ≌ ≍ ≎ ≐ ≑ ≒ ≓ ≖ ≗ ≘ ≙ ≚ ≛ ≜ ≝ ≞ ≟ ≣ ≦ ≧ ≨ ≩ ≪ ≫ ≬ ≭ ≮ ≯ ≰ ≱ ≲ ≳ ≴ ≵ ≶ ≷ ≸ ≹ ≺ ≻ ≼ ≽ ≾ ≿ ⊀ ⊁ ⊃ ⊅ ⊇ ⊉ ⊋ ⊏ ⊐ ⊑ ⊒ ⊜ ⊩ ⊬ ⊮ ⊰ ⊱ ⊲ ⊳ ⊴ ⊵ ⊶ ⊷ ⋍ ⋐ ⋑ ⋕ ⋖ ⋗ ⋘ ⋙ ⋚ ⋛ ⋜ ⋝ ⋞ ⋟ ⋠ ⋡ ⋢ ⋣ ⋤ ⋥ ⋦ ⋧ ⋨ ⋩ ⋪ ⋫ ⋬ ⋭ ⋲ ⋳ ⋴ ⋵ ⋶ ⋷ ⋸ ⋹ ⋺ ⋻ ⋼ ⋽ ⋾ ⋿ ⟈ ⟉ ⟒ ⦷ ⧀ ⧁ ⧡ ⧣ ⧤ ⧥ ⩦ ⩧ ⩪ ⩫ ⩬ ⩭ ⩮ ⩯ ⩰ ⩱ ⩲ ⩳ ⩵ ⩶ ⩷ ⩸ ⩹ ⩺ ⩻ ⩼ ⩽ ⩾ ⩿ ⪀ ⪁ ⪂ ⪃ ⪄ ⪅ ⪆ ⪇ ⪈ ⪉ ⪊ ⪋ ⪌ ⪍ ⪎ ⪏ ⪐ ⪑ ⪒ ⪓ ⪔ ⪕ ⪖ ⪗ ⪘ ⪙ ⪚ ⪛ ⪜ ⪝ ⪞ ⪟ ⪠ ⪡ ⪢ ⪣ ⪤ ⪥ ⪦ ⪧ ⪨ ⪩ ⪪ ⪫ ⪬ ⪭ ⪮ ⪯ ⪰ ⪱ ⪲ ⪳ ⪴ ⪵ ⪶ ⪷ ⪸ ⪹ ⪺ ⪻ ⪼ ⪽ ⪾ ⪿ ⫀ ⫁ ⫂ ⫃ ⫄ ⫅ ⫆ ⫇ ⫈ ⫉ ⫊ ⫋ ⫌ ⫍ ⫎ ⫏ ⫐ ⫑ ⫒ ⫓ ⫔ ⫕ ⫖ ⫗ ⫘ ⫙ ⫷ ⫸ ⫹ ⫺ ⊢ ⊣ ⟂ ⫪ ⫫ <: >:))))
(define prec-pipe<       '(|.<\|| |<\||))
(define prec-pipe>       '(|.\|>| |\|>|))
(define prec-colon       (append! '(: |..|) (add-dots '(… ⁝ ⋮ ⋱ ⋰ ⋯))))
(define prec-plus        (append! '($)
                          (add-dots '(+ - − ¦ |\|| ⊕ ⊖ ⊞ ⊟ |++| ∪ ∨ ⊔ ± ∓ ∔ ∸ ≏ ⊎ ⊻ ⊽ ⋎ ⋓ ⟇ ⧺ ⧻ ⨈ ⨢ ⨣ ⨤ ⨥ ⨦ ⨧ ⨨ ⨩ ⨪ ⨫ ⨬ ⨭ ⨮ ⨹ ⨺ ⩁ ⩂ ⩅ ⩊ ⩌ ⩏ ⩐ ⩒ ⩔ ⩖ ⩗ ⩛ ⩝ ⩡ ⩢ ⩣))))
(define prec-times       (add-dots '(* / ⌿ ÷ % & · · ⋅ ∘ × |\\| ∩ ∧ ⊗ ⊘ ⊙ ⊚ ⊛ ⊠ ⊡ ⊓ ∗ ∙ ∤ ⅋ ≀ ⊼ ⋄ ⋆ ⋇ ⋉ ⋊ ⋋ ⋌ ⋏ ⋒ ⟑ ⦸ ⦼ ⦾ ⦿ ⧶ ⧷ ⨇ ⨰ ⨱ ⨲ ⨳ ⨴ ⨵ ⨶ ⨷ ⨸ ⨻ ⨼ ⨽ ⩀ ⩃ ⩄ ⩋ ⩍ ⩎ ⩑ ⩓ ⩕ ⩘ ⩚ ⩜ ⩞ ⩟ ⩠ ⫛ ⊍ ▷ ⨝ ⟕ ⟖ ⟗ ⨟)))
(define prec-rational    (add-dots '(//)))
(define prec-bitshift    (add-dots '(<< >> >>>)))
;; `where`
;; implicit multiplication (juxtaposition)
;; unary
(define prec-power       (add-dots '(^ ↑ ↓ ⇵ ⟰ ⟱ ⤈ ⤉ ⤊ ⤋ ⤒ ⤓ ⥉ ⥌ ⥍ ⥏ ⥑ ⥔ ⥕ ⥘ ⥙ ⥜ ⥝ ⥠ ⥡ ⥣ ⥥ ⥮ ⥯ ￪ ￬)))
(define prec-decl        '(|::|))
;; `where` occurring after `::`
(define prec-dot         '(|.|))

(define prec-names '(prec-assignment
                     prec-pair prec-conditional prec-arrow prec-lazy-or prec-lazy-and prec-comparison
                     prec-pipe< prec-pipe> prec-colon prec-plus prec-times prec-rational prec-bitshift
                     prec-power prec-decl prec-dot))
```

などと定義されている。(`src/julia-parser.scm`)

今回は、binomial演算子の優先順位を、
* bitshift演算子`<< >> >>>`
* power演算子`^`

の間に設定する。

!!! Note title="既存の~演算子"
すでに~は単項演算子として実装されており、これはビット反転を表す。
!!!

以上を踏まえて、ブラックボックステストのテストケースを作成した。
```julia
# examples/unicode.jl
println(3~~2)       # 一般的なもの
println(1~~1)       # binomial(1,1)=1
println(1~~0)       # binomial(1,0)=1
println(-3~~2)      # -binomial(3,2)=3
println((-3)~~2)    # binomial(-3,2)=6
println((-3)~~3)    # binomial(-3,3)=10
println(2~~(-2))    # binomial(2,-2)=0
println((-3)~~(-1)) # binomial(-3,-1)=0
println(2+6~~2)     # 2+binomial(6,2)=17
println(2-6~~2)     # 2-binomial(6,2)=-13
println(3*6~~2)     # 3*binomial(6,2)=45
println((3*6)~~2)   # binomial(3*6,2)=153
println(6~~2<4)     # binomial(6,2)<4=false
println(4<6~~2)     # 4<binomial(6,2)=true
println(4<6~~2 ? "TRUE" : "FALSE") # 4<binomial(6,2) ? "TRUE" : "FALSE" = "TRUE"
println(4<6~~2?"TRUE" : "FALSE")   # 4<binomial(6,2)?"TRUE" : "FALSE" -> ERROR: ParseError: space required before `?` operator
println(4<6~~2&&6~~2<7 ? "TRUE" : "FALSE")  # 4<binomial(6,2)&&binomial(6,2)<7 ? "TRUE" : "FALSE" = "FALSE"
println(4<6~~2&&6~~2<16 ? "TRUE" : "FALSE") # 4<binomial(6,2)&&binomial(6,2)<16 ? "TRUE" : "FALSE" = "TRUE"
println(4<6~~2||6~~2<7 ? "TRUE" : "FALSE")  # 4<binomial(6,2)||binomial(6,2)<7 ? "TRUE" : "FALSE" = "TRUE"
println(4<6~~2||6~~2<16 ? "TRUE" : "FALSE") # 4<binomial(6,2)||binomial(6,2)<16 ? "TRUE" : "FALSE" = "TRUE"
x=2
x+=6~~2
println(x)              # x=2, x+=6~~2 = 17
println(6~~2/3)         # binomial(6,2)/3 = 5.0
println(90/6~~2)        # 90/binomial(6,2) = 6.0
println(6~~2//3)        # binomial(6,2)//3 = 5//1
println(90//6~~2)       # 90//binomial(6,2) = 6//1
println(1<<5~~2)        # 1<<binomial(5,2) = 1024
println(1024>>4~~2)     # 1024>>binomial(4,2) = 16
println((1024>>4)~~2)   # binomial((1024>>4),2) = 2016
println(2^6~~2)         # binomial(2^6,2) = 2016
println(6~~2^2)         # binomial(6,2^2) = 15

# ~と~~
println(~(-7)~~2)       # binomial(~(-7),2) = 15
println(6~~~(-3))       # binomial(6,~(-3)) = 15
println(6~~3~~2)        # binomial(binomial(6,3),2) = 190
println(~6~~3~~2)        # binomial(binomial(~6,3),2) = 3570
println((~6)~~3~~2)        # binomial(binomial(~6,3),2) = 3570
println(6~~(~3)~~2)        # binomial(binomial(6,~3),2) = 0
```

## `./julia`が生成される過程

演算子を追加する上では、**julia**言語の動作原理をうっすら理解しておく必要がある。

まずは、**julia**言語のソフトウェアとしてのディレクトリ/ファイル構造を示す。

![ディレクトリ構造](/assets/directory.png)

### JULIAコードで基本的な構文や機能を実装

`base/`ディレクトリには、無数の`.jl`ファイルが含まれている。

このディレクトリでは、JULIAのコードにより
* 様々な演算子・便利関数の定義
* 字句規則の制定
* 構文規則の制定
* 字句解析の関数定義
* 構文解析の関数定義

などが行われている。特に、字句解析・構文解析関連の実装は、全て、`base/JuliaSyntax/`で行われている。

### C言語で実行の枠組みを作成

我々が普段書くようなJULIAのプログラムを実行する
```sh
julia ./helloworld.jl
```
といったコマンドは、実際には
* 環境変数に指定されたPATHの先にある、実行ファイル（ここまで`./julia`と呼んできたものに相当）を実行する
* 実行時のコマンドライン引数として、プログラムのPATH`./helloworld.jl`を渡す

ことを意味している。

逆に言えば、実行ファイル`./julia`は、`.jl`形式のJULIAで書かれたプログラムに対し、

* 字句解析: 文字列としてのプログラムコード（`helloworld.jl`の中身）を、文->Tokenと分解する
* 構文解析: Tokenの並びが構文規則に従うか判定する
* 意味解析: 解析結果をもとに中間コード（例: 逆ポーランド記法）に変換する
* 型推論: 各変数や式の型を推論
* 最適化: レジスタの割付や、不要な演算の省略などを行う（LLVM; Low-Level-Virtual-Machine コンパイラフレームワークが行う`deps/`のコード）
* アセンブリに変換
* 実行: アセンブリを順に実行する

といった機能を全て包含していることになる。

!!! Note title="字句解析で文に分解する方法"
改行文字\nを読み取るか、文の区切り文字;を読み取る度に区切る。
!!!

実は、これらの機能の司会進行をしているのは、**`src/`ディレクトリに配置されたC言語ないしはC++のコード**から生成された部分である。

### 構文解析器の作成

さて、これまでの説明を聞いて、ふとある疑問が浮かぶのではないか。

!!! Question title="JULIAの文法をJULIAで定義?"
JULIAの字句規則や構文規則ですらJULIAで書かれていては、それを読みようがないのでは、という疑問が生じますよね。至極まともな疑問なので以下で解説します。
!!!

最終的な目標は、**`./julia`に、`base/`の構文規則等を理解させ、構文解析器を含めること**である。そのための手段として、**bootstrap**というものがある。

!!! Tip title="Bootstrapとは"
コンピュータシステムやソフトウェアが、自身を立ち上げるために必要な基盤を自動的に用意しながら段階的に起動する過程や手法である。

プログラミング言語におけるブートストラップとは、言語処理系（コンパイラやインタープリタ）を新しく作る際、最初はその言語自身ではなく別の言語で基礎を構築することを指す。
!!!

JULIAの構文解析も、最初は`julia-parser.scm`のように**Scheme**などの異なる言語で行い(1)、**徐々にJulia自体が自分のコードを処理できるようになる**(2)。
最終的に、自己ホスティングの準備が整うと、以後はJuliaだけでさらに拡張可能になる(3)。

![ブートストラップイメージ](/assets/bootstrap.png)

!!! Tip title="コンピュータのブートストラップ"
コンピュータの電源を入れると、まず最小限のプログラム（BIOSやUEFI）が起動し、ストレージからオペレーティングシステムをロードする。これが進むにつれて、より多くの機能が動作可能となり、最終的にOSが完全に立ち上がるとユーザーが使用できる状態になる。
!!!

## 変更・追加内容

### 演算子の優先レベル`prec-binomial`を追加

`src/julia-parser.scm`の冒頭では、演算子の優先レベルが`prec-XXX`という形で定義されている。
```scheme
;; julia-parser.scm

;; Operator precedence table, lowest at top

(define prec-bitshift    (add-dots '(<< >> >>>)))
(define prec-binomial    (add-dots '(~~)))
(define prec-power       (add-dots '(^ ↑ ↓ ⇵ ⟰ ⟱ ⤈ ⤉ ⤊ ⤋ ⤒ ⤓ ⥉ ⥌ ⥍ ⥏ ⥑ ⥔ ⥕ ⥘ ⥙ ⥜ ⥝ ⥠ ⥡ ⥣ ⥥ ⥮ ⥯ ￪ ￬)))
```
### 演算子の優先順位テーブルに`prec-binomial`を追加

`src/julia-parser.scm`には、演算子の優先レベル`prec-XXX`のテーブルを用意し、その内部の並び順により優先順位づけしている。
```scheme
;; julia-parser.scm

(define prec-names '(prec-assignment
                     prec-pair prec-conditional prec-arrow prec-lazy-or prec-lazy-and prec-comparison
                     prec-pipe< prec-pipe> prec-colon prec-plus prec-times prec-rational prec-bitshift
                     prec-binomial prec-power prec-decl prec-dot))
```

### 構文解析関数`parse-binomial`を追加

```scheme
;; julia-parser.scm l.930~

;; (define (parse-shift s)    (parse-LtoR        s parse-unary-subtype is-prec-bitshift?))
(define (parse-shift s)    (parse-LtoR        s parse-binomial is-prec-bitshift?))
(define (parse-binomial s) (parse-LtoR        s parse-unary-subtype is-prec-binomial?))
```
ここでは、
`s`を`is-prec-binomial`関数にかけ、その結果により処理を場合分けしている。

`true`ならば、
`~~`演算が左結合演算であると仮定し、
左結合演算の解析関数`parse-LtoR`を呼び出す。

!!! Tip title="演算子の左結合・右結合"
ある二項演算子・があり、A・B・C=(A・B)・C=A・(B・C)を満たさないものについて、

常にA・B・C=(A・B)・Cを満たすものを左結合性演算子、
常にA・B・C=A・(B・C)を満たすものを右結合性演算子という。
!!!
![演算子の右結合性、左結合性とは](https://www.pandanoir.info/entry/2016/06/26/115235)

`false`ならば、
さらに優先順位が高い演算の処理を行う`parse-unary-subtype`関数を呼び出す。単項演算子や`^`演算子の処理はこの関数内で行われている。

### `~`記号の字句解析関数`lex_tilde`を追加

`~`の後に、`~`が続き二項演算子`~~`を表す場合と、別の文字が続き単項演算子`~`を表す場合で、異なるラベルづけをして演算子トークンを生成`emit`する。
```julia
# base/JuliaSyntax/src/tokenize.jl l.970~
function lex_tilde(l::Lexer)
    if accept(l, '~')
        return emit(l, K"~~")
    end
    return emit(l, K"~")
end
```
K"~~"は、特定のトークンの種類を表す。

### 字句解析器での`~`への対処を変更

読んだ文字に応じて、適切なトークン発行関数を呼び出す`_next_token`関数において、`~`を読んだ際の対応を変更した。

具体的には、これまでは`~`に単項演算子としての機能しか存在しなかったが、今回二項演算子の場合もあり得るようになったため、場合分けを行う関数`lex_tilde`を呼ぶ形に変更した。
```julia
# base/JuliaSyntax/src/tokenize.jl l.460~
function _next_token(l::Lexer, c)
    # 省略
    elseif c == '⊻'
        return lex_xor(l);
    elseif c == '~'
        # return emit(l, K"~")
        return lex_tilde(l);
    elseif c == '#'
        return lex_comment(l)
    # 省略
end
```

### 演算子辞書に`~~`を独自のカテゴリ`BINOMIAL`として追加

トークンを発行するにあたって、ラベルづけする際に、演算子であるかどうか、どんな種類の演算子なのかが登録された辞書`_kind_names`がある。
今回は、その辞書に`BINOMIAL`カテゴリを追加した。
```julia
# base/JuliaSyntax/src/kinds.jl l.770~

# Definition of Kind type - mapping from token string identifiers to
# enumeration values as used in @K_str
const _kind_names =
[   
    # 略
    # Level 13
    "BEGIN_BINOMIAL"
        "~~"
    "END_BINOMIAL"
    # 略
]
```

### 与えられた文字列の属するカテゴリが`BINOMIAL`か判定する関数を追加

```julia
# base/JuliaSyntax/src/kinds.jl l.1150~

# Predicates for operator precedence
is_prec_times(x)       = K"BEGIN_TIMES"       <= kind(x) <= K"END_TIMES"
is_prec_rational(x)    = K"BEGIN_RATIONAL"    <= kind(x) <= K"END_RATIONAL"
is_prec_binomial(x)    = K"BEGIN_BINOMIAL"    <= kind(x) <= K"END_BINOMIAL" # 追加
is_prec_power(x)       = K"BEGIN_POWER"       <= kind(x) <= K"END_POWER"
```
### `~~`演算子の機能を定義

元々便利関数として実装されていた、`base/intfuncs.jl`の`binomial(x,y)`を呼び出す形で実装した。
```julia
# base/int.jl l.80~

(*)(x::T, y::T) where {T<:BitInteger} = mul_int(x,y)
(~~)(x::T, y::T) where {T<:BitInteger} = binomial(x,y)  # 追加
```

### 二項演算の実行部へ演算子が`~~`である場合も追加

```julia
# base/int.jl l.850~

# issue #15489: since integer ops are unchecked, they shouldn't check promotion
# for op in (:+, :-, :*, :&, :|, :xor)
for op in (:+, :-, :*, :&, :|, :xor, :~~)
    @eval function $op(a::Integer, b::Integer)
        T = promote_typeof(a, b)
        aT, bT = a % T, b % T
        not_sametype((a, b), (aT, bT))
        return $op(aT, bT)
    end
end
```

### 言語ユーザ向けに`~~`演算子をexport

以上の実装だけでは、`./julia`の引数に渡すプログラムから`~~`演算子を利用することはできない仕組みになっているので、必ずexportする。
```julia
# base/exports.jl l.212~

export
# 略
# Operators
    !,
    !=,
    # 略
    ~~,

# 略
```

## 実装結果

以上の実装を済ませたのち、改めて要件が満たされているか確認するため、
冒頭に記したブラックボックステストを施した。

その結果を以下に示しておく。
```sh
yuri1@Yuri-no-MacBook-Air 2024-05-julia % ./julia ./examples/unicode.jl
3
1
1
6
6
-10
0
0
17
-13
45
153
false
true
TRUE
ERROR: LoadError: ParseError:
# Error @ /Users/yuri1/Documents/00_提出物/DOSS_julia/project/2024-05-julia/examples/unicode.jl:16:15
println(4<6~~2 ? "TRUE" : "FALSE") # 4<binomial(6,2) ? "TRUE" : "FALSE" = "TRUE"
println(4<6~~2?"TRUE" : "FALSE")   # 4<binomial(6,2)?"TRUE" : "FALSE" -> ERROR: ParseError: space required before `?` operator
#             └ ── space required before `?` operator
FALSE
TRUE
TRUE
TRUE
17
5.0
6.0
5//1
6//1
1024
16
2016
2016
15
15
15
190
3570
3570
0
```

きちんと要件が満たされていることが確認された。

## 裏話

### なぜ`~~`を使ったのか

当初は`†`を使おうとしていた。ショートカット`⌥`+`T`により簡単に入力できるからである。

しかし、JULIAの字句規則において、†が属するunicodeの番号帯は厳しく制限されていたため、断念せざるを得なかった。

### 変更箇所をどうやって見出したのか

当初は、`src/`ディレクトリ配下のCファイルを中心に、C/C++デバッガを用いて挙動を調べていた。

しかし、一向に構文解析の本体が見当たらなかったため、
`base/`ディレクトリを見たところ、各種演算子の定義がなされているのを発見した。
一方で、字句解析や構文解析が行われている場所は特定できなかった。

この段階で一旦手詰まりになってしまった。というわけで、各種ファイルをtroll（流し釣り）していたところ、色々なものが`Core`に入っており、例えば、
```c
/* julia.h */
extern JL_DLLIMPORT jl_module_t *jl_main_module JL_GLOBALLY_ROOTED;

/* toplevel.c */
void jl_init_main_module(void)
{
    assert(jl_main_module == NULL);
    jl_main_module = jl_new_module(jl_symbol("Main"), NULL);
    jl_main_module->parent = jl_main_module;
    jl_set_const(jl_main_module, jl_symbol("Core"),
                 (jl_value_t*)jl_core_module);
    jl_set_const(jl_core_module, jl_symbol("Main"),
                 (jl_value_t*)jl_main_module);
}
```
と`Core`モジュールが定義されていることはわかった。
そして、構文解析が、`Core._parse`メソッドで行われており`Core`が隠蔽されていることも明らかになって、再び挫折していた。

再びtrollフェーズに入っていたのだが、ファイルの内容を構文解析する`src/ast.c`の`jl_fl_parse`関数を見ると
```c
// Parse `text` starting at 0-based `offset` and attributing the content to
// `filename`. Return an svec of (parsed_expr, final_offset)
JL_DLLEXPORT jl_value_t *jl_fl_parse(const char *text, size_t text_len,
                                     jl_value_t *filename, size_t lineno,
                                     size_t offset, jl_value_t *options)
{
    JL_TIMING(PARSING, PARSING);
    jl_timing_show_filename(jl_string_data(filename), JL_TIMING_DEFAULT_BLOCK);
    if (offset > text_len) {
        jl_value_t *textstr = jl_pchar_to_string(text, text_len);
        JL_GC_PUSH1(&textstr);
        jl_bounds_error(textstr, jl_box_long(offset+1));
    }
    jl_sym_t *rule = (jl_sym_t*)options;
    if (rule != jl_atom_sym && rule != jl_statement_sym && rule != jl_all_sym) {
        jl_error("jl_fl_parse: unrecognized parse options");
    }
    if (offset != 0 && rule == jl_all_sym) {
        jl_error("Parse `all`: offset not supported");
    }

    jl_ast_context_t *ctx = jl_ast_ctx_enter(NULL);
    fl_context_t *fl_ctx = &ctx->fl;
    value_t fl_text = cvalue_static_cstrn(fl_ctx, text, text_len);
    fl_gc_handle(fl_ctx, &fl_text);
    value_t fl_filename = cvalue_static_cstrn(fl_ctx, jl_string_data(filename),
                                              jl_string_len(filename));
    fl_gc_handle(fl_ctx, &fl_filename);
    value_t fl_expr;
    size_t offset1 = 0;
    if (rule == jl_all_sym) {
        value_t e = fl_applyn(fl_ctx, 3, symbol_value(symbol(fl_ctx, "jl-parse-all")),
                              fl_text, fl_filename, fixnum(lineno));
        fl_expr = e;
        offset1 = e == fl_ctx->FL_EOF ? text_len : 0;
    }
    else {
        value_t greedy = rule == jl_statement_sym ? fl_ctx->T : fl_ctx->F;
        value_t p = fl_applyn(fl_ctx, 5, symbol_value(symbol(fl_ctx, "jl-parse-one")),
                              fl_text, fl_filename, fixnum(offset), greedy, fixnum(lineno));
        fl_expr = car_(p);
        offset1 = tosize(fl_ctx, cdr_(p), "parse");
    }
    fl_free_gc_handles(fl_ctx, 2);

    // Convert to julia values
    jl_value_t *expr = NULL, *end_offset = NULL;
    JL_GC_PUSH2(&expr, &end_offset);
    expr = fl_expr == fl_ctx->FL_EOF ? jl_nothing : scm_to_julia(fl_ctx, fl_expr, NULL);
    end_offset = jl_box_long(offset1);
    jl_ast_ctx_leave(ctx);
    jl_value_t *result = (jl_value_t*)jl_svec2(expr, end_offset);
    JL_GC_POP();
    return result;
}
```
と、怪しげなシンボルを複数生成していることに気づき、
```sh
yuri1@Yuri-no-MacBook-Air src % grep -rl "parse-"    
./ast.c
./jlfrontend.scm
./julia-parser.scm
./ast.scm
./flisp/libflisp.a
./flisp/flisp.o
./flisp/flisp
./flisp/flisp.c
./ast.o
./julia_flisp.boot
```
と、いかにもなファイル`julia-parser.scm`の重要性をようやく認知した。

それ以降は、`julia-parser.scm`を頭から読み解くことで、大方の挙動は把握し、`base/`以下のどこを把握すれば良いかも、適宜`grep`コマンドを用いることで理解できた。

しかし、**JULIAはデバッガの使い勝手が悪い**ため、事実上目視デバッグを強いられることも多く、大変苦労した。

最後、どうしても字句解析がうまくいかないことがあったが、エラーメッセージ自体を`grep`で検索するという荒技により、原因箇所を特定したりもした。

結果、`src/`と`base/`の主要なファイルはほぼ全行目を通しているという状態になってしまったが、なんとか演算子を追加できたことで以って一区切りとした。
