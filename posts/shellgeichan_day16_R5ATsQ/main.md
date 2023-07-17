---
articleTitle: 伝言ゲームの限界にターミナルで挑戦しよう（○○するシェル芸ちゃん_16日目）
Keywords: ○○するシェル芸ちゃん, bash, シェル芸
Copyright: (C) 2021 sickleaf
---


<br>

[○○するシェル芸ちゃん](../../?page=advent_shellgeichan)アドベントカレンダーの16日目です。
<br><br>

予告ラインアップ２つ目。大ネタです。

## ○○する芸能ちゃんブロック

まずはこちらの動画の**0:44～1:54部分**をご覧下さい。<font style="color:red;">全部見ても構いませんよ。</font>

16日目・17日目は、[2日目](../../?post=shellgeichan_day02_c1dl)や[3日目](../../?post=shellgeichan_day03_wap9z)と同様、まる芸ちゃんの動画先行でそのまま問題にしてい
ますので、システムを理解するためにまず動画をご覧下さい。

<div class="youtube">
<iframe width="560" height="315" src="https://www.youtube.com/embed/q_Uz5iWT9ZE" title="YouTube video player" frameborder="0" allow="accelerometer; autoplay; clipboard-write; encrypted-media; gyroscope; picture-in-picture" allowfullscreen></iframe>
</div>

**超楽しい！**

## 問題

```
このゲームを端末を順番に操作する前提で4人でプレイできるようにして下さい。
お題はワンライナーの最初に変数として設定し、プレイヤーが変わる度に前のプレイヤーのヒントだけ表示され、次のヒントや回答を入力出来るようにして下さい。
```

#### ゲームルールの補足

一応ルール把握に必要な部分を抜粋しました。

<img src="https://user-images.githubusercontent.com/5824061/147215517-ddece8f8-e32f-43e7-a6cd-9c3d2eb1fe2e.png" width=10% height=10%>


<img src="https://user-images.githubusercontent.com/5824061/147215327-2386ed61-09be-41c2-a5fa-274d7a0c50dd.png" width=10% height=10%>


回答例は【トリビアの泉】で考えてみます。

ヒントを考えてみるならばこんな感じでしょうか。

・八十点以上粗品

・無駄知識台場泉

・八嶋高橋両司会

泉はどの順番でも入れたらダメなのは分かっていますが、動作確認のために入れています。

いやあこれはオフラインでやりたい。。

#### 出力イメージ

最後の回答者が入力した場合の端末の出力はこのようなイメージです。

```
嶋高橋両司会（削除文字数：1文字）
[[FINALANSWER]]
ガセビアの沼

残念...

回答：ガセビアの沼
正解：トリビアの泉
```

## 回答方針

大変プログラミング的な、システマチックなルールです。これこそプログラムで実装して然るべき内容でしょう。…ワンライナーがそれに適しているかは大いに疑問ですが。

ただ、**ノートPCを変わる変わる操作すれば手っ取り早く同じ体験が出来る**ので、そこまで悪くはないんじゃないかなと思っています。



### 操作者が変わる度に別の入力・出力を扱う

回答や漢字の重複などの判定ロジックは後から入れるとして、まずは枠組みを作りましょう。

まずは1番手から回答者まで、インプットになる情報の表示とホワイトボードの記入に相当する機能を作ります。

変数の表示は`echo`や`printf`、入力は`read`で行います。回答者は最終的な回答（`finalans`）を、それ以外はヒントの元データ（`thishint`）を入力します。

また何番目のヒントを表示しているかも表示するため、ループ内の変数`i`を使います。（ループは`for i in $(seq 3)`とし、回答者のロジックだけ例外のためループ終了後に記述します）

`for i in $(seq 3); do echo $hint; printf "[No.%d_hint] " $i; read thishint; hint=$thishint; done;`

`thishint`はそのループ内でしか使わないのに対し、`hint`はループが回った後や抜けた後にも使う「一番最新のヒント」のため使い回せるようにします。

```
$ theme="トリビアの泉"; hint=$theme; for i in $(seq 3); do echo $hint; printf "[No.%d_hint] " $i; read thishint; hint=$thishint; done;  echo $hint; echo "[[FINALANSWER]]"; read finalans; echo "$judge"
トリビアの泉
[No.1_ans] 八十点以上粗品<ENTER>
八十点以上粗品
[No.2_ans] 無駄知識台場泉<ENTER>
無駄知識台場泉
[No.3_ans] 八嶋高橋両司会<ENTER>
八嶋高橋両司会
[[FINALANSWER]]
ガセビアの沼<ENTER>

```

