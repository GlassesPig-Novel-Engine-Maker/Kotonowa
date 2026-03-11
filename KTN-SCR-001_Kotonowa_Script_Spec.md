
# 文書番号: KTN-SCR-001
# エンジン名: Kotonowa
# 文書名: Kotonowa スクリプト言語仕様書（人間記述形式）
版数: 1.0

---

# 0. 文書情報

|項目|内容|
|---|---|
|文書番号|KTN-SCR-001|
|文書名|Kotonowa スクリプト言語仕様書（人間記述形式）|
|エンジン名|Kotonowa|
|版数|1.0|
|対象|ノベルゲーム / ADVゲーム スクリプト言語|

---

# 1. 本仕様書の目的

本仕様書は **Kotonowaゲームエンジン** において使用する  
**シナリオライターが記述するスクリプト形式**を定義する。

本仕様書は以下を定義する。

1. スクリプト文法（EBNF）
2. 使用可能コマンド
3. スクリプト記述ガイドライン

以下は本仕様の対象外とする。

- スクリプトVM仕様
- インタープリタ実装
- C++実装
- リソースロード内部処理
- GPU描画処理
- セーブデータ内部形式

それらは **別仕様書**で定義する。

---

# 2. スクリプト文法（EBNF）

## 2.1 字句定義

```ebnf
letter      = "A"…"Z" | "a"…"z" | "_" ;
digit       = "0"…"9" ;

identifier  = letter , { letter | digit } ;

integer     = [ "-" ] , digit , { digit } ;
float       = integer , "." , digit , { digit } ;

number      = integer | float ;

boolean     = "true" | "false" ;

string_char = ? any char except " and newline ? ;
string      = '"' , { string_char } , '"' ;

space       = " " | "\t" ;
newline     = "\n" | "\r\n" ;

comment     = "#", { ? any char except newline ? }
            | "//", { ? any char except newline ? } ;
```

---

## 2.2 スクリプト構造

```ebnf
script      = { line } ;

line        = [ statement ] , newline ;

statement   = simple_statement
            | block_statement ;
```

---

## 2.3 単文

```ebnf
simple_statement =
      label_stmt
    | jump_stmt
    | call_stmt
    | return_stmt
    | set_stmt
    | name_stmt
    | text_stmt
    | narration_stmt
    | bg_stmt
    | chara_stmt
    | bgm_stmt
    | se_stmt
    | voice_stmt
    | anim_stmt
    | motion_stmt
    | wait_stmt
    | include_stmt ;
```

---

## 2.4 制御構文

### ラベル

```ebnf
label_stmt = "label", identifier ;
```

例

```
label start
```

---

### ジャンプ

```ebnf
jump_stmt = "jump", identifier ;
```

---

### サブルーチン

```ebnf
call_stmt   = "call", identifier ;
return_stmt = "return" ;
```

---

### 変数代入

```ebnf
set_stmt = "set", identifier, "=", expression ;
```

例

```
set score = 10
```

---

## 2.5 条件分岐

```ebnf
if_block =
    "if", expression
    { statement }
    [ "else", { statement } ]
    "endif" ;
```

例

```
if score > 10
    text "強い"
else
    text "弱い"
endif
```

---

## 2.6 選択肢

```ebnf
choice_block =
    "choice"
    { choice_item }
    "endchoice" ;

choice_item =
    string , "->" , identifier ;
```

例

```
choice
    "進む" -> next
    "戻る" -> back
endchoice
```

---

# 3. コマンド仕様

## 3.1 テキスト表示

### name

話者名表示

```
name "アリス"
```

### text

会話表示

```
text "こんにちは"
```

### narration

地の文

```
narration "空は赤く染まっていた"
```

---

## 3.2 背景

### bg

静止画背景

```
bg "bg/school/day.png"
```

### bg_spine

Spine背景

```
bg_spine file="bg/forest/forest.skel"
```

---

## 3.3 立ち絵

### chara

静止画立ち絵

```
chara id="alice", file="chara/alice/smile.png"
```

### chara_spine

Spine立ち絵

```
chara_spine id="alice", file="chara/alice/alice.skel"
```

---

## 3.4 音声

### bgm

```
bgm "bgm/day.ogg"
```

### se

```
se "se/door.ogg"
```

### voice

```
voice "voice/alice001.ogg"
```

---

## 3.5 アニメーション

### anim

```
anim id="alice", x=500, time=300
```

### motion

```
motion id="alice", type="jump"
```

### wait

```
wait time=300
```

---

# 4. シナリオ記述ガイドライン

## 4.1 基本ルール

1行につき1命令を書く。

良い例

```
name "アリス"
text "おはよう"
```

悪い例

```
name "アリス" text "おはよう"
```

---

## 4.2 ラベル命名

推奨

```
label chapter1_start
label alice_route_day1
label bad_end_01
```

非推奨

```
label a1
label test
```

---

## 4.3 インデント

ブロック内は **4スペース** を推奨。

```
if flag_open
    text "開いている"
else
    text "閉じている"
endif
```

---

## 4.4 リソースパス

スクリプトでは **論理パスのみ使用する**。

良い例

```
bg "bg/school/day.png"
```

悪い例

```
bg "C:\game\bg\school.png"
```

---

# 5. シーンテンプレート

```
label school_morning

bg "bg/school/day.png"
bgm "bgm/day.ogg"

chara id="alice", file="chara/alice/smile.png"

name "アリス"
text "おはようございます"

choice
    "挨拶する" -> greet
    "無視する" -> ignore
endchoice
```

---

# 6. 文書改訂履歴

|版数|内容|
|---|---|
|1.0|初版作成|
