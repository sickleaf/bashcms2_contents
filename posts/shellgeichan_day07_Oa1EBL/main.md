---
articleTitle: 国名分けっこトレーニング（○○するシェル芸ちゃん_7日目）
Keywords: ○○するシェル芸ちゃん, bash, grep,xargs,RANDOM, シェル芸
Copyright: (C) 2021 sickleaf
---


<br>

[○○するシェル芸ちゃん](../../?page=advent_shellgeichan)アドベントカレンダーの7日目です。
<br><br>

今回の問題は、手動で答えを出すのもロジックを組んで自動化するのもバカバカしい奴。

#### 問題

```
（１）列挙する国名パターンをファイルから読み込み、
「アル,イン,アル,イン」から「ゼンチン,ドネシア,ゼンチン,ドネシア」を出力して下さい。
（２）国名分けっこの出題側の文字列をランダムに生成し、その回答となる文字列もセットで出力して下さい。
```
ネタ元はこれ。自分で書いておいて何だけど**国名分けっこの出題側**って何やねん。


<iframe width="560" height="315" src="https://www.youtube.com/embed/b1J39WQFvSs" title="YouTube video player" frameborder="0" allow="accelerometer; autoplay; clipboard-write; encrypted-media; gyroscope; picture-in-picture" allowfullscreen></iframe>



##### 国名パターンファイル

```
アメ,リカ
アル,ゼンチン
アルゼン,チン
イギ,リス
インド,ネシア
インドネ,シア
イン^,ド
イン,ドネシア
中,国
ブラジ,ル
ニッ,ポン
韓,国
ドイ,ツ
ギリ,シャ
イタ,リア
イタリ,ア
モン,ゴル
モンゴ,ル
カナ,ダ
エジ,プト
エジプ,ト
スイ,ス
フラ,ンス
オース,トラリア
```

インドとインドネシアは2文字目で区切った場合を区別しないといけないので、イン-ドの場合はアクセントが上に来るイメージでハット（`イン^`）を入れています。

中国語の四声（2声）のイメージですが、脱力タイムズの「イン↑」が浮かんでしまったあなたは立派なバラエティバカ。ご愁傷さまです。


### 回答方針

出題の文字列がカンマ区切りなので、
パターンファイルで対応する行を1行ずつ出力し、対応する答えを抜き出せば回答の文字列が出てきます。

#### 単なるgrepでは拾えない

grepには標準出力をそのまま取り込むイディオム`grep -f -`があります。


##### 1行のみ

標準出力(`-`)をファイルとみなして取り込むとどうなるか。

```
$ echo "アル," | grep -f - country
アル,ゼンチン
```

カンマの後ろの部分だけ取れば出来そうですね。では2行では？

##### 2行以上（重複なし）

```
$ echo "アル,イン" | tr "," "\n" | sed "s;$;,;g" | grep -f- country
アル,ゼンチン
イン,ドネシア
```
カンマ区切りなのでtrを改行に置き換え、それで消えたカンマを各行の末尾に補完しています。

後は**ゼンチンドネシアゼンチンドネシア**が得られるようにすればカンマで区切って並べるだけ！　と思いきや…

##### 2行以上（重複あり）

```
$ echo "アル,イン,アル,イン" | tr "," "\n" | sed "s;$;,;g" | grep -f- country
アル,ゼンチン
イン,ドネシア
```

"アル,"と"イン,"だけが含まれる標準出力に対してパターンファイルに一致する行を出力するため、重複するものはまとめて出力されてしまいます。

重複があっても1行ごとにパターンファイルにマッチする行を表示するためには、1行ずつ処理しましょう。

### 回答例

#### （１）指定出題の回答作成

1行ずつ処理する例として`xargs`を使います。

`echo "アル,イン,アル,イン" |  tr "," "\n" | sed "s;$;,;g" | xargs -I@ sh -c "cat country | grep @ | cut -d, -f2- | tr '\n' ',' " | sed "s;.$;;g"`

`xargs`で各行を`@`に入れ、パターンファイルに対応する行を出力させます。

並んだら改行をカンマに変え（`tr '\n' ','`）、最後に末尾のカンマを外します。（`sed "s;.$;;g"`）

```
echo "アル,イン,アル,イン" |  tr "," "\n" | sed "s;$;,;g" | xargs -I@ sh -c "cat country | grep @ | cut -d, -f2- | tr '\n' ',' " | sed "s;.$;;g"
ゼンチン,ドネシア,ゼンチン,ドネシア
```

#### （２）ランダム生成

