---
articleTitle: 伝言ゲームの限界にダイアログで挑戦しよう（○○するシェル芸ちゃん_17日目）
Keywords: ○○するシェル芸ちゃん, bash, シェル芸, whiptail
Copyright: (C) 2021 sickleaf
---


<br>

[○○するシェル芸ちゃん](../../?page=advent_shellgeichan)アドベントカレンダーの17日目です。
<br><br>

予告ラインアップ２つ目の延長戦です。昨日のが1.5日分、今日のが0.5日分です。

<font style="color:white;">ただ書くのに2日掛けちゃうとはね！	</font>

## 問題

今回は[16日目](../../?post=shellgeichan_day16_R5ATsQ)の問題・回答をベースに進みますのでそちらを参照して下さい。

```
「伝言ゲームの限界に挑戦しよう」をターミナルで実現した仕組みについて、whiptailを使って入出力をダイアログで表示するようにして下さい。
```

見た目をもう少しマシにしよう、というのが目的です。

## 回答方針

問題の指定の通り`whiptail`を使います。ダイアログを利用するためのコマンドで、ヒントや回答の入力・出力部分で指定している`read`を置き換えます。

基本的に困ったら[マニュアル](https://linux.die.net/man/1/whiptail)を参照しますが、ぱっと見は[こちら](https://ytyaru.hatenablog.com/entry/2020/04/24/000000)や[こちら](https://www.kabipan.com/densan/whiptail.html)のサイトがオススメです。

ダイアログといっても、情報だけ表示するダイアログや変数を入力するダイアログなど様々存在します。`whiptail`は各ダイアログを表示するためのダイアログを備えているので、適切なオプションと共に使用する必要があります。


### 不要なクリア処理を消す

まずは[元々](../../?post=shellgeichan_day16_R5ATsQ)の回答から作っていきます。

```
theme="トリビアの泉"; hint=$theme; (for i in $(seq 3); do clear; echo $hint（削除文字数：${deleteNum:-"0"}文字）; printf "[No.%d_hint] " $i; while true; do read thishint; [ "$(echo $thishint | grep -o '[亜-熙]' | grep -c ^ )" -le 7 ] && break || echo "※ヒントは7文字以内"; done; hint=$(echo $thishint | grep -o '[亜-熙]' | grep -vf <(echo $theme$used | grep -o . | grep -v "/") | tr -d "\n"); deleteNum=$(echo $thishint | grep -o '[亜-熙]' | grep -f <(echo $theme$used | grep -o . ) | wc -l );used=$used"/"$thishint; done; clear; echo "$hint（削除文字数：${deleteNum:-"0"}文字）"; echo "[[FINALANSWER]]"; read finalans; [ $finalans"x" = $theme"x" ] && judge="正解！！" || judge="残念..."; echo $judge; echo $used | tr "/" "\n" | nl)
```

あまりに長いですが、これを編集していきます。

**シェル芸の設問として置いているから行っている作業であって、まともな作業ではないことは頭の中心に置いて進めましょう。**

この時は回答者が変わる度に過去の文言が見えてしまっては意味がないため、ループが回る度に`clear`を行っていますが、画面全面のダイアログ表示で代替可能なため、`clear`はカットします。

### ヒント入力画面のダイアログ設定

ではダイアログを設定していきます。

ダイアログでは入力した値を変数に入れることが出来るため、`read`コマンドの置き換えを行います。

また、ヒントなどの情報表示を行う`echo`や`printf`もまとめて行います。

##### 置換前

置き換え前のヒント入力部分は次の通り。便宜上、カンマで改行して表示します。

いよいよワンライナーでは無くなったが、最終的な回答例は1行で収めるので。一旦。

```
echo $hint（削除文字数：${deleteNum:-"0"}文字）;
printf "[No.%d_hint] " $i;
while true;
do read thishint;
[ "$(echo $thishint | grep -o '[亜-熙]' | grep -c ^ )" -le 7 ] && break || echo "※ヒントは7文字以内";
done;
```


##### 置換後

```
emess=""
while true;
do thishint=$(whiptail --nocancel --inputbox "$i人目：ヒント入力\n\n$hint\n（削除文字数：${deleteNum:-"0"}文字）\n${emess}" --title "伝言ゲームの限界に挑戦しよう" 0 0 3>&1 1>&2 2>&3);
[ "$(echo $thishint | grep -o '[亜-熙]' | grep -c ^ )" -le 7 ] && break || emess="※ヒントは7文字以内";
done;
```

変更箇所は（１）ヒントのための情報表示をダイアログ内に集約、（２）エラーメッセージの変数表示の2点です。

CUI版ではエラーチェック・メッセージ表示・変数再入力が同じ画面内に収まるため、先にヒントを表示してチェックに引っ掛かった場合だけエラーを表示していました。

ダイアログ版では<font style="color:red;">チェックに引っ掛かった場合も同じダイアログの呼び出しになる</font>ため、エラー文言を変数へ外出しし、チェックに引っ掛かった場合のみ変数の更新を行う形で実装している。


#### ヒント入力時のダイアログ表示について

`whiptail`で指定するオプションは下記4点に基づいて設定します。

・キャンセルは表示させないため`--nocancel`を指定

・画面いっぱいに表示するためheight,widthはいずれも0

・`--title`でタイトル表示

・`3>&1 1>&2 2>&3`と共に実行することにより、入力データを変数に格納

以下、`whiptail --help`より抜粋。
```
    --inputbox <text> <height> <width> [init]
    --nocancel          no cancel button
    --title <title>                 display title
```

#### ファイルディスクリプタ操作について

最後の`3>&1 1>&2 2>&3`は、ちょっと難解です。

>--inputbox text height width [init]
>
>An input box is useful when you want to ask questions that require the user to input a string as the answer. If init is supplied it is used to initialize the input string. When inputing the string, the BACKSPACE key can be used to correct typing errors. If the input string is longer than the width of the dialog box, the input field will be scrolled. On exit, the input string will be printed on stderr.

[マニュアル](https://linux.die.net/man/1/whiptail)の--inputboxの最後にある通り、入力した文字列は標準出力(`/dev/fd/1`)ではなく標準エラー出力(`/dev/fd/2`)に表示されるとあります。

入力された値を変数に格納するには、標準出力から受け取るようにする必要があります。

このため`3>&1 1>&2 2>&3`により、ファイルディスクリプタ3を一時保存として使い、標準出力と標準エラー出力を入れ替えています。

仕組みの詳しい解説は[こちら](https://www.kabipan.com/densan/whiptail.html)を参照のこと。


## 回答例

最終ヒントの部分は文言を変えるだけでロジックは同じなので説明は割愛します。

また、member`を外出しして参加人数を変更出来るようにしました。

```
theme="トリビアの泉"; member=4; hint=$theme; (for i in `seq $((member-1))`; do emess=""; while true; do thishint=$(whiptail --nocancel --inputbox "$i人目：ヒント入力\n\n$hint\n（削除文字数：${deleteNum:-"0"}文字）\n${emess}" --title "伝言ゲームの限界に挑戦しよう" 0 0 3>&1 1>&2 2>&3); [ "$(echo $thishint | grep -o '[亜-熙]' | grep -c ^ )" -le 7 ] && break || emess="※ヒントは7文字以内"; done; hint=`echo $thishint | grep -o '[亜-熙]' | grep -vf <(echo $theme$used | grep -o . ) | tr -d "\n"`; deleteNum=$(echo $thishint |  grep -o '[亜-熙]' | grep -f <(echo $theme$used | grep -o . ) | wc -l ); used=$used"/"$thishint; done; finalans=$(whiptail --nocancel --inputbox "最終ヒント\n\n$hint\n（削除文字数：${deleteNum:-"0"}文字）"  --title "伝言ゲームの限界に挑戦しよう" 0 0 3>&1 1>&2 2>&3);  [ $finalans"x" = $theme"x" ] && judge="正解！！" || judge="残念..."; whiptail --msgbox "$judge\n\n回答：$finalans\n正解：$theme\n\n$(echo $used | tr "/" "\n" | sed 1d | grep -n "")" 0 0;)
```

これはエラーチェックなどせず、ようやく実装できて嬉々として試している図です。

webshでは`whiptail`の動作デモが行えないため、こちらで雰囲気だけでも感じて下さい。

<blockquote class="twitter-tweet"><p lang="ja" dir="ltr">『漢字最大7文字で伝達し、お題・過去の伝言で使われた漢字は削除される』伝言ゲーム。絶対ロジックで組めるはず…と前々から思っていたので シェル芸で実装してみた。<a href="https://t.co/fqoPGA5LWU">https://t.co/fqoPGA5LWU</a><a href="https://twitter.com/hashtag/%E3%81%BE%E3%82%8B%E8%8A%B8%E3%81%A1%E3%82%83%E3%82%93?src=hash&amp;ref_src=twsrc%5Etfw">#まる芸ちゃん</a> <a href="https://t.co/YNFbJqp7Wt">pic.twitter.com/YNFbJqp7Wt</a></p>&mdash; 病葉(Yu Kidawara) (@sickleaf3) <a href="https://twitter.com/sickleaf3/status/1441671303535104002?ref_src=twsrc%5Etfw">September 25, 2021</a></blockquote> <script async src="https://platform.twitter.com/widgets.js" charset="utf-8"></script>

## ○○する芸能ちゃんブロック

このゲーム、8人で限界に挑戦するのも良いし、2チームに割って同じお題でそれぞれどう進行するか見ていくのも良い。

もの凄くよく出来ているので何とかしたい。。

<div class="youtube">
<iframe width="560" height="315" src="https://www.youtube.com/embed/itAsH7YLzXQ" title="YouTube video player" frameborder="0" allow="accelerometer; autoplay; clipboard-write; encrypted-media; gyroscope; picture-in-picture" allowfullscreen></iframe>
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