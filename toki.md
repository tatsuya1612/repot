# Franklin.jlパッケージの機能拡張

## Franklin.jlとは
Franklin.jlは、Juliaの外部パッケージの一つで、Markdownファイルをhtmlファイルに変換する静的サイトジェネレータです。
![Franklin.jlの使用例](/assets/franklin.png)
!!! Note
title="実は..."
このブログはFranklin.jlで生成しています!
また、私達が実装した機能を実際に使用して作成しています。
!!!

## 実装した機能: Admonitions
Admonitionsは、以下のように警告、注意、ヒントなどのボックスを簡単に生成できる機能です。pythonの静的サイトジェネレータ(つまりpython版のFranklin.jl)にある機能で、今回はそれをFranklin.jlにも実装してみました。
!!! Example
Admonitionsの例
!!!

タイトルの「Example」はデフォルトのタイトルですが、自分で変えることもできます。
!!! Warning
title="こんな男には気をつけろ"
・男友達が少ない男
・インスタの投稿がやたら多い男
!!!

### 使い方
```
!!! (Admonitionsの種類)
title="タイトル" #必要な場合記述する。書かなければデフォルトのタイトルになる。
本文
!!!
```
上のような文法でAdmonitionsを使用することができます。例えば、以下のように書くと先程の警告のAdmonitionsが生成できます。
```
!!! Warning
title="こんな男には気をつけろ"
・男友達が少ない男
・インスタの投稿がやたら多い男
!!!
```
改行は必要ではなく、`!!! Info 明日は休講です!!!`のように一行で記述することもできます。`!!! (種類)`と`!!!`で挟まれてさえすればよいのです。
!!! Info 明日は休講です!!!

## 実装したAdmonitions一覧
`!!! Info 情報!!!`
!!! Info 情報!!!
`!!! Note メモ!!!`
!!! Note メモ!!!
`!!! Warning 警告!!!`
!!! Warning 警告!!!
`!!! Bug バグ!!!`
!!! Bug バグ!!!
`!!! Summary 要約!!!`
!!! Summary 要約!!!
`!!! Success 成功!!!`
!!! Success 成功!!!
`!!! Failure 失敗!!!`
!!! Failure 失敗!!!
`!!! Danger 危険!!!`
!!! Danger 危険!!!
`!!! Tip ヒント!!!`
!!! Tip ヒント!!!
`!!! Question 質問!!!`
!!! Question 質問!!!
`!!! Example 例!!!`
!!! Example 例!!!
`!!! Quote 引用!!!`
!!! Quote 引用!!!

## 実装
### 実装方針
マークダウン形式の文章では、以下のようにコードブロックを挿入することができます。
```c
int a = 1;
int b = 3;
printf("a * b = %d\n", a*b);
```
これは以下のような文法で書くことができます。
!!! Info
title="コードブロックの文法"
```c
int a = 1;
int b = 3;
printf("a * b = %d\n", a*b);
```
!!!
一方で、Admonitionsは以下のように記述できます。
```
!!! Warning
title="こんな男には気をつけろ"
・男友達が少ない男
・インスタの投稿がやたら多い男
!!!
```
文法が非常に似ていることがわかります。このことから、Admonitionsはコードブロックの実装方法を参考にすれば実装できると考えました。
実際、コードブロックの実装とAdmonitionsの実装は7割同じです。

