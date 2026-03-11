# 文書番号: KTN-SCR-001A
# エンジン名: Kotonowa
# 文書名: Kotonowa スクリプト言語詳細仕様書（人間記述形式補足）
版数: 1.0

---

# 0. 文書情報

|項目|内容|
|---|---|
|文書番号|KTN-SCR-001A|
|文書名|Kotonowa スクリプト言語詳細仕様書（人間記述形式補足）|
|エンジン名|Kotonowa|
|版数|1.0|
|対象|ノベルゲーム / ADVゲーム用スクリプト言語|
|位置づけ|KTN-SCR-001 の補足詳細仕様|
|関連文書|KTN-SCR-001 スクリプト言語仕様書（人間記述形式）, KTN-RES-001 リソースファイル仕様書, 実装系仕様書群|

---

# 1. 本文書の目的と適用範囲

本仕様書は、**Kotonowa** においてシナリオライター、演出担当、スクリプト記述担当者が直接記述する  
**人間記述形式のスクリプト**について、以下を詳細に定義する。

1. スクリプト文法の正式EBNF
2. コマンド一覧完全版
3. スクリプト記述ガイドライン

本仕様書は**人が書くスクリプトの形式**のみを対象とする。  
以下は本仕様書の対象外とし、必ず別文書で定義する。

- インタープリタ実装
- VM 命令仕様
- C++ クラス設計
- リソースマネージャ内部実装
- 並列処理内部仕様
- GPU / 描画パイプライン実装
- Spine ランタイム実装詳細
- セーブデータ内部形式

---

# 2. 設計原則

Kotonowa のスクリプト言語は、以下の原則に従って設計する。

1. **人が読みやすいこと**
2. **1行1命令を基本とすること**
3. **行ベース構文であること**
4. **論理パスでリソースを指定すること**
5. **静止画とSpineを共通思想で扱えること**
6. **背景、立ち絵、イベントCG、UIを統一的に操作できること**
7. **画面切替と汎用アニメーションを高水準命令で指定できること**
8. **実装仕様と混在させないこと**

---

# 3. 基本概念

## 3.1 行ベース構文

スクリプトは**行単位**で解釈される。  
原則として、1行には1命令のみを記述する。

例:

```txt
name "アリス"
text "おはようございます。"
bg "bg/school/day.png"
```

## 3.2 論理パス

スクリプト中のファイル指定は、すべて **論理パス** を用いる。  
OS実パスやプラットフォーム依存パス記法は使用しない。

良い例:

```txt
bg "bg/school/day.png"
voice "voice/alice/ch01_001.ogg"
```

悪い例:

```txt
bg "C:\game\bg\school\day.png"
bg "bg\school\day.png"
```

## 3.3 表示オブジェクト

画面上に表示される要素は、概念上以下の種別を持つ。

- 背景
- 立ち絵
- イベントCG
- UI
- 前景小物
- 演出用オブジェクト

これらは、**静止画**または**Spineアニメーション**のいずれでも表現可能とする。

## 3.4 表示アセットの描画種別

表示アセットの実体は以下のいずれかとする。

- `image` : 静止画
- `spine` : Spineアニメーション

## 3.5 トランジションとアニメーション

視覚演出は以下の3系統に分ける。

1. **トランジション**  
   画面状態の切替演出
2. **オブジェクトアニメーション**  
   個別表示オブジェクトへの動作
3. **スクリーンアニメーション**  
   画面全体またはレイヤー全体への動作

---

# 4. 字句仕様

## 4.1 改行

改行は命令終端として扱う。

## 4.2 空白

半角スペースおよびタブを空白とする。  
行頭・行末・引数区切りの余分な空白は許容される。

## 4.3 コメント

コメントは以下のいずれかで記述する。

```txt
# コメント
// コメント
```

行末コメントも許可する。

```txt
bg "bg/school/day.png"  # 教室背景
```

## 4.4 識別子

識別子は以下の規則に従う。

- 先頭: 英字または `_`
- 続き: 英字、数字、`_`

例:

```txt
start
alice_route_day1
flag_seen_intro
_ui_cursor
```

## 4.5 文字列

文字列は `"` で囲む。

```txt
"こんにちは"
"bg/school/day.png"
"改行\nタブ\t引用符\""
```

## 4.6 数値

整数および浮動小数を許可する。

```txt
10
-5
3.14
0.5
```

## 4.7 真偽値

```txt
true
false
```

## 4.8 null

```txt
null
```

---

# 5. データ型

本言語で扱う基本型は以下とする。

- `int`
- `float`
- `string`
- `bool`
- `null`

将来拡張候補:

- `array`
- `dict`
- `object`
- `handle`

---

# 6. 正式EBNF

## 6.1 字句

```ebnf
letter          = "A"…"Z" | "a"…"z" | "_" ;
digit           = "0"…"9" ;
hex_digit       = digit | "A"…"F" | "a"…"f" ;

identifier      = letter , { letter | digit | "_" } ;

integer         = [ "-" ] , digit , { digit } ;
float           = [ "-" ] , digit , { digit } , "." , digit , { digit } ;
number          = float | integer ;

boolean         = "true" | "false" ;
null            = "null" ;

escape          = "\" , ( "\" | "\"" | "n" | "t" | "r" ) ;
string_char     = ? any character except " and newline ? | escape ;
string          = "\"" , { string_char } , "\"" ;

space           = " " | "\t" ;
newline         = "\r\n" | "\n" | "\r" ;
comment         = "#" , { ? any character except newline ? }
                | "//" , { ? any character except newline ? } ;
spacing         = { space } ;
```

## 6.2 ファイル全体

```ebnf
script          = { line } ;

line            = [ spacing ] , [ statement ] , [ spacing ] , [ comment ] , newline
                | [ spacing ] , comment , newline
                | [ spacing ] , newline ;
```

