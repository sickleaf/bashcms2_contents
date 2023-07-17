---
articleTitle: バビ語エンコーダ（○○するシェル芸ちゃん_21日目）
Keywords: ○○するシェル芸ちゃん, bash, シェル芸
Copyright: (C) 2021 sickleaf
---


<br>

[○○するシェル芸ちゃん](../../?page=advent_shellgeichan)アドベントカレンダーの21日目です。
<br><br>

[第57回シェル芸勉強会](https://usptomo.doorkeeper.jp/events/130621)の[LTスライド](https://speakerdeck.com/sickleaf/surusieruyun-tiyan-falseshao-jie)を作りながら書いていたはずが年が明けている…。

## 問題

```
任意の平仮名の文字列について、各文字の後にばびぶべぼを入れる「バビ語」による暗号化を施して下さい。バビ語のルールは下記の通りです。
・母音に会う「ばびぶべぼ」を各文字の後ろに入れる
・「ん」は「ぶ」に、「ゃゅょ」は前の文字とセットで変換
・「っ」は無視、「ー」はひとつ前の文字の母音に揃える
```


## 回答方針

大方針として、元の文とバビ語に相当する文字列を1文字ずつ縦に並べ、プロセス置換と`paste`で両方並べて作ります。

```
$ w=わくらば; wd=ばぶばば; paste -d","  <(echo $w | grep -o .) <(echo $wd | grep -o . )
わ,ば
く,ぶ
ら,ば
ば,ば
```

他にもやり方はありそうですが、バビ語の文字列を作る場合も元の文を使うため変数に格納します。ここではバビ語の文字列を`wd`としていますが、これは`w`からどうにか作る必要があります。

また、拗音や長音、促音について対応できているか確認するため、例文には「シャーベット」を使います。


### 拗音を前の文字とセットにする

```
$ w="しゃーべっと"; echo $w | sed -e "s/./& /g; s/.\([ゃゅょ]\)./\1 /g" | tr " " "\n" 
しゃ
ー
べ
っ
と

```

sedの`"s/./& /g;"`で1文字ずつ後ろに空白を入れていますが、"ゃゅょ"は前の文字とセットにするため、`"s/.\([ゃゅょ]\)./\1 /g"`によって拗音の前の空白を詰めます。

### 長音を一つ前の文字の母音に置き換える

元の文に長音が含まれていた場合、置き換えが必要です。


改行をまたいで一つ前の母音だけ持ってくるため、一度`uconv`でローマ字に変換します。

```
w="しゃーべっと"; echo $w |sed -e "s/./& /g; s/.\([ゃゅょ]\)./\1 /g" | tr " " "\n" | uconv -x latin | sed -Ez "s;(.)\nー;\1\n\1;g" | uconv -x hiragana
しゃ
あ
べ
っ
と
```

`sed -Ez "s;(.)\nー;\1\n\1;g"`によって「ー」の前にある母音を取ります。

### バビ語用にエンコードする文字列を作る

先ほどまでの処理は「ばびぶべぼ」を挟み込む元の文の加工ですが、挟み込む側のバビ語要素も途中まで同じ流れで作ります。

元の文は`uconv -x higagana`で終わりですが、バビ語用の文字列は`uconv`ではなく`sed`で作ります。



```
$ w="しゃーべっと"; echo $w |sed -e "s/./& /g; s/.\([ゃゅょ]\)./\1 /g" | tr " " "\n" | uconv -x latin | sed -Ez "s;(.)\nー;\1\n\1;g" | sed "y/aiueo/ばびぶべぼ/"
shば
ば
bべ
~tsぶ
tぼ
```

あとは促音を含む行やaiueo以外のアルファベットのカット、そしてこの例では出ませんが「ん」を「ぶ」に置き換える処理を追加します。

```
$ w="しゃーべっと"; echo $w |sed -e "s/./& /g; s/.\([ゃゅょ]\)./\1 /g" | tr " " "\n" | uconv -x latin | sed -Ez "s;(.)\nー;\1\n\1;g" | sed "y/aiueo/ばびぶべぼ/" | sed "s/^~.*//g; s/^n$/ぶ/g; s/[a-z]//g"
ば
ば
べ

ぼ
```

## 回答例

元の文とバビ語用文字列をそれぞれ準備したので、これを`paste`で並べて1行に並べればOKです。


`w=まるまるするげいのうちゃん;paste -d","  <(echo $w |sed -e "s/./& /g; s/.\([ゃゅょ]\)./\1 /g" | tr " " "\n" | uconv -x latin | sed -Ez "s;(.)\nー;\1\n\1;g" | uconv -x hiragana) <(echo $w |sed -e "s/./& /g; s/.\([ゃゅょ]\)./\1 /g" | tr " " "\n" | uconv -x latin | sed -Ez "s;(.)\nー;\1\n\1;g" | sed "y/aiueo/ばびぶべぼ/" | sed "s/^~.*//g; s/^n$/ぶ/g; s/[a-z]//g")| tr -d "\n" | sed "s;,;;g"`

```
$ w=まるまるするげいのうちゃん;paste -d","  <(echo $w |sed -e [ゃゅょ]\)./\1 /g" | tr " " "\n" | uconv -x latin | sed -Ez "s;(.)\nー;\1\n\1;g" | uconv -x hiragana) <(echo $w |sed -e "s/./& /g; s/.\([ゃゅょ]\)./\1 /g" | tr " " "\n" | uconv -x latin | sed -Ez "s;(.)\nー;\1\n\1;g" | sed "y/aiueo/ばびぶべぼ/" | sed "s/^~.*//g; s/^n$/ぶ/g; s/[a-z]//g")| tr -d "\n" | sed "s;,;;g"
まばるぶまばるぶすぶるぶげべいびのぼうぶちゃばんぶ
```

<blockquote class="twitter-tweet"><p lang="ja" dir="ltr">これをこうじゃ！ <a href="https://twitter.com/hashtag/%E3%81%BE%E3%82%8B%E8%8A%B8%E3%81%A1%E3%82%83%E3%82%93?src=hash&amp;ref_src=twsrc%5Etfw">#まる芸ちゃん</a><a href="https://t.co/tLXnXEF3W6">https://t.co/tLXnXEF3W6</a></p>&mdash; 病葉(Yu Kidawara) (@sickleaf3) <a href="https://twitter.com/sickleaf3/status/1416048782823755779?ref_src=twsrc%5Etfw">July 16, 2021</a></blockquote> <script async src="https://platform.twitter.com/widgets.js" charset="utf-8"></script>


あー疲れた。

## ○○する芸能ちゃんブロック

キョコロヒー、番組開始から半年分まで全くバビ語に触れず、出たけど[未公開](https://www.youtube.com/watch?v=u7mb10mQXzM&list=PLRlGJA3aVEXA7C2CSygke6YO6jRjpY5Ah&index=16)行きってどれだけ撮れ高あるんだっていう。

<div class="youtube">
<iframe width="560" height="315" src="https://www.youtube.com/embed/uoVSkw6Grfg" title="YouTube video player" frameborder="0" allow="accelerometer; autoplay; clipboard-write; encrypted-media; gyroscope; picture-in-picture" allowfullscreen></iframe>
</div>

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