### 実際のコード
#### 特定の文字列を辞書に登録
```julia
#Franklin.jl/src/parser/markdown/tokens.jl
const MD_TOKENS = LittleDict{Char, Vector{TokenFinder}}(

               #(中略)

'!'  => [ isexactly("!!! Info") => :ADMONITION_INFO, #toki added
              isexactly("!!! Note") => :ADMONITION_NOTE,
              isexactly("!!! Warning") => :ADMONITION_WARNING,
              isexactly("!!! Danger") => :ADMONITION_DANGER,
              isexactly("!!! Success") => :ADMONITION_SUCCESS,
              isexactly("!!! Bug") => :ADMONITION_BUG,
              isexactly("!!! Summary") => :ADMONITION_SUMMARY,
              isexactly("!!! Tip") => :ADMONITION_TIP,
              isexactly("!!! Failure") => :ADMONITION_FAILURE,
              isexactly("!!! Question") => :ADMONITION_QUESTION,
              isexactly("!!! Quote") => :ADMONITION_QUOTE,
              isexactly("!!! Example") => :ADMONITION_EXAMPLE,
              isexactly("!!!")      => :ADMONITION_CLOSE,
            ]
    )
```
マークダウン形式の文章中の特定の文字列(!!! Info)などをトークンとして辞書MD_TOKENSに登録します。
各トークンにはADMONITION_INFOなどの名前をつけています。
```julia
'`'  => [ isexactly("`",  ('`',), false)  => :CODE_SINGLE, # `⎵
            isexactly("``", ('`',), false)  => :CODE_DOUBLE, # ``⎵*
            isexactly("```",   SPACE_CHAR) => :CODE_TRIPLE,  # ```⎵*
            isexactly("```!",  SPACE_CHAR) => :CODE_TRIPLE!, # ```!⎵*
            isexactly("```>",  SPACE_CHAR) => :CODE_REPL,    # ```>⎵*
            isexactly("```?",  SPACE_CHAR) => :CODE_HELP,    # ```?⎵*
            isexactly("```;",  SPACE_CHAR) => :CODE_SHELL,   # ```;⎵*
            isexactly("```]",  SPACE_CHAR) => :CODE_PKG,     # ```]⎵*
            # 3+ can be named
            is_language(3)                 => :CODE_LANG3,   # ```lang*
            isexactly("````",  SPACE_CHAR) => :CODE_QUAD,    # ````⎵*
            is_language(4)                 => :CODE_LANG4,   # ````lang*
            isexactly("`````", SPACE_CHAR) => :CODE_PENTA,   # `````⎵*
            is_language(5)                 => :CODE_LANG5,   # `````lang*
],
```
!!! Info
title="MD_TOKENSに登録されているもの"
MD_TOKENSには上記のようにコードブロックを記述するための文字列が登録されており、
他にもタイトルヘッダの`#`や数式表示の`\$`も登録されています。
!!!

#### ブロックのリストに追加
```julia
#Franklin.jl/src/parser/markdown/tokens.jl
const MD_OCB = [
  
  　#(中略)

    OCProto(:ADMONITION_INFO, :ADMONITION_INFO, (:ADMONITION_CLOSE,)), 
    OCProto(:ADMONITION_DANGER, :ADMONITION_DANGER, (:ADMONITION_CLOSE,)),
    OCProto(:ADMONITION_WARNING, :ADMONITION_WARNING, (:ADMONITION_CLOSE,)),
    OCProto(:ADMONITION_SUCCESS, :ADMONITION_SUCCESS, (:ADMONITION_CLOSE,)),
    OCProto(:ADMONITION_BUG, :ADMONITION_BUG, (:ADMONITION_CLOSE,)),
    OCProto(:ADMONITION_FAILURE, :ADMONITION_FAILURE, (:ADMONITION_CLOSE,)),
    OCProto(:ADMONITION_TIP, :ADMONITION_TIP, (:ADMONITION_CLOSE,)),
    OCProto(:ADMONITION_SUMMARY, :ADMONITION_SUMMARY, (:ADMONITION_CLOSE,)),
    OCProto(:ADMONITION_QUESTION, :ADMONITION_QUESTION, (:ADMONITION_CLOSE,)),
    OCProto(:ADMONITION_QUOTE, :ADMONITION_QUOTE, (:ADMONITION_CLOSE,)),
    OCProto(:ADMONITION_EXAMPLE, :ADMONITION_EXAMPLE, (:ADMONITION_CLOSE,)),
    OCProto(:ADMONITION_NOTE, :ADMONITION_NOTE, (:ADMONITION_CLOSE,))
]
```
Open-Close BlocksのリストにAdmonitionを追加します。これによって、`!!!(種類)`と`!!!`で挟まれた部分が1ブロックとして認識されます。