## 6.3 文

```ebnf
statement       = simple_statement
                | block_statement ;
```

## 6.4 単文

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
                | append_stmt
                | clear_text_stmt
                | bg_stmt
                | bg_spine_stmt
                | bg_clear_stmt
                | bg_color_stmt
                | chara_stmt
                | chara_spine_stmt
                | chara_mod_stmt
                | chara_hide_stmt
                | chara_focus_stmt
                | chara_unfocus_stmt
                | eventcg_stmt
                | eventcg_spine_stmt
                | eventcg_mod_stmt
                | eventcg_hide_stmt
                | ui_show_stmt
                | ui_spine_stmt
                | ui_hide_stmt
                | ui_mod_stmt
                | show_stmt
                | hide_stmt
                | move_stmt
                | scale_stmt
                | rotate_stmt
                | alpha_stmt
                | layer_stmt
                | order_stmt
                | color_stmt
                | visible_stmt
                | bgm_stmt
                | bgm_stop_stmt
                | bgm_fade_stmt
                | bgm_pause_stmt
                | bgm_resume_stmt
                | se_stmt
                | se_stop_stmt
                | voice_stmt
                | voice_stop_stmt
                | voice_wait_stmt
                | volume_stmt
                | transition_stmt
                | scene_stmt
                | fadein_stmt
                | fadeout_stmt
                | flash_stmt
                | shake_stmt
                | tint_stmt
                | anim_stmt
                | motion_stmt
                | path_stmt
                | playanim_stmt
                | wait_stmt
                | wait_anim_stmt
                | stop_anim_stmt
                | stop_motion_stmt
                | spine_anim_stmt
                | spine_queue_stmt
                | spine_skin_stmt
                | spine_stop_stmt
                | spine_mix_stmt
                | spine_wait_anim_stmt
                | spine_wait_event_stmt
                | window_show_stmt
                | window_hide_stmt
                | window_opacity_stmt
                | namebox_show_stmt
                | namebox_hide_stmt
                | choice_config_stmt
                | auto_mode_stmt
                | skip_mode_stmt
                | preload_stmt
                | preload_async_stmt
                | wait_load_stmt
                | unload_stmt
                | load_group_stmt
                | unload_group_stmt
                | include_stmt
                | call_script_stmt
                | asset_use_stmt
                | autosave_stmt
                | save_enable_stmt
                | save_title_stmt
                | log_stmt
                | assert_stmt ;
```

## 6.5 ブロック文

```ebnf
block_statement = if_block
                | while_block
                | choice_block
                | preload_block
                | asset_block
                | resource_group_block
                | animation_block
                | parallel_block ;
```

## 6.6 値と式

```ebnf
value           = number
                | string
                | boolean
                | null
                | identifier
                | function_call ;

function_call   = identifier , "(" , [ argument_values ] , ")" ;
argument_values = expression , { "," , expression } ;

expression      = logical_or ;

logical_or      = logical_and , { "||" , logical_and } ;
logical_and     = equality , { "&&" , equality } ;
equality        = comparison , { ( "==" | "!=" ) , comparison } ;
comparison      = additive , { ( "<" | "<=" | ">" | ">=" ) , additive } ;
additive        = multiplicative , { ( "+" | "-" ) , multiplicative } ;
multiplicative  = unary , { ( "*" | "/" | "%" ) , unary } ;
unary           = [ "!" | "-" ] , primary ;
primary         = number
                | string
                | boolean
                | null
                | identifier
                | function_call
                | "(" , expression , ")" ;
```

## 6.7 引数

```ebnf
argument        = expression
                | identifier , "=" , expression ;

argument_list   = argument , { "," , argument } ;
```

## 6.8 制御構文

```ebnf
label_stmt      = "label" , spacing , identifier ;
jump_stmt       = "jump" , spacing , identifier ;
call_stmt       = "call" , spacing , identifier ;
return_stmt     = "return" ;

set_stmt        = "set" , spacing , [ scope , spacing ] , identifier ,
                  spacing , "=" , spacing , expression ;

scope           = "local" | "scene" | "global" ;
```

## 6.9 if 文

```ebnf
if_block        = "if" , spacing , expression , newline ,
                  { block_line } ,
                  { elseif_part } ,
                  [ else_part ] ,
                  [ spacing ] , "endif" , newline ;

elseif_part     = [ spacing ] , "elseif" , spacing , expression , newline ,
                  { block_line } ;

else_part       = [ spacing ] , "else" , newline ,
                  { block_line } ;

block_line      = [ spacing ] , [ statement ] , [ spacing ] , [ comment ] , newline
                | [ spacing ] , comment , newline
                | [ spacing ] , newline ;
```

## 6.10 while 文

```ebnf
while_block     = "while" , spacing , expression , newline ,
                  { block_line } ,
                  [ spacing ] , "endwhile" , newline ;
```

## 6.11 choice 文

```ebnf
choice_block    = "choice" , [ spacing , argument_list ] , newline ,
                  { choice_item } ,
                  [ spacing ] , "endchoice" , newline ;

choice_item     = [ spacing ] , string , spacing , "->" , spacing , identifier ,
                  [ spacing , "if" , spacing , expression ] ,
                  [ spacing ] , [ comment ] , newline ;
```

## 6.12 テキスト系

```ebnf
name_stmt       = "name" , spacing , string ;
text_stmt       = "text" , spacing , string ;
narration_stmt  = "narration" , spacing , string ;
append_stmt     = "append" , spacing , string ;
clear_text_stmt = "clear_text" ;
```

## 6.13 汎用コマンド形式

```ebnf
generic_cmd     = identifier , [ spacing , argument_list ] ;
```

代表的な汎用形式命令:

```ebnf
bg_stmt                 = "bg" , spacing , argument_list ;
bg_spine_stmt           = "bg_spine" , spacing , argument_list ;
bg_clear_stmt           = "bg_clear" , [ spacing , argument_list ] ;
bg_color_stmt           = "bg_color" , spacing , argument_list ;

