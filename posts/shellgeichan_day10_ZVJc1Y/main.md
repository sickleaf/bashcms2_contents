---
articleTitle: 世にも奇妙な数字を載せたいよ〜！（○○するシェル芸ちゃん_10日目）
Keywords: ○○するシェル芸ちゃん, bash, シェル芸
Copyright: (C) 2021 sickleaf
---


<br>

[○○するシェル芸ちゃん](../../?page=advent_shellgeichan)アドベントカレンダーの10日目です。
<br><br>

私が覚えたてのシェル芸を早速使います。というかこのネタのためだけに勉強したまである。

#### 問題

```
convertやtextimgを使い、人物のイラストの額に何か数字を載せて下さい。
```

[福沢諭吉の似顔絵イラスト](https://www.irasutoya.com/2013/08/blog-post_6401.html)

<img src="https://user-images.githubusercontent.com/5824061/145985910-f34f59a0-a3b9-461c-8624-136dc3748766.png"  width=20%>


**諭吉の額に10000を載せる問題です。**<font style="color:#FFFFFF;">何でそんなことせなあかんねん。</font>


### 回答方針

`convert`はimagemagickに含まれるコマンド、`textimg`は[jiro](https://github.com/jiro4989)さんによるGo製のコマンドで、端末のテキストを画像に変換することが出来ます。

予め用意されているデータはイラストデータのみなので、数字を画像化したデータを準備し重ね合わせる操作が必要になります。今回実行する手順は以下の通りです。

１．背景を額の色に合わせた状態で、載せる数字を画像にする

２．数字の画像を`convert`に取り込む

３．`convert`で取り込んだ画像とイラストを重ね合わせる

今回実装する方針は[休みの日の自習メモ](../../?post=studylog_20211212)とほぼ同じです。ここではそこから色々抜粋します。

#### 額に載せる数字を画像にする

`textimg`を使います。コマンドのオプションにより、文字サイズや色、背景色を指定します。額の色はRGBで(249,203,160)みたいです。


`echo 10000 | textimg -F25 --background 249,203,160,0 --foreground black`

[webshリンク](https://websh.jiro4989.com/?code=echo%2010000%20%7C%20textimg%20-F25%20--background%20249%2C203%2C160%2C0%20--foreground%20black%20-s)

画像で出力するために`-s`オプションを末尾に入れていますが、繋げる場合は不要みたいです。

#### `convert`の実行

結果だけ書きます。過程は自習メモにて。

`convert -  -write mpr:a +delete /media/0 mpr:a -geometry +125+50 -composite /images/x.png`

webshやシェル芸botでは画像取り込みは`/media/0`で指定しますが、ローカル環境の場合は実際のパスを指定するはずです（試していません）

### 回答例

`echo 10000 | textimg -F25 --background 249,203,160,0 --foreground black | convert -  -write mpr:a +delete /media/0 mpr:a -geometry +125+50 -composite /images/x.png`

<blockquote class="twitter-tweet"><p lang="und" dir="ltr"><a href="https://t.co/FM34oaHJiV">https://t.co/FM34oaHJiV</a> <a href="https://t.co/vsW6ScW7pK">pic.twitter.com/vsW6ScW7pK</a></p>&mdash; シェル芸bot (@minyoruminyon) <a href="https://twitter.com/minyoruminyon/status/1469908952125419522?ref_src=twsrc%5Etfw">December 12, 2021</a></blockquote> <script async src="https://platform.twitter.com/widgets.js" charset="utf-8"></script>


### シェル芸ブロック

画像シェル芸は先駆者が物凄い居ましてねえ。今日はそのほんの一部をご紹介します。

元のワンライナーは**なるほどわからん**というものばかり。そして強烈なインパクトを放っています。

<blockquote class="twitter-tweet"><p lang="und" dir="ltr"><a href="https://t.co/DlysyjRUJf">https://t.co/DlysyjRUJf</a> <a href="https://t.co/Ruc2ySftU3">pic.twitter.com/Ruc2ySftU3</a></p>&mdash; シェル芸bot (@minyoruminyon) <a href="https://twitter.com/minyoruminyon/status/1222186488361697282?ref_src=twsrc%5Etfw">January 28, 2020</a></blockquote> <script async src="https://platform.twitter.com/widgets.js" charset="utf-8"></script>

<blockquote class="twitter-tweet"><p lang="ja" dir="ltr">＿人人人人人人人人人人＿<br>＞　一般人の想像する　＜<br>＞　ハッキング　　　　＜<br>￣Y^Y^Y^Y^Y^Y^Y^Y^Y^Y^￣<br> <a href="https://t.co/86A1WyBJc8">https://t.co/86A1WyBJc8</a> <a href="https://t.co/hhfeTbcDo1">pic.twitter.com/hhfeTbcDo1</a></p>&mdash; シェル芸bot (@minyoruminyon) <a href="https://twitter.com/minyoruminyon/status/1339565830640582658?ref_src=twsrc%5Etfw">December 17, 2020</a></blockquote> <script async src="https://platform.twitter.com/widgets.js" charset="utf-8"></script>



<blockquote class="twitter-tweet"><p lang="und" dir="ltr"><a href="https://t.co/mB0YcJfoUB">https://t.co/mB0YcJfoUB</a> <a href="https://t.co/KZk7qxzF5Y">pic.twitter.com/KZk7qxzF5Y</a></p>&mdash; シェル芸bot (@minyoruminyon) <a href="https://twitter.com/minyoruminyon/status/1363348644485689346?ref_src=twsrc%5Etfw">February 21, 2021</a></blockquote> <script async src="https://platform.twitter.com/widgets.js" charset="utf-8"></script>



TL上にいる本物のシェル芸人による凄すぎる作品を2,3年見て私は感覚がすっかり麻痺しています。

何も分からない。

<blockquote class="twitter-tweet"><p lang="und" dir="ltr"><a href="https://t.co/Wk59NffwmM">https://t.co/Wk59NffwmM</a> <a href="https://t.co/Yr0IGK2n2k">pic.twitter.com/Yr0IGK2n2k</a></p>&mdash; シェル芸bot (@minyoruminyon) <a href="https://twitter.com/minyoruminyon/status/1208051191256502272?ref_src=twsrc%5Etfw">December 20, 2019</a></blockquote> <script async src="https://platform.twitter.com/widgets.js" charset="utf-8"></script>


### ○○する芸能ちゃんブロック

何でこんな出題をするかと言えば、<font style="color:red;">この動画を紹介したかったから</font>に他なりません。


私が7人目で分かったからという訳では無いですが、**6人目まではさっぱり分からない**と思います。

動画の半分まで何のことかさっぱり分からないかもしれませんが、良い問題なので是非見て頂きたい…！

<div class="youtube">
<iframe width="560" height="315" src="https://www.youtube.com/embed/FeX4YxtbySs" title="YouTube video player" frameborder="0" allow="accelerometer; autoplay; clipboard-write; encrypted-media; gyroscope; picture-in-picture" allowfullscreen></iframe>
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

.col-md-9 > iframe,img {
  width: 20% !important;
}


</style>