#### 段落中の挿入への対応
```julia
#Franklin.jl/src/parser/markdown/tokens.jl
const ADMONITION_NAMES = (
    :ADMONITION_INFO,
    :ADMONITION_NOTE,
    :ADMONITION_DANGER,
    :ADMONITION_WARNING,
    :ADMONITION_SUMMARY,
    :ADMONITION_TIP,
    :ADMONITION_FAILURE,
    :ADMONITION_BUG,
    :ADMONITION_QUESTION,
    :ADMONITION_QUOTE,
    :ADMONITION_EXAMPLE,
    :ADMONITION_SUCCESS
)

const MD_CLOSEP = [MD_HEADER..., :DIV, CODE_BLOCKS_NAMES..., MATH_DISPLAY_BLOCKS_NAMES..., ADMONITION_NAMES...]
```
ADMONITION\_NAMESにブロック名をまとめて登録します。それをMD\_CLOSEPに登録します。MD\_CLOSEPに登録することによって、
段落中などにブロックが挿入されたときに、その段落を自動的に閉じることができるようになります。

#### htmlに変換する関数
```julia
#Franklin.jl/src/converter/markdown/blocks.jl
function convert_block(β::AbstractBlock, lxdefs::Vector{LxDef})::AS
    #(中略)
    βn == :ADMONITION_NOTE      && return html_admonition(stent(β), "note")
    βn == :ADMONITION_INFO      && return html_admonition(stent(β), "info")
    βn == :ADMONITION_SUCCESS      && return html_admonition(stent(β), "success")
    βn == :ADMONITION_DANGER      && return html_admonition(stent(β), "danger")
    βn == :ADMONITION_WARNING      && return html_admonition(stent(β), "warning")
    βn == :ADMONITION_TIP      && return html_admonition(stent(β), "tip")
    βn == :ADMONITION_FAILURE      && return html_admonition(stent(β), "failure")
    βn == :ADMONITION_SUMMARY      && return html_admonition(stent(β), "summary")
    βn == :ADMONITION_BUG      && return html_admonition(stent(β), "bug")
    βn == :ADMONITION_QUESTION      && return html_admonition(stent(β), "question")
    βn == :ADMONITION_QUOTE      && return html_admonition(stent(β), "quote")
    βn == :ADMONITION_EXAMPLE      && return html_admonition(stent(β), "example")
    #(中略)
end
```
convert\_block関数は、ブロックをhtmlの文章に変換したものを返す関数です。
βnはマークダウン中から切り出されたブロックです。
ADMONITIONブロックがあれば、ブロックをhtmlの文章に変換するhtml\_admonition関数を呼び出します。html\_admonition関数については次の節で解説しますが、第二引数にAdmonitionの種類を取ります。ここでは、ブロックに対応する種類を第二引数としてhtml\_admonition関数を呼び出しています。
!!! Info
ブロックがコードブロックやタイトルヘッダの場合でも、convert_block関数がそれらをhtml形式に変換する関数を呼び出しています。
!!!

#### html_admonition関数の実装
```julia
#Franklin.jl/src/utils/html.jl
function html_admonition(content::AS, admonition_type::AS="")::String
    isempty(content) && return ""
    
    admonition_class = "admonition $(admonition_type)"
    title = uppercasefirst(admonition_type)

    m = match(r"title=\"([^\"]+)\"", content)
    if m !== nothing
        title = m.captures[1] 
        content = replace(content, r"title=\"[^\"]+\"" => "") 
    end
    
    content = replace(content, "\n" => "<br>")
    
    
    return html_text_admonition(admonition_class, title, content)
end
```
この関数は、Markdownファイルで使用される警告、ヒント、質問などのAdmonitionブロックをHTMLに変換するためのものです。

