---
articleTitle: イイかイカンか審議じゃ（○○するシェル芸ちゃん_12日目）
Keywords: ○○するシェル芸ちゃん, bash, シェル芸,mecab, uconv
Copyright: (C) 2021 sickleaf
---


<br>

[○○するシェル芸ちゃん](../../?page=advent_shellgeichan)アドベントカレンダーの12日目です。
<br><br>

悪いことは言わないので、もし未見の人は[アナグラムの穴](https://www.youtube.com/watch?v=x_U5juzQApQ)は是非。

## 問題

```
複数行書かれた日本語文のうち、「き」「し」「や」「か」「い」「け」「ん」の文字と、それらの濁音・拗音と長音だけで構成されているかどうかを判定して下さい。ただし単語の区切りのカンマは無視するものとします。
```

### 問題データ(`cat data`)

```
イイかイカンか審議じゃ
奇奇怪怪
薬缶やジャー解禁
キンキン限界か
詩吟体験開催
指揮官,見解
金塊,換金,現金,大金
市議会,市議,献金禁止期間
```

奇奇怪怪はOK、詩吟体験開催はNGと判定するようなイメージです。

もじぴったんだったら`「きしゃかいけん」を1コ作れ　『記者が帰社するまで終わりません』`みたいなことになってそう。知らんけど。

今回の問題はコマンドを実行する環境に大いに影響されるので、基本的には[こちらのサイト](https://websh.jiro4989.com/)で実行することをオススメします。


## 回答方針

記事執筆にあたりraspberry piにインストールしたので、その備忘録も兼ねています。

### 許容される文字について

まずどの文字が使えるのか確認しましょう。

`「き」「し」「や」「か」「い」「け」「ん」の文字と、それらの濁音・拗音と長音`です。後はカンマ。

つまり、**きしやかいけんぎじがげゃー,**のみで出来ていればOKです。

`grep -o .`で1文字ずつにバラして、`grep -E [きしやかいけんぎじがげゃー,]`とすれば何か出来そうな感じがしますね。

ただ今回は問題データにひらがな・カタカナ・漢字が混じっております。この変換をしなければいけません。

### `mecab`で漢字の読みを取得

[MeCab](https://taku910.github.io/mecab/)という、日本語の形態素解析を行うオープンソースのツールがあります。例えばこんな感じ。

```
$ echo 庭には二羽鶏がいる | mecab
庭	名詞,一般,*,*,*,*,庭,ニワ,ニワ
に	助詞,格助詞,一般,*,*,*,に,ニ,ニ
は	助詞,係助詞,*,*,*,*,は,ハ,ワ
二羽	名詞,固有名詞,人名,姓,*,*,二羽,ニワ,ニワ
鶏	名詞,一般,*,*,*,*,鶏,ニワトリ,ニワトリ
が	助詞,格助詞,一般,*,*,*,が,ガ,ガ
いる	動詞,自立,*,*,一段,基本形,いる,イル,イル
EOS
```

[使い方](https://taku910.github.io/mecab/#format)にある通り`-Oyomi`とすると、その読みだけを得ることが出来ます。

```
$ echo 庭には二羽鶏がいる | mecab -Oyomi
ニワニハニワニワトリガイル
```

#### 辞書問題

**mecabはあらゆる日本語を正しく読み込めるわけではありません。**

変換にあって読み込み辞書の登録が可能で、辞書を登録することによってその精度を上げることができます。

先ほどの`庭には二羽鶏がいる`の読みを出力したのは[強い辞書](https://github.com/neologd/mecab-ipadic-neologd.git)を読み込んだ環境で実行したものです。

ちなみにデフォルトの辞書ではこの出力になります。

```
$ echo 庭には二羽鶏がいる | mecab -Oyomi
にわにはふたわにわとりがいる
```

今後の回答で示すのは、この強い辞書を読み込んでいる[シェル芸bot](https://github.com/theoremoon/ShellgeiBot-Image)に準拠した環境で実行した結果を出力しています。

先ほどの「庭には二羽鶏がいる」の読みの出力は[こちら](https://websh.jiro4989.com/?code=echo%20%E5%BA%AD%E3%81%AB%E3%81%AF%E4%BA%8C%E7%BE%BD%E9%B6%8F%E3%81%8C%E3%81%84%E3%82%8B%20%7C%20mecab%20-Oyomi)でも確認出来ます。


#### 強い辞書の読み込み

ここからは備忘録です。

```
## 強い辞書の読み込み
git clone --depth 1 https://github.com/neologd/mecab-ipadic-neologd.git

cd mecab-ipadic-neologd/

./bin/install-mecab-ipadic-neologd -n
```

シェル芸botのイメージの元になる[Dockerfile](https://github.com/theoremoon/ShellgeiBot-Image/blob/master/Dockerfile)から抜粋
```
# mecab-ipadic-NEologd
RUN git clone --depth 1 https://github.com/neologd/mecab-ipadic-neologd \
    && (cd mecab-ipadic-neologd && ./bin/install-mecab-ipadic-neologd -u -y -p $PWD/ipadic-utf8)
```


#### raspberry pi（debianベース）にインストールする場合

raspberry pi環境にシェル芸botイメージに入っている辞書と同じ辞書を入れようとしたら[メモリ不足でエラー](https://sanshonoki.hatenablog.com/entry/2018/10/09/231345)吐いたので終了。


```
## mecabのインストール
$ sudo apt install -y mecab libmecab-dev
```


```
radipi@raspberrypi:~/Repository/mecab-ipadic-neologd $ ./bin/install-mecab-ipadic-neologd -n
[install-mecab-ipadic-NEologd] : Start..
[install-mecab-ipadic-NEologd] : Check the existance of libraries
[install-mecab-ipadic-NEologd] :     find => ok
[install-mecab-ipadic-NEologd] :     sort => ok
[install-mecab-ipadic-NEologd] :     head => ok
[install-mecab-ipadic-NEologd] :     cut => ok
[install-mecab-ipadic-NEologd] :     egrep => ok
[install-mecab-ipadic-NEologd] :     mecab => ok
[install-mecab-ipadic-NEologd] :     mecab-config => ok
[install-mecab-ipadic-NEologd] :     make => ok
[install-mecab-ipadic-NEologd] :     curl => ok
[install-mecab-ipadic-NEologd] :     sed => ok
[install-mecab-ipadic-NEologd] :     cat => ok
[install-mecab-ipadic-NEologd] :     diff => ok
[install-mecab-ipadic-NEologd] :     tar => ok
[install-mecab-ipadic-NEologd] :     unxz => ok
[install-mecab-ipadic-NEologd] :     xargs => ok
[install-mecab-ipadic-NEologd] :     grep => ok
[install-mecab-ipadic-NEologd] :     iconv => ok
[install-mecab-ipadic-NEologd] :     patch => ok
[install-mecab-ipadic-NEologd] :     which => ok
[install-mecab-ipadic-NEologd] :     file => ok
[install-mecab-ipadic-NEologd] :     openssl => ok
[install-mecab-ipadic-NEologd] :     awk => ok
（中略）
/home/radipi/Repository/mecab-ipadic-neologd/bin/../libexec/make-mecab-ipadic-neologd.sh: 525 行: 17284 強制終了            ${MECAB_LIBEXEC_DIR}/mecab-dict-index -f UTF8 -t UTF8
```

### `uconv`による変換

mecabでカタカナで読みを得られましたが、ひらがなに変換します。


```
$ echo 庭には二羽鶏がいる | mecab -Oyomi | uconv -x hiragana
にわにはにわにわとりがいる
```

#### raspberry piへのuconvインストール

コマンドが入っているパッケージ名の調査ってちゃんとあるんだろうけど、結局ググって終わりにしてしまう。。

```
## uconvインストール
sudo apt install -y icu-devtools
```

### 指定された文字で使われていない文字を取得する

まず、`data`から1行ずつ読みを取得します。

```
$ cat data | while read a; do echo $a | mecab -Oyomi | uconv -x hiragana; done
いいかいかんかしんぎじゃ
ききかいかい
やかんやじゃあかいきん
きんきんげんかいか
しぎんたいけんかいさい
しきかん,けんかい
きんかい,かんきん,げんきん,たいきん
しぎかい,しぎ,けんきんきんしきかん
```

[実行リンク](https://websh.jiro4989.com/?code=tee%20%3E%20data%20%3C%3CEOF%0A%E3%82%A4%E3%82%A4%E3%81%8B%E3%82%A4%E3%82%AB%E3%83%B3%E3%81%8B%E5%AF%A9%E8%AD%B0%E3%81%98%E3%82%83%0A%E5%A5%87%E5%A5%87%E6%80%AA%E6%80%AA%0A%E8%96%AC%E7%BC%B6%E3%82%84%E3%82%B8%E3%83%A3%E3%83%BC%E8%A7%A3%E7%A6%81%0A%E3%82%AD%E3%83%B3%E3%82%AD%E3%83%B3%E9%99%90%E7%95%8C%E3%81%8B%0A%E8%A9%A9%E5%90%9F%E4%BD%93%E9%A8%93%E9%96%8B%E5%82%AC%0A%E6%8C%87%E6%8F%AE%E5%AE%98%2C%E8%A6%8B%E8%A7%A3%0A%E9%87%91%E5%A1%8A%2C%E6%8F%9B%E9%87%91%2C%E7%8F%BE%E9%87%91%2C%E5%A4%A7%E9%87%91%0A%E5%B8%82%E8%AD%B0%E4%BC%9A%2C%E5%B8%82%E8%AD%B0%2C%E7%8C%AE%E9%87%91%E7%A6%81%E6%AD%A2%E6%9C%9F%E9%96%93%0AEOF%0Acat%20data%20%7C%20while%20read%20a%3B%20do%20echo%20%24a%20%7C%20mecab%20-Oyomi%20%7C%20uconv%20-x%20hiragana%20%20%3B%20done)


ひらがなに出来たので、grepで使ってはいけない文字を使っているかどうかのチェックを行います。

"きしやかいけんぎじがげゃー,"**以外**の文字があった場合、それを表示してみましょう。

```
$ cat data | while read a; do  miss=$(echo $a | mecab -Oyomi | uconv -x hiragana | grep -o . |  grep -Ev [きしやかいけんぎじがげゃー,]); echo "$a:($(echo $miss))"; done
イイかイカンか審議じゃ:()
奇奇怪怪:()
薬缶やジャー解禁:(あ)
キンキン限界か:()
詩吟体験開催:(た さ)
指揮官,見解:()
金塊,換金,現金,大金:(た)
市議会,市議,献金禁止期間:()
```

[実行リンク](https://websh.jiro4989.com/?code=tee%20%3E%20data%20%3C%3CEOF%0A%E3%82%A4%E3%82%A4%E3%81%8B%E3%82%A4%E3%82%AB%E3%83%B3%E3%81%8B%E5%AF%A9%E8%AD%B0%E3%81%98%E3%82%83%0A%E5%A5%87%E5%A5%87%E6%80%AA%E6%80%AA%0A%E8%96%AC%E7%BC%B6%E3%82%84%E3%82%B8%E3%83%A3%E3%83%BC%E8%A7%A3%E7%A6%81%0A%E3%82%AD%E3%83%B3%E3%82%AD%E3%83%B3%E9%99%90%E7%95%8C%E3%81%8B%0A%E8%A9%A9%E5%90%9F%E4%BD%93%E9%A8%93%E9%96%8B%E5%82%AC%0A%E6%8C%87%E6%8F%AE%E5%AE%98%2C%E8%A6%8B%E8%A7%A3%0A%E9%87%91%E5%A1%8A%2C%E6%8F%9B%E9%87%91%2C%E7%8F%BE%E9%87%91%2C%E5%A4%A7%E9%87%91%0A%E5%B8%82%E8%AD%B0%E4%BC%9A%2C%E5%B8%82%E8%AD%B0%2C%E7%8C%AE%E9%87%91%E7%A6%81%E6%AD%A2%E6%9C%9F%E9%96%93%0AEOF%0Acat%20data%20%7C%20while%20read%20a%3B%20do%20%20miss%3D%24%28echo%20%24a%20%7C%20mecab%20-Oyomi%20%7C%20uconv%20-x%20hiragana%20%7C%20grep%20-o%20.%20%7C%20%20grep%20-Ev%20%5B%E3%81%8D%E3%81%97%E3%82%84%E3%81%8B%E3%81%84%E3%81%91%E3%82%93%E3%81%8E%E3%81%98%E3%81%8C%E3%81%92%E3%82%83%E3%83%BC%2C%5D%29%3B%20echo%20%22%24a%3A%28%24%28echo%20%24miss%29%29%22%3B%20done)

`grep -v`で、マッチしなかった文字を拾っています。

ここまで来れば、`miss`の中身が空かどうかで判定すれば完了です。

## 回答例

`cat data | while read a; do miss=$(echo $a | mecab -Oyomi | uconv -x hiragana | grep -o . |  grep -Ev [きしやかいけんぎじがげゃー,]); [ "$miss" = "" ] && echo "$a:OK" || echo "$a:NG($(echo $miss))" ; done`

```
$ cat data | while read a; do miss=$(echo $a | mecab -Oyomi | uconv -x hiragana | grep -o . |  grep -Ev [きしやかいけんぎじがげゃー,]); [ "$miss" = "" ] && echo "$a:OK" || echo "$a:NG($(echo $miss))" ; done
イイかイカンか審議じゃ:OK
奇奇怪怪:OK
薬缶やジャー解禁:NG(あ)
キンキン限界か:OK
詩吟体験開催:NG(た さ)
指揮官,見解:OK
金塊,換金,現金,大金:NG(た)
市議会,市議,献金禁止期間:OK
```


[実行リンク](https://websh.jiro4989.com/?code=tee%20%3E%20data%20%3C%3CEOF%0A%E3%82%A4%E3%82%A4%E3%81%8B%E3%82%A4%E3%82%AB%E3%83%B3%E3%81%8B%E5%AF%A9%E8%AD%B0%E3%81%98%E3%82%83%0A%E5%A5%87%E5%A5%87%E6%80%AA%E6%80%AA%0A%E8%96%AC%E7%BC%B6%E3%82%84%E3%82%B8%E3%83%A3%E3%83%BC%E8%A7%A3%E7%A6%81%0A%E3%82%AD%E3%83%B3%E3%82%AD%E3%83%B3%E9%99%90%E7%95%8C%E3%81%8B%0A%E8%A9%A9%E5%90%9F%E4%BD%93%E9%A8%93%E9%96%8B%E5%82%AC%0A%E6%8C%87%E6%8F%AE%E5%AE%98%2C%E8%A6%8B%E8%A7%A3%0A%E9%87%91%E5%A1%8A%2C%E6%8F%9B%E9%87%91%2C%E7%8F%BE%E9%87%91%2C%E5%A4%A7%E9%87%91%0A%E5%B8%82%E8%AD%B0%E4%BC%9A%2C%E5%B8%82%E8%AD%B0%2C%E7%8C%AE%E9%87%91%E7%A6%81%E6%AD%A2%E6%9C%9F%E9%96%93%0AEOF%0Acat%20data%20%7C%20while%20read%20a%3B%20do%20miss%3D%24%28echo%20%24a%20%7C%20mecab%20-Oyomi%20%7C%20uconv%20-x%20hiragana%20%7C%20grep%20-o%20.%20%7C%20%20grep%20-Ev%20%5B%E3%81%8D%E3%81%97%E3%82%84%E3%81%8B%E3%81%84%E3%81%91%E3%82%93%E3%81%8E%E3%81%98%E3%81%8C%E3%81%92%E3%82%83%E3%83%BC%2C%5D%29%3B%20%5B%20%22%24miss%22%20%3D%20%22%22%20%5D%20%26%26%20echo%20%22%24a%3AOK%22%20%7C%7C%20echo%20%22%24a%3ANG%28%24%28echo%20%24miss%29%29%22%20%3B%20done)

## ○○する芸能ちゃんブロック


今回の引用ネタはこちら。

今回テストデータを作る際に、確かに「きしゃかいけん」の文字は結構使い回しが効くことがよく分かりました。

<div class="youtube">
<iframe width="560" height="315" src="https://www.youtube.com/embed/CLlOmvzjQ2Q" title="YouTube video player" frameborder="0" allow="accelerometer; autoplay; clipboard-write; encrypted-media; gyroscope; picture-in-picture" allowfullscreen></iframe>
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