chara_stmt              = "chara" , spacing , argument_list ;
chara_spine_stmt        = "chara_spine" , spacing , argument_list ;
chara_mod_stmt          = "chara_mod" , spacing , argument_list ;
chara_hide_stmt         = "chara_hide" , spacing , argument_list ;
chara_focus_stmt        = "chara_focus" , spacing , argument_list ;
chara_unfocus_stmt      = "chara_unfocus" , spacing , argument_list ;

eventcg_stmt            = "eventcg" , spacing , argument_list ;
eventcg_spine_stmt      = "eventcg_spine" , spacing , argument_list ;
eventcg_mod_stmt        = "eventcg_mod" , spacing , argument_list ;
eventcg_hide_stmt       = "eventcg_hide" , [ spacing , argument_list ] ;

ui_show_stmt            = "ui_show" , spacing , argument_list ;
ui_spine_stmt           = "ui_spine" , spacing , argument_list ;
ui_hide_stmt            = "ui_hide" , spacing , argument_list ;
ui_mod_stmt             = "ui_mod" , spacing , argument_list ;

show_stmt               = "show" , spacing , argument_list ;
hide_stmt               = "hide" , spacing , argument_list ;
move_stmt               = "move" , spacing , argument_list ;
scale_stmt              = "scale" , spacing , argument_list ;
rotate_stmt             = "rotate" , spacing , argument_list ;
alpha_stmt              = "alpha" , spacing , argument_list ;
layer_stmt              = "layer" , spacing , argument_list ;
order_stmt              = "order" , spacing , argument_list ;
color_stmt              = "color" , spacing , argument_list ;
visible_stmt            = "visible" , spacing , argument_list ;

bgm_stmt                = "bgm" , spacing , argument_list ;
bgm_stop_stmt           = "bgm_stop" , [ spacing , argument_list ] ;
bgm_fade_stmt           = "bgm_fade" , spacing , argument_list ;
bgm_pause_stmt          = "bgm_pause" , [ spacing , argument_list ] ;
bgm_resume_stmt         = "bgm_resume" , [ spacing , argument_list ] ;
se_stmt                 = "se" , spacing , argument_list ;
se_stop_stmt            = "se_stop" , [ spacing , argument_list ] ;
voice_stmt              = "voice" , spacing , argument_list ;
voice_stop_stmt         = "voice_stop" , [ spacing , argument_list ] ;
voice_wait_stmt         = "voice_wait" , [ spacing , argument_list ] ;
volume_stmt             = "volume" , spacing , argument_list ;

transition_stmt         = "transition" , spacing , argument_list ;
scene_stmt              = "scene" , spacing , argument_list ;
fadein_stmt             = "fadein" , spacing , argument_list ;
fadeout_stmt            = "fadeout" , spacing , argument_list ;
flash_stmt              = "flash" , spacing , argument_list ;
shake_stmt              = "shake" , spacing , argument_list ;
tint_stmt               = "tint" , spacing , argument_list ;

anim_stmt               = "anim" , spacing , argument_list ;
motion_stmt             = "motion" , spacing , argument_list ;
path_stmt               = "path" , spacing , argument_list ;
playanim_stmt           = "playanim" , spacing , argument_list ;
wait_stmt               = "wait" , spacing , argument_list ;
wait_anim_stmt          = "wait_anim" , spacing , argument_list ;
stop_anim_stmt          = "stop_anim" , spacing , argument_list ;
stop_motion_stmt        = "stop_motion" , spacing , argument_list ;

spine_anim_stmt         = "spine_anim" , spacing , argument_list ;
spine_queue_stmt        = "spine_queue" , spacing , argument_list ;
spine_skin_stmt         = "spine_skin" , spacing , argument_list ;
spine_stop_stmt         = "spine_stop" , spacing , argument_list ;
spine_mix_stmt          = "spine_mix" , spacing , argument_list ;
spine_wait_anim_stmt    = "spine_wait_anim" , spacing , argument_list ;
spine_wait_event_stmt   = "spine_wait_event" , spacing , argument_list ;

window_show_stmt        = "window_show" , [ spacing , argument_list ] ;
window_hide_stmt        = "window_hide" , [ spacing , argument_list ] ;
window_opacity_stmt     = "window_opacity" , spacing , argument_list ;
namebox_show_stmt       = "namebox_show" , [ spacing , argument_list ] ;
namebox_hide_stmt       = "namebox_hide" , [ spacing , argument_list ] ;
choice_config_stmt      = "choice_config" , spacing , argument_list ;
auto_mode_stmt          = "auto_mode" , spacing , argument_list ;
skip_mode_stmt          = "skip_mode" , spacing , argument_list ;

preload_stmt            = "preload" , spacing , argument_list ;
preload_async_stmt      = "preload_async" , spacing , argument_list ;
wait_load_stmt          = "wait_load" , spacing , argument_list ;
unload_stmt             = "unload" , spacing , argument_list ;
load_group_stmt         = "load_group" , spacing , argument_list ;
unload_group_stmt       = "unload_group" , spacing , argument_list ;
include_stmt            = "include" , spacing , string ;
call_script_stmt        = "call_script" , spacing , string ;
asset_use_stmt          = "asset_use" , spacing , argument_list ;

autosave_stmt           = "autosave" ;
save_enable_stmt        = "save_enable" , spacing , argument_list ;
save_title_stmt         = "save_title" , spacing , argument_list ;

