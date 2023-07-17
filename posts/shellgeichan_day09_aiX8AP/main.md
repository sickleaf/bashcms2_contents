
---
articleTitle: あべこべを作ろう（○○するシェル芸ちゃん_9日目）
Keywords: ○○するシェル芸ちゃん, bash, シェル芸
Copyright: (C) 2021 sickleaf
---


<br>

[○○するシェル芸ちゃん](../../?page=advent_shellgeichan)アドベントカレンダーの9日目です。
<br><br>

**テレビをよく見ていたら何故か頭に入っている業界用語「てれこ」**。

入れ違い・互い違いになっているという意味ですが、”テープレコーダーのA面とB面を間違えたことに由来するんですよね”と書こうとして一応調べてみたら、どうやら歌舞伎由来の言葉のようですね。

芸能ガセビアを堂々と書いて[嘘つき](https://twitter.com/marugeichan/status/1413312937247211524)になるところでした。危ない危ない。


#### 問題

```
芸人のコンビの中から、（１）苗字をもじって作られたコンビ名と、（２）その表記に従った各メンバーのフルネーム（もしくはそれに準ずる名前）、そして（３）コンビ名のうち苗字と関係のない繋ぎの文字をカンマ区切りにセットしたデータがあります。
このデータを用いて、コンビ名の”じゃない方”を生成するワンライナーを作って下さい。
（例：宮下草薙→兼史鷹航基　なすなかにし→あきゆきしげき）
```

##### 問題データ(`cat data`)
```
宮下@草薙,宮下兼史鷹,草薙航基,,
なす@なかにし,なすあきゆき,なかにししげき,,
タカ@トシ,スズキタカヒロ,ミウラトシカズ,アンド
イワイ@イガワ,イワイジョニオ,イガワシュウジ,,
クワバタ@オハラ,クワバタリエ,オハラマサコ,,
アルコ@ピース,アルコールウェル,ピースチャイルド,&
イシバシ@ハザマ,イシバシタカヒサ,ハザマヨウヘイ,,
```

「**お互いの苗字をもじった芸名で有名なコンビ、宮下草薙を除くとオンバト世代が最後説**」ありませんかね。

（わらふぢなるおは口笛なるおが完全に芸名だからノーカウント。反例求む。）

### 回答方針

問題データでコンビ名の両方の要素の区切りを入れているので、割と単純です。

宮下草薙の例に取って考えてみると、2列目（宮下兼史鷹）から1列目の`@`の前（宮下）の文字列を削ったものと、3列目（草薙航基）から1列目の`@`の後（草薙）の文字列を削ったものを3列目で結合すれば完了。

行ごとにカンマ区切りで入っているので、行のデータ格納はカンマ区切りの配列読み込み、文字列を削る操作はパラメータ展開によって実現出来ます。

#### カンマ区切りの配列読み込み

`while IFS=, read -a line`

```
$ cat data | while IFS=, read -a line; do echo ${line[0]}; done
宮下@草薙
なす@なかにし
タカ@トシ
イワイ@イガワ
クワバタ@オハラ
アルコ@ピース
イシバシ@ハザマ
```

#### 文字列を削る操作

[パラメータ展開](https://linuxjm.osdn.jp/html/GNU_bash/man1/bash.1.html#lbBB)

パラメータ展開の、置換機能を使います。

>${parameter/pattern/string}
>
>パターンの置換。 pattern が展開され、 パス名展開の場合と同じようなパターンを作ります。 parameter の展開が行われ、 その値のうち pattern に最長一致する部分が string に置換されます。 pattern が / で始まる場合には、pattern に マッチした部分は全て string に置換されます。 そうでない場合には、最初にマッチした部分だけが置換されます。 

先ほどの`${line[0]}`から`@`を取り除く例を書いてみます。

```
$ cat data | while IFS=, read -a line; do echo ${line[0]/@/}; done
宮下草薙
なすなかにし
タカトシ
イワイイガワ
クワバタオハラ
アルコピース
イシバシハザマ
```

これを組み合わせれば必要な文字列が得られます。

### 回答例

`cat data | while IFS=, read -a line; do s1=$(echo ${line[0]} | cut -d@ -f1); s2=$(echo ${line[0]} | cut -d@ -f2); echo ${line[1]/$s1/}${line[3]}${line[2]/$s2/}; done;`

`${line[0]}`の`@`前後を`s1`,`s2`とし、1人目のコンビ名に使う部分を取り除いた文字列は`${line[1]/$s1/}`とすることが出来ます。2人目も同様。

あとはこれを`${line[3]}`で結合すれば完成です。

```
$ cat data | while IFS=, read -a line; do s1=$(echo ${line[0]} | cut -d@ -f1); s2=$(echo ${line[0]} | cut -d@ -f2); echo ${line[1]/$s1/}${line[3]}${line[2]/$s2/}; done;
兼史鷹航基
あきゆきしげき
スズキヒロアンドミウラカズ
ジョニオシュウジ
リエマサコ
ールウェル&チャイルド
タカヒサヨウヘイ
```


<blockquote class="twitter-tweet"><p lang="ja" dir="ltr">兼史鷹航基<br>スズキヒロアンドミウラカズ<br>タカヒサヨウヘイ<br>ジョニオシュウジ <a href="https://t.co/DHgtIfFNwo">https://t.co/DHgtIfFNwo</a></p>&mdash; シェル芸bot (@minyoruminyon) <a href="https://twitter.com/minyoruminyon/status/1469818517407301637?ref_src=twsrc%5Etfw">December 11, 2021</a></blockquote> <script async src="https://platform.twitter.com/widgets.js" charset="utf-8"></script>

この記事のタイトル、本当は「あべこべを作ろう」の文字列を入れ替えたかったのですが、いくら調べても「あべこべ」の適切な反対語が見つからなかったので諦めました。。


### ○○する芸能ちゃんブロック

フォーマットはこの動画から拝借したものでした。

最後の問題が凄い。。

あと普段企画側のさくどーさんがプレイヤーに回ると、プレイヤーとしての良さが出てるなあと毎回思うなど。

<div class="youtube">
<iframe width="560" height="315" src="https://www.youtube.com/embed/1rfREay6Vu0" title="YouTube video player" frameborder="0" allow="accelerometer; autoplay; clipboard-write; encrypted-media; gyroscope; picture-in-picture" allowfullscreen></iframe>
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