##### 引数
- `content::AS`: Admonitionブロックのコンテンツ
- `admonition_type::AS=""`(オプション): Admonitionのタイプ(例: "Warning", "Tip", "Question"など)

##### 返り値
- AdmonitionブロックのHTML表現

##### 機能
1. `content`が空の場合は空の文字列を返します。
```julia
isempty(content) && return ""
```
2. Admonitionのクラス名を"admonition [admonition_type]"という形式で作成します。
```julia
admonition_class = "admonition $(admonition_type)"`
```
3. Admonitionのタイトルを`admonition_type`から生成しますが、`content`に`title="[TITLE]"`という形式のタイトル指定がある場合はそれを優先します。
```julia
title = uppercasefirst(admonition_type)

m = match(r"title=\"([^\"]+)\"", content)
if m !== nothing
    title = m.captures[1] 
    content = replace(content, r"title=\"[^\"]+\"" => "") 
end
```
4. `content`内の改行を`<br>`に置き換えます。
```julia
content = replace(content, "\n" => "<br>")
```
5. `html_text_admonition`関数を呼び出して、Admonitionのクラス名、タイトル、コンテンツからHTMLを生成します。
```julia
return html_text_admonition(admonition_class, title, content)
```

この関数を使うことで、Markdownファイル内のAdmonitionブロックをHTMLに円滑に変換できます。

#### html\_text\_admoniton関数
```julia
#Franklin.jl/src/utils/html.jl
function html_text_admonition(admonition_class, title, content)
    return """
    <head>
        <style>
            #(中略)
        </style>
    </head>
    <body>
        <div class="$admonition_class">
            <p class="admonition-title">$title</p>
            <p>$content</p>
        </div>
    </body>
    """
end
```
この関数は、Markdownファイルで使用される警告、ヒント、質問などのAdmonitionブロックをHTMLに変換するためのものです。
##### 引数

- `admonition_class`: Admonitionのタイプに応じたCSSクラス名 (例: "admonition warning")
-`title`: Admonitionのタイトル
-`content`: Admonitionのコンテンツ

##### 返り値
AdmonitionブロックのHTML表現
##### 機能
この関数は以下のようなHTMLを生成して返します:
```html
<head>
    <style>
        /* CSS styles for different types of admonitions */
    </style>
</head>
<body>
    <div class="$admonition_class">
        <p class="admonition-title">$title</p>
        <p>$content</p>
    </div>
</body>
```
ヘッダ部分のCSS スタイルでは、Admonitionのタイプに応じて異なる見栄えを定義しています。
この関数を使うことで、Markdownファイル中のAdmonitionブロックをHTMLに変換できるようになります。

##### ヘッダ部分
この関数の中では、ヘッダ部分でAdmonitionのCSSスタイルを定義しています。これにより、各Admonitionタイプに応じた見栄えを実現しています。
ヘッダ部分には以下のような CSS スタイルが定義されています:
```html
<head>
    <style>
        /* Note admonition styles */
        .admonition.note {
            border-left-color: #448aff;
        }
        .admonition.note .admonition-title {
            background-color: rgba(68,138,255,.1);
        }
        .admonition.note .admonition-title::before {
            background-image: url('data:image/svg+xml;...'); 
        }

        /* Warning admonition styles */
        .admonition.warning {
            border-left-color: #ff9100;
        }
        .admonition.warning .admonition-title {
            background-color: rgba(255,145,0,.1);
        }
        .admonition.warning .admonition-title::before {
            background-image: url('data:image/svg+xml;...'); 
        }

        /* Additional admonition styles for other types */
        ...
    </style>
</head>
```
このCSSは、Admonitionの種類ごとに以下の設定を行っています:

- 左境界線の色
- タイトル部分の背景色
- タイトル部分のアイコン (SVG画像)

これにより、Admonitionのタイプに応じた視覚的な区別が可能になります。
例えば、"warning"タイプのAdmonitionは橙色の左境界線と背景色、警告マークのアイコンが表示されます。
このようにヘッダ部分のCSSスタイルが、Admonitionの見栄えを定義しています。