log_stmt                = "log" , spacing , argument_list ;
assert_stmt             = "assert" , spacing , expression ;
```

## 6.14 preload ブロック

```ebnf
preload_block   = "preload" , newline ,
                  { preload_item } ,
                  [ spacing ] , "endpreload" , newline ;

preload_item    = [ spacing ] , preload_kind , spacing , string ,
                  [ spacing , "," , spacing , argument_list ] ,
                  [ spacing ] , [ comment ] , newline ;

preload_kind    = "image" | "spine" | "audio" | "script" | "mask" ;
```

## 6.15 asset ブロック

```ebnf
asset_block     = "asset" , spacing , "id" , "=" , string , newline ,
                  { asset_property_line } ,
                  [ spacing ] , "endasset" , newline ;

asset_property_line
                = [ spacing ] , identifier , "=" , expression ,
                  [ spacing ] , [ comment ] , newline ;
```

## 6.16 resource_group ブロック

```ebnf
resource_group_block
                = "resource_group" , spacing , string , newline ,
                  { resource_group_item_line } ,
                  [ spacing ] , "endgroup" , newline ;

resource_group_item_line
                = [ spacing ] , preload_kind , spacing , string ,
                  [ spacing ] , [ comment ] , newline ;
```

## 6.17 animation ブロック

```ebnf
animation_block = "animation" , spacing , "id" , "=" , string , newline ,
                  { key_line } ,
                  [ spacing ] , "endanimation" , newline ;

key_line        = [ spacing ] , "key" , spacing , argument_list ,
                  [ spacing ] , [ comment ] , newline ;
```

## 6.18 parallel ブロック

```ebnf
parallel_block  = "parallel" , newline ,
                  { block_line } ,
                  [ spacing ] , "endparallel" , newline ;
```

---

# 7. コマンド一覧完全版

本章は、人間が書くスクリプトで使用可能なコマンド群を定義する。  
Kotonowa では、将来拡張を含め**100命令規模**の体系を想定する。

以下では、用途ごとに分類して示す。

---

## 7.1 制御系コマンド

### 7.1.1 `label`
ラベル定義。

```txt
label prologue_start
```

### 7.1.2 `jump`
指定ラベルへ移動。

```txt
jump school_morning
```

### 7.1.3 `call`
サブルーチン呼び出し。

```txt
call common_init
```

### 7.1.4 `return`
`call` の呼び出し元へ復帰。

```txt
return
```

### 7.1.5 `set`
変数代入。

```txt
set affection_alice = 10
set global cleared = true
```

### 7.1.6 `if / elseif / else / endif`
条件分岐。

```txt
if affection_alice >= 10
    text "かなり好感度が高い。"
elseif affection_alice >= 5
    text "まだ普通だ。"
else
    text "距離がある。"
endif
```

### 7.1.7 `while / endwhile`
ループ。

```txt
while count < 3
    set count = count + 1
endwhile
```

### 7.1.8 `choice / endchoice`
選択肢表示。

```txt
choice
    "アリスに話しかける" -> talk_alice
    "そのまま立ち去る" -> leave