上記では分かりやすくするため、「入力後にエンターキーを入力する箇所」に`<ENTER>`と入れました。

ここから個別のロジックを実装していきます。


### 回答の判定

判定結果(`judge`)は最終的な回答（`finalans`）とテーマ(`theme`)が一致するかどうか判定します。無回答だった場合に備えて、文字列の後に"x"を付与して判定します。

`[ $finalans"x" = $theme"x" ] && judge="正解！！" || judge="残念...";`

### ヒントが漢字7文字以内か判定

ヒントの入力は漢字7文字という制約があります。

`read thishint`でチェックせずに受け取っていますが、この部分を条件を満たさない場合は再入力を促すメッセージを出した上で表示するように置き換えます。

`while true`で無限ループを作り、`thishint`にある漢字が7文字以内かどうか判定します。

`while true; do read thishint; [ "$(echo $thishint | grep -o '[亜-熙]' | grep -c ^ )" -le 7 ] && break; || echo "※ヒントは7文字以内"; done;`


`grep -o '[亜-熙]'`により、`thishint`に含まれている漢字を縦に並びます。その行数をカウントし、7以下の場合はループを抜け、そうでない場合はメッセージを表示してやり直しとすればOKです。

```
$ theme="トリビアの泉"; hint=$theme; (for i in $(seq 3); do echo $hint; printf "[No.%d_hint] " $i; while true; do read thishint; [ "$(echo $thishint | grep -o '[亜-熙]' | grep -c ^ )" -le 7 ] && break || echo "※ヒントは7文字以内"; done; hint=$thishint; done; echo "$hint"; echo "[[FINALANSWER]]"; read finalans; [ $finalans"x" = $theme"x" ] && judge="正解！！" || judge="残念..."; echo $judge)
トリビアの泉
[No.1_ans] 八十点以上粗品<ENTER>
八十点以上粗品
[No.2_ans] 無駄知識台場泉無駄知識台場泉<ENTER>
※ヒントは7文字以内
無駄知識台場泉<ENTER>
無駄知識台場泉
[No.3_ans] 八嶋高橋両司会<ENTER>
八嶋高橋両司会
[[FINALANSWER]]
トリビアの泉<ENTER>
正解！！
```

### 禁止文字チェック

いつまで入力された文字をそのまま出しているんだと。一番やるべき実装をやります。

まず入力されたヒントは蓄積する必要があるため、ループが終わる前に以下を入力します。これにより、各々が入力したヒントがスラッシュ区切りで溜まっていきます。

`used=$used"/"$thishint`

そして、次に表示するヒントとして設定している`hint=$thishint`を、この`used`を使って本来設定すべき値にしましょう。

`hint=$(echo $thishint | grep -o '[亜-熙]' | grep -vf <(echo $theme$used | grep -o . | grep -v "/") | tr -d "\n");`

処理の流れは、`thishint`の中身を1文字ずつ縦に並べ、禁止文字（テーマの漢字＋既にヒントで使われた漢字）に該当するものを取り除き、元に戻しています。

ポイントは`grep -vf <(echo $theme$used | grep -o . | grep -v "/")`の部分で、ここでまさしく禁止文字チェックを行っています。

禁止文字はお題（`theme`）とスラッシュ区切りで入った過去ヒント（`used`）に含まれています。これを縦に並べ、区切りのスラッシュを除いた文字群が禁止文字のリストになります。

このリストと、入力されたヒントを縦に並べたもの（`echo $thishint | grep -o '[亜-熙]'`)と突き合わせます。

`grep -vf`では、禁止文字リストに含まれる文字をテキストファイルとして扱い、そちらに含まれない文字だけを表示することが出来ます。また、禁止文字リストをファイルとして扱うためにプロセス置換を行っています。

```
$ theme="トリビアの泉"; hint=$theme; (for i in $(seq 3); do echo $hint; printf "[No.%d_hint] " $i; while true; do read thishint; [ "$(echo $thishint | grep -o '[亜-熙]' | grep -c ^ )" -le 7 ] && break || echo "※ヒントは7文字以内"; done; hint=$(echo $thishint | grep -o '[亜-熙]' | grep -vf <(echo $theme$used | grep -o . | grep -v "/") | tr -d "\n"); used=$used"/"$thishint; done; echo "$hint"; echo "[[FINALANSWER]]"; read finalans; [ $finalans"x" = $theme"x" ] && judge="正解！！" || judge="残念..."; echo $judge)
トリビアの泉
[No.1_ans] 八十点以上粗品
八十点以上粗品
[No.2_ans] 無駄知識台場泉
無駄知識台場
[No.3_ans] 八嶋高橋両司会
嶋高橋両司会
[[FINALANSWER]]
ガセビアの沼
残念...
```

