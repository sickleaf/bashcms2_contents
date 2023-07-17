---
articleTitle: ソニー損保七対子（○○するシェル芸ちゃん_4日目）
Keywords: ○○するシェル芸ちゃん, bash,シェル芸
Copyright: (C) 2021 sickleaf
---

<br>

[○○するシェル芸ちゃん](../../?page=advent_shellgeichan)アドベントカレンダーの4日目です。
<br>

2,3日目が重たすぎたので今日は軽いです。

#### 問題

```
問. 「ソニー損保」の文字列から、「ソソニニーー損損保保」を作成して下さい。
```

モチーフはコレ。

![sonysonpo](https://user-images.githubusercontent.com/52893720/144827492-29d54bba-05fe-4e67-b403-4edb526ded38.png)


<blockquote class=""><p lang="ja" dir="ltr">＼Twitter限定　秘蔵映像🎥／<br><br>瑞原明奈プロ(<a href="https://twitter.com/akn19mj?ref_src=twsrc%5Etfw">@akn19mj</a>)にご出演いただいた、ソニー損保の大好きCM！ラストポーズの別バージョンを公開します✨<br><br>こちらも素敵でしたので、ぜひご覧ください🀄<a href="https://twitter.com/unext_pirates?ref_src=twsrc%5Etfw">@unext_pirates</a><a href="https://twitter.com/hashtag/M%E3%83%AA%E3%83%BC%E3%82%B0?src=hash&amp;ref_src=twsrc%5Etfw">#Mリーグ</a></p>&mdash; ソニー損保の自動車保険(公式) (@sonysonpo_jp) <a href="https://twitter.com/sonysonpo_jp/status/1463787038352814090?ref_src=twsrc%5Etfw">November 25, 2021</a></blockquote> <script async src="https://platform.twitter.com/widgets.js" charset="utf-8"></script>


### 回答方針

文字を複製する前後で、「1文字ずつ縦にバラす」「縦に並んだ文字を横に直す」といった操作が入ることがあります。やってもやらなくても良いです。


### 回答例

これの問題については、まる芸ちゃんから飛んできた皆様も[こちらのサイト](https://websh.jiro4989.com/)でinputに貼り付けてRunを押すと同様に楽しめるので、探り探りやってもらえると幸い。

回答例いくつも出せるので、#シェル芸 でもっと変態な答えが出たりしないかなとワクワクしながら待っております。


#### 回答例１：バラす→複製→横に並べる


##### （１）：sedとxargs

`echo ソニー損保 | grep -o . | sed "p" | xargs -I@ printf @`


縦に並べて、1行ずつ複製（`sed p`は各行を出力するコマンド）して、各行を横に並べる。

##### （２）：awk

`echo ソニー損保 | grep -o . | awk '{printf $0$0}'`


縦に並べて、その行の内容と同じものを改行せずに並べる。

多分これが最も平凡でベタなワンライナーのはず。


#### 回答例２：左から１文字ずつ増やしていく（awk）

`echo ソニー損保 | awk -v FS=""  '{for(i=1;i<=NF;i++)printf "%s%s", $i, $i}'`

左から１文字ずつ読んでいって、複製して出力する。回答例１の（２）と類似しているが、こっちは縦に並べていないので一応別扱い。

処理上仕方ない時は書くけど、この程度の処理でループはあまり書きたくない。


#### 回答例３：左から１文字ずつ増やしていく（sed）

`echo ソニー損保 | sed -r "s;(.);\1\1;g"`

浮かんだ回答例の中で一番スマートっぽい回答例。

#### 回答例４：決め打ち

`echo ソニー損保 | awk -v FS="" '{printf $1$1$2$2$3$3$4$4$5$5}'`

泥臭すぎる回答。文字増えた時どうするんだっていう。


<!--
### 変態回答

-->


<blockquote class="twitter-tweet"><p lang="ja" dir="ltr">ソソニニーー損損保保<br>ソソニニーー損損保保<br>ソソニニーー損損保保<br>ソソニニーー損損保保 <a href="https://t.co/3ev57n2ipD">https://t.co/3ev57n2ipD</a></p>&mdash; シェル芸bot (@minyoruminyon) <a href="https://twitter.com/minyoruminyon/status/1467813508759044098?ref_src=twsrc%5Etfw">December 6, 2021</a></blockquote> <script async src="https://platform.twitter.com/widgets.js" charset="utf-8"></script>



### ○○する芸能ちゃんブロック

**Mリーグ公式実況の松嶋桃さんが普通に出てる**回です。っていうか日髙大介さんが出ているのも物凄いことなんだが。。

企画が強く、画も強く、テロップの再現度が高い。往年のニコ動なら`もっと評価されるべき`タグが付いていたに違いない。

<div class="youtube">
<iframe width="560" height="315" src="https://www.youtube.com/embed/wvZi_L3Mgog" title="YouTube video player" frameborder="0" allow="accelerometer; autoplay; clipboard-write; encrypted-media; gyroscope; picture-in-picture" allowfullscreen></iframe>
</div>

嘘がしれっと挟まれているところ込みでとてもテレビっぽいところが好きです。

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