endchoice
```

---

## 7.2 テキスト系コマンド

### 7.2.1 `name`
話者名表示。

```txt
name "アリス"
```

### 7.2.2 `text`
会話文表示。

```txt
text "おはようございます。"
```

### 7.2.3 `narration`
地の文表示。

```txt
narration "その日は静かな朝だった。"
```

### 7.2.4 `append`
同一ページへ追記。

```txt
append "……たぶん。"
```

### 7.2.5 `clear_text`
メッセージ内容をクリア。

```txt
clear_text
```

---

## 7.3 背景系コマンド

### 7.3.1 `bg`
静止画背景表示。

```txt
bg "bg/school/day.png"
bg file="bg/school/day.png", fade=500
```

### 7.3.2 `bg_spine`
Spine背景表示。

```txt
bg_spine file="bg/forest/forest.skel", atlas="bg/forest/forest.atlas", anim="idle", loop=true
```

### 7.3.3 `bg_clear`
背景クリア。

```txt
bg_clear
```

### 7.3.4 `bg_color`
単色背景表示。

```txt
bg_color color="#000000"
```

---

## 7.4 立ち絵系コマンド

### 7.4.1 `chara`
静止画立ち絵表示。

```txt
chara id="alice", file="chara/alice/smile.png", x=420, y=80, fade=200
```

### 7.4.2 `chara_spine`
Spine立ち絵表示。

```txt
chara_spine id="alice", file="chara/alice/alice.skel", atlas="chara/alice/alice.atlas", skin="default", anim="idle", loop=true, x=420, y=80
```

### 7.4.3 `chara_mod`
立ち絵差分・属性変更。

```txt
chara_mod id="alice", file="chara/alice/angry.png", fade=150
```

### 7.4.4 `chara_hide`
立ち絵非表示。

```txt
chara_hide id="alice", fade=300
```

### 7.4.5 `chara_focus`
指定立ち絵を強調。

```txt
chara_focus id="alice"
```

### 7.4.6 `chara_unfocus`
指定立ち絵の強調解除。

```txt
chara_unfocus id="alice"
```

---

## 7.5 イベントCG系コマンド

### 7.5.1 `eventcg`
静止画イベントCG表示。

```txt
eventcg file="event/ev001.png", fade=500
```

### 7.5.2 `eventcg_spine`
SpineイベントCG表示。

```txt
eventcg_spine id="ev001", file="event/ev001/ev001.skel", atlas="event/ev001/ev001.atlas", anim="idle"
```

### 7.5.3 `eventcg_mod`
イベントCG差分変更。

```txt
eventcg_mod id="ev001", anim="blush"
```

### 7.5.4 `eventcg_hide`
イベントCG非表示。

```txt
eventcg_hide fade=300
```

---

## 7.6 UI系コマンド

### 7.6.1 `ui_show`
静止画UI表示。

```txt
ui_show id="msg_frame", file="ui/message/frame.png", x=0, y=540
```

### 7.6.2 `ui_spine`
Spine UI表示。

```txt
ui_spine id="title_logo", file="ui/title/logo.skel", atlas="ui/title/logo.atlas", anim="idle"
```

### 7.6.3 `ui_hide`
UI非表示。

```txt
ui_hide id="msg_frame"
```

### 7.6.4 `ui_mod`
UI差分変更。

```txt
ui_mod id="cursor", file="ui/cursor_select.png"
```

### 7.6.5 `window_show`
メッセージウィンドウ表示。

```txt
window_show
```

### 7.6.6 `window_hide`
メッセージウィンドウ非表示。

```txt
window_hide
```

### 7.6.7 `window_opacity`
メッセージウィンドウ透明度変更。

```txt
window_opacity value=0.8
```

### 7.6.8 `namebox_show`
名前欄表示。

```txt
namebox_show
```

### 7.6.9 `namebox_hide`
名前欄非表示。

```txt
namebox_hide
```

### 7.6.10 `choice_config`
選択肢表示設定。

```txt
choice_config style="vertical", cursor="ui/cursor.png"
```

### 7.6.11 `auto_mode`
オートモード制御。

```txt
auto_mode true
```

### 7.6.12 `skip_mode`
スキップモード制御。

```txt
skip_mode false
```

---

## 7.7 共通表示オブジェクト操作コマンド

### 7.7.1 `show`
汎用表示。

```txt
show id="obj1", kind="prop", type="image", file="effect/light.png", x=300, y=100
```

### 7.7.2 `hide`
汎用非表示。

```txt
hide id="obj1", fade=200
```

### 7.7.3 `move`
移動。

```txt
move id="alice", x=500, y=90, time=300
```

### 7.7.4 `scale`
拡大縮小。

```txt
scale id="alice", scale=1.1, time=200
```

### 7.7.5 `rotate`
回転。

```txt
rotate id="alice", angle=10, time=200
```

### 7.7.6 `alpha`
透明度変更。

```txt
alpha id="alice", value=0.5, time=200
```

### 7.7.7 `layer`
レイヤー変更。

```txt
layer id="alice", value=40
```

### 7.7.8 `order`
同レイヤー内順序変更。

```txt
order id="alice", value=3
```

### 7.7.9 `color`
色補正。

```txt
color id="alice", value="#ffdddd"
```

### 7.7.10 `visible`
表示状態変更。

```txt
visible id="alice", value=false
```

---

## 7.8 音声系コマンド

### 7.8.1 `bgm`
BGM再生。

```txt
bgm "bgm/day.ogg"
bgm file="bgm/day.ogg", loop=true, fade=1000
```

### 7.8.2 `bgm_stop`
BGM停止。

```txt
bgm_stop fade=1000
```

### 7.8.3 `bgm_fade`
BGMフェード。

```txt
bgm_fade to=0, time=1000
```

### 7.8.4 `bgm_pause`
BGM一時停止。

```txt
bgm_pause
```

### 7.8.5 `bgm_resume`
BGM再開。

```txt
bgm_resume
```

### 7.8.6 `se`
効果音再生。

```txt
se "se/door_open.ogg"
```

### 7.8.7 `se_stop`
効果音停止。

```txt
se_stop channel="env"
```

### 7.8.8 `voice`
ボイス再生。

```txt
voice "voice/alice/ch01_001.ogg"
```

### 7.8.9 `voice_stop`
ボイス停止。

```txt
voice_stop
```

### 7.8.10 `voice_wait`
ボイス終了待ち。

```txt
voice_wait
```

### 7.8.11 `volume`
音量設定。

```txt
volume bgm=80, se=70, voice=100
```

---

## 7.9 画面切替・画面演出コマンド

### 7.9.1 `transition`
次の画面切替に適用するトランジション。

```txt
transition type="fade", time=500
transition type="wipe_left", time=700
transition type="mask", mask="mask/wipe/circle.png", time=1000
```

推奨 `type`:
- `fade`
- `crossfade`
- `wipe_left`
- `wipe_right`
- `wipe_up`
- `wipe_down`
- `slide_left`
- `slide_right`
- `curtain_open`
- `curtain_close`
- `iris_in`
- `iris_out`
- `blur`
- `pixelate`
- `mask`
- `custom`

### 7.9.2 `scene`
シーン単位切替。

```txt
scene bg="bg/night_room.png", trans="fade", time=1000
```

### 7.9.3 `fadein`
画面フェードイン。

```txt
fadein time=500
```

### 7.9.4 `fadeout`
画面フェードアウト。

```txt
fadeout time=500
```

### 7.9.5 `flash`
フラッシュ演出。

```txt
flash color="#ffffff", time=150
```

### 7.9.6 `shake`
画面または対象の振動。

```txt
shake power=12, time=300
shake id="alice", power=8, time=250
```

### 7.9.7 `tint`
画面色味変更。

```txt
tint color="#88aaff", time=500
```

---

## 7.10 汎用アニメーションコマンド

### 7.10.1 `anim`
任意プロパティ補間。

```txt
anim id="alice", x=500, alpha=1.0, time=400, easing="cubic_out"
```

### 7.10.2 `motion`
意味付きモーション。

```txt
motion id="alice", type="jump", height=40, time=300
motion screen=true, type="shake", power=20, time=400
```

推奨 `type`:
- `move`
- `slide`
- `jump`
- `bounce`
- `shake`
- `pulse`
- `zoom`
- `swing`
- `spin`
- `hover`
- `wave`
- `flicker`

### 7.10.3 `path`
パス移動。

```txt
path id="alice", type="bezier", p1="(100,100)", p2="(200,50)", p3="(400,120)", time=800
```

### 7.10.4 `playanim`
キーフレームアニメ再生。

```txt
playanim id="alice", anim="alice_enter"
```

### 7.10.5 `wait`
時間待機。

```txt
wait time=300
```

### 7.10.6 `wait_anim`
アニメ終了待ち。

```txt
wait_anim id="alice"
```

### 7.10.7 `stop_anim`
アニメ停止。

```txt
stop_anim id="alice"
```

### 7.10.8 `stop_motion`
モーション停止。

```txt
stop_motion id="alice"
```

### 7.10.9 `parallel / endparallel`
並列実行ブロック。

```txt
parallel
    anim id="alice", x=450, time=300
    anim id="bob", x=250, time=300