ワンライナーが結構なことになってきました。セミコロン多すぎる。

### 削除文字カウント

ヒントが本来のものになりましたが、ただ表示するだけでは不十分です。**何文字削られてそのヒントになったか**、という情報も出さなければなりません。

禁止文字チェックが出来たので、引っ掛かった文字数を数えれば終了です。

`deleteNum=$(echo $thishint |  grep -o '[亜-熙]' | grep -f <(echo $theme$used | grep -o . ) | wc -l );`

禁止文字チェックでは**禁止文字リストに含まれない**文字だけ通すためgrepに`-v`オプションがありましたが、ここでは引っ掛かった文字数が欲しいのでこのオプションは付けていません。

deleteNumが得られたらヒント表示時に一緒に表示します。ループの初回はお題を表示するだけなのでdeleteNumは空です。このため、デフォルトは0が表示されるように`${deleteNum:-"0"}`とします。

```
$ theme="トリビアの泉"; hint=$theme; (for i in $(seq 3); do echo $hint（削除文字数：${deleteNum:-"0"}文字）; printf "[No.%d_hint] " $i; while true; do read thishint; [ "$(echo $thishint | grep -o '[亜-熙]' | grep -c ^ )" -le 7 ] && break || echo "※ヒントは7文字以内"; done; hint=$(echo $thishint | grep -o '[亜-熙]' | grep -vf <(echo $theme$used | grep -o . | grep -v "/") | tr -d "\n"); deleteNum=$(echo $thishint | grep -o '[亜-熙]' | grep -f <(echo $theme$used | grep -o . ) | wc -l );used=$used"/"$thishint; done; echo "$hint（削除文字数：${deleteNum:-"0"}文字）"; echo "[[FINALANSWER]]"; read finalans; [ $finalans"x" = $theme"x" ] && judge="正解！！" || judge="残念..."; echo $judge)
トリビアの泉（削除文字数：0文字）
[No.1_ans] 八十点以上粗品
八十点以上粗品（削除文字数：0文字）
[No.2_ans] 無駄知識台場泉
無駄知識台場（削除文字数：1文字）
[No.3_ans] 八嶋高橋両司会
嶋高橋両司会（削除文字数：1文字）
[[FINALANSWER]]
ガセビアの沼
残念...
```

チェックを2回やっているのが冗長な気がしてなりませんが、もっと良い方法あるんでしょうか…。アプローチとしては普通だと思うので、シェル芸人による異常な別解求む。


### 入力リセット・ヒント全出力

ようやく終わりが見えてきました。ここまで出力を見やすくするために省略していましたが、前の人のヒントが見えていてはヒントから禁止文字を削った意味がありません。画面をクリアしましょう。ループが回る度に`clear`します。

また、最後の回答者が回答を入力した後はこれまでどのようなヒントが入力されたか見たくなります。これまでは表示を辿れば良かったのですが、`clear`してしまうので新たに出力する必要があります。

それまで入力されたヒントを蓄積している`used`を使いましょう。

`echo $used | tr "/" "\n" | nl`

ヒントの区切りに使っていたスラッシュごとに改行すれば、順番に回答が表示されます。

## 回答例

`theme="トリビアの泉"; hint=$theme; (for i in $(seq 3); do clear; echo $hint（削除文字数：${deleteNum:-"0"}文字）; printf "[No.%d_hint] " $i; while true; do read thishint; [ "$(echo $thishint | grep -o '[亜-熙]' | grep -c ^ )" -le 7 ] && break || echo "※ヒントは7文字以内"; done; hint=$(echo $thishint | grep -o '[亜-熙]' | grep -vf <(echo $theme$used | grep -o . | grep -v "/") | tr -d "\n"); deleteNum=$(echo $thishint | grep -o '[亜-熙]' | grep -f <(echo $theme$used | grep -o . ) | wc -l );used=$used"/"$thishint; done; clear; echo "$hint（削除文字数：${deleteNum:-"0"}文字）"; echo "[[FINALANSWER]]"; read finalans; [ $finalans"x" = $theme"x" ] && judge="正解！！" || judge="残念..."; echo $judge; echo $used | tr "/" "\n" | nl)`


これちゃんとWebで使えるようにしたい。漢字・ひらがな・カタカナ・アルファベットだけ扱えるようにすれば出来るはず…

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