##### 出題文字列の生成

出題側の文字列もパターンファイルから作ります。`cat country | shuf -rn 4`で、4行ランダムに出力します。

```
$ cat country | shuf -rn 4
ニッ,ポン
アメ,リカ
イギ,リス
イタ,リア
$ cat country | shuf -rn 4
フラ,ンス
イン,ドネシア
ギリ,シャ
カナ,ダ
$ cat country | shuf -rn 4
韓,国
イタリ,ア
イン,ドネシア
イギ,リス
```

出題部分はカンマの前半部分（`cut -d, -f1`）をカンマ区切りで横並び(`tr "\n" , | sed -z "s;$;\n;g"`)にすれば一応出来ますが、もう少しひねりましょう。

##### 文字列の長さを操作する

4行固定だと味気ないので、最低10個、最大30個問題の文字列が並ぶようにしてみます。

変えるべきは`shuf -rn 4`の数字部分。

10～30に相当する数字が得るため、ここでは[bashが提供する変数](https://linuxjm.osdn.jp/html/GNU_bash/man1/bash.1.html#lbAW)`RANDOM`を使います。

`RANDOM`は呼ばれるたびに0から32767までのランダムな整数が生成されるので、`RANDOM`を20で割った余りに10を足す（0～20 + 10）と10～30が得られます。

```
$ cat country | shuf -rn$((10+RANDOM%20)) | cut -d, -f1 | tr "\n" "," | sed -z "s;,$;\n;g"
イン^,イン^,ブラジ,ブラジ,イタ,中,イタ,イギ,イギ,イタ,インド,アル,フラ,インドネ,モンゴ,韓,イギ,オース,フラ,ニッ,エジ,韓
$ cat country | shuf -rn$((10+RANDOM%20)) | cut -d, -f1 | tr "\n" "," | sed -z "s;,$;\n;g"
オース,アルゼン,アル,韓,イタリ,ブラジ,ニッ,イン^,アメ,中,スイ,イタ,アルゼン,イン^,フラ,ドイ,韓,カナ,イン^,ドイ,イン^,アメ,イギ,イン,インドネ,イタ,インド,インドネ
$ cat country | shuf -rn$((10+RANDOM%20)) | cut -d, -f1 | tr "\n" "," | sed -z "s;,$;\n;g"
エジプ,中,イン^,韓,スイ,イン,イン^,オース,ドイ,ニッ,モンゴ,アメ,エジ,イタリ
$ cat country | shuf -rn$((10+RANDOM%20)) | cut -d, -f1 | tr "\n" "," | sed -z "s;,$;\n;g"
アメ,イン,オース,モンゴ,モン,モンゴ,アメ,韓,イタリ,オース,モンゴ,インドネ,カナ,スイ,モン,アルゼン,インドネ,ギリ
```

##### 出題・回答をセットで出力

ランダムで得られた出題文字列を変数にセット・出力し、回答を得るワンライナーに突っ込めば完成です。

`str=$(cat country | shuf -rn$((10+RANDOM%20)) | cut -d, -f1 | tr "\n" "," | sed -z "s;,$;\n;g"); echo $str; echo $str |  tr "," "\n" | sed "s;$;,;g" | xargs -I@ sh -c "cat country | grep @ | cut -d, -f2- | tr '\n' ',' " | sed "s;.$;;g"`

```
$ str=$(cat country | shuf -rn$((10+RANDOM%20)) | cut -d, -f1 | tr "\n" "," | sed -z "s;,$;\n;g"); echo $str; echo $str |  tr "," "\n" | sed "s;$;,;g" | xargs -I@ sh -c "cat country | grep @ | cut -d, -f2- | tr '\n' ',' " | sed "s;.$;;g"
イギ,フラ,イギ,ドイ,イン,オース,ギリ,インドネ,中,イギ,イン^,イタ,ギリ,ギリ,ドイ,イタ,イン,カナ,イタ,ギリ,ニッ,ブラジ,アルゼン
リス,ンス,リス,ツ,ドネシア,トラリア,シャ,シア,国,リス,ド,リア,シャ,シャ,ツ,リア,ドネシア,ダ,リア,シャ,ポン,ル,チン
$ str=$(cat country | shuf -rn$((10+RANDOM%20)) | cut -d, -f1 | tr "\n" "," | sed -z "s;,$;\n;g"); echo $str; echo $str |  tr "," "\n" | sed "s;$;,;g" | xargs -I@ sh -c "cat country | grep @ | cut -d, -f2- | tr '\n' ',' " | sed "s;.$;;g"
モン,ドイ,カナ,ニッ,カナ,フラ,オース,イタリ,ブラジ,中,イン^,ニッ,インドネ,オース,イタ,イタリ,韓,インド,オース,アル
ゴル,ツ,ダ,ポン,ダ,ンス,トラリア,ア,ル,国,ド,ポン,シア,トラリア,リア,ア,国,ネシア,トラリア,ゼンチン
$ str=$(cat country | shuf -rn$((10+RANDOM%20)) | cut -d, -f1 | tr "\n" "," | sed -z "s;,$;\n;g"); echo $str; echo $str |  tr "," "\n" | sed "s;$;,;g" | xargs -I@ sh -c "cat country | grep @ | cut -d, -f2- | tr '\n' ',' " | sed "s;.$;;g"
モンゴ,ギリ,イタリ,アメ,イタ,フラ,イタリ,ニッ,ドイ,中,イン,エジプ,モンゴ,アル,オース,アルゼン
ル,シャ,ア,リカ,リア,ンス,ア,ポン,ツ,国,ドネシア,ト,ル,ゼンチン,トラリア,チン
$ str=$(cat country | shuf -rn$((10+RANDOM%20)) | cut -d, -f1 | tr "\n" "," | sed -z "s;,$;\n;g"); echo $str; echo $str |  tr "," "\n" | sed "s;$;,;g" | xargs -I@ sh -c "cat country | grep @ | cut -d, -f2- | tr '\n' ',' " | sed "s;.$;;g"
エジ,中,アメ,韓,ブラジ,ブラジ,カナ,スイ,イタリ,モンゴ,イギ,インドネ,エジ
プト,国,リカ,国,ル,ル,ダ,ス,ア,ル,リス,シア,プト
```

カンマ区切りにして並べる部分は繰り返し使うので、本当は関数にするべきですね。やらないけど。


#### ちなみに

元々4分頃に出てくるこの問題が解きたくて考え始めたのでした。

`アル,アル,イン,イン,イン,イン,アル,アル,イン,イン,アル,アル,オース,オース,エジ,中,ギリ,モン,イタ,ドイ,イタ,ドイ,イン,イン,イン,アル`

```
$ echo "アル,アル,イン,イン,イン,イン,アル,アル,イン,イン,アル,アル,オース,オース,エジ,中,ギリ,モン,イタ,ドイ,イタ,ドイ,イン,イン,イン,アル" |  tr "," "\n" | sed "s;$;,;g" | xargs -I@ sh -c "cat country | grep @ | cut -d, -f2- | tr '\n' ',' " | sed "s;.$;;g"
ゼンチン,ゼンチン,ドネシア,ドネシア,ドネシア,ドネシア,ゼンチン,ゼンチン,ドネシア,ドネシア,ゼンチン,ゼンチン,トラリア,トラリア,プト,国,シャ,ゴル,リア,ツ,リア,ツ,ドネシア,ドネシア,ドネシア,ゼンチン
```

型を破りたい相方たち（くりぃむナンタラ）で、出来ない前提の出題を見事に解くみたいなことをやったりしないかしら。


### ○○する芸能ちゃんブロック

出題がまる芸ちゃんと関係ない時はただただオススメを貼っていきます。

ただ見る側でも楽しめる良い内容です。

<div class="youtube">
<iframe width="560" height="315" src="https://www.youtube.com/embed/c_wCJHQ04r8" title="YouTube video player" frameborder="0" allow="accelerometer; autoplay; clipboard-write; encrypted-media; gyroscope; picture-in-picture" allowfullscreen></iframe>
</div>

しかし何かの形でクイズグランプリ復活しないかなあ。色んな人が持ち寄るジャンルで10～50ポイントの問題を作って、生放送で時間ギリギリまで出題するストロングスタイルの番組が見たい。

今のところそれに最も近いのは**地下クイズ決定戦**なんだけど、地上でも見たい。


<style>
.youtube {
  position: relative;
  width: 100%;
  padding-top: 56.25%;
}
.youtube iframe {
  position: absolute;
  top: 0;
  right: 0;
  width: 100% !important;
  height: 100% !important;
}
</style>


<!--


## 書籍近影

### 別ページへのリンク
[memolink](../../?page=memolink)

### 複数行引用
<blockquote></blockquote>

#### CSS埋め込み

<style>
.col-md-9 > iframe {
	width:100% !important;
	height:500px !important;}
</style>

<font style="color:#FFFFFF;"></font>

#### JS埋め込み

<script type="text/javascript">
</script>

-->