endparallel
```

### 7.10.10 `animation / endanimation`
キーフレームアニメ定義。

```txt
animation id="alice_in"
    key time=0, x=700, alpha=0
    key time=300, x=420, alpha=1
endanimation
```

---

## 7.11 Spine専用コマンド

### 7.11.1 `spine_anim`
Spineアニメ再生。

```txt
spine_anim id="alice", anim="talk", loop=true
```

### 7.11.2 `spine_queue`
次アニメ予約。

```txt
spine_queue id="alice", anim="smile", loop=false
```

### 7.11.3 `spine_skin`
スキン変更。

```txt
spine_skin id="alice", skin="winter"
```

### 7.11.4 `spine_stop`
Spineアニメ停止。

```txt
spine_stop id="alice", track=1
```

### 7.11.5 `spine_mix`
アニメ間ミックス設定。

```txt
spine_mix id="alice", from="idle", to="talk", time=0.15
```

### 7.11.6 `spine_wait_anim`
再生終了待ち。

```txt
spine_wait_anim id="alice", track=0
```

### 7.11.7 `spine_wait_event`
Spineイベント待ち。

```txt
spine_wait_event id="alice", event="blink_end"
```

---

## 7.12 リソース操作コマンド

### 7.12.1 `preload`
事前ロード。

```txt
preload type="image", file="bg/school/day.png"
```

ブロック形式:

```txt
preload
    image "bg/room.png"
    spine "chara/alice/alice.skel"
    audio "bgm/day.ogg"
endpreload
```

### 7.12.2 `preload_async`
非同期ロード。

```txt
preload_async file="bg/city.png"
```

### 7.12.3 `wait_load`
ロード完了待ち。

```txt
wait_load file="bg/city.png"
```

### 7.12.4 `unload`
リソース解放。

```txt
unload "bg/school/day.png"
unload type="image", file="bg/school/day.png"
```

### 7.12.5 `load_group`
リソースグループ読込。

```txt
load_group "school_scene"
```

### 7.12.6 `unload_group`
リソースグループ解放。

```txt
unload_group "school_scene"
```

### 7.12.7 `include`
外部スクリプト読込。

```txt
include "script/common/system.adv"
```

### 7.12.8 `call_script`
別スクリプト呼出。

```txt
call_script "script/chapter02/event01.adv"
```

### 7.12.9 `asset_use`
事前定義アセット使用。

```txt
asset_use id="alice", asset="alice_default"
```

### 7.12.10 `resource_group / endgroup`
リソース群定義。

```txt
resource_group "school_scene"
    image "bg/school/day.png"
    spine "chara/alice/alice.skel"
    audio "bgm/day.ogg"
endgroup
```

### 7.12.11 `asset / endasset`
アセット定義。

```txt
asset id="alice_default"
    type="spine"
    file="chara/alice/alice.skel"
    atlas="chara/alice/alice.atlas"
    skin="default"
endasset
```

---

## 7.13 セーブ関連コマンド

### 7.13.1 `autosave`
自動保存要求。

```txt
autosave
```

### 7.13.2 `save_enable`
セーブ可否切替。

```txt
save_enable false
```

### 7.13.3 `save_title`
セーブ画面用タイトル設定。

```txt
save_title "放課後の教室"
```

---

## 7.14 デバッグ支援コマンド

### 7.14.1 `log`
ログ出力。

```txt
log "scene start"
```

### 7.14.2 `assert`
条件検証。

```txt
assert affection_alice >= 0
```

---

# 8. コマンド体系一覧（100命令規模の分類表）

以下は Kotonowa スクリプト言語で想定するコマンド体系の一覧表である。  
将来拡張も含む体系として整理し、スクリプト記述側の棲み分けを明確化する。

## 8.1 制御系
- label
- jump
- call
- return
- set
- if
- elseif
- else
- endif
- while
- endwhile
- choice
- endchoice

## 8.2 テキスト系
- name
- text
- narration
- append
- clear_text

## 8.3 背景系
- bg
- bg_spine
- bg_clear
- bg_color

## 8.4 立ち絵系
- chara
- chara_spine
- chara_mod
- chara_hide
- chara_focus
- chara_unfocus

## 8.5 イベントCG系
- eventcg
- eventcg_spine
- eventcg_mod
- eventcg_hide

## 8.6 UI系
- ui_show
- ui_spine
- ui_hide
- ui_mod
- window_show
- window_hide
- window_opacity
- namebox_show
- namebox_hide
- choice_config
- auto_mode
- skip_mode

## 8.7 汎用表示操作系
- show
- hide
- move
- scale
- rotate
- alpha
- layer
- order
- color
- visible

## 8.8 サウンド系
- bgm
- bgm_stop
- bgm_fade
- bgm_pause
- bgm_resume
- se
- se_stop
- voice
- voice_stop
- voice_wait
- volume

## 8.9 画面切替・画面演出系
- transition
- scene
- fadein
- fadeout
- flash
- shake
- tint

## 8.10 汎用アニメーション系
- anim
- motion
- path
- playanim
- wait
- wait_anim
- stop_anim
- stop_motion
- parallel
- endparallel
- animation
- endanimation

## 8.11 Spine専用系
- spine_anim
- spine_queue
- spine_skin
- spine_stop
- spine_mix
- spine_wait_anim
- spine_wait_event

## 8.12 リソース操作系
- preload
- endpreload
- preload_async
- wait_load
- unload
- include
- call_script
- load_group
- unload_group
- resource_group
- endgroup
- asset
- endasset
- asset_use

## 8.13 セーブ系
- autosave
- save_enable
- save_title

## 8.14 デバッグ系
- log
- assert

上記分類で、**100命令規模の体系**として運用可能である。

---

# 9. 組み込み関数

式中で使用可能な代表関数を以下に示す。

## 9.1 `rand(min, max)`
乱数取得。

```txt
set dice = rand(1, 6)
```

## 9.2 `strlen(str)`
文字列長取得。

```txt
if strlen(player_name) > 8
```

## 9.3 `substr(str, start, len)`
部分文字列取得。

## 9.4 `int(x)`
整数変換。

## 9.5 `float(x)`
浮動小数変換。

## 9.6 `str(x)`
文字列変換。

## 9.7 `min(a, b)`
小さい方を返す。

## 9.8 `max(a, b)`
大きい方を返す。

## 9.9 `abs(x)`
絶対値取得。

## 9.10 `resource_exists(path)`
リソース存在確認。

```txt
if resource_exists("voice/alice/ch01_001.ogg")
    voice "voice/alice/ch01_001.ogg"
