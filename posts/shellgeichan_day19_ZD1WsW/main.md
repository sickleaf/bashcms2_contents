---
articleTitle: M-1敗者復活出番順抽選順番サイコロ（○○するシェル芸ちゃん_19日目）
Keywords: ○○するシェル芸ちゃん, bash, シェル芸
Copyright: (C) 2021 sickleaf
---


<br>

[○○するシェル芸ちゃん](../../?page=advent_shellgeichan)アドベントカレンダーの19日目です。
<br><br>

「今年のM-1は良かった」ってここ数年言ってる気がする。

## 問題

```
M-1の敗者復活戦の登場出番のクジを引く順番を、50音順・50音逆順・エントリーナンバー順・エントリーナンバー逆順・アルファベット順・アルファベット逆順のいずれかで決まります。
敗者復活戦のメンバーの情報から、上記のそれぞれの並び順に従ってクジを引く順番を示して下さい。
表示データは「コンビ名　ソートするデータ」の順に2列で表示して下さい。
```


### M-1_2021年敗者復活戦メンバー(`cat data`)

```
entryNo name kana alphabet
2179 ハライチ ハライチ haraichi
2460 金属バット キンゾクバット kinzokubat
1219 男性ブランコ ダンセイブランコ danseiblanko
4000 見取り図 ミトリズ mitorizu
3914 東京ホテイソン トウキョウホテイソン tokyohoteison
4284 マユリカ マユイカ mayurika
3582 アインシュタイン アインシュタイン einstein
3980 ニューヨーク ニューヨーク newyork
3027 からし蓮根 カラシレンコン karashirrenkon
901 ヨネダ2000 ヨネダニセン yoneda2000
2291 アルコ＆ピース アルコアンドピース alcoandpeace
1942 さや香 サヤカ sayaka
2621 ヘンダーソン ヘンダーソン henderson
811 ダイタク ダイタク daitaku
3026 カベポスター カベポスター kabeposter
4379 キュウ キュウ kyu
```

## 回答方針

特に考えることはなく、sortしてawkで取り出すだけです。。

### データの並び順

どの表示順にするかによって`sort`でどの列を扱うかが変わってきます。

#### エントリーナンバー順

`sort -k1,1n`で、1列目のフィールドを数字の昇順に並びます。

#### 50音順

`sort -k3,3`で、3列目のフィールドを名前の昇順に並びます。

#### アルファベット順

`sort -k4,4`で、4列目のフィールドを名前の昇順に並びます。


それぞれ、逆順にする場合はオプションに`-r`を追加します。


### 表示データの加工

データの先頭はラベルで不要なので`sed 1d`で削除します。

また、ソートした後のデータは「コンビ名　ソートするデータ」で表示するため、`awk '{print $2,$1}'`とします。

$2の後ろはエントリーナンバー順の場合は`$1`、50音順は`$3`、アルファベット順は`$4`にします。

## 回答例

それぞれ表示します。

### 50音順・50音逆順

```
$ cat data | sed 1d | sort -k3,3 | awk '{print $2,$3}'
アインシュタイン アインシュタイン
アルコ＆ピース アルコアンドピース
カベポスター カベポスター
からし蓮根 カラシレンコン
キュウ キュウ
金属バット キンゾクバット
さや香 サヤカ
ダイタク ダイタク
男性ブランコ ダンセイブランコ
東京ホテイソン トウキョウホテイソン
ニューヨーク ニューヨーク
ハライチ ハライチ
ヘンダーソン ヘンダーソン
マユリカ マユイカ
見取り図 ミトリズ
ヨネダ2000 ヨネダニセン

$ cat data | sed 1d | sort -k3,3r | awk '{print $2,$3}'
ヨネダ2000 ヨネダニセン
見取り図 ミトリズ
マユリカ マユイカ
ヘンダーソン ヘンダーソン
ハライチ ハライチ
ニューヨーク ニューヨーク
東京ホテイソン トウキョウホテイソン
男性ブランコ ダンセイブランコ
ダイタク ダイタク
さや香 サヤカ
金属バット キンゾクバット
キュウ キュウ
からし蓮根 カラシレンコン
カベポスター カベポスター
アルコ＆ピース アルコアンドピース
アインシュタイン アインシュタイン
```

### エントリーナンバー順・逆順

```
$ cat data | sed 1d | sort -k1,1n | awk '{print $2,$1}'
ダイタク 811
ヨネダ2000 901
男性ブランコ 1219
さや香 1942
ハライチ 2179
アルコ＆ピース 2291
金属バット 2460
ヘンダーソン 2621
カベポスター 3026
からし蓮根 3027
アインシュタイン 3582
東京ホテイソン 3914
ニューヨーク 3980
見取り図 4000
マユリカ 4284
キュウ 4379

$ cat data | sed 1d | sort -k1,1rn | awk '{print $2,$1}'
キュウ 4379
マユリカ 4284
見取り図 4000
ニューヨーク 3980
東京ホテイソン 3914
アインシュタイン 3582
からし蓮根 3027
カベポスター 3026
ヘンダーソン 2621
金属バット 2460
アルコ＆ピース 2291
ハライチ 2179
さや香 1942
男性ブランコ 1219
ヨネダ2000 901
ダイタク 811
```

### アルファベット順・逆順

```
$ cat data | sed 1d | sort -k4,4 | awk '{print $2,$4}'
アルコ＆ピース alcoandpeace
ダイタク daitaku
男性ブランコ danseiblanko
アインシュタイン einstein
ハライチ haraichi
ヘンダーソン henderson
カベポスター kabeposter
からし蓮根 karashirrenkon
金属バット kinzokubat
キュウ kyu
マユリカ mayurika
見取り図 mitorizu
ニューヨーク newyork
さや香 sayaka
東京ホテイソン tokyohoteison
ヨネダ2000 yoneda2000

$ cat data | sed 1d | sort -k4,4r | awk '{print $2,$4}'
ヨネダ2000 yoneda2000
東京ホテイソン tokyohoteison
さや香 sayaka
ニューヨーク newyork
見取り図 mitorizu
マユリカ mayurika
キュウ kyu
金属バット kinzokubat
からし蓮根 karashirrenkon
カベポスター kabeposter
ヘンダーソン henderson
ハライチ haraichi
アインシュタイン einstein
男性ブランコ danseiblanko
ダイタク daitaku
アルコ＆ピース alcoandpeace
```


## ○○する芸能ちゃんブロック

M-1見ていない人にも説明不要だし、M-1見ている人には「なるほどそこか」となるテーマ。

しかも公開がM-1の翌日。ピント合うてるわー。

<div class="youtube">
<iframe width="560" height="315" src="https://www.youtube.com/embed/foh7TVn7CFU" title="YouTube video player" frameborder="0" allow="accelerometer; autoplay; clipboard-write; encrypted-media; gyroscope; picture-in-picture" allowfullscreen></iframe>
</div>

ちなみに本家は「何でやねん！」の後ではなく「お前」の後に間が入ります。

## M-1ブロック

ネタ動画は少し過ぎたら削除されてしまうので、消されないであろう出番順抽選会から。このシステムを見て「シェル芸ちゃんのネタに使える！！」と思いました。

<div class="youtube">
<iframe width="560" height="315" src="https://www.youtube.com/embed/FQaC74UkoGo" title="YouTube video player" frameborder="0" allow="accelerometer; autoplay; clipboard-write; encrypted-media; gyroscope; picture-in-picture" allowfullscreen></iframe>
</div>

「出番順抽選会」がコンテンツになるM-1凄すぎる。そしてこの面子も演芸番組としてめちゃくちゃ豪華。


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