endif
```

---

# 10. シナリオライター向け記述ガイドライン

本章は、シナリオライター、演出担当、スクリプト記述担当者向けの運用ルールである。  
目的は、**読みやすく、保守しやすく、誤記や事故の少ないスクリプトを書くこと**にある。

## 10.1 基本原則

### 10.1.1 1行1命令を守る

良い例:

```txt
name "アリス"
text "おはようございます。"
```

悪い例:

```txt
name "アリス" text "おはようございます。"
```

### 10.1.2 ラベル名は意味が分かる名前にする

良い例:

```txt
label chapter01_start
label alice_route_day1
label bad_end_01
```

悪い例:

```txt
label a1
label test2
label temp
```

### 10.1.3 インデントは4スペース推奨

```txt
if affection_alice >= 10
    text "かなり仲が良い。"
else
    text "まだ距離がある。"
endif
```

---

## 10.2 テキスト記述の指針

### 10.2.1 会話は `text`
キャラクターの発話は `text` を使う。

```txt
name "アリス"
text "今日はいい天気ですね。"
```

### 10.2.2 地の文は `narration`
状況説明やモノローグは `narration` を使う。

```txt
narration "窓の外では雨が降り続いていた。"
```

### 10.2.3 `append` は同一ページ継続時のみ使う

```txt
text "彼女は少し間を置いて、"
append "静かにうなずいた。"
```

### 10.2.4 長文は適度に分割する
1命令が長すぎるとログ・翻訳・レビューで不利になる。

---

## 10.3 分岐記述の指針

### 10.3.1 条件は簡潔に保つ

避けたい例:

```txt
if affection_alice >= 7 && gift_done == true && seen_event_a == true && !bad_flag
```

推奨例:

```txt
set can_alice_good = affection_alice >= 7 && gift_done == true && seen_event_a == true && !bad_flag

if can_alice_good
    jump route_alice_good
endif
```

### 10.3.2 分岐ラベルは役割で命名する

```txt
label route_alice_good
label route_alice_normal
label route_alice_bad
```

---

## 10.4 選択肢記述の指針

### 10.4.1 プレイヤーが選ぶ自然な文にする

良い例:

```txt
choice
    "アリスに声をかける" -> talk_alice
    "そのまま立ち去る" -> leave
endchoice
```

避けたい例:

```txt
choice
    "好感度+1ルートへ行く" -> route1
    "通常ルート" -> route2
endchoice
```

### 10.4.2 条件付き選択肢は乱用しない
見えない条件を増やしすぎるとテストが難しくなる。

---

## 10.5 リソース記述の指針

### 10.5.1 必ず論理パスを使う

良い例:

```txt
bg "bg/school/day.png"
```

悪い例:

```txt
bg "C:\game\bg\school\day.png"
bg "bg\school\day.png"
```

### 10.5.2 アセット定義を積極的に使う
長いパスの繰り返し記述を避ける。

悪い例:

```txt
chara_spine id="alice", file="chara/alice/alice.skel", atlas="chara/alice/alice.atlas", skin="default"
chara_spine id="alice2", file="chara/alice/alice.skel", atlas="chara/alice/alice.atlas", skin="default"
```

良い例:

```txt
asset id="alice_default"
    type="spine"
    file="chara/alice/alice.skel"
    atlas="chara/alice/alice.atlas"
    skin="default"
endasset

asset_use id="alice", asset="alice_default"
```

### 10.5.3 preload は少し前で行う
重いシーン直前の一括ロードを避ける。

```txt
preload
    image "bg/park/evening.png"
    spine "chara/alice/alice.skel"
    audio "bgm/evening.ogg"
endpreload
```

---

## 10.6 画像とSpineの使い分け

### 静止画が向く場面
- 差分が少ない立ち絵
- 単純なUI
- 動き不要のイベントCG
- 軽量優先の場面

### Spineが向く場面
- 口パク
- 呼吸
- まばたき
- 動くUI
- 高演出シーン
- 軽く動くイベントCG

同一シーン内で、必要以上にSpineと静止画を混在させない方が管理しやすい。

---

## 10.7 演出記述の指針

### 10.7.1 まず表示し、その後に動かす

良い例:

```txt
chara id="alice", file="chara/alice/smile.png", x=700, y=80, alpha=0
anim id="alice", x=420, alpha=1, time=400, easing="cubic_out"
```

### 10.7.2 `anim` と `motion` を使い分ける

- 微調整は `anim`
- 意味付き動作は `motion`

```txt
anim id="alice", x=430, alpha=1.0, time=400, easing="cubic_out"
motion id="alice", type="jump", height=40, time=250
```

### 10.7.3 演出命令を使いすぎない
毎行アニメーションを入れるとテンポが悪くなる。

避けたい例:

```txt
anim id="alice", x=422, time=100
text "あ"
anim id="alice", x=420, time=100
text "の"
anim id="alice", x=421, time=100
text "ね"
```

---

## 10.8 トランジションの使い分け

### `fade`
自然な場面転換。最も基本。

### `wipe_left` / `wipe_right`
移動感のある転換。

### `mask`
夢、回想、特殊演出向け。

通常会話は `fade` を基準にし、特別な場面だけ別演出を使うと統一感が出る。

---

## 10.9 変数名・フラグ名の付け方

### フラグ
`flag_` 接頭辞を推奨。

```txt
set flag_seen_intro = true
set flag_got_key = false
```

### 数値変数
対象名を含める。

```txt
set affection_alice = affection_alice + 1
set trust_bob = 3
set chapter_index = 2
```

悪い例:

```txt
set a = 10
set x1 = true
set tmp = 2
```

---

## 10.10 ラベル分割の指針

1ラベルが長すぎると保守しづらい。  
目安として、**1シーン1ラベル**、または**1イベント1ラベル**程度で分ける。

```txt
label school_morning_start
label school_morning_choice
label school_morning_result
```

---

## 10.11 include の使いどころ

`include` は共通定義や共通処理に用いる。

向いている内容:
- 共通アセット定義
- UI定義
- 共通演出
- 共通初期化ラベル

向いていない内容:
- その章しか使わない短い会話

---

## 10.12 命令順の推奨

シーン冒頭は以下の順で書くと見やすい。

1. `label`
2. 必要なら `preload`
3. 背景 / BGM
4. キャラ配置
5. テキスト
6. 選択肢 / 分岐

例:

```txt
label school_morning

preload
    image "bg/classroom_day.png"
    audio "bgm/school_day.ogg"
endpreload

bg "bg/classroom_day.png", fade=500
bgm "bgm/school_day.ogg", loop=true

chara id="alice", file="chara/alice/smile.png", x=420, y=80

name "アリス"
text "おはようございます。"
```

---

## 10.13 シーンテンプレート例

```txt
label scene_example

# 次に使う素材を先読み
preload
    image "bg/park_evening.png"
    audio "bgm/evening.ogg"
endpreload

# シーン開始
bg "bg/school_day.png", fade=500
bgm "bgm/day.ogg", loop=true

chara_spine id="alice", file="chara/alice/alice.skel", atlas="chara/alice/alice.atlas", anim="idle", x=420, y=80

name "アリス"
spine_anim id="alice", anim="talk", loop=true
text "今日は授業のあと、少しお時間ありますか？"
spine_anim id="alice", anim="idle", loop=true

choice
    "もちろん、と答える" -> scene_yes
    "用事がある、と断る" -> scene_no
endchoice
```

---

## 10.14 禁止・非推奨事項

### 禁止
- 実OSパスの記述
- 予約語を識別子に使う
- 同一ラベル内で無秩序なジャンプを多用する
- 意味不明な短い変数名を濫用する

### 非推奨
- 演出命令の過剰使用
- 差分管理しにくい長大ラベル
- 同じ論理パスのベタ書きの繰り返し
- `if` の深すぎるネスト

---

## 10.15 レビュー時のチェック項目

提出前に最低限以下を確認する。

- ラベル名は分かりやすいか
- 選択肢文は自然か
- 不要な演出が多すぎないか
- 論理パスに誤記がないか
- 同じアセットをベタ書きしていないか
- 変数名・フラグ名が意味を持っているか
- `endif` / `endchoice` / `endpreload` などの閉じ忘れがないか

---

# 11. 予約語

以下は予約語であり、識別子として使用してはならない。

```txt
label
jump
call
return
set
local
scene
global
if
elseif
else
endif
while
endwhile
choice
endchoice
true
false
null
name
text
narration
append
clear_text
bg
bg_spine
chara
chara_spine
eventcg
eventcg_spine
ui_show
ui_spine
show
hide
move
scale
rotate
alpha
layer
bgm
se
voice
transition
anim
motion
wait
preload
include
asset
endasset
resource_group
endgroup
parallel
endparallel
animation
endanimation
```

---

# 12. エラー時の取り扱い方針（記述側）

本仕様書は実装仕様を扱わないが、記述側の運用上、以下の点を前提とする。

- リソースパス誤記は重大な記述ミスとして扱う
- ラベル未定義参照は重大な記述ミスとして扱う
- `endif` や `endchoice` の閉じ忘れは重大な記述ミスとして扱う
- 論理的に複雑な条件式は事前に `set` で整理することを推奨する

---

# 13. 最小実用コマンドセット

初期運用で最低限必須となるのは以下である。

```txt
label / jump / call / return
set
if / elseif / else / endif
choice / endchoice
name / text / narration
bg / chara / chara_hide
bgm / se / voice
transition / anim / motion / wait
include / preload / unload
autosave
```

---

# 14. 文書改訂履歴

|版数|内容|
|---|---|
|1.0|本チャット内で定義した EBNF / コマンド一覧 / 記述ガイドラインを網羅した詳細版として初版作成|

